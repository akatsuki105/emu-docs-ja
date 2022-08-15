# Libretorコアの開発

## 🖲 Libretro API

Libretro APIは、汎用のオーディオ、ビデオ、入力コールバックを公開する、軽量なCベースのアプリケーション・プログラミング・インターフェース（API）です。スタンドアロンゲーム、ゲームエミュレータ、メディアプレイヤーなどの「コア」の開発者は、Direct3DやOpenGL用の異なるビデオドライバを書いたり、可能なすべての入力API、サウンドAPI、ゲームパッドなどに対応することを心配する必要はありません。

Libretro APIの利用を選択すると、あなたのプログラムは1つのライブラリファイル（「libretro core」と呼ばれます）になります。Libretro APIをサポートするフロントエンドは、そのライブラリファイルを読み込んでアプリを実行することができます。フロントエンドの責任は、実装特有の詳細を提供することです。libretro coreの役割は、メインプログラムを提供することだけです。

このAPIで動作するように移植されたプロジェクトは、今も昔も、どのlibretroフロントエンドでも動作させることができます。メインプログラムだけを扱う単一のコードベースを維持し、単一のAPI（libretro）をターゲットにして、プログラムを一度に複数のプラットフォームに移植することができます。移植性の高いCやC++で書かれたlibretro coreは、ほとんど、あるいは全く移植の手間をかけずに、多くのプラットフォーム上でシームレスに動作します。他の言語のためのlibretroバインディングも、ますます一般的で包括的になってきています。

注意: Libretroは、ライセンス料や縛りがなく、100%無料で実装できるオープンな仕様となっています。私たちのリファレンスフロントエンドは RetroArch です。この2つのプロジェクトは同じものではなく、これはライセンスに反映されています。RetroArch は GPLv3 でライセンスされていますが、libretro API は MIT ライセンスの API です。

## 📚 コア開発のためのリソース一覧

### `libretro.h`

`libretro.h`は、libretroのコアやフロントエンドの開発者にとって、最も重要な技術的リファレンスです。

`libretro.h`の最新の標準的なコピーは、`libretro-common`リポジトリの masterブランチにあります。

### サンプルコア実装の`skeletor`

RetroArchのコントリビュータである bparker06氏は、最小限のlibretroコアの実装として`skeletor`を作成しました。

`skeletor`は、スタブのlibretro `Makefile`と`Makefile.common`ファイルも提供しています。

### `Vectrexia` codebase and development log

beardypig氏は、libretroのために一から設計されたオリジナルのエミュレータコアである`Vectrexia`の作成の一環として、`libretro.h`の実装プロセスを説明する2部構成のガイドを公開しました。([Part 1](https://web.archive.org/web/20190219134430/http://www.beardypig.com/2016/01/15/emulator-build-along-1/), [Part 2](https://web.archive.org/web/20190219134028/http://www.beardypig.com/2016/01/22/emulator-build-along-2/))

### `libretro-common`

[`libretro-common`](https://github.com/libretro/libretro-common/) は、libretro のコアやフロントエンドの開発に役立つ、クロスプラットフォームの必須コーディングブロックを集めたもので、主にC言語で書かれています。

### `libretro-deps`

[`libretro-deps`](https://github.com/libretro/libretro-deps/) は、libretro コアで使用するために事前に変更されたサードパーティの依存関係のコレクションです。

### `libretro-samples`

[`libretro-samples`](https://github.com/libretro/libretro-samples) は、libretro APIの実装例です。

### OpenGLによってハードウェアアクセラレートされたコアの実装について

[OpenGLによってハードウェアアクセラレートされたコアの実装ガイド](https://github.com/libretro/docs/blob/952b6f0343995e6910b7840b0d415c508f366c9a/docs/development/cores/opengl-cores.md)が利用可能です。

## 🛠 Libretro APIの実装

[こちら](implementing-the-api.md)を参照

