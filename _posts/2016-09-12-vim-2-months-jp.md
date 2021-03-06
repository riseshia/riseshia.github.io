---
layout: post
title: "二ヶ月の間、VIMを使ってみた感想"
date: 2016-09-12 23:05:35 +0900
categories:
---

今までメインエディタをVIMに乗り換えようと色々見て、そろそろ使えそうだな、という判断ができたので今までの感想を簡単に共有します。

## Why VIM?

- キーストロークを減らせる
- マウス操作を減らせる
- サーバー内の作業が楽になる
- 使用可能なキーボードの幅を広くできる
- 一貫性ある設定管理
- ~~やっぱ開発者ならCLIでしょう~~

みたいな理由とSublimeも段々不便だな、と思い始めたところで習い始めました。

## How to

### [vimtutor](http://linuxcommand.org/man_pages/vimtutor1.html)

最初はvimtutorから。vimと一緒にインストールされるもので基本的な操作を教えてくれます。何より、自然に昨日を使えるようにしてくれるのはいい。
日本語版もあるので、気軽に始められます。

### [Practical VIM](https://pragprog.com/book/dnvim2/practical-vim-second-edition)

次はこの本。オススメされて読み始めたのですが、VIMを使いたいのなら、この本は必須で読むべきだと思います。
良い点の一つは各例題でどうやってキーボードを操作すればいいのか、それによってどんな結果がもたらされるのか、を親切に見せてくれます。
こんな感じですね。

```
// []はカーソルの位置
{start} => The end is nig[h]
db => The end is [h]
x => The end is[ ]
```

おかげで各キー入力がどんな動作をするのか簡単に、確実に理解できます。

それに、いくつの状況に対する方法の集まりなので、必要なときに必要なものを容易に探せます。例えば、「数式計算ってどうやるんだっけ」とおもったら`Tip 10. Use Counts to Do Simple Arithmetic`を見れば必要な説明を纏めてみれます。なので、この本を読みきったら実務でvimをつかえそうだな、見たいな錯覚を覚えます（笑）。
強いて惜しい点を挙げるとするなら、ここではVIM本体だけでできるものを教えてくれるので、プラグインに対する情報が少ないです。なのでこれが不便だな、と思う時は自力で使えそうなプラグインを探さないといけないですね。


## Plugin

今使っている`.vimrc`設定は[こちら](https://github.com/riseshia/dotfiles/blob/master/vimrc)で確認できます。マネージャーは`Vundle`を使っていますね。他にも色々ある模様ですが、個人的に一番気に入ったこと、かつ一番使われているな、と思ったので選択。
話が逸れましたが、今はこんなプラグインを使っています。

- [SirVer/ultisnips](https://github.com/SirVer/ultisnips): スニペット生成用。思ってるより使ってない気がします。
- [ctrlpvim/ctrlp.vim](http://ctrlpvim.github.io/ctrlp.vim/): ファイルナビゲーションのためのプラグイン。ただ、`ctags` + bufferの使い方を覚えたことと、`node_modules`のせいで初期ローディングの時間が結構長くなってしまって段々使わなくなっています。特定フォルダを無視する設定があるようなので、それを活用してみるか、もしくは削除ですね。
- [tpope/vim-surround](https://github.com/tpope/vim-surround): 囲むタグか「"」、「'」とかの処理を簡単にできるようにしてくれます。短縮キーがめんどいのですが、それなりに使えます。

## 長所／短所

今まで使いながら感じたことです。

### いいところ

- 普段ラインごと最大80文字を守ろうとするので、画面を割るのが結構楽です。左右に割るとちょうど80文字の幅がなるので心が平穏になりますね（笑）。
- 方向キー操作が減りました。もちろん`hjkl`は使っているのですが。右手の動きが少なくなっただけでもそれなりにいいです。
- キーストロークが少し減りました。下で説明しますが、思ったより簡単には減りませんでしたが。
- マクロはいいものですね。単純な置き換えでは対応できないとき（HTMLのフォーマッティングとか）にとてもありがたいです。

### 惜しいところ

- ファイルの間を行き来するのが少し不便です。以前は`Ctrl + [`、`Ctrl + ]`で転換してましたので、`bp`か`CtrlP`プラグインだけだとね。。。まあ、今は画面を割って`Ctrl+w`で対応しています。これで大半の問題は解決できます。
- コードの全体像が見づらいです。前にスクロールが不便だから、という意見をもらったことがあります。その時は同意しがたかったのですが、今は結構納得してます。探せば楽に移動できるプラグインがありそうだな、とか思ってます。
- いろんなファイルをいっぺんに置き換えするのが難しいです。正確にはストロークが多いです。範囲指定して、`vimgrep`して、`cfdo`で置き換え、パイプで`update`して。。。なので楽にしてくれるプラグインないのかな、と色々探してます。
- 思ったよりストロークを減らすのが難しいです。最初だからこそですが、Aという結果を得るために撮れる方法は多様で、その中で一番ストロークが少ない方法が何なのかを常に考える必要があります。考えるのを諦めた瞬間、VIMを使っているメリットがなくなります。。
- ファイルに英語以外のものは書かなくなります。英語の状態ではないと命令を入力ができないからですOTL　なのでこういう作業が多いときには他のエディタを使っています。

## 結論

これだけを見れば何で使うの？っていう考えにも至ると思うのですが、またこれが、思いの外使い心地がよくて、捨てようとは思わなくなります。
それと使いこなせようと努力した分、ちゃんと仕事してくれる感があるのからでしょうか。

思ったよりVIMは難しくありません。ただ、うまく使いこなせるのは簡単ではないな、と切実に感じるこの二ヶ月でした。
しばらくはSublimeとVIMの奇妙な同居が続きそうです。

