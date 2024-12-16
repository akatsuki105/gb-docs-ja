# DMA転送(OMA DMA)

HDMAは[別のページ](../../cgb/hdma.md)で説明しています。

ゲームボーイではROMまたはRAMからOAMへのDMA転送が行えます。

## FF46 - DMA - DMA制御レジスタ (R/W)

このレジスタに書き込むと、ROMまたはRAMから OAMへのDMA転送が開始されます。

```
  Bit
  0-7  転送元アドレス上位バイト (0x00..DF)
         このレジスタに0xXXを書き込んだ場合
           転送元: 0xXX00..XX9F
           転送先: 0xFE00..FE9F
```

転送にかかる時間は160Mサイクルで、通常速モードでは640ドット(1.4ライン), 倍速モードでは320ドット(0.7ライン)です。

## OAM DMA bus conflicts

DMGでは、DMA転送中、CPUはHRAM（`0xFF80..FFFE`）にのみアクセスできます。なのでゲーム側はHRAMにこのレジスタへの書き込み処理と転送が終わるまでビジーウェイトする処理をあらかじめ書いて実行することがほとんどです。

CGBでは、カートリッジとWRAMは別々のバス上にあります。つまり、CPUはWRAMからの転送中にROMまたはSRAMにアクセスでき、またはROMまたはSRAMからの転送中にWRAMにアクセスできます。しかし、`call`はリターンアドレスをスタックに書き込み、スタックと変数は通常WRAMにあるため、CGBでもDMAが終了するまでHRAMでビジーウェイトすることを推奨します。

> 割り込み  
> An interrupt writes a return address to the stack and fetches the interrupt handler’s instructions from ROM. Thus, it’s critical to prevent interrupts during OAM DMA, especially in a program that uses timer, serial, or joypad interrupts, since they are not synchronized to the LCD. This can be done by executing DMA within the VBlank interrupt handler or through the di instruction.

While an OAM DMA is in progress, the PPU cannot read OAM properly either. Thus, most programs execute DMA during Mode 1, inside or immediately after their VBlank handler. But it is also possible to execute it during display redraw (Modes 2 and 3), allowing to display more than 40 objects on the screen (that is, for example 40 objects in the top half, and other 40 objects in the bottom half of the screen), at the cost of a couple lines that lack objects. If the transfer is started during Mode 3, graphical glitches may happen.

The details:
- If OAM DMA is active during OAM scan (mode 2), most PPU revisions read each object as being off-screen and thus hidden on that line.
- If OAM DMA is active during rendering (mode 3), the PPU reads whatever 16-bit word the DMA unit is writing to OAM when the object is fetched. This causes an incorrect tile number and attributes for objects already determined to be in range.

## Best practices

この10バイトのルーチンは転送を開始し、それが終了するまで待ちます。多くのゲームでは、このようなルーチンをHRAMにコピーし、モード1中に呼び出します。

```assembly
run_dma:
    ld a, HIGH(start address)
    ldh [$FF46], a  ; start DMA transfer (starts right after instruction)
    ld a, 40        ; delay for a total of 4×40 = 160 M-cycles
.wait
    dec a           ; 1 M-cycle
    jr nz, .wait    ; 3 M-cycles
    ret
```

HRAMが不足している場合は次のようにすることでHRAMを5バイト節約できます。(数Mサイクル遅くなります)

```assembly
run_dma:          ; This part must be in ROM.
    ld a, HIGH(start address)
    ld bc, $2846  ; B: wait time; C: LOW($FF46)
    jp run_dma_tail


run_dma_tail:     ; This part must be in HRAM.
    ldh [c], a
.wait
    dec b
    jr nz, .wait
    ret z         ; Conditional `ret` is 1 M-cycle slower, which avoids
                  ; reading from the stack on the last M-cycle of DMA.
```

If starting a mid-frame transfer, wait for Mode 0 first so that the transfer cleanly overlaps Mode 2 on the next two lines, making objects invisible on those lines.
