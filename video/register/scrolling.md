# ðº åº§æ¨ã¨ã¹ã¯ã­ã¼ã«

ãããã®ã¬ã¸ã¹ã¿ã¯ãã¢ã¼ã3ã§ãã¢ã¯ã»ã¹å¯è½ã§ãããç¾å¨ã®ã¹ã­ã£ã³ã©ã¤ã³ãçµäºããã¾ã§ã¯å½±é¿ãåã¼ãã¾ãããå½±é¿ãç¾ããã®ã¯æ¬¡ã®ã¹ã­ã£ã³ã©ã¤ã³ããã§ãã

## FF42 - SCY - Yã¹ã¯ã­ã¼ã« (R/W)

256x256ãã¯ã»ã«ã®BGãããã®ä¸­ã§ã160x144ãã¯ã»ã«ã®ç»é¢é åã®å·¦ä¸ã®Yåº§æ¨ãæå®ãã¾ãã0ã255ã®ç¯å²ã®å¤ãä½¿ç¨ã§ãã¾ãã

![](../../images/jsgb-gpu-bg-scrl.jpg)

## FF43 - SCX - Xã¹ã¯ã­ã¼ã« (R/W)

256x256ãã¯ã»ã«ã®BGãããã®ä¸­ã§ã160x144ãã¯ã»ã«ã®ç»é¢é åã®å·¦ä¸ã®Xåº§æ¨ãæå®ãã¾ãã0ã255ã®ç¯å²ã®å¤ãä½¿ç¨ã§ãã¾ãã

## FF44 - LY - Yåº§æ¨ã¬ã¸ã¹ã¿ (R)

LYã¯ãç¾å¨ã®ã¹ã­ã£ã³ã©ã¤ã³ãç¤ºãã0ãã153ã¾ã§ã®ä»»æã®å¤ãæã¤ãã¨ãã§ãã¾ãã

LYã144ãã153ã¾ã§ã®å¤ã«åã¾ã£ã¦ããã¨ãã¯ãVBlankæéã§ãããã¨ãç¤ºãã¦ãã¾ãã

## FF45 - LYC - LYæ¯è¼ã¬ã¸ã¹ã¿ (R/W)

ã²ã¼ã ãã¼ã¤ã¯ãLYCã¬ã¸ã¹ã¿ã¨LYã¬ã¸ã¹ã¿ã®å¤ãå¸¸ã«æ¯è¼ãã¾ãã

ä¸¡æ¹ã®å¤ãåãã§ããå ´åãSTATã¬ã¸ã¹ã¿ã®`LYC=LY`ãã©ã°(bit6)ãã»ãããããï¼æå¹ãªå ´åã¯ï¼STATå²ãè¾¼ã¿ããªã¯ã¨ã¹ãããã¾ãã

## FF4A - WY - ã¦ã£ã³ãã¦Yåº§æ¨ (R/W)

ã¦ã£ã³ãã¦ã®å·¦ä¸ã®Yåº§æ¨ãæå®ãã¾ãã

ã¦ã£ã³ãã¦ã¯ãéå¸¸ã®èæ¯ããæéã«è¡¨ç¤ºãããããä¸ã¤ã®èæ¯é åã®ãããªãã®ã§ããOBJ ã¯éå¸¸ã® BG ã¨åæ§ã«ã¦ã£ã³ãã¦ã®æåãã¾ãã¯å¥¥ã«è¡¨ç¤ºããã¾ãã

ä¾ãã°ãä¸ã®ç»åã§ã¯ãã¡ãã¥ã¼ç»é¢(ç»é¢ä¸é¨)ã«ã¦ã£ã³ãã¦ãä½¿ã£ã¦ãã¾ãã

![](../../images/GBwindow_zelda.png "https://wentwayup.tamaliver.jp/e448453.html ããå¼ç¨")

XYåº§æ¨ã®ä¸¡æ¹ãããããWX=0..166ãWY=0..143ã®ç¯å²ã«ããã¨ãã«ãã¦ã£ã³ãã¦ãæå¹(LCDCã®bit5)ãªãã¦ã£ã³ãã¦ãç»é¢ã«è¡¨ç¤ºããã¾ãã

WX=7ãWY=0ã®å ´åãã¦ã£ã³ãã¦ã¯ç»é¢ã®å·¦ä¸ã«éç½®ãããèæ¯ãå®å¨ã«è¦ãã¾ãã

WXã0..6ããã³166ã®ã¨ãã¯ããã¼ãã¦ã§ã¢ã®ãã°ã«ããæåã«ä¿¡é ¼æ§ãããã¾ãããWXã0ã«è¨­å®ããã¨ãSCXãå¤åããã¨ãã«ã¦ã£ã³ãã¦ãæ°´å¹³æ¹åã«ä¹±ãã¾ãã

`SCX%8`ã®å¤ã«å¿ãã¦ãåä½ãå°ãè¤éã«ãªãã®ã§ãèªåã§è©¦ãã¦ã¿ã¦ãã ããã

## FF4B - WX - ã¦ã£ã³ãã¦Xåº§æ¨+7 (R/W)

ã¦ã£ã³ãã¦ã®å·¦ä¸ã®Xåº§æ¨ãæå®ãã¾ãã

è©³ç´°ã¯WYã®ã»ããè¦ã¦ãã ããã


