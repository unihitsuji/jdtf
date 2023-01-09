# [Gauche-tk](https://github.com/shirok/Gauche-tk) の意訳な和訳

# Gauche-tk について

Gauche のシンプルな Tk バインディングだよ (^o^)/
パイプ経由で wish (Tcl/Tk のインタープリタ) を呼び出してるから
このパッケージをコンパイルすることすら不要なほどシンプル (^^)y

> This is a simple Tk binding for Gauche.
> It's so simple that you don't even need to compile this package---
> we invoke the 'wish' command (Tcl/Tk's interactive shell)
> and communicate to it via pipes.

## 前提条件

要 Gauche 0.9.3 以降

## 簡単な例

```scheme:
(use tk)

(tk-init '())
(tk-button '.b :text "Click me" :command (^[] (print "Yeah!")))
(tk-pack '.b)
(tk-mainloop)
```

- `(^[] ...)` は `(lambda () ...)` の別名だよ (^-^)!  
  [Gauche リファレンス 手続きを作る](http://practical-scheme.net/gauche/man/gauche-refj/Shou-Sok-kiwoZuo-ru.html#g_t_624b_7d9a_304d_3092_4f5c_308b)
   を見てね (^^)

このコードはボタンを作り、
ボタンが押されたときに呼び出されるコールバックを設定してるよ (^-^)!
あなたが Tcl/Tk を知ってるなら納得できるよね 
文法はちょっと違ってるけど (-_-;;;

> This code creates a button and set a callback which will be called
> when the button is clicked.
> If you know Tcl/Tk, you can make sense,
> although the syntax is slightly different.

`tk-init` は tk サブシステムを初期化し wish のプロセスを起動するよ (^-^)!

> `tk-init` initializes tk subsystem. It invokes wish process.

`tk-button` コマンドは ".b" と名付けられたボタンを作るよ (^-^)!
wish とのやり取りは全て文字列に変換されるので
シンボルを渡すのか文字列を渡すのか問題にならない (^^)v
オプション名、例えば `-text` は キーワード `:text` を使うよ (^-^)!

> `tk-button` command creates a button named ".b".
> When we talk to wish, we convert everything into strings,
> so it doesn't matter whether you pass a symbol or a string here.
> For the option names (e.g. `-text`), use keywords (e.g. `:text`).

トリッキーなのはコールバックで、
Tk が Tcl コードを期待する場面でも Scheme のプロシージャを渡せるよ (^-^)v
Gauche-tk は Tk 側に
(Tk が何をしていいかわからないような) Scheme プロシージャを渡してはいないんだ (=_=)
代わりに小さなダミーの Tcl コードをコールバックとして登録してるよ (^-^)!
ダミーコードの実行を検出したら Scheme プロシージャを呼び出すんだ (^-^)!
この詳細は、アプリケーションを書く上でほぼ問題にならないだろうけど
クロージャは Scheme 側で実行してることを気に留めておくと
トラブルシューティングの助けになるかもね (^^)v

> A tricky part is the callbacks---
> you can pass Scheme procedure where Tk expects Tcl code to be called.
> Gauche-tk does not pass the Scheme procedure to the Tk world
> (Tk doesn't know what to do with it!);
> instead we register a small dummy Tcl code as a callback,
> and when we detect that dummy code is executed,
> we call Scheme procedure in our side.
> This detail may not matter much while you're writing applications,
> but keeping in mind that the closures are executed in the Scheme world
> (not in the Tcl/Tk world) may help troubleshooting.

`tk-mainloop` 呼び出しはイベントループに入るよ (^^)y
でもね～ Tk ウィンドウが閉じられるまで戻ってこない (>_<)
コールバックを機能させるには `tk-mainloop` を呼び出す必要がある (=_=)

> `tk-mainloop` call enters the event loop.
> It doesn't return until the Tk window is closed.
> You need to call `tk-mainloop` to make callbacks work.

REPL で作業してるときに `tk-mainloop` が戻らないのは不便だよね (>_<)
イベントループを走らせながらも REPL プロンプトが必要なら
`background` キーワード引数で `tk-mainloop` を呼び出そう(^-^)!

> If you're working on REPL,
> it is inconvenient that `tk-mainloop` doesn't return.
> If you want REPL prompt even while running event loop,
> call `tk-mainloop` with `background` keyword argument:

```scheme:
(tk-mainloog :background #t)
```

これはイベントループを別のスレッドで走らせるよ (^-^)!
REPL プロンプトで作業を続けられるようにね (^^)v

> This runs the event loop in a separate thread,
> enabling you to keep working in REPL prompt.

Tcl/Tk 側では `button .b` コマンドは新しいコマンド `.b` を作って
その後 ボタンの振る舞いを変えたり、属性を問い合わせるために `.b` が使われるんだ (-_-)
Scheme 側では `tk-call` コマンドを使わなきゃいけないよ (^-^)!
以下のコードは最初に現在のテキスト値を問い合わせし、それを変更してるよ

> In Tcl/Tk, the `button .b` command creates a new command `.b`,
> which can be subsequently used to change the button's behavior
> and to query its attributes.
> In Scheme, you need to use `tk-call` command.
> The following code first queries the current text value of the button,
> then change it.

```scheme:
(tk-call '.b 'cget :text)
(tk-call '.b 'configure :text "Don't click me")
```

## 華麗に終了

Tk 側 (wish コマンド) は Gauche と別のプロセスなので
Gauche が終了した後も Tk プロセスが残る可能性があるよ (=_=)
tk-shutdown を呼ぶと Tk プロセスが終了するよ (^-^)!
Tk プロセスの終了を確実にするひとつのやり方は、
あなたのアプリケーションの中で `exit-handler` を設定することだね (^^)
例えば

> Since the Tk part (wish command) is a separate process from Gauche,
> it is possible that the Tk process remains
> after the Gauche process terminates.
> Calling `tk-shutdown` terminates the Tk process.
> One way to ensure termination of Tk process is
> to set `exit-handler` in your application, e.g.:

```scheme:
(exit-handler (^ [code fmtstr args] (tk-shutdown)))
```

詳細は [Gauche マニュアル](http://practical-scheme.net/gauche/man/gauche-refj/index.html) の [exit-handler のドキュメント](http://practical-scheme.net/gauche/man/gauche-refj/sisutemuintahuesu.html#g_t_30d7_30ed_30b0_30e9_30e0_306e_7d42_4e86) を読んでね (-人-)
アプリケーションの対応は `exit-handler` で決まるので大事だよ (^^)
ライブラリはその値を変えてはいけない

> See the documentation of `exit-handler` in Gauche manual for the details.
> It is important that it's application's responsibility
> to decide what to do with `exit-handler`---
> a library shouldn't change its value.

REPL で作業してて、やり直しがしたいなら `tk-shutdown` が便利だよ (^-^)!
`tk-shutdown` して `tk-init` を呼び出せば、新たな Tk プロセスを得られるよ (^^)v

> `tk-shutdown` is also convenient
> if you're working in REPL and want to start over---
> call `tk-shutdown`, then `tk-init` again, gives you a fresh Tk process.

## コールバックのパラメータ

いくつかのコールバックはパラメータが必要だよ (=_=)
例えば、マウスクリックイベントは、マウスの位置情報が必要だよね～
Tcl 内では置換で処理してるよ (^-^)!
`%x` のような特別な文字列(シーケンス)をスクリプトに埋め込むと
Tcl ランタイムが実行前にそれを置換しちゃうんだよ (^-^)!
Gauche-tk はパラメータを取得する特別なマクロ `tklambda` を使うことができるよ (^-^)v

> Some callbacks needs to receive parameters;
> for example, mouse click event wants to know where the mouse cursor is.
> In Tcl, it is handled by substitution---
> you embed a special sequence such as %x in the script,
> and Tcl runtime substitutes it with the number before executing the script.
> In Gauche-tk,
> you can use a special macro `tklambda` to receive the parameters.

```scheme:
(use tk)

(tk-init '())
(tk-bind "." '<Key>      (tklambda (%K) (print #`"Pressed ,%K")))
(tk-bind "." '<Button-1> (tklambda (%x %y) (print #`"Clicked at (,%x ,%y)")))
(tk-mainloop)
```
`%K` のようなパラメータの名前は何の値かで決まるよ (^^)
[有用な値](http://www.tcl.tk/man/tcl8.7/TkCmd/bind.html#M25) は
[Tk のドキュメント](http://www.tcl.tk/doc/) を読んでね (^^)

> The parameter name such as `%K` determines what kind of value it receives.
> See the Tk document for the available names.
> Within the Scheme code, `%x` etc. are just ordinary variables.

## Tcl/Tk とのやりとり

Tcl/Tk プロセスとのやりとりのために、いくつかの API が用意されてるよ (^^)v

> Some APIs are provided to communicate with Tcl/Tk process.

`(tk-ref varname)` は Tcl 変数 `varname` の値を返すよ (^-^)!
通常、トップレベルの変数が欲しいので、厳密な修飾名を使うほうが良いよ (^^)
例えば `(tk-ref "::thevar")` はトップレベル変数 `thevar` が得られるよ (^^)v

> `tk-ref varname` returns the value of Tcl variable varname.
> Normally you want to refer to toplevel variable,
> and it's better to use explicitly qualified name---
> e.g. `(tk-ref "::thevar")`
> to ask the value of the toplevel variable "thevar".

Tcl 側では全てが文字列だということを思い出そう (^^)
返された文字列をあなたが望む形式に翻訳しなきゃね (^-^)!

> Note that in the Tcl world, everything is a string.
> You need to interpret the returned string as you wish.

`(tk-parse-list value)` は Tcl の(ネストした)リストを構文解析するよ (^^)v

> `tk-parse-list value` is a utility procedure
> that parses (nested) Tcl string as a list.

```scheme:
(tk-parse-list "{a b \"c d\" e} {f g}")
  => (("a" "b" "c d" "e")("f" "g"))
```

`(tk-set! varname value)` は Tcl 変数をセットするよ (^-^)!
`value` は Tcl に渡される前に文字列に変換されるよ (^^)
トップレベル変数として `::varname` みたいに修飾して明確にしたいね～ (^^)

> `(tk-set! varname value)` sets the Tcl variable.
> The `value` is converted to string before passed to Tcl.
> Normally you want to qualify varname as `::varname`
> to make sure it is a toplevel variable.

Gauche-tk は Tcl/Tk 8.4 で可能な (例えば `bind` のための `tk-bind` のような)
対応する API を提供してるよ (^-^)!
別の Tcl/Tk コマンドを使いたいなら `tk-call` で使えるよ (^^)v
`tk-call` はコマンドと引数をとって Tk プロセスに送り結果を文字列として受け取るよ (^^)

> Gauche-tk provides APIs corresponding to Tk commands available
> in Tcl/Tk 8.4 (e.g. `tk-bind` for `bind` Tk command).
> If you want to use other Tcl/Tk command, you can use `tk-call`.
> It takes a command and arguments, and send it over to Tk process,
> then receives the result as a string.

```scheme:
(tk-call 'expr "3 + 4") => "7"
```

Tk 側でエラーが発生すると、Scheme 側に `<tk-error>` が投げられるよ (^-^)!

> If an error occurs in the Tk side,
> `<tk-error>` condition is thrown in the Scheme world.


```scheme:
gosh> (tk-call 'expr "3 +")
*** TK-ERROR: syntax error in expression "3 +": premature end of expression
```

かなりの頻度で `tk-call` を使って Tcl コマンドを実行してるなら
そのための Scheme プロシージャを作ることができるよ (^^)v

> If you find you invoke some Tcl command via `tk-call` often enough,
> you can create a Scheme procedure to do so.

```scheme:
(define-tk-command tk-expr expr)
(tk-expr "3 + 4") => "7"
```

実際、`tk-bind` 等などはこう定義されてるよ (^-^)!

> In fact, this is how `tk-bind` etc. is defined.

## トラブルシューティング

### wish へのパス

`(use tk)` で Gauche-tk モジュールがロードされると
Gauche は wish 実行ファイルを PATH の中から探すよ (^^)
見つからなかったら、`tk-init` は失敗になるよ (=_=)
wish コマンドが標準的でないフォルダにインストールされてる場合
(または何らかのカスタマイズコマンドが必要な場合)
Gauche に知らせるために、wish 実行ファイルへのパスを `wish-path` に設定しよう (^-^)v

> When Gauche-tk module is loaded by `(use tk)`,
> Gauche scans paths in PATH to find 'wish' executable.
> If it can't find one, tk-init will fail.
> In certain cases that you have 'wish' command in nonstandard location
> (or want to use other customized command),
> set up `wish-path` parameter to tell the path to the executable to Gauche.

```scheme:
(wish-path "/path/to/wish")
```

`tk-init` の前に実行しようね (^^)

> It should be executed before `tk-init`.

### 通信のダンプ

抽象化の壁は薄いので、
ときに Gauche - Tk 間の低レベルなやり取りを調べ上げたいことがあるよね (^^)
そんなときに助けてくれるのが `*tk-debug*` 変数だよ (^^)v

> The wall of abstraction isn't strong enough
> and sometimes you need to dig into the low-level communication
> between Gauche and Tk.
> There's a hidden variable *tk-debug* that helps you to do so.

```scheme:
(with-module tk (set! *tk-debug* #t))
```

この後は、Tk - Gauche 間の全てのやり取りが、標準出力にダンプされるよ (^^)v

> After this, all communication between Tk and Gauche is dumped to stdout.

### リークの回避

コールバックとして渡される Scheme のプロシージャは
グローバルなハッシュテーブルに登録されるよ (^^)
現在、このハッシュテーブルはガベージコレクトされないよ (>_<)
例えば、次のコードはボタン "." のコールバックを変更してるけど...... (=_=)

> Scheme closures passed as callbacks are registered in a global hashtable.
> Currently, this table won't be GC-ed.
> For example, the following code changes the callback to the button ".b":

```scheme:
(button ".b" :command (^[] (foo)))
(tk-call ".b" 'configure :command (^[] (bar)))
```

このコードでは、初期のコールバック `(^[] (foo))` が二度と呼ばれないのに
ハッシュテーブルの中に残ってしまうんだよ (>_<)
コールバックの登録を頻繁に変えると問題になるよね (=_=)

> With this code, the initial callback `(^[] (foo))` remains
> in the hashtable even it will never be called again.
> This will be an issue if you change the registered callbacks too often.

幸運なことに簡単な回避策があるよ (^-^)v
コールバックの振る舞いを変更したいなら Scheme 側を変えるのさ (^^)

> Fortunately, there's an easy workaround.
> If you need to change callback behaviors, you change it in the Scheme side:

```scheme:
(define *callback* (^[] (foo)))

(define (bridge) (*callback*))
(button ".b" :command bridge)

(set! *callback* (^[] (bar)))
```
- コールバックとして ".b" に登録するのは `bridge` にする
- `bridge` は `*callback*` を指している
- `*callback*` を `set!` で変えれば、".b" のコールバックも変わる (^-^)v
