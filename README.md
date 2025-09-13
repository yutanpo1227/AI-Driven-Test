# Codexを用いた自動開発

## Issueの発行をトリガーにIssueを解決させる

1. Issueの発行でGithubActionsをトリガー
2. codexにissueの内容を渡す
3. codexが修正
4. codexがPRを作成
5. PRの作成をトリガーにPRのdiffの内容を要約させる
6. 人間によるレビューとマージ

**人間は新たなIssueを投げるだけが理想**

## セットアップ
### Personal Access Tokenの作成
GithubのSettings > Developer settings > Personal access tokens からトークンを作成

### PAT_TOKENの設定
1. GithubのSettings > Secrets and variables > Actions から `PAT_TOKEN` を設定
2. トークンを貼り付け

### OPENAI_API_KEYの設定
GithubのSettings > Secrets and variables > Actions から `OPENAI_API_KEY` を設定

## 動作
### Codexによる実装(`.github/workflows/issue-to-pr.yaml`) 
1.  **トリガー**: Issue に `codex` ラベルが付与された際に起動する。
2.  **環境セットアップ**:
    - リポジトリのコードをチェックアウトする。
    - Git のユーザー情報を設定する（ボットとしてコミットするため）。
    - Node.js 環境をセットアップする。
    - OpenAI Codex CLI をインストールする。
3.  **Codex による修正とパッチ生成**:
    - Issue 番号に基づいた新しいブランチを作成する (`codex/issue-${{ github.event.issue.number }}`)。
    - 既存の `node_modules` と `package-lock.json` があれば削除し、クリーンな状態にする。
    - Issue のタイトルと本文を連結し、Codex へのプロンプトを生成する。この際、「最後に`npm ci && npm run build`を実行してエラーが出ないことを確認してください。 .git のような'.'から始まるファイル、フォルダの中身は絶対に変更しないでください。」といった指示も追加している。
    - `codex -a full-auto --quiet "$PROMPT"` コマンドを実行し、Codex にコードの自動修正とコミットを行わせる。
    - もし `codex` コマンドがコミットしなかった場合でも変更があれば、変更点をステージングし、コミットする。
    - 作成したブランチをリモートリポジトリにプッシュする。
4.  **プルリクエストの作成**:
    - `gh pr create` コマンドを使用し、修正内容を含むプルリクエストを `main` ブランチに向けて作成する。PR のタイトルと本文も自動生成される。

このワークフローにより、Issue の内容に基づいて Codex が自動的にコードを修正し、人間がレビュー可能なプルリクエストとして提出される。

### CodexによるPRの要約(`.github/workflows/pr-summary.yaml`)
1.  **トリガー**: プルリクエストがオープンされた際に起動する (`pull_request_target` イベントの `opened` タイプ)。
2.  **環境セットアップ**:
    - PR の HEAD ブランチ（変更が含まれるブランチ）のコードをチェックアウトする。
    - Node.js 環境をセットアップする。
3.  **Codex による要約とコメント**:
    - OpenAI Codex CLI をインストールする。
    - `gh pr diff ${PR_NUMBER} > pr-diff.txt` コマンドで PR の差分を取得し、`pr-diff.txt` ファイルに保存する。
    - `codex -m o4-mini -a auto-edit --quiet "pr-diff.txt から更新差分を日本語で要約して。変更があったファイルの内容を確認して。プルリクエストとして要約内容をcodex-summary.md に保存して。"` コマンドを実行し、Codex に差分ファイルの日本語での要約を依頼し、結果を `codex-summary.md` に保存する。ここでは `o4-mini` モデルを使用している。
    - `gh pr comment --body-file codex-summary.md "${PR_URL}"` コマンドで、生成された要約 (`codex-summary.md`) を該当の PR にコメントとして投稿する。

これにより、レビュアーは PR を開くとすぐに、Codex によって生成された変更点の要約を読むことができ、レビューの効率化が期待できる。