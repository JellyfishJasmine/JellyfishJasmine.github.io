# 30% 左右分離型キーボードを作ってみる

## 経緯

### まずは左右分離型な40%キーボードの自作

普段はHHKB Professional BT 無刻印/墨（英語配列） (<https://www.pfu.fujitsu.com/direct/hhkb/detail_hhkb-pro-bt-nl.html>)を愛用しているのですが、以前から自作キーボードにも興味がありました。コロナウイルスの影響で、時間にも気持ちにも余裕ができたので、ふと思い立って作ってみたのが、遊舎工房さんのcorne cherry (<https://yushakobo.jp/shop/corne-cherry/>)でした。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">今さらだけど、ずっと興味があった <a href="https://twitter.com/hashtag/%E3%82%AD%E3%83%BC%E3%83%9C%E3%83%BC%E3%83%89?src=hash&amp;ref_src=twsrc%5Etfw">#キーボード</a> <a href="https://twitter.com/hashtag/%E8%87%AA%E4%BD%9C?src=hash&amp;ref_src=twsrc%5Etfw">#自作</a>！ <a href="https://twitter.com/hashtag/corne?src=hash&amp;ref_src=twsrc%5Etfw">#corne</a> <a href="https://twitter.com/hashtag/cherry?src=hash&amp;ref_src=twsrc%5Etfw">#cherry</a> 作ったよ！頑張ってLEDも54個フル点灯！ <a href="https://t.co/LFkZhSlZfm">pic.twitter.com/LFkZhSlZfm</a></p>&mdash; Jellyfish Jasmine (@jf_jasmine) <a href="https://twitter.com/jf_jasmine/status/1250427677900566536?ref_src=twsrc%5Etfw">April 15, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">実はもう1組組んでたんだけど、こちらはLEDギブアップ。そのかわりキーキャップをポップにしてみた。 <a href="https://t.co/0QBDJ9b5R8">pic.twitter.com/0QBDJ9b5R8</a></p>&mdash; Jellyfish Jasmine (@jf_jasmine) <a href="https://twitter.com/jf_jasmine/status/1250440104599904256?ref_src=twsrc%5Etfw">April 15, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

corne cherryを選んだきっかけは、以下のような感じです。

- せっかくなので左右分離型キーボードが良い
- ロープロファイルは苦手
- せっかくなので縦3キータイプにチャレンジしてみたい

自分用にキーマップもチマチマといじってます。
<https://github.com/JellyfishJasmine/qmk_firmware/tree/master/keyboards/crkbd/keymaps/jellyfishjasmine>

なかなか手強いと言われるLEDも54個全部取り付けて、Gateron Silentな茶軸とKB Paradiseのキーキャップを付けたりして、自作キーボード1号機が完成しました。これはこれで愛着があって気に入っているのですが、さらに自分好みにカスタマイズしたキーボードを自作してみたくなった（＝沼にハマった）と言うわけです。

### 止まらない30%キーボードへの興味

自分でキーボードをデザインする前に試したいことがありました。それは「30%キーボードは自分的にOKなのか？」でした。またCorne Cherryをしばらく使ってみて、自分はcolumn staggeredは苦手なんじゃないか、という疑問もありました。設置のコツが掴めていないだけな可能性も高いのですが、どうも左右にズレてミスタイプする回数が気になっていました。

そこで次に手を出したのが、同じく遊舎工房さんで購入可能なNomu30 (<https://yushakobo.jp/shop/nomu30kit/>)でした。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">調子に乗って#nomu30 も作ってみた。見事に#自作キーボード 沼にハマってる。文房具感高くて、これはこれでカワイイ。 <a href="https://t.co/Id0mEYnAxd">pic.twitter.com/Id0mEYnAxd</a></p>&mdash; Jellyfish Jasmine (@jf_jasmine) <a href="https://twitter.com/jf_jasmine/status/1255023195221970945?ref_src=twsrc%5Etfw">April 28, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

こちらもキーマップを地味にいじってます。
<https://github.com/JellyfishJasmine/qmk_firmware/tree/master/keyboards/nomu30/keymaps/jellyfishjasmine>

確かにこれ以上キー数を少なくすると厳しいけど、このコンパクトさは捨てがたい。あとは、分離してないせいかもしれないけど、row staggered、やっぱり打ちやすい気がする。

## 完成希望図

そんな感じで自作キーボード沼にハマったので、もう少し自分味を出していくことに。今回は、TALP KEYBOARDさんのSU120 自作キーボード用基板（分割キーボードセット）(<https://talpkeyboard.stores.jp/items/5e7ede422a9a4210748af1eb>)をベースに作っていきます。概略としては、Happyな配列でキー数が少ない分割キーボードを目指す感じです。

- 左右分離型
- 30% キーボード （片側縦3キー、横5キー）
- row staggered

（続く）
