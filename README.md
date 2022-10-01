# 言語
スコープベースの静的型付け言語

## 組み込み型
型             | Types          | Example
---------------|----------------|----------
整数型         | Int            | `val x: Int = 10`
浮動小数点数型 | Float          | `val x: Float = 10.0`
真偽値型       | Boolean        | `val x: Bool = false`
文字列型       | String         | `val x: String = "ten"`
文字型         | Char           | `val x: Char = 'c'`
Unit           | Unit           | `val x: Unit = ()`
Option         | Option         | `val x: Option(int) = Some(10)`
タプル型       | Tuple          | `val x: (int, string) = (10, "ten")`
リスト型       | List           | `val x: List<Int> = [1, 2, 3]`
配列型         | Array          | `val x: Array<Int> = [1, 2, 3]`
関数型         | Function      | `val x: (int, int) => int = (a, b) => a + b`

## 特徴
### スコープベース
関数型言語のように、この言語は全ての式が関数であり、全ての関数がスコープである。
全てのスコープは第一級オブジェクトである。

### 制限を与える

#### コンテキストバインド
特定のコンテキストを持つスコープからしか呼び出すことが出来ないスコープを作成できる。
```ocaml
ctx Transactional = <ds: Datasource, conn: Connection>
val with_transactional = fn: () => (Int, String) {
    conn.begin
    with Transactional fn
    | match {
        case Ok conn.commit
        case Err conn.rollback
    }
    conn.close
}

val studentDaoScope = @transactional {
    StudentDao conn
}

val save = @transactional student: Student {
    studentDaoScope { dao ->
        if student.id == null {
            then dao.insert student
            else dao.update student
        }
    }
}

with_transactional {
    Student "Takeshi" "221569" { student ->
        save student
        | print
    }
}

```

#### 提供の制限
特定の型に定義された関数以外の呼び出しを制限するスコープを作成できる。
```ocaml


```


#### 単純な例
```ocaml
print "Hello, World!"
```
#### FizzBuzz
```ocaml
val fizzBuzz = n:Int {
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
// 無名スコープ
// 代入していない場合は即時実行される
// トップレベルの記述とほぼ同じだが、外側の変数を変更できないなどスコープのルールが適用される
{}

// 変数
val foo = "Foo"
// 上記はこのようにも書ける
val bar = { "Bar" }

// 関数でありスコープである
val f = {}
// 呼び出し
f{}
// {}は省略可能
f

// 引数があるスコープ
// 引数の型は省略できない
val one = arg1:String {
    print arg1
}
// 引数が2つ以上の場合はスペースで区切る
val two = arg1:String arg2:String {
    print arg1
    print arg2
}
// 引数がある関数の呼び出し
two "Hello, " "World!"    // Hello, World!

// 全てのスコープは戻り値を内包したスコープを返す
// 戻り値は`context`キーワードに含まれる
// 呼び出し後にスコープを定義した場合、続けざまに実行される
two "Hello " "I'm " {
    // ここでの`this` の中身はtwoの戻り値なので`unit`
    one "Ischca"
} // Hello I'm Ischca
```
スコープはScope<T>型で表現される  
よって最も冗長な記述は以下となる
```ocaml
val scope: Scope<Unit> = {}
```
また、引数にスコープを受け取ることで、中間に処理を挟むことができる
```ocaml
val scope = function:Scope<Unit> {
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
val createCounter = {
  val function = n:Int {
    function n + 1;
  }
  function 0
}

var count = createCounter;
print count; // 0
print count; // 1
print count; // 2
```

#### ネスト
```ocaml
val scopeA = {
    val numA = 3
}

val scopeB = {
    val numB = 4
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
val scopeA = {
    val numA = 3
}

val scopeB = {
    val numB = 4
}

scopeA + scopeB {
    print numA // 3
    print numB // 4
}
```
#### 連結
```ocaml
val scopeA = {
    val numA = 3
}

val scopeB = num:Int {
    print num
}

scopeA
| scopeB 15    // 15

// 引数を省略した場合、`context`が渡される
scopeA {
    numA + 10
}
| scopeB    // 13
```
#### 制御構文
```ocaml
// ifスコープ
val n = 1
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
#### 内包表記
```ocaml
for (x <- (1 to 10) if x %2 ==0) yield x    // Scala
[x for x in range(1, 11) if x%2 == 0]       # Python
[x | x <- [1..10], x `mod` 2 == 0]          -- Haskell
// [2, 4, 6, 8, 10]
```
`処理 繰り返し条件(or 配列) 呼出し条件`で記述する
```ocaml
{{it} [1..10] {true}}
```

`繰り返し条件`以外は省略可能  
以下の場合、各要素が加工されず、条件に従って偶数のみのリストが返る
```ocaml
{[1..10] {it % 2 == 0}}
// [2, 4, 6, 8, 10]
```
以下の場合、各数値が2乗されたリストが返る
```ocaml
{{it * it} [1..10]}
// [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

`繰り返し条件`は配列である必要は無い  
以下の例では、RPCへの接続が成功するまで無限に繰り返している
```ocaml
{{it - 1 + it - 2} {sleep 1000 | fix {getRpc | isOk}} {it > 1}}
{
    {it - 1 + it - 2}
    {sleep 1000 | fix {getRpc | isOk}}
    {it > 1}
}

```
ここで、fix関数は以下の定義とする
```ocaml
val fix = <T> f: () -> T { f fix g }
```

```ocaml
// [[1, 2], [3, 4], [5, 6]]
// -> [1, 3. 5]
// -> [2, 4, 6]

val l = {{it} [1..6] {}} // [[1, 2], [3, 4], [5, 6]]
{{it} {l | }}
```

## マイルストーン
- 型推論
- 例外処理
- 並列処理
- 型
- メタプログラミング
- 継続
- セルフホスト
- 