# 制御コマンド

## SGBコマンド $17 - MASK_EN

Used to mask the Game Boy window, among others this can be used to freeze the Game Boy screen before transferring data through VRAM (the SNES then keeps displaying the Game Boy screen, even though VRAM doesn’t contain meaningful display information during the transfer).

```
  Byte  Content
  0     0xB9; ヘッダ
  1     Game Boy Screen Mask (0-3)
          0  Cancel Mask   (Display activated)
          1  Freeze Screen (Keep displaying current picture)
          2  Blank Screen  (Black)
          3  Blank Screen  (Color 0)
  2-15  不使用 (0)
```

Freezing works only if the SNES has stored a picture, that is, if necessary wait one or two frames before freezing (rather than freezing directly after having displayed the picture). The Cancel Mask function may be also invoked (optionally) by completion of PAL_SET and ATTR_SET commands.

## SGBコマンド $0C - ATRC_EN

Used to enable/disable Attraction mode, which is enabled by default.

Built-in borders other than the Game Boy frame and the plain black border have a “screen saver” activated by pressing R, L, L, L, L, R or by leaving the controller alone for roughly 7 minutes (tested with 144p Test Suite). It is speculated that the animation may have interfered with rarely-used SGB features, such as OBJ_TRN or JUMP, and that Attraction Disable disables this animation.

```
  Byte  Content
  0     0x61; ヘッダ
  1     Attraction Disable  (0=Enable, 1=Disable)
  2-15  不使用 (0)
```

## SGBコマンド $0D - TEST_EN

Used to enable/disable test mode for “SGB-CPU variable clock speed function”. This function is disabled by default.

This command does nothing on some SGB revisions. (SGBv2 confirmed, unknown on others)

```
  Byte  Content
  0     0x69; ヘッダ
  1     Test Mode Enable    (0=Disable, 1=Enable)
  2-15  不使用 (0)
```

Maybe intended to determine whether SNES operates at 50Hz or 60Hz display refresh rate ??? Possibly result can be read-out from joypad register ???

## SGBコマンド $0E - ICON_EN

Used to enable/disable ICON function. Possibly meant to enable/disable SGB/SNES popup menues which might otherwise activated during Game Boy game play. By default all functions are enabled (0).

```
  Byte  Content
  0     0x71; ヘッダ
  1     Disable Bits
          Bit0   Use of SGB-Built-in Color Palettes    (1=Disable)
          Bit1   Controller Set-up Screen    (0=Enable, 1=Disable)
          Bit2   SGB Register File Transfer (0=Receive, 1=Disable)
          Bit3-6 不使用 (0)
  2-15  不使用 (0)
```

Above Bit 2 will suppress all further packets/commands when set, this might be useful when starting a monochrome game from inside of the SGB-menu of a multi-gamepak which contains a collection of different games.

## SGBコマンド $0F - DATA_SND

Used to write one or more bytes directly into SNES Work RAM.

```
  Byte  Content
  0     0x79; ヘッダ
  1     SNES Destination Address, low
  2     SNES Destination Address, high
  3     SNES Destination Address, bank number
  4     Number of bytes to write ($01-$0B)
  5-15  Data to write
```

Unused bytes at the end of the packet should be set to zero, this function is restricted to a single packet, so that not more than 11 bytes can be defined at once. Free Addresses in SNES memory are Bank 0 1800-1FFF, Bank $7F 0000-FFFF.

## SGBコマンド $10 - DATA_TRN

Used to transfer binary code or data directly into SNES RAM.

```
  Byte  Content
  0     0x81; ヘッダ
  1     SNES Destination Address, low
  2     SNES Destination Address, high
  3     SNES Destination Address, bank number
  4-15  不使用 (0)
```

The data is sent by VRAM-Transfer (4KB).

```
  000-FFF  Data
```

Free Addresses in SNES memory are Bank 0 1800-1FFF, Bank $7F 0000-FFFF. The transfer length is fixed at 4KBytes ???, so that directly writing to the free 2KBytes at 0:1800 would be a not so good idea ???

## SGBコマンド $12 - JUMP

Used to set the SNES program counter and NMI (vblank interrupt) handler to specific addresses.

```
  Byte   Content
  0      Command*8+Length    (fixed length=1)
  1      SNES Program Counter, low
  2      SNES Program Counter, high
  3      SNES Program Counter, bank number
  4      SNES NMI Handler, low
  5      SNES NMI Handler, high
  6      SNES NMI Handler, bank number
  7-15   不使用 (0)
```

The game Space Invaders uses this function when selecting “Arcade mode” to execute SNES program code which has been previously transferred from the SGB to the SNES. The SNES CPU is a Ricoh 5A22, which combines a 65C816 core licensed from WDC with a custom memory controller. For more information, see “fullsnes” by nocash.

Some notes for intrepid Super NES programmers seeking to use a flash cartridge in a Super Game Boy as a storage server:

- JUMP overwrites the NMI handler even if it is $000000.
- The SGB system software does not appear to use NMIs.
- JUMP can return to SGB system software via a 16-bit RTS. To do this, JML to a location in bank $00 containing byte value $60, such as any of the stubbed commands.
- IRQs and COP and BRK instructions are not useful because their handlers still point into SGB ROM. Use SEI WAI.
- If a program called through JUMP does not intend to return to SGB system software, it can overwrite all Super NES RAM except $0000BB through $0000BD, the NMI vector.
- To enter APU boot ROM, write $FE to $2140. Echo will still be on though.
