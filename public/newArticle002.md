---
title: VS Codeで選択範囲を引用表示する方法
tags:
  - VSCode
private: true
updated_at: '2025-08-05T10:32:09+09:00'
id: b07459f10b9508adec43
organization_url_name: null
slide: false
ignorePublish: false
---

VS Codeの標準的な

## mdファイル上で「`Ctrl` + `/`」でコメントアウトすると

```markdown:引用表示にしたいテキスト群
- 項目1
- 項目2
- 項目3
- 項目4
- 項目5
```

```markdown:「Ctrl + /」するとHTMLコードとしてコメントアウトされる
<!-- - 項目1
- 項目2
- 項目3
- 項目4
- 項目5 -->
```

## 選択範囲を引用表示にする方法

数行の場合は手動で追加するのもあり。

### VS Codeのショートカットキーを駆使して追加

ネットで検索した結果、[こちらの記事](https://zenn.dev/d_ske104/articles/vscode-markdown-put-quote-mark)を参照にさせて頂きました。

> 1. 対象のテキストをドラッグなどして範囲選択する。
> 1. Shift + Alt + I [1] で選択範囲の行末にマルチカーソルを置く。
> 1. Home キーでマルチカーソルを行頭に移動させる。
> 1. 引用符 > を入力する。
> 1. おわり。
>
> 引用元：[手順 - VSCodeで複数行にMarkdownの引用符を付ける｜Zenn](https://zenn.dev/d_ske104/articles/vscode-markdown-put-quote-mark#%E6%89%8B%E9%A0%86)

### VS Codeの置換機能（正規表現を有効）を使って追加

選択範囲の先頭行に対し、「`> `」を追加。

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

この「`> `」は、[markdownlint](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint)などで警告表示されてしまいます。
これを解消するため、「`> `」を「`>`」のみにする場合は、前述した手順で

5. 検索欄に「`^> $`」を入力
6. 置換欄に「`>`」を入力

に置き換えることで対応可能です。

```markdown:追加の置換で「> 」を「>」に変換
> - 項目1
> - 項目2
>
> - 項目3
> - 項目4
> - 項目5
```

### 専用の拡張機能「Markdown Blockquote Toggler」を使って追加

https://marketplace.visualstudio.com/items?itemName=EliYing.markdown-blockquote-toggler

上記の拡張機能をインストール後、`Ctrl` + `^` で変換可能。

### PowerShellを使って追加

<details><summary>クリップボードのテキストを引用表示に変換する「ConvertTo-MarkdownQuote」</summary>

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

```powershell
amq
```

`. $PROFILE`でプロファイルに登録すると、より便利に。

## 選択範囲の引用表示を解除する方法

数行の場合は手動で解除するのもあり。

### VS Codeのショートカットキーを駆使して解除

https://zenn.dev/d_ske104/articles/vscode-markdown-put-quote-mark

1. 対象の文字列を範囲指定
1. `Shift` + `Alt` + `I` で選択した各行の末尾にマルチカーソルを挿入
1. `Home` でマルチカーソルの位置を先頭行に移動
1. `Delete` を2回押して先頭行の `> ` を削除

### VS Codeの置換機能（正規表現を有効）を使って解除

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

### 専用の拡張機能「Markdown Blockquote Toggler」を使って解除

https://marketplace.visualstudio.com/items?itemName=EliYing.markdown-blockquote-toggler

`Ctrl` + `^` で変換。

### PowerShellを使って解除

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

```powershell
rmq
```

`. $PROFILE`でプロファイルに登録すると、より便利に。

## 参考文献

https://zenn.dev/d_ske104/articles/vscode-markdown-put-quote-mark
