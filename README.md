# apm-skills

社内開発チーム向け、Agent Skills集約リポジトリです。
[Microsoft APM (Agent Package Manager)](https://github.com/microsoft/apm) 経由で各プロジェクトへ配布されます。

## 目次

* [概要](#概要)
* [収録Skill一覧](#収録skill一覧)
* [apm CLI のインストール](#apm-cli-のインストール)
* [使い方（利用側プロジェクト）](#使い方利用側プロジェクト)
* [このリポジトリに新しいSkillを追加する](#このリポジトリに新しいskillを追加する)
* [リリース（タグ付け）の手順](#リリースタグ付けの手順)
* [ディレクトリ構造](#ディレクトリ構造)

## 概要

このリポジトリは、Claude / GitHub Copilot / Cursor / Codex 等のAIエージェントクライアントで共通利用するSkill群を一元管理するためのものです。
各プロジェクトの `apm.yml` に依存として記載することで、`apm install` 時に自動的に各クライアントの規定ディレクトリへ展開されます。

* **Claude Code** -> `.claude/skills/<name>/`
* **GitHub Copilot / Cursor / OpenCode / Codex / Gemini** -> 統合パス `.agents/skills/<name>/` (converged path)

## 収録Skill一覧

収録されている Skill は [`.apm/skills/`](.apm/skills/) を参照してください。各ディレクトリの `SKILL.md` 冒頭の `description` が「いつ使うか」のサマリです。サードパーティ由来の Skill は同ディレクトリに `LICENSE` を同梱しています。

## apm CLI のインストール

消費側プロジェクトで `apm install` を実行するため、各メンバーの端末に apm CLI を入れる必要があります。

### 前提

* macOS / Windows / Linux (x86_64 または ARM64)
* Git (依存解決に必須)
* Python 3.10+ (`pip` 経由でインストールする場合のみ)

### macOS

#### Homebrew (推奨)

```bash
brew install microsoft/apm/apm
```

#### インストールスクリプト

```bash
curl -sSL https://aka.ms/apm-unix | sh
```

### Windows

#### Scoop (推奨)

```powershell
scoop bucket add apm https://github.com/microsoft/scoop-apm
scoop install apm
```

#### インストールスクリプト (PowerShell)

```powershell
irm https://aka.ms/apm-windows | iex
```

**Tip**: PowerShell の実行ポリシーで弾かれる場合は、管理者で起動した PowerShell から `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned` を実行してから再試行してください。

### 動作確認

```bash
apm --version
```

## 使い方（利用側プロジェクト）

### 1. `apm.yml` に依存を追加

各プロジェクトのルートに `apm.yml` を作成（既にあれば追記）：

```yaml
name: my-project
version: 0.1.0
description: "プロジェクトの説明"
repository: "https://github.com/your-username/my-project"

# 配置先クライアントを明示。自動検出に頼らず固定するのが推奨運用。
# 利用しているクライアントだけ列挙すればよい (例: geminiのみ、gemini + claude など)。
targets:
  - gemini
  - claude
  - copilot
  - codex
  # - cursor

dependencies:
  apm:
    # 特定のSkillだけ取り込む場合（<your-username> はご自身のGitHub IDに書き換えてください）
    # <your-username>/apm-skills/.apm/skills/code-review#0.1.0
    # またはパッケージ全体を取り込む
    - <your-username>/apm-skills#0.1.0
```

### 2. インストール

```bash
apm install
```

これで各クライアント用ディレクトリにSkillが配備されます。生成された `apm.lock.yml` は **必ずコミット** してください（再現性確保のため）。

## このリポジトリに新しいSkillを追加する

### 手順

1. `.apm/skills/<skill-name>/` ディレクトリを作成
2. その配下に `SKILL.md` を配置
3. `SKILL.md` 先頭にYAML frontmatterで `name` と `description` を記述
4. PRを作成・レビューを受ける
5. `main` へマージ

### リリース（タグ付け）の手順

**重要**: `apm.yml` の `version` と Git タグは **完全に同じ文字列** にすること（どちらも 'v' プレフィックス無し、例: `0.1.0`）。
APM は技術的には両者の一致を強制しないが、不一致は監査・トレーサビリティ上の事故になる。社内ルールとして必ず守る。

1. `apm.yml` の `version` を上げる (例: `0.1.0` -> `0.2.0`) - SemVer 形式
2. その変更をコミット・main へマージ
3. main の最新コミットに Git タグを切る - タグ名は `apm.yml` の version と完全一致 (例: `0.2.0`)
4. タグを push

```bash
# 例: 0.2.0 をリリースするとき
git tag 0.2.0
git push origin 0.2.0
```

### バージョニング規則 ([SemVer](https://semver.org/lang/ja/))

* **MAJOR** ('1.0.0' -> '2.0.0'): 既存Skillの破壊的変更（インターフェース変更、削除）
* **MINOR** ('0.1.0' -> '0.2.0'): 新Skillの追加、後方互換のある変更
* **PATCH** ('0.1.0' -> '0.1.1'): SKILL.md内文言の修正、軽微な改善

## ディレクトリ構造

```text
apm-skills/
├── apm.yml           # パッケージ定義
├── README.md
├── .gitignore
└── .apm/
    └── skills/
        └── code-review/
            └── SKILL.md
        └── etc
```

## 参考

* [APM 公式ドキュメント](https://microsoft.github.io/apm/)
* [APM Skills ガイド](https://microsoft.github.io/apm/guides/skills/)
* [APM Dependencies ガイド](https://microsoft.github.io/apm/guides/dependencies/)
* [microsoft/apm-action](https://github.com/microsoft/apm-action)
* [microsoft/apm-sample-package](https://github.com/microsoft/apm-sample-package)
