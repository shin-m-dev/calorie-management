# AIエージェント引き継ぎ資料 (HANDOFF)

このドキュメントは、このプロジェクトで作業する将来のAIエージェントまたは人間の開発者のための引き継ぎノートです。重要なタスクを完了した後、または作業を中断するときに、このファイルを更新してください。

## 1. Current Status (現在の状況)
- **Status**: Partially Working (一部動作) / Review Needed (要レビュー)
- **Overview**: 本プロジェクトは、カロリー管理のためのシングルページHTMLアプリケーションです。手動入力、Gemini APIを使用したカメラによる食品解析、履歴管理、薬・サプリの記録機能が含まれています。主要な機能は実装されていますが、コードレビューによりいくつかの改善点が特定されています。

## 2. Recent Changes (直近の変更点)
- `HANDOFF_TEMPLATE.md`: 引き継ぎ資料のテンプレートを作成しました。
- `HANDOFF.md`: 初回のコードレビュー結果を記録するために、このドキュメントを作成しました。

## 3. Key Decisions & Rationale (重要な決定とその理由)
- **Single File Architecture (単一ファイル構成)**: 現在、すべてのコード（HTML, CSS, JS）が `index.html` に含まれています。これによりデプロイ（ファイルを開くだけ）は簡単になりますが、プロジェクトの成長に伴い保守性が低下します。
- **Client-Side Only (クライアントサイドのみ)**: アプリはデータ永続化に `localStorage` を使用し、ブラウザから直接APIを呼び出します。これによりバックエンドサーバーは不要になりますが、APIキーの取り扱いに注意が必要です。

## 4. Code Review Notes (コードレビュー指摘事項)

### General Architecture (全体構成)
- **File**: `index.html`
- **Severity**: Low (低)
- **Issue**: モノリシックなファイル構造になっています。
- **Suggestion**: 整理とキャッシュ管理のために、CSSを `style.css` に、JavaScriptを `app.js` に分割することを推奨します。

### Security (セキュリティ)
- **File**: `index.html` (Script section)
- **Severity**: Medium (中)
- **Issue**: APIキーの露出リスクがあります。Gemini APIキーが `localStorage` に保存され、クライアントサイドの `fetch` 呼び出しで直接使用されています。
- **Suggestion**: 個人のローカルツールとしては許容範囲ですが、公開デプロイする場合はリスクがあります。共有する場合は、シンプルなバックエンドプロキシを実装するか、Firebase Functionsなどを使用してキーを隠蔽することを推奨します。また、ユーザーにローカルストレージデータを共有しないよう警告すべきです。

### API Usage / Bugs (API利用 / バグ)
- **File**: `index.html` -> `analyzeWithGemini` 関数
- **Severity**: High (高)
- **Issue**: モデル名が `gemini-2.5-flash` と指定されています。
  ```javascript
  const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${apiKey}`;
  ```
  現在の知識では、安定版のモデルバージョンは `gemini-1.5-flash` または `gemini-1.5-pro` です。`gemini-2.5-flash` はタイプミスか存在しないモデルの可能性が高く、API呼び出しが失敗する恐れがあります。
- **Suggestion**: モデル名を確認してください。`2.5` が有効であると確認できない限り、`gemini-1.5-flash` に変更することを推奨します。

### DOM Manipulation (DOM操作)
- **File**: `index.html` -> `renderCategories`, `addEntry`
- **Severity**: Low (低)
- **Issue**: `innerHTML` が多用されています。
- **Suggestion**: 現在入力は制御されていますが、データソースが信頼できなくなった場合、`innerHTML` の使用はXSSの脆弱性につながる可能性があります。動的なコンテンツ挿入には `document.createElement()` と `textContent` の使用を検討してください。

### User Experience (ユーザー体験)
- **File**: `index.html`
- **Severity**: Low (低)
- **Issue**: エラー処理がカスタムトーストに依存しています。これは良いことですが、APIからの具体的なエラーメッセージ（例：割り当て超過）がエンドユーザーにとって専門的すぎる可能性があります。
- **Suggestion**: APIのエラーコードを、よりユーザーフレンドリーなメッセージ（例：「429」ではなく「1日の利用制限に達しました」など）にマッピングすることを推奨します。

## 5. Known Issues / TODOs (既知の問題 / 次やること)
- [ ] **Bug**: `analyzeWithGemini` 内のGeminiモデル名を修正する（おそらく `2.5` を `1.5` に変更）。
- [ ] **Refactor**: CSSとJSを別ファイルに抽出する。
- [ ] **Feature**: `localStorage` 以外の場所にデータをバックアップできるよう、データのエクスポート/インポート機能（JSON）を追加する。

## 6. Environment & Setup (環境構築・実行方法)
- **API Keys**: カメラ解析機能にはGoogle AI StudioのAPIキーが必要です。
- **Commands**: モダンなWebブラウザで `index.html` を開くだけです。ビルド手順は不要です。
