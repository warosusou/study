# The Internet Address Architecture

## 2.1 Introduction

- IP アドレスがどのように割り当てられ、ルーティングやブロードキャストなどへ影響しているのかを議論
- IPv4、IPv6 両方にフォーカス

- インターネット、TCP/IP のネットワークではデバイスに 1 つ以上のアドレスが振られる
- IP アドレスは、電話番号と対比されることが多い
  - IP アドレスは DNS を用いて隠蔽される
  - DNS が死んだときやネットワークのセットアップ時には IP アドレスを直に向き合う
- アドレスはユニークである必要がある
    - ローカルネットワークではその組織のポリシーで割り当てを決定
    - グローバルインターネットアドレスは、通常、ISPから受け取る

## 2.2 Expressing IP Address
- 一般的にIPアドレスというと、IPv4が連想される。
- v4アドレスはdotted-quadまたはdotted-decimalで表現される。
    - 例えば165.195.130.107
    - 4文字の10進数をプリオドで区切って表現
    - 値域は[0, 255]
    - 32bitのIPv4アドレスを、扱いやすい10進数で簡単に表現できる
    - 実際には2進数と対応している(Table 2-1)
- IPv6は128bit、つまりIPv4と比べて4倍複雑
    - 記法は4桁の16進数をコロンで区切って(ブロック、フィールド)表現
    - v6アドレスは8個のブロックを持つ
    - 例えば5f05:2000:80ad:5800:0058:0800:2023:1d71
    - 10進数ほど浸透していないが、2進数との変換が簡単
- IPv6アドレスの簡略化についてはRFC4291で標準化
    1. 各ブロックの先頭から連続する0は省略可能
    1. 全て0になっているブロックは連続して::で省略できる この省略はアドレス内で1回だけ適用可能
    1. IPv4アドレスの前にffffのブロックを繋げ、右詰めでIPv6アドレスに組みことで、IPv4アドレスをIPv6アドレスで表現できる(IPv4-mapped IPv6 address)
    1. 従来型の32bitアドレス(IPv4?)をIPv6のdotted-quad表記で表現することもできる(IPv4-compatible IPv6 address)
        - IPv4-mapped addressとは異なる
        - 互換性があるのはIPv4アドレスのようなアドレスを扱えるソフトウェアのみ
        - もともとはv4からv6への移行措置で考えられたが、現在は必須項目ではない(RFC4291)
- URLのようにv6アドレスのコロンと他のコロン(URLであればポート番号の指定)が混在する場合はv6アドレスを[]で囲むことで表現できる
- RFC4291の仕様では同一の内容を表現するのに、何通りかのv6アドレスが許されるので、RFC5952で記法について制限が加えられた
    1. ブロック先頭の0は省略する
    1. ::の省略は省略できる範囲が最も大きい箇所で適用する もし、同じ長さの0ブロックが複数ある場合は先頭のブロックへ適用する
    1. 16進数の[a, f]は小文字で表記する

## 2.3 Basic IP Address Structure
- IPアドレスは巨大なアドレス空間を持つことから、いくつかの塊に分割して利用するのが、便利である
- 大半のIPv4アドレスは単一のネットワークインターフェースを識別できる、ユニキャストアドレスとなる
- その他にもブロードキャスト、マルチキャスト、エニキャスト用のアドレスもある
- IPv6アドレス空間の大半は使用されていない

## 2.3.1 Classful Addressing
- インターネットアドレスの構造について、IPアドレスを持つインターフェースとそれが参加しているネットワークが見つけられるように設計された
- net numberとhost number
- host numberについては、大半のホストが1つのインターフェースしか持たないことからinterface addressとも呼ばれている
- アドレス空間の分割について、5つのクラスが考えられた(Figure 2-1)
- A,B,Cがユニキャスト用のクラス
- クラスフルアドレスは識別できるネットワークとホストの関係がトレードオフになっている
- A,B,Cクラスでアドレッシングを行うことにスケーリング上で問題が出てきた(1980年初期)
    - A,Bでは無駄なホストアドレスが大きいが、Cではホストアドレスが不足

## 2.3.2 Subnet Addressing
- スケーリング問題はLANの進歩と普及にも原因がある
- 解決方法としては、ネットワークアドレスを最初に決定し、その後で、組織の管理者がさらに分割を行うという考え
- subnet addressing(RFC0950)の登場
- サイトはA,B,Cクラスでネットワークアドレスを割り当てられてから、本来のホストアドレス部分をSubnet IDとHost IDに分割
- ルータやホストはSubnetとHostの境界を知る必要性が出てくる
- インターネット(サイト外)のルータ、ホストはSubnetを気にせず、アドレス指定をする

## 2.3.3 Subnet Masks
- サブネットマスクはネットワーク、サブネットとホストの分離を行うために導入された
- サブネットマスクの長さは対応するIPアドレスと同じ(32bit,128bit)
- IPアドレスの設定と同じように設定される
- IPv4ではdotted-decimalの記法
- サブネットマスクは0b1が連続した後に、0b0が連続しているので、0b1の部分をプレフィックスとして、プレフィックス長で記述する方法もある
- IPアドレスの先頭から、サブネットマスクのプレフィックス長までの部分がネットワーク、サブネットを表現している
- ネットワークアドレス部はIPアドレスとサブネットマスクのAND演算にて取り出せる

## 2.3.4 Variable-Length Subnet Masks(VLSM)
- 同一サイト内で異なる長さのサブネットを運用できる
- 運用コストは上昇するが、サブネットごとに保持できるホストの数を柔軟に変えられる
- VLSMを用いたネットワークはOSPF、IS-IS、RIPv2などの動的ルーティングプロトコルにて通信が行える
- 自明でない構成として、サブネットが2つのホストしか含まない構成がある
    - ルータが2つのネットワークを繋いでいて、それぞれの終端となっている場合、ルータのネットワークは/31または/127とするのが一般的である

## 2.3.5 Broadcast Address
- サブネットブロードキャストアドレスはサブネットアドレスのホスト部のビットを全て1にすることで得られる
- このアドレスを用いるデータグラムはdirected broadcastとして知られている
    - ディレクテッドブロードキャストはセキュリティーの観点から問題があるために、今日では無効化されていることも多い
    - RFC2644によって、ルータは標準設定でディレクテッドブロードキャストのフォワーディングを無効化している
    - そもそもフォワーディングができない仕様も許容される
- local net broadcast(limited broadcast)として255.255.255.255予約されている
- このアドレスはルータによってフォワーディングされない
- リンク層におけるブロードキャストの機構が利用可能であればそれらも利用される
- ブロードキャストアドレスはUDP/IPやICMPで使用される
- IPv6にはブロードキャストの仕組みを備えていないので、マルチキャストにて代替される

## 2.3.6 IPv6 Addresses and Interface Identifiers

- IPv6アドレスはv4と比べて4倍も長いので他のアドレス構造が存在する
- 予約されたIPv6におけるプレフィックスはスコープと呼ばれ、ネットワークのどこで使用できるかを示す
- スコープとしてはnode-local、link-local、globalがある
- IPv6では単一のインターフェースが複数のアドレスを持っていることも一般的である
- マルチキャストアドレスも含めて、IPv6ノードに要求されるアドレスはRFC4291で決められている
- site-local(fec0::/10)はRFC3879で非推奨となった サイトの定義が厳格でないことがあることによる
- リンクローカルといくつかのグローバルIPv6アドレスではIIDをユニキャストに用いる
- IIDはIPv6の下位bitとして用いられる
    - IPv6アドレスが0b000で始まる場合を除く
- IIDは一般的に64bitでMACアドレスをベースにしたmodified EUI-64 format(EUI64)か、アドレストラッキング防止のために乱数を用いる方法で決定される
- extended unique identifier(EUI)
- EUIは24bitのOrganizationally Unique Identifier(OUI)と続く40bitのextension identifier(OUIの組織が決定)で構成される
- OUIはIEEERAによって管理されている
- OUIはEUIはuniversally adinisteredとlocally administeredがあり、インターネットの話をすると大半は前者を指している
- 多くのIEEE標準では省略記法を用いている(48bit)
    - フォーマットとしての違いは長さのみ
- OUIの1バイト目(24bit中の先頭8bit)における下位2ビットはそれぞれuビット、gビットと呼ばれる
    - uビットが1の時、OUIがlocally administeredであることを示す
    - gビットが1の時、アドレスがグループ、またはマルチキャストであることを示す
- 今回はgビットが0であることを想定して話を進める
- EUI-48が既に存在する場合、EUI-64はEUI-48の3バイト目の後に0xFFFEを挿入することで作成できる
- IIDに用いられるmodifed EUI-64はuビットを反転することで作成される
- EUI-48が存在しないが、他の下位アドレス(例: AppleTalk)が存在する場合は、48bitになるように0で右詰めして作成する
- トンネルやシリアルリンクなどの他の識別子を持たないインターフェースでは同一ノードの他のインターフェースやノードに振られている何らかの識別子から派生して割り当てられる
- 最終手段は手入力
## 2.3.6.1 Examples
- link-local IPv6アドレスがEthernetのハードウェアアドレスとリンクローカルプレフィックスのfe80::/10から作成されている
- リンクローカルアドレスの/64はRFC4291で策定された
- Windowsにおけるtunnel endpointの例
    - IPv6のトラフィックをIPv4専用のネットワークへ転送
    - ISATAP(RFC5214)
    - 物理アドレスはIPv4アドレスを16進数で表現したそのもの
    - OUIはIANA
- v6アドレスに付属する%2はWindowsにおける、zoneID
    - IPv6アドレスに応じたインターフェースのインデックス
