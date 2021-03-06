## CSSアーキテクチャ

多くのWebデベロッパとって、良いCSSとはビジュアルモックアップをコードで完全に再現できることを意味する。tableタグを使わず、また出来る限り画像を少なくすることに誇りを持つ。もしあなたが本当に優れたデベロッパであれば、メディアクエリ、Transitions、Transformといった最新で偉大な技術をも使うことだろう。
これらすべてが良いCSSデベロッパに必要なすべてであることは確かではあるものの、スキルとして評価されるときにあまり言及されないCSSのまったく別の側面がある。

興味深いことに、私たちは通常他の言語においてはこのようなことを見過ごすことはない。Railsのデベロッパは仕様通りにコードが動くというだけでは良いコードを書いたとは考えない。それはあくまで基本的なことなのだ。もちろん仕様通りに動くことは必要であるが、気を使う部分は以下のようなことだろう。

そのコードは読みやすいか？変更・拡張しやすいか？アプリケーションのその他のパーツと分離できるか？拡張性があるか？といったことだ。

これらの問いは、コードそのもの以外を評価するときに自然に湧いてくるものであり、CSSでも同じであるべきだ。今時のWebアプリケーションはこれまでよりも規模が大きくなっているため、まともに考えられていないCSSアーキテクチャは開発を不便にする。  
CSSはすべてのWebアプリケーションの一部として他言語同様に評価する時がきている。ただの思いつきや、単にデザイナーの問題として終わらせるわけにはいかない。

## 良いCSSアーキテクチャのゴール

CSSコミュニティにおいて、ベストプラクティスの合意を得るのは非常に難しい。純粋に[CSS Lint](http://csslint.net/)がリリースされたときの[Hacker Newsでのコメント](https://news.ycombinator.com/item?id=2658948)と、[デベロッパの反応](http://2002-2012.mattwilcox.net/archive/entry/id/1054/) から判断するかぎり、根幹的な事柄ですら、CSSを書く人がやるべきこと、やるべきでないことについて多くの人が同意しないのは明白だ。

ここでは私自身のベストプラクティスの議論を並べる代わりに、自分たちのゴールを定義することから始めるべきだと私は考えている。もしこれらのゴールについて同意することができるのであれば、先入観から得た良さからではなく、開発プロセスの阻害要因となりうる悪いCSSを指摘しはじめることができるようになることだろう。

良いCSSアーキテクチャのゴールは、すべての良いソフトウェアのゴールと異なるべきではないと確信している。CSSは予測、再利用、保守、拡張しやすいものであってほしい。

### 予測しやすい

予測しやすいCSSとはルールが期待通りに振る舞うことを意味する。ルールを追加・更新したとき、そのルールが意図せずサイトの一部に影響を与えるべきではない。滅多に変更されない小規模なサイトであれば、このことはあまり重要ではないが、数十、数百ページの大規模なサイトであれば、予測しやすいCSSは必須といえる。

### 再利用しやすい

CSSのルールは抽象的で、十分に分離されているべきである。それはパターンとすでに解決した問題を書きなおす必要なく、既存のパーツから新しいコンポーネントを速くつくることができるということだ。

### 保守しやすい

サイトに新しいコンポーネントと機能が追加・更新されるか、再編される必要があるとき、既存のCSSのリファクタリングを必要とすべきではない。ページにコンポーネントXを追加するときに、そのわずかな存在によってコンポーネントYを壊すべきではない。

### 拡張しやすい

サイトが大きく、複雑に成長していくにつれて、通常はたくさんのデベロッパがメンテナンスのために必要となる。
拡張しやすいCSSとはひとりのデベロッパか、大きなエンジニアチームかを問わず、容易に管理できることを意味する。またそのサイトのCSSアーキテクチャに、巨大な学習曲線を必要することなく容易に近づけるという意味でもある。あなたが今日CSSを触る唯一のデベロッパだからといって、先々にも常にあなただけであるというわけではない。

## よくあるバッドプラクティス

良いCSSアーキテクチャのゴールを達成するための方法を考える前に、ゴールの妨げとなるよくあるプラクティスを考えるのが有用だと考えている。それはしばしば過ちを繰り返すことによってのみ、代替手段の可能性について考えはじめることができるものだ。

以下の例はすべて私が実際に書いたことのあるコードを一般化したもので、技術的に正当であるものの、それぞれ災難や頭痛を招いた。細心の注意を払い、今回は違うと見込んでいたにも関わらず、それらのパターンは常に私を問題に巻き込んだ。

### 親に基づいてコンポーネントを修正する

ほとんどのWebサイトにおいて、たった一箇所を除いて、全く同じ見た目である特定のデザイン要素がある。そして、こうした単発なシチューエーションに直面したとき、ほとんどの新人CSSデベロッパは（経験豊富なデベロッパさえも）同じ方法で対処する。この特定の要素（または自身で作った要素）のユニークな親を見つけ出し、その特定の要素を操作するための新しいルールを書く。

```css
.widget {
  background: yellow;
  border: 1px solid black;
  color: black;
  width: 50%;
}

#sidebar .widget {
  width: 200px;
}

body.homepage .widget {
  background: white;
}
```

一見それほど害の無いものだと思えるが、前述のゴールに基づいて分析してみよう。

最初に、例にあるウィジェットは予測ができない。デベロッパは、誰かがつくったいくつかのウィジェットが同じような見た目であることを期待するも、それをサイドバーまたはホームページで使うとき、マークアップは一緒であるにもかからわず、見た目が異なるだろう。

また再利用性、または拡張性もほとんど無い。ホームと同じ見た目の要素を他のページにも求められた時どうなるだろうか？新しいルールを追加しなければならない。

最後に再利用性や保守性もない。なぜなら、もしそのウィジェットのデザインが変更されたら、あらゆるところにあるCSSを更新する必要があり、また例とは異なり、このアンチパターンとなるルールはめったにそれぞれのすぐ隣には現れることはないからだ。

他言語においてこのようなコーディングをする場合を想像してほしい。

クラス定義を一旦終えてから、特定のユースケースのためだけにそのクラス定義を変更していることになる。これはソフトウェア開発におけるオープン/クローズの原則に直接反する:

> ソフトウェアの仕様（クラス、モジュール、関数、など）は拡張に対してオープンにあるべきで、修正に対してはクローズであるべきだ。

本記事の後半で、どうやって親セレクタに頼らずにコンポーネントをどう修正するのかをみてみよう。

### 過剰に複雑なセレクタ

時々、CSSセレクタの能力を紹介するのに、サイト全体でクラスまたはIDを使わずにスタイルしてみせるような、ショーケースサイトの記事がある。

技術的には正しいが、私はCSSの開発をすればするほど、複雑なセレクタを避けるようになってきた。セレクタが複雑になるほど、HTMLとの関係が密になる。HTMLタグそのものをキレイにしておくことは、CSSを粗く汚くする。

```css
#main-nav ul li ul li div { }
#content article h1:first-child { }
#sidebar > div > h3 + p { }
```

上記のコード例はすべて意味を成している。ひとつめはおそらくドロップダウン型メニューのスタイルであること、ふたつめは記事の主な見出しが他のh1要素とは異なる見た目であることを意味し、最後の例はサイドバーセクションにおいて、最初の段落に何らかの余白を加えたいように見える。

もしHTMLが今後まったく変わらないのであればメリットはありえるが、果たして現実的にHTMLが変わらないことを前提にできるだろうか。過剰に複雑なセレクタは目を引くかもしれないし、見た目のフックとなるHTMLを取り除くことができるが、良いCSS設計のゴールを成し遂げる手助けにはほとんどならない。

上記の例はすべて再利用しづらい。セレクタがマークアップの非常に特定な場所を指していたら、どうやって異なるHTML構造にある他のコンポーネントでこれらのスタイルを再利用できるだろうか？例にある（ドロップダウンの）最初のセレクタを取りあげると、もし同じ見た目のドロップダウンは違うページで、#main-nav要素の中になければどうだろうか？まったく同じスタイルを再度つくることになるだろう。

また、もしHTMLを変更する必要がある場合、これらのセレクタは非常に予測しづらい。デベロッパが3つ目の例にあるdivタグをHTMLのsectionタグに変更したいと思ったときを想像すると、すべてのルールが破綻するだろう。

最後に、HTMLが一定に保たれている時にだけセレクタが機能するので、当然保守性や拡張性もない。

大規模なアプリケーションにおいて、妥協や譲歩は必須となる。
複雑なセレクタの壊れやすさは、HTMLを"クリーン"に保つということの価値とほとんど見合わない。

#### 過剰に一般的なクラス名

再利用しやすいデザインのコンポーネントを作るとき、コンポーネントのクラス名の中のコンポーネントの副要素を（いわば）スコープにするのはとてもよくあることだ。

例を見てみよう:

```html
<div class="widget">
  <h3 class="title">...</h3>
  <div class="contents">
    Lorem ipsum dolor sit amet, consectetur adipiscing elit.
    In condimentum justo et est dapibus sit amet euismod ligula ornare.
    Vivamus elementum accumsan dignissim.
    <button class="action">Click Me!</button>
  </div>
</div>
```

```css
.widget {}
.widget .title {}
.widget .contents {}
.widget .action {}
```

アイデアとしてはtitle、.contents、.actionといった副要素が、同じクラス名を持つその他の要素に広がるという心配をする必要もなく、安全にスタイルできるというものだ。これは正しいが、そのコンポーネント内で同じ名前のクラスをスタイルすることを防ぐことができない。

大規模なプロジェクトにおいて、`.title`のような名前のクラスはその他の文脈やそれ自身としても使われる可能性が高い。そうすると、ウェジェットのタイトルが突然意図とは異なる見た目になる。

過剰に一般的なクラス名は、非常に予測しづらいCSSにしてしまうことがある。

#### 1つのルールで過剰にスタイルする

時々、サイト内のセクション左上の角から20pxに配置する必要があるというような見た目のコンポーネントをつくることがある。

```css
.widget {
  position: absolute;
  top: 20px;
  left: 20px;
  background-color: red;
  font-size: 1.5em;
  text-transform: uppercase;
}
```

やがて、異なる場所でまったく同じ名前のコンポーネントを使う必要があるときがある。前述のCSSは、異なるコンテキストでは再利用しづらいので機能しない。

この問題は、ひとつのセレクタに多くのことをやらせようとしていることにある。同じルールに見た目もレイアウト・配置も定義している。見た目は再利用できるが、レイアウト・配置は再利用しづらい。一緒に使ってしまうことで、ルールすべてが信用できなくなる。

はじめは害がないように見えても、時々CSSの経験が少ないデベロッパがコピー＆ペーストしてしまうのを誘発してしまう。もし新しいメンバーが特定のコンポーネントと同じ見た目のものが欲しいと思った場合、例えば.infoboxというようなものであれば、まず彼らはたぶんそのクラスを試みることから始めるだろう。ところが、新しいinfoboxは望まない形で配置されるため機能しない。次に彼らはどのようにするだろうか？私の経験上、ほとんどの新しいデベロッパは、それらのルールを再利用しやすいパーツに分けることはしない。その代わりに、単純に特定のインスタンスに必要なコードの数行を新しいセレクタにコピー&ペーストし、不要なコードを複製することになる。

### 原因

前述のすべてのバッドプラクティスは類似しており、これらはCSSに過剰な負担をかけている。

少々奇妙に思える言葉かもしれない。スタイルシートなのだから、(すべてではないにしろ)スタイルについての負担に耐えるべきではないか？それこそが私達が望んでいることではないのか？

この問いへの単純な答えは"イエス"だが、いつもの通り、物事はいつもそんなに単純ではない。見た目からコンテンツを切り離すのは良いことだが、CSSをHTMLから切り離したというだけで、見た目からコンテンツを切り離したということにはならない。

言い換えれば、HTMLからすべての見た目に関するコードをはがすことで、CSSにHTML構造に関する深い知識を要したとしたら、ゴールを果たすことにはならないだろう。

さらに、HTMLがただのコンテンツであるということはめったになく、たいていは構造でもあるのだ。また、構造内にCSSで他の要素グループと分離するためだけの目的をもつコンテイナとなる要素も存在しうる。見た目用のクラスが無かったとしても、HTML内に見た目が混ざっていることはまだ明白だ。しかし、コンテンツと見た目が混ざることは必要なのだろうか？

現在の状態のHTMLとCSSでは、それは必要だ。

HTMLとCSSを見た目のレイヤーとして機能させることは必要であり、多くの場合賢い選択だといえる。
コンテンツレイヤーはテンプレートとパーシャルから抽象化できる。

### 解決方法

もしHTMLとCSSがWebアプリケーションのプレゼンテーションレイヤーとして機能するとすれば、良いCSS設計の原則を形成するすべての方法を必要とする。

CSSに対して可能な限り小さなHTML構造を持たせることこそ、ベストアプローチだと考えている。

CSSはビジュアル要素の見た目のセットを定義するべきで、（HTMLとの連結を極力小さくするために）HTMLがどこに出現するかに関わらず定義されたように見えなければいけない。もし特定のコンポーネントが異なるシナリオの中で異なる見た目である必要がある場合、それは異なって呼び出されるべきだし、それこそがHTML側の責務でもあるべきだ。

例をあげると、CSSはボタンコンポーネントを'.button'クラスとして定義するだろう。もしHTMLがボタンのような見た目の特定の要素を求める場合、そのクラスを使うべきだ。もしそうしたシチュエーションで異なる見た目（たぶん大きなものや、幅がいっぱいのもの）のボタンを必要とした場合、新しくクラスを使ってその見た目を定義してやるべきだ。そしてHTMLはその新しい見た目の役割をする新しいクラスを活用すればいい。

CSSがコンポーネントがどういう見た目であるかを定義し、HTMLがページ上の要素に見た目のCSSを割り当てる。CSSはHTML構造との関連が少なければすくないほどよい。

HTMLが何を求めているかを明確に宣言することの大きな利益は、他のデベロッパがマークアップから見つけ、要素がどのような見た目になるかを明確に知ることを許容することだ。

その目的は明白だ。

このプラクティスが無ければ、もしある要素の見た目が意図的か偶発的を伝えることは不可能で、チームを混乱に導く。

マークアップに多くのクラスを置くことに対するよくある異論は、そうすることに余分な努力を必要とすることだ。単一クラスルールはある特定のコンポーネントのたくさんのインスタンスを対象とすることができる。マークアップ上でクラスを何千回も明示的に宣言することは本当に価値があるのだろうか。

その懸念は明らかに正当であるものの、誤解を招きかねない。この意味あいは、CSSで親セレクタを使うか、それともHTMLに1000回クラスを手で書くかということであるが、代替案があることは明白だ。Railsのビューレベルの抽象概念かその他のフレームワークは、同じクラスを何度も書く必要無く、HTMLで明確に宣言された見た目を維持する目的で大いに役立つ。

### ベストプラクティス

これらの過ちを何度も繰り返し、すこし後でその結果に苦しんだ後に、ちょっとしたアドバイスを思いついた。決して総合的であるとはいえないが、私の経験でこれらの原則に忠実であることが良いCSSアーキテクチャのゴールを成し遂げる手助けになるということを説明しよう。

#### 意図的であれ

セレクタが望んでいない要素にスタイルしないようにする確実な方法は、その機会をセレクタに与えないことだ。'#main-nav ul li ul li div'のようなセレクタは非常に容易に、将来的にマークアップが変わったとしても望まない要素にスタイルを適用しやすい。一方で、'.subnav'のような形式であれば、望まない要素に偶然にもスタイルが適用される機会はほとんどありえない。スタイルを適用させたい要素に直接クラスを適用することは、予測しやすいCSSを維持する一番の方法だ。

```css
/* Grenade */
#main-nav ul li ul { }

/* Sniper Rifle */
.subnav { }
```

上の2つの例を見て欲しい。1つ目は手榴弾のように、2つ目をスナイパーライフルだと考えてほしい。この手榴弾は今は思っている通りに機能するが、いつか無実な市民がその爆発圏内に入ってくるかは知りようがない。

#### 関心を分離せよ

すでに言及したが、よく構造化されたコンポーネントレイヤーはCSSの中のHTML構造との関係をゆるめるのを手助けすることができる。付け加えると、CSSコンポーネント自身もモジュールであるべきだ。コンポーネントはそれ自身がどのようなスタイルで、どのように振る舞うかを知っておくべきであるが、それらのレイアウトや配置や、周りの要素との関係からどう引き離されるかについての多すぎる仮説の責任を持つべきではない。

一般的に、コンポーネントはそれらがどのように見えるかを定義すべきで、レイアウトや配置については定義するべきではない。'background'、'color'、'font'のようなプロパティと、'position'、'width'、'height'、'margin'といったプロパティが同じルールにある場合には注意しよう。

レイアウトや配置は他の分離されたレイアウト用のクラスや、コンテナ要素によって制御されるべきだ。（プレゼンテーションとコンテンツを効果的に分離することは、大抵コンテナとコンテンツを分離するために不可欠であることを思い出してほしい。）

#### クラスの名前空間

なぜ親セレクタの利用だけがカプセル化や相互汚染の予防に100%効果的でないことは、すでに調べた通りだ。より良いアプローチはクラス自身に名前空間を持たせる方法だ。もし、ある要素がビジュアルコンポーネントの1つである場合、そのサブ要素となるクラスすべてが、コンポーネントのべースクラス名を名前空間として使用すべきだ。

```css
/* 相互汚染のリスクが高い */
.widget { }
.widget .title { }

/* 相互汚染のリスクが低い */
.widget { }
.widget-title { }
```

クラスに名前空間を持たせることで、コンポーネントを独立、モジュールとして維持させることができる。これは既存のクラスが衝突する可能性を最小限にし、子要素にスタイルを適用するセレクタの詳細度を低く抑えることができる。

#### コンポーネントはモディファイアクラスで拡張する

既存のコンポーネントが、あるコンテキストにおいてわずかに異なる見た目である必要があるとき、その拡張をするためのモディファイアクラスをつくる。

```css
/* Bad */
.widget { }
#sidebar .widget { }

/* Good */
.widget { }
.widget-sidebar { }
```

親要素のひとつに基づいて修正されたコンポーネントのマイナス面についてはすでに分かっているが、繰り返そう: モディファイアクラスはどこでも使うことができる。場所を軸にしたクラスは、特定の場所でしか使えない。モディファイアクラスは必要なだけ何度も再利用することができる。最後に、モディファイアクラスはデベロッパの意図をHTML上で表現する。一方で場所を軸にしたクラスの場合、そのデベロッパしかHTML上で判別できないよう完全に隠蔽し、見落とされる可能性を非常に高めてしまう。

#### CSSをロジカルに体系化せよ

[Jonathan Snook](http://snook.ca/)氏の素晴らしい書籍、[SMACSS](http://smacss.com/)では、CSSルールを4つに分類されたカテゴリで体系化することを唱えている。それはベース、レイアウト、モジュール、ステートの4つだ。ベースはルールのリセットと要素のデフォルトスタイルの定義で構成される。レイアウトはsite-wideな要素の配置や、グリッドシステムのような一般的なレイアウトヘルパーとなる。モジュールは再利用しやすいビジュアル要素で、ステートはJavaScriptでオン・オフを切り替えるスタイルを指す。

SMACSSのシステムでは、モジュール（私がコンポーネントと呼んでいるものと同等のもの）は、すべてのCSSルールの大部分を形成するため、私はもっと抽象的なテンプレートに分類する必要があるルールによく出くわす。

コンポーネントは独立したビジュアル要素だ。一方、テンプレートは構成要素だ。テンプレートは自立せず、見た目を表現することは滅多にない。
テンプレートは単独でコンポーネントを形成するためにまとめられた繰り返しやすいパターンである。

具体的な例を挙げると、モーダルダイアログがコンポーネントのひとつだ。このモーダルはサイトの特徴的なグラデーション背景をヘッダーに持ち、周辺にドロップシャドウ、右上の角にモーダルを閉じるボタン、天地中央に固定配置されるとしよう。これら4つのパターンがそれぞれサイト全体で繰り返して使われるとしたら、その度にこれらのパターンを残そうとは思わないだろう。そのように、これらはすべてのテンプレートに存在し、一緒にモーダルコンポーネントから成る。

例によって私は、特別な理由がない限り、HTML上でテンプレートのクラスを使うことはない。代わりにコンポーネント定義にあるテンプレートのスタイルを含めるためにプリプロセッサを使う。この点と私が合理的にどうするかについては後ほど解説しよう。

#### クラスをスタイルのために使い、スタイルのためだけに使う

大きなプロジェクトに関わったことのある誰しもが、目的がまったく不明なクラスがあるHTMLの要素に出くわすことがあるだろう。それを取り除きたいと思っても、自分が知らないところで何かの目的をもって存在しているかもしれない、とためらってしまう。このようなことが何度も起こると、そのうちHTMLはただ単にチームメンバーはそれらのクラスを削除することを恐れているというだけで、何の目的も持たないクラスで溢れることになる。

この問題はフロントエンドWeb開発において、クラスに対して大抵の場合責任を多く与えすぎることにある。クラスはHTMLの要素のスタイルをつくる、JavaScriptのフックとして機能させる、特徴検出（フィーチャーディテクション）のためにHTMLに追加される、自動テストで使われる、などだ。

これは問題である。クラスがアプリケーションの大部分で使われる時、HTMLからクラスを取り除くのを非常に恐れてしまう。

しかしながら定着した慣習で、この問題を完全に避けることができる。HTML上でクラスを見つけたら、その目的が何であるかをすぐに伝えるべきだ。私のおすすめはスタイルを持たないすべてのクラスに接頭辞をつけることだ。私は、JavaScriptのためにjs-を、Modernizrのためには.supports-を使う。接頭辞がないクラス以外のすべてはスタイルを持ち、スタイルだけをする。

この方法は使われていないクラスの発見や、スタイルシートのディレクトリから検索するように、HTMLから取り除くことができるようになる。JavaScriptを用いてHTMLと`document.styleSheets`オブジェクトを相互参照して、このプロセスを自動化することもできる。

一般的に、見た目からコンテンツを分離することがベストプラクティスであるのと同じように、機能性から見た目を分離するのもまた重要である。見方によれば、スタイルを持つクラスをJavaScriptのフックとして使うことはCSSとJavaScriptを深く結合することになり、機能性を失わずに特定の要素の見た目を更新することが難しい、または不可能な状態にしてしまう。

#### 論理的な構造でクラスの命名をする

最近では多くのデベロッパがCSSを書くときにハイフンを単語の区切りとして使う。しかしハイフン単独では普通十分にクラスのタイプの違いの区別がつかない。

[Nicolas Gallagher](http://nicolasgallagher.com/)は、私も（わずかな変更で）より大きな成果のために導入した[彼のこの問題への解決方法](http://nicolasgallagher.com/about-html-semantics-front-end-architecture/)について記事を書いている。

```css
/* A component */
/* コンポーネント */
.button-group { }

/* コンポーネントの修飾子(`.button`の修飾) */
.button-primary { }

/* コンポーネントのサブオブジェクト (`.button`の中で有効なオブジェクト) */
.button-icon { }

/* これはコンポーネントクラスなのか、レイアウトクラスなのか */
.header { }
```

前述のクラスを見ると、それぞれがどういうルールが適用されるのかを伝えることは不可能だ。これは開発中の混乱を増やすだけでなく、CSSとHTMLを自動的にテストすることを困難にしてしまう。構造化された命名規則はクラス名のみでヒントを与え、他のクラスとはどういう関係か、HTMLのどこに現れるのかが、はっきりと分かる。あらかじめ命名規則があれば、命名をより容易に、テストも可能にしてくれる。

```css
/* テンプレートルール (Sass プレースホルダセレクタ) */
%template-name
%template-name--modifier-name
%template-name__sub-object
%template-name__sub-object--modifier-name

/* コンポーネントルール */
.component-name
.component-name--modifier-name
.component-name__sub-object
.component-name__sub-object--modifier-name

/* レイアウトルール */
.l-layout-method
.grid

/* ステートルール */
.is-state-type

/* スタイルの無いJavaScriptのフック */
.js-action-name
```

最初の例を見直すと、

```css
/* A component */
/* コンポーネント */
.button-group { }

/* コンポーネントの修飾子(`.button`の修飾) */
.button--primary { }

/* コンポーネントのサブオブジェクト (`.button`の中で有効なオブジェクト) */
.button__icon { }

/* レイアウトクラス */
.l-header { }
```

### ツール

効果的でよくまとまったCSS設計の保守は非常にむずかしく、特に大きなチームであるほどそうだ。あちこちにあるいくつかの悪いルールはすぐにごちゃごちゃになり、さらに雪だるま式に増えていき、管理できなくなる。
そうなり始めるとアプリケーションのCSSは詳細度の戦いや、!importantの切り札を使いはじめる

幸運なことにサイトのCSS設計をより容易に制御できるツールはある。

#### プリプロセッサ

最近はプリプロセッサの話題を抜きにして、CSSツールについての話をすることはできず、本記事も例外ではない。しかしそれらの便利さを称賛する前に、いくつかの注意点について提案するべきだろう。

プリプロセッサはCSSをより速く書く手助けをしてくれるが、より良くしてくれるわけではない。結局のところ、プリプロセッサはプレーンなCSSになり、同じルールが適用される。もしプリプロセッサがCSSコーディングを速くするとしたら、それは悪いCSSコーディングを早くするともいえるので、プリプロセッサが問題を解決してくれるかもしれないということを考える以前に、良いCSS設計を理解することが重要だ。

プリプロセッサの特徴として多く挙げられていることは、実際にはCSS設計を非常に悪くしてしまう。下記に挙げるのは、私が避けているコストととしてのいくつかの特徴だ。（これらはの考えは全般的にすべてのプリプロセッサ言語に当てはめることができるが、これらのガイドラインは特にSassに対してのものとなる）

- ネストを単にコードをまとめるために使ってはいけない。ネストは出力されたCSSで必要なときにだけ使う。
- 引数を渡さないミックスインを使ってはいけない。引数を持たないミックスインは、拡張することを前提としたテンプレートとして使われるのが好ましい。
- `@extend`は単一クラスセレクタではないセレクタに使ってはいけない。設計視点からみてつじつまが合わないし、コンパイルされたCSSを膨れ上がらせる。
- コンポーネント修飾子ルールの中のUIコンポーネントのために`@extend`を使ってはいけない、なぜなら継承チェーンを失うからだ。（詳しくは後述）

プリプロセッサの優れた機能は[@extend](http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#extend) と [%placeholder](http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#placeholder_selectors_)だ。両者とも、CSSにルールを追加してCSSを膨らませたり、HTMLに管理が難しくなるほどの膨大な量のベースクラスで溢れさせることなく、容易にCSSを管理できるようにしてくれる。

@extendは注意して使われるべきにも関わらず、そのうちHTMLにそれらのクラスを求めてしまう。例えば、はじめて@extendを覚えたとき、それを次のように修飾子クラスのすべてに使いたいと思うかもしれない。

```css
.button {
  /* button styles */
}

/* Bad */
.button--primary {
  @extend .button;
  /* modification styles */
}
```

このように書くことの問題は、HTML上の継承チェーンを失ってしまうことだ。すぐにこれはJavaScriptでボタンインスタンスを選択することを非常に難しくする。

全般的なルールとして、私はUIコンポーネントや、後でどういうタイプかを知りたいと思う何かに対して拡張することはしない。
これはテンプレートがどういうものであるか、テンプレートとコンポーネントを区別するための別の方法である。
テンプレートは、アプリケーションロジックにおいて今後ターゲットにする必要が無い何かであるから、安全にプリプロセッサで拡張することができるのだ。

ここでモーダルウィンドウをどのような見た目で作られるかの例を下記に挙げると、

```css
.modal {
  @extend %dialog;
  @extend %drop-shadow;
  @extend %statically-centered;
  /* other modal styles */
  /* その他モーダルウィンドウのスタイル */
}

.modal__close {
  @extend %dialog__close;
  /* other close button styles */
  /* その他閉じるボタンのスタイル */
}

.modal__header {
  @extend %background-gradient;
  /* other modal header styles */
  /* その他モーダルウィンドウのヘッダーのスタイル */
}
```

#### CSS LINT

[Nicole Sullivan](http://www.stubbornella.org/content/) と [Nicholas Zakas](http://www.nczonline.net/) は、デベロッパがCSSの中にあるバッドプラクティスを見つける手助けをする、コードの品質ツール [CSS Lint](http://csslint.net/)を作った。彼らのサイトではこのように述べられている。

> CSS Lintの特徴はCSSコードから問題を露見させることにある。基本的な構文チェックも、問題のあるパターンまたは非効率の予兆を探すためのルールセットをコードに適用することをおこなう。CSS Lintのルールはすべてプラグイン的に使えるので、容易に自分用のルールにすることや、望まないルールを省略することもできる。

CSS Lint全般のルールセットは、多くのプロジェクトにおいておおよそ完璧に当てはまるものではないものの、CSS Lintで最も特徴的といえるのは、欲しいルールで厳密にカスタマイズできる能力にある。これはデフォルトで用意されているルールから欲しいルールを選別できるも、自身でルールを書けることを意味する。

CSS Lintのようなツールは、いくらか大きなチームにおいて、少なくとも一貫性、規則の順守の基準を保証するために欠かせない。そして私が先にヒントを出したように、規則を持つことの大きな理由のひとつは、規則を壊す何かの特定を容易にするためのCSS Lintのようなツールを考慮することだ。

先に提案したように、規則に基づくことによって、特定のアンチパターンを調べるためのルールを書くことが非常に容易になる。

- IDセレクタを許可しない
- `div`、`span`といった意味を持たないセレクタを複数のルール上で使わない
- 2つ以上の結合子をセレクタで使わない
- `js-`から始めるクラス名にすることを許可しない
- `l-`接頭辞がついていないルールでレイアウト・配置のスタイルが頻繁に使われるようであれば注意する
- 自身によって定義されたクラスが、その他の何かの子要素として後に再定義されるようになったら注意する

これらはただの提案であることは明白だが、自身のプロジェクトで、あなたが望む標準化をどのように強制するか、を考えるのを目的としている。

#### HTML INSPECTOR

前に私が提案したHTML上のクラスと、リンクしたすべてのスタイルシートを容易に調査できる方法で、もし、どのスタイルシートでも定義されていないのに、HTML上で使われているクラスがあれば注意しよう。現在私が開発している[HTML Inspector](https://github.com/philipwalton/html-inspector) と呼ばれるツールは、このプロセスをより簡単にしてくれる。

HTML Inspectorは（CSS Lintのように）HTMLを横断し、自身で書いたルールで、規則が守られていない時にエラーと注意を返してくれる。

- 同じIDがページ上で2つ以上使われていれば注意する
- どのスタイルシートでも使われていなかったり、（接頭辞に`js-`がある、というような）ホワイトリストをパスしていないクラスは使わない
- 修飾子クラスはベースクラス無しで使われるべきではない
- サブオブジェクトクラスは祖先にベースクラスを含んでいないときに使われるべきではない
- クラスが適用されていない、プレーンな`div`や`span`要素はHTML上で使ってはいけない

### まとめ

CSSは単純にビジュアルデザインをつくるためのものではない。CSSを書いているからといって、プログラミングのベストプラクティスをないがしろにしてはいけない。それは、OOP（オブジェクト指向プログラミング）、DRY（Don't Repeat Yourself）、開放/閉鎖原則、関心の分離、などの概念だ。これらはCSSにも適用できる余地がある。

要するにどんなにコードをうまくまとめるために、その手法が正確に開発をより容易にすることを手助けしたり、長期間に渡ってより保守しやすいかどうかで判断するということを確かめてほしい。
