---
title: VS Codeで選択範囲の文字列を引用表示に変換する5つの方法（解除方法も紹介）
tags:
  - Markdown
  - PowerShell
  - VSCode
private: true
updated_at: '2025-08-07T08:28:34+09:00'
id: b07459f10b9508adec43
organization_url_name: null
slide: false
ignorePublish: false
---

VS Codeの標準的な機能や代表的なMarkdownの拡張機能で実現できないか試行錯誤しましたが、良い方法が見つかりませんでした。
深掘りして調べて、いくつか良さそうな方法があったので共有します。

## VS Codeの標準的な機能で実現しようとしてもNG

```markdown:引用表示にしたいテキスト群
- 項目1
- 項目2

- 項目3
- 項目4
- 項目5
```

上記のようなmdファイル上で「`Ctrl` + `/`」でコメントアウトすると……

```markdown:「Ctrl + /」するとHTMLコードとしてコメントアウトされる
<!-- - 項目1
- 項目2

- 項目3
- 項目4
- 項目5 -->
```

上記のように「HTMLコード」として扱われてしまいました。

## 選択範囲を引用表示にする方法

数行の場合は手動で先頭行に対して「`> `」を追加するのもありですが、対象文字列が多いと面倒です。
調べた結果、5種類の対応が見つかりました。

### 変換方法1. VS Codeのショートカットキーを駆使して追加

ネットで検索した結果、[こちらの記事](https://zenn.dev/d_ske104/articles/vscode-markdown-put-quote-mark)を参照にさせて頂きました。

> 1. 対象のテキストをドラッグなどして範囲選択する。
> 1. Shift + Alt + I [1] で選択範囲の行末にマルチカーソルを置く。
> 1. Home キーでマルチカーソルを行頭に移動させる。
> 1. 引用符 > を入力する。
> 1. おわり。
>
> 引用元：[手順 - VSCodeで複数行にMarkdownの引用符を付ける｜Zenn](https://zenn.dev/d_ske104/articles/vscode-markdown-put-quote-mark#%E6%89%8B%E9%A0%86)

なかなかスマートなやり方です。

### 変換方法2. VS Codeの置換機能（正規表現を有効）を使って追加

正規表現を使った置換で選択範囲の先頭行に対し、「`> `」を追加する手順。

1. 対象の文字列を範囲指定
1. `Ctrl` + `H` で置換機能を表示
1. `Alt` + `L` で「選択範囲を検索」をオンにする
1. `Alt` + `R` で「正規表現を使用する」をオンにする
1. 検索欄に「`^`」を入力
1. 置換欄に「`> `」を入力
1. `Ctrl` + `Alt` + `Enter` で「すべて置換」

対象の文字列内に空行がある場合は、下記のとおり「`> `」のみの行が生まれます。

```markdown:引用表示にしたいテキスト群
> - 項目1
> - 項目2
> 
> - 項目3
> - 項目4
> - 項目5
```

項目2と項目3の間にある「`> `」だけの行は、[markdownlint](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint)などの構文・スタイルチェックツールで警告表示（markdownlintだと[MD009 - 末尾のスペース](https://github.com/DavidAnson/markdownlint/blob/v0.38.0/doc/md009.md)）されてしまいます。

これを解消するため、「`> `」を「`>`」には、追加で置換しましょう。
1 ～ 4 は同じ手順で後続の手順を、

5. 検索欄に「`^> $`」を入力
6. 置換欄に「`>`」を入力

というように置き換えることで対応可能です。

```markdown:追加の置換で「> 」を「>」に変換
> - 項目1
> - 項目2
>
> - 項目3
> - 項目4
> - 項目5
```

テキストを範囲選択しないと結果が判別しにくいですが、置換により末尾のスペースを削除することができました。

### 変換方法3. 専用の拡張機能「Markdown Blockquote Toggler」を使って追加

https://marketplace.visualstudio.com/items?itemName=EliYing.markdown-blockquote-toggler

上記の拡張機能をインストール後、下記の手順で対応可能。

1. 対象の文字列を範囲指定
1. `Ctrl` + `^`で選択範囲の文字列の各行頭に「`> `」が挿入

VS Codeで簡単に導入したい場合は、この方法がオススメです。

### 変換方法4. VS Codeでショートカットキーを自作して追加

#### 設定方法（VS Codeでショートカットキーを自作して追加）

1. `Ctrl` + `Shift` + `P` でコマンドパレットを開き、「`>keyboard shortcuts`」と入力。

    ```plaintext:コピー用
    keyboard shortcuts
    ```

1. 下記の項目を選択

    > 基本設定:キーボードショートカットを開く(JSON)
    > Preferences: Keyboard Shortcuts (JSON)

1. `keybindings.json`が自動で開かれる
1. 下記の内容を追加
    定義するショートカットキーは任意ですが、下記の内容を追加することで、
    選択範囲の文字列を引用表示に変換できます。

    ```json:keybindings.json
    [
        {
            // 既存の定義
        },
        {
            "key": "ctrl+alt+9", // 引用を「挿入」するキー
            "command": "editor.action.insertSnippet",
            "when": "editorTextFocus && editorHasSelection",
            "args": {
                // 選択範囲の各行頭に「> 」を挿入する
                "snippet": "${TM_SELECTED_TEXT/^(.*)(\\r?\\n|$)/> $1$2/gm}"
            }
        }
    ]
    ```

<details><summary>補足情報：自作したショートカットキーの追加と解除の定義</summary>

追加と解除の設定方法は同じなので、どちらか対応する際に合わせて設定することをオススメします。

```json:keybindings.json
[
    {
        // 既存の定義
    },
    {
        "key": "ctrl+alt+9", // 引用を「挿入」するキー
        "command": "editor.action.insertSnippet",
        "when": "editorTextFocus && editorHasSelection",
        "args": {
            // 選択範囲の各行頭に「> 」を挿入する
            "snippet": "${TM_SELECTED_TEXT/^(.*)(\\r?\\n|$)/> $1$2/gm}"
        }
    },
    {
        "key": "ctrl+alt+8", // 引用を「削除」するキー
        "command": "editor.action.insertSnippet",
        "when": "editorTextFocus && editorHasSelection",
        "args": {
            // 選択範囲の各行頭にある「> 」または「>」を削除する
            "snippet": "${TM_SELECTED_TEXT/^> ?//gm}"
        }
    }
]
```

</details>

#### 使用方法（VS Codeでショートカットキーを自作して追加）

1. 対象の文字列を範囲指定
1. `Ctrl` + `Alt` + `9`で選択範囲の文字列の各行頭に「`> `」が挿入

JIS配列のキーボードでは、「`Shift` + `9` → `)`（丸括弧閉じ）」と入力できるため、 `Ctrl` + `Alt` + `9` を`引用表示の追加`のショートカットキーとして割り当てました。

今回は標準的な結果になるような正規表現を定義しましたが、独自の正規表現で変換結果をカスタマイズしたい場合にオススメの方法。

### 変換方法5. PowerShellを使って追加

<details><summary>クリップボードのテキストを引用表示に変換する「Add-MarkdownQuote」</summary>

```powershell
# クリップボードのテキストにMarkdownの引用記号を追加する
function Add-MarkdownQuote {
    [CmdletBinding()]
    [Alias('amq')] # 'amq' という短いエイリアス(別名)を自動で設定する
    param()

    # 1. クリップボードからテキストを取得
    $clipboardText = Get-Clipboard -Raw -ErrorAction SilentlyContinue
    if ([string]::IsNullOrEmpty($clipboardText)) {
        # クリップボードが空なら何もしない
        return
    }

    # 2. 改行コードを判定 (Windows形式か、それ以外か)
    $newline = if ($clipboardText -match "`r`n") { "`r`n" } else { "`n" }

    # 3. 各行を処理
    # .Split()で全行を保持し、1行ずつ変換
    $processedLines = foreach ($line in $clipboardText.Split($newline)) {
        if ([string]::IsNullOrWhiteSpace($line)) {
            # 空行やスペースのみの行は、単一の'>'に
            '>'
        }
        else {
            # テキストがある行は、'> 'を先頭に追加
            '> ' + $line
        }
    }

    # 4. 変換後のテキストを改行で結合してクリップボードに書き戻す
    $resultText = $processedLines -join $newline
    Set-Clipboard -Value $resultText
}
```

</details>

```powershell:エイリアスでの実行方法
amq
```

[こちらの記事](https://zenn.dev/haretokidoki/articles/e2a6c521035d94#参考情報：powershellのプロファイルを使ったプロセス環境変数の設定方法)を参考にプロファイルに登録すると簡単にPowerShellの自作関数が呼び出せるようになりました。

この方法が一番自由度が高いです。
また、クリップボードを使用した方法のため、VS Codeに限らずメールの文章作成やドキュメント作成などでも活用したい場合にオススメしたい方法です。

## 選択範囲の引用表示を解除する方法

数行の場合は手動で先頭行に対して「`> `」を削除するのもありですが、対象文字列が多いと面倒です。
調べた結果、5種類の対応が見つかりました。
（前述した`引用表示の追加`で紹介した5つの方法と同じ手法を用いています。そのため、全体を通してみると冗長的な表現になっています。
　ただ、読者に対して目的に応じた手順を紹介したいため、意図的にこのような記事の構成としています。ご了承ください。）

### 解除方法1. VS Codeのショートカットキーを駆使して解除

追加の際と同様、[こちらの記事](https://zenn.dev/d_ske104/articles/vscode-markdown-put-quote-mark)をベースに手順を考えました。

1. 対象の文字列を範囲指定
1. `Shift` + `Alt` + `I` で選択した各行の末尾にマルチカーソルを挿入
1. `Home` でマルチカーソルの位置を先頭行に移動
1. `Delete` を2回押して先頭行の `> ` を削除

### 解除方法2. VS Codeの置換機能（正規表現を有効）を使って解除

正規表現を使った置換で選択範囲の先頭行にある「`> `（または`>`）」を削除する手順。

1. 対象の文字列を範囲指定
1. `Ctrl` + `H` で置換機能を表示
1. `Alt` + `L` で「選択範囲を検索」をオンにする
1. `Alt` + `R` で「正規表現を使用する」をオンにする
1. 検索欄に「`^> `」を入力
1. 置換欄を「``（空文字）」とする
1. `Ctrl` + `Alt` + `Enter` で「すべて置換」

対象の文字列で「`>`」のみの行が存在する場合は、前述した手順で

5. 検索欄に「`^>$`」を入力
6. 置換欄に「``（空文字）」とする

とすることで対応可能。

### 解除方法3. 専用の拡張機能「Markdown Blockquote Toggler」を使って解除

https://marketplace.visualstudio.com/items?itemName=EliYing.markdown-blockquote-toggler

上記の拡張機能をインストール後、下記の手順で対応可能。

1. 対象の文字列を範囲指定
1. `Ctrl` + `^`で選択範囲の文字列の各行頭にある「`> `（または`>`）」が削除

※ 追加も解除も同じショートカットキーで機能します。

### 解除方法4. VS Codeでショートカットキーを自作して解除

#### 設定方法（VS Codeでショートカットキーを自作して解除）

1. `Ctrl` + `Shift` + `P` でコマンドパレットを開き、「`>keyboard shortcuts`」と入力。
1. 下記の項目を選択
    > 基本設定:キーボードショートカットを開く(JSON)
    > Preferences: Keyboard Shortcuts (JSON)
1. `keybindings.json`が自動で開かれる
1. 下記の内容を追加
    定義するショートカットキーは任意ですが、下記の内容を追加することで、
    選択範囲の文字列を引用表示に変換できます。

    ```json:keybindings.json
    [
        {
            // 既存の定義（追加ショートカットキーを含む）
        },
        {
            "key": "ctrl+alt+8", // 引用を「削除」するキー
            "command": "editor.action.insertSnippet",
            "when": "editorTextFocus && editorHasSelection",
            "args": {
                // 選択範囲の各行頭にある「> 」または「>」を削除する
                "snippet": "${TM_SELECTED_TEXT/^> ?//gm}"
            }
        }
    ]
    ```

<details><summary>補足情報：自作したショートカットキーの追加と解除の定義</summary>

追加と解除の設定方法は同じなので、どちらか対応する際に合わせて設定することをオススメします。

```json:keybindings.json
[
    {
        // 既存の定義
    },
    {
        "key": "ctrl+alt+9", // 引用を「挿入」するキー
        "command": "editor.action.insertSnippet",
        "when": "editorTextFocus && editorHasSelection",
        "args": {
            // 選択範囲の各行頭に「> 」を挿入する
            "snippet": "${TM_SELECTED_TEXT/^(.*)(\\r?\\n|$)/> $1$2/gm}"
        }
    },
    {
        "key": "ctrl+alt+8", // 引用を「削除」するキー
        "command": "editor.action.insertSnippet",
        "when": "editorTextFocus && editorHasSelection",
        "args": {
            // 選択範囲の各行頭にある「> 」または「>」を削除する
            "snippet": "${TM_SELECTED_TEXT/^> ?//gm}"
        }
    }
]
```

</details>

#### 使用方法（VS Codeでショートカットキーを自作して解除）

1. 対象の文字列を範囲指定
1. `Ctrl` + `Alt` + `8`で選択範囲の文字列の各行頭にある「`> `（または`>`）」が削除

JIS配列のキーボードでは、「`Shift` + `8` → `(`（丸括弧開き）」と入力できるため、 `Ctrl` + `Alt` + `8` を`引用表示の解除`のショートカットキーとして割り当てました。

### 解除方法5. PowerShellを使って解除

<details><summary>クリップボードのテキスト内の引用表示を解除する「Remove-MarkdownQuote」</summary>

```powershell
# クリップボードのテキストからMarkdownの引用記号を削除する
function Remove-MarkdownQuote {
    [CmdletBinding()]
    [Alias('rmq')] # 'rmq' という短いエイリアスを設定
    param()

    # 1. クリップボードからテキストを取得
    $clipboardText = Get-Clipboard -Raw -ErrorAction SilentlyContinue
    if ([string]::IsNullOrEmpty($clipboardText)) {
        return
    }

    # 2. 改行コードを判定
    $newline = if ($clipboardText -match "`r`n") { "`r`n" } else { "`n" }

    # 3. 各行を処理
    $processedLines = foreach ($line in $clipboardText.Split($newline)) {
        # 正規表現を使用して、行頭の'> 'または'>'を削除する
        # '^'は行頭を意味するアンカー
        # '> ?'は'>'の後にスペースが0個または1個ある場合にマッチ
        $line -replace '^> ?'
    }

    # 4. 変換後のテキストを改行で結合してクリップボードに書き戻す
    $resultText = $processedLines -join $newline
    Set-Clipboard -Value $resultText
}
```

</details>

```powershell:エイリアスでの実行方法
rmq
```

[こちらの記事](https://zenn.dev/haretokidoki/articles/e2a6c521035d94#参考情報：powershellのプロファイルを使ったプロセス環境変数の設定方法)を参考にプロファイルに登録すると簡単にPowerShellの自作関数が呼び出せるようになりました。

## まとめ

引用表示にする（または解除）5つの方法を紹介しました。

1. VS Codeのショートカットキーを駆使した方法
1. VS Codeの置換機能（正規表現を有効）を使った方法
1. 専用の拡張機能「Markdown Blockquote Toggler」を使った方法
1. VS Codeでショートカットキーを自作する方法
1. PowerShellを使った方法

目的別に方法を提示していくと……

### 事前の作業が不要ですぐ対応できる方法が……

- 方法1. VS Codeのショートカットキーを駆使した方法
- 方法2. VS Codeの置換機能（正規表現を有効）を使った方法

### VS Codeで簡単に対応する方法が……

- 方法3. 専用の拡張機能「Markdown Blockquote Toggler」を使った方法

### 独自の正規表現で変換結果をカスタマイズする方法が……

- 方法4. VS Codeでショートカットキーを自作する方法

### VS Codeに限らずメール作成やドキュメント作成時などにも幅広く使用する方法が……

- 方法5. PowerShellを使った方法

という感じになると思います。
この記事が、同様の課題をお持ちの方にとって、解決の一助となりますと幸いです。

## 参考文献

https://zenn.dev/d_ske104/articles/vscode-markdown-put-quote-mark

https://zenn.dev/haretokidoki/articles/e2a6c521035d94
