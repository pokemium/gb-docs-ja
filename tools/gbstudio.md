# GB Studio

```
Note: この記事は筆者(pokemium)が昔描いたGBStudioの内部を説明するQiitaの記事を移植したものです。
```

## 本文

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/250476/db7ae8fc-b68b-7fd4-24d7-f22af00eebe8.png)

> gb-studio: A quick and easy to use drag and drop retro game creator for your favourite handheld video game system

GBStudio はプログラミング不要でGUI上でパーツをドラッグ&ドロップしていくことで、ゲームボーイのソフトを開発できる、最近流行りの NoCode（ノーコード） なソフトです。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/250476/815cce0b-ff06-8f7e-80d7-bd46d9ef45c7.png)

chrismaltby さんが作っているOSSで、ソフトウェアの開発には React+Redux と electron を使用しています。

[Github](https://github.com/chrismaltby/gb-studio), [公式サイト](https://www.gbstudio.dev/)

こちらの動画が、実際にゲームを作っている様子の参考になると思います。

[GB Studio Tutorial 1: Getting Started](https://youtu.be/hNXlV2tt7eE)


今回は GBStudio のバージョン1.2.1 の内部構造を解説していき、どのようにしてGUIソフトで作ったゲームがゲームボーイのソフトにビルドされるのかみていこうと思います。

## 💾 GBソフトについて

あまり難しく捉えなくて大丈夫です。

普通のコンピュータやOSなどで、elf形式など特別なフォーマットのファイルが実行ファイルとして実行できるように、ゲームボーイのソフト(以下 GBソフト)の実態はゲームボーイというコンピュータ上で実行できるフォーマットの実行ファイル(以下 GBファイル)です。

GBファイルを専用の記憶ディスク(CDのようなもの)に書き込んだのがGBのカセットです。

<img width="200px" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/250476/4cab80fb-5995-31f7-90df-7c593ab61688.png" />


GBStudioが吐き出すのは、このGBファイルです。作ったGBファイルは市販のフラッシュカートリッジに書き込めば、GBカセットとしてGB本体で遊ぶことができます。

## 🧰 GBDK

GBStudioのワークフローをみていく前に知っておいて欲しいツールがあります。それが [GBDK](http://gbdk.sourceforge.net/) です。

GBDKは GB Development kit の略でGBファイルを作るためのSDKです。

GBDK は 内部にGBのアーキテクチャ向けにC言語をコンパイルできる組み込みデバイス向けのCコンパイラ SDCC を内包しており、 GBDK が提供する線やバンク切り替え、ボタン入力などGB開発に便利なAPIとコンパイラを使って比較的お手軽にGBファイルをC言語で作成することができます。

## 📁 ディレクトリ構成

GBStudioのレポジトリのディレクトリ構成はおおまかに次のようになっています。

- appData/ 
- buildTools/
- src/

### appData

エミュレータやゲームエンジンなど、作成したゲームを遊ぶのに必要なものが入っています。

 ファイル  |  内容
---- | ----
 js-emulator/  |  [JS製のGBエミュレータ](https://github.com/taisel/GameBoy-Online)をフォークして改造したもの 記事後半で説明
 src/  |  C言語を解釈するGB用のゲームエンジン<br/>[ZGB](https://github.com/Zal0/ZGB)を改造したもの
 templates/  |  サンプルROMのアセットデータが入っている

### buildTools

GBDK が丸ごと入っています。 上述の `appData/src/Makefile` から呼び出されGBファイルをビルドするのに利用されます。

### src

GBStudioの本体です。

Reactのディレクトリ構成をメインとしており、electronのビルドに対応しています。

Javascriptメインで書かれています。

#### npmモジュール

利用しているnpmモジュールについて

 npmモジュール  |  概要
---- | ----
react | アプリ構築
redux | ステート管理
gbdkjs | 未使用
ggbgfx | 画像をGameBoyで利用するグラフィックフォーマットに変換するライブラリ
electron-notarize | Apple公証をパスするためのコード署名ライブラリ
about-window | electronのウィンドウメニューに"About This App"を簡単に導入するためのモジュール

#### ライブラリ

利用しているライブラリについて  
lib/以下に存在

 ライブラリ  |  概要
---- | ----
l10n | 多言語対応
ggbgfx | 画像をGameBoyで利用するグラフィックフォーマットに変換するライブラリ

#### ファイル一覧

 ファイル  |  内容
---- | ----
actions/ | reduxのActionを定義
assets/ | GBStudioのアプリで使うアセットを定義
components/ | Presentational Component(見た目を担当)
containers/ | Container Component(ロジックを担当)
hooks/ | Electron Forgeのhooks (Reactのhooksとは無関係)
lang/ | 多言語対応
lib/ | npmパッケージ化されていないスクリプトや独自のライブラリ
middleware/ | reduxのmiddlewareを定義
reducers/ | reduxのReducerを定義
store/ | reduxのStoreを定義
styles/ | CSS
windows/ | プロジェクトの土台となるhtmlを定義している？
consts.js | 定数定義
index.js | electronアプリ本体を定義
menu.js | electronアプリのウィンドウのメニューバーを定義

## 💎 アセット

GBStudioで作れるGBソフトは version1.2.1 時点ではRPG風のゲームのみになっています。(version2ではもっと増える模様)

GBStudioではユーザーは以下のゲーム内のアセットを組み合わせてゲームを作っていくことになります。

- tileset タイルという8*8pxのグラフィックデータの集合体
- background タイルを組み合わせて作った背景データ
- sprite 人物などのオブジェクト
- scene backgroundとspriteを組み合わせた実際のゲーム画面
- string 会話などのテキストデータ
- music 音声データ
- script イベントなどのスクリプト処理

## ⛏ Workflow

アプリの操作からゲームボーイのソフト(以下 GBソフト)になるまでのワークフローです。

```
アプリ -> gbsproj -> JSのオブジェクト -> Cのコード -> GBファイル
```

ビルドツールとビルド対象がフローの中でごっちゃになっていますが、そこはご容赦ください。

### 1. アプリ -> gbsproj

gbsprojは GBStudio の中で使われている jsonファイル で、ユーザーがGUI操作を通じて作成したゲームは全部このファイルに特別なJSONフォーマットで書き込まれていきます。

```json-doc
{
    "scene": [
        {
            "id": "ID",
            "name": "シーン名",
            "backgroundId": "背景のタイルデータID",
            "x": -1,
            "y": -1,
            "width": "画面X幅のタイル数",
            "height": "画面Y幅のタイル数",
            "actors": [{
                    "id": "ID",
                    "spriteSheetId": "スプライトのタイルデータID",
                    "x": "ActorのいるX座標(タイル単位)",
                    "y": "ActorのいるY座標(タイル単位)",
                    "movementType": "動くかどうか",
                    "direction": "向いている方向",
                    "script": [
                        // Aボタンを押すと次のイベントを実行する
                        {
                            "id": "7ba26d3c-c1ca-46c8-a4ae-ed0a431b1d03",
                            "command": "EVENT_TEXT", // テキスト表示
                            "args": {
                                "text": "話掛けるとしゃべるテキスト1枚目"
                            }
                        },
                        {
                            "id": "1cf687c1-e1fe-4e89-a7ef-9ddc6412ef4f",
                            "command": "EVENT_TEXT", // テキスト表示
                            "args": {
                                "text": "話掛けるとしゃべるテキスト2枚目"
                            }
                        },
                        {
                            "id": "a50d16e9-2660-4fde-94c2-825e6fe799ee",
                            "command": "EVENT_ACTOR_SET_POSITION", // Actorの位置を変更
                            "args": {
                                "actorId": "対象のActor",
                                "x": "X座標(タイル)",
                                "y": "Y座標(タイル)"
                            }
                        },
                        {
                            "id": "f5569ecd-cf2f-43e0-93ff-f6ccc1392f41",
                            "command": "EVENT_END" // イベント終了
                        }
                    ],
                    "frame": 0
                }],
            "triggers": [],
            "collisions": [0, 0, 0, 0], 
            "script": []
        }
    ]
}
```

gbsprojファイルを保存しておけば、たとえ端末が変わってもgbsprojをインポートすれば続きからゲーム開発が再生できます。

### 2. gbsproj -> JSのオブジェクト

gbsproj は JSONファイルなので、そのままJSのオブジェクトとしてパースすればそのまま使えますが、次のステップでC言語のコードに変換しやすいように、GBDKのC言語にそったフォーマットに変換していきます。

この工程は `src/lib/compiler/compileData.js` の `precompile` という関数で行われます。

### 3. JSのオブジェクト -> Cのコード

gbsprojの内容をJSオブジェクト化(precompile)したら、GBファイルにGBDKがコンパイルできるように C言語のソースファイルを吐き出す作業になります。

ここで注意したいのは、GBStudioでは、GBDKのAPIをさらにラップしたものをゲームエンジンとして利用しています。(詳しくは割愛。[ZGB](https://github.com/Zal0/ZGB)というGBDKをラップしたゲームエンジンをさらにカスタマイズしたもの appData/src にファイルの実体があります。)

これらの処理は `src/lib/compiler/compileData.js` の中の `compile` で行われています。  上述の `precompile` 処理はこの関数の先頭で呼ばれています。

`compile` では、

1. BankedDataインスタンス を作成
2. `compileEntityEvents` で precompileされたオブジェクトをバイト列に変換
3. BankedDataインスタンスの `push`メソッドでバイト列を適したバンクに仕分けしていく
4. 最後にC言語のコードとして出力して終了

という手順でコンパイルが行われる。

実際に中を覗いてみると、

```js
exportCData() {
    return (
      this.data.map((data, index) => {
        const bank = this.dataWriteBanks[index];
        const bankData = cIntArray(`bank_${bank}_data`, data);
        return `#pragma bank=${bank}\n\n${bankData}\n`;
      }).filter(i => i)
    );
  }
```

など確かにC言語のコードを出力している部分があります。

### 4. C言語のコード -> GBファイル

この時点で、C言語のソースファイルが得られています。

あとは、上述のGBDKでGBファイルにCをコンパイルするだけ...というわけにはならないです。

GBStudioでは、GBDKをさらにラップした独自のゲームエンジンを使っています。

このゲームエンジンの規定したフォーマットでC言語を書くことで、ゲームエンジンとGBDKを合わせてGBソフトをビルドすることが可能になっています。

このゲームエンジンのためのC言語は、かなりフォーマットが単純です。

そのおかげでJSからCのコードを吐き出すことが、割と単純にできるようになっています。

実際に中身をみてみましょう。

```
.
├── src
│   ├── bank_6.c
│   ├── bank_7.c
│   └── data_ptrs.c
├── music
│   └── music_7ced013b0.c
└── *.c, *.s
```

src/ と music/ 以外はゲームエンジン部分です。 バンク切り替えやスプライトの衝突処理、スクロールなど、低レベルなハードウェア操作を担当してくれています。 

OSに近いものだと考えるといいかもしれません。

src/ と music/ にユーザーが作成したゲームアセットやスクリプト処理が記述されており、それらをゲームエンジン部分がゲームにしています。

GBStudioが単純なRPGゲームしか作成できないのは、このゲームエンジン部分がRPGゲームにしか対応していないのが理由になっています。

#### src

- `bank_6.c`, `bank_7.c` 各バンクのデータ
- `data_ptrs.c` 各アセットデータへのポインタ

バンク5まではゲームエンジンに使用されるため、ユーザーが作成したアセットデータのために利用されるのはバンク6以降からとなっています。 まずは `bank_6.c`の中身をみていきましょう。

```bank_6.c
#pragma bank=6

const unsigned char bank_6_data[] = {
0x00,0x08,0x01,0x28,0x00,0x00,0x01,0x00,0x00,0x00,...};
```

見ての通りただのバイトデータです。　この中にユーザーの定義したアセットデータや、スクリプト処理が格納されているのですが、人間がパッとみただけではわかりません。 `bank_7.c`も同様にバイトデータが並んでいるだけです。 

ファイル先頭の`#pragma bank=6` という記述はCコンパイラに、このファイルのデータは GBソフトのBank6部分に配置してねということを伝える役割を持っています。

次に `data_ptr.c`をみていきましょう。

```data_ptr.c
#pragma bank=5
#include "data_ptrs.h"
#include "banks.h"

const unsigned char (*bank_data_ptrs[])[] = {
0,0,0,0,0,0,&bank_6_data,&bank_7_data
};

const BANK_PTR tileset_bank_ptrs[] = {
{0x06,0x0561},{0x06,0x1152}
};

const BANK_PTR background_bank_ptrs[] = {
{0x06,0x16D3},{0x06,0x183E},{0x06,0x19A9},{0x06,0x1B14},{0x06,0x1C7F},{0x06,0x2082},{0x06,0x2485},{0x06,0x25F0}
};

const BANK_PTR sprite_bank_ptrs[] = {
{0x06,0x3C93},{0x06,0x3CD4},{0x06,0x3D95},{0x06,0x3DD6},{0x06,0x3E57},{0x06,0x3F58},{0x07,0x0000},{0x07,0x00C1},{0x07,0x0182},{0x07,0x0303},{0x07,0x0484},{0x07,0x04C5},{0x07,0x0506},{0x07,0x0547},{0x07,0x05C8},{0x07,0x0609}
};

const BANK_PTR scene_bank_ptrs[] = {
{0x07,0x064A},{0x07,0x074C},{0x07,0x07DC},{0x07,0x085E},{0x07,0x08F5},{0x07,0x092B},{0x07,0x0961},{0x07,0x0A36}
};

const BANK_PTR string_bank_ptrs[] = {
{0x06,0x01EC},{0x06,0x0202},{0x06,0x0222},{0x06,0x0241},{0x06,0x0248},{0x06,0x026C},{0x06,0x028A},{0x06,0x02A6},{0x06,0x02CA},{0x06,0x02E7},{0x06,0x030B},{0x06,0x031D},{0x06,0x033C},{0x06,0x0365},{0x06,0x0387},{0x06,0x03A5},{0x06,0x03C7},{0x06,0x03E6},{0x06,0x040D},{0x06,0x0429},{0x06,0x0440},{0x06,0x045A},{0x06,0x047C},{0x06,0x049F},{0x06,0x04B2},{0x06,0x04C9},{0x06,0x04F0},{0x06,0x0512},{0x06,0x051C},{0x06,0x053F}
};

const BANK_PTR avatar_bank_ptrs[] = {
{0x00,0x0000}
};

const unsigned char * music_tracks[] = {
music_7ced013b0_Data, 0
};

const unsigned char music_banks[] = {
8, 0
};

unsigned char script_variables[16] = { 0 };

```

`data_ptr.c`は、上述した `bank_6.c`や `bank_7.c`のバイト列のバンク番号やオフセットを保持することです。

これは、オブジェクトや、スクリプト処理など、ゲーム内アセットのオフセットです。

これのおかげで無意味だったバイト列をゲームエンジンは意味のあるものとして解釈することが可能になっています。

例えば、 変数`tileset_bank_ptrs` は タイルセット(タイルという8*8pxのグラフィックデータの集合体) が、bank_6_dataの 0x0561番目 と 0x1152番目から始まることを意味しています。

#### music

音楽データが入っています。

srcで使用したバンクの次のバンクから配置されます。

#### ビルド

これらのC言語のプロジェクトフォルダに対して、 `src/lib/compiler/makeBuild.js` の `makeBuild`関数を呼び出すことでビルドを行います。

`makeBuild`関数では、 `appData/src/gb/Makefile`用の makeコマンドを作ってそれを実行することで、 GBDK内の Cコンパイラによって GBファイルが出力されます。

長くなりましたがこれにてGBファイルの完成です！

## 💻 ブラウザ対応

GBStudioは、GBファイルとしてゲームを吐き出すだけでなく、作ったゲームをブラウザで遊んでもらうことも可能にしています。

`makeBuild`関数には `buildType`という引数があり、これに "web"を渡す(実際にはユーザーがGUIで指定)と作ったゲームがブラウザで遊べるようになります。

これは `appData/js-emulator`の中にあるブラウザ上で動くGBエミュレータで作成したGBファイルを動かすことでブラウザ上で作ったゲームを遊べるようにしています。

## 🍬 gbdkjs

見たところ、現在は使われていませんでしたが package.jsonの dependencies に入っているため昔使われていたと思われる `gbdkjs`についてもついでに調べてみました。

これは、GBStudioと同じく、 chrismaltby さんが作った npmパッケージです。 ([Github](https://github.com/gbdkjs/gbdkjs))

`gbdkjs` は レポジトリの説明書きにも `Emscripten bindings for GBDK` と書いてあるように、GBDK向けのC言語で書かれたGBソフトのソースコードを、ブラウザ上で実行できるJavascriptにトランスパイルするものです。 おそらく昔は、ブラウザ上でのGBソフトの実行にこちらを使っていたと思われますが、ライセンスかなんらかの理由で素直なエミュレータ方式に修正したのだと思います。

ワークフローとしては

1. emcc: GBDK仕様のC -> LLVM bitcode
2. emcc: LLVM bitcode -> Javascript

となっています。

このとき吐き出されるJavascriptは、gbdkjsに内包されたGBエミュレータのようなもので動くようになっています。

ユーザーからはあたかもC言語のソースコードがJavascriptのWebAppになったように見えますが、正直、中途半端なJSのコードとエミュレータを使うなら、普通にブラウザ上で動く正確なGBエミュレータに GBファイルそのまま食わした方がいいのではと思いました。(作者もそう思ったのでしょうか...)

## 🎁 最後に

プログラミング不要でGUI操作だけからGBソフトが作れるという魔法のようなソフトですが、中をみてみると意外と単純でした。

RPGという形式に特化したゲームエンジンのおかげでC言語のフォーマットがかなり単純になっているのが、個人的には重要なポイントかなと思います。

GBStudio2.0ではRPG以外にもシューティングゲームなどの2Dアクションゲームに対応するようになっていて、おそらく2.0へのアップデートでゲームエンジン部分を改良したのでしょう。

ゲームエンジン部分は、[ZGB](https://github.com/Zal0/ZGB)というゲームエンジンをカスタマイズして作ったようです。

また、それなりに大きいプロジェクトなのに、Typescriptを使っていないのも少し驚きました。

GBStudio2.0も時間があればみてみようと思いました。

