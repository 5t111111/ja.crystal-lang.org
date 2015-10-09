# 型文法

以下の場合、

* [型制約]の指定(type_restrictions.html)
* [型引数]の指定(generics.html)
* [変数の宣言](declare_var.html)
* [エイリアス]の宣言(alias.html)
* [typedef] の宣言(type.html)
* 擬似呼び出し [is_a?] (is_a.html) の引数
* [as](as.html) 式の引数
* [sizeof](sizeof.html) 式の引数
* [instance_sizeof](instance_sizeof.html) 式の引数
* メソッドの[戻り値の型](return_types.html)

一般的な型に対する便利な記法が用意されています。これらは特に [C バインディング](c_bindings/index.html)を書くときに有効ですが、上記した箇所であればどこでも利用することができます。

## パスとジェネリクス

一般的な型とジェネリクスは以下のように利用します。

```ruby
Int32
My::Nested::Type
Array(String)
```

## 型の組み合わせ (ユニオン)

```ruby
alias Int32OrString = Int32 | String
```

型でパイプ (`|`) を使うと、型の組み合わせ (ユニオン型) となります。`Int32 | String` は「Int32 または String」と読みます。通常のコードにおいては、`Int32 | String` が意味するのは、`Int32` に対して `String` を引数として `|` メソッドを実行することです。

## Nil を許容する (nilable)

```ruby
alias Int32OrNil = Int32?
```

これは以下と同じです。

```ruby
alias Int32OrNil = Int32 | ::Nil
```

通常のコードにおいては、`Int32?` はシンタックスエラーとなります。

## ポインタ

```ruby
alias Int32Ptr = Int32*
```

これは以下と同じです。

```ruby
alias Int32 = Pointer(Int32)
```

通常のコードにおいては、`Int32*` が意味するのは、`Int32` に対して `*` メソッドを実行することです。

## 静的配列 (StaticArray)

```ruby
alias Int32_8 = Int32[8]
```

これは以下と同じです。

```ruby
alias Int32_8 = StaticArray(Int32, 8)
```

通常のコードにおいては、`Int32[8]` が意味するのは、`Int32` に対して `8` を引数として `[]` メソッドを実行することです。

## タプル (Tuple)

```ruby
alias Int32StringTuple = {Int32, String}
```

これは以下と同じです。

```ruby
alias Int32StringTuple = Tuple(Int32, String)
```

通常のコードにおいては、`{Int32, String}` は `Int32` と `String` を要素として持つタプルのインスタンスです。これは上記のタプル**型**とは異なります。

## Proc

```ruby
alias Int32ToString = Int32 -> String
```

これは以下と同じです。

```ruby
alias Int32ToString = Proc(Int32, String)
```

引数を持たない Proc を指定するには以下のようにします。

```ruby
alias ProcThatReturnsInt32 = -> Int32
```

複数の引数を持つ Proc を指定するには以下のようにします。

```ruby
alias Int32AndCharToString = Int32, Char -> String
```

ネストされた Proc に対しては (および実際にはどんな型であっても) カッコを利用することができます。

```ruby
alias ComplexProc = (Int32 -> Int32) -> String
```

通常のコードにおいては、`Int32 -> String` はシンタックスエラーとなります。

## self

`self` は型文法では `self` の型であることを示すために利用できます。詳しくは[型制約](type_restrictions.html)を参照してください。

## クラス (class)

`class` は、インスタンスの型ではなく、クラスの型を参照するために利用できます。

例をあげます。

```ruby
def foo(x : Int32)
  "instance"
end

def foo(x : Int32.class)
  "class"
end

foo 1     # "instance"
foo Int32 # "class"
```

`class` はクラスの型を持つ配列やコレクションを作る場合にも有効に使うことができます。

```ruby
class Parent
end

class Child1 < Parent
end

class Child2 < Parent
end

ary = [] of Parent.class
ary << Child1
ary << Child2
```

## アンダースコア

型制約でアンダースコアを使うことが可能です。それはすべてにマッチすることを示します。

```ruby
# 何も制約を与えないことと同じ。これではあまり役には立たない
def foo(x : _)
end

# もう少し役に立つパターンで、引数を2つ取って Int32 を返す Proc
def foo(x : _, _ -> Int32)
end
```

## typeof

型文法では `typeof` を使うことができます。渡された式の型の組み合わせ (ユニオン型) を返します。

```ruby
alias SameAsInt32 = typeof(1 + 2)
alias Int32OrString = typeof(1, "a")
```
