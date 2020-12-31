---
title: "BioNTechとファイザーのSARS-CoV-2ワクチンのソースコードのリバースエンジニアリング"
date: 2020-12-25T20:12:20+01:00
draft: false
images:
 - bnt162b2.png
---
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:site" content="@powerdns_bert">
<meta name="twitter:creator" content="@powerdns_bert">
<meta name="twitter:title" content="BioNTechとファイザーのSARS-CoV-2ワクチンのソースコードのリバースエンジニアリング">
<meta name="twitter:description" content="この記事ではBioNTechとファイザーのSARS-CoV-2向けmRNAワクチンのソースコードを一文字づつ見ていきます。">
<meta name="twitter:image" content="https://berthub.eu/articles/bnt162b2.png">

**Translations**: [ελληνικά](https://berthub.eu/articles/posts/greek-reverse-engineering-source-code-of-the-biontech-pfizer-vaccine/)
/ [中文](https://mp.weixin.qq.com/s/b0Mw8uKLYuXHJ5Bj3t2Dwg)
/ [Deutsch](https://berthub.eu/articles/posts/german-reverse-engineering-source-code-of-the-biontech-pfizer-vaccine/)
/ [Español](https://docs.google.com/document/d/1FxswEeem2kP1AUF979P7INBlkD1OzMzVHPEKeU9R2kw/edit)
/ [Français](https://renaudguerin.net/posts/explorons-le-code-source-du-vaccin-biontech-pfizer-sars-cov-2/)
/ [עִברִית](https://github.com/chilik/Hebrew-ReversingSARS-CoV-2mRNAVaccine/blob/main/%D7%94%D7%A0%D7%93%D7%A1%D7%94%20%D7%9C%D7%90%D7%97%D7%95%D7%A8%20%D7%A9%D7%9C%20%D7%A7%D7%95%D7%93%20%D7%94%D7%9E%D7%A7%D7%95%D7%A8%20%D7%A9%D7%9C%20%D7%94%D7%97%D7%99%D7%A1%D7%95%D7%9F%20BioNTech%20-%20Pfizer%20SARS-CoV-2.pdf)
/ [Hrvatski](https://docs.google.com/document/d/1BODRitAvGuDYGZCHU5LY-AkNhs9_1cVDubdRvz-cSPY/edit)
/ [Italiano](https://berthub.eu/articles/posts/italian-reverse-engineering-source-code-of-the-biontech-pfizer-vaccine/)
/ [नेपाली](https://onedrive.live.com/view.aspx?resid=9C571BA15BC4287D!15298&ithint=file%2cdocx&authkey=!ALATa2b8xetI7lQ)
/ [Polskie](https://randomseed.pl/rna/reverse-engineering-kodu-zrodlowego-szczepionki-biontech-pfizer-covid-sars-cov-2/)
/ [русский](https://localcrew.ru/reversepfizer) 
/ [Português](https://docs.google.com/document/d/1pDo40DXcpXjzqAUfhFfup50-IQ2Qct-mhLnmRpjFZWM/edit).
/ [Markdown for translating](https://raw.githubusercontent.com/berthubert/bnt162b2/master/reverse-engineering-source-code-of-the-biontech-pfizer-vaccine.md)

ようこそ!　この記事ではBioNTechとファイザーのSARS-CoV-2向けmRNAワクチン(メッセンジャーRNAワクチン)のソースコードを一文字づつ見ていきます。

> *この記事の読みやすさと正確さの確認のために時間を割いてもらった多くの人々に感謝します。
> すべての誤りの責任は自分にありますが、もし誤りがあれば bert@hubertnet.nl もしくは
> [@PowerDNS_Bert](https://twitter.com/PowerDNS_Bert) までご連絡いただければ幸いです*
> (この日本語訳については masahiro.sakai@gmail.com もしくは [@masahiro_sakai](https://twitter.com/masahiro_sakai) まで)

まず、ワクチンのソースコードという表現には違和感を感じるかも知れません。
ワクチンは腕に注射する液体なのに、そのソースコードとはどういうことなのでしょうか？

これはとても良い質問で、まずはBioNTechとファイザーのワクチンのソースコードの一部を見ていきましょう。
(このワクチンは[BNT162b2](https://en.wikipedia.org/wiki/Tozinameran) もしくは Tozinameran もしくは [Comirnaty](https://twitter.com/PowerDNS_Bert/status/1342109138965422083) としても知られています。)

<center>
{{< figure src="https://berthub.eu/articles/bnt162b2.png" caption="BNT162b2 mRNAワクチンの最初の500文字。出典: [World Health Organization](https://mednet-communities.net/inn/db/media/docs/11889.doc)">}}
</center>

BNT162b mRNAワクチンの中核となるのは、このデジタルなコードです。
これは4284文字で、したがって一連のツイートに収まるほどの長さしかありません。
ワクチン製造過程の一番最初は、このコードをDNAプリンター(!)にアップロードし、このバイト列を実際のDNAの分子に変換することです。

<center>
{{< figure src="https://berthub.eu/articles/bioxp-3200.jpg" caption="[Codex DNA](https://codexdna.com/products/bioxp-system/) 社のDNAプリンタ BioXp 3200" >}}
</center>

DNAプリンタの出力は少量のDNAで、その後に多くの生物的・化学的な処理を経ることでワクチンのアンプルに収まっているRNAになります（RNAについては後で詳しく説明します）。
30マイクログラムの用量には実際に30マイクログラムのRNAが含まれています。
さらに、このmRNAを我々の細胞の中に運ぶためには、脂質による巧妙なパッケージングが用いられています。

RNAはは、揮発性の「作業メモリー」版のDNAです。
DNAは生物学におけるフラッシュメモリのようなもので、永続性と内部的な冗長性があり、またとても信頼性が高いのです。
しかし、計算機と同様、フラッシュメモリ上のコードを直接実行することはなく、その前により高速で融通が効き、代わりに壊れやすいようなシステムへとコードを複製します。

計算機の場合にはそれはRAMで、生物の場合にはそれはRNAです。
これは特筆すべき対応です。
フラッシュメモリと異なり、RAMは甲斐甲斐しく手入れをしない限り、急速に状態が劣化します。
ファイザーとBioNTechのmRNAワクチンを大型冷凍庫の最深部に保管しなくてはならないのも同じ理由です。
RNAは儚い花なのです。

RNAの一文字は、およそ 0.53&middot;10⁻²¹ グラムの重さで、したがって 30 マイクログラムのワクチン用量には、およそ 6&middot;10¹⁶ 文字が含まれています。
バイト単位で表現すると約25ペタバイト(PB)ですが、これが同じ4284文字の約2兆回の繰り返しであることは言っておくべきでしょう。
ワクチンの実際の情報量は1キロバイトちょっとしかありません。
ちなみに、[SARS-CoV-2それ自体](https://www.ncbi.nlm.nih.gov/projects/sviewer/?id=NC_045512&tracks=[key:sequence_track,name:Sequence,display_name:Sequence,id:STD649220238,annots:Sequence,ShowLabel:false,ColorGaps:false,shown:true,order:1][key:gene_model_track,name:Genes,display_name:Genes,id:STD3194982005,annots:Unnamed,Options:ShowAllButGenes,CDSProductFeats:true,NtRuler:true,AaRuler:true,HighlightMode:2,ShowLabel:true,shown:true,order:9]&v=1:29903&c=null&select=null&slim=0)は約7.5キロバイトです。

背景を少しだけ
------------
DNAはデジタルなコードです。
ただし、計算機が0と1を使っているのに対して、生命は A, C, G, UもしくはTの4つのコード(「ヌクレオチド」「ヌクレオシド」「塩基」などと呼ばれます)を用います。

計算機では0と1を電荷の有無、電流（量）、磁化、電圧、信号の変調、反射率の変化などによって保存します。
言い方を変えると、0と1は抽象的な概念ではなく、電子もしくは他の物理的実体として存在しているのです。

自然では、A, C, G, U/T は分子であり、DNA(もしくはRNA)の鎖として保存されています。

計算機では、8ビットを1バイトとしてグループ化し、バイトがデータ処理の典型的な単位となります。

自然では、3つのヌクレオチドを1コドンとしてグループ化し、コドンが処理の典型的な単位となります。
コドンは6ビットの情報を持ちます（DNAの文字あたり2ビットで3文字なので6ビット。つまり、2⁶ = 64 通りの異なるコドンの値があります）。

ここまでは非常にデジタルな話でした。
もし疑問に思ったことがあれば、デジタルコードが記載された[WHOのドキュメント](https://mednet-communities.net/inn/db/media/docs/11889.doc)を自分の目で見てみてください。

> *さらなる文献が[ここにあります](https://berthub.eu/articles/posts/what-is-life/)。
> このリンク 'What is life' (「生命とは何か」) は、このページの残りの意味を理解するのに役立つかも知れません。
> もしくは、動画の方が良ければ、[二時間の動画もあります](https://berthub.eu/dna)。*

ではこのコードは何をするのでしょうか?
--------------------------------

ワクチンは、実際にその病気になることなく、ある病原菌との戦い方を我々の免疫系に教えるためのものです。
歴史的には、弱毒化もしくは不活化したウイルスと、免疫系を怖がらせて行動を起こさせるための「アジュバント」（adjuvant）を注射することによって行われてきました。
これは数十億個の卵や昆虫を用いて行われる極めてアナログな技術です。
さらに、多くの運とリードタイムが必要でした。
時には、異なる（無関係の）ウイルスが用いられることもありました。

mRNAワクチンは、「免疫系を教育する」という同じ目的を、レーザーのような方法で実現します。
ここでレーザーのようなと言っているのは、非常に狭められたという意味と、非常に強力という２つの意味でです。

ではどう機能するかを説明しましょう。
注射にはSARS-CoV-2の有名な「スパイク」タンパク質を表す一時的な遺伝物質が含まれています。
巧妙な化学的手段によって、ワクチンはこの遺伝物質を我々の細胞の一部の内部に取り入れます。

細胞に取り込まれた遺伝物質は、我々の免疫系が行動を起こすのに充分なSARS-CoV-2のスパイクタンパク質の生産を開始します。
スパイクタンパク質と、（より重要な）細胞が乗っ取られているという明確な兆候に直面して、我々の免疫系はスパイクタンパク質とその生成過程の「両方」の複数の側面に対抗する強力な対応手段を開発します。

これが有効性95%のワクチンを達成した仕組みなのです。

ソースコード！
----------------
[ドレミの歌のように](https://youtu.be/jp0opnxQ4rY?t=8)、最初から見ていきましょう。
WHOのドキュメントには便利な図があります:

<center>
{{< figure src="https://berthub.eu/articles/vaccine-toc.png"  >}}
</center>

これは一種の目次のようなものです。
小さな帽子のように描かれている「cap」から見ていきましょう。

計算機上のファイルにいきなりオペコードを書いて実行することが出来ないのと同じように、生物学的オペレーティングシステムにはヘッダーが必要であり、リンカーや呼び出し規約のようなものも存在しています。

ワクチンのコードは以下の２つのヌクレオチドから始まります。

```
GA
```

これは [DOS や Windows の実行ファイルが MZ から始まり](https://en.wikipedia.org/wiki/DOS_MZ_executable)、UNIXのスクリプトが [`#!`](https://en.wikipedia.org/wiki/Shebang_(Unix)) から始まるのに対応しています。
生命の場合でもオペレーティングシステムの場合でも、これらの二文字はいかなる意味でも実行されることはありません。
しかし、それがないと何も起こらないため、そこに存在しなくてはならないのです。

mRNAのcap[は多くの機能を持っています](https://en.wikipedia.org/wiki/Five-prime_cap#Function)。
第一に細胞核から来たコードをマーキングします。
今回の場合、コードは細胞核から来たものではなく、ワクチン接種によってもたらされたものです。
しかし、細胞にそのことを伝える必要はありません。
capの存在は、ワクチンのコードを正当なものに見せかけ、コードが破壊されることを防ぎます。

また、最初の２つの `GA` ヌクレオチドは残りのRNAとは化学的に微妙に異なります。
その意味でこの `GA` はOOB(out-of-band)信号を伝達していると言えます。

5'（ファイブ・プライム）非翻訳領域
------------------------------
いくつか専門用語を使わせて下さい。
RNA分子は一方向にしか読み進むことが出来ません。
混乱しがちな名前ですが、読み始める位置は 5' (ファイブ・プライム) と呼ばれています。
そして、読み込みは 3' (スリー・プライム) 末端で終了します。

生命はタンパク質（やタンパク質によって作られる物質）によって構成されており、それらタンパク質はRNAによって記述されています。
RNAがタンパク質に変換される過程は翻訳と呼ばれます。

以下がワクチンの 5' 非翻訳領域（five prime untranslated region：5' UTR）で、従ってこれはタンパク質には翻訳されません:

```
GAAΨAAACΨAGΨAΨΨCΨΨCΨGGΨCCCCACAGACΨCAGAGAGAACCCGCCACC
```

ここで最初に驚くのは、通常のRNAは A, C, G, U (U は DNA では T として知られています) からなるのに対して、 ここには Ψ が含まれています。
これは一体どうしたことでしょうか？

これはこのワクチンの超絶巧妙な点です。
私達の肉体はウィルスに対抗する強力なシステム（"the original one"）を持っているため、細胞は外部から来たRNAを非常に嫌っていて、それが何かをする前に全力で破壊しようとします。

これは我々のワクチンにとってやや問題なので、免疫システムをこっそりやり過ごす必要があります。
何年にも渡る実験の結果、RNAにおけるUを微妙に変化させることで、免疫システムの目を逸らすことが分かっていました（マジで！）。

そこで、BioNTechとファイザーのワクチンでは、すべての U を ψ で表される 1-メチル-3'-プソイドウリジリル (1-methyl-3'-pseudouridylyl) に置き換えています。
非常に巧妙なのは、置き換えられた ψ は免疫系を鎮める一方で、細胞の関連する部分においては通常の U として受け入れられる点です。

計算機セキュリティーの分野にも同じトリックがあります。ファイアーウォールやセキュリティソルトウェアを混乱させる一方で、バックエンドのサーバーには受けいられるように微妙に壊したメッセージを送信することで、サーバーをハックすることができることがあるのです。

これは過去に行った基礎科学研究の成果の収穫です。
Ψを用いるテクニックの[発見者たち](https://twitter.com/PennMedicine/status/1341766354232365059)は、[研究](https://www.statnews.com/2020/11/10/the-story-of-mrna-how-a-once-dismissed-idea-became-a-leading-technology-in-the-covid-vaccine-race/)の資金を得て受け入れてもらうために闘わなくてはなりませんでした。
私達は皆このことに感謝すべきですし、[いずれノーベル賞が彼らに授与されるであろう](https://twitter.com/PowerDNS_Bert/status/1329861047168225281)と私は確信しています。

> ウイルスも ψ テクニックを使って我々の免疫システムを出し抜くことができるのかと、多くの人に聞かれました。
> 簡潔に答えると、その可能性は極めて低いです。
> というのも、生命は 1-メチル-3'-プソイドウリジリル ヌクレオチドを合成する機構を持っていないためです。
> ウイルスは生命の機構に依存して自己複製を用いますが、必要な機構がそこには存在しないのです。
> mRNAワクチンは人間の体内で急速に劣化するので、Ψに置き換えられたRNAがψが残った状態で複製される可能性はありません。
> "[No, Really, mRNA Vaccines Are Not Going To Affect Your DNA](https://www.deplatformdisease.com/blog/no-really-mrna-vaccines-are-not-going-to-affect-your-dna)" (「mRNAワクチンはあなたのDNAには影響を与えません」)も参考になる記事です。

5' 非翻訳領域 に話を戻すと、この51文字は一体何をするのでしょうか？
自然に存在するものは何でもそうであるように、単一の明確な機能を持っているものはほとんど何もありません。

私達の細胞がRNAをタンパク質に*翻訳*する必要があるとき、リボソーム（ribosome）という機械を用いて行います。
リボソームはタンパク質の3Dプリンターのようなものです。
それはRNA鎖を取り込み、それに基づいてアミノ酸の鎖を放出し、それはタンパク質に折り畳まれます。
リボソームはRNA鎖を取り込み、それに基づいてアミノ酸の鎖を作り出し、タンパク質へと折り畳みます。

<center>
<video controls width="90%" loop>
<source src="https://berthub.eu/articles/protein-short.mp4" type="video/mp4">
</video>
<br/>
出典: [Wikipedia利用者Bensaccount](https://commons.wikimedia.org/wiki/File:Protein_translation.gif)
</center>

上図で起こっているのがこの過程です。
下部にある黒いリボン状のものがRNAであり、緑の部分に現れてくるリボン状のものが形成されるタンパク質です。
飛んできて飛び去っていくのが、アミノ酸およびRNAにフィットさせるためのアダプターです。

リボソームはRNA鎖の上に物理的に接していないと機能することが出来ません。
一旦RNAに接すると、さらに取り込むRNAに基づいてタンパク質の形成を始めます。
このことから、最初に接触した部分については読み込むことができないと想像することが出来ます。
このリボソームの着地領域としての機能は、非翻訳領域の機能の一つに過ぎません。
非翻訳領域は「導線」(lead-in)を提供するのです。

これに加えて、非翻訳領域は「いつ翻訳が起こるべきか」「どの程度翻訳が起こるべきか」というメタデータも含んでいます。
今回のワクチンでは、見つけられた限りで最も「今すぐ」を表す非翻訳領域が用いられており、これは[αグロビン遺伝子(alpha globin gene)](https://www.tandfonline.com/doi/full/10.1080/15476286.2018.1450054)に由来するものです。
この遺伝子は多くの蛋白質を確実に生成することで知られています。
（WHOの資料によれば）過去数年で科学者たちはこの非翻訳領域を更に最適化する方法を見つけており、したがってこれはαグロビンの非翻訳領域そのものではなく、より改良されたものです。

S糖タンパク質シグナルペプチド
-------------------------
先に述べたように、ワクチンのゴールは細胞にSARS-CoV-2のスパイクタンパク質を大量に作らせることです。
これまでに、ワクチンのソースコード中の主にメタデータや「呼び出し規約」にい関わる部分を見てきましたが、今度はウイルス性のタンパク質の領域を見ていきましょう。

ですが、見るべきメタデータの層がまだ一層残っています。
リボソームが（上の素晴らしいアニメーションのように）タンパク質を作ったあと、そのタンパク質はどこかに運ばれる必要があります。
この情報は「S糖タンパク質シグナルペプチド（拡張リーダー配列）」(S glycoprotein signal peptide (extended leader sequence))に符号化されています。

これを理解するためには、まず、タンパク質の先頭に（タンパク質自体の一部として符号化された）宛名ラベルの一種があります。
今回のケースでは、シグナルペプチドには「このタンパク質は『小胞体』（endoplasmic reticulum）経由で細胞外に放出されるべき」と書かれています。
スタートレック用語でさえ、これほど風変わりなものはありません！

この「シグナルペプチド」は長いものではありませんが、実際に見てみるとウイルスのRNAとワクチンのRNAで違いがあることが分かります:

(ただし、比較を簡単にするために、風変わりなψを、通常のRNAのUに置き換えてあります)

```
           3   3   3   3   3   3   3   3   3   3   3   3   3   3   3   3
ウイルス: AUG UUU GUU UUU CUU GUU UUA UUG CCA CUA GUC UCU AGU CAG UGU GUU
ワクチン: AUG UUC GUG UUC CUG GUG CUG CUG CCU CUG GUG UCC AGC CAG UGU GUU
               !   !   !   !   ! ! ! !     !   !   !   !   !            
```

さあ、どうなっているでしょうか？
RNAを3文字毎にグループして並べたのは偶然ではありません。
RNA 3文字はコドンを構成し、すべてのコドンは固有のアミノ酸を符号化しています。
そして、ワクチン中のシグナルペプチドはウイルス中のアミノ酸と*完全に*同じアミノ酸からなっています。

それならば、なぜRNAが異なるのでしょうか？

RNAには4種類の文字があり、コドンは3つの文字からなるので、4³=64種類のコドンが存在します。
しかし、アミノ酸はたったの20種類しかありません。
つまり、複数のコドンが同じアミノ酸を符号化しているのです。

生命は、RNAコドンをアミノ酸へ写像するために、以下のようなほぼ普遍的な表を用いています：

<center>
{{< figure src="https://berthub.eu/articles/rna-codon-table.png" caption="[RNA・コドン表](https://en.wikipedia.org/wiki/DNA_and_RNA_codon_tables) (Wikipedia)" >}}
</center>

この表から、ワクチンでの変更（UUU → UUC）はすべて*同義*のコドンへの変更であることを見て取ることができます。
ワクチンのRNAコードは異なるものではあるものの、結果として得られるアミノ酸とタンパク質は同じものなのです。

さらに、注意深く見ることで、変更の多くはコドンの３文字目（上の比較で「3」と書かれている箇所で）で起こっていることが分かります。
そして、普遍コドン表を確認すると、実際３文字目の変更は生成されるアミノ酸を変えないことが多いことが分かります。

意味を変えない変更ならば、何故そもそも変更する必要があるのでしょうか？
よく見てみると、*一箇所を除いて*すべての変更はCとGを増やす変更になっていることが分かります。

なぜそうするのでしょうか？
上でも述べたように、我々の免疫系は細胞外からやってきた「外来の」RNAを快く思っておらず、このワクチンでは免疫系による検知をすり抜けるために既にRNA中のUをψに置き換えたのでした。

しかし、GとCを[多量](https://www.nature.com/articles/nrd.2017.243)に含むRNAも、[より効率的にタンパク質に変換される](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC1463026/)ことが分かっています。

そして、これはワクチンのRNAにおいて可能な限り多くの文字をGとCに置き換えることによって達成できるのです。

> 自分はCもしくはGの増加に繋がらない*ただ1箇所*の変更である CCA から CCU への変更が少し気になっています。
> もし、理由を知っている人がいたら、ぜひ教えて下さい！
> なお、一部のコドンは他のコドンよりも人間のゲノムの中でより頻繁に現れることは知っていますが、[それが翻訳速度に大きな影響を与えないということも読みました](https://journals.plos.org/plosgenetics/article?id=10.1371/journal.pgen.1006024)。

実際のスパイクタンパク質
--------------------
ワクチンRNAの続く3777文字も、同様にCとGを増やす「コドン最適化」が行われています。
スペースの都合でコード全体をここに掲載することはしませんが、非常に特殊な部分があるのでそこをクローズアップしてみましょう。
これはワクチンが機能するために必要で、したがって我々が普通の生活に戻る助けとなる部分です。

```
                  *   *
          L   D   K   V   E   A   E   V   Q   I   D   R   L   I   T   G
ウイルス: CUU GAC AAA GUU GAG GCU GAA GUG CAA AUU GAU AGG UUG AUC ACA GGC
ワクチン: CUG GAC CCU CCU GAG GCC GAG GUG CAG AUC GAC AGA CUG AUC ACA GGC
          L   D   P   P   E   A   E   V   Q   I   D   R   L   I   T   G
           !     !!! !!        !   !       !   !   !   ! !              
```

ここでも意味を変えないようなRNAの変更がされていることが見て取れます。
例えば、最初のコドンはCUUからCUGに変更されています。
このようにワクチンにGを追加することで、タンパク質の生成を高めることができるのでした。
CUUとCUGは両方ともLで表されるロイシン (leucine) というアミノ酸を符号化しているので、生成されるタンパク質は変わりません。

ワクチン中のスパイクタンパク質の全体をウイルスのそれと比較すると、変更は二箇所を除いてはすべて意味を変えない変更です。
その二箇所がまさに今見ている箇所です。

上記の３つ目と４つ目のコドンが実際の変更箇所です。
KとVで表されるアミノ酸が両方ともプロリン (Proline) を表すPに置き換えられています。
そのために、Kに対しては3文字の変更(「!!!」)と、Vに対しては2文字のみの変更（「!!」）が必要です。

**この二箇所の変更はワクチンの有効性を非常に高めることが判明しています**。

一体何が起こっているのでしょうか？
SARS-CoV-2の実際の粒子を見ると、たくさんのトゲ状になっているスパイクタンパク質が見えます:

<center>
{{< figure src="https://berthub.eu/articles/sars-em.jpg" caption="[SARSウイルスの粒子](https://en.wikipedia.org/wiki/Severe_acute_respiratory_syndrome_coronavirus) (Wikipedia)" >}}
</center>

これらのスパイクはウイルス本体（「ヌクレオカプシドタンパク質」）に固定されています。
ここで鍵となるのは、我々のワクチンはスパイクそれ自体を生成するのみで、スパイクをいかなる種類のウイルス本体にも固定しない点です。

固定されていない独立したスパイクタンパク質は、異なる構造に潰れてしまうことが判明しています。
それをワクチンとして接種すると、免疫を得ることが出来ますが、得られる免疫は潰れたスパイクタンパク質に対するものだけになってしまいます。

そして、実際のSARS-CoV-2はトゲトゲしたスパイクなので、ワクチンはあまり有効に機能しません。

ではどうすれば良いのでしょうか？
2017年に、[適切な箇所を２つのプロリンで置き換えること](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5584442/)によって、SARS-CoV-1 と MERS のSタンパク質を、ウイルス全体の一部にすることなく「結合前」(pre-fusion)の状態に保つことができることが報告されています。
なぜそんなことが出来るかというと、プロリンは剛性の高いアミノ酸であるためです。
プロリンは添え木のような役割を果たして、タンパク質を免疫系に見せたい状態のまま安定化させるのです。

これを[発見](https://twitter.com/KizzyPhD)した[人達](https://twitter.com/goodwish916)はきっと、耐えきれない気持ちの高まりから、ひっきりなしにハイタッチしながら歩き回っているに違いありません。
[それに値するだけの価値があるのです](https://twitter.com/McLellan_Lab/status/1291077489566142464)。

> 追記！ プロリンに関する発見に関わった [マクレラン研究室（McLellan lab）](https://twitter.com/McLellan_Lab/status/1291077489566142464)から連絡がありました。
> パンデミックが進行中なのでハイタッチは控えているものの、このワクチンに対して貢献できて光栄ですとのことでした。
> また、彼ら以外の多くのグループや、労働者やボランティアの果たした役割の重要性についても強調していました。

タンパク質部分の終わりと、次のステップ
--------------------------------
ソースコードの残りをスクロールしていくと、スパイクタンパク質の終わりに小さな変更が見つかります:

```
          V   L   K   G   V   K   L   H   Y   T   s             
ウイルス: GUG CUC AAA GGA GUC AAA UUA CAU UAC ACA UAA
ワクチン: GUG CUG AAG GGC GUG AAA CUG CAC UAC ACA UGA UGA 
          V   L   K   G   V   K   L   H   Y   T   s   s          
               !   !   !   !     ! !   !          ! 
```

タンパク質の終わりに、小文字のsで表される「終止」(stop)コドンがあります。
これはタンパク質の終了を表す行儀の良い方法です。
もとのウイルスではUAA終止コドン一つが用いられているのに対して、ワクチンの方では２つのUGA終止コドンが用いられています。
これはただのオマケかも知れません。

3'（スリー・プライム）非翻訳領域
----------------------------
リボソームが5'末端で「導線」(lead-in)として5'非翻訳領域を必要としていたように、タンパク質のコーディング領域の終わりには3'（スリー・プライム）非翻訳領域 と呼ばれる同様の機構があります。

3'非翻訳領域については、長々と説明することも出来ますが、ここではその代わりに[Wikipediaの説明](https://en.wikipedia.org/wiki/Three_prime_untranslated_region)を引用しておきましょう:
3'非翻訳領域は、mRNAの「局所化」「安定化」「核外輸送」「翻訳後」に影響を与えることにより、遺伝子発現において重要な役割を果たします。
…… **3'非翻訳領域については現在ある程度分かってはいるものの、比較的謎の部分も多い**。

分かっていることは、ある種の3'非翻訳領域はタンパク質の発現の促進に非常に成功していることです。
WHOのドキュメントによれば、BioNTechとファイザーのワクチンの3'非翻訳領域は、「RNAの安定性と高いタンパク質発現量を実現するために、AES（amino-terminal enhancer of split） mRNA とミトコンドリアの12SリボソームRNA（the mitochondrial encoded 12S ribosomal RNA）から取った」とのこと。
私に言えることは、「よくやった」ということです。

<center>
{{< figure src="https://berthub.eu/articles/vaccine.jpg" >}}
</center>


すべての終わりAAAAAAAAAAAAAAAAAAAAAA
----------------------------------
mRNAの一番最後はポリアデニル化されています。
ポリアデニル化というのは大量のAAAAAAAAAAAAAAAAAAAの並びで終わっているということの気取った表現です。
mRNAですら2020年にはもう飽き飽きしてきているようです。

mRNAは何度も再利用することができますが、そのたびに末尾のAのうちのいくらかを失います。
Aを使い果たすと、そのmRNAは機能しなくなり破棄されます。
したがって、多量のAを持つ末尾は、劣化して破棄されることを防いでいます。

mRNAワクチンにとっての最適な末尾のAの数を見つけるための研究がされており、120かそこらがピークになっているという公開の文献を読んだとことがあります。

BNT162b2ワクチンの末尾は以下のようになっています:

```
                                     ****** ****
UAGCAAAAAA AAAAAAAAAA AAAAAAAAAA AAAAGCAUAU GACUAAAAAA AAAAAAAAAA 
AAAAAAAAAA AAAAAAAAAA AAAAAAAAAA AAAAAAAAAA AAAAAAAAAA AAAA
```

30個のAに続いて、「10文字のヌクレオチドのリンカー」（GCAUAUGACU)　があり、更に70個のAが続いています。

これは、タンパク質の発現を更に促進するためにプロプライエタリな最適化手法を適用した結果ではないかと推測しています。

まとめ
-----
これで、BNT162b2ワクチンのmRNAの内容をすべて知ったことになり、そのほとんどの部分についてはそれが何故そこにあるのかがw借りました:

 * RNAが通常のmRNAに見えるようにするためにあるCAP、
 * 既知の成功した5'非翻訳領域に基づいてさらに最適化された5'非翻訳領域、
 * スパイクタンパク質を正しい場所に運ぶための、（ウイルスのシグナルペプチドを100%コピーして）コドン最適化を行ったシグナルペプチド、
 * もとのスパイクをコドン最適化し、タンパク質が正しい形になることを保証するために２つのプロリン置換を行ったスパイクタンパク質、
 * 既知の成功した3'非翻訳領域に基づいて更に最適化された3'非翻訳領域、
 * 未解説の「リンカー」を含む、若干若干ミステリアスな多数のAによる末尾。

コドン最適化はmRNAのGとCを大量に増やします。
一方、Uの代わりにψ（1-メチル-3'-プソイドウリジリル）を用いることで、免疫系を回避してワクチンのmRNAを十分長く存在させ、免疫系を訓練することを（実際には）助けることができます。

さらなる読み物など
---------------
2020年に私が行ったDNAに関する2時間の講演を[ここで見ることができます](https://berthub.eu/dna)。
これはこのページと同様に計算機系の人々に向けた講演になっています。

それに加えて、私は「[プログラマのためのDNA](https://berthub.eu/amazing-dna)」(DNA for programmers)というページを2001年からメンテナンスしています。

[我々の驚異の免疫系に関するこの紹介](https://berthub.eu/articles/posts/immune-system/)も楽しめるかも知れません。

最後に、[私のブログの記事一覧](https://berthub.eu/articles)には、DNA、SARS-CoV-2、COVIDに関する記事がかなりあります。

<!--

Aminoacids
----------
Life is built out of or by proteins. Proteins are built out of 20 different
building blocks called amino acids. These amino acids have different chemical
and physical properties. Some attract each other, some are rigid, some have
a positive charge etc.

Once amino acids are chained together we call them a 'protein', and will
adopt a shape.  This is the famous 'folding'.  The shape and state of a
protein can depend on temperature, acidity, presence of (UV) light, magnetic
fields (!!) and many other things.

The famous "Spike" of SARS-CoV-2 is such a protein. 

As noted, DNA is organized in 6-bit codons. Each codon uniquely encodes for
a single amino acid, using a nearly universal table:

<center>
{{< figure src="https://berthub.eu/articles/codon-table.png" caption="[The codon table](https://en.wikipedia.org/wiki/DNA_and_RNA_codon_tables) (Wikipedia)" >}}
</center>

From this table, we see that 'CCT' in DNA corresponds to the Proline
amino acid.


Ok but what is REALLY in the vaccine
------------------------------------
So, how does this work? DNA is like the 'flash drive' storage of biology.
DNA is very durable, internally redundant and very reliable. But much like
computers do not execute code directly from a flash drive, before something
happens, the code gets copied to a faster, more versatile yet far more
fragile system.

For computers, this is RAM, for biology it is RNA. The resemblence is
striking. RAM degrades very quickly unless lovingly tendered. The reason the
Pfizer/BioNTech mRNA vaccine must be stored in the deepest of deep freezers is the
same: RNA is a fragile flower.

We'll get to the difference later.

How the vaccine works
---------------------



For this reason also, there are no RNA printers. The DNA printer emits tiny
amounts of DNA, which are then injected into bacteria as little circular
loops of DNA called plasmids.



-->
