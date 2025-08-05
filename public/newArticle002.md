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
## mdファイル上で「`Ctrl` + `/`」でコメントアウトすると

```markdown:引用表示にしたいテキスト群
1. 項目1
1. 項目2
1. 項目3
1. 項目4
1. 項目5
```

```markdown:「Ctrl + /」するとHTMLコードとしてコメントアウトされる
<!-- 1. 項目1
1. 項目2
1. 項目3
1. 項目4
1. 項目5 -->
```

## 対応方法

### 数行の場合は手動で追加

### 正規表現を使う方法

選択範囲の先頭行に対し、「`> `」を追加。
なお、可能であれば空行に対しては、「`>`」だけにしたい。

XXX

## 参考文献

https://zenn.dev/d_ske104/articles/vscode-markdown-put-quote-mark
