## tbs6812_drv

ISDB-T/ISDB-S/ISDB-S3/ISDB-C 対応Linux向けTBS6812 a.k.a. PT4Kドライバ

カーネルモジュールのビルド環境を構築後、dkmsでインストールします。

```sh
sudo cp -a ./ /usr/src/tbs6812_drv-0.2.0
sudo dkms add tbs6812_drv/0.2.0
sudo dkms autoinstall tbs6812_drv/0.2.0
```

DVBデバイスとして見えるのでdvb-toolsなどを導入するとストリームを得ることができます。

```sh
dvbv5-zap -a 0 -c default-channels.conf defaultTLV -r -P -o defaultTLV.mmts
dvbv5-zap -a 0 -c default-channels.conf defaultTS -r -P -o defaultTS.ts
```

recdvbのfork <https://github.com/otya128/recdvb> を用いるとストリームIDを指定してストリームを得ることができます。

```sh
# デフォルトTLV (ISDB-S3)を受信
recdvb --dev 0 0xb110 - defaultTLV.mmts
# デフォルトTS (ISDB-S)を受信
recdvb --dev 0 0x40f1 - defaultTS.ts
# 地上波の物理26chを受信
recdvb --dev 0 26 - terr-26ch.ts
```

### 仕様

`DELIVERY_SYSTEM = ISDBS`かつ`STREAM_ID`が8未満のときISDB-Sの相対TS番号で選局されます。

`DELIVERY_SYSTEM = ISDBS`かつ`STREAM_ID`の下位16ビット部の上位4ビットが0xbまたは0xcのとき (0xbXXX, 0xcXXX) ISDB-S3のストリームIDで選局、それ以外のときISDB-SのTSIDで選局されます。

### 実験的な ISDB-C 対応について

ISDB-CをDVB-C Annex Aの 6MHz信号として扱う実験的な実装です。
帯域幅は 6MHz、シンボルレートは 5,274,000 Baud に限定します。
周波数は Hz 単位で 45–868 MHz、変調は QAM/64、QAM/256、QAM/AUTO を受け付けますが、
QAM/256 は未確認です。復調器は 6MHz 用の探索範囲内でシンボルレートと変調を自動検出します。
一般の 7/8MHz DVB-C には対応しません。

```ini
# isdbc-channels.conf
[C56]
DELIVERY_SYSTEM = DVBC/ANNEX_A
FREQUENCY = 423000000
SYMBOL_RATE = 5274000
MODULATION = QAM/AUTO
```

```sh
dvbv5-zap -a 0 -c isdbc-channels.conf -r -P -t 30 -o C56.ts C56
```
