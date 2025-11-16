---
title: FinchなどのDocker以外のランタイムから起動したコンテナをDocker CLIから操作できる理由
tags:
  - 'docker'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
# 疑問
なんでRDのコンテナをDcokerから操作できる？
# コンテナがrunするまでの全体図
- docker CLI
- socket
- dockerd, containerd
- runc
- cgroups
# Docker互換API、DOCKER_HOST、ソケット

# containerd, dockerd
# runc, cgroups
# まとめ
