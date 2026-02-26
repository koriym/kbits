# OpenClawセキュリティ強化完全ガイド：3段階実装

**原題:** How to Harden OpenClaw Security: Complete 3-Tier Implementation Guide  
**著者:** Fernando Lucktemberg（aimaker.substack.com 寄稿）  
**公開日:** 2026年2月  
**ソースURL:** https://aimaker.substack.com/p/openclaw-security-hardening-guide  
**アーカイブ日:** 2026-02-26

---

## 要約

OpenClawはGitHub史上類を見ない速度でバイラル拡散したAIエージェントプラットフォームだが、デフォルト設定のままでは重大なセキュリティリスクを抱える。本記事はセキュリティ専門家のFernando Lucktemberg（Next Kick Labs）による実践的なセキュリティ強化ガイドで、「原則の説明」ではなく「実際に動くコマンドと設定」を提供することを目的としている。

### ガイドの前提

著者は本ガイドを「エンドースメント」ではなく「ハームリダクション」と位置づける。セキュリティリスクを受け入れた上でOpenClawを使う人々が、最低限の被害で実験できるよう支援するのが目的。「やめろ」という警告は既に出尽くしており、それでも使う人はいる。

**絶対に接続してはいけないアカウント（NEVER LIST）:**
- 主要メールアカウント
- 銀行・金融サービス
- パスワードマネージャー
- 仕事用アカウント（Slack、Google Workspace等）
- 置き換え不可能な履歴を持つSNSアカウント
- 暗号資産ウォレット
- 政府・医療ポータル
- 主要GitHubアカウント

接続可能なのは「失っても1時間で再構築できるサービス」のみ。

---

### Tier 1：最低限の保護（所要時間：3〜4時間）

**Tier 1は必須。スキップしたらOpenClawを使うな。**

主要ステップ：

| ステップ | 内容 |
|---------|------|
| 1 | 隔離されたVPS（Hetzner $3.49/月等）またはサブマシンで動かす |
| 2 | UFWファイアウォール設定（ポート18789は公開しない） |
| 3 | Tailscaleインストール（暗号化リモートアクセス） |
| 4 | Node.js 22.12.0以上（古いバージョンは権限バイパス脆弱性あり） |
| 6 | ファイルパーミッション700/600で認証情報を保護 |
| 7 | gateway.yamlで`host: "127.0.0.1"`（0.0.0.0は絶対禁止）、`dangerouslyDisableDeviceAuth: false` |
| 8 | ファイルシステムアクセス制限（.ssh、credentials等はdeny） |
| 9 | `age`または`pass`で認証情報を暗号化 |
| 11 | バーナーアカウントのみ接続 |
| 13 | `openclaw security audit --deep`で監査実行 |
| 14 | `ss -tlnp | grep 18789`で127.0.0.1バインドを確認 |

---

### Tier 2：標準的な保護（+2時間セットアップ、月30〜45分のメンテ）

**ほとんどのホビーユーザーはここで止まれ。**

- **ツール許可リスト（ allowlist）:** 拒否リストではなく許可リストで制御。`ls`、`cat`、`grep`等のみ明示的に許可する
- **MCPサーバー管理:** バージョン固定（`autoUpdate: false`）、`filesystem`・`shell`・`ssh`は無効化
- **OAuthスコープ最小化:** readonlyで十分な場合はreadonlyに
- **週次セキュリティ監視スクリプト:** 不審なプロンプトインジェクション、危険コマンド実行を自動検出
- **月次メンテナンスルーティン:** ソフトウェア更新、ログレビュー、設定確認

```bash
# 週次チェックのcron設定例
0 9 * * 0 /home/openclaw/check-openclaw-security.sh
```

---

### Tier 3：高度な保護（+2〜3時間、月60分のメンテ）

**Tier 3でもNEVER LISTのアカウントは接続しない。**

主要構成：

**Docker/Podman サンドボックス（Step 20）:**
- OpenClawコンテナ → 外部インターネットへのアクセスなし
- LiteLLMコンテナ → APIキー管理＋レートリミット（OpenClawは本物のAPIキーを見ない）
- Podman（rootless）推奨：コンテナエスケープ時に非特権ユーザーで着地
- Dockerの場合：`userns-remap`または rootless Docker を使用

**その他のTier 3対策:**
- エージェントをリスクプロファイルで分離（ファイル整理エージェント vs 開発エージェント）
- Squidプロキシでネットワーク出口フィルタリング（ドメイン許可リスト方式）
- 四半期ごとのAPI鍵・OAuth・SSH鍵ローテーション
- スキル・MCPサーバーのソースコード手動レビュー
- インシデントレスポンス計画の文書化

**自動デプロイ（Ansible）:**
```bash
git clone https://github.com/Next-Kick/openclaw-hardened-ansible.git
cd openclaw-hardened-ansible
./deploy.sh --target 10.0.5.100 --provider ollama --model llama3
```
手動Tier 1〜3全体を10〜15分で自動化。

---

### 修正不可能な脆弱性

どれだけ強化しても残る根本的リスク：

- **プロンプトインジェクション:** 悪意あるメール・ドキュメント・Webページによる操作。LLMに内在する問題
- **サプライチェーン攻撃:** ClawsHubスキルレジストリにはモデレーションプロセスが存在しない
- ゼロデイ脆弱性

---

## 論評

本記事が際立っているのは、「使うな」という警告を前置きした上で、それでも使う人への実践的支援を惜しまない姿勢にある。セキュリティの文章は往々にして「原則」で終わるが、Fernando Lucktemberg はコピペ可能なコマンド、期待される出力、検証手順まで提供しており、実際に手を動かせるガイドになっている。

3段階の段階的アプローチも合理的だ。Tier 1を「最低限」として絶対的に位置づけ、Tier 2を「ほとんどの人の着地点」とし、Tier 3を特定ニーズのためと明確に区別している。「全部やらなくていい、でもTier 1だけは絶対やれ」という設計は、完璧主義の罠を避けながら実際の行動変容につながりやすい。

AIエージェントのセキュリティ強化事例として、LiteLLMをプロキシに使ってAPIキーを分離するパターンは参考になる。OpenClawに限らず、APIキーを直接保持させたくないエージェント設計全般に応用できる考え方だ。

---

**タグ:** #security #ai-agent #openclaw #devops #linux #docker
