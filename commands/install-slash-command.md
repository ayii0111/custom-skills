---
name: install-slash-command
description: 安裝指定 cc slash command
argument-hint: [github倉庫] [全局安裝選項 -g] [具體 command 內文]
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Grep, WebFetch, Bash
---

Feature: 安裝指定 cc slash command
  將一個 slash command 安裝至使用者層級或專案層級的 commands 目錄。
  參數：[repo_url] [global_option] [command_content]
  其中 repo_url 與 command_content 擇一必填。

  Background:
    Given 安裝規格依循 cc 官網對 slash command 的規範
    And 使用者層級的 commands 目錄為 ~/.claude/commands/
    And 專案層級的 commands 目錄為 當前專案的 .claude/commands/
    And 若目標目錄不存在則先建立該目錄

  Rule: repo_url 與 command_content 必須擇一提供

    Scenario: 兩者皆未提供
      Given repo_url 為空
      And command_content 為空
      Then 安裝失敗並提示「repo_url 與 command_content 擇一必填」

    Scenario: 兩者同時提供
      Given repo_url 與 command_content 皆有值
      Then 以 repo_url 為主，並提示已忽略 command_content

  Rule: 依輸入來源決定 command 內容與檔名

    Scenario: 由 repo_url 取得內容（單一）
      Given 已提供 repo_url
      And 該倉庫中只有一個可安裝的 command/skill
      When 從該倉庫下載其 .md 檔（使用 WebFetch 或 gh/curl）
      And 讀取下載回來的 .md 內容
      Then 以該內容作為要安裝的 command 內容
      And 以原倉庫的專案名稱作為 .md 檔的檔名

    Scenario: 由 repo_url 取得內容（多個）
      Given 已提供 repo_url
      And 該倉庫中有多個可安裝的 command/skill
      Then 逐一列出並詢問是否安裝
      And 僅下載並安裝使用者確認要安裝的項目
      And 各項皆以其原始 .md 檔名作為檔名

    Scenario: 由 command_content 直接提供內容
      Given 已提供 command_content
      Then 以 command_content 作為要安裝的 command 內容
      And 以該內容 frontmatter 中的 name 欄位作為 .md 檔的檔名
      And 若內容缺少 name 欄位則安裝失敗並提示需提供 name

  Rule: 安裝前決定命令前綴

    Scenario: 使用者輸入自訂前綴
      Given 已確認要安裝某個 command
      When 詢問使用者要使用的命令前綴字串
      And 使用者輸入了非空字串
      Then 以該字串作為命令前綴
      And 最終檔名為 <前綴>-<檔名>.md

    Scenario: 使用者未輸入前綴
      Given 已確認要安裝某個 command
      When 詢問使用者要使用的命令前綴字串
      And 使用者輸入為空
      Then 以當前專案目錄名稱作為命令前綴
      And 最終檔名為 <前綴>-<檔名>.md

  Rule: 寫入前確保 frontmatter 含 disable-model-invocation

    Scenario: 被安裝的 command 缺少該選項
      Given 要安裝的 command 內容 frontmatter 中沒有 disable-model-invocation
      Then 在其 frontmatter 中加入 disable-model-invocation: true 後再寫入

    Scenario: 被安裝的 command 已有該選項
      Given 要安裝的 command 內容 frontmatter 已含 disable-model-invocation
      Then 將其值設為 true 後再寫入

  Rule: 依 global_option 決定安裝位置

    Scenario: 全局安裝
      Given global_option 的值為 "-g"
      When 進行安裝
      Then 將 command 寫入 ~/.claude/commands/<前綴>-<檔名>.md

    Scenario: 專案安裝（預設）
      Given global_option 為空
      When 進行安裝
      Then 將 command 寫入 當前專案 .claude/commands/<前綴>-<檔名>.md

  Rule: 寫入前的安全檢查

    Scenario: 目標檔案已存在
      Given 目標路徑已有同名 .md 檔
      Then 提示該 command 已存在並詢問是否覆蓋，未確認前不覆蓋
