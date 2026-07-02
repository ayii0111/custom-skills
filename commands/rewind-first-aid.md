---
name: rewind-first-aid
description: rewind 過頭，要取得某次對話的回覆
argument-hint: [topic]
arguments: [topic]
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Grep, WebFetch, Bash
---

使用者剛才不小心 /rewind 對話過頭了，請先從 history 中找看看，是否還可以看到有關於 $topic 的對話紀錄，若有的話去 cc 的 context 中找看看，若還存在就幫他再次輸出該話題的回覆內容
