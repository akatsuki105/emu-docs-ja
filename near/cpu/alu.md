# ALU

この記事で紹介するのは、Z80系のCPUで使われる、足し算，引き算のオーバーフロー，キャリー，ハーフキャリーのフラグを，分岐を使わずに計算するアルゴリズムです。

結果の計算に余分なビットを必要としないように設計されているので、より大きなサイズの型にテンプレート化することができ、あるマシンが使用できる最大の型でも使用することができます。

もちろん、これらをテンプレート化するには、オーバーフローとキャリーマスクのために、与えられた整数型の最上位ビットを抽出するポータブルな方法が必要になります。

事前準備として最上位bitのマスク変数が必要です。次のコードをみてください。

```c++
natural sign = (natural(0) - 1 >> 1) + 1;
// natural == uint8: sign -> 0b1000_0000
```

上のコードについて、型`natural`は`uint1`から`uintmax`までの任意の型のどれかです。これで最上位bitが1の整数を得ることができます。

## ADC: add with carry

```c++
auto adc(natural target, natural source, boolean carry) -> uint8 {
  natural result   = target + source + carry;
  natural carries  = target ^ source ^ result;
  natural overflow = (target ^ result) & (source ^ result);

  flag.overflow  = overflow & sign;
  flag.carry     = (carries ^ overflow) & sign;
  flag.halfCarry = carries & 16;  //Z80

  return result;
}
```

## SBB: subtract with borrow

ほとんどすべてのプロセッサは、減算をボローで実装しています。一部のCPUベンダーはこれを混同し、別の演算であるにもかかわらずSBC（subtract with carry）と名付けています。東芝のTLCS900Hがその例です。

```c++
auto sbb(natural target, natural source, boolean carry) -> uint8 {
  natural result   = target - source - carry;
  natural carries  = target ^ source ^ result;
  natural overflow = (target ^ result) & (source ^ target);

  flag.overflow  = overflow & sign;
  flag.carry     = (carries ^ overflow) & sign;
  flag.halfCarry = carries & 16;  //Z80

  return result;
}
```

**SBB from ADC**

```c++
auto sbb(natural target, natural source, boolean carry) -> uint8 {
  natural result = adc(target, ~source, !carry);

  flag.carry     = !flag.carry;
  flag.halfCarry = !flag.halfCarry;

  return result;
}
```

## SBC: subtract with carry

6502シリーズのプロセッサ（MOS6502、リコー6502、HuC6280、WDC65816）では、減算時にBorrowではなくCarryを使用しています。

```c++
auto sbc(natural target, natural source, boolean carry) -> uint8 {
  natural result   = target - source - !carry;
  natural carries  = target ^ ~source ^ result;
  natural overflow = (target ^ result) & (source ^ target);

  flag.overflow  = overflow & sign;
  flag.carry     = (carries ^ overflow) & sign;
  flag.halfCarry = carries & 16;  //Z80

  return result;
}
```

**SBC from ADC**

Carryを使用する理由は、次のもう1つの実装を見るとよくわかります。

```c++
auto sbc(natural target, natural source, boolean carry) -> uint8 {
  return adc(target, ~source, carry);
}
```

6502はトランジスタ数が3500と少ないため、SBCのADC回路を再利用できるのは有利です。SBBでも同じことができますが、フラグを反転させる必要があるため、より複雑になります。

## 謝辞

分岐のないキャリー計算を実現するためのヒントを与えてくれたTalarubi氏に感謝します。
