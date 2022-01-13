---
theme: default
background: /david-edelstein-N4DbvTUDikw-unsplash.jpg
class: 'text-center'
download: true
highlighter: 'prism'
lineNumbers: true
colorSchema: 'dark'
aspectRatio: '16/9'
drawings:
  persist: false
fonts:
  sans: 'Noto Sans JP'

---

<div class="font-black text-8xl">
  
# はじめてのGemを作った話

</div>
<span class="font-800 text-2xl text-lime-100">日本の地方公共団体コードを扱うためのGem「jp_local_gov」</span>

<div class="absolute bottom-10 left-16">
  <span class="font-700 text-xl">
    2022-01-20 K-Ruby#28
  </span>
</div>

<div class="absolute bottom-10 right-16">
  <span class="font-700 text-xl">
    @ikuma-t
  </span>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# 目次

1. 自己紹介
2. 作ったGemの紹介
3. 工夫したところ
4. 感想

---

# 自己紹介

K-Rubyは初参加です！よろしくお願いします😄

<div class="grid grid-cols-2 gap-4">

- 名前：ikuma-t 
  - <a href="https://github.com/IkumaTadokoro" target="_blank"><img alt="Github" src="https://img.shields.io/badge/IkumaTadokoro-%2312100E.svg?&style=flat-square&logo=Github&logoColor=white" class="inline" /></a>
<a href="https://twitter.com/ikumatdkr" target="_blank"><img alt="Twitter" src="https://img.shields.io/badge/@ikuma-%231DA1F2.svg?&style=flat-square&logo=twitter&logoColor=white" class="mx-1 inline" /></a>
<a href="https://zenn.dev/ikuma" target="_blank"><img alt="Zenn" src="https://img.shields.io/badge/ikuma-3EA8FF.svg?&style=flat-square&logo=Zenn&logoColor=white" class="mx-1 inline" /></a>
<a href="https://speakerdeck.com/ikumatadokoro" target="_blank"><img alt="SpeakerDeck" src="https://img.shields.io/badge/ikuma_t-009287.svg?&style=flat-square&logo=SpeakerDeck&logoColor=white" class="mx-1 inline" /></a>
- [#fjordbootcamp](https://twitter.com/hashtag/fjordbootcamp)で学習中です！！
- 趣味：ツール・自動化、作曲家巡り、ドット絵
- 住んでいるところ：千葉県
- 今年のお正月は鹿児島黒牛の牛しゃぶを食べました@千葉


<div class="flex flex-col">

<div class="flex flex-row justify-around">

<img src="/ikuma-t.png" class="w-30 rounded-full"/>
<img src="/fjord.png" class="w-30 rounded-full" />
<img src="/chiba.png" class="w-30 h-30 rounded-full" />

</div>

![](/kurogewagyu.png)

</div>

</div>


---
class: 'text-center'
---

<div class="absolute top-60 right-80">

<h1 class="text-8xl font-black">🦀作ったGemの紹介🦀</h1>

</div>

---

# Gem：jp_local_gov

<v-clicks>
<div class="my-4">

### 概要

</div>

- [GitHub](https://github.com/IkumaTadokoro/jp_local_gov)
- [RubyGems](https://rubygems.org/gems/jp_local_gov)
- 日本の地方公共団体コードをパースしてくれるGem
- > 全国地方公共団体コードは、日本の地方公共団体につけられた、数字3桁または5桁または6桁の符号（コード）である <br> [全国地方公共団体コード：Wikipedia](https://ja.wikipedia.org/wiki/%E5%85%A8%E5%9B%BD%E5%9C%B0%E6%96%B9%E5%85%AC%E5%85%B1%E5%9B%A3%E4%BD%93%E3%82%B3%E3%83%BC%E3%83%89)
- 例えば鹿児島県鹿児島市は、`462012`というコードが割り振られている。

</v-clicks>

<v-clicks>
<div class="my-4">

### 主な機能 

</div>


1. 地方公共団体コードからの地方公共団体情報の取得（正引き）
2. AND条件検索での地方公共団体情報の取得
3. 【Rails】Modelに地方公共団体コードを扱うメソッドを拡張する

</v-clicks>

---

# コードの変換

<div class="mt-4">

<v-clicks>

<div class="my-4">

### 地方公共団体コード → 地方公共団体の情報

</div>

```ruby {1|3|6|7|2-10}
JpLocalGov.find('462012')
# <JpLocalGov::LocalGov:0x000000010c4cd670                   
#  @city="鹿児島市", 
#  @city_kana="カゴシマシ",
#  @code="462012",
#  @prefecture="鹿児島県",
#  @prefecture_capital=true,
#  @prefecture_code="46",
#  @prefecture_kana="カゴシマケン"
# >          
```

<div class="my-4">

### 地方公共団体を条件検索

</div>

```ruby {1,2|4,5,6}
# 複数条件でAND検索が可能
kagoshima = JpLocalGov.where(prefecture: '鹿児島県', prefecture_capital: true)

# 条件によっては複数ヒットする可能性もあるので、戻り値はArray
kagoshima.map(&:city)
# ["鹿児島市"]
```

</v-clicks>

</div>


---

# Railsで使用する

- Modelクラスに`include`することで、地方公共団体コード関連の拡張メソッドを使用することができる。
- DBには地方公共団体コードさえ保持していればOK

<v-clicks>

```ruby
# app/models/insurance_fee.rb:
class InsuranceFee < ActiveRecord::Base
  include JpLocalGov
  jp_local_gov :local_gov_code
end
```

```ruby
# app/controllers/insurance_fee_controller.rb:
class InsuranceFeeController < ActionController
  def index
    @insurance_fees = InsuranceFee.all
  end
end
```

```ruby {3-4}
# app/views/insurance_fee/index.html.erb
<% @insurance_fees.each do |insurance_fee| %>
  <p><%= insurance_fee.local_government.city %></p>
  # DBに保存された地方公共団体コードに紐づく、地方公共団体名が表示される。
<% end %>
```

</v-clicks>

---

# どうして作ったのか

自作サービスで必要だったので...

![](/img_1.png)

<v-clicks>

- 自作サービスで地方公共団体ごとにレコードを持つ必要があった
- ちゃんとDBを正規化すると、「地方公共団体テーブル」が必要
- 「だるいなあ、なんかGemないかなあ」
- 「一応あったけど、データが更新されてなくて使えない & 自分の欲しい機能がない」
- 作るか！！！！🔥

</v-clicks>

<v-after>

...なお自作サービスは鋭意製作中です

</v-after>

---

# 工夫したところ①：データ更新処理の自動化

- 似たようなアイデアのGemはあったが、更新が4年以上前で、最新のデータに対応していない
- Gemの実装自体は複雑なものではないので、データ更新作業がボトルネック：**→自動化しよう🦀**

![](/auto-update.png)

---

# 工夫したところ②：RBSの導入

<div class="grid grid-cols-2 gap-4">

<v-clicks>

- `bundle gem`した数日後にTLに流れてきた
- でかいGemじゃないので型定義の恩恵はあまりないけれど、「小さなRubyistもRBSを見ているぞ！Ruby好きだぞ！」という謎アピール
- RBSファイルの自動生成スクリプト、CIでのSteepによるチェックまで導入
![](/steep_check.png)
- たぶんRBSファイルよりも<del>余計な</del>自動化スクリプトの方が長いくらい

</v-clicks>



<Tweet id="1465868273681985536" />

</div>

---

# 感想：初めてGemを作ってみて

<div class="my-4">

### 作り手になることで、よりOSSが身近になった

</div>

<v-clicks>

- 今までもGemを使っている箇所に当たったら、ソースリーディングとかはしていた。
  - MinitestとかOmniAuthとかRailsとか
- けど作ってみたのは初めて
- 作る側になることで、今まで機能しか見ていなかったところから、それ以外の構成とか、メンテナの労力もわずかながらわかるようになった

</v-clicks>

---

# 感想：初めてGemを作ってみて

<div class="my-4">

### もっと強くなりてえな

</div>

<div class="grid grid-cols-2 gap-4">

<v-clicks>

- 静的型チェッカーのSteepはActiveSupport7.0にあげると動かない（2022/01/13時点）
- このGemもテストコードでActiveRecordを利用しているため、影響があった。が、自分ではPRは送れなかった...。
- 自分はただIssueに👍をつけるだけで精一杯だった...。
- もっと力をつけて、OSSを利用する側であるとともに、OSSに貢献できるように、今年も勉強頑張りたい💪

</v-clicks>

![](/steep-issue.png)

</div>
