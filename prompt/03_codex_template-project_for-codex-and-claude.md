あなたはリポジトリ作成・実装・実行確認まで行うAI開発者です。
WSL上で動く「template-fullstack」リポジトリを新規作成してください。
目的は “複数のWeb/APIスタックの選択肢を持つ、量産用テンプレ” を用意することです。

# ゴール（必須）
- 1つのテンプレリポジトリとして、以下の選択肢を提供する
  - Web: Next.js または React Router
  - API: Node/TypeScript または Python/FastAPI
- 初期状態（デフォルト）は「Next.js + Node/TypeScript API」で `make dev` すれば動く
- どの組み合わせでも、手順が統一される（makeコマンド、ポート、envキーなど）
- Dev Container（.devcontainer）を含み、WSL + VS Code Remote-WSL で “Reopen in Container” して動く
- AGENTS.md / CLAUDE.md / CONTRIBUTING.md を含め、AI運用前提のルールを明文化する
  - 最低限含めるセクション:
    - 利用可能コマンド一覧（make xxx）
    - コミットメッセージ規約（Conventional Commits）
    - テスト必須ルール（テスト未通過のコミット禁止）
    - 禁止操作（.env の直接コミット、lock ファイルの手動編集、DB マイグレーションの自動実行）
    - レビュー基準（lint 通過、型エラーなし）

# 前提条件・実行環境（必須）
- Node.js 最新 LTS, Python 最新
- パッケージマネージャ: npm（pnpm/yarn は使わない）
- Dev Container ベースイメージ: mcr.microsoft.com/devcontainers/typescript-node:20
  - Python は Dev Container 内で別途インストール（Feature or apt）
- Docker ソケットはホスト共有（Docker outside of Docker）
- Postgres / Redis のみ docker compose で起動。アプリ自体はコンテナ化しない
- モノレポツール（turborepo 等）は使わない

# 重要な運用要件（必須）
- コマンドの統一:
  - make setup  : 依存インストール + DB/Redis コンテナ起動
  - make dev    : DB/Redis の docker compose up -d → API起動 → Web起動（全て一括）
  - make dev-web / make dev-api : 個別起動も可能にする
  - make test / make lint / make fmt
- ポートの統一（原則変更しない）:
  - Web: 3000
  - API: 8000
  - Postgres: 5432
  - Redis: 6379
- envキー統一:
  - DATABASE_URL
  - REDIS_URL
  - API の公開URLは .env に API_BASE_URL として定義
    - Next.js テンプレでは next.config 等で NEXT_PUBLIC_API_BASE_URL へマッピング
    - React Router(Vite) テンプレでは vite.config 等で VITE_API_BASE_URL へマッピング
    - 開発者が触る .env 上は API_BASE_URL に統一する
- 依存は最小限。テンプレは軽く保つ（過剰なライブラリを入れない）
- 生成物は “そのまま GitHub Template として使える” 品質（README/セットアップ/動作確認手順が明確）

# 実装方針（必須）
「1つのリポジトリに選択肢を内包」する方式にする。
具体的には次のどちらかで実現してよい（あなたがより堅牢だと思う方を採用）:

A案: /templates 配下に4パターンの雛形を置き、scripts/scaffold.sh で選択して apps/ 以下へ展開
  - templates/web-nextjs
  - templates/web-react-router
  - templates/api-node-ts
  - templates/api-fastapi
  - scaffold.sh が対話または引数で組み合わせを選び、apps/web と apps/api を生成（既存があれば安全に上書きしない）
  - `make setup` で依存、`make dev` で起動まで通る

B案: apps/web と apps/api に “両方” 置き、切り替えは makeターゲットで選べる（例: make dev WEB=next API=node）
  - ただし混在で複雑になりやすいので、整合性と保守性に注意

いずれの案でも「デフォルト状態（選択不要）で make dev が動く」こと。

【デフォルト状態の定義】
- リポジトリの初期状態で apps/web（Next.js）と apps/api（Node/TS）を配置済みにする
- scaffold.sh は「別の組み合わせに切り替えたいとき」に使うツール
- つまり clone 直後に make setup → make dev で動くこと（scaffold 不要）

# 技術選定の具体（必須）
- Web:
  - Next.js: App Router（最小のページでAPI疎通表示）
  - React Router: モダン構成（Vite + React Router でOK）。最小の画面でAPI疎通表示
- API:
  - Node/TypeScript: Fastify を採用（最小の /health と /api/hello を実装）
  - FastAPI: /health と /api/hello を実装
- DB/Cache:
  - Postgres + Redis を docker compose で起動（devcontainer compose に含める）
  - アプリからは接続文字列を env で受ける（将来拡張のため、現時点で必須実装はしなくてOK）
- Lint/Format/Test:
  - Web(Next/React Router): eslint + prettier（最低限 lint/fmt が動く）
  - Node API: eslint + prettier + tsc（最低限 lint/test もしくは lint/typecheck が動く）
  - FastAPI: ruff + pytest（最低限 lint/test が動く）
  - ただし “最小” を保ち、過剰な設定は避ける

# リポジトリ構造（例、変更可だが意図は維持）
- .devcontainer/
  - devcontainer.json
  - docker-compose.yml
  - Dockerfile
- apps/
  - web/  (scaffold後に配置される)
  - api/  (scaffold後に配置される)
- scripts/
  - scaffold.sh   (A案の場合は必須)
  - dev.sh / lint.sh / fmt.sh / test.sh（必要なら）
- Makefile
- README.md（テンプレの使い方、組み合わせ選択、起動、確認方法）
- .env.example
- .editorconfig / .gitignore
- AGENTS.md（Codex向け：コマンド、ルール、レビュー基準、危険操作禁止）
- CLAUDE.md（Claude Code向け：同様の作業規約）
- CONTRIBUTING.md


# 動作確認（必須）
- 確認は Dev Container 内（WSL + VS Code "Reopen in Container"）で行う
- Codex 等のサンドボックスでは Docker/ポート転送が動かない可能性があるため、
  実行確認が不可能な場合は「コマンドとしてエラーなく完走すること」を確認し、
  ネットワーク疎通は手動確認手順として README に明記すればよい
- Dev Container 内で以下を実行して問題ないことを確認し、READMEに明記
  - make setup
  - make dev
- デフォルト構成で以下が確認できること
  - http://localhost:3000 にアクセスすると、APIの /api/hello の結果が画面に表示される
  - http://localhost:8000/health が ok を返す
- make test の期待:
  - Node API: 最低1つのテスト（/health のレスポンス確認）が pass する
  - FastAPI: 最低1つの pytest（/health のレスポンス確認）が pass する
  - Web: lint が通ること（ユニットテストは任意）

# 仕上げ要件（必須）
- 生成/切替手順が明確で、初学者でも迷わないREADME
- 変更は小さく、ファイルの意図が分かるコメント/命名
- 秘密情報はコミットしない（.envはgitignore、.env.exampleでキーを示す）
- GitHub Actions の最小 CI を追加する（必須）
  - lint と typecheck のみ（テスト実行は DB 依存があるため任意）
  - 対象は apps/web と apps/api（デフォルト構成の Next.js + Node/TS）


# 進め方（必須）
- まず plan を簡潔に提示してから実装開始
- 実装後に `tree` 相当で主要ファイル一覧を示し、どこを編集すれば拡張できるか説明する
- 最後に make setup / make dev の結果（成功した旨）を報告する

