---
name: session-clone
description: 克隆當前 session 到另一個目錄中
argument-hint: [dir_path]
arguments: [dir_path]
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Grep, WebFetch, Bash
---

在 cc 的 session 系統下建立 $dir_path 的 slug 目錄
並將 cc 系統中當前的 session 檔，複製一份到該目錄下
再將複製過去的 session 檔，於其中的 pwd 改為 $dir_path
