# ARCLINE STG

ARCLINEの検証専用サイト。ソースの正本は [bittrove88/bittrove88.github.io](https://github.com/bittrove88/bittrove88.github.io) の ARCLINE/ です。このリポジトリでゲーム本体を別途編集しません。

## 運用

- 開始ステージを指定して試遊します。進行・在庫・設定はこのプレイ中だけメモリに保持し、再読み込みで初期化します。
- 本番のセーブデータを読み込み・書き換え・削除しません。本番SDKの保存機能も使いません。
- 同じオリジンのGitHub Pagesなので、ブラウザ保存領域の物理的隔離ではなく、実装とブラウザ検証で非干渉を確認します。
- 指定した完全なコミットSHAから生成・検証し、このリポジトリのPagesだけを手動更新します。本番への自動反映は行いません。

## 初期設定

1. Settings → Pages → Build and deployment → Source を **GitHub Actions** に設定します。
2. Actions → **ARCLINE STG manual publish** → Run workflow を開きます。
3. [PR #65](https://github.com/bittrove88/bittrove88.github.io/pull/65)の検証済みheadなど、STG実装を含む完全な40桁SHAを source_commit に入力します。
4. 検証・配信成功後、[STGを開く](https://bittrove88.github.io/arcline-staging/)。画面に表示された版とコミットを確認してください。

公開URLの確認・実機操作はActions成功と区別して記録します。初期の配信結果は準備中です。
