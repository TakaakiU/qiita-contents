---
title: Qiita CLIの導入完了後にGitHub Actionsの自動ビルドでエラーが発生
tags:
  - Qiita
  - GitHub
  - GitHubActions
  - QiitaCLI
private: false
updated_at: '2025-08-05T09:30:15+09:00'
id: 1f84f4c58b04ee241720
organization_url_name: null
slide: false
ignorePublish: false
---

[こちらの公式README](https://github.com/increments/qiita-cli)を参照し、Qiita CLIを導入しました。

1. Qiita CLI をインストール・アップデート
1. Qiita CLI のアップデート
1. Qiita CLI のセットアップでinit コマンドを実行
1. Qiita CLI のセットアップでQiita のトークンを発行
1. Qiita CLI のセットアップでQiita CLI のログイン
1. Qiita Preview の起動
1. Qiita CLI で記事を作成

と順調に進み完了しましたが、記事を作成したタイミングでGitHubリポジトリを新規作成し初回のコミットを行った結果、
**GitHub Actionsのワークフローでエラーが発生**しました。

この件について調べ対応した内容を共有します。

## 結論

おそらく本来であれば自動的に登録されるはずのQiitaの認証トークンがGitHub Actionsの**Secrets and variables**の設定に反映されていないことが原因でした。

Qiitaコンテンツのリポジトリに対し、Qiitaのアクセストークンを手動で設定した事により解決しました。

「なぜ、自動でQiitaのアクセストークンが登録されなかったか」については不明ですが、Qiitaのアカウント作成やQiita CLI導入時にブラウザのプライベートモードと通常モードの2つを使って作業していた事が要因なのかもしれません。

もし、紹介する手法で解決できなかった場合、最終手段として1から新規リポジトリを作成し、やり直す方法の方が早いかも。

## 事象

下記がGitHub Actions ワークフローで発生したエラーの抜粋。

```:エラー（抜粋）
Error: ENOENT: no such file or directory, open '/home/runner/.config/qiita-cli/credentials.json'
```

<details><summary>GitHub Actions ワークフローのエラー全文</summary>

```:エラー（全文）
Run increments/qiita-cli/actions/publish@v1
Run actions/setup-node@v4
Attempting to download 20.16.0...
Acquiring 20.16.0 - x64 from https://github.com/actions/node-versions/releases/download/20.16.0-10080284600/node-20.16.0-linux-x64.tar.gz
Extracting ...
/usr/bin/tar xz --strip 1 --warning=no-unknown-keyword --overwrite -C /home/runner/work/_temp/218da015-6487-483f-b8bb-5d3e61a908ea -f /home/runner/work/_temp/12c88de6-7aff-4007-9450-567aa37a6e24
Adding to the cache ...
Environment details
Run npm install -g @qiita/qiita-cli@v1.6.2

added 121 packages in 4s

39 packages are looking for funding
  run `npm fund` for details
Run qiita publish --all --root .
Error: ENOENT: no such file or directory, open '/home/runner/.config/qiita-cli/credentials.json'
    at async open (node:internal/fs/promises:639:25)
    at async Object.readFile (node:internal/fs/promises:1249:14)
    at async Credential.load (/opt/hostedtoolcache/node/20.16.0/x64/lib/node_modules/@qiita/qiita-cli/dist/lib/config.js:174:26)
    at async Credential.getCredential (/opt/hostedtoolcache/node/20.16.0/x64/lib/node_modules/@qiita/qiita-cli/dist/lib/config.js:186:32)
    at async accessToken (/opt/hostedtoolcache/node/20.16.0/x64/lib/node_modules/@qiita/qiita-cli/dist/lib/get-qiita-api-instance.js:18:59)
    at async getQiitaApiInstance (/opt/hostedtoolcache/node/20.16.0/x64/lib/node_modules/@qiita/qiita-cli/dist/lib/get-qiita-api-instance.js:11:20)
    at async Object.publish (/opt/hostedtoolcache/node/20.16.0/x64/lib/node_modules/@qiita/qiita-cli/dist/commands/publish.js:21:22)
    at async exec (/opt/hostedtoolcache/node/20.16.0/x64/lib/node_modules/@qiita/qiita-cli/dist/commands/index.js:41:9) {
  errno: -2,
  code: 'ENOENT',
  syscall: 'open',
  path: '/home/runner/.config/qiita-cli/credentials.json'
}
エラーが発生しました (ENOENT: no such file or directory, open '/home/runner/.config/qiita-cli/credentials.json')
  バグの可能性がある場合は、Qiita Discussionsよりご報告いただけると幸いです
  https://github.com/increments/qiita-discussions
Error: Process completed with exit code 1.
```

</details>

## 対応方法：Qiitaコンテンツのリポジトリで設定でキーを設定

### 手順1. クライアント端末のアクセスキーをメモ

クライアントがWindows OSであるため、`C:\Users\<ユーザー名>\.config\qiita-cli\credentials.json`を参照し、
`accessToken`にあるキーをメモしておく。

```json:credentials.json
{
  "default": "qiita",
  "credentials": [
    {
      "name": "qiita",
      "accessToken": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
    }
  ]
}
```

なお、Windows以外（macOS / Linux）の場合は、下記に対象ファイルがあるとのこと（未検証）。
`~/.config/qiita-cli/credentials.json`

<details><summary>補足情報：QiitaコンテンツのGitHubリポジトリ上の設定を確認すると……</summary>

Qiitaコンテンツのリポジトリ上にある`.github/workflows/publish.yml`は下記のとおり。

```yml:publish.yml
# Please set 'QIITA_TOKEN' secret to your repository
name: Publish articles

on:
  push:
    branches:
      - main
      - master
  workflow_dispatch:

permissions:
  contents: write

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: false

jobs:
  publish_articles:
    runs-on: ubuntu-latest
    timeout-minutes: 5
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: increments/qiita-cli/actions/publish@v1
        with:
          qiita-token: ${{ secrets.QIITA_TOKEN }}
          root: "."

```

上記の設定のとおり、`qiita-token: ${{ secrets.QIITA_TOKEN }}`となっています。
そのため、GitHubリポジトリ上に同じ変数名の`QIITA_TOKEN`をキーにアクセスキーを登録してあげる必要があります。

</detaiils>

### 手順2. GitHubリポジトリにQiitaのアクセスキーを登録

1. エラーが発生したGitHubリポジトリのページにアクセス
1. 「**Settings**」タブをクリック
1. 左のメニューから「**Secrets and variables**」>「**Actions**」を選択
1. 「**New repository secret**」ボタンをクリック
    ![設定前のSecrets and variables画面｜GitHub](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/4156147/afce70dd-474d-45f1-8f4d-611b18bff213.png)
1. **Name** に `QIITA_TOKEN` を入力し、**Secret** のテキストボックスに、手順1でコピーしたアクセストークン（`xxxxxxxx...`の部分）を貼り付け
    ![NameとSecretを入力後の画面｜Actions secrets](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/4156147/30a407a4-1242-489e-aabf-e462e198f8b2.jpeg)
1. 「**Add secret**」ボタンをクリックして保存

この手順でQiitaのアクセスキーをGitHubのリポジトリ上に設定できました。
![設定後のSecrets and variables画面｜GitHub](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/4156147/7bc74df3-55c7-418a-99d3-d233ebc814ea.jpeg)

### 手順3. 次回コミット時に正常終了することを確認

記事の更新や設定ファイルの更新に合わせて、エラーが解消するか確認します。
下記のとおり、エラーがあったGitHub Actionsで正常終了したことを確認すれば解決です。

![GitHubリポジトリにQiitaアクセストークンを登録し次回コミットで正常終了を確認](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/4156147/e27ef39e-6072-40ed-bf3e-c0ca96de11e6.jpeg)

## まとめ

- Qiita CLI導入後の初回のコミットでエラー「`Error: ENOENT: no such file or directory, open '/home/runner/.config/qiita-cli/credentials.json'`」が発生
- エラーの原因はQiita CLIのアクセストークンがコミット先のリポジトリに設定されていなかったため
- なぜ公式の手順でアクセストークンが自動で設定されなかったのかは不明
    おそらく設定作業におけるブラウザ操作でプライベートモードや通常モードの2つを使って対応してしまったことが原因かも。
