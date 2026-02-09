# 誰もシステム全体の仕組みを知らない

**原文:** [Nobody knows how the whole system works](https://surfingcomplexity.blog/2026/02/08/nobody-knows-how-the-whole-system-works/)  
**著者:** Lorin Hochstein  
**日付:** 2026-02-08  
**アーカイブ日:** 2026-02-09

---

## 要約

現代のソフトウェア開発において、システム全体の仕組みを完全に理解している人間は誰もいない――これは危機なのか、それとも必然なのか？

Lorin Hochsteinが、LinkedIn上で交わされた3つの投稿（Simon Wardley、Adam Jacob、Bruce Perens）と、1994年のMIT教授Louis Bucciarelliの洞察を通じて、「理解」の限界について考察した記事。

### 核心的な問い

**Simon Wardley:** AI時代において、仕組みを理解せずに物を作ることの危険性を懸念。

**Adam Jacob:** AIは確かに新しい能力だが、その利益はリスクを上回る。根本的な仕組みからさらに遠ざかるが、それは避けられない変化。

**Bruce Perens:** Wardleyが恐れるシナリオは、実はすでに起きている。現代のCPUやOSは複雑すぎて、多くの開発者は実際の仕組みを知らない。

**Louis Bucciarelli (1994):** 電話の仕組みを「知っている」とはどういうことか？振動する振動板の物理？ルーティングアルゴリズム？衛星通信？企業間の規制？――誰も電話の仕組み全体を知らない。

### 著者の考察

- **「魔法」としてのフレームワーク:** Ruby on Railsのような「魔法」は、下層の仕組みを意図的に隠蔽することで生産性を高める。Wardleyの懸念はここにある。
- **面接質問の深さ:** 「URLを入力してEnterを押すと何が起きるか？」――この質問には際限がない。HTTPから始まり、DNS、TCP/IP、OSの割り込み処理、Wi-Fiの変調方式（QAM vs QPSK）、ARMプロセッサのメモリモデル、JVMのガベージコレクション、トランジスタのデジタルロジック……どこまで答えられるか？
- **Brendan Greggの面接哲学:** 候補者の知識の「限界」を見つけ、そこでどう反応するかを観察する。「知らない」と認めるか、ごまかすか――誰もシステムの底まで理解していないことを前提にしている。

### 結論

4人全員が正しい：

- **Wardley:** 理解せずに作ることは危険
- **Jacob:** AIの利益はリスクを上回る
- **Perens:** その状況はすでに起きている
- **Bucciarelli:** 複雑なシステムは本質的に、誰も全体を理解できない

そしてAIは、この状況をさらに悪化させる。しかしそれは新しい問題ではない――**私たちはずっとこの状況の中にいた**。

---

## コメント

この記事が示しているのは、「抽象化」と「理解」のトレードオフが、ソフトウェア工学の本質的なジレンマだということ。

**「魔法」の必要性:**

Wardleyは「魔法」を危険視するが、魔法なしでは現代のソフトウェア開発は成り立たない。Ruby on Railsの「魔法」は生産性を爆発的に高めた。問題は「魔法を使うこと」ではなく、「魔法が壊れたときに対処できないこと」。

**AI時代のエンジニアリング:**

AIが書いたコードを人間が理解できない――これは新しい恐怖のように聞こえるが、実際には既存の問題の延長線上にある。私たちはすでに、完全には理解していないシステムの上で開発している。ARMのメモリモデル、TCP/IPスタック、JVMの最適化……どこまで理解しているだろうか？

**重要な教訓:**

> "Nobody knows how the whole system works"

これは諦めではなく、前提条件。その前提の上で、どう設計し、どうテストし、どう保守するか――それこそが現代のソフトウェア工学の本質。

---

## 関連リンク

- [Active knowledge](https://surfingcomplexity.blog/2023/06/26/active-knowledge/) - Bucciarelliの引用元
- [ARM sequential consistency](https://developer.arm.com/community/arm-community-blogs/b/tools-software-ides-blog/posts/armv8-sequential-consistency)
- [Brendan Gregg's blog](https://www.brendangregg.com/)

---

*この記事は個人ブログからのサマリーです。全文は原文をご覧ください。*
