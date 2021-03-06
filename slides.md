# TestFramework and Lint

---

## 私について

---

![](https://avatars0.githubusercontent.com/u/1124024?v=3&s=230)

Vim scriptのLint developer。

専門はJavaScript。

いまJavaScriptのテスト本を執筆中です。


---

## Lintとは何か

---

### 構文解析に基づく静的検査ツール

---

#### トークン列の例
```
          if ( condition ) { }
トークン列: AA B CCCCCCCCC D E F
```

#### 構文木の例
```
if ( condition ) { }
```

```
{
    type: "if",
    condition: {
      type: "identifier",
      name: "condition"
    },
    ...
}
```

---

### 3種の検査項目
では、Lintは何を検査するのでしょうか？

検査項目はいろいろあるが、次の3つに分類できます：

- 確実に間違えている
- いつか間違えそう
- スタイルが乱れている

---

#### 確実に間違えている
確実に間違えているもの。

単純な構文エラーや、未定義の変数参照など。


---

#### いつか間違えそう
braceなしif文など。

その時点では問題はおきていないが、将来的な変更によって危険なコードになる可能性があるもの。


```
goto fail;
goto fail;
```

が有名ですね。


---

### Lintの検査項目1: 代表的な間違いたち
実際に、どんな間違いがあるのかみてみましょう。


---

#### equals or == (Java)
リファレンスの比較とクラス定義の比較は異なります。

文字列の比較を`==`でやって痛い目にあった人は数知れないでしょう。


---

#### 'use strict' (JavaScript)
JavaScriptは処理系の検査を厳しくするための構文をもっています。

ファイル先頭に`'use strict'`と書くと、そのファイル全体の検査を厳しくできます。

ただし、他言語とは異なりJSにはソースを結合する文化があることが問題を引き起こしました。

たまたま`'use strict'`が適用されているファイルが結合順の先頭にくると、`'use strict'`に耐えられないスクリプトが例外で死ぬのです。


---

#### do-end or {} (Ruby)
ブロック付きメソッド呼び出し構文は`do ... end`と`{}`があります。

この2つは、演算子の結合の強さに違いがあります。

これを理解せずに単に置換してしまうと、解釈に違いがでてしまいます。


```
foobar a, b do .. end
foobar a, b { .. }
```

---

#### 2引数 open (Perl)
ファイルのopen関数は1~3引数で呼び出せます。

このうち、2引数のものが危険。

ファイル名にあたる第二引数部分でシェルコマンドが実行できるのです。

Webサーバー等でこれをやるとOSCiが発生します。

ヤバイ。


---

#### =~演算子とignorecase (Vim script)
Vimには`ignorecase`という大文字/小文字の違いを無視させる設定があります。

この設定は、Vim scriptの文字列の比較演算`=~`や`!~`にまで影響します。

そのため、ignorecaseの影響を受けない`=~#`や`=~?`を使うべきなのです。


---

#### すべてを覚えられますか？
たくさんの言語があり、たくさんの闇があります。

これ、すべて覚える必要があるのでしょうか。

これらの闇は、ほとんど本質的ではないように思えます。

ちなみに、紹介した事例はすべてLintによって指摘されます。


---

### Lintの検査項目2: コーディングスタイル

---

#### スタイルが乱れるとは

ソフトタブ・ハードタブの混在や、インデント幅の乱れなどのことをさします。
他にも、空白の開け方が異なっていたりから、コメントの先頭文字の大小が一貫していない、というものまであります。

しかし、スタイルが乱れるのはプログラマの感性の問題ではありません。
そもそもスタイルの派閥が多すぎるのが問題なのです。

---

#### braceの改行位置だけとってもこの有様
[Wikipedia - 字下げスタイル](https://ja.wikipedia.org/wiki/%E5%AD%97%E4%B8%8B%E3%81%92%E3%82%B9%E3%82%BF%E3%82%A4%E3%83%AB)

- K&Rのスタイル
- BSD/オールマンのスタイル
- BSD/KNFスタイル
- ホワイトスミスのスタイル
- GNUスタイル
- Picoスタイル
- Bannerスタイル

---

#### アレルギーさえ引き起こすコーディングスタイル
ときにコーディングスタイルは、アレルギー反応のごとき気持ち悪い感を与えます。

特に、JavaScriptのセミコロン前置スタイルとか。

私は耐えられません。発狂します。


>     foo();
>     [1,2,3].forEach(bar);
>
> you could do this:
>
>     foo()
>     ;[1,2,3].forEach(bar)

引用: [An Open Letter to JavaScript Leaders Regarding Semicolons](http://blog.izs.me/post/2353458699/an-open-letter-to-javascript-leaders-regarding)

---

### ここまでのまとめ
Lintの検査項目はいろいろありますが、次の3つに分類できます

- 確実に間違えている
- いつか間違えそう
- スタイルが乱れている

---

## なぜ私はLintを使うのか?
さて、私は自他共に認めるLintFreakです。

なぜこんなにもLintが好きなのかを説明します。


---

### テストはしている、しかし、見つけられない問題たち
私はテストも好きなので、よほどのことがない限りテストを書きます。

しかし、テストを書いても見つけられない問題たちがあることに気がつきます。

動作に支障はなくても、保守性に支障があるパターンはたくさんあるということです。


---

#### 到達不能な処理
到達不能な処理は、明らかに不必要なコードです。

コードは文書という側面ももちます。

だから、原則として不必要なものをコードに残してはいけません。


---

#### 未使用の変数
未使用の変数も、不必要なコードです。

しかも頻繁に発生する厄介者です。


---

#### メモリリーク
メモリリークは長時間動作するプログラムでは致命的な問題です。

ただし、メモリリークが原因でユニットテストが失敗することはまずありません。

奮起して耐久テストをやらない限りはみつからないのです。

しかし、簡単なパターンであれば静的検査で機械的に発見できます。


---

### フィードバックの速さ
ユニットテストでは、ユニットテストを書き、コードを実行するまで問題があるかどうかわかりません。

Lintは静的な検査なので、実行するまでもなく問題が露見します。

そのため、ユニットテストよりも手間が少なくフィードバックがはやいのです。

---

#### 変数名のtypoを単体テストで発見するのは遅すぎる
検証できる問題のレイヤーは異なりますが、Lintで見つかる問題をユニットテストで見つけたくはありません。

---

### 一貫したコーディングスタイルの実現
コードの外観は、エラープルーフのために重要です。

しかし、あたりまですが、ユニットテストはコードがどんな外観をしているのかについて無関心です。

コードがどう振る舞うかに興味があるからです。

---

### ここまでのまとめ

私がLintを愛する理由は3つ：

- テストはしていても、見つけられない問題があるから
- フィードバックがとても速いから
- 一貫したコーディングスタイルを実現できるから

---

## Lint の様々な側面
では、私の主観をはずれて、Lintをいろいろな角度からみつめてみます。

すると、次のような側面を見つけることができます。


---

### Lintの側面: コードレビューの省力化ツール
そもそも、機械でできることを人間がするべきではありません。

人間がコーディングスタイルの乱れ等（姑チェック）をおこなうのは不毛です。

人間はもっと別のことに注力するべきです。

コードレビューの補助ツールとしての側面をつかうとよいでしょう。


---

### Lintの側面: コードレビューの品質向上ツール
人間はミスをする生き物です。

しかし、きちんとテストされたLintはミスをおかしません。

これは、コードレビューの補完ツールとしての側面として有用なことを意味しています。


---

### Lintの側面: 落とし穴の形式知化
おおよそのLintのルールは、先人が踏んだ地雷の教訓です。

たとえば、さきほどの`'use strict'`問題は、実際にAmazonで起きていたことです。

この問題が、Lintのルールとして明文化されたからこそ、広く知られるようになりました。

もしLintに組み込まれていなければ、知る人ぞ知る落とし穴として、人知れず開発者を苦しめていたことでしょう。

このように、Lintは、地雷の被害者のみの暗黙知を形式知化する側面があります。


---

### Lintの側面: 教材
はじめての言語をLintとともにはじめるのはとてもよいことです。

Lintは先人が歩んだ地雷源のガイドのようなものだからです。

はじめから地雷を踏まない歩き方を学ぶことで、早く上達できます。

また、多くの場合、地雷の知識は言語やライブラリに深い理解をもたらします。

実際に、Lintの原典である「Perl Best Practice」や「JavaScript: Good Parts」の評判は非常に高いです。


---

### Lintの側面: 紛争の予防手段
コーディングスタイルは紛争の種なので、はじめからコーディングスタイルをきめておくことは重要です。

文章でもいいが、網羅的に担保できるLintの方が望ましいです。

これを怠って、GitHubで怒られたことがあります。


「コーディングスタイルについてとやかく言うなら、スタイルチェッカーいれろ」

まったくその通りで、スタイルチェッカーがあれば、相手の時間を節約できたはずです。

つまり、紛争を予防し、双方の時間を節約するためのツールという側面があることがわかります。


---

### Lintの側面: 動的型検査に静的型検査つけるより楽に実装できるお手軽手法
プログラミングのうち、デバッグ時間の割合が大きいのはよく知られています。

なるべくデバッグ時間を減らすために、静的型検査をつけたくなることがよくあります。


ただ、後付けで静的型検査をつけるのは非常に難しいのです。

静的型検査がなかった言語は、静的型検査に向かない仕様を多くもつからです。


そこで、少しリスクを許容して、Lintレベルの実装にすると、動的型検査言語であろうがすぐ実装できます。

つまり、デバッグ時間を減らすための簡便な方法という側面を持っています。

---

### ここまでのまとめ

- Lintにはいろいろな側面がある
    - コードレビューの省力化ツール
    - コードレビューの品質向上ツール
    - 落とし穴の形式知化
    - 教材
    - 紛争の予防手段
    - 動的型検査に静的型検査つけるより楽に実装できるお手軽手法

---

## エラープルーフ機構の比較
では、他のエラープルーフ機構からみたLintはどうでしょうか。

エラープルーフの機構のなかでどのような立ち位置なのかを確認してみましょう。


---

### 速度とリスクのトレードオフが存在する
エラープルーフ機構は、速度とリスク抑制の関係がほぼトレードオフの関係にあります。


- もっとも高速な機構: 構文ハイライト
- 高速な機構: Lint ユニットテスト
- 低速な機構: インテグレーションテスト
- もっとも高速な機構: E2Eテスト

---

- もっとも残存リスクが高い機構: 構文ハイライト
- 残存リスクが高い機構: Lint ユニットテスト
- 残存リスクが低い機構: インテグレーションテスト
- もっとも残存リスクが低い機構: E2Eテスト

---

つまり、Lintは高速だがリスク抑制は大きくないという位置付けです。

---

このトレードオフから、別のこともみえてきます。

Lintが検証できる問題をインテグレーションテストやE2Eテスト発見してしまうと、フィードバックが遅くなるのです。

つまり、高速な機構で検証できる問題は、高速な機構に任せるべきです。

Lintは、高速さを損なわない範囲で、構文ハイライトで検証できる問題よりも大きな問題を取り扱うべきです。

---

さて、Lintの背景説明はこれまでにしておきましょう。

ここからは、Lintとテストレフームワークの関係をみていきます。

---

## 疑問: Lintとテストフレームワークは何が違うのだろうか

---

### Lintとテストの両方をかけるためのシステム構成が複雑すぎる
最近のJS界隈は、ESLint + JSCS + Mocha + Karma + ... というように、多段のエラープルーフが別々のモジュールで提供されています。

これによって、多段のエラープルーフ機構を組み合わせるために、スクリプトが必要となるほど複雑な構成になっています。

そのため、これらのエラープルーフ機構のうち、Lintとテストレフームワークを共通化できないかと悩んでいました。


---

### 同じこと
- コードに潜む問題を探し、報告すること

### 違うこと
- 検査ルールは
  - 開発者がつくる（テストフレームワーク）
  - あらかじめ用意されている（Lint）

---

### 見えてきた違い
つまり、こういうことではないでしょうか？

- [非共通] 検査ルールは、開発者が用意する/あらかじめ用意されている
- [共通] コードに潜む問題を、検査ルールにもとづき収集・整形・表示する機能（テストランナーの機能）

---

#### 人的資源の選択と集中
非共通部分については、人的資源の選択と集中の問題にみえます。

本来、検査ルールはすべて開発者が実装してもよいはずです。

だが、検査ルールがマイナーでなく、かつ機械で検査できるものについては、誰かが一度実装すればそれですむのです。

---

つまり、Lintテストとレフームワークを取り巻く環境の本質は、検査ルール実装の分業にあると考えています。

そして、テストレフームワークの機能のうち、テストランナーの機能については共通化できるはずです。

---

#### 検査ルール実装の分業の線引き
また、分業の線引きについてもおもしろい見方があると考えています。


---

歴史的に見れば、機械でできることは、ほとんどLintが担ってきました。

つまり、線引きはこうなっているのではないでしょうか？

- 機械でもできることが、Lintのやること
- 機械ではできないことが、テストフレームワーク+人間のやること

---

現状の「機械でできる」のラインは構文解析でわかるかどうか程度にみえます。

しかし、将来にわたってもこの傾向は続くのでしょうか？

---

#### Lintが人間から仕事を奪えるかも？: 変数名の妥当性推論
最近、難読化されたJSの変数名を機械学習によって推論する、プロジェクト「[JSNICE](http://www.jsnice.org/)」がニュースになりました。

このように、機械学習を応用すれば、もっと「機械ができる」範囲が広がるのではないでしょうか？

たとえば、変数名の妥当性検証のように。

---

### ここまでのまとめ

- Lintテストとレフームワークの違いの本質は、検査ルール実装の分業
- テストレフームワークの機能のうち、テストランナーの機能については共通化できそう

---

## Lintのこれから
最後に、これからのLintがどうなっていくかを予想しようと思います。

---

### Lint on テストランナー
テストフレームワークの、テストランナーにあたる部分はLintと共通化できます。

そのため、テストランナーのうえで一体として実行できるLintが登場すると考えています。


---

### 機械学習
先ほども言及したが、検査ルールに機械学習の成果が活用されるブレークスルーがおこると予想しています。


---

### フレームワークのLintの台頭
これまで紹介してきたLintは、どれも言語に対するLintでした。

しかし、Lintの紐づくものを言語にとどめておくのはもったいないと思いませんか？

言語固有の落とし穴以外にも、フレームワーク固有の落とし穴があるものです。

これからは、フレームワーク固有の落とし穴を検査するLintが脚光を浴びてくると考えています。

---

### ここまでのまとめ

LintFreakの未来予想：

- Lint on テストランナーの登場
- Lint with 機械学習のブレークスルー
- Lint for Frameworkの浸透

---

## 余談: Lintを開発していてわかったこと
もし興味があれば、捕まえて聞いてください。

- ダメLintの三原則
- 検査ルールのプラグイン化にあたって目指すべき設計
