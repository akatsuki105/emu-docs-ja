# Core info

https://github.com/libretro/libretro-core-info/blob/c4c5b5d8fc094239c7f52f095d99f2dbff2bec03/00_example_libretro.info

```yaml
# すべてのパラメータは任意ですが、ユーザー体験の向上に役立ちます。

# Software Information - Information about the core software itself

# ユーザーがコアを選択する際に表示される名前
display_name = "Nintendo - Game Boy (Core Name)"

# コアが属するカテゴリ（オプション）
categories = "Emulator"

# コアを書いた著者の名前
authors = "Martin Freij|R. Belmont|R. Danbrook"

# コアの名前(内部用)
corename = "Nestopia"

# コアがサポートする拡張子のリスト
supported_extensions = "nes|fds"

# コアのソースコードのライセンス
license = "GPLv3"

# コアを使用するために必要なプライバシー固有のパーミッション
permissions = ""

# コアのバージョン
display_version = "v0.2.97.30"

# Hardware Information - Information about the hardware the core supports (when applicable)

# エミュレートされたシステムを製造したメーカー名
manufacturer = "Nintendo"

# コアが対象とするシステムの名称(任意)
systemname = "Nintendo Entertainment System"

# コアが使用する主なプラットフォームのID。可能であれば、他のコア情報ファイルを参考にしてください。
# 空白または未使用の場合は、標準のコアプラットフォームが使用されます。
systemid = "nes"

# コアが必要とする 必須 or 任意 のファームウェアファイルの数です。
firmware_count = 7

# Firmware entries should be named from 0
# Firmware description
firmware0_desc = "filename (Description)" # ex: firmware0_desc = "bios.gg (GameGear BIOS)"
# Firmware path
firmware0_path = "filename.ext" # ex: firmware0_path = "bios.gg"
# Is firmware optional or not, if not defined RetroArch will assume it is required
firmware0_opt = "true/false"

# 追加の注意事項:
notes = "(!) hash|(!) game rom|(^) continue|[1] notes|[^] continue|[*] list"

# Libretro Features - コアがサポートしているlibretro APIの機能です。コアのソートに便利です。

# コアがセーブステートをサポートしているか
savestate = "true"
# `savestate`が`true`の場合に、セーブステート対応の完成度を表します。  basic, serialized (巻き戻しに必要), deterministic (ネットプレイ/先取りに必要)
savestate_features = "serialized"
# コアが libretro チートインターフェイスをサポートしていますか？
cheats = "false"
# コアが libretro の Input descripterをサポートしているか
input_descriptors = "true"
# コアが libretro の Memory descripterをサポートしているか(アチーブメントで利用)
memory_descriptors = "false"
# コアは libretro save インターフェースを使用しているのか、それとも独自のものを使用しているのかどうか。
libretro_saves = "true"
# コアオプションのインターフェースをサポートしていますか？
core_options = "true"
# コアオプションの対応バージョンは？(新しめのバージョンなら localization と descriptions に対応)
core_options_version = "1.0"
# コアがサブシステム(SGBのゲームボーイ)のインターフェースをサポートしているか
load_subsystem = "false"
# コアが動作するために外部ファイルを必要とするかどうか。
supports_no_game = "false"
# コアがサポートするデータベースの名前(任意)
database = "Nintendo - Nintendo Entertainment System|Nintendo - Famicom Disk System"
# フロントエンドでの`libretro-gl`やその他のハードウェアアクセラレーションのサポートを許可しているかどうか。
hw_render = "false"
# コアがサポートしているハードウェアレンダリングのAPIは？パイプ文字(|)で区切られています。
required_hw_api = "Vulkan >= 1.0 | Direct3D >= 10.0 | OpenGL Core >= 3.3 | OpenGL ES >= 3.0"
# 読み込み後、コアがファイルへの継続的なアクセスを必要とするか？ 主にソフトパッチやデータのストリーミングに使用されます。
needs_fullpath = "false"
# コアは、その場でディスクを交換するためのlibretroディスク制御インターフェースをサポートしていますか？
disk_control = "false"
# そのコアは現在、一般的な使用に適していますか？ つまり、一般ユーザーが便利に使えるのか、それとも開発専用なのか？
is_experimental = "true"

# 説明用テキストで、ツールチップに便利です。
description = "This is a brief description of the core. It should be informative, but probably not super-long. 1024 characters, tops, all on one line (i.e., no manual line-breaks)."
```
