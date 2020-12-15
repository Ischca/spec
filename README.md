# 言語
スコープベースの静的型付け言語
## 特徴
### スコープベース
関数型言語のように、この言語は全ての式が関数であり、全ての関数がスコープである。
#### 単純な例
```
let text = { "Hello" }
let greet = print text

greet      // Hello
```
#### FizzBuzz
```
let fizzBuzz = n:Int {
    if n > 1 then fizzBuzz n-1

    if n%15 == 0 then println "FizzBuzz"
    else if n%5 == 0 then println "Buzz"
    else if n%3 == 0 then println "Fizz"
    else println n
}

fizzBuzz 20
```
#### スコープ
```
// 空のスコープ、何もしない
{}

// これは関数でありスコープである
let f = {}
// 呼び出し
f(){}
// ()および{}は省略可能
f

// 引数があるスコープ
// 引数の型は省略できない
// 引数が1つの場合は()を省略できる
let f2 = arg1:String {
    print
}
// 引数が2つ以上の場合は()で囲む
let f3 = (arg1:String, arg2:String) {
    print arg1
    print arg2
}
// 引数がある関数の呼び出し
f3 "Hello, " "World!"    // Hello, World!

// 呼び出し後のスコープ内の記述は続けざまに呼び出される
f3 "Hello " "I'm " {
    print "Ischca"
} // Hello I'm Ischca

```
#### ネスト
```
let scopeA = {
    let numA = 3
}

let scopeB = {
    let numB = 4
}

scopeA {
    print numA // 3

    scopeB {
        print numA // Error
        print numB // 4
    }
}
```
#### スコープの合成
```
let scopeA = {
    let numA = 3
}

let scopeB = {
    let numB = 4
}

scopeA + scopeB {
    print(numA) // 3
    print(numB) // 4
}
```
#### 制御構文
```
// ifスコープ
// ifはthenスコープとelseスコープを持っている
let n = 1
if n == 0 {
    then {
        "n is 0"
    }
    else {
        "n is not 0"
    }
}   // n is not 0

// forスコープ


```
## マイルストーン
- 型推論
- 