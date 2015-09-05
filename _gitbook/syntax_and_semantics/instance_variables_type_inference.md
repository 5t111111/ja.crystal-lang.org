# インスタンス変数と型推論

これまでの例の中で、`Person` の `@name` と `@age` に対して特に型の指定をしていなかったことに気づいたでしょうか。その理由は、コンパイラには自動的に型を推論する機能があるからです。

例えば、以下のように書いたとします。

```ruby
class Person
  getter name

  def initialize(@name)
    @age = 0
  end
end

john = Person.new "John"
john.name #=> "John"
john.name.length #=> 4
```

ここで、`Person.new` を実行するときにその引数に `String` 型を与えています。そうすると、コンパイラが `@name` も 同様に`String` 型にしてくれます。

もし、`Person.new` の引数が別の型だった場合は、`@name` もそれに応じて別の型になります。

```ruby
one = Person.new 1
one.name #=> 1
one.name + 2 #=> 3
```

これらのプログラムに `hierarchy` コマンドでコンパイラを起動すると、推論された型を階層的に表示することができます。1つの例の場合:

```
- class Object
  |
  +- class Reference
     |
     +- class Person
            @name : String
            @age  : Int32
```

2つ目の例の場合:

```
- class Object
  |
  +- class Reference
     |
     +- class Person
            @name : Int32
            @age  : Int32
```

それでは、もし2つの people を作るとき、一方は `String` 型でもう一方は `Int32` 型にした場合にはどうなるでしょうか？試してみましょう。

```ruby
john = Person.new "John"
one = Person.new 1
```

`hierarchy` コマンドでコンパイラを起動すると、結果は以下となります。

```
- class Object
  |
  +- class Reference
     |
     +- class Person
            @name : (String | Int32)
            @age  : Int32
```

`@name` の型が `(String | Int32)` となっていますね。これは、`String` と `Int32` 型の「組み合わせ」を意味しています。このように、コンパイラは `@name` が与えられたすべての型を持つようにしています。

このケースでは、コンパイラは `@name` は常に `String` か `Int32` のいずれかの型であるとして解釈します。したがって、もしその「両方」の型に存在しないメソッドが呼び出された場合、コンパイル時にエラーが発生します。

```ruby
john = Person.new "John"
one = Person.new 1

# Error: undefined method 'length' for Int32
john.name.length

# Error: no overload matches 'String#+' with types Int32
john.name + 3
```

さらに、後から変数の型を変更されるときには、変更する前の時点でも同様のエラーが発生します。

```ruby
john = Person.new "John"
john.name.length
one = Person.new 1
```

これはコンパイル時エラーとなります。

```
Error in foo.cr:14: instantiating 'Person:Class#new(Int32)'

one = Person.new 1
             ^~~

instantiating 'Person#initialize(Int32)'

in foo.cr:12: undefined method 'length' for Int32

john.name.length
          ^~~~~~
```

つまり、コンパイラの型推論はグローバルに働き、クラスやメソッドの誤った使用を常に検知できるようになっています。さらに一歩進んで、`def initialize(@name : String)` のように書くことで型を制約することも可能です。ただ、こうすると、コードが少し冗長になり、汎用性を欠いたものになってしまいます。`Person` の `name` を常に `String` として扱っている限り、`String` の「インターフェース」を持つ型で `Person` のインスタンスを作成すれば、すべては問題なく動作するでしょう。

もし、`@name` が `Int32` である `Person` 型と、`@name` が `String` である `Person` 型の2つを使いたい場合は、 [ジェネリクス](generics.html) を利用すべきです。

## Nil を許容する (nilable) インスタンス変数

もし、あるインスタンス変数が、クラスで定義されているすべての `initialize` で初期化されなかったとき、そのインスタンス変数は `Nil` 型を持つと解釈されます。

```ruby
class Person
  getter name

  def initialize(@name)
    @age = 0
  end

  def address
    @address
  end

  def address=(@address)
  end
end

john = Person.new "John"
john.address = "Argentina"
```

このプログラムの階層グラフは以下となります。

```
- class Object
  |
  +- class Reference
     |
     +- class Person
            @name : String
            @age : Int32
            @address : String?
```

`@address` が `String?` となっています。これは `String | Nil` の短縮表記です。このとき、以下はコンパイル時エラーが発生します。

```ruby
# Error: undefined method 'length' for Nil
john.address.length
```

`Nil` や組み合わせ型を扱うときには、[if var](if_var.html)/[if var.is_a?](if_varis_a.html)/[case](case.html)/[is_a?](is_a.html) を利用することができます。

## Catch-all initialization

インスタンス変数を `initialize` メソッドの外側で初期化することもできます。

```ruby
class Person
  @age = 0

  def initialize(@name)
  end
end
```

上記の例では、`@age` はすべてのコンストラクタで0に初期化されます。これは、重複を避けることができるだけではなく、クラスを再オープンしてインスタンス変数を追加する際に`Nil` 型になるのを防ぐことにも役立ちます。

## インスタンス変数の型を指定する

ときには、インスタンス変数の型をコンパイラに固定してもらいたいときもあるでしょう。その場合は、`::` を使って型を指定できます。

```ruby
class Person
  @age :: Int32

  def initialize(@name)
    @age = 0
  end
end
```

こうしておくと、`@age` に `Int32` ではない値を代入しようとしたときに、その場所でコンパイル時エラーが発生します。

ただ、型に対する「デフォルト」の値というものは存在しないので、この場合においても、catch-all initializer か `initialize` メソッドでインスタンス変数の初期化を行う必要があることを覚えておいてください。
