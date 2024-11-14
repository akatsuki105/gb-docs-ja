# カラーアトリビュートコマンド

## SGBコマンド $04 - ATTR_BLK

画面の色属性を設定するためのコマンドです。

画面内の特定の領域の 内側または外側 の色属性を設定できます。

```
  Byte    Content
  0       0x20+x; ヘッダ (x=Length, 1..7)
  1       データセットの数 (1..18)
  2-7     データセット1
            Byte 0 - Control Code (0-7)
              Bit 0 - 内側の色変更   (1=Yes)
              Bit 1 - 枠部分の色変更 (1=Yes)
              Bit 2 - 外側の色変更   (1=Yes)
              Bit 3-7 - 不使用(0)
              ; 内側(bit0)または外側(bit2)のみを変更すると、枠部分も自動的に同じ色に変更されます。
            Byte 1 - Color Palette Designation
              Bit 0-1 - 内側のパレット番号 (0-3)
              Bit 2-3 - 枠部分のパレット番号 (0-3)
              Bit 4-5 - 外側のパレット番号 (0-3)
              Bit 6-7 - 不使用(0)
            Byte 2-5 - 領域の座標 (Byte2: X1, Byte3: Y1, Byte4: X2, Byte5: Y2)
              X1が左上のX座標、Y1が左上のY座標、X2が右下のX座標、Y2が右下のY座標 (つまり、X1<=X2, Y1<=Y2)
  8-13    データセット2 (任意, ないなら全部0)
  14-15   データセット3 (任意, ないなら全部0)

  ~~~ 以下、データセットが3個以上の場合 ~~~
  16-21   データセット4
  ...
```

データセットが3つ以上の場合、データはさらにパケットに分割して送信されます。最後のパケットの末尾の未使用バイトはゼロにする必要があります。個別のデータセットのフォーマットは以下に説明されています。

## SGBコマンド $05 - ATTR_LIN

特定の行または列の色属性を設定するためのコマンドです。

```
  Byte  Content
  0     0x28+x; ヘッダ (x=Length, 1..7)
  1     データセットの数 (1..110)
  2     データセット1
          Bit 0-4 - 対象の 列(X) or 行(Y)    (bit7でXかYかを指定)
          Bit 5-6 - パレット番号 (0-3)
          Bit 7   - H/V Mode Bit   (0=Vertical line, 1=Horizontal Line)
  3     データセット2 (if any)
  4     データセット3 (if any)
  ...
  15    データセット14 (if any)
```

When sending 15 or more data sets, data is continued in further packet(s). Unused bytes at the end of the last packet should be set to zero. The format of the separate Data Sets (one byte each) is described below. The length of each line reaches from one end of the screen to the other end. In case that some lines overlap each other, then lines from lastmost data sets will overwrite lines from previous data sets.

## SGBコマンド $06 - ATTR_DIV

画面を特定の 行 or 列 で2分割し、それぞれの領域とその分割線に別々の色属性を割り当てるためのコマンドです。

```
  Byte   Content
  0      0x31; ヘッダ
  1      Color Palette Numbers and H/V Mode Bit
           Bit 0-1  Palette Number below/right of division line
           Bit 2-3  Palette Number above/left of division line
           Bit 4-5  Palette Number for division line
           Bit 6    H/V Mode Bit  (0=split left/right, 1=split above/below)
  2      X- or Y-Coordinate (depending on H/V bit)
  3-15   不使用 (0)
```

## SGBコマンド $07 - ATTR_CHR

Used to specify color attributes for separate characters.

```
  Byte  Content
  0     0x38+x; ヘッダ (x=Length, 1..6)
  1     Beginning X-Coordinate
  2     Beginning Y-Coordinate
  3-4   データセットの数 (1-360)
  5     Writing Style   (0=Left to Right, 1=Top to Bottom)
  6     Data Sets 1-4   (Set 1 in MSBs, Set 4 in LSBs)
  7     Data Sets 5-8   (if any)
  8     Data Sets 9-12  (if any)
  etc.
```

When sending 41 or more data sets, data is continued in further packet(s). Unused bytes at the end of the last packet should be set to zero. Each data set consists of two bits, indicating the palette number for one character. Depending on the writing style, data sets are written from left to right, or from top to bottom. In either case the function wraps to the next row/column when reaching the end of the screen.

## SGBコマンド $15 - ATTR_TRN

スーパーファミコンのRAM内の属性ファイル（ATF）を初期化するために使用します。

Each ATF consists of 20×18 color attributes for the Game Boy screen. 

この関数は表示属性に直接影響を与えるものではありません。代わりに、定義済みのATFの1つが、後で`ATTR_SET`または`PAL_SET`を使用して実際の表示メモリにコピーされる場合があります。

```
  Byte  Content
  0     0xA9; ヘッダ
  1-15  不使用 (0)
```

The ATF data is sent by VRAM-Transfer (4KB).

```
 000-FD1  Data for ATF0 through ATF44 (4050 bytes)
 FD2-FFF  不使用
```

Each ATF consists of 90 bytes, that are 5 bytes (20×2 bits) for each of the 18 character lines of the Game Boy window. The two most significant bits of the first byte define the color attribute (0-3) for the first character of the first line, the next two bits the next character, and so on.

## SGBコマンド $16 - ATTR_SET

Used to transfer attributes from Attribute File (ATF) to Game Boy window.

```
  Byte  Content
  0     0xB1; ヘッダ
  1     Attribute File Number ($00-$2C), Bit 6=Cancel Mask
  2-15  不使用 (0)
```

When above Bit 6 is set, the Game Boy screen becomes re-enabled after the transfer (in case it has been disabled/frozen by MASK_EN command). Note: The same functions may be (optionally) also included in PAL_SET commands, as described in the chapter about Color Palette Commands.
