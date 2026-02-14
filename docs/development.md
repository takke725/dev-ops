

# 開発運用メモ（WSL + template-fullstack + Codex / Claude Code）

## 目的

* WSL上で開発環境を統一し、Webアプリを量産できる運用を作る
* **テンプレ（親）**と**実プロジェクト（子）**を分離して管理する
* AI（Codex / Claude Code）を活用しつつ、Gitで差分管理・レビューしながら安全に進める

---

## 全体像（重要）

### リポジトリは2種類

1. **テンプレ（親）**

* `~/src/templates/template-fullstack/`
* 役割：新規プロジェクト作成の雛形
* 更新頻度：たまに（テンプレ改善時）

2. **実プロジェクト（子）**

* `~/src/projects/<project-name>/`
* 役割：実際に開発・デプロイするアプリ
* 更新頻度：毎日

> 子プロジェクトは増えていくが、テンプレ親リポジトリに子が「含まれる」わけではない。
> **それぞれが別リポジトリとして独立**する。

---

## WSL上のフォルダ構成

```
/home/<user>/
  src/
    templates/
      template-fullstack/      # テンプレ（親）
    projects/
      <project-a>/             # 実プロジェクト（子）
      <project-b>/
    playground/
      scratch-*/               # 捨ててOKな実験
  bin/                         # 便利スクリプト（任意）
```

---

## 前提（すでに完了していること）

* WSLに Codex と Claude Code はインストール済み
* `template-fullstack` のローカルリポジトリが `~/src/templates/template-fullstack` に存在
* `template-fullstack` のリモートリポジトリ（GitHub）も作成済み

---

## 今後の基本フロー（新しいアプリを作るたびに行う）

### 1) GitHub上で「テンプレから新しいリモートリポジトリ」を作成

* `template-fullstack` → **Use this template** → 新しいリポジトリ（例：`my-saas`）を作る
* ここでできるのは「子プロジェクト用のリモート」

### 2) WSLの `projects/` 配下に clone

```bash
cd ~/src/projects
git clone <新しく作ったリポジトリのURL>
cd <project-name>
```

> clone URLは GitHubの **Code** ボタンからコピペする（手入力しない）

### 3) Dev Container or ローカルで起動

テンプレがDev Container前提の場合：

* VS Codeで開く → “Reopen in Container”
* その中で：

```bash
make setup
make dev
```

（テンプレのREADMEに従う）

### 4) Web/API構成の生成（テンプレが選択式の場合）

テンプレが `scripts/scaffold.sh` 等を持っているなら、プロジェクト内で必要に応じて選択して生成する。
例（実際のコマンド名はテンプレ仕様に合わせる）：

```bash
./scripts/scaffold.sh --web next --api node
# or
./scripts/scaffold.sh --web react-router --api fastapi
```

### 5) 日々の開発（AI活用）

* 小さな修正・調査・ログ解析：Claude Code（対話的に進める）
* まとまった実装・テンプレ修正・タスク駆動：Codex（計画→実装→検証）

※ どちらを使う場合も、最後は必ず `git diff` と `make test / make lint` で確認する。

---

## AIに依頼するときの基本ルール（事故防止）

### 推奨の進め方

1. **Plan（やること一覧）を先に出させる**
2. 変更は小さく区切る
3. 重要な節目で `git diff` を確認
4. 動作確認（`make dev` / `make test`）
5. コミット

### Codexの承認設定

* 最初は「毎回承認（ask me to approve）」推奨
* 慣れてきてから「フォルダ内は無承認」に切り替える

---

## Git運用ルール（最小）

### ブランチ（例）

* `main`: 安定
* `feat/<topic>`: 機能追加
* `fix/<topic>`: バグ修正

### コミット

* 生成直後・大きい変更前後にチェックポイントを作る
* PR単位は小さく保つ（後で直せる）

---

## よくある詰まりポイント

### 1) `fatal: empty ident name ...`（コミットできない）

Gitの `user.name` / `user.email` が未設定か壊れている。

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

このリポジトリだけ上書きしたいなら：

```bash
git config --local user.name "Your Name"
git config --local user.email "you@example.com"
```

### 2) `git clone` でユーザー名/パスワードを聞かれる（HTTPS）

* Password欄に入れるのは **アカウントのパスワードではなくトークン（PAT）**が基本
* もしくはSSH方式（鍵登録）に切り替える

### 3) `Permission denied (publickey)`（SSH）

* WSLの公開鍵をGitHubに登録していない/ssh-agentが掴んでいない可能性

---

## テンプレ（親）を更新したいとき

* 更新は `~/src/templates/template-fullstack/` で行う
* 変更後はテンプレのリモートへ push
* **既存の子プロジェクトにテンプレ更新を自動反映しない**（基本方針）

  * 反映したくなったら、必要な差分だけ手動で取り込む（衝突管理が発生するため）

> 結果：テンプレ改善は「次に作る新規プロジェクト」から効く、という運用にする。

---

## “いま何個アプリがあるか”を把握する方法（メモ）

* 基本は GitHub上で一覧化（命名規則やタグで管理）
* ローカルは `~/src/projects/` を見れば「手元にあるもの」が分かる

---

## 1コマンドで増やしたい場合（将来の改善アイデア）

`~/bin/newapp` を作って、

* GitHubで作ったリポジトリをclone
* 必要なら scaffold を実行
* Dev Container起動まで案内
  …を自動化するとさらに楽になる。

---

### 自分用のチェックリスト（新規プロジェクト作成時）

* [ ] GitHubでテンプレから新リポジトリ作成
* [ ] `~/src/projects` にclone
* [ ] `make setup` → `make dev`
* [ ] Web/APIの選択・生成（必要なら）
* [ ] 最初の動作確認が通ったら初回コミット

