# ブロックの捕捉

ブロックを捕捉して `Proc` にすることができます (captured block) 。 これはコンテキストに結びついたコードブロック (クロージャ) を表現するものです。

ブロックを捕捉するには、メソッドにブロック引数を設定し、その名前とインプット/アウトプットの型を指定する必要があります。例をあげます。

```ruby
def int_to_int(&block : Int32 -> Int32)
  block
end

proc = int_to_int { |x| x + 1 }
proc.call(1) #=> 2
```

上記のコードでは、`int_to_int` に渡されたコードブロックを `block` 変数の中に捕捉し、それをメソッドの戻り値として返しています。このとき、`proc` の型は [Proc(Int32, Int32)](http://crystal-lang.org/api/Proc.html) で、単一の `Int32` の引数を受け取り、`Int32` の戻り値を返す関数となります。

この方法で、ブロックをコールバックとして保存しておくこともできます。

```ruby
class Model
  def on_save(&block)
    @on_save_callback = block
  end

  def save
    if callback = @on_save_callback
      callback.call
    end
  end
end

model = Model.new
model.on_save { puts "Saved!" }
model.save # "Saved!" と出力
```

上記の例において、`&block` の型を指定していません。これは捕捉されたブロックが引数を何も受け取らず、戻り値も返さないことを示しています。

戻り値の型が指定されていないとき、proc の呼び出しは何も返さないことに注意してください。

```ruby
def some_proc(&block : Int32 ->)
  block
end

proc = some_proc { |x| x + 1 }
proc.call(1) # void
```

何か返して欲しい場合には、戻り値の型を指定するか、もしくはすべての型を許容したいときはアンダースコアを使ってください。

```ruby
def some_proc(&block : Int32 -> _)
  block
end

proc = some_proc { |x| x + 1 }
proc.call(1) # 2

proc = some_proc { |x| x.to_s }
proc.call(1) # "1"
```

## break と next

`break` と `next` を捕捉されたブロックの中で使用することはできません。`return` を使うことで (周りのメソッドではなく) ブロックを抜けることができます。

捕捉されたブロックにおける `next` と `return` の動作は[将来的に入れ替わるかもしれません](https://github.com/manastech/crystal/issues/420)。

## with ... yield

捕捉されたブロック内のデフォルトのレシーバは、`with ... yield` を使用して変更することが可能です。
