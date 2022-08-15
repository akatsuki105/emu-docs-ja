# Netplay(WIP)

<pre>
Note:

Netplayは、同じエミュレーションコアと同じコンテンツを実行している複数のRetroArchインスタンスを連続的に同期させることで、インターネット上で複数人プレイをエミュレートするRetroArchの機能です。

RetroArchのNetplayは、Nintendo64のような1つのゲーム機に複数人が集まってやるような複数人プレイにのみ対応しており、GBAのような複数のゲーム機を通信ケーブルでつないで行うような複数人プレイには対応していません。
</pre>

RetroArchでは、インターネット経由で2人以上のプレイヤーや観客を接続することができます。

RetroArchのネットプレイコードはリプレイをベースにしており、デフォルト設定では入力遅延のない信頼性の低いネットワークでのネットプレイを実現しています。

ネットプレイは最大16人のプレイヤーと任意数の観客に対応しています。

RetroArchのネットプレイは、次の3つの細かい制約があるものの、完璧な同期を保って動作することが保証されています。

1. コアが`deterministic`であること。
2. コアが操作する入力デバイスは、ゲームパッドとアナログスティックだけであること。
3. コアとロードされたコンテンツの両方が、ホストとクライアントで同一なものであること。

また、ネットプレイを適切に動作させるためには、コアがシリアライズに対応している必要があります。シリアライズに対応していないコアでは、ネットプレイは限定的にしか動作しません。シリアライズに対応することで、よりスムーズな体験ができるようになります。

RetroArchのネットプレイは、**ネットワークからの遅延入力を期待し、その遅延入力を巻き戻して再生することで、一貫した状態を得る**という仕組みになっています。

ある時点では、すべてのネットプレイクライアントは矛盾した状態にあるかもしれませんが、お互いに遅延データを受信すると、矛盾していた最後の時点に目に見えない形で巻き戻し、新しい入力でエミュレータを実行して、巻き戻す前の状態よりも正規の状態に近い新しい状態に遷移します。

どのフレームがどのフレームであるかについて双方が合意している限り、それぞれの入力イベントが常に正しいフレームで起こるので、両者の同期がずれることはありません。

## 仕様

ネットプレイのトランスポートプロトコルはTCPを使用しています。これは信頼性と順番通りのデータ転送が正しい動作のために必要だからです。

1台のネットプレイサーバが、複数のネットプレイクライアントからの接続を受けることがあります。

ほとんどの場合、サーバとクライアントは同等の参加者ですが、グローバルな同期を必要とする操作では、サーバが正規の参加者となります。

通常の動作では、ゲームをプレイ中の各クライアントは、フレームごとに自分のコントローラの入力データを送信するだけであり、サーバはクライアント間で入力データを転送します。

すべてのクライアントとサーバーは、どのプレイヤーのスロットが使用されているか（つまり、どのコントローラが接続されているか）を認識しており、サーバーはどのクライアントがどのプレイヤーのスロットに対応しているかを認識しています。

すべてのプレイヤーがフレームごとに入力データを順番どおりに送信することが重要であるため、すべてのクライアントはすべてのプレイヤーのフレームカウンタを保持しています。

予想外に低いフレーム数の入力データを受信した場合は無視され、予想外に高いフレーム数の入力データを受信した場合は接続を終了します。

また、すべてのプレイヤーがどのフレームがどのフレームであるかに同意することが重要です。

そのため、最初の接続時に、サーバーは正規のフレームカウントとシリアライズされたセーブステートをクライアントに与え、クライアントがストリームの途中で参加できるようにします。観戦者は入力データを送ることはありません。

プレイヤーから新しい入力データを受け取ったとき、それが現在実行中のフレームより前であれば、RetroArchは目立たないように巻き戻して新しい入力データを使って再実行し、元のフレームに戻ってくるので、ローカルプレイヤーの入力は常に無意識に行われます。

その他のイベントは、サーバのフレームカウンタに連動しており、ほとんどのイベントはサーバのみが行うことができます。例えば、クライアントが観戦からプレイに切り替える場合、単に入力データの送信を開始することはできず、モード変更のリクエストをサーバーに送信する必要があります。このとき、クライアントは、以前のフレームのデータを送信するために巻き戻したり、ローカルのフレーム数がサーバのフレーム数に達するまで入力データの送信を待つ必要があるかもしれません。

特に、リセットやセーブステートのロードは、常にサーバのフレームカウントに同期しているため、サーバのみがコアのリセットやセーブステートのロードを行うことができます。

セーブステートのロードの前の入力はロード後には必要ないので、セーブステートのロードコマンドを受信すると、すべてプレイヤーのフレームカウントは、少なくともサーバーのフレームカウント（該当する場合はローカルのフレームカウントを含む）に更新されます。これは、フレームカウントが、実行フレームあたり1フレーム以上の割合でスキップする唯一の条件です。

## 実装

Netplayは実質的に、リングバッファとして実装されている入力データのバッファと、いくつかのプリフレームとポストフレームの挙動で構成されています。

RetroArchのNetplayでは、`Self`, `Other`, `Unread` という、3つの重要なフレームのロケーションが存在します。

それぞれのロケーションが、フレームと、そのフレームに対応するステートバッファを参照します。ステートバッファには、そのフレームのセーブステートと、ローカルとリモートの両方のプレーヤーからの入力が含まれます。

`Self`はRetroArchが自分自身の位置を認識しているロケーションで、相手から読み取った位置よりも前か後ろかになります。`Self`は、ステートをロードするためにローカルのフレームカウントが強制的にスキップされる場合を除き、実行されたフレームごとに1フレームの割合で進行します。

`Unread`とは、すべてのプレイヤーのデータが読み込まれていない最初のフレームのことです。通常、`Unread`は`Self`よりも小さいですが、あるクライアントが他のクライアントよりも 他のクライアントよりも先に進むことが可能です。

`Other`は、直近で完全に同期していたロケーションのことです。つまり、`Other-1`は、ローカルとリモートのすべての入力が実行された最後のフレームを指します。同期をとるために`Other`よりも遠くに巻き戻す必要はなく、`Other`は常に`Self`と`Unread`の両方以下です。ステートバッファはリングなので、`Other`は上書きしてはいけない最初のフレームとなります。

サーバーは、複数のクライアントを扱うことができるので、若干ですが複雑な仕事をする必要があります。プレイ中の各コネクションについて、プレイヤーごとの`Unread`フレームを維持し、各プレイヤーの`Unread`フレームのうち最も早いものをグローバル`Unread`フレームとします。

また、サーバーは入力データを転送します。入力データがサーバの現在のフレームよりも前のフレームから受信された場合、サーバは直ちにそれを転送します。

それ以外の場合は、そのフレームに達した時点で転送します。つまり、フレーム`n`の間、サーバーは自分のデータと他のプレイヤーのデータを任意の数だけフレーム`n`に送ることができますが、フレーム`n+1`を送ることはありません。これは、サーバーのクロックが、プレーヤーの反転、プレーヤーの加入・離脱、ステートの保存・読み込みなど、すべての同期関連イベントのアービターであるためです。

TODO

## プロトコル

ネットプレイの接続には、クライアントとサーバが同じソフトウェア(ROM)を使用していることを確認し、クライアントを同期させるためのハンドシェイクが必要です。その後、入力パケットの交換が行われます。

**ハンドシェイクの手順**

この部分はサーバーとクライアントの両方が行います。

1. コネクションヘッダの送信
2. コネクションヘッダを受信・検証
3. ニックネームの送信
4. ニックネームの受信

クライアント側のみ

5. 必要ならばPASSWORDを送信
6. INFOを受信
7. INFOを送信
8. SYNCを受信

サーバー側のみ

5. 必要ならばPASSWORDを受信
6. INFOを送信
7. INFOを受信
8. SYNCを送信

なお、サーバとクライアントの両方が、コネクションヘッダを受け取る前に送信しています。これは意図的なものです。

これにより、サーバとクライアントのどちらか（両方ではない）が、相手側のコネクションヘッダをエコーすることで、同じコネクションヘッダを使用することができます。

## その他の特徴

一般的には、入力レイテンシは望まれないと考えられています。しかし、入力レイテンシも選択肢の一つです。

入力レイテンシの利点は、実際の実行がフレーム数よりも遅れることです。

フレーム数に比べて実際の実行が遅れるため、遠隔地のデータが利用できることが多くなり、巻き戻しの頻度もコストも少なくて済みます。

ステートバッファには、入力遅延が有効な場合に使用される`run`というロケーションが追加されています。

この場合、`self`は入力が読み込まれている場所を指し、`run`は実際に実行されているフレームを指します。`run`は純粋にローカルです。

## コマンドフォーマット

Netplayコマンドは、32bitのコマンド識別子と、それに続く32bitのペイロードサイズ（いずれもネットワークのバイトオーダーに準拠）と、それに続くペイロードで構成されています。

コマンド識別子は`netplay.h`に記載されており、以下にコマンドを説明します。特に指定のない限り、ペイロードの値はすべてネットワークのバイトオーダーに準拠です。

**Command**: ACK

Payload: None

Description:
> Acknowledgement. Not used.

**Command**: NAK

Payload: None

Description:
> Negative Acknowledgement. If received, the connection is terminated. Sent
> whenever a command is malformed or otherwise not understood.

**Command**: DISCONNECT

Payload: None

Description:
> Gracefully disconnect. Not used.

**Command**: INPUT

Payload:

    {
       frame number: uint32
       is server data: 1 bit
       player: 31 bits
       controller input: uint32
       analog 1 input: uint32
       analog 2 input: uint32
    }

Description:

> Input state for each frame. Netplay must send an INPUT command for every
> frame in order to function at all. Client's player value is ignored. Server
> indicates which frames are its own input data because INPUT is a
> synchronization point: No synchronization events from the given frame may
> arrive after the server's input for the frame.

**Command**: NOINPUT

Payload:

    {
       frame number: uint32
    }

Description:

> Sent by the server to indicate a frame has passed when the server is not
> otherwise sending data.

**Command**: NICK

Payload:

    {
       nickname: char[32]
    }

Description:
> Send nickname. Mandatory handshake command.

**Command**: PASSWORD

Payload:

    {
       password hash: char[64]
    }

Description:
> Send hashed password to server. Mandatory handshake command for clients if
> the server demands a password.

**Command**: INFO

Payload:

    {
       core name: char[32]
       core version: char[32]
       content CRC: uint32
    }

Description:
> Send core/content info. Mandatory handshake command. Sent by server first,
> then by client, and must match. Server may send INFO with no payload, in
> which case the client sends its own info and expects the server to load the
> appropriate core and content then send a new INFO command. If mutual
> agreement cannot be achieved, the correct solution is to simply disconnect.

**Command**: SYNC

Payload:

    {
       frame number: uint32
       paused?: 1 bit
       connected players: 31 bits
       flip frame: uint32
       controller devices: uint32[16]
       client nick: char[32]
       sram: variable
    }

Description:
> Initial state synchronization. Mandatory handshake command from server to
> client only. Connected players is a bitmap with the lowest bit being player
> 0. Flip frame is 0 if players aren't flipped. Controller devices are the
> devices plugged into each controller port, and 16 is really MAX_USERS.
> Client is forced to have a different nick if multiple clients have the same
> nick.

**Command**: SPECTATE

Payload: None

Description:
> Request to enter spectate mode. The client should immediately consider itself
> to be in spectator mode and send no further input.

**Command**: PLAY

Payload:

    {
       reserved: 31 bits
       as slave?: 1 bit
    }

Description:
> Request to enter player mode. The client must wait for a MODE command before
> sending input. Server may refuse or force slave connections, so the request
> is not necessarily honored. Payload may be elided if zero.

**Command**: MODE

Payload:

    {
       frame number: uint32
       reserved: 13 bits
       slave: 1 bit
       playing: 1 bit
       you: 1 bit
       player number: uint16
    }

Description:
> Inform of a connection mode change (possibly of the receiving client). Only
> server-to-client. Frame number is the first frame in which player data is
> expected, or the first frame in which player data is not expected. In the
> case of new players the frame number must be later than the last frame of the
> server's own input that has been sent, and in the case of leaving players the
> frame number must be later than the last frame of the relevant player's input
> that has been transmitted.

**Command**: MODE_REFUSED

Payload:

    {
       reason: uint32
    }

Description:
> Inform a client that its request to change modes has been refused.

**Command**: CRC

Payload:

    {
       frame number: uint32
       hash: uint32
    }

Description:
> Informs the peer of the correct CRC hash for the specified frame. If the
> receiver's hash doesn't match, they should send a REQUEST_SAVESTATE command.

**Command**: REQUEST_SAVESTATE

Payload: None

Description:
> Requests that the peer send a savestate.

**Command**: LOAD_SAVESTATE
Payload:

    {
       frame number: uint32
       uncompressed size: uint32
       serialized save state: blob (variable size)
    }

Description:
> Cause the other side to load a savestate, notionally one which the sending
> side has also loaded. If both sides support zlib compression, the serialized
> state is zlib compressed. Otherwise it is uncompressed.

**Command**: PAUSE

Payload:

    {
       nickname: char[32]
    }

Description:
> Indicates that the core is paused. The receiving peer should also pause.  The
> server should pass it on, using the known correct name rather than the
> provided name.

**Command**: RESUME

Payload: None

Description:
> Indicates that the core is no longer paused.

**Command**: STALL

Payload:

    {
       frames: uint32
    }

Description:
> Request that a client stall for the given number of frames.

**Command**: RESET

Payload:

    {
       frame number: uint32
    }

Description:
> Indicate that the core was reset at the beginning of the given frame.

**Command**: CHEATS

Unused

**Command**: FLIP_PLAYERS

Payload:

    {
       frame number: uint32
    }

Description:
> Flip players at the requested frame.

**Command**: CFG

Unused

**Command**: CFG_ACK

Unused

