# 決定論的ネットコード

>**Note**  
> この記事は [Preparing your game for deterministic netcode](https://yal.cc/preparing-your-game-for-deterministic-netcode/) を翻訳したものです。

>**Note**  
> ネットコード: ネットプレイ(インターネットを使ったマルチプレイ)を実現するプログラム

決定論的ネットコードとは、**各ゲームクライアントが、フレームごとに同じ初期状態と入力を与えられたときに、同じ状態になること**を意味します。

一般的な決定論的ネットコードには、ロックステップやロールバックなどがあります。

## ロックステップ

ロックステップは決定論的プロトコルの比較的シンプルな実装で、各プレイヤーの入力/アクションがそのフレームで判明した場合にのみ、次のゲームフレームが処理されるようになっています。

ロックステップベースのネットプレイでは、リモート入力が時間通りに届くように、RTTの中央値の半分の入力遅延を加える必要があります。

また、フレームに対するすべてのリモート入力がわかるまでゲームを停止する必要があるため、モバイルやNintendo Switchなどの接続が不安定なデバイスでプレイするゲームには最適ではありません。

## ロールバック

ロールバック は、ロックステップ を改良したもので、遠隔地(相手プレイヤー)からの入力が間に合わなかった場合に、プレイヤーが入力を推測します。その後、入力が判明した時点でゲームを巻き戻し、修正した入力でゲームフレームを再実行します。

つまり、100msの長さの接続障害が発生した場合、リモートプレイヤーはその6フレームほどの間、すでに保持していた入力を保持し続け、実際の入力が到着した時点で修正し、視覚的にはキャラクターが新しい場所に一時的に適応するだけであると仮定することができます。

もちろん、予測されるフレーム数が多ければ多いほど、状態が修正された後に何かが全く違う方向に進んでしまう可能性が高くなります。数フレームならまだしも、1秒分の入力を予測すると、相手のキャラクターがまったく別の場所に移動してしまう可能性が高くなります。そのため、対戦ゲームでは、ゲームがロックステップのような失速をするまでの予測フレーム数に上限を設ける傾向があります。

## メリット

- 低速なネット環境でも問題なく動作
    - 入力データを送るだけだからです。
- プレイヤー間の有利不利が比較的小さい
    - ホストが他のプレーヤーより優位に立つことはなく、ホストに近いプレーヤーも優位に立つことはありません。
- ゲームプログラムとネットコードが分離できる
    - つまり、例えば格闘ゲームに追加の技を加える場合、通常はネットコードを触る必要がないため、ゲーム開発とネットコードを別々の人が担当するチームでは、やりとりの回数を減らすことができます。

## デメリット

- 入力遅延
    ロールバックで解決できますが、遅延をRTTの半分の中央値よりも低く設定すると、リモートのプレイヤーが一貫して予測を誤るため、常にグリッチが発生してしまいます。それはともかく、多くの人は入力遅延よりもそちらを好みます。
- スケーラビリティ
    - 各プレイヤーは、自分の入力を他のプレイヤーに送信する必要があります。言うまでもなく、接続数が多ければ多いほど、特定のペアのプレーヤーがお互いに接続不良となり、他のプレーヤーに問題を引き起こす確率が高くなります。
    - 一般的に決定論的ネットコードは、4人以上のプレイヤーがいるゲームでは、メッシュトポロジーと組み合わせて使用することはありません。入力遅延が少ないゲーム（RTSゲームなど）では、スター型トポロジー（全員がホストに接続）を使用することができます。
- チート
    - 各プレイヤーはゲームの状態を常に把握しているので、見えてはいけない情報を見てしまうこともあります。
    - テンポの速いゲームでは、それはあまり気になりません。例えば、格闘ゲームでは、自分と相手の見え方に違いがあることはほとんどなく、推測できる余分な情報は、多くのプレイヤーがすでに暗記しているヒットボックスやクールダウンだけです。
- 同期ずれ
    - もしネットコードが決定論的でない場合は問題が生じます。例えば、グローバル変数が設定値に達したときにインスタンスを生成し、セッション開始時にリセットすることを忘れています。これにより、プレイヤーが同じゲーム状態を見ることができなくなり、ステートのずれが発生する可能性があります。
    - ゲームの状態を再同期させようとすると、計算コストがかかり、かなりの帯域幅を必要とします。また、ゲームが再び同期ずれを起こさないとは限らず、ゲームの状態を繰り返し再同期するという死のループに陥る可能性があります。
    - 同期ずれのデバッグは、決定論的ネットコードを扱う上でより複雑な部分の一つであり、ゲームステートダンプ/ログの比較が必要です。

## 注意

### ネット環境の重要性

パケットが遅れると、ストール（ロックステップ）や映像の不具合（ロールバック）が発生するため、プレイヤー間の接続をできるだけ良好に保つことが何よりも重要になります。

パケットロスを軽減する技術は必須であり、最近のゲームでは、プライベートネットワークを介してトラフィックをルーティングする傾向が強まっています。これは通常のP2Pよりもレイテンシや安定性が向上する場合があります

### ツール特有の問題

ゲームエンジンやフレームワークの中には、決定論的性質よりもパフォーマンスを優先するコンポーネントが組み込まれているため、本質的に決定論的ネットコードに適していないものがありますが、これはソースコードにアクセスできたとしても、簡単に対処できる問題ではありません。

例えば、GameMakerには比較のためのイプシロンが組み込まれており（近い数は等しいとみなされる）、また、コリジョンチェック関数が歴史的経緯から座標を丸めるように組み込まれているため、小さな浮動小数点のエラーは全く気づかれないという点で、この点に関しては比較的良い状況にあります。

一方、Unity は決定論的ネットコードには適していません。というのも、ビルトインAPI のほとんど（物理演算を含む）が決定論的ではなく、決定論を求める場合は、そのかなりの部分を固定小数点構造体で再実装することになるからです。

### 可変フレームレート

ゲームロジックはプレイヤー間で同じペースで進行する必要があり、ビジュアルは補間/外挿する必要があるため、様々な画面のリフレッシュレートでゲームを動作させることは、決定論的ネットコードではより困難です。

## どのようなゲームが決定論的ネットコードを必要とするのか？

### ゲームスピードの速い対戦ゲーム

例えば、格闘ゲームやプラットフォーマーゲームでは、ほとんどのゲームでロールバックやロックステップのネットコードが採用されています。基本的にはロールバックが推奨されます。

### ゲームスピードの速い協力型ゲーム

一般的に、厳密な協力プレイを行うゲームであれば、従来のクライアント・サーバーモデルを採用し、可能な限りプレイヤーを優遇することができますが、協力モードと対戦モードの両方を備えたゲームや、高い精度が要求されるゲームでは、ロールバックネットコードを利用することができます。

最近の例では、「Spelunky 2」が最も有名ですが、ロールバックネットコードは、高予算のベルトスクロールアクションゲームにも見られます。

### RTSゲーム

ゲーム内で何百ものユニットが動き回っている場合、それらのユニットに関する情報を効果的に同期させることは困難であり、これがRTSゲームが歴史的にロックステップに傾いてきた理由です。

2021年現在、インターネットの速度の中央値は一般的に十分な速度であり、多くのRTSゲームはロックステップの代わりにクライアント・サーバーモデルを使用することができるようになりました。クライアント・サーバーモデルの導入はいくつかの不正行為の解決策にもなりました。

### エミュレータ

各ゲームのROMにネットワークロジックを組み込むことは、一般的には不可能です。また、エミュレータは本質的に決定論的であるため、決定論的ネットコードに適しています。

## ネットコード実装の準備

プレイの快適さに応じて変わってきます。

### Tier 0: 前準備

これらは、ネットコードを実装するかどうかわからなくても、やっておいたほうがいいと思います。

- ゲームにオンラインのマルチプレイを搭載する場合は、何らかの形でローカルネットワーク上のマルチプレイを動作させる必要があります。ゲームのあらゆる部分をマルチプレイに対応させるには時間がかかり、大作ゲームでは極端な場合、ネットコードを実装するよりもゲームを一から作り直したほうが安くつくこともあります。
- データの保存先と読み込み先、ゲームの状態に影響を与えるものなどを記録しておきましょう。例えば、プレイヤーが何かをアンロックした後にのみ、レベル上の特定のエンティティが出現する場合、マルチプレイヤーではその事実を同期させる必要があります。

### Tier 1: デスクトップPC上のロックステップ

必要なのは

- Tier0の前準備を終えている
- 入力ポーリングを`button_check(player_index, button_index)`に抽象化するか、あるいは各入力の状態を示す変数をどこかに割り当てるかして、一箇所にまとめるようにします。

具体的な例としては、ゲームにリプレイシステムを導入してみましょう。

リプレイとは、初期状態（ゲームプレイに影響を与える設定やアンロックなど）を含むファイルであり、マッチ/セッション開始以降のフレームごとのプレイヤーの入力内容を含みます。

リプレイでは、初期状態を適用し、デバイスをポーリングするのではなく、各フレームの入力をファイルから取得することで、ゲームを再生することができます。

同期ずれをせずにリプレイを動作させることができれば、それでいいと思います。

### Tier 2: Web/モバイル/ゲーム機上のロックステップ

ロックステップなのはTier1と同じですが、次の違いがあります。

デスクトッププラットフォームでは、ネットワークAPIは一般的に同期バージョンの関数を持っています。つまり、ゲームを一時的に停止させる必要がある場合は、データが利用可能になるまでソケット/APIを繰り返しポーリングすることで可能になります。

一方、他のプラットフォームでは、同期ポーリングは推奨されない場合もあれば、不可能な場合もあります。（特にHTML5では不可能です）

なので、ここでは以下のことが必要になってきます。

- Tier0とTier1を実装済み
- 実際のフレームに対して、任意の数（ゼロを含む）のゲームロジックのフレームを処理できるようにします。
    - これは通常、ゲームロジックのコードを別の場所に移動して、必要に応じて簡単に呼び出せるようにすることで実現できます。例えば、GameMakerでは`Step`イベントのコードを`User Event`に移動したり、Unityでは`Update/FixedUpdate`を自分で定義した関数に移動したりします。
    - また、あなたが使っているゲームエンジンで自動的に処理されるロジックも処理しなければならないことに注意してください。

Tier2の内容が実装できると、具体的には、従来のリプレイシステムに一時停止や早送り（2倍速）の機能を実装することができるようになります。

### Tier 3: ロールバック

必要なのは

- Tier2までを全て実装済み
- オンデマンドなゲームステートのセーブ・ロードを実装
    - これは、ゲームステート（ゲームプレイに影響を与えるすべてのもの）全体をシリアライズ／デシリアライズして、後から読めるようなフォーマットにしなければなりません。慣例的にはバイナリ形式のシリアライズですが、技術的には十分な速度が出れば何をしても大丈夫です。
    - 実装の難しさは、ゲーム本体の数、データの量、使えるツールの種類などによって、ゲームやエンジンごとに大きく異なります。

具体的な例としては、先に作られたリプレイシステムに、あるタイミングでのセーブ/ロード機能を実装する必要があります。

セーブとは、ゲームの状態と現在のファイルの読み取りタイミングを保存することです。

ロードとは、ゲームの状態を読み込み、ファイルの読み取りタイミングを先に保存したものにリセットすることです。

これらにより効果的にリプレイを巻き戻すことができます。

## どのくらいの時期にネットコードを実装すべきか？

理想的には早ければ早いほどいいのですが、ゲームの制作には時間がかかりますし、ゲームのコードベースは開発中に大きく変化することがありますので、ネットコードの準備はできていても、実際にネットコードに着手するのはゲームの機能が完成に近づいてからということもよくあります。

初めてネットコードを実装する場合は、問題が発生する可能性があるため、最低でも数ヶ月の余裕を持ってテストとデバッグを行うようにしてください。


