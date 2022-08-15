# 巨人の肩に乗ろう

この記事に書かれていることは、エミュレータ開発だけでなく、もっと幅広い分野に当てはまると思いますが、それでも私はこの記事をそのような観点から紹介したいと思います。

新しいエミュレータのプロジェクトを始めようとするとき、何から始めればいいのか、非常に悩むことがあります。他のエミュレータのソースコードを見てもいいのだろうか？ 様々なディテールの実装方法を説明したノートを読んでもいいのでしょうか？ それとも、ただのコピーなのでしょうか？ などと疑問に思う箇所はたくさんあります。

また、どうすればエミュレータ開発シーンに良い影響を与えることができるのでしょうか？

## 筆者の意見

2004年にbsnesを始めたとき、私はゼロから始めたわけではありません。1996年から2004年の間にSNESのリバースエンジニアリングとエミュレーションを手伝ってくれた何十人もの才能ある人たちの知恵と研究を結集して始めたのです。そのときに私はZSNESやSnes9Xなどのソースコード、ドキュメント、フォーラムへの投稿、バグフィックスの変更履歴など、8年分の進捗状況を手にすることができました。

(また、自分が手がけたスーファミゲームの翻訳パッチの開発を通じてリバースエンジニアリングの経験が6年分あったのも収穫でした。）

bsnesの初期に遡って検索してみると、bsnesが初めての大きなエミュ開発プロジェクトだったにもかかわらず、わずか半年で彼らの一般的な精度や互換性のレベルに追いつくことができたのです。

これが私の特別な才能を表しているというのは冗談ですが、私には多くの助けがありましたし、そのことを恥じてはいません。私は、先人たちを頼り、彼らの仕事を研究し、彼らに質問し、彼らの忍耐と援助から大いに恩恵を受けました。

anomie氏やTRAC氏などがbsnesの実現に貢献してくれました。彼らのおかげで今の私があるのです。

## Success Begets Success

半年間の開発期間を経て、今度は私が最前線で技術的な改善を行う番になりました。

スーファミのサイクルタイミングをより正確に解析するための新しい技術を開発したり、何百ものテストROMを書いて、無数のエッジケースや新しい動作を確認したりしました。

既存の知識に頼っていた私は、自分で新しい情報を発見することに移行しなければなりませんでした。

そのためには、ハードウェアのセットアップと、スーファミ用のプログラムを書くための知識が必要です。

厳密にはROMハックをしていた頃から持っていたものですが、新しいエミュレータを作るのに必要なものではありませんでした。

最近では、主要なレトロシステムについては、オリジナルのハードウェアを所有していなくても、比較的高品質なエミュレータを作成できるだけの情報があります。

例えば、MiSTerのSNESエミュレーションコアは、スーパーファミコンを持っていない人がbsnesのソースコードだけを参考にして作ったものです。

## Returning the Favor

bsnesは現在、15年前から積極的に開発を続けています。しかし、私は自分の原点を忘れたわけではありません。むしろ、恩返しをしたいと思っています。今日のSNES界では、私の貢献はほとんどすべての分野で見ることができます。Snes9X、SNESGT、Mesen-S、Super Ntなどの質問に答えたり、ソースコードを提供したりしています。

私のソースコードは常にオープンソースで、必要に応じて他のプロジェクトで使用するために再ライセンスしたこともあります。例えば、Snes9Xの非商用ライセンスのための私のAPUコアなどです。

そして、私が以前のZSNESやSnes9Xにほんのわずかな時間で追いつけたように、2018年以降の最新のスーファミエミュレータもすぐに我々のレベルに追いつくことができました。

スーパーNtはわずか9ヶ月で、Mesen-Sはわずか4ヶ月で作られました。

## Research

例えば、スーファミの『Speedy Gonzales』は悪名高いバグでした。15年ほど前から、どのエミュレータでもこのゲームを動かす方法がわからなくなっていました。ステージ6-2の途中で何の理由もなくデッドロックしてしまうのです。

私は2週間のうち、約80時間をこのゲームのリバースエンジニアリングに費やし、何が起こっているのかを理解しようとしました。

問題となるコードを回避する方法はいくつかありました。しかし、正解だった方法はたった1つだけでした。他の方法を実行すると、その問題はなおっても新たに多くのバグが生まれてしまう有様でした。誤って修正すると、将来的に別のゲームを壊してしまう可能性もあります。

そのためには、他の可能性を排除するために、徹底的にテストを行い、確実に正しい方法を見つけることが必要でした。

最終的にたどり着いた結論は、「ゲームはマッピングされていないメモリアドレスから読み込んで、セットされることのないビットを待っていた」というものでした。しかし、偶然にも、何千ものスキャンラインの後、Hblank DMA転送が、そのビットがセットされた値をフェッチし、サイクルがうまく揃えば、その値はバス上に残り、ループ状態は終了しました。

この現象が発見された後、私はその答えを他のSNESエミュレータ開発者に提供し、彼らは数秒でこの動作を実装することができました。

これは、昔のSnes9Xの変更履歴や、問題のあるゲームに関するフォーラムの投稿を読んで得られたバグフィックスと同じです。

## Cheating?

エミュレータ開発はテストを受けるようなもので、他のエミュレータはそのテストの回答キーを持っているようなものだと考えれば、そのように見えるかもしれません。また、ハードウェアのリバースエンジニアリングを学ぶことが目的であれば、過去に行われたことを学ばなくても構わないでしょう。

しかし、もしあなたの目的がオリジナルのハードウェアの保存であるならば、それは決して「不正行為」ではありません。現実的に考えて、利用可能なリソースを活用しているだけなのです。

この世の時間は限られていて、私たちが保存しようとしているハードウェアは、もう若くもなく、すぐに手に入るものでもありません。最終的には、動くオリジナルのハードウェアを手に入れて分析することはできなくなるでしょう。それは、すでにいくつかの非常に古い希少なシステムではそうなっていますし、最新のものでも[アタリ ジャガーCD](https://ja.wikipedia.org/wiki/Atari_Jaguar)のように法外に高価で壊れやすいものになっています。

車輪を再発明する必要はありません。今ある道具を利用して、さらに良い車輪を作る努力をしましょう。罪悪感があっても、それを社会に還元すれば、簡単に解消されます。

## 終わりに

私が望むのは、エミュレータ開発を競争ではなく、チームワークとして捉えてもらえるようになることです。誰が最初にやったとか、誰が一番うまくやったとか、そういうことではありません。それは、ゲームスタジオの作品が生き続けられるようにすることであり、ひ孫たちが望めば、ビデオゲームがどのように始まったかを振り返ることができるようにすることです。

エミュレータ開発では、誰もが神でも王でもありません。誰も他の人よりも重要ではありません。私たちは皆、一丸となっているのです。

繰り返しになりますが、既存の知識に頼る必要はありません。自分の道を切り開くのは自由です。しかし、先人たちの肩の上に立つことを恥じるべきではありません。

読んでいただきありがとうございました。これが、皆さんが何かを始めるきっかけになれば幸いです。皆さんがどんな作品を作るのか、楽しみにしています。頑張ってください。そして、私たちはあなたのためにここにいることを忘れないでください。