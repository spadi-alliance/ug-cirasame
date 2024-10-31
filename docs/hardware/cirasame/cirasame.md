# CIRASAME board

CIRASAME (CITIROC based multi-MPPC readout electronics for continuous timing measurement) is a readout board for multi-MPPCs using [CITIROC 1A](https://www.caen.it/products/citiroc-1a/) ASICs.
This is dedicated electronics to readout scintillating fiber trackers for the J-PARC E50 experiment, and is designed as an on-detector type electronics to be attached to those detectors.
But, it can readout others detectors equipped with MPPCs.
This board design is sophisticated for implementing a streaming readout TDC since the J-PARC E50 experiment uses a trigger-less data-streaming type DAQ system.

- Manufacturer: 有限会社ジー・エヌ・ディー
- Product No: GN-2107-3

The [figure](#CIRASAME-PIC) shows the photograph of the CIRASAME board.
CIRASAME is a 128 mm x 200 mm sized readout board with connectors on the back of the board for mounting an MPPC.
The MPPC connector position is designed to mount HAMAMATSU S14826(ES1), which is special order product.
See the MPPC section for information on MPPC and ASIC channel mapping.
CIRASAME has a SFP+ port for a data link, a JTAG port, NIM IO, analog output ports, a MIKUMARI port for clock synchronization, and a power connector.
The FPGA mounted on the board is AMD Xilinx Kintex-7 FPGA (XC7K-160T-2FFG676C), which is the same as that of AMANEQ.
This board has the same a jitter (CDCE62002) and 2Gb DDR3-SDRAM as those on AMANEQ.
Thus, the digital part of CIRASAME is based on the design of AMANEQ, it uses many same ICs.
CIRASAME also has dedicated functionalities for operating MPPCs, i.e., an APD bias supply IC (MAX19232ETC+T).
Please also see the [AMANEQ user guide](https://spadi-alliance.github.io/ug-amaneq/).

![CIRASAME-PIC](pic-cirasame.png "Picture of CIRASAME GN-2107-3"){: #CIRASAME-PIC width="80%"}

The specification is summarized as follows.

- Size: 128 mm (H) x 200 mm (V)
- Num of MPPCs: 2 (S14826(ES1) x2)
    - 128ch in total
    - If you prepare an intermediate board changing a connector type, other types of MPPCs can be read.
- ASIC: Weeroc/Omega CITIROC 1A BGA x4
- Bias: MAX1932ETC+T
    - Supply range: 40-70V (256 steps)
- Analog outputs:
    - One analog high-gain output from CITIROC
    - One probe output from CITIROC
- FPGA: AMD Xilinx XC7K-160T-2FFG676C
- Flash memory: CYPRESS S25FL128SAGMFIR01
- Data links: SFP+ (10 Gbps in maximum)
    - Link media depends on the type of SFP modules.
- Clock synchronization: One MIKUMARI port (SFP)
- Num of NIM input: 1 (LEMO)
- Num of NIM output: 1 (LEMO)
- Power supply: 20-35V DC
    - Jack: 2.10mm ID, 5.50mm OD
    - Main connector: 日本圧着端子 S2P-VH(LF)(SN)
    - **Note that the connector is different from that on AMANEQ**
- Fuse limit: 1A
    - Fuse product: Littelfuse 0251001.NRT1L
- Num of DIP switch bits: 4
- Clock generator IC: TI CDCE62002
- External memory: 2Gb DDR3-SDRAM
    - Speed: DDR3-1333 (max)

![CIRASAME-BLOCK](cirasame-block.png "Block diagram of GN-2006-4"){: #CIRASAME-BLOCK width="70%"}

MPPCs are biased by the MAX1932 bias supply, which is controlled by FPGA.
It can control output voltage with 256 steps between 40V to 70V.
The MPPC signal lines are parallel terminated with 100 ohm registers together with 0.22 uF capacitances for DC blocking.
The MPPC signals are fed into four CITIROCs, and are amplified, shaped, and discriminated in CITIROC.
For details of this ASIC, please see its data sheet.
The comparator (discriminator) outputs are input in parallel to the FPGA.
The signals are input to the streaming TDC and the scaler in the FPGA.

Since this board aims to have the function for the streaming TDC, the design of a charge measurement line is rather simplified.
Only the analog output for the high-gain side is arranged to ADC (AD9220) and to the analog output port with the LEMO type connector.
Since the analog output of the ASIC without setting the read register becomes high impedance, four outputs are simply merged to one line.
The probe output from CITIROC are once connected to an analog switch controlled by the FPGA, and its output is arranged to the LEMO type connector.
The developer expects that these analog lines are used only for checking the signals using an oscilloscope.
The charge measurements with the DAQ function is not considered.

In the [figure](#CITIROC-BLOCK), the ASIC control lines from the FPGA are omitted.

## Board interface

### MPPC connector

AMANEQにバックプレーンが存在しないので、前後両方に入出力が存在します。後ろ側には2つのメイン入力 (Main Input) が存在します。これらはHULのメイン入力と互換です。上側をUpper Main Input、下側をLower Main Input Lとします。

コネクタはハーフピッチの68極コネクタ（KEL 8831E-068-170L-F）です。適合するコネクタでケーブルを作成して接続してください。チャンネルアサインはコネクタの基準マーカー側が若い番号になります。GND接続はA1A2、およびB1B2の2ペア (基準マーカー直下) です。コモンモードレンジで-4Vから+5Vまでの差動信号をサポートします。LVDS、ECL、PECL、LVPECLなどがサポート対象です。ECLおよびPECL信号規格を利用する場合、AMANEQ側には電流制限抵抗が存在しないため、ドライバ側で電流制御されている必要があります。

メイン入力では差動信号をLVCMOSに変換してからFPGAへ入力します。最高繰り返しスピードは560 Mbpsです。それよりも高速な信号を入力する場合メザニンカードの使用を検討してください。

### SFP1, 2

データ通信用のSFP+ポートです。FPGAのGTXトランシーバへ接続されています。最大通信速度は10 Gbpsです。SFPおよびSFP+モジュールが適合します。SFP1と2の利用方法は各ファームウェアに依存します。

[SiTCP](https://www.bbtech.co.jp/sitcp/)ないし[SiTCP-XG](https://www.bbtech.co.jp/products/sitcp-xg-license/)を利用したファームウェアでは、このポートへGbE用か10GbE用のSFPモジュールを挿入してください。
SFPモジュールは多数の選択肢が存在しますが、複数の通信レートに対応したモジュール、例えば1Gbps、100Mbps、10Mbpsの3種類に対応したもの、は利用しないでください。オートネゴシエーションがうまく働かないためです。SiTCPではれば1Gbps専用の物を、SiTCP-XGであれば10Gbps専用の物を選択してください。

SFPモジュールは光ファイバーケーブルをリンクメディアに想定した規格ですが、LANケーブルに対応したRJ45付きモジュールも存在します。AMANEQでも光ファイバーの使用を想定していますが、SiTCPを搭載したファームウェアではLANケーブル用のSFPモジュールでも利用可能です。一方SiTCP-XGではLANケーブルを用いた10GbE通信は未テストです。

### DIP, LED, NIM

![LED-DIP-NIM](amaneq-led-nim.png "LED, DIP, and NIM"){: #LED-DIP-NIM width="75%"}

縦に5つ並んだLEDのうち一番上がDONE LEDです。このLEDはFPGAのコンフィグレーションが終了すると点灯します。残りの4つは上からLED1, 2, 3, 4とラベルされています。各LEDの意味はファームウェアに依存します。
DIPスイッチは左側が0 (ONとラベルされている側)、右側が1と定義されています。各ビットの意味はファームウェアに依存します。
NIM入出力は基板上側が入力、下側が出力です。AMANEQのNIM出力はNIM信号規格へ変換する部分で簡略化した方式を採用しており、 logic-highとして認識する電圧に基板差があります。そのため、同一のファームウェアでNIM出力をしたとしても、信号幅やタイミングが若干異なって見えることがあります。AMANEQのNIM入出力は補助入力と考えてください。タイミングが重要な高精度信号の入出力には向いていません。
2つあるNIM1とNIM2は基板上のシルクに印字されています。

### Power supply

電源コネクタはAD/DCアダプタ用のジャック（径 2.10mm ID, 5.50mm OD）と、外部DC電源用に日本圧着端子のS4P-VH(LF)(SN)が備わっています。
想定している電源電圧は20-35Vです。
**2つ端子は基板上でつながっており、30V程度の電圧が印加された電極がむき出し状態になります。**
ジャックへキャップをはめたり、S4Pコネクタにテープ等でカバーをしたりして感電を防いでください。
S4Pコネクタは電極が4つあり、上2つが電源、した二つがグランドです。

AMANEQではスイッチングレギュレータを用いて高い電圧から5Vへ降圧しています。このスイッチングレギュレータの仕様的に10V程度から40V程度まで入力を受け付けることができますが、下限はヒューズの電流制限で20V程度に、上限は破損を防ぐために35V程度を推奨にしています。
ヒューズは外部電源の35V系に挿入されており、電流リミットは1Aです。20V以下の電源電圧ではヒューズが切れるので、印加しないでください。また、同じ理由からゆっくり電圧を35Vまで上げるのもやめてください。AC/DCアダプタは24V出力の物が適合しますが、出力パワーが20W程度あるものを選んでください。AMANEQ基板の消費電力はSkeletonファームウェアを除くと全て10Wを超えています。

搭載ヒューズはLitelfuseの0251001.NRT1Lです。

### MIKUMARI port

GN-2006-4にはフロント側にクロック信号同期（[MIKUMARI](https://github.com/RyotaroHonda/mikumari)）用のSFPポートが存在します。MIKUMARI使用時には1Gbps用の光ファイバー用SFPモジュールを挿入してください。LANケーブル用のモジュールは利用不可です。

GN-2006-1には同じ場所にBelle/Belle-II実験のトリガーポート (B2TT) 互換のRJ45ポートが存在しますが、全てのファームウェアでこのポートは未使用です。AMANEQが設計された当初、クロック信号分配にB2TT規格を利用する予定だったなごりです。GN-2006-1でMIKUMARIを使用するためには、mine-mezzanine clock receiver (CRV) を右上の小型メザニンスロットに搭載してください。

### Mezzanine slots

AMANEQにはHULと互換性のあるメザニンスロットが2つ搭載されており、HUL用に開発されたメザニンカードを搭載する事が出来ます。HUL用のメザニンについては[HULのページ](https://openit.kek.jp/project/HUL/HUL)を参照してください。
**基板がたわんで接続が悪くなるため、3つあるメザニンカード用の支柱のうち真ん中の1つへねじ止めをしない方針にしました。**HULでも同様なので真ん中のネジをリリースする事をお勧めします。

メザニンカードとAMANEQのFPGAは32ペアのLVDS信号線で接続されています。信号入出力の方向はファームウェアに依存するので、搭載しているメザニンカードと適合しないファームウェアをダウンロードした場合信号がショートする事があります。最悪部品を破損させるので、ファームウェアの入れ替えを行ってからメザニンは取り付けてください。メザニンへはベースコネクタから3.3Vが給電されています。

Mimi-mezzanine slotは現状GN-2006-1にMIKUMARIポートを増設するためだけに利用しています。

## Board ICs

### FPGA

FPGAはAMD XilinxのXC7K-160T-2FFG676Cです。HULのFPGAとはサイズは同じですが、スピードグレードが異なり、AMANEQではスピードグレード-2のFPGAを利用しています。GTXチャンネルで10Gbpsまでの高速通信が可能です。

### Jitter cleaner

AMANEQには2出力のクロックジェネレータ、TI社のCDCE62002が搭載されています。いくつかのファームウェアで基準クロックを生成するためや、MIKUMARIの変調クロックからクロック信号を復元するために利用されます。そのジッタ―パフォーマンスはFPGA内部のMMCMよりも優秀です。

<!-- ただし、zero-delay modeが搭載されていないため、出力ディバイダで分周を行った場合、分周比に従って位相の不定性が出ます。これを電源サイクルに対して確定的にしたい場合、各ファームウェアに搭載されている位相選択の機能を有効にしてください。 -->

納入時のCDCE62002にはファクトリーデフォルトが書き込まれており、自分で必要な設定を書き込まないとAMANEQのファームウェアは動作しません。AMANEQが納品されたらSkeletonファームウェアを書き込み、CDCE62002の設定を行ってください。このICは不揮発性メモリを持っているので、一度設定を書き込んだら以降のアクセスは必要ありません。

### DDR3-SDRAM

AMANEQには2GbのDDR3-SDRAMが搭載されています。最高動作速度はDDR3-1333です。2024年7月現在このメモリを活用したファームウェアはまだ存在しません。

### Flash memory

AMANEQのSPI flash memoryはCYPRESSのS25FL128SAGMFIR01です。Vivado上ではCypress/Spansion s25fl128sxxxxxx0-spi-x1_x2_x4を選択してください。拡張信号線を配線しているので、VivadoでbitstreamやMCSを生成するときにはSPIx4を選択してください。

## Power supply ICs

![3V3D](3v3d.png "3.3V power IC"){: #3V3D"}

AMANEQでは外部入力の35VをLT8612UDE#PBFを用いて5Vへ降圧し、それを更に複数の電源電圧へ降圧しています。各電源ICのそばには[図](#3V3D)左上に示すようなテストポイントが設置されており、出力電圧を確認する事が出来ます。AMANEQ納入後の初回電源投入時に全ての電源電圧の確認を行ってください。

<div align="center">
<table class="vmgr-table">
  <thead><tr>
    <th class="nowrap"><span class="mgr-20">電源ラベル</span></th>
    <th class="nowrap"><span class="mgr-20">電圧</span></th>
    <th class="nowrap"><span class="mgr-20">部品番号</span></th>
    <th class="nowrap"><span class="mgr-20">テストポイント番号</span></th>
  </tr></thead>
  <tbody>
  <tr>
    <td class="tcenter">+5V</td>
    <td class="tcenter">+5.0V</td>
    <td class="tcenter">U50</td>
    <td class="tcenter">なし</td>
  </tr>
  <tr>
    <td class="tcenter">+3V3D</td>
    <td class="tcenter">+3.3V</td>
    <td class="tcenter">U45</td>
    <td class="tcenter">TP4</td>
  </tr>
  <tr>
    <td class="tcenter">+2V5D</td>
    <td class="tcenter">+2.5V</td>
    <td class="tcenter">U46</td>
    <td class="tcenter">TP5</td>
  </tr>
  <tr>
    <td class="tcenter">+1V8D</td>
    <td class="tcenter">+1.8V</td>
    <td class="tcenter">U44</td>
    <td class="tcenter">TP3</td>
  </tr>
  <tr>
    <td class="tcenter">+1V0D</td>
    <td class="tcenter">+1.0V</td>
    <td class="tcenter">U43</td>
    <td class="tcenter">TP2</td>
  </tr>
  <tr>
    <td class="tcenter">+1V8GTX</td>
    <td class="tcenter">+1.8V</td>
    <td class="tcenter">U49</td>
    <td class="tcenter">TP8</td>
  </tr>
  <tr>
    <td class="tcenter">+1V2GTX</td>
    <td class="tcenter">+1.2V</td>
    <td class="tcenter">U47</td>
    <td class="tcenter">TP6</td>
  </tr>
  <tr>
    <td class="tcenter">+1V05GTX</td>
    <td class="tcenter">+1.05V</td>
    <td class="tcenter">U48</td>
    <td class="tcenter">TP7</td>
  </tr>
  <tr>
    <td class="tcenter">-3V3D</td>
    <td class="tcenter">-3.0V</td>
    <td class="tcenter">U20</td>
    <td class="tcenter">なし</td>
  </tr>
</tbody>
</table>
</div>
