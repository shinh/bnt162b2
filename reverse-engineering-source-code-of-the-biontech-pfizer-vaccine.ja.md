---
title: "Reverse Engineering the source code of the BioNTech/Pfizer SARS-CoV-2 Vaccine"
date: 2020-12-25T20:12:20+01:00
draft: false
images:
 - bnt162b2.png
---
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:site" content="@powerdns_bert">
<meta name="twitter:creator" content="@powerdns_bert">
<meta name="twitter:title" content="Reverse Engineering the source code of the BioNTech/Pfizer SARS-CoV-2 Vaccine">
<meta name="twitter:description" content="Welcome! In this post, we'll be taking a character-by-character look at the source code of the BioNTech/Pfizer SARS-CoV-2 mRNA vaccine.">
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

ようこそ!  この記事ではBioNTechとファイザー(Pfizer)のSARS-CoV-2(訳注: いわゆるCOCID-19)向けmRNAワクチン(メッセンジャーRNAワクチン)のソースコードを一文字づつ見ていきます。

> *I want to thank the large cast of people who spent time previewing this
> article for legibility and correctness. All mistakes remain mine though,
> but I would love to hear about them quickly at bert@hubertnet.nl or
> [@PowerDNS_Bert](https://twitter.com/PowerDNS_Bert)*

> *この記事の読みやすさと正確さの確認のために時間を割いてもらった多くの人々に感謝します。
> すべての誤りの責任は自分にありますが、もし誤りがあれば bert@hubertnet.nl もしくは
> [@PowerDNS_Bert](https://twitter.com/PowerDNS_Bert) までご連絡いただければ幸いです*
> (この日本語訳については masahiro.sakai@gmail.com もしくは [@masahiro_sakai](https://twitter.com/masahiro_sakai) まで)

まず、ワクチンのソースコードという表現にいは違和感を感じるかも知れません。
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
さらに、このmRNAを我々の細胞の中に運ぶためにいは、脂質による巧妙なパッケージングが用いられています。

RNAはは、揮発性の「作業メモリー」版のDNAです。
DNAは生物学におけるフラッシュメモリのようなもので、永続性、内部的な冗長性があり、またとても信頼性が高いのです。
しかし、計算機と同様、フラッシュメモリ上のコードを直接実行することはなく、その前により高速で融通が効き、代わりに壊れやすいようなシステムへとコードを複製します。

計算機の場合にはそれはRAMで、生物の場合にはそれはRNAです。
これは特筆すべき対応です。
フラッシュメモリと異なりRAMは、甲斐甲斐しく手入れをしない限り、急速に状態が劣化します。
ファイザーとBioNTechのmRNAワクチンを大型冷凍庫の最深部に保管しなくてはならないのも同じ理由です。
RNAは儚い花なのです。

RNAの一文字は、およそ 0.53&middot;10⁻²¹ ぐらむの重さで、したがって 30 マイクログラムのワクチン用量には、およそ 6&middot;10¹⁶ 文字が含まれています。
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
p時には、異なる（無関係の）ウイルスが用いられることもありました。

mRNAワクチンは、「免疫系を教育する」という同じ目的を、レーザーのような方法で実現します。
ここでレーザーのようなと言っているのは、非常に狭められたという意味と、非常に強力という２つの意味です。

ではどう機能するか説明しましょう。
注射にはSARS-CoV-2の有名な「スパイク」タンパク質を表す一時的な遺伝物質が含まれています。
巧妙な化学的手段によって、ワクチンはこの遺伝物質を我々の細胞の一部の内部に取り入れます。

細胞に取り込まれた遺伝物質は、我々の免疫系が行動を起こすのに充分なSARS-CoV-2のスパイクタンパク質の生産を開始します。
スパイクタンパク質と、(より重要な)細胞が乗っ取られているという明確な兆候に直面して、我々の免疫系はスパイクタンパク質の「両方」に対抗する強力な対応手段を開発します。

これが有効性95%のワクチンを達成したのです。

ソースコード！
----------------
[ドレミの歌のように](https://youtu.be/jp0opnxQ4rY?t=8)、最初から見ていきましょう。
WHOのドキュメントには便利な図があります:

<center>
{{< figure src="https://berthub.eu/articles/vaccine-toc.png"  >}}
</center>

これは一種の目次のようなものです。
小さな帽子のように描かれている「cap」から見ていきましょう。

計算機上のファイルにいきなりオペコードを書いて実行することが出来ないのと同じように、生物学的オペレーティングシステムにはヘッダーが必要であり、リンカーや呼び出し規約のようなものが存在しています。

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

また、最初の２つの `GA` ヌクレオチドは残りのRNAとは化学的に微妙に異ります。
その意味でこの `GA` はOOB(out-of-band)信号を伝達していると言えます。

5' 非翻訳領域
------------
いくつか専門用語を使わせて下さい。
RNA分子は一方向にしか読み進むことが出来ません。
混乱することに、読み始める位置は 5' (ファイブ・プライム) と呼ばれています。
そして、読み込みは 3' (スリー・プライム) 末端で終了します。

生命はタンパク質（やタンパク質によって作られる物質）によって構成されており、それらタンパク質はRNAによって記述されています。
RNAがタンパク質に変換される過程は翻訳と呼ばれます。

以下がワクチンの 5' 非翻訳領域（five prime untranslated region：5' UTR）で、従ってこれはタンパク質には翻訳されません:

```
GAAΨAAACΨAGΨAΨΨCΨΨCΨGGΨCCCCACAGACΨCAGAGAGAACCCGCCACC
```

ここで最初に驚くのは、通常のRNAは A, C, G, U (U は DNA では「T」として知られています) からなるのに対して、 ここには Ψ が含まれています。
これは一体どうしたことでしょうか？

これはこのワクチンの超絶巧妙な点です。
私達の肉体はウィルスに対抗する強力なシステム（"the original one"）を持っているため、細胞は外部から来たRNAを非常に嫌っていて、それが何かをする前に全力で破壊しようとします。

これは我々のワクチンにとってやや問題なので、免疫システムをこっそり越える必要があります。
何年にも渡る実験の結果、RNAにおけるUを微妙に変化させることで、免疫システムの目を逸らすことが分かっていました（マジで！）。

そこで、BioNTechとファイザーのワクチンでは、すべての U を ψ で表される 1-メチル-3'-プソイドウリジリル (1-methyl-3'-pseudouridylyl) に置き換えています。
非常に巧妙なのは、置き換えられた ψ は免疫系を鎮める一方で、細胞の関連する部分においては通常の U として受け入れられる点です。

計算機セキュリティーの分野にも同じトリックがあります。ファイアーウォールやセキュリティソルトウェアを混乱させる一方で、バックエンドのサーバーには受けいられるように微妙に壊したメッセージを送信することで、サーバーをハックすることができることがあるのです。

これは過去に行った基礎科学研究の成果の収穫です。
Ψを用いるテクニックの[発見者たち](https://twitter.com/PennMedicine/status/1341766354232365059)は、[研究](https://www.statnews.com/2020/11/10/the-story-of-mrna-how-a-once-dismissed-idea-became-a-leading-technology-in-the-covid-vaccine-race/)の資金を得て受け入れてもらうために闘わなくてはなりませんでした。
私達はみなこのことに感謝すべきですし、[いずれノーベル賞が授与されるであろう](https://twitter.com/PowerDNS_Bert/status/1329861047168225281)と確信しています。

> ウイルスも ψ テクニックを使って我々の免疫システムを出し抜くことができるのかと、多くの人に聞かれました。
> 簡潔に答えると、その可能性は極めて低いです。
> というのも、生命は 1-メチル-3'-プソイドウリジリル ヌクレオチドを合成する機構を持っていないためです。
> ウイルスは生命の機構に依存して自己複製を用いますが、必要な機構がそこには存在しないのです。
> mRNAワクチンは人間の体内で急速に劣化するので、Ψに置き換えられたRNAがψが残った状態で複製される可能性はありません。
> "[No, Really, mRNA Vaccines Are Not Going To Affect Your DNA](https://www.deplatformdisease.com/blog/no-really-mrna-vaccines-are-not-going-to-affect-your-dna)" (「mRNAワクチンはあなたのDNAには影響を与えません」)も参考になる記事です。

5' 非翻訳領域 に話を戻すと、この51文字は一体何をするのでしょうか？
自然に存在するものは何でもそうであるように、ほとんど何も単一の明確な機能を持っているわけではありません。

私達の細胞がRNAをタンパク質に*翻訳*する必要があるとき、リボソーム（ribosome）という機械を用いて行います。
リボソームはタンパク質の3Dプリンターのようなものです。
それはRNA鎖を取り込み、それに基づいてアミノ酸のストリングを放出し、それはタンパク質に折り畳まれます。
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

The S glycoprotein signal peptide
---------------------------------
As noted, the goal of the vaccine is to get the cell to produce copious
amounts of the Spike protein of SARS-CoV-2. Up to this point, we have mostly
encountered metadata and "calling convention" stuff in the vaccine source
code. But now we enter the actual viral protein territory. 

We still have one layer of metadata to go however. Once the ribosome (from the
splendid animation above) has made a protein, that protein still needs to go
somewhere. This is encoded in the "S glycoprotein signal peptide (extended leader
sequence)". 

The way to see this is that at the beginning of the protein there is a sort
of address label - encoded as part of the protein itself.  In this specific
case, the signal peptide says that this protein should exit the cell via the
"endoplasmic reticulum".  Even Star Trek lingo is not as fancy as this!

The "signal peptide" is not very long, but when we look at the code, there
are differences between the viral and vaccine RNA:

(Note that for comparison purposes, I have replaced the fancy modified Ψ by a
regular RNA U)

```
           3   3   3   3   3   3   3   3   3   3   3   3   3   3   3   3
Virus:   AUG UUU GUU UUU CUU GUU UUA UUG CCA CUA GUC UCU AGU CAG UGU GUU
Vaccine: AUG UUC GUG UUC CUG GUG CUG CUG CCU CUG GUG UCC AGC CAG UGU GUU
               !   !   !   !   ! ! ! !     !   !   !   !   !            
```

So what is going on? I have not accidentally listed the RNA in groups of 3
letters. Three RNA characters make up a codon. And every codon encodes for a
specific amino acid. The signal peptide in the vaccine consists of *exactly*
the same amino acids as in the virus itself. 

So how come the RNA is different?

There are 4³=64 different codons, since there are 4 RNA characters, and
there are three of them in a codon. Yet there are only 20 different
amino acids. This means that multiple codons encode for the same amino acid.

Life uses the following nearly universal table for mapping RNA codons to
amino acids:

<center>
{{< figure src="https://berthub.eu/articles/rna-codon-table.png" caption="[The RNA codon table](https://en.wikipedia.org/wiki/DNA_and_RNA_codon_tables) (Wikipedia)" >}}
</center>

In this table, we can see that the modifications in the vaccine (UUU ->
UUC) are all *synonymous*.  The vaccine RNA code is different, but the same
amino acids and the same protein come out.

If we look closely, we see that the majority of the changes happen in the
third codon position, noted with a '3' above. And if we check the universal
codon table, we see that this third position indeed often does not matter
for which amino acid is produced.

So, the changes are synonymous, but then why are they there?  Looking
closely, we see that all changes *except one* lead to more C and Gs.

So why would you do that? As noted above, our immune system takes a very dim
view of 'exogenous' RNA, RNA code coming from outside the cell. To evade
detection, the 'U' in the RNA was already replaced by a Ψ. 

However, it turns out that RNA with [a higher
amount](https://www.nature.com/articles/nrd.2017.243) of Gs and Cs is
also [converted more efficiently into
proteins](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC1463026/), 

And this has been achieved in the vaccine RNA by replacing many characters
with Gs and Cs wherever this was possible.

> I'm slightly fascinated by the *one* change that did not lead to an
> additional C or G, the CCA -> CCU modification. If anyone knows the reason,
> please let me know! Note that I'm aware that some codons are more common
> than others in the human genome, but [I also read that this does not
> influence translation speed a
> lot](https://journals.plos.org/plosgenetics/article?id=10.1371/journal.pgen.1006024).

The actual Spike protein
------------------------
The next 3777 characters of the vaccine RNA are similarly 'codon optimized'
to add a lot of C's and G's. In the interest of space I won't list all
the code here, but we are going to zoom in on one exceptionally special
bit. This is the bit that makes it work, the part that will actually help us
return to life as normal:

```
                  *   *
          L   D   K   V   E   A   E   V   Q   I   D   R   L   I   T   G
Virus:   CUU GAC AAA GUU GAG GCU GAA GUG CAA AUU GAU AGG UUG AUC ACA GGC
Vaccine: CUG GAC CCU CCU GAG GCC GAG GUG CAG AUC GAC AGA CUG AUC ACA GGC
          L   D   P   P   E   A   E   V   Q   I   D   R   L   I   T   G
           !     !!! !!        !   !       !   !   !   ! !              
```

Here we see the usual synonymous RNA changes. For example, in the first
codon we see that CUU is changed into CUG. This adds another 'G' to the
vaccine, which we know helps enhance protein production. Both CUU
and CUG encode for the amino acid 'L' or Leucine, so nothing changed in the
protein.

When we compare the entire Spike protein in the vaccine, all changes are
synonymous like this.. except for two, and this is what we see here.

The third and fourth codons above represent actual changes. The K and V
amino acids there are both replaced by 'P' or Proline. For 'K' this required
three changes ('!!!') and for 'V' it required only two ('!!'). 

**It turns out that these two changes enhance the vaccine efficiency
enormously**.

So what is happening here? If you look at a real SARS-CoV-2 particle, you
can see the Spike protein as, well, a bunch of spikes:

<center>
{{< figure src="https://berthub.eu/articles/sars-em.jpg" caption="[SARS virus particles](https://en.wikipedia.org/wiki/Severe_acute_respiratory_syndrome_coronavirus) (Wikipedia)" >}}
</center>

The spikes are mounted on the virus body ('the nucleocapsid protein'). But
the thing is, our vaccine is only generating the spikes itself, and we're
not mounting them on any kind of virus body.

It turns out that, unmodified, freestanding Spike proteins collapse into a
different structure. If injected as a vaccine, this would indeed cause our
bodies to develop immunity.. but only against the collapsed spike protein.

And the real SARS-CoV-2 shows up with the spiky Spike. The vaccine would not
work very well in that case.

So what to do? In [2017 it was described how putting a double Proline
substitution in just the right
place](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5584442/) would make the
SARS-CoV-1 and MERS
S proteins take up their 'pre-fusion' configuration, even without being part of
the whole virus. This works because Proline is a very rigid amino acid. It
acts as a kind of splint, stabilising the protein in the state we need to
show to the immune system.

The [people](https://twitter.com/goodwish916) that
[discovered](https://twitter.com/KizzyPhD) this should be walking
around high-fiving themselves incessantly. Unbearable amounts of smugness
should be emanating from them. [And it would all be well
deserved](https://twitter.com/McLellan_Lab/status/1291077489566142464). 

> Update!  I have been contacted by the [McLellan
> lab](https://twitter.com/McLellan_Lab/status/1291077489566142464), one of the
> groups behind the Proline discovery.  They tell me the high-fiving is
> subdued because of the ongoing pandemic, but they are pleased to have
> contributed to the vaccines. They also stress the importance of many other
> groups, workers and volunteers.

The end of the protein, next steps
----------------------------------
If we scroll through the rest of the source code, we encounter some small
modifications at the end of the Spike protein:

```
          V   L   K   G   V   K   L   H   Y   T   s             
Virus:   GUG CUC AAA GGA GUC AAA UUA CAU UAC ACA UAA
Vaccine: GUG CUG AAG GGC GUG AAA CUG CAC UAC ACA UGA UGA 
          V   L   K   G   V   K   L   H   Y   T   s   s          
               !   !   !   !     ! !   !          ! 
```

At the end of a protein we find a 'stop' codon, denoted here by a lowercase
's'.  This is a polite way of saying that the protein should end here. The
original virus uses the UAA stop codon, the vaccine uses two UGA stop
codons, perhaps just for good measure.

The 3' Untranslated Region
--------------------------
Much like the ribosome needed some lead-in at the 5' end, where we found the
'five prime untranslated region', at the end of a protein coding region we find a similar
construct called the 3' UTR. 

Many words could be written about the 3' UTR, but here I quote [what the
Wikipedia
says](https://en.wikipedia.org/wiki/Three_prime_untranslated_region): "The 3'-untranslated region plays a crucial role in gene
expression by influencing the localization, stability, export, and
translation efficiency of an mRNA .. **despite our current understanding of
3'-UTRs, they are still relative mysteries**".

What we do know is that certain 3'-UTRs are very successful at promoting
protein expression.  According to the WHO document, the BioNTech/Pfizer
vaccine 3'-UTR was picked from "the amino-terminal enhancer of split (AES)
mRNA and the mitochondrial encoded 12S ribosomal RNA to confer RNA stability
and high total protein expression". To which I say, well done.

<center>
{{< figure src="https://berthub.eu/articles/vaccine.jpg" >}}
</center>


The AAAAAAAAAAAAAAAAAAAAAA end of it all
----------------------------------------
The very end of mRNA is polyadenylated. This is a fancy way of saying it
ends on a lot of AAAAAAAAAAAAAAAAAAA. Even mRNA has had enough of 2020 it
appears.

mRNA can be reused many times, but as this happens, it also loses some of
the A's at the end. Once the A's run out, the mRNA is no longer functional
and gets discarded. In this way, the 'poly-A' tail is protection from
degradation.

Studies have been done to find out what the optimal number of A's at the end
is for mRNA vaccines. I read in the open literature that this peaked at 120
or so. 

The BNT162b2 vaccine ends with:

```
                                     ****** ****
UAGCAAAAAA AAAAAAAAAA AAAAAAAAAA AAAAGCAUAU GACUAAAAAA AAAAAAAAAA 
AAAAAAAAAA AAAAAAAAAA AAAAAAAAAA AAAAAAAAAA AAAAAAAAAA AAAA
```

This is 30 A's, then a "10 nucleotide linker" (GCAUAUGACU), followed by another 70
A's.

I suspect that what we see here is the result of further proprietary
optimization to enhance protein expression even more.

まとめ
-----
With this, we now know the exact mRNA contents of the BNT162b2 vaccine, and
for most parts we understand why they are there: 

 * The CAP to make sure the RNA looks like regular mRNA
 * A known successful and optimized 5' untranslated region (UTR)
 * A codon optimized signal peptide to send the Spike protein to the right
 place (copied 100% from the original virus)
 * A codon optimized version of the original spike, with two 'Proline'
 substitutions to make sure the protein appears in the right form
 * A known successful and optimized 3' untranslated region
 * A slightly mysterious poly-A tail with an unexplained 'linker' in there

The codon optimization adds a lot of G and C to the mRNA. Meanwhile, using Ψ
(1-methyl-3'-pseudouridylyl) instead of U helps evade our immune system, so
the mRNA stays around long enough so we can actually help train the immune
system.

さらなる読み物など
---------------
In 2017 I held a two hour presentation on DNA, which you can [view
here](https://berthub.eu/dna). Like this page it is aimed at computer
people.

In addition, I've been maintaining a page on '[DNA for
programmers](https://berthub.eu/amazing-dna)' since 2001.

You might also enjoy [this introduction to our amazing immune
system](https://berthub.eu/articles/posts/immune-system/).

Finally, [this listing of my blog posts](https://berthub.eu/articles) has quite some
DNA, SARS-CoV-2 and COVID related material.

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
