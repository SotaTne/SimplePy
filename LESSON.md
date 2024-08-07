# 猿でもわかるプログラミング言語

# 導入

おそらく、プログラミング言語を作ってみたいという人はたくさんいると思いますが、敷居が高そうであったり、専門書などを読んで意味のわからなさで挫折したという人も多いと思います。
今回は、そんな方やプログラミング初心者の方にもわかるという記事を目標に書いてみました。
いくつかの処理はわかりづらくなるため飛ばしていたり、私の知識が間違っている部分もあると思うのでそこはバンバン指摘してください。

## 自己紹介

現在[RpakaScript](https://github.com/SotaTne/RpakaScript)というJavaScriptと互換性のあるAssemblyScriptを用いたスクリプト言語を開発している[Sotatne](https://github.com/SotaTne)です。
まだまだ勉強している段階のため、一つのアウトプットとして今回の記事を書くことにしました。
最近はReactのようなライブラリの自作に興味を持っており、仮想Domの勉強をしています。

## 今回のコードについて

今回解説ように制作したプログラミング言語はMITライセンスのを用いて[GitHub](https://github.com/SotaTne/SimplePy)上で公開中です。

# プログラミング言語とは？

これについては色々ツッコミが入りそうなところではありますが、すごくざっくりというと**コンピュータに命令することのできるもの**という解釈が一般的だと思います。
現在ではPythonやJavaのような言語が主につかわれていますが、昔はパンチカードや紙テープなどの共通テストの回答用紙のようなものやも使われていたみたいです。
|パンチカード|紙テープ|パンチカードと紙テープ|
|---|---|---|
|![](https://storage.googleapis.com/zenn-user-upload/961eefc40fe9-20240621.gif)|![](https://storage.googleapis.com/zenn-user-upload/8d7e461c1e23-20240621.gif)|![](https://storage.googleapis.com/zenn-user-upload/e89e9ccc3bc6-20240621.jpg)

詳しい情報は井芹 康統 教授のホームページからご覧ください。
<https://www.chiba-kc.ac.jp/user/~iseri/siryo/card.html>

# どうやって作るの？

本題としてじゃあ、そのプログラミング言語をどうやって作るのかということについて解説しようと思います。
先に注意として本来は最適化などのプロセスを挟むのですが、今回は本当に必要最低限の部分だけ実装しようと考えております。
もし、他の詳しい部分に挑戦したい方はもしかしたらその記事を書くかもしれないのでそちらを参考にするか、もっと詳しいことの書いてある記事や本などで勉強してみてください

## プログラミング言語の構造

では、はじめにプログラミング言語の構造について解説したいと思います。
本来はここには最適化や中間言語などが入ってきたりしますが、今回はそれを除いたものすごくシンプルな言語についての構造です。

* **字句解析(Lexer)**
* **構文解析(Parser)**
* **意味解析+実行(Interpreter)**

この三つによって最小限の機能を持ったPythonのようなインタプリタは実装されています。

## この3つのやっていること

この**字句解析****構文解析****意味解析+実行**が行なっていることをものすごくざっくりというと、
**文字列を読み、それを種類分けした単語の集まりに変えて、その単語の集まりを文法のルールから構造に分けて、構造の細かい部分から実行する**
ということを行なっています。
なんとな〜くは何をすればいいか理解していただけましたか？
ここからはソースコードを交えながらより細かく解説していこうと思います。

# 本題-プログラミング言語の作り方

ここからが本題です。今回はPythonを用いて300行程度のソースコードで字句解析~意味解析+実行までを標準ライブラリのみで作ろうと思います。

## 字句解析について

字句解析は**Lexer**とも呼ばれる**字句解析機**を使って行います。
まず、Lexerで何をしてほしいかを少し整理してみましょう。

### やりたいこと

```python
1+2*3
```

のような文字列を

```python
[
    {
        "type":"NUMBER",
        "val":"1"
    },
    {
        "type":"ADD",
        "val":""
    },
    {
        "type":"NUMBER",
        "val":"2"
    },
    {
        "type":"STAR",
        "val":""
    },
    {
        "type":"NUMBER",
        "val":"3"
    },
]
```

このよう**Token**の配列として返すようにします
このように種類ごとに分けられた単語を**Token**と呼びます。一部のTokenでは、それが値を持っている場合があります。

### どのように処理するか

早速コードを書いてもいいのですが、まず、どのように処理をしていけばいいかを考えてみましょう。

#### 記号の場合

みている文字が**＋**だったら**ADD** , **ー**だったら**MINUS**を返すように見ている記号に対応する**Token**返せばいい

#### 数字や文字の場合(今回の例は数字)

1. 数字(1~9)が来ると、現在の文字が文字列の**何番目かをSTARTに記録**する
2. 数字(0~9)が来ると、今見ている文字を**次の文字**へと移動する
3. もし、**最後の文字**または、**数字以外の**だと終了する
4. 現在の文字が文字列の**何番目かをENDに記録**する
5. 文字列の**START**から**ENDー1**までの文字**val**として**NUMBER**の**Token**を返せば良い

```python
123+
```

のような場合
|0番目|1番目|2番目|3番目|
|---|---|---|---|
|**1**|**2**|**3**|**＋**|

なので、**START**が**0番目**で、**END**が**3番目**。返却する**val**が**0~2番目**までの**123**となる

#### Lexerのコード

ここまでで、Lexerをどのように実装すればいいかがなんとな〜くわかったと思うのでそのソースコードについてPythonで書こうと思います。

```python

typesMap = {
    "+": "ADD",  # + の時ADDを返す
    "-": "MINUS",  # - の時MINUSを返す
    "*": "STAR",  # * の時STARを返す
    "/": "SLASH",  # / の時SLASHを返す
    "=": "EQUAL",  # = の時EQUALを返す
    "\n": "NEWLINE",  # \nの時NEWLINEを返す
    "NUMBER": "NUMBER",  # 数値の時NUMBERを返す
    "IDENTIFY": "IDENTIFY",  # 変数や関数の名前に当てはまる時、IDENTIFYを返す
}
# 今回返却するTypeのまとまり


def lexer(input_str):
    tokens = []  # 返すTokenのList
    current = 0  # 現在地について

    while current < len(input_str):
        # 数字のトークン化
        if "1" <= input_str[current] <= "9":  # 1~9の時
            START = current  # STARTを現在の文字が何番目かに保存
            END = current
            while True:
                current += 1
                if (
                    current >= len(input_str) - 1 or not input_str[current].isdigit()
                ):  # 最後の文字、または0~9じゃない時
                    END = current  # ENDを現在の文字が何番目かに保存
                    break
            # Pythonの[A:B]はA~B-1までのindex番号の要素をとってくるもの
            tokens.append({"type": typesMap["NUMBER"], "val": (input_str[START:END])})

        # 識別子(変数名など)のトークン化
        elif (
            input_str[current].isalpha() or input_str[current] == "_"
        ):  # アルファベットかアンダーバー(_)の時
            START = current
            END = current
            while True:
                current += 1  # １つ次の文字へ移動
                if (
                    current >= len(input_str) - 1
                    or (
                        not input_str[current].isalnum()
                        and not input_str[current] == "_"
                        and not input_str[current].isdigit()
                    )
                ):  # 最後の文字でなくて、アルファベットでもアンダーバー(_)でも数字でものないとき
                    END = current
                    break
            # Pythonの[A:B]はA~B-1までのindex番号の要素をとってくるもの
            tokens.append({"type": typesMap["IDENTIFY"], "val": input_str[START:END]})

        # 演算子のトークン化
        elif (
            input_str[current] in typesMap
        ):  # typesMapの中に、当てはまる文字があるかどうか
            tokens.append({"type": typesMap[input_str[current]], "val": ""})
            current += 1

        # 空白文字をスキップ
        elif input_str[current].isspace():
            current += 1

        # 全て当てはまらない時エラーを出力する
        else:
            print("Lexer Error: Invalid character")
            exit()

    return tokens

```

ここまででどのように**Lexer**が動き、どうやって作ればいいかがわかったと思います。
これで**Lexer**についての説明は終了です。

## 構文解析

ここからは構文解析について書いていこうと思います。
構文解析は**Parser**とも呼ばれる**構文解析機**で行います。
先ほどと同じように何をして欲しいかを整理してみましょう

### やりたいこと

```python
[
    {
        "type":"NUMBER",
        "val":"1"
    },
    {
        "type":"ADD",
        "val":""
    },
    {
        "type":"NUMBER",
        "val":"2"
    },
    {
        "type":"STAR",
        "val":""
    },
    {
        "type":"NUMBER",
        "val":"3"
    },
]
```

このようなTokenの配列を

```python
[
    {
    "val": {
        "type": "EXPR",
        "op": "ADD",
        "left": "1",
        "right": {
            "type": "EXPR",
            "op": "STAR",
            "left": "2",
            "right": "3",
        },
    }
]
```

このような配列に変換して返すようにします
Lexerよりも出力する量は配列の数は減っていますが、正直一番ここが複雑だと思うので頑張ってください

### 処理方法の前に

おそらく、この記事を読む方はプログラミング言語の作り方の本やサイトで挫折した方や、これから挑戦しようという方が多いでしょう。
そのため、ただ処理を書くだけではなく、皆さんが作るオリジナル言語に応用を利かせたり、他の本やサイトでもここで得た知識を使えるようにするため、**文脈自由文法**を交えながら解説をしようと思います。

## 文脈自由文法とは?

自作言語を初めて作る方は、おそらく初めましての言葉だと思います。
文脈自由文法をすごく簡単にいうと、**プログラミング言語を作れるほどに強力な人間の考えて文法のルール**
です。ですので、これはプログラミング言語のためのものではなく言語学で考えられていたものをコンピュータ・サイエンスの領域に持ってきたものです。
これを使うメリットは、**機械的**に文法を解析できるという点です。これによりプログラミングで再現しやすく、**効率的**な解析を行えるというメリットがあります。

### 文脈自由文法でのルール

文脈自由文法では、機械的に処理するためにいくつかのルールがあります。
そのルールの最も重要なものが**非終端記号は必ず最終的に終端義号に変換される**というものです。
おそらく大半の人が ??? という状態だと思いますが、先に規則についてくわしく解説させてもらいます。

##### まず、文法Gについて$G = (V_n, V_t, R, S)$と定義します

これは、この言語はこの4つの要素からのみでできているという意味です。

ここで、

* $V_n$ は非終端記号の集合
* $V_t$ は終端記号の集合
* $R$ は生成規則の集合
* $S$ は開始記号
となります。

これで、さらに ??? となった方もいると思いますが、ここから一つ一つ解説するので安心してください。

#### 非終端記号$V_n$

まず、非終端記号について説明します。
例えば、

```python
a = 100*2+3
```

というプログラムがあったとします。
これ全体を`<プログラム>`とすると、この`<プログラム>`のは、`a`や`=`,`100*2+3`のような部品に分解できますよね？
この、まだ分解先を残している`<プログラム>`や、`100*2+3`のようなものを非終端記号と言います。

#### 終端記号$V_t$

次に終端記号について説明します。
終端記号とは、非終端記号とは逆にこれ以上変化することのない値のことです。
例えば`100`や`=`などは、これ以上変化させることはできませんよね？
このようなものを終端記号と言います

#### 生成規則$R$

この4つの中で唯一のルールである生成規則について説明します。
例えば

```python
a = 100*2+3
```

のようなソースコードがあったとしましょう。
はじめに説明した通り、**非終端記号は必ず最終的に終端義号に変換される**というルールが文脈自由文法ではあります。
このルールに則って考えると、このソースコード自体が**非終端記号**だということができますよね？
ですが、一気にソースコードから、**終端記号**まで変換するのはおそらく欲しいであろう構造を作れないし、欲しい処理ではないです。
そのため、生成規則は**非終端記号**$\Rightarrow$**非終端記号**または、**非終端記号**$\Rightarrow$**終端記号**などの少しずつ変換していくルールを持っています。また、すべての変換を**非終端記号** $\Rightarrow^*$(0回以上の変換)**終端記号**と、表すこともできます。
文脈自由文法の説明の最後にBNF(バッカス・ナウア記法)を用いた例を書いておきます

#### 開始記号$S$

ここまで、**非終端記号**,**終端記号**,**生成規則**と説明してきましたが、これが最後です。
文脈自由文法の最後要素である生成規則の説明の前に、一度、文脈自由文法の大切なルールをもう一度おさらいしましょう。
はじめに説明した通り、**非終端記号は必ず最終的に終端義号に変換される**というものです。
つまり、**プログラム自体も何かから始まる**ということです。**開始記号**はその何かを指します。
具体的に説明すると、$S\Rightarrow^*V_t$となるもので、変換される元を辿っていったら行き着く先とも言えます。

ここまでで文脈自由文法の説明が全て完了しました。
今回は、具体的な数学や論理学の用語はできるだけ使わずに説明しましたが、少し難しかったと思います。
おまけとして、今回の言語で使う予定の**BNF**を以下に書いているのでよければ参考にしてみてください

#### BNF

```bnf
<program> ::= <stmt> <program> | <stmt>
<stmt> ::= <printStmt> | <exprStmt>
<printStmt> ::= "print" <assign> "\n"
<exprStmt> ::= <expr> "\n"
<expr> ::= <arith_expr> "=" <expr> | <arith_expr>
<arith_expr> ::= <term> (("+" | "-") <arith_expr>) | <term>
<term> ::= <primary> (("*" | "/") <term>) | <primary>
<primary> ::= <digit> | <identify>
<number> ::= "0" | ("1" | ... | "9") <digits>
<digits> ::= <digit> | <digit> <digits>
<identify> ::= <alphabet> <alphabets>
<alphabets> ::= <alphabet> <alphabets> | <digit> <alphabets> | ε
<alphabet> ::= "_" | ("a" | ... | "z") | ("A" | ... | "Z")
<digit> ::= "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"
```

一応**BNF**についてすごく簡単に説明すると、文脈自由文法であるプログラミング言語の文法を形式的に表す一つの方法です。
`<>`で括っているものが**非終端記号**で、`""`で括っているものが**終端記号**、`$V_n$::=$V_t$`や、`$V_n$::=$V_n$`のようになっているものが**生成規則**で、`<program>`が**開始記号となっています**

## 構文解析(続き)

**文脈自由文法**はどうでしたか？重要な概念だとわかっても正直そこまで楽しくないですよね
ここまでで、皆さんは文法をなんとな〜くどうやって定義して処理すればいいかわかったと思います。
ここからはそれをより具体的なものにしていきましょう

### どのように処理するか

おそらく、何をすればいいかなんとなくここまで読み進めている方はわかるが、どのような方針で進めれば良いかわからないという方が多いと思います。
ここで役に立つのが先ほど作った**BNF**です。
このBNFを整理するとして、手順を考えていきましょう

#### 大まかな処理

1. Stmt(文)を読む
2. Expr(式)を読む
主に、処理自体が全く違うものはこの二つだと思います。
では、まずStmt(文)をどうやって読めばいいかを考えてみましょう

#### Stmt(文)の読み方

今回定義しているStmtは`printStmt`と`exprStmt`のたった二つだけなので結構処理は楽ですが、for文なども同じような方法で実装することができます。
以下は、Stmtの集合のようなものです

```python
a = 10+2*4
print a \n
```

#### Stmt(文)の処理方法

現在の**Token**が`print`かどうかを確認
printの場合、[printStmtの読み方](#printstmt%E3%81%AE%E8%AA%AD%E3%81%BF%E6%96%B9)で説明します
それ以外の場合、[exprStmtの読み方](#exprstmt%E3%81%AE%E8%AA%AD%E3%81%BF%E6%96%B9)で説明します

#### printStmtの読み方

BNFを読むとわかる通り、**printStmt**は、右側に、`"print"`左側に`<expr>`を持つものです。
つまり、**print**が来て、次に**<expr>**が来て、最後に`\n`がくるというふうに読めばいいということです。
イメージしずらいかもなので、例を挙げると

```python
print 1+3+4 \n
```

このようなStmt(文）を読むことができます。
`1+3+4`は？と思うかもしれませんが、これを処理するのは**Expr(式)**で処理をします。

#### exprStmtの読み方

こちらは、**printStmt**の後だと簡単ですね
これは左辺を持たずに`<expr>`の後に`\n`が来ているものです。
こちらも例を挙げると

```python
a=100*2+3
1*a \n
```

このようなStmt(文）を読むことができます。
この言語の文法では、`a=100*2+3`などの代入や変数の定義も**Stmt(式)**として扱います。

#### Expr(式)の読み方

ここからは先ほどより、少し難しくなりますよ。
優先順位のことを考えながら実装する必要があるためです。
一度`<expr>`についての**BNF**を読んで少しおさらいしてみましょう

##### BNF

```bnf
<expr> ::= <arith_expr> "=" <expr> | <arith_expr>
<arith_expr> ::= <term> (("+" | "-") <arith_expr>) | <term>
<term> ::= <primary> (("*" | "/") <term>) | <primary>
<primary> ::= <digit> | <identify>
<number> ::= "0" | ("1" | ... | "9") <digits>
<digits> ::= <digit> | <digit> <digits>
<identify> ::= <alphabet> <alphabets>
<alphabets> ::= <alphabet> <alphabets> | <digit> <alphabets> | ε
<alphabet> ::= "_" | ("a" | ... | "z") | ("A" | ... | "Z")
<digit> ::= "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"
```

ここでわかることは<expr>は`<identify>"="<expr>`または、`<arith_expr>`のどちらかということです。
まず、前提として`printStmt`から、この処理に来た場合は`print`はすでに読まれていて、間の`<expr>`のみを読めばいい状態だというのはわかりますか？
また、`exprStmt`の場合は先に読む必要はないので、`\n`の前の<expr>のみを読めばいい状態というのもわかりますね。
では、ここから本格的に処理を考えていきましょう

#### <expr>の読み方

まず、この**Parser**に渡されているのは**Token**の配列ということは覚えていますか？
わかりやすくするため、以下のような**Token**の配列が渡されたと仮定しましょう

```python
[
    {"type": "IDENTIFY", "val": "print"},
    {"type": "IDENTIFY", "val": "a"},
    {"type": "EQUAL", "val": ""},
    {"type": "NUMBER", "val": "3"},
    {"type": "STAR", "val": ""},
    {"type": "NUMBER", "val": "1"},
    {"type": "ADD", "val": ""},
    {"type": "NUMBER", "val": "2"},
    {"type": "NEWLINE", "val": ""},
]
```

こうすると`print`は読み込まれているので、現在読んでいるIndex番号は1番の`IDENTIFY`(a)ということがわかります。
また、`<expr>`は`<arith_expr>"="<expr>`か`<arith_expr>`のどちらかですまた、`print`は先ほども言った通りすでに読んでいるので、現在いる場所は、<arith_expr>内の最初の**Token**または、のはずです
下に図を用意したので参考にしてください

```python
print a = 3*1+2 \n
```

のような場合
|0番目|1番目|2番目|...|
|---|---|---|---|
|**print**|**a**|**=**|...|
|"print"|<arith_expr>|"="|<expr>|

と表現することもできます。
2列目はイメージしやすいようにしているため本当は、もう少し定義をしないとダメなんですが一旦このまま進めます。

では、処理の方に戻りましょう。
ここで重要なことは、現在いるのが`<arith_expr>`の次に来るのが`=`であるかということです。
**ここで重要なのが、"="を読むと、現在見ているTokenを一つ進めなければならないということです。**
そうじゃないと次の処理にいけませんからね
もし、`=`ならばそれは<expr>としてもう一度処理する必要があり、そうでないなら<arith_expr>として処理をすれば良いということです。
この処理は、`<arith_expr>`を処理した後に`=`だったら`<expr>`で再度処理して、そうではなければ処理を終了すればいいということがわかります。

つまり、とりあえず`<arith_expr>`をどのように読み取るかを考えなければならないことがわかります。

#### <arith_expr>の読み方

ここからは慣れたら少し楽な処理です。
まず、`<arith_expr>`は`<term>("＋"か"ー")<arith_expr>`または`<term>`です。
**ここで重要なのが、"＋"か"ー"を読むと、現在見ているTokenを一つ進めなければならないということです。**
処理の関係上、ずっと同じTokenを見ていると

```python
1+2
```

という単純な処理が内部的には`1+++++++`という意味のわからないプログラムとして処理されてしまうからです。
これも先ほどと同じように`<term>`を処理して、その後に"＋"か"ー"のどちらが来ていれば、もう一度`<arith_expr>`を。そうでなければ処理した<term>を返せばいいということです。
なんだか周りくどい方法をしていると思いますか？
しかし、これには以下のような式を読むことを可能にしています。

```python
1+2+3+4
```

これは、イメージですがこのBNFで読み込むと以下のようになります。

```python
{
    "op":"ADD",
    "left":1,
    "right":{
        "op":"ADD",
        "left":2,
        "right":{
            "op":"ADD",
            "left":3,
            "right":4
        }
    }
}
```

#### <term>の読み方

ここまで来たら、次はどうするのか予測できる人が多いと思いますがちゃんと説明していきます
まず、`<term>`は、`<primary>("＊"か"/")<term>`または、`<primary>`です。
**ここで重要なのが、"＊"か"/"を読むと、現在見ているTokenを一つ進めなければならないということです。**
理由は[<arith_expr>の読み方](#%3Carith_expr%3E%E3%81%AE%E8%AA%AD%E3%81%BF%E6%96%B9)で説明した理由と同じです
これも先ほどと同じようにまず`<primary>`を処理して、その後`"＊"か"/"`ある場合は、もう一度<term>を。そうでない場合は処理した`<primary>`をそのまま返せば実装できます。

#### <primary>の読み方

結構パターン化してきましたね
ここからはかなり楽ですよ
まず、`<primary>`は`<digit>もしくは<identify>`です。つまり、**自然数**か、**変数名**を調べて当てはまる場合はそれを返して、そうじゃない場合はエラーを吐くようにすればいいだけです。
**BNF**では少し複雑になっていますが、いくつかの制限があるためあのような表記になっていますが、ぶっちゃけるとこれだけです。
ここが他の処理と唯一違うのが、返すものが**終端記号**ということです。
また、**ここで重要なことが、現在見ているTokenを一つ進めなければならないということです。**
なぜかというと、現在読んでいる値を読み終わったのに、もう一度その値を読むことになってしまうからです

```python
1
```

これが、

```python
[
    {
        "type":"NUMBER",
        "val":1
    },
    {
        "type":"NUMBER",
        "val":1
    }
    .
    .
    .
]
```

のようになってしまいますからね

#### 処理のまとめ

気づいた方もいるかもしれませんが、**Parser**の処理で現在読んでいる配列の位置を移動させるときは、**終端記号**が来たときだけです。

#### Parserのコード

では、どのように処理をすれば良いかわかったところで、以下にPythonのサンプルコードを書きます。

```python
def parser(tokens):
    parsed = []

    def error(message):
        # エラーメッセージを表示し、プログラムを終了する
        print(message)
        exit()

    def stmt(current):
        # トークンリストの末尾に達していない場合
        if len(tokens) > current:
            # 式を解析し、結果を left として取得
            current, left = expr(current)
            # "print" 文の場合
            if (
                current < len(tokens)
                and left["op"] == "IDENTIFY"
                and left["val"] == "print"
            ):
                # "print" 文の引数を解析
                current, val = expr(current)
                # 解析結果を文としてパース済リストに追加
                parsed.append({"type": "STMT", "op": "PRINT", "val": val})
            else:
                # その他の文の場合
                parsed.append({"type": "STMT", "op": "", "val": left})

            # 改行トークンのチェック、または最後のトークンでない場合
            if (current < len(tokens) and tokens[current]["type"] == "NEWLINE") or (
                current == len(tokens) - 1 and tokens[current]["type"] != "NEWLINE"
            ):
                current += 1
            elif current < len(tokens):
                # 改行がないまたは、最後のトークンでない場合はエラーを報告
                print(tokens[current]["type"])
                error("Expected NEWLINE in stmt")

            # 再帰的に次の文を解析
            stmt(current)
        return current

    def expr(current):
        # 算術式を解析し、結果を取得
        current, val = arith_expr(current)
        # 代入演算子 "=" のチェック
        if current < len(tokens) and tokens[current]["type"] == "EQUAL":
            current += 1
            # 右辺の式を再帰的に解析
            current, right = expr(current)
            return current, {
                "type": "EXPR",
                "op": "ASSIGNMENT",
                "left": val,
                "right": right,
            }
        return current, val

    def arith_expr(current):
        # 項を解析し、結果を取得
        current, val = term(current)
        # 加算または減算演算子のチェック
        while current < len(tokens) and (
            tokens[current]["type"] == "ADD" or tokens[current]["type"] == "MINUS"
        ):
            Type = tokens[current]["type"]
            current += 1
            # 右辺の項を再帰的に解析
            current, right = term(current)
            val = {"type": "EXPR", "op": Type, "left": val, "right": right}
        return current, val

    def term(current):
        # プライマリを解析し、結果を取得
        current, val = primary(current)
        # 乗算または除算演算子のチェック
        while current < len(tokens) and (
            tokens[current]["type"] == "STAR" or tokens[current]["type"] == "SLASH"
        ):
            Type = tokens[current]["type"]
            current += 1
            # 右辺のプライマリを再帰的に解析
            current, right = primary(current)
            val = {"type": "EXPR", "op": Type, "left": val, "right": right}
        return current, val

    def primary(current):
        # 識別子トークンのチェック
        if current < len(tokens) and tokens[current]["type"] == "IDENTIFY":
            val = tokens[current]["val"]
            current += 1
            return current, {"type": "EXPR", "op": "IDENTIFY", "val": val}
        # 数値トークンのチェック
        elif current < len(tokens) and tokens[current]["type"] == "NUMBER":
            val = tokens[current]["val"]
            current += 1
            return current, {"type": "EXPR", "op": "NUMBER", "val": val}
        else:
            if current < len(tokens):
                print(tokens[current]["type"])
            error("Expected IDENTIFY or NUMBER in primary")

    stmt(0)  # トークンリストの先頭から解析を開始
    return parsed
```

おそらくインタプリタで一番大変な**Parser**についての説明が終了しました。
案外簡単でしたか？

## 意味解析+実行

ここからは**意味解析+実行**につての解説を始めようと思います。
**Interpreter**とも呼ばれる**翻訳系**で行われます。
これまでと同じように何をして欲しいかを整理してみましょう。と言いたいところですが、**Interpreter**ではソースコードをこれまで作ってきた**Lexer**と**Parser**に通したものを実行していくものです。
つまり、これまでとは違い何か文字を返す関数を作るわけではありません。
一応以下に、print関数と四則演算を用いた**Parse**に通した配列と、それによって出力されるものをまとめたものを書いておきます。

```python

```

この**Parser**を通した配列により、以下が出力されます

```python
```

このような実行結果を出力します。

# 引用元
>
> <https://www.chiba-kc.ac.jp/user/~iseri/siryo/card.html>
