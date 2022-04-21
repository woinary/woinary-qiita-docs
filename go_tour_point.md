<!--
title: 私家版 Go言語のポイント
tags: go,POEM,ポエム
-->

「[Welcome to a tour of Go](https://go-tour-jp.appspot.com/list)」のポイントを個人的にまとめたものになります[^1]。

[^1]:単なる個人的まとめなので、ポエム扱いにしています。

:::note
演習問題も自分なりの回答を記載してありますが、Go初心者ですので、あくまで参考程度です。なお、各演習の下の「Go Playground」リンクをクリックすると、Playgroundへ飛んで、実際に動かすことができる、はずです[^2]。
:::

[^2]:これはいつまで保存されるのだろう?]

## 基本

### パッケージ/変数/関数

#### Package

+ Goプログラムはパッケージで構成される
+ mainパッケージが起点となる
+ パッケージ名はパスの最後の部分となる(math/rand→rand)

#### Import

+ importは単独でもグループでも書くことができる

```go :group-import.go
 import(
    "fmt"
    "math"
 )
```

```go :single-import.go
import "fmt"
import "math"
```

#### Exported name

+ 大文字で始まる名前は外部のパッケージから参照できる

逆に言えば、外部に公開するものは大文字で始めないといけない

#### 関数

+ 変数名や関数名の**後ろに**型名を書く
+ 同じ型であれば最後に書けば良い
+ 複数の戻り値が使える
+ 戻り値に名前をつけることができる

基本的な関数宣言

```go :functions.go
func add(x int, y int) int {
    return x + y
}
```

同じ型の省略

```go :functions2.go
func add(x, y int) int {
    return x + y
}
```

複数の戻り値

```go :multi-result.go
func swap(x, y string) (string, string) {
    return y, x
}
```

戻り値に名前をつける

```go :named-result.go
func split(sum int) (x, y int) {
    x = sum * 4 / 9
    y = sum - x
    return
}
```

:::note
後ろに型名を書くのはなかなか慣れません。ついつい、前に書きそうになります。
:::

#### 変数

+ varで宣言（var 変数名 型）
+ 宣言時に初期化する場合は型名を省略可能(型推論される)
+ 関数内では`:=`を使って暗黙の型宣言が可能
+ 基本の型は以下
  + bool
  + string
  + int/int8/int16/int32/int64
  + uint/uint8/uint16/uint32/uint64
  + byte // = uint8
  + rune // = int32 ※ユニコードのコードポイント
  + float32/float64
  + complex64/complex128
+ 初期化しない変数はゼロ値になる（数値は0、論理はfalse、文字列は空）
+ `型名(変数名)`で型変換
+ 定数はconstで宣言

:::note
キャストはついつい型名を`()` で囲みたくなります。
:::

基本的な宣言

```go :variables.go
var c, python, java bool
```

型名の省略

```go :variable-with-initialize.go
var i, j int = 1, 2
var k = 3
```

暗黙の型宣言

```go :short-variable-declarations.go
var i, j = int 1, 2
k := 3
```

### 制御構造(for/if-else/switch/defer)

#### for

+ `()`は書かない、`{}`は必須
+ 条件部は必須、初期化部と後処理部は任意（その場合、`;`は省略可能＝他の言語のwhile文、Go言語にwhile文はない）
+ `for`だけだと無限ループになる

```go :for.go
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
```

whileはない

```go :while.go
sum := 1
for sum < 1000 {
    sum += sum
}
```

無限ループ

```go forever.go
for {
    // ループ内容
}
```

#### if-else

+ `()`は不要、`{}`は必須
+ 条件の前に簡単な文を書くことができる
  + その場合、else部でも使用可能

```go if.go
if x < 0 {
    // x < 0の時の処理
}
```

```gp :if-with-short-statement.go
if v := math.Pow(x, n); v < lim {
    return v
}
```

:::note
変数のスコープはifやforのブロックの中なので注意が必要。特に`if <前置式>; <条件式>`の前置式で宣言した変数（上のサンプルだと変数v）はブロックの外では使えないので、前置式にせずにif文の前に書く必要がある。
ついつい後で使う変数を前置式の中に書いて、後で修正する羽目になる。
:::

#### 演習: ループと関数

```go :exercise-loops-and-functions.go
package main

import (
    "fmt"
)

func Sqrt(x float64) float64 {
   var z float64 = 1.0
   var lastValue = 0.0
   for i := 0; i < 20; i++ {
      z -= (z * z - x) / (2 * z)
      //fmt.Println(i, z)
      if lastValue == z {
         break
      }
      lastValue = z
   }
   return z
}

func main() {
    fmt.Println(Sqrt(5))
}
```

[Go Playground](https://go.dev/play/p/vfqnAQBOGLx)

#### switch

+ breakは不要（当該case部だけ実行される）
+ if同様、条件の前に簡単な文を書くことができる
+ caseは定数でも整数でもなくて良い
+ 最初に該当するcaseだけ実行する
+ switchの後に条件がなくても良い

条件なしのswitch

```go :switch-with-no-condition.go
switch {
case t.Hour() < 12:
    fmt.Println("Good morning!")
case t.Hour() < 17:
    fmt.Println("Good afternoon.")
default:
    fmt.Println("Good evening.")
}
```

#### defer

+ deferを渡した関数の実行を、呼び出し元の関数の終わりまで遅延
+ 複数のdeferはLIFO（つまり、後のものが先）

```go :defer.go
func main() {
    defer fmt.Println("world")

    fmt.Println("hello")
}
```

```:実行結果
hello
world
```

:::note
普通はこんな変な使い方ではなく、例えばOpenしたファイルをCloseするのに使う。
:::

### その他の型(struct,slice,map)

#### ポインタ

+ 型名の前に`*`をつけるとポインタ変数
+ 変数名の前に`&`をつけると参照
+ ポインタ演算はない
+ フィールドへのアクセスは`.`を使う
+ ポインタを使える(`(*p).x`を`p.x`と書くことができる)
+ `名前{初期値}`で初期化可能

:::note
今時の言語に珍しく、ポインタの管理はプログラマに委ねられるので取扱注意。
C系の言語に慣れているとポインタ演算したくなるが、できないので注意。
:::

#### 構造体

+ `type 名前 struct { ... }`で定義
+ `{}`内にフィールド名と型名を書く

構造体の初期化

```go :struct-literals.go
type Vertex struct {
    X, Y int
}

var (
    v1 = Vertex{1, 2}  // has type Vertex
    v2 = Vertex{X: 1}  // Y:0 is implicit
    v3 = Vertex{}      // X:0 and Y:0
    p  = &Vertex{1, 2} // has type *Vertex
)
```

#### 配列

+ 型名の前に`[要素数]`をつけると配列

```go
var a [10]int
```

:::note
配列はあくまで固定長なので要注意。
:::

#### slice

+ 配列の要素数を指定しないものがスライス
+ `スライス名[low:high]`でlowからhighのひとつ前までの部分配列になる
+ 配列のスライスはポインタの様な扱いになる。スライスへの変更は配列への変更になる
+ スライスには長さ(len)と容量(cap)がある
+ `スライス[n:m]`で後ろを切り落とした場合、長さは変わる（減る）が、容量は変わらない（要素は見えないだけで存在する）
+ その場合、長さを伸ばすと元の値が復活する
+ 逆に前を切り落とした場合は長さも容量も変わる（減る）
+ スライスのゼロ値はnilになる
+ スライスはmake関数でも作成可能 `make([]int, 10)`
+ スライスのネストも可能
+ スライスはappend関数で追加が可能

```go
var a []int
```

#### range

+ rangeはスライスやマップの反復処理に使う
+ rangeはインデックスと、その値を返す
+ インデックスや値を使わない場合は`_`で受ける

```go :range.go
for i, v := range pow {
    fmt.Printf("2**%d = %d\n", i, v)
}
````

`_`で受ける

```go
for i, _ := range pow { ... }
for _, value := range pow { ... }
```

インデックスだけでよければ値は省略可能

```go
for i := range pow { ... }
```

#### 演習: スライス

```go :exercise-slices.go
package main

import "golang.org/x/tour/pic"

func Pic(dx, dy int) [][]uint8 {
    // 領域の確保
    pic := make([][]uint8, dy)
    for i := range pic {
        pic[i] = make([]uint8, dx)
    }

    // グラフの描画
    for x := 0; x < dx; x++ {
        for y := 0; y < dy; y++ {
            //pic[y][x] = uint8((x + y) / 2)
            //pic[y][x] = uint8(x * y)
            pic[y][x] = uint8(x ^ y)
        }
    }

    return pic
}

func main() {
    pic.Show(Pic)
}
```

[Go Playground](https://go.dev/play/p/81odeBXxW-Q)

#### map

+ キーと値を関連づける
+ ゼロ値はnil
+ make関数で作成

```go :maps.go
package main

import "fmt"

type Vertex struct {
    Lat, Long float64
}

var m map[string]Vertex

func main() {
    m = make(map[string]Vertex)
    m["Bell Labs"] = Vertex{
        40.68433, -74.39967,
    }
    fmt.Println(m["Bell Labs"])
}
```

mapの要素操作

```go
// 作成
m := make(map[string]int)

// m へ要素(elem)の挿入や更新:
m[key] = elem

// 要素の取得:
elem = m[key]

// 要素の削除:
delete(m, key)

// 要素の確認
elem, ok = m[key] // okがtrueなら存在
```

#### 演習: map

```go :exercise-maps.go
package main

import (
    "golang.org/x/tour/wc"
    "strings"
)

func WordCount(s string) map[string]int {
    items := strings.Split(s, " ")
    
    wc := make(map[string]int)
    
    for _, v := range items {
        wc[v] ++
    }
    
    return wc
}

func main() {
    wc.Test(WordCount)
}
```

[Go Playground](https://go.dev/play/p/Qb7yMURChw3)

:::note
演習問題だけ見てもよくわかりませんでしたが、importしている"golang.org/x/tour/wc"の中身を見て理解できました。
:::

#### function value

+ 関数も変数に格納できる
+ 関数を戻す関数も可能

#### function closures

+ Goの関数はクロージャ、スコープ内の関数外の変数を参照可能で、その値は個別に管理される

#### 演習: フィボナッチ数

```go :exercise-fibonacci-closure.go
package main

import "fmt"

// fibonacci is a function that returns
// a function that returns an int.
// 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610, 987, 1597, 2584, 4181, 6765, 10946,17711, …
func fibonacci() func() int {
    n := 0
    return func() int {
        a, b := 1, 0
        for i:= 0; i < n; i++ {
            a, b = b, a + b
        }
        n++
        return b
    }
}

func main() {
    f := fibonacci()
    for i := 0; i < 20; i++ {
        fmt.Println(f())
    }
}
```

[Go Playground](https://go.dev/play/p/BKRRITz1lev)

## メソッドとインターフェイス

### メソッド

+ Go言語にはクラスはない
+ その代替がメソッド
+ `func レシーバ 関数名() 型名 { ... }`
+ 任意の型にメソッドを宣言可能
+ レシーバとメソッドは同じパッケージ内
+ レシーバはポインタも可能
  + メソッドの先の変数を変更
  + メソッド呼び出し時のコピーを避ける
  + 変数レシーバとポインタレシーバの混在はしない方が良い

メソッド(構造体)

```go :methods.go
type Vertex struct {
    X, Y float64
}

func (v Vertex) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
```

メソッド（他の型）

```go :methods-other.go
type MyFloat float64

func (f MyFloat) Abs() float64 {
    if f < 0 {
        return float64(-f)
    }
    return float64(f)
}
```

メソッド（ポインタレシーバ）

```go :methods-pointer.go
type Vertex struct {
    X, Y float64
}

func (v *Vertex) Scale(f float64) {
    v.X = v.X * f
    v.Y = v.Y * f
}
```

### インターフェイス

+ メソッドのシグニチャの集まり
+ インターフェイスの値は`(値, 型)`というタプル

```go interfaces.go
type Abser interface {
    Abs() float64
}

func main() {
    var a Abser
    f := MyFloat(-math.Sqrt2)
    v := Vertex{3, 4}

    a = f  // a MyFloat implements Abser
    a = &v // a *Vertex implements Abser

    // In the following line, v is a Vertex (not *Vertex)
    // and does NOT implement Abser.
    a = v

    fmt.Println(a.Abs())
}
```

### 演習:Stringers

```go :exercise-stringer.go
package main

import "fmt"

type IPAddr [4]byte

// TODO: Add a "String() string" method to IPAddr.
func (ipaddr IPAddr) String() string {
    return fmt.Sprintf("%v.%v.%v.%v", ipaddr[0], ipaddr[1], ipaddr[2], ipaddr[3])
}

func main() {
    hosts := map[string]IPAddr{
        "loopback":  {127, 0, 0, 1},
        "googleDNS": {8, 8, 8, 8},
    }
    for name, ip := range hosts {
        //fmt.Printf("%v: %v\n", name, ip)
        fmt.Printf("%v: %v\n", name, ip.String())
    }
}
```

[Go Playground](https://go.dev/play/p/XfNTWDutIaZ)

### Error

+ エラーの状態はerror値で表す
+ 関数はerror変数を返す
+ エラーがnilかどうかでハンドリングする

errorインターフェイス

```go
type error interface {
    Error() String
}
```

実装例

```go
i, err := strconv.Atoi("42")
if err != nil { // nilは成功、それ以外はエラー
    fmt.Printf("couldn't convert number: %v\n", err)
    return
}
fmt.Println("Converted integer:", i)
```

### 演習:Error

```go :exercise-errors.go
package main

import (
    "fmt"
)

// エラー
type ErrNegativeSqrt float64

func (e ErrNegativeSqrt) Error() string {
    return fmt.Sprint("cannot Sqrt negative number: ", float64(e))
}

func Sqrt(x float64) (float64, error) {
    var z float64 = 1.0
      var lastValue = 0.0
    
    if x < 0 {
        return 0, ErrNegativeSqrt(x)
    }
    
      for i := 0; i < 20; i++ {
        z -= (z * z - x) / (2 * z)
          //fmt.Println(i, z)
          if lastValue == z {
            break
          }
          lastValue = z
       }
       return z, nil
}

func main() {
    var result float64
    var err error
    
    fmt.Print("sqrt(2):")
    result, err = Sqrt(2)
    if err != nil {
        fmt.Println(err)
    } else {
        fmt.Println(result)
    }
    
    fmt.Print("sqrt(-2):")
    result, err = Sqrt(-2)
    if err != nil {
        fmt.Println(err)
    } else {
        fmt.Println(result)
    }
}
```

[Go Playground](https://go.dev/play/p/Czgp0Q81u7T)

### Reader

+ データストリームからの読み込みはio.Readerインターフェイス
+ Readメソッドを持っている
  + `func (T) Read(b []byte) (n int, err error)`
  + 読み込んだサイズとerrorを返す

### 演習：Reader

```go :exercise-reader.go
package main

import "golang.org/x/tour/reader"
//import "fmt"

type MyReader struct{}

// TODO: Add a Read([]byte) (int, error) method to MyReader.
func (mr MyReader) Read(b []byte) (int, error) {
    //fmt.Printf("%d\n", len(b))
    for i := 0; i < len(b); i++ {
        b[i] = 'A'
    }
    return len(b), nil
}

func main() {
    /*b := make([]byte, 20)
    var size int
    var err error
    var mr MyReader
    size, err = mr.Read(b)
    
    if err == nil {
        fmt.Printf("%s(%d)\n", b, size)
    } else {
        fmt.Println("read error.")
    }*/
    
    reader.Validate(MyReader{})
}
```

[Go Playground](https://go.dev/play/p/8-iBg2u8FDr)

### 演習: rot13Reader

```go :exercise-slices.go
package main

import (
    "fmt"
    "io"
    "strings"
)

type rot13Reader struct {
    r io.Reader
}

func (self rot13Reader) Read(b []byte) (int, error) {
    n, err := self.r.Read(b)
    if err != nil {
        return 0, err
    }
    // DEBUG:処理前
    //fmt.Printf("\nrot13Reader:<%s>(%d:%d) -> ", b, n, len(b))

    // ROT13処理
    // ABCDEFGHIJKLMNOPQRSTUVWXYZ abcdefghijklmnopqrstuvwxyz
    // NOPQRSTUVWXYZABCDEFGHIJKLM nopqrstuvwxyzabcdefghijklm
    for i, c := range b {
        if c >= 'A' && c <= 'M' {
            b[i] += 13
        } else if c >= 'N' && c <= 'Z' {
            b[i] -= 13
        } else if c >= 'a' && c <= 'm' {
            b[i] += 13
        } else if c >= 'n' && c <= 'z' {
            b[i] -= 13
        }
    }

    // DEBUG:処理後
    //fmt.Printf("rot13Reader:<%s>(%d:%d)\n", b, n, len(b))

    return n, nil
}

func main() {
    s := strings.NewReader("Lbh penpxrq gur pbqr!")
    r := rot13Reader{s}
    //io.Copy(os.Stdout, &r)

    b := make([]byte, 1024)
    n, err := r.Read(b)
    if err != nil {
        fmt.Println("read error : " + err.Error())
        return
    }
    fmt.Printf("<%s>(%d)\n", b, n)
}
```

[Go Playground](https://go.dev/play/p/_hnEt3MbR6K)

### イメージ

### 演習: イメージ

```go :exercise-slices.go
package main

import "golang.org/x/tour/pic"
import (
    "image"
    "image/color"
)

type Image struct{}

func (i Image) ColorModel() color.Model {
    return color.RGBAModel
}

func (i Image) Bounds() image.Rectangle {
    return image.Rect(0, 0, 250, 250)
}

func (i Image) At(x, y int) color.Color {
    //v := uint8((x + y) / 2)
    //v := uint8(x * y)
    v := uint8(x ^ y)
    return color.RGBA{v, v, 255, 255}
}

func main() {
    m := Image{}
    pic.ShowImage(m)
}
```

[Go Playground](https://go.dev/play/p/abg7n86Qkj6)

## 並列処理

### Goルーチン

+ Goランタイムが管理する軽量スレッド
+ `go f(x, y, z)`で実行
+ f,x,y,zの評価は実行元、実行は新しいGoルーチン

### チャネル

+ `ch <- v`でvをチャネルchへ送信
+ `v := <- ch`でchから受信した変数をvに割り当て
+ `ch := make(chan int)`の様に事前に生成

### バッファ・チャンネル

+ チャンネルはバッファとして使える（FIFO）
+ チャネルが詰まるとエラーになる

### rangeとclose

+ 送信元は送信が終わったことを示すためにクローズする
+ 受信先は送信が終わったことを第2返り値で確認できる
+ rangeでループすると、自動的に処理してくれる
+ 受信先でチャネルをcloseしない
+ 送信元でのcloseは必須ではない

### select

+ selectのcaseでチャネルを指定した場合、どれか受信できるまでブロックし、準備ができたcaseを実行
+ 複数のcaseの準備が出てきている場合は、どれか1つがランダムで実行される
+ defaultがあればブロックなしで準備ができているcaseを実行、なければそのまま抜ける

### 演習: 2分木の比較

```go :exercise-equivalent-binary-trees.go
package main

import "golang.org/x/tour/tree"
import "fmt"

// Walk walks the tree t sending all values
// from the tree to the channel ch.
func Walk(t *tree.Tree, ch chan int) {
    if t.Left != nil { Walk(t.Left, ch) }
    ch <- t.Value
    if t.Right != nil { Walk(t.Right, ch) }
}

// Same determines whether the trees
// t1 and t2 contain the same values.
func Same(t1, t2 *tree.Tree) bool {
    ch1 := make(chan int)
    ch2 := make(chan int)
    go Walk(t1, ch1)
    go Walk(t2, ch2)
    
    for i := 0; i < 10; i++ {
        if <- ch1 != <- ch2 {
            return false
        }
    }
    return true
}

func main() {
    ch := make(chan int) // チャンネルを生成
    go Walk(tree.New(1), ch)
    
    // Exersize 1,2
    /*
    for i := 0; i < 10; i++ {
        if i != 0 { fmt.Print(",") }
        fmt.Print(<- ch)
    }
    fmt.Println("")*/
    
    // Exercise 3,4
    fmt.Println(Same(tree.New(1), tree.New(1)))
    fmt.Println(Same(tree.New(1), tree.New(2)))
}
```

[Go Playground](https://go.dev/play/p/Lgu-SDjFH6L)

### sync.Mutex(排他制御)

+ `lock()`と`unlock()`で囲むことで排他制御をする

### 演習: Webクローラー

```go :exercise-web-crawler.go
package main

import (
    "fmt"
)

type CrawlResult struct {
    body string
    urls []string
    err  error
}

type Fetcher interface {
    // Fetch returns the body of URL and
    // a slice of URLs found on that page.
    Fetch(url string) (body string, urls []string, err error)
}

// Crawl uses fetcher to recursively crawl
// pages starting with url, to a maximum of depth.
func Crawl(url string, depth int, fetcher Fetcher, cache map[string]*CrawlResult) {
    // TODO: Fetch URLs in parallel.
    // TODO: Don't fetch the same URL twice.
    // This implementation doesn't do either:
    if depth <= 0 {
        return
    }

    ch := make(chan CrawlResult)

    //body, urls, err := fetcher.Fetch(url)
    var result CrawlResult
    if cache[url] == nil {
        go crawlFunc(url, depth, cache, ch)
        result = <-ch
    } else {
        // fmt.Printf("***Cache hit!:%s\n", url)
        return
    }
    if result.err != nil {
        fmt.Println(result.err)
        return
    }
    fmt.Printf("found: %s %q\n", url, result.body)

    for _, u := range result.urls {
        Crawl(u, depth-1, fetcher, cache)
    }

    return
}

func crawlFunc(url string, depth int, cache map[string]*CrawlResult, ch chan CrawlResult) {
    var result CrawlResult
    body, urls, err := fetcher.Fetch(url)
    result.body = body
    result.urls = urls
    result.err = err
    cache[url] = &result
    // fmt.Printf("***cache in %s,%s(%d)\n", url, body, depth)
    ch <- result
}

func main() {
    cache := make(map[string]*CrawlResult)
    Crawl("https://golang.org/", 4, fetcher, cache)
}

// fakeFetcher is Fetcher that returns canned results.
type fakeFetcher map[string]*fakeResult

type fakeResult struct {
    body string
    urls []string
}

func (f fakeFetcher) Fetch(url string) (string, []string, error) {
    if res, ok := f[url]; ok {
        return res.body, res.urls, nil
    }
    return "", nil, fmt.Errorf("not found: %s", url)
}

// fetcher is a populated fakeFetcher.
var fetcher = fakeFetcher{
    "https://golang.org/": &fakeResult{
        "The Go Programming Language",
        []string{
            "https://golang.org/pkg/",
            "https://golang.org/cmd/",
        },
    },
    "https://golang.org/pkg/": &fakeResult{
        "Packages",
        []string{
            "https://golang.org/",
            "https://golang.org/cmd/",
            "https://golang.org/pkg/fmt/",
            "https://golang.org/pkg/os/",
        },
    },
    "https://golang.org/pkg/fmt/": &fakeResult{
        "Package fmt",
        []string{
            "https://golang.org/",
            "https://golang.org/pkg/",
        },
    },
    "https://golang.org/pkg/os/": &fakeResult{
        "Package os",
        []string{
            "https://golang.org/",
            "https://golang.org/pkg/",
        },
    },
}
```

[Go Playground](https://go.dev/play/p/DC6GsN77YsZ)
