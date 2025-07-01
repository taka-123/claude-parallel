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

### 3. tmux 設定（初回のみ）

```bash
# claude-parallelディレクトリに移動
cd claude-parallel

# tmux設定をコピー
cp .tmux.conf.example ~/.tmux.conf

# 設定を反映
tmux source-file ~/.tmux.conf
```

**注意**: 既存の `~/.tmux.conf` がある場合は上書きされます。

### 4. 並列開発環境の起動

```bash
# プロジェクトディレクトリに戻る
cd ..

# セットアップスクリプトを実行
./claude-parallel/setup-claude-parallel    # デフォルト: 6並列
./claude-parallel/setup-claude-parallel 4  # 4並列で起動
```

スクリプト実行後、自動的に tmux セッションが開始されます。

## 📖 詳細ガイド

### コマンドオプション

```bash
./claude-parallel/setup-claude-parallel [並列数] [プロジェクト名]
```

- **並列数**（省略可、デフォルト: 6）: 作成する worktree とペインの数
- **プロジェクト名**（省略可、デフォルト: 現在のディレクトリ名）: tmux セッション名に使用

### tmux 基本操作

| 操作           | キーバインド       | 説明                    | 設定       |
| -------------- | ------------------ | ----------------------- | ---------- |
| ペイン移動     | `Ctrl+g + h/j/k/l` | vim 風のペイン移動      | カスタム   |
| ペインズーム   | `Ctrl+g + z`       | ペインを全画面/元に戻す | デフォルト |
| 横分割         | `Ctrl+g + \`       | ペインを左右に分割      | カスタム   |
| 縦分割         | `Ctrl+g + -`       | ペインを上下に分割      | カスタム   |
| ペイン削除     | `Ctrl+g + x`       | 現在のペインを削除      | デフォルト |
| タイル整列     | `Ctrl+g + =`       | 全ペインをタイル状整列  | カスタム   |
| セッション終了 | `Ctrl+g + d`       | バックグラウンドに移行  | デフォルト |

**注意**: プレフィックスキー自体が `Ctrl+b`（デフォルト）から `Ctrl+g`（カスタム）に変更されています。

### 使い方

#### Step 1: 各ペインで Claude Code 起動

各ペインで以下を実行：

```bash
claude

# または、権限エラーが出る場合
claude --dangerously-skip-permissions
```

#### Step 3: タスクの割り当て

各 Claude Code インスタンスに独立したタスクを指示：

**例 1：Web アプリケーション開発**

- Task1: 「ユーザー認証 API を実装して」
- Task2: 「商品管理画面の UI コンポーネントを作成して」
- Task3: 「データベースのマイグレーションを作成して」
- Task4: 「E2E テストを追加して」
- Task5: 「API ドキュメントを更新して」
- Task6: 「パフォーマンス改善を行って」

**例 2：GitHub Issues ベース**

```bash
# まず Issues を確認
gh issue list
```

- Task1: 「Issue #15 のユーザー認証改善を実装して」
- Task2: 「Issue #16 の API レスポンス最適化を行って」
- Task3: 「Issue #17 の UI コンポーネントを追加して」
- Task4: 「Issue #18 のバグを修正して」

#### Step 4: 作業完了後の統合

```bash
# 各ブランチをマージ
git checkout main
git merge feature/task1
git merge feature/task2
# ...
```

#### Step 5: クリーンアップ

```bash
# 作業完了後、worktreeを削除
git worktree remove ../[プロジェクト名]-task1
git worktree remove ../[プロジェクト名]-task2
# ...
```

## ⚙️ カスタマイズ

### 並列数の調整

スクリプトは以下の並列数に最適化されています：

- 2, 3, 4, 6, 9, 12

```bash
./claude-parallel/setup-claude-parallel 6    # 6並列（推奨）
./claude-parallel/setup-claude-parallel 12   # 12並列（上級者向け）
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
./claude-parallel/setup-claude-parallel 6  # 12→6に減らす
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
./claude-parallel/setup-claude-parallel 3

# システムリソースを確認
top
```

### セッション管理

```bash
# セッション一覧表示
tmux list-sessions

# セッションにアタッチ（セッション名は実際のプロジェクト名に応じて変わります）
tmux attach-session -t claude-my-project

# セッションにアタッチ
tmux attach-session -t claude-[プロジェクト名]

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
