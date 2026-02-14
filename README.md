# dev-ops（運用リポジトリ）

このリポジトリは、WSL 上で Web アプリを量産・運用していくための **「司令塔」**です。
`template-fullstack`（テンプレ）や `projects/` 配下の各アプリ（子リポジトリ）を横断して、**運用ルール・台帳・共通スクリプト**を一元管理します。

---

## このリポジトリで管理すること

### 1. 開発運用ドキュメント

* WSL 上での基本運用
* Codex / Claude Code を使う際の作法（安全に回す手順）
* Git 運用（ブランチ/PR/コミットの基本）

> 目的：プロジェクトが増えても迷子にならない「標準手順」をここに置く

### 2. アプリ台帳（いま何があるか）

* どのアプリが存在するか
* Web/API の構成（Next.js / React Router / NodeTS / FastAPI）
* 状態（active / paused / idea / archived）
* URL やメモ（任意）

> 目的：いま Web アプリが何個あるか・どれが稼働中かを一瞬で把握する

### 3. 共通スクリプト（任意）

* 新規プロジェクト開始の補助（clone → 初期化 → 起動確認）
* 環境診断（Git設定、SSH、Docker、WSL などのチェック）

> 目的：繰り返し作業を減らす

---

## 前提：リポジトリ構造（WSL）

おすすめの配置は以下：

* テンプレ（親）

  * `~/src/templates/template-fullstack/`
* 実プロジェクト（子）

  * `~/src/projects/<project-name>/`
* 運用（本リポジトリ）

  * `~/src/ops/dev-ops/`（例）

> 注意：`templates/` と `projects/` は **別々の Git リポジトリ**。
> テンプレ親が子プロジェクトを内包するわけではありません。

---

## 新しいアプリを作る基本フロー（要点）

1. GitHub で `template-fullstack` から **Use this template** で新しいリモートリポジトリを作成
2. WSL の `~/src/projects` に clone
3. プロジェクト内で `make setup` / `make dev`（テンプレのREADMEに従う）
4. Codex / Claude Code で開発
5. `apps.yaml` にアプリを登録（台帳更新）

---

## このリポジトリのディレクトリ（例）

```
ops/
  README.md
  docs/
    development.md      # 開発運用（備忘録）
    ai-workflow.md      # Codex/Claude Code運用（任意）
    wsl-setup.md        # WSL前提/注意点（任意）
  catalog/
    apps.yaml           # アプリ台帳（必須）
    templates.yaml      # テンプレ台帳（任意）
  scripts/
    newapp.sh           # 新規開始補助（任意）
    doctor.sh           # 環境診断（任意）
  standards/
    naming.md           # 命名/ポート/ENVキー標準
    branching.md        # ブランチ/PR標準
    security.md         # トークン/SSH/Secrets標準
```

※最初は `docs/development.md` と `catalog/apps.yaml` だけでも十分です。

---

## 命名規則（推奨）

* Webアプリ（子）：`app-<name>`
* API/サービス：`svc-<name>`
* 共通ライブラリ：`lib-<name>`
* テンプレ：`template-<name>`
* 運用：`dev-ops`（本リポジトリ）

---

## 運用ルール（最小）

* **標準コマンド**は各プロジェクトで統一（`make setup / dev / test / lint / fmt`）
* 重要な変更は `git diff` を確認し、節目でコミット
* 秘密情報はコミットしない（`.env` は gitignore、`.env.example` にキーだけ載せる）

---

## まずやること（セットアップ）

1. `docs/development.md` に自分の運用メモを置く
2. `catalog/apps.yaml` を作り、最初のアプリを1件登録
3. 必要に応じて `scripts/newapp.sh` を追加して作業を省力化する

---

## 参考：このリポジトリの位置づけ

* `template-fullstack`：雛形（テンプレ）のためのリポジトリ
* `projects/*`：実アプリ（子）のためのリポジトリ
* `dev-ops`（ここ）：横断の運用（ルール・台帳・スクリプト）
