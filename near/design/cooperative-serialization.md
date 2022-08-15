# 協調型シリアライズ

[前回の協調型スレッディングに関する記事](cooperative-threading.md)では、コンポーネントを協調スレッドとしてエミュレートすることの利点について説明しました。

このアプローチの最大の問題点は、シリアル化(シリアライズ)、つまりエミュレーションで言うところのセーブステートです。

今回の記事では、この問題をより詳細に検討し、解決策を提案します。

## Recap

シリアル化が難しいのは、関数内での位置関係を維持するステートマシン変数が、各スレッドが使用するネイティブスタックに移動しているからです。

このスタックデータは極めて移植性が低く、仮にディスクからスタックの保存と読み込みを行ったとしても、プログラムの複数回の実行で特定の予約メモリアドレスを取得できる保証はありません。また、仮にできたとしても、ASLRの意味が薄れてしまいます。

しかし、使える方法はいくつかあり、実際には、1つの方法だけではなく、それぞれの方法を組み合わせて使うのが最も効果的な傾向にあります。

## 例

この記事では、シンプルなベアボーンの協調型スレッドCPUコアを定義しましょう。

```c++
void CPU::main() {
  if(interruptPending) interrupt();
  instruction();
}

void CPU::instruction() {
  auto opcode = fetch();
  if(opcode == 0xa9) return opcodeLDAconst();
}

void CPU::opcodeLDAconst() {
  A = fetch();
  N = A & 0x80;
  Z = Z == 0;
}

void CPU::fetch() {
  step(2);
  auto data = bus.read(PC++);
  step(4);
  return data;
}

void CPU::step(uint clocks) {
  apu.clock -= clocks;
  while(apu.clock < 0) scheduler.switch(apu.thread);
}
```

この例では、CPUコアはスタックフレームの4つの関数を終了した後、APUコアに実行を譲っています。ここで、APUコアがしばらく動作した後、セーブステートを取得する必要があると仮定します。

残念ながら、今はCPUの命令を実行している最中です。単純にCPUスレッドを再作成すると、`CPU::main()`の開始時に実行を開始することになりますが、これは全く望んでいないことです。

## Thread Alignment

上記のコードを次のように変更することもできます。

```c++
void CPU::main() {
  scheduler.leave(Scheduler::Serialize);
  if(interruptPending) interrupt();
  instruction();
}
```

つまり、すべてのスレッドがmain関数の開始時に終了するまで、エミュレーションを実行し続けるという考え方です。

問題は、エミュレータの複雑さが増すにつれて、これがすぐに崩れてしまうことです。スレッド数が2～3個しかない場合でも、エミュレータがすべてのスレッドが完全に整列した状態にならないことがすぐにわかります。

## Method #1: 高速同期

1つ目の方法は、シリアル化のためにすべてのスレッドを同期（整列）させようとしている間、スケジューラが他のスレッドに切り替わるのを単純に止めることです。

```c++
void CPU::step(uint clocks) {
  apu.clock -= clocks;
  while(apu.clock < 0 && !scheduler.synchronizing()) scheduler.switch(apu.thread);
}
```

しかし、上のコードでは、CPUがAPUより先に進んでいても、`bus.read(PC)`を呼び出すことができ、APUはPCが指すアドレスに書き込むことができるため、決定論が破られています。言い換えれば、エミュレートされたコンポーネントが非同期になるということです。

CPUコードがAPUと同期していないのは、せいぜいCPU命令1つの分です。しかし、そのわずかな量でも、最も繊細なゲームの一部を壊すことができるのです。

## アウトオブオーダー実行

また、協調スレッドの最も優れた使用例の一つである「アウトオブオーダー実行」を使い始めると、より深刻な問題が発生します。

上記のコードでは、ステートマシンのように、時間が経過するたびにCPUからAPUへと常に切り替えています。

しかし、CPUが変更できないROMや、APUがアクセスできないメモリ範囲から読み込んでいることがわかっていたらどうでしょう？ つまり、APUはCPUがバスから読み込んだバイトを変更することはできません。その場合、上のコードを次のように変更することができます。

```c++
void CPU::fetch() {
  step(2);
  auto data = bus.read(PC++);
  step(4);
  return data;
}

void CPU::step(uint clocks) {
  apu.clock -= clocks;
}

uint8_t Bus::read(uint16_t address) {
  //this is ROM, it cannot be changed
  if(address < 0x8000) return rom.read(address);
  //this is internal RAM, the APU cannot access it
  if(address < 0xc000) return iram.read(address - 0x8000);
  //this is external RAM, the APU can modify it
  while(apu.clock < 0 && !scheduler.synchronizing()) scheduler.switch(apu.thread);
}
```

これは、CPUがAPUと共有しているRAMから命令を実行する可能性が低いため、結果的に非常に大きなスピードアップとなります。しかし、これは、CPUがAPUと長い間同期していないため、CPUがAPUよりも数百または数千命令進んでいる可能性があることを意味します。

(実際には、上記のコード例では不十分で、チップが通信しない場合にAPUがデッドロックするのを防ぐために、最終的にAPUに強制的に切り替えなければなりませんが、例示のためにここでは省略しています。)

つまり、CPUに1回の読み込みを完了させると、まれにAPUよりも数千命令も先に読み込まれてしまうことがあり、これははるかに重大な同期違反となります。このように、方法1（高速同期）は、間違いなくいくつかのゲームを壊し始めるでしょう。

いずれにしても、先に進む前に、高速なシリアル化がどのようなコードになるかを見てみましょう。

```c++
void System::run() {
  scheduler.setMode(Scheduler::Running);
  //resume whichever thread was last active at last scheduler.leave() call
  scheduler.switch(scheduler.active->thread);
}

void System::serializeFast() {
  scheduler.setMode(Scheduler::Synchronizing);
  scheduler.switch(cpu.thread);
  scheduler.switch(apu.thread);
  //all threads are now paused at the start of their main() functions.
  //we can safely serialize the system now and ignore their stacks.
  //when unserializing, we simply recreate new, empty stack frames.
}
```

## Method #2: 厳密同期

TODO

## 決定論

厳密同期にはもう一つの問題があります。それは処理の完了までに潜在的に多くの時間がかかることです。

システムをシリアル化する際、理想的には時間を全く進めたくありません。時間を進めると決定性が損なわれます。

コントローラの入力をあらかじめ記録しておき、それを再現する場合を考えてみましょう。つまり、以前にプレイしたゲームを録画したムービーを再生しているのです。

1回目は普通にムービーを再生し、2回目は途中で状態を保存するように指示しました。この小さな動作が、もし高速同期法を使ってしまうと、ごくわずかな非同期化を引き起こし、ドミノ倒しのようにムービーの同期が失われてしまう可能性があります。

リアルタイム巻き戻しのようなものを実装したい場合、巻き戻しのための履歴バッファを持つために、システムを常にシリアライズする必要があります。このような潜在的な非同期化は、どんどん増えていきます。

## Method #3: Hibernation (Unsynchronized)

TODO

## Putting It All Together

TODO

## 終わりに

TODO

