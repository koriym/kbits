# 自作Gitで学ぶバージョン管理の内部構造

**原題:** I made my own git  
**著者:** TonyStr  
**出典:** https://tonystr.net/blog/git_immitation  
**日付:** 2026-01-27  
**アーカイブ日:** 2026-01-27

---

## 要約

開発者TonyStr氏が、Gitの内部構造を理解するために独自のバージョン管理システム「tvc (Tony's Version Control)」を実装した記録。ハッシュ、圧縮、ツリーオブジェクト、コミットオブジェクトという基本要素を、Rustで実装しながら解説している。

### 主要なポイント

#### 1. Gitはハッシュベースのキーバリューストア

Gitの全てはハッシュを中心に構成されている：

- ファイルをコミット → ハッシュ化して `.git/objects/` に保存
- ツリーオブジェクト（ファイルとディレクトリのリスト + ハッシュ）を作成 → ハッシュ化して保存
- コミットオブジェクト（ツリーハッシュ + 親コミット + メタデータ）を作成 → ハッシュ化して保存

**設計判断：**
- Git は SHA-1（暗号学的に破られている）を使用
- tvc は SHA-256 を採用（互換性よりも新規性）
- 圧縮アルゴリズムも zlib ではなく zstd を選択

#### 2. 実装の流れ

必要な機能を順に実装：

1. コマンドライン引数の読み取り
2. `.tvcignore` ファイルの処理
3. `ls` コマンド（無視されないファイルをリスト）
4. ファイルのハッシュ化
5. ファイルの圧縮・解凍
6. ツリーオブジェクトの生成
7. コミットオブジェクトの生成
8. HEAD ファイルの管理
9. コミットのチェックアウト

#### 3. コミットの構造

```rust
/* ===== commit format =====
<type> \0
tree\t<tree>\n
parent\t<parent>\n
author\t<author>\n
message\t<message>\n
*/
```

コミットには以下の情報が含まれる：
- オブジェクトの種類（commit）
- ファイルシステムの状態（tree）
- 前のコミット（parent / HEAD）
- 著者（author）
- コミットメッセージ

Gitは author と committer を区別するが、tvc では実装を省略（大半のコミットで同じため）。

#### 4. ツリーの再帰的生成

`generate_tree()` がヘビーリフティングを担当：

1. ルートディレクトリをループ
2. 各ファイルをハッシュ化・圧縮して `.tvc/objects/` に保存
3. ファイル名とハッシュを文字列に追加
4. サブディレクトリは同じ関数で再帰的に処理
5. ツリーオブジェクト自体もハッシュ化して保存

**最適化：**
前回のコミットから変更されていないファイルは、同じハッシュを生成するため、新しいファイルは書き込まれない。数百ファイルのリポジトリで1-2ファイルだけ変更した場合、大半のファイルは同じオブジェクトを指す。

#### 5. チェックアウトの実装

オブジェクトフォーマットをパースして構造体に変換：

```rust
#[derive(Debug, Clone)]
struct Commit {
    tree: String,
    parent: String,
    author: String,
    message: String,
}

#[derive(Debug, Clone)]
struct Tree {
    trees: Vec<(String, Tree)>,     // (path, Tree)
    blobs: Vec<(String, String)>,   // (path, hash)
}
```

ツリー構造をメモリ上に再現したら、`generate_fs()` でファイルシステムを生成：

```rust
impl Tree {
    fn generate_fs(&self, path: &Path) -> std::io::Result<()> {
        fs::create_dir(&path).unwrap_or(());

        for (dirname, blob) in &self.blobs {
            let file = decompress_object(blob)?;
            fs::write(path.join(dirname), file.as_bytes())?;
        }

        for (dirname, tree) in &self.trees {
            tree.generate_fs(&path.join(dirname))?;
        }

        Ok(())
    }
}
```

#### 6. 学んだこと

**Git = コンテンツアドレサブルなファイルストア（キーバリューデータストア）**

この認識が最も重要。プロジェクトで最も難しかったのは実はパース処理。もし再実装するなら、YAML や JSON のような定義された言語でオブジェクト情報を保存するだろう。

---

## 論評

この記事は、「車輪の再発明」が最高の学習方法であることを示す好例。

### なぜ優れているか

1. **抽象からの脱却**  
   Gitを「ブラックボックス」から「理解可能なシステム」に変える。`git commit` の裏で何が起きているかを体感できる。

2. **実装の順序が教育的**  
   ハッシュ → 圧縮 → ツリー → コミット → チェックアウトという順序は、Gitの概念的な層にも対応している。

3. **設計判断の透明性**  
   「なぜSHA-256を選んだか」「なぜauthorとcommitterを統合したか」という判断が明示されており、トレードオフを学べる。

### 技術的洞察

**Git の本質 = ハッシュベースの不変データ構造**

- ファイル内容が変わらなければハッシュも変わらない
- 同じハッシュは同じオブジェクトを指す
- これにより、変更されたファイルだけが新しいオブジェクトとして保存される

この「コンテンツアドレサブル」という概念は、ブロックチェーンやIPFSなど、他の分散システムにも共通する基礎原理だ。

### 実装言語としてのRust

Rustを選んだ理由は語られていないが、Gitのような低レベルシステムツールには適している：

- メモリ安全性（ファイルシステム操作で重要）
- パフォーマンス（ハッシュ計算・圧縮は重い）
- エコシステム（sha256、zstd のクレートがすぐ使える）

ただし、著者自身が「Rust は苦手」と認めているように、パース処理が冗長になりがち。YAMLやJSONを使えばもっと簡潔に書けただろう。

### 「車輪の再発明」の価値

> 「ドキュメントを読むだけでは理解できない。実装して初めて分かる。」

これは開発者の普遍的真理。特に：

- データベース（SQLiteクローン）
- インタプリタ（言語実装）
- ネットワークプロトコル（HTTPサーバー）
- バージョン管理（この記事）

などの基礎インフラを「小さく」再実装することは、システムプログラミングの最高の教材になる。

### 日本の開発者への示唆

日本のGit利用者は多いが、内部構造を理解している人は少ない。「使えればいい」という実用主義は重要だが、基礎を理解することで：

- トラブル時に原因を推測できる
- パフォーマンス特性を理解できる
- Git以外の分散システムにも応用できる

この記事は、技術系ブログの理想形でもある：**短く、具体的で、実装可能**。

---

## 引用

> "Version control used to be a black box for me; I had no idea how files were stored, how diffs were generated or how commits were structured. Since I love reinventing the wheel, why not take a stab at git?"

> "This was a fun exercise. It really drove home the idea that git is just a content-addressable file store (a key-value data store)."

> "The hardest part about this project was actually just parsing. If I were to do this again, I would probably use a well-defined language like yaml or json to store object information."

---

## リンク

- **ソースコード:** [GitHub](https://github.com/TonyStr/tvc) *(記事内に言及)*
- **参考:** [zstd圧縮](https://facebook.github.io/zstd/)

---

**タグ:** #Git #バージョン管理 #Rust #学習 #実装 #システムプログラミング

**関連トピック:** #分散システム #ハッシュ #データ構造 #ファイルシステム
