# XMLは安価なDSLである

**原題:** XML is a Cheap DSL  
**著者:** unplannedobsolescence.com  
**公開日:** 2026-03-13  
**ソースURL:** [https://unplannedobsolescence.com/blog/xml-cheap-dsl/](https://unplannedobsolescence.com/blog/xml-cheap-dsl/)  
**アーカイブ日:** 2026-03-15

---

## 要約

### 背景：IRS Tax Withholding Estimator

著者はIRS（米国内国歳入庁）の新しい[Tax Withholding Estimator](https://www.irs.gov/individuals/tax-withholding-estimator)（TWE）のエンジニアリングリードを務めた。このツールは納税者が源泉徴収額を推計するためのもので、無料・オープンソースかつ[公開コントリビューション対応](https://github.com/IRS-Public/tax-withholding-estimator)という画期的な特徴を持つ。著者はその経験から「XMLは現代のソフトウェア開発においても有効な選択肢であり、クロスプラットフォームな宣言的仕様のDSLとして最も安価な実装手段だ」という結論に至った。

### Fact Graphと宣言的税法モデル

TWEは2つのXML設定ファイルから生成される静的サイトである。核心となる「Fact Dictionary」はUS Tax Codeを表現したXML設定で、「Fact Graph」と呼ばれるロジックエンジンがこれを解釈して税務計算を行う。

たとえば「総支払額（totalOwed）」は次のように表現される：

```xml
<Fact path="/totalOwed">
  <Derived>
    <Subtract>
      <Minuend>
        <Dependency path="/totalTax"/>
      </Minuend>
      <Subtrahends>
        <Dependency path="/totalPayments"/>
      </Subtrahends>
    </Subtract>
  </Derived>
</Fact>
```

非還付クレジットは税負担をゼロ以下にできないため、`<GreaterOf>`演算子でゼロとの大きい方を取る設計も自然に表現できる。入力値（writable）と計算値（derived）の区別も明確だ。

### なぜJavaScriptではダメか

同じ計算をJavaScriptで書くことは一見シンプルだが、すぐに問題が露呈する：

**実行順序の問題**：`getInput()`関数はユーザー回答を待つ必要があり、関係ない計算まで待たされる。かつ「配偶者がいない場合は配偶者の収入を聞かない」ような条件分岐もあり、上部に一括で質問を置くことも不可能だ。

**実装詳細の混入**：ソーシャルセキュリティ収入の合算に`map`と`reduce`が必要になる瞬間、コードは税務の概念から外れてJavaScriptの実装詳細に踏み込んでしまう。

**監査可能性の欠如**：命令型プログラムは中間値を捨てるため「どうやってその値を計算したか」を後から問い合わせることができない。US Tax Codeは数百の中間計算からなり、デバッガで逐一確認するスケールではない。

宣言的なFact Graphでは、すべての計算に自動的に監査可能性と内省能力が付与される。IntuitもTurboTaxで同様の結論に達し、2020年に[Tax Knowledge Graph白書](https://arxiv.org/pdf/2009.06103)を発表している（ただし非公開）。IRS Fact Graphはオープンソース・パブリックドメインだ。

### JSONよりXMLが優れている理由

JSONで同じFact表現を試みると：

```json
{
  "definition": {
    "type": "Expression",
    "kind": "GreaterOf",
    "children": [
      { "type": "Value", "kind": "Dollar", "value": 0 },
      {
        "type": "Expression",
        "kind": "Subtract",
        "minuend": { "type": "Dependency", "path": "/totalTentativeTax" },
        "subtrahend": { "type": "Dependency", "path": "/totalNonRefundableCredits" }
      }
    ]
  }
}
```

JSONが利用できる複雑なデータ構造はオブジェクトのみであるため、すべての子オブジェクトが自分の「種類」を宣言する必要がある。XMLでは「種類」がデリミタ自体に埋め込まれている。

また、XMLにはJSONにない利点がある：コメントが書ける、長いテキストのホワイトスペース処理が自然、属性と名前付き子要素で表現の重みを制御できる、`<Dollar/>`や`<Boolean/>`のような独自型を定義しやすい。

YAMLについては原著者の[Chris Given](https://chrisgiven.com/)が「何があってもInternal Revenue CodeのロジックをYAMLで表現するな」と述べている。

### XMLが安価な理由：ユニバーサルなエコシステム

S式やPrologやKDLでも同様のDSLを構築できる。これらは文法的にはXMLより美しいかもしれない。しかしXMLを使うことの本質的な価値は別にある。

**パーサーと普遍的なツールエコシステムが無料でついてくる。**

著者がFact定義をファジー検索したくなったとき、シェルコマンド一発で解決した：

```bash
cat facts.xml | xpath -q -e '//Fact/@path' | grep -o '/[^"]*' | fzf
```

さらに依存関係の逆引き検索も60行のbashで構築できた。チームの複数メンバーが独自のデバッグツールを作り、それが全員のワークフローの一部になった。すべてのツールが「Fact GraphのScala実装に触れずに、XMLを各自の得意言語でパースできること」に依存していた。

PrologからはXMLを[単一の述語](https://www.scryer.pl/sgml)で変換できる。XSLTで変換、XPathでクエリ。S式はLispで動き、Prologタームはprologで動く。XMLはあらゆるものに変換できるため、優れたカノニカル・クロスプラットフォームデータフォーマットとなる。

**結論**：汎用データ表現は金の重みがある。その選択肢は実質JSONとXMLしかない。ほとんどの場合JSONを選べばよい。しかしDSLが必要なときは、XMLが圧倒的に安価な選択肢であり、そのコスト効率がチームのイノベーション予算を別の場所に使わせてくれる。

---

## 論評

「XMLは時代遅れ」という認識は広く共有されているが、本記事はその前提を実際のプロダクション経験から覆す。重要なのは「XMLが優れているか」ではなく「DSLを構築するコストをどう最小化するか」という問いへの答えとしてXMLを位置づけている点だ。

宣言的vs命令的の対比は教科書的に見えるが、著者はそれをUS Tax Codeという具体的・複雑なドメインで実証している。「中間値を捨てる命令型プログラムは、数百の計算からなる税務ロジックの監査に耐えられない」という論点は、設計の哲学として説得力がある。

XMLのエコシステム価値（XPath・XSLT・あらゆる言語のパーサー）が「無料でついてくる」という観察は、DSL設計の実務判断として長く参照に値する。特に「カスタム構文を選ぶたびにツールチェーンも自作することになる」というトレードオフを明示している点が鋭い。

---
