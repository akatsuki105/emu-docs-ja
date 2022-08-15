# ビデオ録画

## 3rdパーティの録画ソフトウェア

多くの場合、このようなプログラムを使ってゲームプレイを録画するのが一番良いし、簡単だと思います。

- Bandicam (must use version 1.8.2 or higher if you want lossless recording via third-party VFW codecs)
- DxTory (¥3600)
- Fraps
- MSI Afterburner (freeware)
- Open Broadcaster Software (Local Recording mode) (free software)
    - なお、OBSでは、ロスレス映像を生成する設定やロスレスコーデックを使用しても、ロスレス映像を得ることはできません。録画された映像は、処理される前からロスが発生しています。
- ShadowPlay (Nvidia exclusive) (freeware)
- CamStudio (Open Source)
- Camtasia

## General Recording

Snes9x や Nestopia などのエミュレータでは、Video for Windows（VFW）フレームワークを利用して、映像を録画する際に特定のビデオコーデックを選択できるようになっています。

オプションには、Lagarithのようなロスレスコーデックで録画するものもあり、非圧縮の場合よりもファイルサイズを大幅に削減することができます。

これらのエミュレータは、オーディオを非圧縮PCMで記録し、記録が終わったら両方のストリームをAVIファイルにミックスします。

## RetroArch

RetroArchはロスレスのRGB x264とFLACオーディオで録画しますが、高解像度のビデオの場合はディスクの容量や処理能力をかなり消費します。

録画オプションは現在RGUIには実装されていませんので、Phoenixを使うかCLIを使う必要があります。MKV形式での録画をお勧めします。

コマンドラインで次のように入力してください。

```sh
> retroarch.exe -r "C:\Path_to_Recordings\recording_name.mkv" --menu
```

デフォルトでは、RAはネイティブな解像度で録画しますが、より高い解像度で録画するには、`--size`を使用し、`WIDTHxHEIGHT`を指定します。また、`--recordconfig`を設定することで、録画時に特定の設定ファイルを使用することができます。デフォルトでは、フィルターのみを記録します。

シェーダーを使って録画するには、設定ファイルに `video_gpu_record = true` を追加してください。解像度やシェーダーによっては、パフォーマンスが大幅に低下することがあります。([動画例](https://youtu.be/jflrB0UIcFE))

## PCSX2

PCSX2でビデオを録画するには、GSdx videoプラグインを使用している状態で、ゲームプレイ中にF12を押すだけです。

設定可能なウィンドウが表示されます。オーディオについては、SPU2-X の設定で `Enable Debug Options` をチェックし、`Debugging options` で `Dump Memory Contents` をチェックします。

## Dolphin

かつてはDolphin経由での録画は推奨されていませんでした。A/Vシンク用に編集された特定のビルドが必要になり、Dual CoreやIdle Skipping（同期ずれの原因）が使えなかったからです。

これらの問題は解決されており、Dolphinは2017年からffmpegのダンプを内蔵しています。

また、ロスレスでダンプしたい場合は、グラフィックス設定ダイアログボックスの最後のタブにある`Frame Dumps use FFV1`にチェックを入れることをお勧めします。

