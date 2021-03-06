# クロージャ

捕捉されたブロックと proc リテラルはローカル変数と `self` を閉包 (クロージャ) します。例を見てみるとわかりやすいでしょう。

```crystal
x = 0
proc = ->{ x += 1; x }
proc.call #=> 1
proc.call #=> 2
x         #=> 2
```

もしくは、メソッドが返す proc の場合は以下となります。

```crystal
def counter
  x = 0
  ->{ x += 1; x }
end

proc = counter
proc.call #=> 1
proc.call #=> 2
```

上記の例において、`x` はローカル変数であるにも関わらず、proc リテラルによって捕捉されています。このとき、コンパイラは `x` をヒープに割り当て、proc が動作するためのコンテキストのデータをして利用します。通常であれば、ローカル変数はスタックに存在し、メソッドが終了すると消えます。

## クロージャの変数の型

ローカル変数の型に対して、コンパイラは「それなりに」賢く解釈します。例をあげます。

```crystal
def foo
  yield
end

x = 1
foo do
  x = "hello"
end
x # :: Int32 | String
```

コンパイラは、ブロックの後に `x` が Int32 か String であることを判断できます (ただ、この場合だとメソッドは必ず yield するので、常に String であることは明白です。将来的にはそこまで判断できるように改善するつもりです) 。

もし、ブロックの後で `x` に何かが代入されたとき、コンパイラはその型が変更されたと判断します。

```crystal
x = 1
foo do
  x = "hello"
end
x # :: Int32 | String

x = 'a'
x # :: Char
```

しかし、もし `x` が proc によってクロージャに包まれた場合は、その型はすべての代入された型の組み合わせとなります。

```crystal
def capture(&block)
  block
end

x = 1
capture { x = "hello" }

x = 'a'
x # :: Int32 | String | Char
```

この理由は、捕捉されたブロックはグローバル変数やクラス変数、そしてインスタンス変数に保持されることもあり、そして別々のスレッドで実行される可能性もあるためです。このことに対して、コンパイラが綿密な分析をすることはありません。コンパイラはただ、proc に変数が捕捉されていたら、その proc がいつどこで実行されるかは未知である、として扱います。

これは通常の proc リテラルにも当てはまります。そして、その proc が実行も保持もされないことが明白であっても同様です。

```crystal
def capture(&block)
  block
end

x = 1
->{ x = "hello" }

x = 'a'
x # :: Int32 | String | Char
```



