<!--
title:   GoのScanfでunexpected newlineが出た話
tags:    Go,ポエム,POEM
id:
private: true
-->

Goのコーディング練習も兼ねて、P社の、お題に沿って簡単なプログラムを作成するとレーティングされるアレをやってました。
その中で、思いがけず「unexpected newline」エラーが発生してしまいました。
まあ、知っている人は知ってそうなので例によってポエムです。

## 現象

やりたかったのは`0101`みたいな4桁の二進文字列を読み取ること。サクッとScanfを使って以下のようなコードを書きました。

```go:scanf_binary.go
package main

import "fmt"

func main() {
    const COUNT = 2
    var number_decimal int

    for i := 0; i < COUNT; i++ {
    if _, err := fmt.Scanf("%04b", &number_decimal); err != nil {
    fmt.Println(err.Error())
    break
    }
    }
}

```

これを実際に動かすとこうなります。

```sh
% go run scanf_binary.go
0101
unexpected newline
```

## 暫定対応

現象はわかったので、対処です。エラーが「思いがけず改行に出会った」と言っているので、書式文字列に改行を追加してみます。

```go:scanf_binary_newline.go
package main

import "fmt"

func main() {
    const COUNT = 2
    var number int

    for i := 0; i < COUNT; i++ {
    if _, err := fmt.Scanf("%04b\n", &number); err != nil {
    fmt.Println(err.Error())
    break
    }
    }
}
```

今度は問題ありません。

```sh
% go run scanf_binary_newline.go 
0101
1010
```

とりあえず、暫定対応はできましたが、なんなのでしょう？

## 調査

困った時のGoogle頼みで、「go "unexpected newline"」とか検索してみて、以下の記事を見つけました。

https://qiita.com/tutuz/items/020bc05d3957189222e8

こちらは環境依存の話のようですが、その中で

> The scanning code in fmt is cursed. cc @rsc

というコメントが紹介されています。

これをもとに上記の暫定対応をしてみた、というのが正しい経緯になります。

## ふかぼってみる

さて、原因はScanfであるのは確かなようですが、技術者たるものもうちょっと場合分けが必要です。普通に`fmt.Scanf("%d", &count)`みたいなものでは問題がないのですから。

で、色々試しました。今回使った`%04b`がいけないのか、いけないとしたら問題は`b`なのか`04`なのか`4`なのか。色々書き比べて試してみた結果、どうも桁数指定をした時にそれぴったりの入力をすると掲題のエラーになるようです。

```go:scanf_minimal_unexpected_newline.go
package main

import "fmt"

func main() {
    const COUNT = 2
    var number int

    for i := 0; i < COUNT; i++ {
    if _, err := fmt.Scanf("%4d", &number); err != nil {
    fmt.Println(err.Error())
    break
    }
    }
}
```

こちらが実行結果です。

```sh
% go run scanf_minimal_unexpected_newline.go
123
1234
% go run scanf_minimal_unexpected_newline.go
1234
unexpected newline
```

注意が必要なのは、桁数いっぱいいっぱいの入力をした時にエラーが起こるのではなく、次のScanfでエラーが起きることです。

ちなみに、試したのはmacOS環境であり、他の環境ではどうなるか試していません。とりあえずpaiza.ioで試してみても同じ結果でした。

なお、入力する文字列が5桁以上だとこれまたおかしな動作をします。

## そもそもScanfの使い方が悪い

おかしな動作と書きましたが、これは人間の勝手な言い分で、厳密にいうとScanfは悪くない、悪いのは人間（のScanfの使い方）となります。実際、C言語で書いてた頃はscanfってあまり使いませんでした。

"%0d"という指定は、与えられた入力から4桁だけ数字列をとってこい、というものです。ですので`12345`が与えられれば最初のScanfで`1234`、次で`5`を持ってくるのが仕様です。延々と数字が続いているところから指定したブロックを取り出してくるものですから、その間に改行なんてものが入っているのがおかしくて、だから「unexpected newline」と言うのでしょう。
その意味では1行に4桁の数字がある入力を受け付ける際には改行も書式指定文字列に書くと言うのは当然のことでした。反省。

## 正しい対応

と言うわけで、scanf使わない、sscanf使えと昔教わった気がするので、そのように直しました。厳密に言えば、sscanfもダメ派とかも居ますし他のより良い方法もありますが、とりあえず今回はsscanfで妥協しておきます。
入力はbufioにして、改行はトリムします。毎回新たに取得した1行を処理しているので、実は余計な文字があっても全て無視してくれるので、TrimRightは不要だったりします。まあ、一般的に入力行の末尾改行は落として処理するのが普通だったりするので、つけてみました。

```go:sscanf_sample.go
package main

import (
    "bufio"
    "fmt"
    "os"
    "strings"
)

func main() {
    const COUNT = 2
    var number int

    sc := bufio.NewScanner(os.Stdin)

    for i := 0; i < COUNT; i++ {
    sc.Scan()
    line := strings.TrimRight(sc.Text(), "\n")
    if _, err := fmt.Sscanf(line, "%4d", &number); err != nil {
    fmt.Println(err.Error())
    break
    }
    fmt.Println(number)
    }
}
```

もっとも、P社のアレとか競プロ等で毎回こんなコード書くのも面倒なので、絶対正しい仕様の入力が来ると言う前提におんぶに抱っこで`%d`で処理するのが現実的な落とし所ではないかと思います。

## オチはない

というわけで、特にオチはありません。

強いていうならば、以下のようなエラーハンドリングしないコードを書くとサクッと何もわからないままに終わってしまいます。

```go:no_error_handling.go
package main

import "fmt"

func main() {
    const COUNT = 2
    var number int

    for i := 0; i < COUNT; i++ {
    fmt.Scanf("%4d", &number)
    }
}
```

ですので、やはりエラーハンドリングは大事だね、というところです。
P社のアレの場合、入力される値はちゃんと仕様に沿ったキレイな値が与えられる前提があるので、あまりエラーチェックとかハンドリングとかしないのかもしれませんが、自分は回答時間が延びても最低限のエラーチェックやハンドリングは入れるようにしています。ついでにコメントも。

つまり、テストとエラーハンドリング、ちょー大事。

冗談は半分にして、GoのScanfは色々試してC言語のscanfよりは安全そうだったので気楽に使っていましたが、やはり、気をつけないといけない部分が色々あるようです。そう言う意味でも、安易にScanf使うのは失敗だったな、と感じています。

## 蛇足：scanfダメ、絶対

C言語なんかでは昔はよく言われました。ちょっとWeb検索したら以下のような記事を見つけました。

https://detail.chiebukuro.yahoo.co.jp/qa/question_detail/q11131329556

結構、古いものですが今はどうなのでしょう。

絶対ダメとは言いませんけど、scanf使った上であれこれ対応を考えるよりは、素直に他の今時の入出力ライブラリ使って処理する方が問題は少なくなる、という点は同意です。どうせあれこれするなら複雑で挙動を追いきれないscanf使う意味なくね、と。

P社のアレとか競プロとか、絶対にキレイな入力が来ることを期待しても良いところで使うのはまあ良いのではないかと思います。ただ、ユーザが何入力するかわからんところで使うのは危ないし、昔は先輩がダメ出ししてくれました。今、どうなんでしょうね。
