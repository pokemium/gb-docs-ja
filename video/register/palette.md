# ð¨ ãã¬ãã

## ã¢ãã¯ã­ãã¬ãã(éCGBã¢ã¼ãã®ã¿)

### FF47 - BGP - BGãã¬ãããã¼ã¿ (R/W)

ãã®ã¬ã¸ã¹ã¿ã¯ãèæ¯ã¿ã¤ã«ã¨ã¦ã£ã³ãã¦ã¿ã¤ã«ã®è²çªå·ã«å¯¾ãã¦ã°ã¬ã¼ã·ã§ã¼ããå²ãå½ã¦ã¾ãã

- bit0-1: è²çªå·0ã®è²ãã¼ã¿
- bit2-3: è²çªå·1ã®è²ãã¼ã¿
- bit4-5: è²çªå·2ã®è²ãã¼ã¿
- bit6-7: è²çªå·3ã®è²ãã¼ã¿

è²ãã¼ã¿ã¯

- 0: ç½
- 1: ç½ç°
- 2: é»ç°
- 3: é»

ã«ãªã£ã¦ãã¾ãã

ä¾ãã°ã2bppãã©ã¼ãããã§ãã¯ã»ã«ã2ãç¤ºããå ´åãBGPã®bit4-5(è²çªå·2)ãè¦ã¾ãã  
ããã§bit4-5ã3ãç¤ºãã¦ããã¨ãããªãå¯¾å¿ããè²ãã¼ã¿ã¯é»ãªã®ã§ãã®ãã¯ã»ã«ã®è²ã¯é»ããªãã¾ãã

### FF48 - OBP0 - OBJãã¬ãããã¼ã¿0 (R/W)

ãã®ã¬ã¸ã¹ã¿ã¯ããã®ãã¬ãããä½¿ç¨ããOBJã®è²çªå·ã«ã°ã¬ã¼ã·ã§ã¼ããå²ãå½ã¦ã¾ãã

OBP0ã¨OBP1ã®ãã¬ããã®ã©ã¡ããä½¿ç¨ãããã¯OAMã®å±æ§/ãã©ã°ã®bit4ã§åãæ¿ãããã¾ãã

ã¹ãã©ã¤ãã®è²çªå·0ã¯éæãæå³ãããããä¸ä½2ããããç¡è¦ããããã¨ãé¤ãã°ãBGPã¨åãããã«åä½ãã¾ãã

### FF49 - OBP1 - OBJãã¬ãããã¼ã¿1 (R/W)

OBP0ã¨åå®¹ã¯åãã§ãã

OAMã®å±æ§/ãã©ã°ã®bit4ã1ã®å ´åã¯ãã¡ãã®ãã¬ãããä½¿ããã¾ãã

## ã«ã©ã¼ãã¬ãã(CGBã¢ã¼ãã®ã¿)

CGBã§ã¯64ãã¤ã(0x40ãã¤ã)ã®ãã¬ããç¨ã®ã¡ã¢ãªãªããã®ããBGç¨ã¨OBJç¨ã«åè¨2ã¤å­å¨ãã¾ãããããã¯ã¢ãã¬ã¹ã¨ç´ã¥ãããã¦ããããéå¸¸ã®ã¡ã¢ãªã¢ã¯ã»ã¹ã§ã¯ã¢ã¯ã»ã¹ã§ãã¾ããã

ããã§å©ç¨ãããã®ãããããèª¬æãããã¬ã¸ã¹ã¿ãã¡ã§ãã

BG/OBJãã¬ããã¤ã³ããã¯ã¹ã¯ããã¬ããã¡ã¢ãªã®ã¢ãã¬ã¹ãæå®ãã¾ãã(ãã³ã¯çªå·ã«è¿ã)

BG/OBJãã¬ãããã¼ã¿ã¯ããã¬ããã¤ã³ããã¯ã¹ã§æå®ãããã¬ããã¡ã¢ãªã®åå®¹ããããã³ã°ããã¾ãããã®ã¬ã¸ã¹ã¿ãèª­ã¿æ¸ããããã¨ã§ãã¬ããã¡ã¢ãªã«ã¢ã¯ã»ã¹ãããã¨ãã§ãã¾ãã

### FF68 - BCPS/BGPI - BGãã¬ããã¤ã³ããã¯ã¹

ãã®ã¬ã¸ã¹ã¿ã¯ãCGBã«ããã¦BGãã¬ããã¡ã¢ãªã®ã¢ãã¬ã¹ãæå®ããããã«å©ç¨ãã¾ãã

ä¸ã§è¿°ã¹ãããã«BGãã¬ããã¡ã¢ãªã¯å¨é¨ã§64ãã¤ãã§ã1ã¤8ãã¤ãã®ãã¬ãããã¼ã¿ã8ã¤æ ¼ç´ããã¦ãã¾ãã(BGP0~BGP7)

```
0   BGP0
        - è²çªå·0ã®è²ãã¼ã¿(2ãã¤ã, å¾è¿°)
        - è²çªå·1
        - è²çªå·2
        - è²çªå·3
8   BGP1
        - è²çªå·0 
        - è²çªå·1
        - è²çªå·2
        - è²çªå·3
16  BGP2
        - è²çªå·0 
        - è²çªå·1
        - è²çªå·2
        - è²çªå·3
    ...
56  BGP7
        - è²çªå·0 
        - è²çªå·1
        - è²çªå·2
        - è²çªå·3
```

```
Bit 7     ãªã¼ãã¤ã³ã¯ãªã¡ã³ã  (0=ç¡å¹, 1=æ¸ãè¾¼ã¿å¾ã«ã¤ã³ã¯ãªã¡ã³ãçºç)
Bit 5-0   ã¤ã³ããã¯ã¹ (00-3F)
```

BGãã¬ããã¡ã¢ãªã®ã¤ã³ããã¯ã¹ãæå®ããå¾ã¯ãå¾è¿°ã®ã¬ã¸ã¹ã¿FF69ãä»ãã¦ãBGãã¬ãããã¼ã¿ãèª­ã¿æ¸ããããã¨ãã§ãã¾ãã

ãªã¼ãã¤ã³ã¯ãªã¡ã³ã(bit7)ãã»ããããã¦ããå ´åãFF69ã¸ã®æ¸ãè¾¼ã¿ãã¨ã«ã¤ã³ããã¯ã¹ãèªåçã«ã¤ã³ã¯ãªã¡ã³ãããã¾ããFF69ããã®èª­ã¿åºãæã«ã¯ãªã¼ãã¤ã³ã¯ãªã¡ã³ãã®å¹æã¯ãªãã®ã§ããã®å ´åã¯æåã§ã¤ã³ããã¯ã¹ãã¤ã³ã¯ãªã¡ã³ãããå¿è¦ãããã¾ããã¬ã³ããªã³ã°ä¸­ã«FF69ã«æ¸ãè¾¼ãã§ãããªã¼ãã¤ã³ã¯ãªã¡ã³ãã¯çºçãã¾ãã

### FF69 - BCPD/BGPD - BGãã¬ãããã¼ã¿

ãã®ã¬ã¸ã¹ã¿ã¯ï¼ã¬ã¸ã¹ã¿`$FF68`ãä»ãã¦ã¢ãã¬ã¹æå®ãããCGBã®èæ¯ãã¬ããã¡ã¢ãªã¸ã®ãã¼ã¿ã®èª­ã¿æ¸ããå¯è½ã«ãã¾ãã

åè²ã¯2ãã¤ãã§å®ç¾©ããã¾ããbit0-7ã1ãã¤ãç®ãbit8-15ã2ãã¤ãç®ã§ãã

```
 Bit 0-4   èµ¤   (00 - 1F)
 Bit 5-9   ç·   (00 - 1F)
 Bit 10-14 é   (00 - 1F)
```

### FF6A - OCPS/OBPI - OBJãã¬ããã¤ã³ããã¯ã¹

BGãã¬ããã¨åå®¹ã¯åãã§ãã

### FF6B - OCPD/OBPD - OBJãã¬ãããã¼ã¿

BGãã¬ããã¨åå®¹ã¯åãã§ãã

ãã ãè²0ã¯éæãè¡¨ããã¨ã«ã¯æ³¨æãã¦ãã ããã

## RGB Translation by CGBs

![](../../images/VGA_versus_CGB.png)

When developing graphics on PCs, note that the RGB values will have different appearance on CGB displays as on VGA/HDMI monitors calibrated to sRGB color. Because the GBC is not lit, the highest intensity will produce Light Gray color rather than White. The intensities are not linear; the values 10h-1Fh will all appear very bright, while medium and darker colors are ranged at 00h-0Fh.

The CGB displayâs pigments arenât perfectly saturated. This means the colors mix quite oddly; increasing intensity of only one R,G,B color will also influence the other two R,G,B colors. For example, a color setting of 03EFh (Blue=0, Green=1Fh, Red=0Fh) will appear as Neon Green on VGA displays, but on the CGB itâll produce a decently washed out Yellow. See the image above.

## RGB Translation by GBAs

Even though GBA is described to be compatible to CGB games, most CGB games are completely unplayable on older GBAs because most colors are invisible (black). Of course, colors such like Black and White will appear the same on both CGB and GBA, but medium intensities are arranged completely different. Intensities in range 00h..07h are invisible/black (unless eventually under best sunlight circumstances, and when gazing at the screen under obscure viewing angles), unfortunately, these intensities are regularly used by most existing CGB games for medium and darker colors.

Newer CGB games may avoid this effect by changing palette data when detecting GBA hardware (see how). Based on measurement of GBC and GBA palettes using the 144p Test Suite ROM, a fairly close approximation is GBA = GBC * 3/4 + 8h for each R,G,B intensity. The result isnât quite perfect, and it may turn out that the color mixing is different also; anyways, itâd be still ways better than no conversion.

This problem with low brightness levels does not affect later GBA SP units and Game Boy Player. Thus ideally, the player should have control of this brightness correction.
