# Claude Code 並列開発環境

Git worktree + tmux + Claude Code を組み合わせた真の並列開発環境です。MacBook 1 画面で最大 12 並列の開発作業が可能になり、**70-85%の開発時間短縮**を実現できます。

## 📁 ファイル構成

```
claude-parallel/
├── setup-claude-parallel     # 並列開発環境セットアップスクリプト
├── .tmux.conf.example        # 最適化されたtmux設定ファイル
└── README.md                 # このドキュメント（プロジェクトにはコピー不要）
```

## 🚀 クイックスタート

### 1. 前提条件

以下のツールがインストールされている必要があります：

```bash
# 必要なツールの確認
brew --version    # Homebrew
git --version     # Git
tmux -V          # tmux
claude --version # Claude Code
```

### 2. インストール

```bash
# あなたのプロジェクトディレクトリに移動
cd /your/project

# このリポジトリをクローン
git clone https://github.com/taka-123/claude-parallel.git claude-parallel

# セットアップ後のプロジェクト構造
# your-project/
# ├── claude-parallel/            # 並列開発環境ツール
# │   ├── setup-claude-parallel   # セットアップスクリプト
# │   ├── .tmux.conf.example     # tmux設定サンプル
# │   └── README.md              # ツールのドキュメント
# ├── src/                       # あなたのプロジェクト
# ├── package.json
# └── README.md                  # あなたのプロジェクトのREADME
```

**オプション**: 個人用にカスタマイズしたい場合はフォークして使用することも可能です。

### 3. tmux 設定

```bash
# 初回のみ：tmux設定をコピー（既存設定がある場合はバックアップ推奨）
cp ~/.tmux.conf ~/.tmux.conf.backup 2>/dev/null || true
cp .tmux.conf.example ~/.tmux.conf

# 設定を反映
tmux source-file ~/.tmux.conf
```

**注意**: 既存の `~/.tmux.conf` がある場合は、上書きされます。重要な設定がある場合は事前にバックアップしてください。

### 4. 並列開発環境の起動

```bash
# プロジェクトディレクトリで実行
cd /your/project
./claude-parallel/setup-claude-parallel

# カスタマイズ例
./claude-parallel/setup-claude-parallel my-app 6    # プロジェクト名とタスク数を指定
./claude-parallel/setup-claude-parallel blog 4     # 4並列で起動
```

## 📖 詳細ガイド

### tmux 操作方法

| 操作           | キーバインド        | 説明                    |
| -------------- | ------------------- | ----------------------- |
| ペイン移動     | `Ctrl+g + h/j/k/l`  | vim 風のペイン移動      |
| ペイン選択     | `Ctrl+g + q + 数字` | 番号でペイン選択        |
| ペインズーム   | `Ctrl+g + z`        | ペインを全画面/元に戻す |
| 横分割         | `Ctrl+g + \`        | ペインを左右に分割      |
| 縦分割         | `Ctrl+g + -`        | ペインを上下に分割      |
| 設定再読込     | `Ctrl+g + r`        | tmux 設定をリロード     |
| セッション終了 | `Ctrl+g + d`        | バックグラウンドに移行  |

### 並列開発の実践

#### Step 1: 環境起動

```bash
./claude-parallel/setup-claude-parallel my-project 6
```

#### Step 2: 各ペインで Claude Code 起動

各ペインで以下を実行：

```bash
claude

# または、権限エラーが出る場合
claude --dangerously-skip-permissions
```

#### Step 3: タスクの割り当て

各 Claude Code インスタンスに独立したタスクを指示：

**例：Web アプリケーション開発**

- Task1: 「ユーザー認証 API を実装して」
- Task2: 「商品管理画面の UI コンポーネントを作成して」
- Task3: 「データベースのマイグレーションを作成して」
- Task4: 「E2E テストを追加して」
- Task5: 「API ドキュメントを更新して」
- Task6: 「パフォーマンス改善を行って」

#### Step 4: 作業完了後の統合

```bash
# メインブランチに戻る
cd ~/projects/my-project

# 各ブランチをマージ
git checkout main
git merge feature/task1
git merge feature/task2
# ...

# または Pull Request を作成
gh pr create --base main --head feature/task1 --title "ユーザー認証API実装"
```

#### Step 5: クリーンアップ

```bash
# 作業完了後、worktreeを削除
git worktree remove ../my-project-task1
git worktree remove ../my-project-task2
# ...

# ブランチ削除（必要に応じて）
git branch -d feature/task1
git branch -d feature/task2
```

## 💡 使用パターン

### パターン 1: GitHub Issues ベース

```bash
# GitHubのIssuesを確認
gh issue list

# 各worktreeで異なるIssueに取り組む
# Task1: Issue #15 - ユーザー認証改善
# Task2: Issue #16 - APIレスポンス最適化
# Task3: Issue #17 - UIコンポーネント追加
```

### パターン 2: 機能別分担

```bash
# 大きな機能を細分化して並列実行
# Task1: Backend API
# Task2: Frontend UI
# Task3: Database Migration
# Task4: Testing
# Task5: Documentation
```

### パターン 3: リファクタリング・最適化

```bash
# コードベース全体の改善を並列実行
# Task1: コンポーネントA のリファクタリング
# Task2: コンポーネントB のリファクタリング
# Task3: パフォーマンス改善
# Task4: セキュリティ強化
```

## ⚙️ カスタマイズ

### 並列数の調整

スクリプトは以下の並列数に最適化されています：

- 2, 3, 4, 6, 9, 12

```bash
./claude-parallel/setup-claude-parallel my-project 6   # 6並列（推奨）
./claude-parallel/setup-claude-parallel my-project 12  # 12並列（上級者向け）
```

### tmux 設定のカスタマイズ

`.tmux.conf.example` を参考に、お好みの設定に調整してください：

```bash
# プレフィックスキーを変更（例：Ctrl+a）
set -g prefix C-a

# 分割キーを変更
bind | split-window -h  # | キーで横分割
bind - split-window -v  # - キーで縦分割
```

## 🔧 トラブルシューティング

### よくある問題

**Q: tmux ペインが小さすぎる**

```bash
# ターミナルウィンドウのサイズを大きくする
# または、並列数を減らす
./claude-parallel/setup-claude-parallel my-project 6  # 12→6に減らす
```

**Q: Claude Code の起動が失敗する**

```bash
# 権限とインストールの確認
claude --help
claude auth status

# 必要に応じて再インストール
```

**Q: worktree の作成が失敗する**

```bash
# 既存のworktreeとブランチを確認
git worktree list
git branch -a

# 重複がある場合は削除
git worktree remove ../path-to-worktree
git branch -D branch-name
```

**Q: メモリ不足**

```bash
# 並列数を減らす
./claude-parallel/setup-claude-parallel my-project 3

# システムリソースを確認
top
```

### セッション管理

```bash
# セッション一覧表示
tmux list-sessions

# セッションにアタッチ（セッション名は実際のプロジェクト名に応じて変わります）
tmux attach-session -t claude-my-project

# バックグラウンドで実行中のセッションに戻る
tmux attach-session -t claude-parallel

# セッション強制終了
tmux kill-session -t claude-my-project

# 全てのセッションを終了
tmux kill-server
```

## 📊 効果測定

### 期待される成果

- **開発時間**: 70-85%短縮
- **生産性**: 3-5 倍向上
- **同時作業**: 最大 12 タスクの並列実行
- **環境効率**: MacBook 1 画面での完結

### 測定方法

1. **Before**: 従来の順次開発での所要時間を記録
2. **After**: 並列開発での所要時間を記録
3. **比較**: 時間短縮率を計算

## 🤝 コントリビューション

改善提案やバグ報告は Issue または Pull Request でお願いします。

## 📄 ライセンス

MIT License

## ⚠️ 注意事項

- この手法は**独立したタスク**に最適です。相互依存関係の強いタスクには適さない場合があります
- 大量の Claude Code インスタンスを同時実行するため、API 使用量にご注意ください
- 初回セットアップには時間がかかりますが、一度構築すれば劇的な効率向上が期待できます

## 🙏 謝辞

この並列開発手法は @kamui_qai さんのアイデアを基に作成しました。
