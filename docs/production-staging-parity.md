# 本番・STGの仕様差と同期計画

確認日：2026-11-09（日本時間）。初回比較結果と変更計画、および下記の反映記録を区別して記載する。

## 比較の原則

「Codexの環境設定」「ゲームの公開バージョン」「STG固有の動作」を区別する。環境設定を公開しても、GitHub Pagesのゲームが本番の最新版に更新されたことにはならない。

本番とSTGの基本的な違いは次の3点であり、同期時にも維持する。

1. 保存：STGの進行・在庫・設定はプレイ中のメモリにだけ保持し、再読み込みで初期化する。
2. 保存領域・SDK：STGは本番の保存データを読み込み・書き換え・削除せず、本番SDKの保存機能も使わない。
3. 配信先：同じGitHub Pagesの別パスに配信する。本番は `/ARCLINE/`（ゲームは `/ARCLINE/game/dist/`）、STGは `/arcline-staging/`（ゲームは `/arcline-staging/game/`）。

同一オリジンなので、別パスというだけではブラウザ保存領域は隔離されない。STGのセッションアダプターと非干渉検証が必要。

## 基本仕様に付随するSTG専用動作

以下は公開中の `staging-session.js`、`staging-loader.js`、ゲームHTMLで確認した。すべてを「本番との差分だから削除」と判断しない。

| 動作 | 目的・扱い |
| --- | --- |
| localStorage/sessionStorageをメモリ実装に置換、IndexedDB・SDKを無効化 | 保存隔離を維持する |
| Cookieアクセス・Service Worker利用を禁止 | 永続化・本番への干渉を防ぐ |
| manifestを除去、PWAインストール案内を抑制 | STGを通常アプリとして永続利用させない。バージョン遅れとは別の検証用仕様 |
| 開始ステージ1〜100を指定、指定なしは1面 | 任意ステージの検証導線 |
| 途中ステージの進行・仲間・在庫などを初期データから構成、乱数seedを固定 | 検証用条件。本番の実プレイ進行・在庫と同一とは主張しない |
| previewパラメーターを拒否 | 現行STGは開始ステージ指定で検証する設計 |
| STGバッジ、コミット・開始面・保存なし表示、専用CSS、noindex/nofollow | 検証環境を識別する |
| 専用ローダーが隔離の成功後にゲームを読み込む | 隔離失敗時は起動させない |
| STG専用のキャッシュ識別子・ビルド情報 | 検証対象を追跡し、本番の識別子と混同しない |

## 更新前に確認した差分（初回比較の記録）

比較元：

- 本番：<https://bittrove88.github.io/ARCLINE/game/dist/> と同ディレクトリの `release-info.js`。
- STG：<https://bittrove88.github.io/arcline-staging/game/> と <https://bittrove88.github.io/arcline-staging/STAGING_BUILD.json>。
- 設定：このリポジトリの `staging-source.json` と `.github/workflows/deploy-staging.yml`。

### リリース・キャッシュ

| 項目 | 本番公開物 | STG公開物 |
| --- | --- | --- |
| version | 0.29.5 | 0.24.1 |
| cacheVersion | 0.29.5 | 0.24.1 |
| campaignVersion | 5.0 | 5.0 |
| saveSchema | 8 | 8 |
| resetVersion | 2 | 2 |
| STGの固定ソース | 対象外 | fb8dbc1000321a885ece8ffc29b2c5cd47c8f366 |

STGの固定ソースはローカル設定と公開ビルド情報で一致。本番公開物を生成した完全なコミットSHAは今回未特定。0.29.5という版名だけで同期対象のSHAを決めない。

### 本番にあり、STGの読込一覧にないファイル

JavaScript：`attack-catalog.js`、`attack-motion.js`、`battle-aim.js`、`battle-catalog.js`、`battle-ui.js`、`component-art.js`、`enemy-actions.js`、`enemy-attacks.js`。

CSS：`battle-ui.css`。

これはSTGの保存隔離に必要な差ではなく、ゲーム更新に伴う同期対象。ファイル名から挙動の詳細までは断定しない。

### 共通JavaScriptの実ファイル差分

両公開URLから同名ファイルを取得し、バイト列を比較した。異なったファイルは以下。

`asset-loader.js`、`audio-manager.js`、`character-hero.js`、`charge-camera.js`、`cinematic-data.js`、`combat.js`、`difficulty-ui.js`、`effects.js`、`ending-roll-data.js`、`engine.js`、`expedition-ui.js`、`final-team.js`、`game.js`、`geometry.js`、`i18n.js`、`lessons.js`、`levels.js`、`master-mode.js`、`master-progress.js`、`media-data.js`、`physics.js`、`presentation.js`、`preview.js`、`release-info.js`、`render.js`、`save.js`、`screens.js`、`sfx-manager.js`、`skills.js`、`story.js`、`viewport.js`、`weapons.js`。

これは内容が異なるという確認であり、各変更の機能・不具合・互換性を評価済みという意味ではない。`save.js`の差もSTGアダプターの保存隔離とは分けて調査する。両版のsaveSchemaが同じでも、初期データとの互換性は検証する。

### 未確認範囲

共通CSSの内容、画像・音源・動画などの全アセット、全HTML構造、HTTPキャッシュヘッダー、GitHub Pages管理画面上の設定、実機動作は今回の比較に含めていない。「これ以外はすべて同じ」とは判断しない。本番用Codex環境の設定もこのチャットでは取得しておらず、STGのCodex環境設定との全面一致は未確認。

## 変更計画

1. **対象を固定する。** 正本リポジトリの本番配信記録と公開ファイルを照合し、本番公開物に対応する完全な40桁SHAを特定する。作業開始時に公開版を再確認し、0.29.5から進んでいれば比較表を更新する。
2. **固定SHAで全体差分を確定する。** 現行STGソースと対象SHAの差分を取り、ランタイム読込一覧・読込順、共通CSS、HTML、画像・音源等も比較する。基本仕様・付随仕様・ゲーム同期差分を別々に記録する。
3. **STG生成処理を追従させる。** 正本の `ARCLINE/environments/` の生成処理が新しいランタイム一覧・CSS・アセットを取り込むか確認する。必要な修正は正本で行い、STGリポジトリへゲーム本体をコピー編集しない。保存隔離・失敗時の起動停止・STG表示を維持する。
4. **配信前検証を行う。** 既存ワークフローに沿って `npm test`、`npm run check:generated`、セッション試験、STG生成、Python試験、ブラウザ試験を実行する。1/50/100面の開始・実戦開始、言語・操作設定・在庫の再読込初期化、音源再生、追加された戦闘UIと敵行動を確認する。本番保存領域・SDKにアクセスせず、同一オリジンの別タブのテスト用保存データを変更しないことを新規ブラウザプロファイルで検証する。
5. **STG更新候補を提示する。** 検証済みSHA・変更ファイル・試験結果・残る制限をまとめる。現行ワークフローでは `staging-source.json` の変更をmainへpushすると配信が始まるため、この計画段階では変更・push・workflow_dispatchを行わない。更新指示を受けた際に検証済みSHAへ変更し、STGのみ配信する。本番は配信しない。
6. **配信後に公開実体を再確認する。** Actions成功と公開URL確認を区別する。`STAGING_BUILD.json` のSHA・版、公開ゲームの読込一覧、保存隔離と主要導線を照合する。許容したSTG専用差以外に説明のない差が残っていないことを確認し、本書を更新する。

完了条件：固定した対象SHAからSTGが生成され、ゲームの差分が説明でき、保存隔離等のSTG仕様を維持した試験が通り、公開後の実ファイルまで確認できたこと。計画作成や環境設定の公開だけでは同期完了としない。


## 0.29.5への反映記録

ユーザーの「反映して」の指示によりSTG更新を実施済み。本番リポジトリ・本番Pages設定への書き込みは行わない。

- 対象SHA：`415e7bc9cb28ee591abd80063ce4eab18da07607`（0.29.5）。[本番Pages配信成功記録](https://github.com/bittrove88/bittrove88.github.io/actions/runs/42537765422)の対象コミットと照合。
- 変更：STGリポジトリの `staging-source.json` の固定SHAと運用文書。正本のゲーム・STG生成処理・保存隔離コードの変更は不要。
- 配信前の実ファイル照合：新STG候補のJavaScript 70本・CSS 12本・Worker 1本、素材127ファイルが、取得時点の本番公開物とバイト列またはSHA-256で一致。以前不足していた8本のJavaScriptと `battle-ui.css` も含む。
- 生成確認：153ファイル一致。最初の確認は浅いcloneに過去の生成基準コミットがなく停止したが、必要な履歴を取得して再実行し成功。依存ファイルの固定値や検証条件は変更していない。
- 保存隔離：セッション試験69項目成功。
- 配信パッケージ・出力先保護：Python試験13件すべて成功（生成済みSTGを指定してスキップなし）。
- ブラウザ：44項目成功。1/50/100面起動、50/100面実戦開始、難易度切替、再読込・別タブでの初期化、Worker整合、AAC再生、本番相当のテスト用保存データへの非干渉、隔離失敗時の起動停止を確認。
- STG生成物：218ファイル、75,895,020 bytes。`persistentSave: false`。開始ステージ・検証用初期データ・manifest除去等のSTG専用仕様を維持。
- HTMLは保存隔離ローダー・素材の相対パス等の意図した加工があるため、全文一致とはしない。STG固有ファイル・メタデータも本番との一致対象外。
- 本番公開ファイルとの一致は、Pages管理画面上の全設定や本番用Codex環境設定の一致を意味しない。実機iPhone/Androidは未確認。

配信・公開URL検証：完了。

- [STG配信Actions](https://github.com/bittrove88/arcline-staging/actions/runs/42541984741)：成功。既存ワークフローのゲーム単体検査・生成物チェック・セッション試験・Python試験・ブラウザ試験・STG配信を完了。
- [公開URLブラウザ検証Actions](https://github.com/bittrove88/arcline-staging/actions/runs/42542507583)：成功。新規ブラウザプロファイルで公開URLの1/50/100面、50/100面実戦開始、再読込初期化、保存非干渉、AAC再生、例外・欠落アセットなしを確認。
- 公開 `STAGING_BUILD.json`：版0.29.5、対象SHA一致、`persistentSave: false`、`sourceDirty: false`。公開された218ファイルすべてをHTTPSで取得し、配信前に試験したパッケージのSHA-256と一致。
- 補助ファイル `attack-review-model.js` と `icon.svg` も本番公開物と一致。本番HTMLは固定ソースと一致し、STG側HTMLの差は生成処理による相対パス・隔離ローダー等として扱う。
- この管理環境での公開URL向けChromium試験は `ERR_CERT_AUTHORITY_INVALID` により停止。証明書検証は無効化していない。上記GitHub Actionsで同等の公開URLブラウザ検証が成功したことを根拠とする。
- 反映後も本番mainのSHAは `415e7bc9cb28ee591abd80063ce4eab18da07607`、公開ゲーム版は0.29.5で変化なし。本番リポジトリへのpushや本番配信操作は行っていない。
- [機械可読の検証記録](verification-0.29.5.json)。実機iPhone/Androidは未確認。

