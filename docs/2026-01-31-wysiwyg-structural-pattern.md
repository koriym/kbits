# What You See Is What It Does: A Structural Pattern for Legible Software

## メタデータ
- **タイトル**: What You See Is What It Does: A Structural Pattern for Legible Software  
- **著者**: Daniel Jackson (MIT CSAIL)  
- **出典**: arXiv:2508.14511v2 (Onward! '25 Conference Proceedings)  
- **公開日**: 2025年8月  
- **アーカイブ日**: 2026年1月31日  
- **URL**: https://arxiv.org/html/2508.14511v2

---

## 要約

この論文は、LLM（大規模言語モデル）を用いたコーディングの台頭により露呈したソフトウェア開発の構造的問題に対処する新しいアーキテクチャパターンを提案している。

### 問題意識

現代のソフトウェアは「illegible（読み取り不可能）」である——コードと観察可能な振る舞いの間に直接的な対応がなく、モジュール性も不十分。これにより、堅牢なコーディングに必要な3つの要件が満たされない：

1. **Incrementality（漸進性）**: 局所的な変更で小さな機能追加を実現できること
2. **Integrity（完全性）**: 既存機能を壊さないこと  
3. **Transparency（透明性）**: ビルド時の変更内容、実行時のアクション履歴が明確であること

### 提案する解決策: Concepts と Synchronizations

**Concepts（概念）**  
- ユーザー向けの独立したサービス単位で、明確な目的を持つ  
- 例: Post、Comment、Upvote、Friend、Password など  
- 各Conceptは独自の状態機械として定義され、他のConceptに依存しない  
- マイクロサービスと似ているが、概念間で相互呼び出しや状態参照は行わない

**Synchronizations（同期ルール）**  
- イベントベースの粒状なルールで、Concept間のデータとフローを調停  
- 「このアクションが発生したとき、この条件下で、これらのアクションを実行する」という形式  
- 宣言的で細粒度なため、LLMによる生成に適している

### アーキテクチャの特徴

1. **完全な分離**: Conceptは互いを知らず、Synchronizationのみが協調を担当  
2. **状態の可視性**: Conceptは状態を抽象的に公開（GraphQL/SPARQLなど）し、ゲッターを不要にする  
3. **Provenance（来歴追跡）**: すべてのアクションは、それを引き起こしたSynchronizationと因果関係のあるアクションに遡れる  
4. **Flow Token**: 単一のWebリクエストに起因するすべてのアクションを関連付ける仕組み  
5. **Named Arguments**: 部分的パターンマッチングを可能にし、可読性を向上

### ケーススタディ: RealWorld Benchmark

Medium風のブログプラットフォーム「Conduit」を実装:

1. 各Conceptの仕様をLLMで生成（単発プロンプトでほぼ成功）  
2. 仕様からコードをLLM生成  
3. Synchronizationsを生成（数回のイテレーションが必要）  
4. Postman自動テストスイートに合格

**LLM生成の結果**:  
- Concept仕様とコードはほぼワンショット生成に成功  
- Synchronizationsは数回の反復が必要だったが、既存の実装よりもモジュール性が高い設計を実現  
- 例: Favorite機能は他の実装ではArticleとUserモデルに分散していたが、この実装では独立したFavorite Conceptに集約

### 従来のアーキテクチャとの比較

多くのRealWorld実装は3層構造（Router、Controller、Model）を採用しているが、以下の設計ルール違反が頻発:

- ControllerがModelをバイパスして直接DB操作  
- 他のControllerのModelにアクセス（ArticleがUserやTagにアクセス）  
- Router層がControllerをスキップして直接呼び出し

提案手法では、設計ルールが構造的に保証される:

- Conceptのアクションは他のConceptの状態やアクションにアクセスしない  
- Synchronizationsは Conceptの状態とアクションのみにアクセス  
- Bootstrap concept（Web）のみが外部刺激を受け付ける

---

## コメント

### アーキテクチャ思想の深化

この論文は、LLMによるコード生成という新しい現実に対応するために、ソフトウェアアーキテクチャの根本的な再考を促す野心的な試みだ。Daniel Jacksonは以前の著書 *The Essence of Software* (2021) で Concept Design の理論を提唱したが、本論文はそれを実装可能な形に昇華している。

**興味深いのは、以下の逆説的な発見**:  
LLMのために設計された構造が、人間のプログラマーにとっても有益であるということ。粒度の高いSynchronizationsは、複雑なルートハンドラーを置き換え、宣言的で理解しやすいコードを実現する。これは「AIファースト設計」が必ずしも人間の可読性を犠牲にしないという示唆を与える。

### モジュール性の再定義

従来のオブジェクト指向では、エンティティ（Article、User）ごとにモジュールを分割するが、「関係的な側面」（FavoriteやFollow）の扱いが曖昧になりがちだった。Conceptベースの設計では、これらを独立した一級市民として扱うことで、より自然なモジュール分割を実現している。

### Provenance と Transparency

Flow TokenとSynchronization来歴追跡の仕組みは、デバッグとオーディティングを劇的に改善する可能性がある。特に、マイクロサービス環境での分散トレーシングや、コンプライアンスが求められるシステムでの応用が期待できる。

### 実装上の課題

論文では触れられていないが、実用化には以下の課題がありそう:

1. **パフォーマンス**: 細粒度のSynchronizationsは、実行時のオーバーヘッドを増加させる可能性  
2. **学習曲線**: 新しいパラダイムへの移行コスト  
3. **ツールチェーン**: RDF/SPARQL前提の実装は既存のエコシステムとの統合が課題

### LLM時代のソフトウェア工学

この論文は、LLMが単なる「コーディング支援ツール」ではなく、ソフトウェアアーキテクチャそのものを再考させる契機であることを示している。「仕様が復活した（Specifications are back）」という指摘は鋭い——アジャイル開発が軽視してきた非コード成果物が、プロンプトという形で必須になった。

概念設計とLLM生成を組み合わせることで、**仕様駆動開発（Specification-Driven Development）の新しい形**が見えてくる。Concept仕様は人間にとって理解可能であり、LLMにとっても生成可能である。この「二重の可読性」が鍵になるだろう。

---

## 関連トピック

- #software-architecture #modularity #concept-design  
- #llm-code-generation #ai-assisted-development  
- #synchronization #event-driven-architecture  
- #legibility #transparency #provenance  
- #microservices #domain-modeling

---

## 参考文献

- Jackson, D. (2021). *The Essence of Software: Why Concepts Matter for Great Design*. Princeton University Press.  
- Jackson, D. (2006). *Software Abstractions: Logic, Specification, and Analysis*. MIT Press.  
- Jimenez et al. (2024). SWE-bench: Can Language Models Resolve Real-World GitHub Issues?  
- Aleithan et al. (2024). A critical study of what code-LLMs (do not) learn.

---

**タグ:** #ソフトウェアアーキテクチャ #LLM #コーディング #概念設計 #可読性 #モジュール性 #イベント駆動

**関連トピック:** #ソフトウェア工学 #デザインパターン #AI開発 #保守性

---

**推奨理由**: LLM時代のソフトウェアアーキテクチャに対する根本的な問いを投げかけ、実装可能な解決策を提示している。概念設計とイベント駆動型の同期ルールを組み合わせることで、可読性・モジュール性・保守性を同時に達成する野心的な試み。長期的な参照価値がある。
