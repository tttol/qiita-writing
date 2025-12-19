---
title: Outboxパターンとは
tags:
  - 分散システム
private: false
updated_at: '2025-12-20T05:31:03+09:00'
id: f77ad79eda40896de67f
organization_url_name: null
slide: false
ignorePublish: false
---
Outboxパターンは、分散システムにおけるデータベース操作とメッセージング（イベント発行）の整合性を保証するための設計パターンです。

# 課題
例えば以下のようなJavaアプリの処理があったとします。
```java
@Transactional
public void createOrder(Order order) {
    orderRepository.save(order);  // DBに保存
    eventPublisher.publish(new OrderCreatedEvent(order));  // イベント発行
}
```

この実装の問題点は、DBトランザクションorイベント発行のどちらかが失敗した場合にデータの整合性が取れなくなる恐れがある点です。

- DB成功・イベント失敗
    - DBにはデータがあるが、イベントは失敗しているので期待する状態になっていない
- DB失敗・イベント成功
    - `@Transactional`アノテーションによりDBトランザクションはロールバックされるが、イベント発行処理はロールバックされない。
    - DBにデータがないが、イベント側は処理が走ってしまう。

# Outboxパターンによる解決法
Outboxテーブルという新しいテーブルを同じDBに作成します。そして先ほどの処理を以下のように修正します。
```sql
CREATE TABLE outbox (
    id UUID PRIMARY KEY,
    event_type VARCHAR,
    payload JSONB,
    status VARCHAR DEFAULT 'pending',
    retry_count INT DEFAULT 0,
    created_at TIMESTAMP,
    processed_at TIMESTAMP
);
```

```java
@Transactional
public void createOrder(Order order) {
    orderRepository.save(order); // 注文データを保存
    // Orderから OrderCreatedEvent を生成
    OrderCreatedEvent event = new OrderCreatedEvent(order);
    outboxRepository.save(event); // イベントをOutboxテーブルに保存
}
```

アプリケーション側の処理はこれで終了です。
イベント発行処理は別のプロセスに委譲します。

例えばAWS Lambdaなどで、Outboxテーブルをポーリングしてイベント発行する処理を書きます。
```py
def process_outbox(self):
    """未処理イベントを取得して発行"""
    events = self.outbox_repository.find_unprocessed()

    for event in events:
        try:
            # イベント発行
            self.event_publisher.publish(event)
            # Outboxテーブルを更新
            self.outbox_repository.mark_as_processed(event.id)
            logger.info(f"Processed event: {event.id}")
        except Exception as e:
            # リトライ回数をインクリメント
            self.outbox_repository.increment_retry_count(event.id)

            # 上限を超えた場合はステータスを失敗に更新
            if event.retry_count >= MAX_RETRY:
                self.outbox_repository.mark_as_failed(event.id)
                # アラート通知など
                self.alert_service.notify_failed_event(event.id)

            logger.error(f"Failed to process event {event.id}: {e}")
```
このLambdaをEventbridgeなどで定期実行することでOutboxテーブルをポーリングすることができます。


以上の流れをシーケンス図に起こします。
```mermaid
sequenceDiagram
    participant Client as クライアント
    participant API as APIサーバー(Javaアプリ)
    participant DB as データベース
    participant Outbox as Outboxテーブル
    participant Worker as Outboxワーカー(Lambda)
    participant MQ as メッセージキュー<br/>(Kafka/SNS等)
    participant Sub as サブスクライバー

    %% 通常のフロー
    rect rgb(200, 220, 240)
        Note over Client,Outbox: Phase 1: トランザクション内での書き込み
        Client->>API: 注文作成リクエスト
        activate API
        API->>DB: BEGIN TRANSACTION
        API->>DB: INSERT INTO orders
        DB-->>API: OK
        API->>Outbox: INSERT INTO outbox<br/>(イベントデータ)
        Outbox-->>API: OK
        API->>DB: COMMIT
        DB-->>API: OK
        API-->>Client: 注文作成完了
        deactivate API
    end

    rect rgb(220, 240, 200)
        Note over Worker,Sub: Phase 2: 非同期でのイベント発行
        loop ポーリング (例: 1秒ごと)
            Worker->>Outbox: SELECT * FROM outbox<br/>WHERE status = 'pending'<br/>ORDER BY created_at
            Outbox-->>Worker: 未処理イベント一覧
            
            alt イベントが存在する場合
                Worker->>MQ: イベント発行<br/>(OrderCreatedEvent)
                activate MQ
                MQ-->>Worker: ACK
                deactivate MQ
                Worker->>Outbox: UPDATE outbox<br/>SET processed_at = NOW()
                Outbox-->>Worker: OK
                
                MQ->>Sub: イベント配信
                activate Sub
                Sub->>Sub: イベント処理<br/>(在庫更新、通知送信等)
                Sub-->>MQ: ACK
                deactivate Sub
            end
        end
    end

    %% エラーケース
    rect rgb(255, 220, 220)
        Note over Worker,MQ: Phase 3: リトライ処理 (エラー時)
        Worker->>Outbox: SELECT未処理イベント
        Outbox-->>Worker: イベント取得
        Worker->>MQ: イベント発行試行
        MQ--xWorker: エラー (接続失敗等)
        Note over Worker: 次のポーリングで<br/>自動的にリトライ
    end
```


# Outboxパターンのポイント
- イベント発行処理をワーカー側(Lambda)に委譲できる
    - Javaアプリ側はOutboxテーブルへのINSERTまで担保できていればOK。単純に2つのテーブルへのコミットを1つのトランザクションで行うだけなので、特に複雑な実装は不要。
    - イベントの発行自体はワーカーが責任を持つ
    - 上限回数までリトライしてもイベントに失敗する場合は、別途そのことを通知する仕組みをワーカー側もしくはCloudWatchなどのアラートで通知する必要がある。
- イベントは非同期処理で行う前提
    - イベントが正常終了した後に別の処理を行う、といった要件がある場合はOutboxパターンは適していない。
    - その場合は同期処理を使った別の実装が必要

# 注意点
- **べき等性の保証が必要**
    - ワーカーがイベント発行後、Outboxテーブルの更新前に失敗すると、同じイベントが複数回発行される可能性があります
    - サブスクライバー側で、同じイベントを複数回受け取っても問題ないように実装する必要があります
    - 例：イベントIDを記録して、既に処理済みのイベントはスキップするなど

# 参考

https://docs.aws.amazon.com/ja_jp/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html

https://miraitranslate-tech.hatenablog.jp/entry/2022/09/20/120000

