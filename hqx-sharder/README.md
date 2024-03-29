# HQx-shader

>**Note**  
> このドキュメントは [HQx-shader](https://github.com/CrossVR/hqx-shader/blob/53540f5f0d985c385dc108b41ab89980f2b214f4/README.md) のREADMEを日本語に翻訳したものです。

ドット絵を高画質化(アップスケール)するフィルターHQxをCgシェーダーを用いて実装したものです。

## 使い方

RetroArchなど、アップスケールに対応したエミュレータで、希望するアップスケールのプリセットファイルを読み込みます。

## エミュレータへの組み込み

このシェーダのサポートをエミュレータに実装したい場合は、ゲーム画面のテクスチャの他に、シェーダ用の追加テクスチャのロードをサポートする必要があります。ロードする必要のあるテクスチャは、`resources`ディレクトリにあります。

`resource`内の追加テクスチャのサポートを実装した後は、`single-pass/shader-files`ディレクトリにあるシングルパスシェーダを統合することができます。

さらに、パフォーマンスを向上させるために、マルチパスシェーダをサポートすることができます。このシェーダは冗長な計算を減らし、シングルパス版よりもはるかに高速に動作します。マルチパスシェーダは`shader-files`ディレクトリにあり、デフォルトではプリセットファイルで使用されます。

## 実装

このシェーダは、オリジナルのHQxフィルタと同様に、ルックアップテーブルを必要とします。HQxでは、このルックアップテーブルには、加重平均によるピクセルの補間に使われる重みが含まれています。オリジナルのアルゴリズムの詳細については、[ドキュメント](algorithm.md)をご覧ください。

HQxアルゴリズムは、元のピクセル（w5）をその8つの近傍ピクセルで補間します。

```
     +----+----+----+
     |    |    |    |
     | w1 | w2 | w3 |
     +----+----+----+
     |    |    |    |
     | w4 | w5 | w6 |
     +----+----+----+
     |    |    |    |
     | w7 | w8 | w9 |
     +----+----+----+
```

すべてのピクセルはYUV色空間で比較され、検出されたパターンの重みがルックアップテクスチャから取得されます。補間に使用したいピクセルも、この重みによって選択されます。（他のピクセルの重みはゼロに設定されます）

ルックアップテクスチャの各エントリには、重みを格納するための4つのコンポーネントがあります。重みは、単純な行列とベクトルの乗算によって適用されます。この行列に含まれるピクセルは、元のピクセルに対してどのアップスケールピクセルを処理しているかによって異なります。例えば、元のピクセルに対して左上にある場合は、w1、w2、w4、w5のピクセルを補間するだけで済みます。

## クレジット

- このレポジトリ: LGPL-2.1
- オリジナルのC実装: Maxim Stepin氏、Cameron Zemek氏
- このレポジトリの実装: Hyllian氏(xBRの作者)が手助けしてくれました
- その他サポート: Hunter K.氏

