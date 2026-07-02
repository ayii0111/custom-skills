
---
name: issue-explore
description:
argument-hint: [github_url]
arguments: [github_url]
disable-model-invocation: true
---

- 若用戶有提供明確的 GitHub URL $github_url 給你，你就去該倉庫網址，翻閱該 issue 分析一下這個產品的情況
- 反之若沒提供，取對話中最近一次出現的 GitHub URL 作為目標倉庫；若對話中出現多個不同倉庫，優先取最後一次提到的；若完全找不到，明確告知用戶「找不到可用的 GitHub URL，請提供」，不要臆測
