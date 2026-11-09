# ARCLINE STG

ARCLINEの検証専用サイト。ソースの正本は [bittrove88/bittrove88.github.io](https://github.com/bittrove88/bittrove88.github.io) の ARCLINE/ です。このリポジトリでゲーム本体を別途編集しません。

## 本番とSTGの違いの扱い

保存をセッション内に限定すること、本番の保存領域・SDKから隔離すること、配信先のURLパスが異なることは、STGの意図した仕様です。本番に合わせて解除しません。一方、ゲームの古いバージョンや不足した機能は、この仕様とは別の同期差分です。「STGなので違う」で済ませず、対象コミットと実ファイルを比較します。

開始ステージ指定・検証用初期データ・STG表示・PWA無効化などの付随仕様と、現在の同期差分・変更計画は [本番・STG比較と同期計画](docs/production-staging-parity.md) を参照してください。環境設定の公開と、ゲームのGitHub Pages配信も別々に確認します。

## 運用

- 事前のSTG配信は必要と判断した変更だけに行います。全PR・全変更に義務化せず、変更の影響に応じて自動／ローカル／STG検証を選びます。判断理由と試した条件はソースPRに記録します。
- 開始ステージを指定して試遊します。進行・在庫・設定はこのプレイ中だけメモリに保持し、再読み込みで初期化します。
- 本番のセーブデータを読み込み・書き換え・削除しません。本番SDKの保存機能も使いません。
- 同じオリジンのGitHub Pagesなので、ブラウザ保存領域の物理的隔離ではなく、実装とブラウザ検証で非干渉を確認します。
- 指定した完全なコミットSHAから生成・検証し、このリポジトリのPagesだけを手動更新します。本番への自動反映は行いません。

## 現行STGの更新対象

本番公開版0.29.5の `415e7bc9cb28ee591abd80063ce4eab18da07607` に固定。保存隔離とSTG専用仕様を維持し、ゲームの実ファイルを同期します。配信前検証は完了し、配信・公開確認の結果は [反映記録](docs/production-staging-parity.md#0295への反映記録) に記録します。下記の0.24.1は初回配信時の履歴です。

## 初期設定

1. Settings → Pages → Build and deployment → Source を **GitHub Actions** に設定します。
2. Actions → **ARCLINE STG manual publish** → Run workflow を開きます。
3. [PR #65](https://github.com/bittrove88/bittrove88.github.io/pull/65)の検証済みheadなど、STG実装を含む完全な40桁SHAを source_commit に入力します。
4. アシスタントに更新を依頼する場合は、`staging-source.json` の sourceCommit を変更するコミットで同じ検証・配信処理を開始できます。ソースPRの更新では配信しません。
5. 検証・配信成功後、[STGを開く](https://bittrove88.github.io/arcline-staging/)。画面に表示された版とコミットを確認してください。

公開URLの確認・実機操作はActions成功と区別して記録します。初回配信と公開URLのブラウザ検証は成功しました。

- ソース：`fb8dbc1000321a885ece8ffc29b2c5cd47c8f366`（0.24.1のゲーム本体を利用）
- [検証・配信Actions](https://github.com/bittrove88/arcline-staging/actions/runs/42344852129)
- [公開URLのブラウザ検証](https://github.com/bittrove88/arcline-staging/actions/runs/42345229073)
- 公開URLで1/50/100面起動、50/100面の実戦開始、再読込初期化、音源実再生、保存非干渉を確認。実機iPhone／Androidは未確認です。
- 以後の配信後も「Verify public ARCLINE STG」が公開URLを確認します。試験には新しいブラウザプロファイルを使い、利用者の実データは取得しません。
