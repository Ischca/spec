# 言語
スコープベースの静的型付け言語
## 特徴
### スコープベース
関数型言語のように、この言語は全ての式が関数であり、全ての関数がスコープである。
#### 単純な例
```ocaml
let text = "Hello"
let greet = print text

greet      // Hello
```
#### FizzBuzz
```ocaml
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
```ocaml
// 代入していない場合は即時関数
{}

// 変数
let foo = "Foo"
// 上記はこのようにも書ける
let bar = { "Bar" }

// 関数でありスコープである
let f = {}
// 呼び出し
f(){}
// ()および{}は省略可能
f

// 引数があるスコープ
// 引数の型は省略できない
// 引数が1つの場合は()を省略できる
let one = arg1:String {
    print arg1
}
// 引数が2つ以上の場合は()で囲む
let two = (arg1:String, arg2:String) {
    print arg1
    print arg2
}
// 引数がある関数の呼び出し
two "Hello, " "World!"    // Hello, World!

// 呼び出し後のスコープ内の記述は続けざまに呼び出される
two "Hello " "I'm " {
    one "Ischca"
} // Hello I'm Ischca
```
スコープはScope<T>型で表現される  
よって、
```ocaml
let scope: Scope<void> = {}
```
のように書くことができる。  
また、引数にスコープを受け取ることで、中間に処理を挟むことができる
```ocaml
let scope function:Scope<void> = {
    println "start"
    function
    println "end"
}

scope {
    println "Hello"
}

---

start
Hello
end

```
#### ネスト
```ocaml
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
#### 合成
```ocaml
let scopeA = {
    let numA = 3
}

let scopeB = {
    let numB = 4
}

scopeA + scopeB {
    print numA // 3
    print numB // 4
}
```
#### 連結
```ocaml
let scopeA = {
    let numA = 3
}

let scopeB = {
    let numB = 4
}

scopeA {
    numA + 10
}
| scopeB {
    print that // 13
}
```
#### 制御構文
```ocaml
// ifスコープ
let n = 1
if n == 0 {
    then {
        "n is 0"
    }
    else {
        "n is not 0"
    }
}   // n is not 0

// matchスコープ
match {
    case n == 0 {
        "n is 0"
    }
    case n == 1 {
        "n is 1"
    }
    else {
        "n is not 0 and not 1"
    }
}   // "n is 1"

```
## マイルストーン
- 型推論
- 