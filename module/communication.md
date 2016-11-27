# モジュールとの通信仕様

## SPI

- コマンドの開始は、CSピンの立ち下がりで検知します。
- コマンドの終了は、CSピンの立ち上がりで検知します。
- 各コマンド送信開始時にCSピンをLowに、終了時にHighにしてください。
- CSピン操作、1byte送信毎に10us程度のdelayを入れてください。
- MSB first

## I2C

- 7ビットアドレスです。
- スレーブアドレス 0x4F
	+ 送信時(9Eh)

	| A7 | A6 | A5 | A4 | A3 | A2 | A1 | R/W |
	|--:|--:|--:|--:|--:|--:|--:|--:|
	| 1 | 0 | 0 | 1 | 1 | 1 | 1 | 0 |


	+ 受信時(9Fh)

	| A7 | A6 | A5 | A4 | A3 | A2 | A1 | R/W |
	|--:|--:|--:|--:|--:|--:|--:|--:|
	| 1 | 0 | 0 | 1 | 1 | 1 | 1 | 1 |