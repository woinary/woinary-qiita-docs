<!--
title:   Swiftで文字列中の数値を取り出したかった件
tags:    Swift,ポエム,POEM
id:
private: false
-->

Swiftのコーディング練習も兼ねて、P社の、お題に沿って簡単なプログラムを作成するとレーティングされるアレをやってました。その中で、文字列中の数値を取り出そうと正規表現を使ってパッとやろうとしたら意外に苦労したので、簡単にまとめます。

:::note 
とくに目新しい話もないので、ポエムタグをつけておきます。Qiitaで検索する際は`-POEM`とかするとポエムタグを除外できるのでご活用ください。
:::

## やりたいこと

やりたいことは簡単です。`abc123`みたいな文字列を与えられたら、そこから数値として`123`を取り出したい、というものです。
正規表現でチャチャっと取り出せば良いかと思ったのですが、やってみたら意外と色々面倒でした。結局、他の方法を使ってしまいました。

他の言語で言えば、こんな感じで書けばサクッとできる程度のことです。

```go: getInt.go
package main

import (
  "fmt"
  "regexp"
  "strconv"
)

func getIntByRegexp(target string) (int, error) {
  regexDecimal := regexp.MustCompile("[^0-9]*([0-9]+)[^0-9]*")
  matchString := regexDecimal.ReplaceAllString(target, "$1")
  n, err := strconv.Atoi(matchString)
  if err != nil {
    return 0, err
  }
  return n, nil
}

func main() {
  numStr := "abc123"
  if num, err := getIntByRegexp(numStr); err == nil {
    fmt.Println(num)
  } else {
    fmt.Println(err.Error())
  }
}

```

今時の言語なんだから、これくらいサクッと書きたかったのですが、Swiftに慣れないせいでだいぶ苦労しました。

## 回答例

正規表現を使う方法の他、いくつか思いついたので挙げてみます。

### 1文字ずつ見る

文字列を頭から1文字ずつループして、数字を見つけたら抜き出します。手抜きしたので、`abc12d3`みたいなものも`123`が返ってきます。今回、数値を取り出すということしか仕様として考えていませんので、気にしないことにします。お題としては`abc123`みたいに前半はアルファベット、後半は数字と決まっており、混在しないことになっています。

```swift: getIntByIndex.swift
func getIntByIndex(string: String) -> Int {
  var num = 0
  for i in 0..<string.count {
    let si = string.index(string.startIndex, offsetBy: i)
    if string[si] > "0" && string[si] < "9" {
      num = num * 10 + Int(String(string[si]))!
    }
  }
  return num
}
```

目的は果たせますが、なんかこうもうちょっとサクッと取り出したいものです。

### Componentsを使う

正規表現が使えなかったのでWeb検索して見つけた方法です。
`String.components()`を使います。これはいわゆるsplitにあたるもので、`seperatedBy`で指定したものを区切り文字として分解するものですが、ここでは`CharacterSet.decimalDigits.inverted`を指定しています。つまり、10進数値を表す文字でないもの、を指定しています。そうすると、`abc`の部分は区切り文字扱いになるので、`["", "123"]`と分解されます。それをfilterで空のものを除外すると`["123"]`になりますので、mapで数値に変換します。これは配列なので、先頭要素をリターンして終わりになります。

```swift: getIntByComponents.swift
func getIntByComponents(string :String) -> Int {
  let resultCompoents = string
    .components(separatedBy: CharacterSet.decimalDigits.inverted)
    .filter { $0 != ""}
    .map {Int($0) ?? 0}
  return resultCompoents[0]
}
```

なお、わざわざfilterとmapを使わないでもjoinedをjoined()を使う手もあります。これが一番スッキリします。

```swift: getIntByComponents.swift
func getIntByComponents(string :String) -> Int {
  let resultCompoents = string.components(separatedBy: CharacterSet.decimalDigits.inverted).joined()
  return Int(resultCompoents) ?? 0
}
```

### 正規表現を使う

最後に、当初思いついた正規表現を使う方法です。なんか、正規表現とか文字列切り出しの仕様が面倒臭く感じるものになっており、やりたいことはシンプルなのに、ごちゃごちゃする感じです。もっとうまい書き方があるのでしょうか。

```swift: getIntByRegexp.swift
func getIntByRegexp(string: String) -> Int {
  let regexDecimal = try! NSRegularExpression(pattern: "[0-9]+")
  let resultMatch = regexDecimal.firstMatch(in: string, options: [], range: NSRange(location: 0, length: string.count))!
  let start = string.index(string.startIndex, offsetBy: resultMatch.range(at: 0).location)
  let end = string.index(start, offsetBy: resultMatch.range(at: 0).length)
  return Int(String(string[start..<end])) ?? 0
}
```

Swiftの文字列の扱いはバージョンで仕様変更が色々あったようで、Webで調べた結果を使おうと思ったら、それは古いバージョンのやり方で現在は使えない、みたいなことが多くて情報を探すのに苦労しました。
return文のところでも切り出した文字列はsubstring型なので、一旦、Stringに変換してから、Intに変換するといった、面倒なものになってます。

## 振り返り

Go版もちょっと冗長ですが、これはエラー処理を入れてるからかと思います。その部分を省くともうちょっとスッキリします。この程度のことを書きたいだけなのに、なんであれだけごちゃごちゃするのだろう、とかついつい思ってしまいます。

```go: getIntBad.go
package main

import (
  "fmt"
  "regexp"
  "strconv"
)

func getIntByRegexp(target string) (int, error) {
  regexDecimal := regexp.MustCompile("[^0-9]*([0-9]+)[^0-9]*")
  matchString := regexDecimal.ReplaceAllString(target, "$1")
  n, _ := strconv.Atoi(matchString)
  return n, nil
}

func main() {
  numStr := "abc123"
  if num, err := getIntByRegexp(numStr); err == nil {
    fmt.Println(num)
  }
}
```

## 参考情報

色々Web検索しましたが、主に以下を参考にさせていただきました。情報提供ありがとうございました。

https://www.2nd-walker.com/2020/04/21/swift-how-to-use-nsregularexpression/

https://qiita.com/ayutaso/items/288bdc0d2c175b725c86

https://qiita.com/shtnkgm/items/83e88f230366adfad8e8
