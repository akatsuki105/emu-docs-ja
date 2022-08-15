# IPSファイル

<pre>
Note:

アドレスと長さはビッグエンディアンで格納されます。
</pre>

## フォーマット

```rust
struct IPS {
    // ASCII文字列"PATCH"
    magic: String;

    // 任意の数の、`Record` and/or RLEレコード`RLERecord`(後述)
    // RecordKind = Record | RLERecord
    records: Vec<RecordKind>;

    // ASCII文字列"EOF"
    eof: String;

    // Truncateされたデータ(後述)
    truncate: Vec<u8>;
}
```

## `Record`フォーマット

```rust
struct Record {
    // 3バイト、パッチ対象の開始オフセットを表す
    offset: [u8; 3];

    // 2バイト、パッチデータの長さを示す
    length: [u8; 2];

    // パッチデータ(サイズは`.length`バイト)
    // `.offset`の示すオフセットから`.length`バイトだけ書き込まれる
    data: Vec<u8>;
}
```

## `RLERecord`フォーマット

RLEレコードは、1つのバイトを複数回書き込みます。

```rust
struct RLERecord {
    // 3バイト、パッチ対象の開始オフセットを表す
    offset: [u8; 3];

    // 2バイト、両方とも0で、RLERecordであることを示唆するためのマジックナンバー
    magic: [u8; 2]; // [0, 0]

    // 2バイト、パッチデータの長さを示す
    length: [u8; 2];

    // パッチデータ
    // offsetからlengthバイトバイトだけ繰り返し書き込まれる
    data: u8;
}
```

## Truncate

SNESToolや他のIPSパッチャー(Lunar IPS, NINJAなど)では、ファイル切り捨て機能をサポートしています。

この機能は、EOFの後に3バイトのオフセットを格納する形で、IPSフォーマットを拡張したものです。この機能をサポートしているIPSパッチャーは、パッチされたファイルを切り捨て、オフセットの後のデータをすべて削除します。

## 参考記事

- [IPS file format](http://www.smwiki.net/wiki/IPS_file_format)
