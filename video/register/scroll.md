# 🗺 座標とスクロール

これらのレジスタは、モード3でもアクセス可能ですが、変更がすぐに反映されない場合もあります。(後述)

## FF42 - SCY - Yスクロール (R/W)

256x256ピクセルのBGマップの中で、160x144ピクセルの画面領域の左上のY座標を指定します。0〜255の範囲の値が使用できます。

<img src="../../images/scroll.jpg" alt="scroll" width="240" />

## FF43 - SCX - Xスクロール (R/W)

256x256ピクセルのBGマップの中で、160x144ピクセルの画面領域の左上のX座標を指定します。0〜255の範囲の値が使用できます。

## 描画中にレジスタを変更したときの挙動

The scroll registers are re-read on each tile fetch, except for the low 3 bits of SCX, which are only read at the beginning of the scanline (for the initial shifting of pixels).

All models before the CGB-D read the Y coordinate once for each bitplane (so a very precisely timed SCY write allows “desyncing” them), but CGB-D and later use the same Y coordinate for both no matter what.
