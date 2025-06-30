# @kamui_qai 式 Git Worktree + tmux + Claude Code 並列開発セットアップガイド

## 概要

@kamui_qai さんが提唱する、Git worktree、tmux、Claude Code を組み合わせた MacBook 1 画面での 12 並列開発環境のセットアップ手順を詳しく解説します。

この方法により、**70-85%の開発時間短縮**と**真の並列処理**が実現できます。

## 前提条件

### 必要なツール

1. **macOS** (MacBook)
2. **Homebrew** (パッケージマネージャー)
3. **Git** (バージョン管理)
4. **Claude Code** (Anthropic のコーディングアシスタント)

### インストール確認

```bash
# Homebrew がインストールされているか確認
brew --version

# Git がインストールされているか確認
git --version

# Claude Code がインストールされているか確認
claude --version
```

## ステップ 1: tmux のインストールと設定

### 1.1 tmux インストール

```bash
# tmux をインストール
brew install tmux

# インストール確認
tmux -V
```

### 1.2 tmux 基本設定（オプション）

```bash
# tmux 設定ファイル作成
cat > ~/.tmux.conf << 'EOF'
# マウス操作を有効化
set -g mouse on

# プレフィックスキーをCtrl+aに変更（使いやすくするため）
set -g prefix C-a
unbind C-b

# ペインの境界線を見やすくする
set -g pane-border-style fg=colour238
set -g pane-active-border-style fg=colour51

# ステータスバーの設定
set -g status-bg colour235
set -g status-fg white
set -g status-left-length 30
set -g status-left '#[fg=green]#H#[fg=white]:#[fg=blue]#S '

# 設定の再読み込み（Prefix + r）
bind r source-file ~/.tmux.conf \; display "Reloaded!"
EOF

# 設定を反映
tmux source-file ~/.tmux.conf
```

## ステップ 2: プロジェクトの準備

### 2.1 メインプロジェクトの準備

```bash
# プロジェクトディレクトリに移動（例）
cd ~/projects/my-project

# Gitリポジトリ初期化（まだの場合）
git init
git add .
git commit -m "Initial commit"

# メインブランチの確認
git branch
```

### 2.2 並列作業用ディレクトリ構造の作成

```bash
# プロジェクトの親ディレクトリに移動
cd ~/projects

# 作業ディレクトリ構造
# ~/projects/
#   ├── my-project/           # メインプロジェクト
#   ├── my-project-task1/     # Task 1用 worktree
#   ├── my-project-task2/     # Task 2用 worktree
#   └── my-project-task3/     # Task 3用 worktree
```

## ステップ 3: Git Worktree の作成

### 3.1 複数の Worktree を作成

```bash
# メインプロジェクトに移動
cd ~/projects/my-project

# Task 1用のブランチとworktreeを作成
git worktree add ../my-project-task1 -b feature/task1

# Task 2用のブランチとworktreeを作成
git worktree add ../my-project-task2 -b feature/task2

# Task 3用のブランチとworktreeを作成
git worktree add ../my-project-task3 -b feature/task3

# 12並列の場合は、必要な数だけ作成
git worktree add ../my-project-task4 -b feature/task4
git worktree add ../my-project-task5 -b feature/task5
git worktree add ../my-project-task6 -b feature/task6
git worktree add ../my-project-task7 -b feature/task7
git worktree add ../my-project-task8 -b feature/task8
git worktree add ../my-project-task9 -b feature/task9
git worktree add ../my-project-task10 -b feature/task10
git worktree add ../my-project-task11 -b feature/task11
git worktree add ../my-project-task12 -b feature/task12

# worktree一覧を確認
git worktree list
```

### 3.2 各 Worktree の初期設定

```bash
# 各worktreeで必要な依存関係をインストール
# （Node.js プロジェクトの例）

cd ../my-project-task1
npm install
cd ../my-project-task2
npm install
cd ../my-project-task3
npm install

# 以下、全てのworktreeで同様の作業を実行
```

## ステップ 4: tmux セッションの設定

### 4.1 tmux 自動分割スクリプトの作成

```bash
# 12並列用のtmuxセットアップスクリプトを作成
cat > ~/setup-parallel-claude.sh << 'EOF'
#!/bin/bash

# セッション名
SESSION_NAME="claude-parallel"

# 既存セッションがあれば削除
tmux kill-session -t $SESSION_NAME 2>/dev/null

# 新しいセッションを作成（デタッチ状態で）
tmux new-session -d -s $SESSION_NAME -c ~/projects/my-project

# 4x3のグリッド（12ペイン）を作成
# 最初に3つの縦分割を作成（4列）
tmux split-window -h -t $SESSION_NAME
tmux split-window -h -t $SESSION_NAME:0.0
tmux split-window -h -t $SESSION_NAME:0.2

# 各列を3つに横分割（3行）
tmux split-window -v -t $SESSION_NAME:0.0
tmux split-window -v -t $SESSION_NAME:0.2

tmux split-window -v -t $SESSION_NAME:0.1
tmux split-window -v -t $SESSION_NAME:0.4

tmux split-window -v -t $SESSION_NAME:0.3
tmux split-window -v -t $SESSION_NAME:0.6

tmux split-window -v -t $SESSION_NAME:0.5
tmux split-window -v -t $SESSION_NAME:0.8

# 各ペインにworktreeディレクトリを設定
tmux send-keys -t $SESSION_NAME:0.0 "cd ~/projects/my-project-task1" Enter
tmux send-keys -t $SESSION_NAME:0.1 "cd ~/projects/my-project-task2" Enter
tmux send-keys -t $SESSION_NAME:0.2 "cd ~/projects/my-project-task3" Enter
tmux send-keys -t $SESSION_NAME:0.3 "cd ~/projects/my-project-task4" Enter
tmux send-keys -t $SESSION_NAME:0.4 "cd ~/projects/my-project-task5" Enter
tmux send-keys -t $SESSION_NAME:0.5 "cd ~/projects/my-project-task6" Enter
tmux send-keys -t $SESSION_NAME:0.6 "cd ~/projects/my-project-task7" Enter
tmux send-keys -t $SESSION_NAME:0.7 "cd ~/projects/my-project-task8" Enter
tmux send-keys -t $SESSION_NAME:0.8 "cd ~/projects/my-project-task9" Enter
tmux send-keys -t $SESSION_NAME:0.9 "cd ~/projects/my-project-task10" Enter
tmux send-keys -t $SESSION_NAME:0.10 "cd ~/projects/my-project-task11" Enter
tmux send-keys -t $SESSION_NAME:0.11 "cd ~/projects/my-project-task12" Enter

# 最初のペインを選択
tmux select-pane -t $SESSION_NAME:0.0

# セッションにアタッチ
tmux attach-session -t $SESSION_NAME
EOF

# スクリプトに実行権限を付与
chmod +x ~/setup-parallel-claude.sh
```

### 4.2 tmux セッション起動

```bash
# 並列開発環境を起動
~/setup-parallel-claude.sh
```

## ステップ 5: Claude Code の並列起動

### 5.1 各ペインで Claude Code を起動

tmux セッションが起動したら、各ペインで以下の操作を行います：

```bash
# 各ペインで Claude Code を起動
claude --dangerously-skip-permissions

# または、セーフモードで起動
claude
```

### 5.2 自動起動スクリプトの改良（オプション）

```bash
# Claude Code自動起動版のスクリプト作成
cat > ~/setup-parallel-claude-auto.sh << 'EOF'
#!/bin/bash

SESSION_NAME="claude-parallel"

# 既存セッションがあれば削除
tmux kill-session -t $SESSION_NAME 2>/dev/null

# 新しいセッションを作成
tmux new-session -d -s $SESSION_NAME -c ~/projects/my-project

# 4x3のグリッド作成（前と同じ）
tmux split-window -h -t $SESSION_NAME
tmux split-window -h -t $SESSION_NAME:0.0
tmux split-window -h -t $SESSION_NAME:0.2

tmux split-window -v -t $SESSION_NAME:0.0
tmux split-window -v -t $SESSION_NAME:0.2
tmux split-window -v -t $SESSION_NAME:0.1
tmux split-window -v -t $SESSION_NAME:0.4
tmux split-window -v -t $SESSION_NAME:0.3
tmux split-window -v -t $SESSION_NAME:0.6
tmux split-window -v -t $SESSION_NAME:0.5
tmux split-window -v -t $SESSION_NAME:0.8

# 各ペインでディレクトリ移動とClaude Code起動
tmux send-keys -t $SESSION_NAME:0.0 "cd ~/projects/my-project-task1 && claude" Enter
tmux send-keys -t $SESSION_NAME:0.1 "cd ~/projects/my-project-task2 && claude" Enter
tmux send-keys -t $SESSION_NAME:0.2 "cd ~/projects/my-project-task3 && claude" Enter
tmux send-keys -t $SESSION_NAME:0.3 "cd ~/projects/my-project-task4 && claude" Enter
tmux send-keys -t $SESSION_NAME:0.4 "cd ~/projects/my-project-task5 && claude" Enter
tmux send-keys -t $SESSION_NAME:0.5 "cd ~/projects/my-project-task6 && claude" Enter
tmux send-keys -t $SESSION_NAME:0.6 "cd ~/projects/my-project-task7 && claude" Enter
tmux send-keys -t $SESSION_NAME:0.7 "cd ~/projects/my-project-task8 && claude" Enter
tmux send-keys -t $SESSION_NAME:0.8 "cd ~/projects/my-project-task9 && claude" Enter
tmux send-keys -t $SESSION_NAME:0.9 "cd ~/projects/my-project-task10 && claude" Enter
tmux send-keys -t $SESSION_NAME:0.10 "cd ~/projects/my-project-task11 && claude" Enter
tmux send-keys -t $SESSION_NAME:0.11 "cd ~/projects/my-project-task12 && claude" Enter

# 最初のペインを選択
tmux select-pane -t $SESSION_NAME:0.0

# セッションにアタッチ
tmux attach-session -t $SESSION_NAME
EOF

chmod +x ~/setup-parallel-claude-auto.sh
```

## ステップ 6: 並列開発の実践

### 6.1 タスクの割り当て

各 Claude Code インスタンスに独立したタスクを割り当てます：

1. **Task 1**: ユーザー認証機能の実装
2. **Task 2**: データベース設計の最適化
3. **Task 3**: API エンドポイントの作成
4. **Task 4**: フロントエンド UI コンポーネント
5. **Task 5**: テストコードの作成
6. **Task 6**: ドキュメント作成
7. **Task 7**: バグ修正
8. **Task 8**: パフォーマンス最適化
9. **Task 9**: セキュリティ強化
10. **Task 10**: CI/CD パイプライン
11. **Task 11**: 翻訳・国際化対応
12. **Task 12**: モニタリング・ログ設定

### 6.2 tmux 操作のコツ

```bash
# ペイン間の移動
Ctrl+a + 矢印キー

# ペインの選択
Ctrl+a + q + 番号

# ペインのズーム/ズーム解除
Ctrl+a + z

# セッションのデタッチ（バックグラウンド実行）
Ctrl+a + d

# セッションへの再アタッチ
tmux attach-session -t claude-parallel
```

## ステップ 7: 作業の統合

### 7.1 完了したタスクのマージ

```bash
# メインプロジェクトに戻る
cd ~/projects/my-project

# 各ブランチの作業をメインブランチにマージ
git checkout main
git merge feature/task1
git merge feature/task2
git merge feature/task3
# ... 他のタスクも同様
```

### 7.2 Worktree のクリーンアップ

```bash
# 作業完了後、不要なworktreeを削除
git worktree remove ../my-project-task1
git worktree remove ../my-project-task2
git worktree remove ../my-project-task3
# ... 他のタスクも同様

# ブランチの削除（必要に応じて）
git branch -d feature/task1
git branch -d feature/task2
git branch -d feature/task3
```

## トラブルシューティング

### よくある問題と解決策

1. **tmux ペインが小さすぎる**

   ```bash
   # ターミナルウィンドウのサイズを大きくする
   # または、ペイン数を減らす
   ```

2. **Claude Code の起動が失敗する**

   ```bash
   # 権限の確認
   claude --help

   # 認証の確認
   claude auth status
   ```

3. **Worktree の作成が失敗する**

   ```bash
   # 既存のディレクトリ/ブランチ名の確認
   git worktree list
   git branch -a
   ```

4. **メモリ不足**
   ```bash
   # 並列数を減らす（12→6→3）
   # または、システムリソースの確認
   top
   ```

## 効果測定

### 開発効率の測定方法

1. **Before**: 従来の順次開発での所要時間を記録
2. **After**: 並列開発での所要時間を記録
3. **比較**: 時間短縮率を計算（目標：70-85%短縮）

### 期待される成果

- **開発時間**: 70-85%短縮
- **生産性**: 3-5 倍向上
- **同時作業**: 12 タスクの並列実行
- **環境効率**: MacBook 1 画面での完結

## まとめ

この手順に従うことで、@kamui_qai さんが提唱する Git worktree + tmux + Claude Code による真の並列開発環境を構築できます。初期設定には時間がかかりますが、一度構築すれば劇的な開発効率向上が期待できます。

**重要**: この方法は独立したタスクに最適です。相互依存関係の強いタスクには適さない場合があります。
