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
#### 組み込み型
型             | Types          | Example
---------------|----------------|----------
整数型         | Int            | `let x: Int = 10`
浮動小数点数型 | Float          | `let x: Float = 10.0`
真偽値型       | Boolean        | `let x: Bool = false`
文字列型       | String         | `let x: String = "ten"`
文字型         | Char           | `let x: Char = 'c'`
Unit           | Unit           | `let x: Unit = ()`
Option         | Option         | `let x: Option(int) = Some(10)`
タプル型       | Tuple          | `let x: (int, string) = (10, "ten")`
リスト型       | List           | `let x: List<Int> = [1, 2, 3]`
配列型         | Array          | `let x: Array<Int> = [1, 2, 3]`
関数型         | Functions      | `let x: (int, int) => int = (a, b) => a + b`

#### スコープ
```ocaml
// 無名スコープ
// 代入していない場合は即時実行される
// トップレベルの記述とほぼ同じだが、外側の変数を変更できないなどスコープのルールが適用される
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

// 全てのスコープは戻り値を内包したスコープを返す
// 戻り値は`this`キーワードに含まれる
// 呼び出し後にスコープを定義した場合、続けざまに実行される
two "Hello " "I'm " {
    // ここでの`this` の中身はtwoの戻り値なので`unit`
    one "Ischca"
} // Hello I'm Ischca
```
スコープはScope<T>型で表現される  
よって最も冗長な記述は以下となる
```ocaml
let scope: Scope<Unit> = () {}
```  
また、引数にスコープを受け取ることで、中間に処理を挟むことができる
```ocaml
let scope = function:Scope<Unit> {
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
#### クロージャ
全てのスコープはクロージャでもある
```ocaml
let createCounter = {
  let function = n:Int {
    function n + 1;
  }
  function 0
}

var count = createCounter;
print(count); // 0
print(count); // 1
print(count()); // 2
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
        print numA // 3
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

let scopeB = num:Int {
    print num
}

scopeA
| scopeB 15    // 15

// 引数を省略した場合、`this`が渡される
scopeA {
    numA + 10
}
| scopeB    // 13
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
- 例外処理
- 並列処理
- 型
