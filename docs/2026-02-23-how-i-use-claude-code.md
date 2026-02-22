# Claude Codeの使い方 ― 私のワークフロー

**Original:** https://boristane.com/blog/how-i-use-claude-code/  
**Author:** Boris Tane  
**Date:** 2026-02-23

---

Claude Codeを9ヶ月間にわたって主要な開発ツールとして使ってきた著者が、自身のワークフローを公開する記事。「プロンプトを打って、エラーを修正して繰り返す」という典型的な使い方とは根本的に異なる、計画駆動型のアプローチを解説している。

## コアの原則

**「Claudeにコードを書かせる前に、必ず書面による計画をレビューして承認する」**

計画と実行を分離することが最も重要で、これにより無駄な作業を防ぎ、アーキテクチャの決定権を自分が持ち続けられる。

## フェーズ 1: リサーチ

タスク開始時に必ず「深読みディレクティブ」から入る。

```
read this folder in depth, understand how it works deeply, what it does 
and all its specificities. when that's done, write a detailed report of 
your learnings and findings in research.md
```

**「深く」「詳細に」「あらゆる点において」という言葉が重要**。これらを省くとClaudeは表面だけさらって先に進む。

`research.md` という成果物が核心。これはClaudeに宿題をさせるためではなく、**自分のレビュー面**として機能する。研究が間違っていれば計画も間違い、実装も間違う。最も高コストな失敗パターン（既存のキャッシュ層を無視した関数、ORMのコンベンションを考慮しないマイグレーションなど）はすべてここで防げる。

## フェーズ 2: 計画

リサーチのレビュー後、別の `plan.md` ファイルに詳細な実装計画を要求する。

```
I want to build a new feature <name> that extends the system to perform 
<outcome>. write a detailed plan.md document outlining how to implement 
this. include code snippets
```

**組み込みのplanモードは使わない**。Markdownファイルなら自分のエディタで編集でき、永続的なアーティファクトとして残る。

うまく動くOSSの参照実装がある場合は、そのコードをコンテキストとして渡すと効果的。「ゼロから設計させる」より「具体的な参照から学ばせる」方が大幅に良い結果が出る。

## アノテーションサイクル（最大の差別化点）

計画書をエディタで開き、**Claudeの計画に直接インラインノートを追記する**。

実際のノート例:
- `"use drizzle:generate for migrations, not raw SQL"` — Claudeが知らないドメイン知識
- `"no — this should be a PATCH, not a PUT"` — 間違った前提の修正
- `"remove this section entirely, we don't need caching here"` — アプローチの却下
- `"this is wrong, the visibility field needs to be on the list itself"` — セクション全体の方向修正

ノートを追記したらClaudeに返す:

```
I added a few notes to the document, address all the notes and update 
the document accordingly. don't implement yet
```

**「don't implement yet（まだ実装するな）」というガードが必須**。これがないとClaudeは十分と判断した瞬間にコードに飛びつく。

このサイクルを1〜6回繰り返す。Markdownファイルが「自分とClaudeの間の共有可変状態」として機能する。チャット履歴をスクロールして決定を再構成する必要がない。3回のアノテーションサイクルで、汎用的な実装計画を既存システムに完璧に合致したものに変えられる。

## Todo リスト

実装前に必ず細かいタスク分解を要求する:

```
add a detailed todo list to the plan, with all the phases and individual 
tasks necessary to complete the plan - don't implement yet
```

数時間かかるセッションでも、どこまで進んだか一目でわかる進捗トラッカーになる。

## フェーズ 3: 実装

計画が固まったら実装コマンドを出す:

```
implement it all. when you're done with a task or phase, mark it as 
completed in the plan document. do not stop until all tasks and phases 
are completed. do not add unnecessary comments or jsdocs, do not use 
any or unknown types. continuously run typecheck to make sure you're 
not introducing new issues.
```

このプロンプトにはすべてが詰まっている:
- "implement it all" — 計画全体を、選択的にではなく
- "mark it as completed" — 計画書が進捗の唯一の情報源
- "do not stop until all tasks are completed" — 途中確認のポーズ禁止
- "continuously run typecheck" — 最後ではなく早期にキャッチ

「実装は退屈であるべき」。創造的な作業はアノテーションサイクルで済んでいる。実装はその忠実な実行であるべきで、ここで驚きがあってはいけない。

## 実装中のフィードバック

一旦実装が始まったら、自分の役割は「建築家」から「監督」に変わる。修正は超短文:

- `"You didn't implement the deduplicateByTitle function."`
- `"wider"`
- `"there's a 2px gap"`

フロントエンド作業にはスクリーンショットを活用。既存コードへの参照も有効：「このテーブルはusersテーブルと全く同じ見た目にして」。

方向が間違ったらgitで差し戻して再スコープする。「インクリメンタルに修正する」より「差し戻してスコープを絞る」ほうが常に良い結果が出る。

## 長時間のシングルセッション

リサーチ・計画・実装を一つの長いセッションで完結させる。コンテキストウィンドウ50%を超えてもパフォーマンス劣化は見られないという。計画書へのアノテーション作業がコンテキストを整理し続けているためと考えられる。

---

**Tags:** #claude-code #ai-workflow #development #planning #software-engineering

**Curation Note:** Claude Codeの実践的ワークフローとして非常に精度が高い。「計画と実行の分離」「アノテーションサイクル」「plan.mdの共有可変状態化」という3つのアイデアは、AIコーディングツールの使い方を根本から変える可能性がある。特にアノテーションサイクルのコンセプトは独自性が高く、長期的に参照価値がある記事。
