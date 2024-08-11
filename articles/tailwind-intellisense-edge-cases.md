---
title: "Tailwind CSS IntelliSenseで珍しいケースに補完を効かせたい"
emoji: "🔍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["tailwindcss", "frontend", "vscode"]
published: false
---

## はじめに
TailwindCSSを使って開発する場合、Intellisenceは割と欠かせないと思っています。
普通にHTMLなどではそのまま補完が効いてくれていいのですが、ちょっと珍しいケースに遭遇したので簡単にまとめます。

### 結論

settings.jsonに以下の設定を追加する。
以下はpythonの特定の形式を対象とするものです。

```json:settings.json
{
... 
"tailwindCSS.includeLanguages": {
    "python": "html"
  },
  "editor.quickSuggestions": {
    "strings": "on"
  },
  "tailwindCSS.experimental.classRegex": [
    "ここにマッチする正規表現を記述する"
  ]
}
```

```javascript:tailwind.config.js
module.exports = {
 content: [
    "./**/foobar/*.py",
  ],
  ...
}
```
&nbsp;

ファイル形式を追加するほどではない場合(js系など)は以下を参考にするといいかと思います。
https://github.com/paolotiu/tailwind-intellisense-regex-list?tab=readme-ov-file


## 解説
### 拡張機能の設定について
念のため言っておくとこちらの拡張機能の話です。
https://marketplace.visualstudio.com/items?itemName=bradlc.vscode-tailwindcss

今回はDjangoのFormで.py形式のファイル内にTailwindCSSのクラスを記述することがあり、そこに対して補完を効かせたかったケースに遭遇しました。
具体的にはこんな感じです。
```python
self.fields["クラス名"].widget.attrs["class"] = "ここにクラス名を入力するときに補完を効かせたい"
```
&nbsp;

ビルド自体は以下でスタイルが適用されるようになります。
```javascript:tailwind.config.js
module.exports = {
 content: [
    "./**/forms/*.py",
  ],
  ...
}
```

そこで補完を効かせるためには拡張機能の設定を変更してあげる必要があります。
この拡張機能ではデフォルトで対象となるファイル形式が決まっています。
詳細は以下のページで定義してあります。以下でhtmlLanguagesとして指定されている形式は補完が効きます。
https://github.com/tailwindlabs/tailwindcss-intellisense/blob/main/packages/tailwindcss-language-service/src/util/languages.ts

Pythonはないため、以下を追加します。
```json:settings.json
{
"tailwindCSS.includeLanguages": {
    "python": "html"
  },
}
```
&nbsp;

そして、今回は=""の内部であるため、任意の文字列として認識されてしまうことから通常では補完対象となりません。
以下を追加します。
```json:settings.json
{
  "editor.quickSuggestions": {
    "strings": "on"
  },
}
```
&nbsp;

最後に特定の形式をマッチさせたいため、以下の設定を記述します。
```json:settings.json
{
  "tailwindCSS.experimental.classRegex": [
    "self\\.fields\\[\".*\"\\]\\.widget\\.attrs\\[\"class\"\\] = \"([^\"]*)\""
  ]
}
```
エスケープ対象文字が多いせいで読みづらいですが、後半の([^\"]*)部分がマッチするようになっています。
""で囲まれた中の"以外の文字にマッチします。
&nbsp;

少しわかりづらいですが、"rou"と入力した時点でrounded系の補完が効いています。
![これまでの設定を適用した場合のエディタ](/images/tailwind-intellisense-edge-cases/intellisense.png)

## まとめ
ファイル形式、文字列形式、正規表現の組み合わせでほぼどんなパターンでも補完を効かせることができるかと思います。
```json:settings.json
{
... 
"tailwindCSS.includeLanguages": {
    "ファイル形式": "html"
  },
  "editor.quickSuggestions": {
    "strings": "on"
  },
  "tailwindCSS.experimental.classRegex": [
    "マッチする正規表現"
  ]
}
```

良い開発ライフを！

### あとがき
アウトプットや情報交換していきたいので良ければX(Twitter)のフォローもお願いします・・！
https://x.com/\_taltalsource\_