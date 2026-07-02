---
name: install-skill
description: 安裝指定 cc skill
argument-hint: [github倉庫] [全局安裝選項 -g] [具體 skill 內文]
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Grep, WebFetch, Bash
---

Feature: 安裝指定 cc skill
  將一個 skill 安裝至使用者層級或專案層級的 skills 目錄。
  每個 skill 為一個獨立目錄，內含 SKILL.md。
  參數：[repo_url] [global_option] [skill_content]
  其中 repo_url 與 skill_content 擇一必填。

  Background:
    Given 安裝規格依循 cc 官網對 skill 的規範
    And 使用者層級的 skills 目錄為 ~/.claude/skills/
    And 專案層級的 skills 目錄為 當前專案的 .claude/skills/
    And 每個 skill 安裝為 <skills 目錄>/<目錄名>/SKILL.md
    And 若目標目錄不存在則先建立該目錄

  Rule: repo_url 與 skill_content 必須擇一提供

    Scenario: 兩者皆未提供
      Given repo_url 為空
      And skill_content 為空
      Then 安裝失敗並提示「repo_url 與 skill_content 擇一必填」

    Scenario: 兩者同時提供
      Given repo_url 與 skill_content 皆有值
      Then 以 repo_url 為主，並提示已忽略 skill_content

  Rule: 依輸入來源決定 skill 內容與目錄名

    Scenario: 由 repo_url 取得內容（單一）
      Given 已提供 repo_url
      And 該倉庫中只有一個可安裝的 skill
      When 從該倉庫下載其 SKILL.md（使用 WebFetch 或 gh/curl）
      And 讀取下載回來的 SKILL.md 內容
      Then 以該內容作為要安裝的 skill 內容
      And 以原倉庫的專案名稱作為 skill 的目錄名

    Scenario: 由 repo_url 取得內容（多個）
      Given 已提供 repo_url
      And 該倉庫中有多個可安裝的 skill
      Then 逐一列出並詢問是否安裝
      And 僅下載並安裝使用者確認要安裝的項目
      And 各項皆以其原始 skill 目錄名作為目錄名

    Scenario: 由 skill_content 直接提供內容
      Given 已提供 skill_content
      Then 以 skill_content 作為要安裝的 skill 內容
      And 以該內容 frontmatter 中的 name 欄位作為 skill 的目錄名
      And 若內容缺少 name 欄位則安裝失敗並提示需提供 name

  Rule: 安裝前決定目錄前綴

    Scenario: 使用者輸入自訂前綴
      Given 已確認要安裝某個 skill
      When 詢問使用者要使用的目錄前綴字串
      And 使用者輸入了非空字串
      Then 以該字串作為目錄前綴
      And 最終目錄名為 <前綴>-<目錄名>

    Scenario: 使用者未輸入前綴
      Given 已確認要安裝某個 skill
      When 詢問使用者要使用的目錄前綴字串
      And 使用者輸入為空
      Then 以當前專案目錄名稱作為目錄前綴
      And 最終目錄名為 <前綴>-<目錄名>

  Rule: 依 global_option 決定安裝位置

    Scenario: 全局安裝
      Given global_option 的值為 "-g"
      When 進行安裝
      Then 將 skill 寫入 ~/.claude/skills/<前綴>-<目錄名>/SKILL.md

    Scenario: 專案安裝（預設）
      Given global_option 為空
      When 進行安裝
      Then 將 skill 寫入 當前專案 .claude/skills/<前綴>-<目錄名>/SKILL.md

  Rule: 寫入前的安全檢查

    Scenario: 目標目錄已存在
      Given 目標路徑已有同名 skill 目錄
      Then 提示該 skill 已存在並詢問是否覆蓋，未確認前不覆蓋
