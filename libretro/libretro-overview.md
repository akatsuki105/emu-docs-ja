# Libretroを用いた開発について

Libretro APIは、汎用的なオーディオ、ビデオ、入力のコールバックを公開する、軽量なCベースのアプリケーション・プログラミング・インターフェース（API）です。

## フロントエンド と コア

Libretroの開発にあたって、考慮すべき2つの側面があります。

- フロントエンド: libretro互換のコアを動作させることができるプログラムです。
- コア: ゲーム、エミュレータ、メディアプレイヤーなどのプログラムで、libretroのフロントエンドで実行できるようにlibretro APIを実装したものです。

スタンドアロンゲーム、ゲームエミュレータ、メディアプレイヤーなどのコアの開発者は、Direct3DやOpenGL用の異なるビデオドライバを書いたり、可能なすべての入力API、サウンドAPI、ゲームパッドなどに対応することを心配する必要はありません。

コアは1つのライブラリファイルとして構築され、libretro APIをサポートするあらゆるフロントエンドで実行することができます。フロントエンドの責任は、すべての実装固有の詳細を提供することです。コアの役割はメインプログラムを提供することだけです。

## `libretro.h`

libretro APIは、RetroArchのソースパッケージに含まれる`libretro.h`に記載されているいくつかの関数で構成されています。

APIのヘッダはC99とC++に対応しています。C99では、bool型と`<stdint.h>`が使われています。このファイルの最新版は [`libretro-common`](https://github.com/libretro/RetroArch/blob/master/libretro-common/include/libretro.h) にあります。

## コア開発に便利なリソース

libretroの[githubリポジトリ](http://github.com/libretro/)の一部として管理されているコアの部分的なリストは `For Users > Core Documentation`セクション で見ることができます。

コア開発の概要については[こちら](cores/developing-cores.md)を参照してください。

## フロントエンド開発に便利なリソース

libretroのフロントエンドの数は増え続けており、様々なホストシステムやユースケースに対応しています。

RetroArch は libretro のフロントエンドの参考例と言っていいほどの完成度で、さまざまなホストプラットフォームで利用できます。RetroARchの開発については、`For Developers > RetroArch Development`のセクションで詳しく説明しています。

## Libretro上で動作するOS

LibreELECをベースにした[`Lakka`](http://www.lakka.tv/)は、Libretroを使ったOSディストリビューションのいいリファレンスと言えるでしょう。

また、以下は、バックエンド技術の一部として libretro または RetroArch を使用している外部ディストリビューションの一部です。

- [batocera.linux](http://batocera-linux.xorhub.com/)
- [RetroPie](http://retropie.org.uk/)
- [Recalbox](http://recalbox.com/)

注意: Libretroは、ライセンス料や縛りがなく、100%無料で実装できるオープンな仕様となっています。私たちのリファレンスフロントエンドは RetroArch です。この2つのプロジェクトは同じものではなく、これはライセンスに反映されています。RetroArch は GPLv3 でライセンスされていますが、libretro API は MIT ライセンスの API です。

