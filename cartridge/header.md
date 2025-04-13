# カートリッジヘッダ

各カートリッジには、`0x0100..014F`までのアドレス範囲にヘッダが含まれています。ヘッダには、ゲームと、そのゲームが動作するハードウェアに関する以下の情報が含まれています。

## 0100-0103 - エントリポイント

任天堂ロゴが表示された後、内蔵のブート手順ではこのアドレス（`0x0100`）にジャンプし、カートリッジ内の実際のメインプログラムにジャンプするはずです。

ほとんどの市販ゲームでは、この4バイトの領域には `nop`命令と、それに続く`jp $0150`命令が含まれています。

## 0104-0133 - 任天堂のロゴ

これらのバイトは、ゲームボーイの電源を入れたときに表示される任天堂ロゴのビットマップを定義します。このビットマップの16進数のダンプは次のようになります。

```binary
 CE ED 66 66 CC 0D 00 0B 03 73 00 83 00 0C 00 0D
 00 08 11 1F 88 89 00 0E DC CC 6E E6 DD DD D9 99
 BB BB 67 63 6E 0E EC CC DD DC 99 9F BB B9 33 3E
```

DMGのブート手順では、（表示後に）このビットマップの内容を検証し、このバイトが正しくない場合には動作を停止します。CGBでは、ビットマップの前半部分（18バイト）のみを検証します。

## 0134-0143 - タイトル

大文字のASCII文字列によるゲームのタイトルです。16文字に満たない場合、後のバイトは`0x00`バイトで埋められます。

後のゲームでは、タイトルの上限を15文字(`0x0134..0142`)または11文字(`0x0134..013E`)に縮小して、残った領域を次の用途で使用しています。

## 013F-0142 - 製造元コード

古いカートリッジでは、この領域はタイトルの一部となっていますが、新しいカートリッジでは、このエリアに4文字で大文字のASCII文字列による製造元コードが入っています。目的や深い意味は不明です。

## 0143 - CGBフラグ

（上述のとおり）古いカートリッジでは、このバイトはタイトルの一部となっています。

CGB以降のゲームでは、このバイトを解釈して、CGBモードを有効にするか、非CGBモードにフォールバックするかを決定します。

```
  0x80: CGB機能をサポートしているが、古いゲームボーイでも動作します
  0xC0: CGB専用ゲーム
```

`bit7=1`で`bit2=1 or bit3=1`のとき、ゲームボーイはPGBモードという特殊な非CGBモードに切り替わります。(このモードについては不明です)

## 0144-0145 - ライセンスコード(新)

ゲームの開発元または販売元を示す2文字のASCIIライセンスコードを指定します。

この2バイトをライセンスコードとして使用するのは、0x14Bに`0x33`が設定されている場合のみです。SGBがリリースされた後に発売されたゲームでは全てのゲームが該当します。

<details>
  <summary>一覧</summary>

Code | Publisher
-----|-----------
`00` | None
`01` | Nintendo R&D1
`08` | Capcom
`13` | Electronic Arts
`18` | Hudson Soft
`19` | b-ai
`20` | kss
`22` | pow
`24` | PCM Complete
`25` | san-x
`28` | Kemco Japan
`29` | seta
`30` | Viacom
`31` | Nintendo
`32` | Bandai
`33` | Ocean/Acclaim
`34` | Konami
`35` | Hector
`37` | Taito
`38` | Hudson
`39` | Banpresto
`41` | Ubi Soft
`42` | Atlus
`44` | Malibu
`46` | angel
`47` | Bullet-Proof
`49` | irem
`50` | Absolute
`51` | Acclaim
`52` | Activision
`53` | American sammy
`54` | Konami
`55` | Hi tech entertainment
`56` | LJN
`57` | Matchbox
`58` | Mattel
`59` | Milton Bradley
`60` | Titus
`61` | Virgin
`64` | LucasArts
`67` | Ocean
`69` | Electronic Arts
`70` | Infogrames
`71` | Interplay
`72` | Broderbund
`73` | sculptured
`75` | sci
`78` | THQ
`79` | Accolade
`80` | misawa
`83` | lozc
`86` | Tokuma Shoten Intermedia
`87` | Tsukuda Original
`91` | Chunsoft
`92` | Video system
`93` | Ocean/Acclaim
`95` | Varie
`96` | Yonezawa/s'pal
`97` | Kaneko
`99` | Pack in soft
`A4` | Konami (Yu-Gi-Oh!)
</details>

## 0146 - SGBフラグ

ゲームがSGB機能をサポートしているかどうかを示すフラグです。

SGB機能をサポートしていない場合、`0x00`が設定されます。SGB機能をサポートしている場合、`0x03`が設定されます。

このバイトが `0x03` 以外の場合、SGB側はゲームボーイ側から送られてきたコマンドパケットを無視します。

## 0147 - カートリッジタイプ

カートリッジで使用されているマッパーと、外部ハードウェアが存在するかどうかを指定します。

Code | Type
-----|------
 $00 | MBC0
 $01 | MBC1
 $02 | MBC1+RAM
 $03 | MBC1+RAM+BATTERY
 $05 | MBC2
 $06 | MBC2+BATTERY
 $08 | MBC0+RAM <sup>[1](#rom_ram)</sup>
 $09 | MBC0+RAM+BATTERY <sup>[1](#rom_ram)</sup>
 $0B | MMM01
 $0C | MMM01+RAM
 $0D | MMM01+RAM+BATTERY
 $0F | MBC3+TIMER+BATTERY
 $10 | MBC3+TIMER+RAM+BATTERY <sup>[2](#mbc30)</sup>
 $11 | MBC3
 $12 | MBC3+RAM <sup>[2](#mbc30)</sup>
 $13 | MBC3+RAM+BATTERY <sup>[2](#mbc30)</sup>
 $19 | MBC5
 $1A | MBC5+RAM
 $1B | MBC5+RAM+BATTERY
 $1C | MBC5+RUMBLE
 $1D | MBC5+RUMBLE+RAM
 $1E | MBC5+RUMBLE+RAM+BATTERY
 $20 | MBC6
 $22 | MBC7+SENSOR+RUMBLE+RAM+BATTERY
 $FC | POCKET CAMERA
 $FD | BANDAI TAMA5
 $FE | HuC3
 $FF | HuC1+RAM+BATTERY

<sup id="rom_ram">1: このオプションを使用しているライセンスカートリッジはありません。正確な動作は不明です。</sup>

<sup id="mbc30">2: RAMサイズが64KBのMBC3とは、ポケモンクリスタルの日本ROMでのみ使用されたMBC30のことです。</sup>

## 0148 - ROMサイズ

カートリッジのROMサイズを示す領域です。基本的に値が`N`ならROMサイズは`32KB << N`になります。

Code | Size      | Amount of banks
-----|-----------|-----------------
 $00 |  32 KB | 2 (ROMバンク無し)
 $01 |  64 KB | 4
 $02 | 128 KB | 8
 $03 | 256 KB | 16
 $04 | 512 KB | 32
 $05 |   1 MB | 64
 $06 |   2 MB | 128
 $07 |   4 MB | 256
 $08 |   8 MB | 512
 $52 | 1.1 MB | 72 <sup>[3](#weird_rom_sizes)</sup>
 $53 | 1.2 MB | 80 <sup>[3](#weird_rom_sizes)</sup>
 $54 | 1.5 MB | 96 <sup>[3](#weird_rom_sizes)</sup>

<sup id="weird_rom_sizes">3: 非公式のドキュメントにのみ記載されています。これらのサイズを使用したカートリッジやROMファイルは知られていません。他のROMサイズはすべて2の累乗なので、これらの値は正確ではないと思われます。これらの値の出典は不明です。</sup>

## 0149 - RAMサイズ

カートリッジのRAMサイズを示す領域です。(カートリッジのRAMは基本的にセーブ用途に用いるので実質セーブデータのサイズです)

Code | Size   | Comment
-----|--------|---------
 $00 | 0      | RAM無し <sup>[4](#mbc2)</sup>
 $01 | -      | 不使用 <sup>[5](#2kib_sram)</sup>
 $02 | 8 KB   | 1バンク
 $03 | 32 KB  | 4バンク
 $04 | 128 KB | 16バンク
 $05 | 64 KB  | 8バンク

 Various "PD" ROMs ("Public Domain" homebrew ROMs generally tagged "(PD)" in the filename) are known to use the $01 RAM Size tag, but this is believed
to have been a mistake with early homebrew tools and the PD ROMs often don't use cartridge RAM at all.

<sup id="mbc2">4: MBC2チップを使用する場合、MBC2には 512×4 bit のRAMが内蔵されていますが、RAM Sizeとして$00を指定する必要があります。</sup>

<sup id="2kib_sram">5: 様々な非公式ドキュメントには2KBと記載されています。しかし、2KBのRAMチップがカートリッジに使われたことはありません。これらの値の出典は不明です。</sup>

## 014A - リージョンコード

このバージョンのゲームが日本で販売されることになっているのか、それ以外の場所で販売されることになっているのかを指定します。2つの値のみが定義されています。

```
  0x00: 日本
  0x01: 日本以外
```

## 014B - ライセンスコード(旧)

ゲーム開発元/販売元コードを`0x00..FF`の範囲で指定します。`0x33`を指定すると、新しいほうのライセンスコード（ヘッダの`0x0144..0145`）が使用されます。

SGBでは、このバイトが`0x33`以外の場合、ゲームボーイ側から送られてきたコマンドパケットを無視します。

ライセンスコードの一覧は[こちら](https://raw.githubusercontent.com/gb-archive/salvage/master/txt-files/gbrom.txt)をみてください。

## 014C - バージョン番号

ゲームのバージョン番号を指定します。たいていのゲームで`0x00`です。意味的にはリビジョン番号のようなものです。

例えば、ポケットモンスター赤緑では、初期版と後期版がありますが、初期版では`0x00`, 後期版では`0x01`が設定されています。

## 014D - ヘッダチェックサム

カートリッジのヘッダバイト(`0x0134..014C`)のチェックサム(8bit)が含まれています。

ブートROMは起動時にチェックサムを次のように計算して、`0x014D`のバイトと比較します。一致しない場合、ブートROMが停止してしまい、ゲームが起動しません。

```cpp
  uint8_t checksum = 0;
  for (uint16_t address = 0x0134; address <= 0x014C; address++) {
    checksum = checksum - rom[address] - 1;
  }
```

## 014E-014F - グローバルチェックサム

カートリッジROM全体のチェックサム(16bit, BE)が含まれています。(ただし、この2バイトはチェックサムの計算には含まれません)

ゲームボーイはこのチェックサムを検証しませんが、ポケモンスタジアム内蔵のGBエミュレータでは、このチェックサムを検証しているようです。

