Sixel グラフィックス - VT100.net: VT330/VT340 プログラマーリファレンスマニュアル
======================================================================

この文書について
----------------------------------------------------------------------

  * 出典: VT330/VT340 Programmer Reference Manual Volume 2: Graphics Programming
    + 14 Sixel Graphics
    + <http://www.vt100.net/docs/vt3xx-gp/chapter14.html>
  * 日本語訳: 佐藤 文優
    + <https://fumiyas.github.io/>
    + <https://twitter.com/satoh_fumiyasu>
  * 日本語バージョン: 2014-09-08

Sixel とは?
----------------------------------------------------------------------

VT300 は Sixel グラッフィックスデータの送受信に対応しています。
Sixel データでモノクロあるいはカラー (VT340 のみ) のイメージを描画することができます。

Sixel は 6 つの縦列のピクセルの集合です。
ピクセル (画像の要素) は、ビデオスクリーン上に表示可能な最小の単位です。
Sixel はグラフィックスイメージのビットマップデータを表現します。
端末は Sixel データをビットマップ情報として処理します。
ビット値の 1 はピクセルのオン、0 はオフを意味します。

各 Sixel ごとに 1 つの文字コードを使用します。
端末は、Sixel を表す 1 つの
8 ビット文字コードのうち 6 ビットにエンコードされたビットマップデータを用います。

Sixel で文字セットやフォントをデザインして表示することができます。
本マニュアルの Volume 1, Chapter 5 にて、
ソフト文字セットをデザインして端末にロードする手順を解説しています。

Sixel データ形式
----------------------------------------------------------------------

VT300 は、デバイス制御文字列を用いて Sixel イメージを送受信します。

*注意:*
デバイス制御文字列についての情報は本マニュアルの Volume 1, Chapter 2 を参照のこと。

Digital 社のプリンターの多くは制御文字列フォーマットを受け付けます。
以下に対応プリンターの例を挙げます.

  * LA12
  * LA50
  * LA100
  * LA34-VA
  * LN03

プリンターは機種ごとに異なる出力品質を持ちます。
例えば、
ドットマトリックスプリンターはレーザープリンターとはかなり異なります。
端末上で印刷用の Sixel イメージをデザインする場合は、
プリンターに適したパラメーター値を用いなければなりません。
詳細は、お使いのプリンターのプログラマーリファレンスマニュアルを参照してください。

### デバイス制御文字列

デバイス制御文字列の形式は以下の通りです。

| DCS | P1   | `;`  | P2`;` | P3`;`  | `q` | s..s   | ST   |
| --- | ---- | ---- | ----- | ------ | --- | ------ | ---- |
| 9/0 | \*\* | 3/11 | \*\*  | \*\*   | 7/1 | \*\*\* | 9/12 |

DCS は C1 制御文字であり、Sixel データ列の開始を示します。
7 ビット環境では DCS はエスケープシーケンス ESC `P` でも表すことができます。

*P1* はマクロパラメーターであり、
アプリケーションあるいは端末で使用されるピクセルのアスペクト比を示します。
ピクセルのアスペクト比は、
端末がイメージを描画する際に用いるピクセルのドット形状を指示します。
例えば、ドットの縦幅が横幅の 2 倍の場合、アスペクト比は 2:1 になります。
以下は P1 に指定できる値の一覧です。

*注意*:
マクロパラメーターは、既存の
Digital 社製ソフトウェアとの互換性のために用意されています。
新しいアプリケーションは、 P1 は `0` に設定し、
代わりにラスター属性設定コマンド(後述)を使用してください。

| P1            | ピクセルのアスペクト比 (縦:横) |
|-------------- | ------------------------------ |
| 省略          | 2:1 (デフォルト)               |
| `0`, `1`      | 2:1                            |
| `2`           | 5:1                            |
| `3`, `4`      | 3:1                            |
| `5`, `6`      | 2:1                            |
| `7`, `8`, `9` | 1:1                            |

マクロパラメーターの設定は、Sixel データ文字列中でラスター属性文字 (`"`、2/2)
をセットすることで上書きできます。下記を参照のこと。

DCS 文字列中の数値パラメーターはセミコロン `;` (3/11) で区切ります。 

*P2* は背景色の描画方式を選択します。
次の 3 つから 1 つを選択します。

| P2                          | 意味                                                      |
| --------------------------- | --------------------------------------------------------- |
| `0` または `2` (デフォルト) | ピクセルに 0 が指定された場所は現在の背景色で描画される。 |
| `1`                         | ピクセルに 0 が指定された場所は現在の色のままになる。     |

*P3* は水平グリッドサイズのパラメーターです。
水平グリッドサイズは 2 つのピクセルドット間の幅です。
VT300 ではこのパラメーターは無視され、
水平グリッドサイズは 0.0195 cm (0.0075 インチ) に固定されています。

`q` は、このデバイス制御文字列が Sixel コマンドであることを示します。

*s...s* は Sixel エンコードされたデータ文字列です。
**Sixel データ文字** は、
`?` (0x3F) から `~` (0x7E) の範囲の文字です。
各 Sixel データ文字は、縦 6 つ分のピクセルであり、
文字コードの値から 0x3F を引いたバイナリー値を表します。

例:

  * `?` (0x3F) はバイナリー値 000000 を表す。
  * `t` (0x74) はバイナリー値 110101 を表す。
  * `~` (0x7E) はバイナリー値 111111 を表す。

端末はこの 6 ビットの値を *Sixel* (縦の 6 つのピクセル) と解釈します。
最下位のビットが一番上のピクセルになります。

*注意*:
Sixel 文字を作成する手順については、
本マニュアルの Volume 1, Chapter 5「ソフト文字セット」を参照してください。

Sixel データ文字列中には Sixel 制御機能も使用することができます。
次の節で制御文字と機能について解説します。

*ST* は文字列の終端を示します。
ST は C1 制御文字です。
7 ビット環境では ST はエスケープシーケンス ESC `\` でも表すことができます。

Sixel 制御機能
----------------------------------------------------------------------

特殊な機能を実行するには Sixel 制御機能を利用します。
例えば色やラスター属性を選択できます。

### グラフィックスの繰り返しの開始 (`!`)

文字 `!` (2/1) は繰り返しの開始を示します。
以下の形式で指定した回数だけグラフィックスを繰り返し描画することができます。

| `!` | Pn   | 文字 |
| --- | ---- | ---- |
| 2/1 | \*\* | \*\* |

*Pn* は繰り返す回数であり、任意の十進数の値を与えることができます。
例えば `23` を指定すると、続く文字を 23 回繰り返します。

*文字*
は繰り返す文字です。
`?` (0x3F) から `~` (0x7E) の範囲の文字を指定します。

### ラスター属性 (`"`)

文字 `"` (2/2) は、ラスター属性設定コマンドです。
このコマンドで後に続く Sixel データ文字列に対するラスター属性を選択します。
あらゆる Sixel データ文字列の前に使用する必要があります。
コマンド `"` は、前述のマクロパラメーターによるラスター属性よりも優先されます。
コマンドの形式は以下の通りです。

| `"` | Pan  | `;`  | Pad`;` | Ph`;` | Pv   |
| --- | ---- | ---- | ------ | ----- | ---- |
| 2/2 | \*\* | 3/11 | \*\*   | \*\*  | \*\* |

*Pan* と *Pad* は、続く Sixel データ文字列のピクセルのアスペクト比を指示します。
Pan は分子、Pad は分母です。

```
Pan
--- = ピクセルのアスペクト比
Pad
```

ピクセルのアスペクト比は端末が Sixel イメージを描画する際のピクセルの形状となります。

Pan はピクセルの縦幅、Pad は横幅です。
例えば、ドットの縦幅が横幅の 2 倍の場合、Pan には `2` を、Pad には `1` を指定します。

Sixel データ文字列にラスター属性設定コマンド (`"`) を使用する場合、
ピクセルのアスペクト比は必ず指定しなければなりません。
Pan と Pad には整数値のみ指定することができます。
VT300 はピクセルのアスペクト比を最も近い整数値に丸めます。

*Ph* と *Pv* は、イメージの横と縦のサイズを指示します(ピクセル単位)。

Ph と Pv は Sixel データで示されるイメージのサイズを*制限しません*。
しかし、Ph と Pv によって、 イメージデータから背景色となる部分の
Sixel データを省略することが可能となります。また、 
アプリケーションあるいは端末にイメージサイズを通知する手軽な方法でもあります。

*注意*:
VT300 は、P2 が `0` または `2` に設定されている場合、Ph と Pv を使用して背景色で塗り潰します。

### 色指定 (`#`)

文字 `#` (2/3) は色指定の開始です。
色指定には 2 つの形式があります。

  * 色番号で描画するピクセルの色を選択
  * HLS (色相、輝度、彩度) または RGB (赤、緑、青) 形式で色番号に色を割り当てる

### 描画色の選択

以下の形式を使用することで色が割り当てられた色番号を選択することができます。

| `#` | Pc   |
| --- | ---- |
| 2/3 | \*\* |

*Pc* は色番号です(Table 14-1)。

*注意*:
VT330 は 4、VT340 は 16 の色を割り当てることができます。

### HLS / RGB による色の割り当て

以下の形式で HLS または RGB で色を番号に割り当てます。
HLS と RGB のパラメーター値が色として識別されます。

| `#` | Pc   | `;`  | Pu`;` | Px`;` | Py`;` | Pz   |
| --- | ---- | ---- | ----- | ----- | ----- | ---- |
| 2/3 | \*\* | 3/11 | \*\*  | \*\*  | \*\*  | \*\* |

*Pc* は色番号です。

*Pu* は表色系を選択します(HLS または RGB)。

*Px*、*Py*、*Pz* は指定した表色系で色を調合します。

以下の Table 14-1 は有効値の一覧です。

| パラメーター | 有効値       | 定義                   |
| ------------ | ------------ | ---------------------- |
| Pc           | `0` 〜 `255` | 定義する色番号         |
| Pu (必須)    | `1`          | HLS (色相、輝度、彩度) |
|              | `2`          | RGB (赤、緑、青)       |

*注意*:
このパラメーターに続く値は指定した表色系 (HLS または RGB) に依存します。

HLS 値

| Px  | `0` 〜 `360` 度 | 色相 (Hue angle)  |
| --- | --------------- | ----------------- |
| Py  | `0` 〜 `100` %  | 明度 (Lightness)  |
| Pz  | `0` 〜 `100` %  | 彩度 (Saturation) |

RGB 値

| Px  | `0` 〜 `100` % | 赤の明度 |
| --- | -------------- | -------- |
| Py  | `0` 〜 `100` % | 緑の明度 |
| Pz  | `0` 〜 `100` % | 青の明度 |

*注意*:
See the "<A HREF="http://www.vt100.net/docs/vt3xx-gp/chapter2.html#S2.4">Output Mapping</A>" section in <A HREF="http://www.vt100.net/docs/vt3xx-gp/chapter2.html">Chapter 2</A> for a discussion of
shade and color programming.

### グラフィックスの復帰 (`$`)

文字 `$` (2/4) は Sixel 行の終端を示します。
描画位置が現在の Sixel 行の左端に戻ります。
この文字を利用して行を上書きすることができます。

### グラフィックスの改行 (`-`)

文字 `-` (2/13) は Sixel 行の終端を示します。
描画位置が現在の Sixel 行の次行の左端に移動します。

### パラメーターの区切り (`;`)

文字 `;` (3/11) はデバイス制御文字中の数値パラメーターを区切ります。
区切り文字の前に数値が指定されていない場合、
端末はパラメーター値が `0` であると認識します。
区切り文字の後に数字が指定されている場合、
端末はパラメーター値が `0` であると認識します。<!-- FIXME: 意味不明 -->

Sixel スクロールモード
----------------------------------------------------------------------

You can set the sixel scrolling mode by using the *Sixel Scrolling* feature in the
Graphics Set-Up screen. You can also select this mode by using the sixel
display mode (DECSDM) control function.

### Sixel Scrolling Enabled

When sixel display mode is enabled, the sixel active position begins at the
upper-left corner of the ANSI text active position. Scrolling occurs when the
sixel active position reaches the bottom margin of the graphics page. When
sixel mode is exited, the text cursor is set to the current sixel cursor position.

The VT300 sends a sixel next line (`-`) character following a sixel dump. The top
line of the sixel image may scroll off the screen
if (1) your application returns the sixel dump to the terminal, or (2) you perform
a sixel dump to a video terminal connected to the VT300 printer port.

*NOTE*:
You can prevent the sixel image from scrolling off the screen by disabling
the sixel scrolling feature.

### Sixel Scrolling Disabled

When sixel scrolling is disabled, the sixel active position begins at the upper-left
corner of the active graphics page. The terminal ignores any commands
that attempt to advance the active position below the bottom margin of the
graphics page. When sixel mode is exited, the text cursor does not change from
the position it was in when sixel mode was entered.

### Sixel Display Mode Control Function

You can set the sixel scrolling mode by using the sixel display mode
(DECSDM) control function.

When sixel display mode is set, the *Sixel Scrolling* feature is enabled. When
sixel display mode is reset, the *Sixel Scrolling* feature is disabled.

To set DECSDM, the control function is.

| CSI  | `?`  | `8` | `0` | `h` |
| ---- | ---- | --- | --- | --- |
| 9/11 | 3/15 | 3/8 | 3/0 | 6/8 |

To reset DECSDM, the control function is.

| CSI  | `?`  | `8` | `0` | `l`  |
| ---- | ---- | --- | --- | ---- |
| 9/11 | 3/15 | 3/8 | 3/0 | 6/12 |

---
<http://vt100.net/docs/vt3xx-gp/chapter14.html>
