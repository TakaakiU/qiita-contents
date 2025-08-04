---
title: Qiita CLIを構築中にGitHub Actionsにてエラーが発生
tags:
  - 'qiita'
  - 'qiitacli'
  - 'github'
  - 'githubactions'

private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

## 発生したエラー

```
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

## Gemini

```
ご提示いただいたGitHub Actionsのエラーログ、拝見いたしました。
これはQiita CLIとGitHub Actionsを連携させる際に非常によく発生する典型的なエラーです。

### エラーの直接的な原因

エラーメッセージの核心部分はここにあります。

```
Error: ENOENT: no such file or directory, open '/home/runner/.config/qiita-cli/credentials.json'
```

これは、GitHub Actionsを実行している仮想環境（runner）の `/home/runner/.config/qiita-cli/` というディレクトリ内に、Qiitaの認証情報が書かれた `credentials.json` ファイルが見つからない、というエラーです。

**つまり、「GitHub ActionsがQiitaに記事を投稿しようとしたが、誰として投稿すればいいのか（＝どのアクセストークンを使えばいいのか）分からなかった」というのが原因です。**

### なぜこのエラーが起きるのか？

1.  **ローカル環境とGitHub Actions環境は別物**
    ユーザー様がご自身のPC（ローカル環境）で `qiita configure` コマンドを実行した際に、`credentials.json` ファイルがPC内に作成されました。Qiita CLIはローカルではそのファイルを読み込んで認証しています。

2.  **GitHub Actionsは毎回クリーンな環境で実行される**
    GitHub Actionsのワークフローが実行されるたびに、GitHubはまっさらな仮想環境（runner）を準備します。この環境には、当然ながらユーザー様のPC内にある `credentials.json` ファイルは存在しません。

3.  **認証情報がないためエラーになる**
    そのクリーンな環境で `qiita publish` コマンドが実行されると、Qiita CLIは認証情報を探して `/home/runner/.config/qiita-cli/credentials.json` を読み込もうとしますが、ファイルが存在しないため「ファイルが見つからない（ENOENT）」というエラーで処理が失敗します。

### 解決策：GitHub Secretsを使ってアクセストークンを渡す

この問題を解決するには、GitHub Actionsのワークフローに**安全な方法でQiitaのアクセストークンを渡す**必要があります。そのための仕組みが **GitHub Secrets** です。

以下の手順で修正してください。

#### 手順1：Qiitaのアクセストークンを取得する

まず、ローカルのPCに保存されているアクセストークンを確認します。
お使いのOSに応じて、以下の場所にある `credentials.json` ファイルをテキストエディタで開いてください。

*   **Windows:** `C:\Users\<ユーザー名>\.config\qiita-cli\credentials.json`
*   **macOS / Linux:** `~/.config/qiita-cli/credentials.json`

ファイルの中身は以下のようになっています。この `"accessToken": "..."` の部分の英数字の羅列がアクセストークンです。これをコピーしておきます。

```json
{
  "accessToken": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
```

#### 手順2：GitHubリポジトリにSecretsを登録する

1.  エラーが発生したGitHubリポジトリのページにアクセスします。
2.  「**Settings**」タブをクリックします。
3.  左のメニューから「**Secrets and variables**」>「**Actions**」を選択します。
4.  「**New repository secret**」ボタンをクリックします。
5.  **Name** に `QIITA_ACCESS_TOKEN` と入力します。（この名前は後で使います）
6.  **Secret** のテキストボックスに、手順1でコピーしたアクセストークン（`xxxxxxxx...`の部分）を貼り付けます。
7.  「**Add secret**」ボタンをクリックして保存します。

#### 手順3：GitHub Actionsのワークフローファイル（.yml）を修正する

リポジトリ内にある `.github/workflows/` ディレクトリの中の、該当するYAMLファイル（例: `qiita-sync.yml` など）を編集します。

`increments/qiita-cli/actions/publish@v1` を使っている箇所を以下のように修正してください。

**修正前（おそらく現在のコード）：**
```yaml
      - name: Qiita Publish
        uses: increments/qiita-cli/actions/publish@v1
        # with: 以下に何も設定されていないか、qiita-tokenがない状態
```

**修正後（`with` を追加）：**
```yaml
      - name: Qiita Publish
        uses: increments/qiita-cli/actions/publish@v1
        with:
          qiita-token: ${{ secrets.QIITA_ACCESS_TOKEN }}
```

`with:` ブロックを追加し、`qiita-token` というパラメータに、先ほどGitHub Secretsに登録した `${{ secrets.QIITA_ACCESS_TOKEN }}` を渡すように指定します。

`${{ ... }}` という構文は、GitHub ActionsがSecretsの値を安全に読み込むためのものです。

### まとめ

この修正を行うことで、ワークフローが実行される際に `increments/qiita-cli/actions/publish@v1` アクションがGitHub Secretsから安全にアクセストークンを受け取り、Qiita APIの認証を成功させることができます。これにより、`credentials.json` ファイルが見つからないというエラーは解消されるはずです。

修正したYAMLファイルをコミット＆プッシュして、再度GitHub Actionsが正常に動作するか確認してみてください。
```

## Qiitaのリポジトリ設定でキーを設定

### GitHub Actions

```
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

![無題.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/4156147/afce70dd-474d-45f1-8f4d-611b18bff213.png)

![スクリーンショット_4-8-2025_114841_github.com.jpeg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/4156147/30a407a4-1242-489e-aabf-e462e198f8b2.jpeg)

![スクリーンショット_4-8-2025_114938_github.com.jpeg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/4156147/7bc74df3-55c7-418a-99d3-d233ebc814ea.jpeg)

```markdown
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

InPrivateモードでアクセストークンの認証をしていたため、GitHub上に登録されなかったのかもしれない。

手動でGitHubのActions設定にアクセストークンを設定することで解決。

作業端末ごとに設定する必要があると思うので、新たな端末でQiitaの記事を編集する際は要注意！
