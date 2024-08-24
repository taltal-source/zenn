---
title: "DjangoのFormとTailwindの組み合わせについてちゃんと考えてみる"
emoji: "🎨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", "django", "tailwindcss"]
published: true
---


## はじめに
Webの開発をする際にDjango・TailwindCSSという組み合わせをよく採用しています。
何かを形にするときの手軽さとTailwindCSSの使い勝手の良さがかなりマッチしていると思っているのですが、以前からFormに関しての相性は微妙に思うところがあり、手軽さと便利さをうまく共存させる方法がないか考えてみます。

※Tailwindはnode環境のCLIでビルドし、生成されたcssを使用する想定です。

## Django Formについて
Djangoでは簡単にFormを作成する方法としてformクラスが提供されています。
詳細は以下。
https://docs.djangoproject.com/ja/5.0/topics/forms/#forms-in-django

### Django Formのメリット/デメリット
#### メリット
ModelFormとFormViewの組み合わせはかなり実装が楽で、Modelで定義した型と制約をもとに適切な要素を配置してくれます。
この組み合わせではサーバでのバリデーションやエラーメッセージの表示等もやってくれて、かなり便利です。
各要素についてはラベルの変更などある程度のカスタマイズ性もあります。

#### デメリット
ロジック的にはかなり便利な一方、見た目的な面では少し微妙です。

以下のように使うことができますが、この場合inputなどの要素にはスタイルは何も適用されず、Template側からも記述できません。

```html:form.html
<form method="POST">
  {% csrf_token %}
    {% for field in form %}
        <div>
            {{ field.label_tag }}
            {{ field }}
            {% if field.help_text %}
                <span>{{ field.help_text }}</span>
            {% endif %}
            {{ field.errors }}
        </div>
    {% endfor %}
  <button type="submit">送信</button>
</form>
```


![DjangoFormをカスタマイズせずFormViewで表示](/images/django-form-tailwind/default_form_view.png)
(題材としてコーヒーを飲んだ記録をつけるものとしています)


### スタイル追加について
上記のデメリットに対してはDjango Formでcssのクラスを付与する方法で対応できます。
```python:forms.py
class CoffeeRecordForm(forms.ModelForm):

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.fields["name"].widget.attrs["class"] = "クラス名"

    class Meta:
        model = CoffeeRecord
        fields = "__all__"
```

&nbsp;
## DjangoとTailwindCSSのform事情
### 選択肢
やっと本題ですが、このDjangoとTailwindCSSを併用する場合

- Django Form × TailwindCSS
- Django Form × django-widget-tweaks × TailwindCSS
- そもそもこの構成をあきらめる

おそらくこの3択あたりになるのではないでしょうか。
これらを比較してみます。


### Django Form × TailwindCSS
先に述べたようにformクラスの__init__でクラスを追加できるため、そこにtailwindのクラスを付与してあげればいいだけです。
```python:form.py
class CoffeeRecordForm(forms.ModelForm):

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.fields["coffee_name"].widget.attrs["class"] = "border p-1 rounded"
        self.fields["has_sugar"].widget.attrs["class"] = "scale-150"
        self.fields["has_milk"].widget.attrs["class"] = "scale-150"
        self.fields["temperature"].widget.attrs["class"] = "border p-1 rounded"
        self.fields["price"].widget.attrs["class"] = "border p-1 rounded"
        self.fields["satisfaction"].widget.attrs["class"] = "border p-1 rounded"
        self.fields["notes"].widget.attrs["class"] = "border p-1 rounded"

    class Meta:
        model = CoffeeRecord
        fields = "__all__"
```

```html:form.html
<form method="POST">
  <div
    class="mx-auto my-4 flex w-1/2 flex-col gap-4 rounded-3xl border bg-white p-4"
  >
    {% csrf_token %} 
    {% for field in form %}
    <div class="flex items-center gap-2">
      <div>{{ field.label_tag }}</div>
      <div>{{ field }}</div>

      {% if field.help_text %}
      <span>{{ field.help_text }}</span>
      {% endif %} {{ field.errors }}
    </div>
    {% endfor %}
    <button
      type="submit"
      class="w-fit rounded-lg bg-blue-600 px-4 py-2 font-bold text-white hover:opacity-70"
    >
      送信
    </button>
  </div>
</form>

```
一番シンプルな解決法かと思います。
VSCodeのTailwind CSS IntelliSenseを使う場合は一工夫が必要です。
このパターンで補完を効かせる方法については以下の記事で詳しく書いています。
https://zenn.dev/taltalsource/articles/tailwind-intellisense-edge-cases

拡張機能を使えない環境などの場合は@applyを使うと上記よりは少しまとまりが良くなります。
(個人的には@applyはナシ派なので微妙ですが・・・。)

#### デメリット
スタイルを記述する箇所がTemplateとFormのクラス内に分散してしまうのが難点です。
特にmarginやpaddingなどはどこに書いたのかややこしくなりがちです。
&nbsp;

### Django Form × django-widget-tweaks × TailwindCSS
上記のデメリットを解消するためにライブラリを追加するパターンです。
下記ライブラリをpipで追加し、settings.pyに追記します。
詳細は以下。
https://github.com/jazzband/django-widget-tweaks


以下のようにhtmlファイルにまとめてクラス名も書けます。
```html:form-tweak.html
{% load widget_tweaks %}
<form method="POST">
  <div
    class="mx-auto my-4 flex w-1/2 flex-col gap-4 rounded-3xl border bg-white p-4"
  >
    {% csrf_token %}
    <div class="flex items-center gap-2">
      {{ form.coffee_name.label }}:
      {% render_field form.coffee_name class+="border p-1 rounded" placeholder="名前" %}
    </div>
    <div class="flex items-center gap-2">
      {{ form.has_sugar.label }}:
      {% render_field form.has_sugar class+="scale-150" %}
    </div>
    <div class="flex items-center gap-2">
      {{ form.has_milk.label }}:
      {% render_field form.has_milk class+="scale-150" %}
    </div>
    <div class="flex items-center gap-2">
      {{ form.temperature.label }}:
      {% render_field form.temperature class+="border p-1 rounded" %}
    </div>
    <div class="flex items-center gap-2">
      {{ form.price.label }}:
      {% render_field form.price class+="border p-1 rounded" placeholder="価格" %}
    </div>
    <div class="flex items-center gap-2">
      {{ form.satisfaction.label }}:
      {% render_field form.satisfaction class+="border p-1 rounded" placeholder="満足度" %}
    </div>
    <div class="flex items-center gap-2">
      {{ form.notes.label }}:
      {% render_field form.notes class+="border p-1 rounded" %}
    </div>
    <button
      type="submit"
      class="w-fit rounded-lg bg-blue-600 px-4 py-2 font-bold text-white hover:opacity-70"
    >
      送信
    </button>
  </div>
</form>
```
この場合だとクラス名はclass+=で記述するため、補完の拡張機能では以下の正規表現を対象とします。
`"class\\+=\"([^\"]*)\""`

#### デメリット
追加のライブラリを使うため管理対象が増えます。
また、記述量はどうしても増えがちです。
&nbsp;



### そもそもこの構成をあきらめる
すごく本末転倒ですが、開発方針として上記2つが受け入れられない場合は諦めてしまうのも手です。

Django Form側を諦める場合は普通にHTMLで書き、Viewのpost関数をオーバーライドするなどで対応できます。
ただし、バリデーションなどは行われないため、手動でサーバサイドでのバリデーションを行う必要があります。

TailwindCSS側を諦める場合は普通にFormクラスでcssのクラスを指定するだけで良いかと思います。
RESTFrameworkにしてしまってReact等を使う手もあります。
&nbsp;

## まとめ
単発の手軽さならFormクラスにスタイルを記述、
継続したメンテナンスや複数人も視野に入れる場合はdjango-widget-tweaksを入れてしまうのが良いかと思いました。
ライブラリ追加を許容するか否かの問題はありますが、TailwindCSSの思想に寄せるならdjango-widget-tweaksはアリだと思います。


### あとがき
個人的な話ですが、僕自身はdjango-widget-tweaksをいったん使ってみることにしました。
(Githubのスター数、更新頻度的にもなんとか許容できるかと思いました。)

アウトプットや情報交換していきたいので良ければX(Twitter)のフォローもお願いします・・！
https://x.com/\_taltalsource\_