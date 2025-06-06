# 🛠️ Development Environment Setup with Ansible

このリポジトリは、Ansible を使用して開発環境を自動セットアップするためのプレイブックです。macOS と WSL (Windows Subsystem for Linux) の両方に対応しています。

## ✨ 特徴

- **クロスプラットフォーム対応**: macOS と WSL の両方をサポート
- **GitHub CLI 統合**: WSL環境での認証を自動化
- **完全自動化**: 一度のコマンドで開発環境が整う
- **Git設定の完全管理**: エイリアス、設定を含む完全な`.gitconfig`
- **モジュラー設計**: 必要な機能だけを選択して実行可能
- **冪等性保証**: 何度実行しても安全

## 🚀 クイックスタート

```bash
# リポジトリをクローン
git clone https://github.com/kopug/dev-env-ansible.git
cd dev-env-ansible

# 設定ファイルを作成
cp vars.yml.example vars.yml
vim vars.yml  # 個人情報を設定

# セットアップ実行
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass
```

## 🔧 事前準備

### Ansibleのインストール

#### Ubuntu/WSL
```bash
sudo apt update
sudo apt install ansible
```

#### macOS
```bash
# Homebrewがない場合
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Ansibleインストール
brew install ansible
```

## 📦 インストールされるもの

### 共通 (macOS/WSL)
- **Git**: 完全な設定とエイリアス
- **Neovim**: エディタとプラグイン環境
- **Zsh**: シェル環境の改善
- **開発ツール**: 基本的なコマンドラインツール

### WSL 固有
- **GitHub CLI**: ブラウザ認証統合
- **WSL Utilities**: Windows連携ツール
- **npiperelay**: SSH Agent連携

### macOS 固有
- **Homebrew**: パッケージマネージャー
- **macOS専用ツール**: システム最適化

## 🔧 カスタマイズ

### 設定ファイルの編集
```bash
# サンプルから個人設定を作成
cp vars.yml.example vars.yml

# 個人情報を設定
vim vars.yml
```

### vars.yml の設定項目
```yaml
# Git設定（必須）
git_config:
  user:
    name: "Your Name"
    email: "your-email@example.com"

# Neovim設定リポジトリ（オプション）
neovim_config_repo: "https://github.com/your-username/neovim.git"

# 開発環境（gitignoreに影響）
development_environments:
  - node
  - python
  - go
```

## 🎯 セレクティブ実行

特定の機能のみ実行する場合：

```bash
# Git設定のみ
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass --tags git

# GitHub CLI設定のみ (WSL)
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass --tags github-cli

# Neovim設定のみ
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass --tags neovim

# シェル設定のみ
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass --tags shell
```

## 🔐 認証設定

### GitHub CLI (WSL)
セットアップ後、GitHub CLI で認証を行います：

```bash
# 認証開始
gh auth login

# 設定選択
# → GitHub.com
# → HTTPS  
# → Yes (authenticate Git)
# → Login with a web browser

# 認証確認
gh auth status
```

これで今後のgit操作（clone、push、pull）が認証なしで行えます。

### Git 設定の確認
```bash
# Git設定確認
git config --list --global

# SSH設定確認（SSH使用の場合）
ssh -T git@github.com
```

## 🚨 トラブルシューティング

### WSL環境での問題

#### GitHub CLI認証エラー
```bash
# ブラウザが開かない場合
sudo apt install wslu
export BROWSER="wslview"

# 手動認証の場合
# → https://github.com/login/device にアクセス
# → 表示されたワンタイムコードを入力
```

#### SSH接続の問題（SSH使用時）
```bash
# SSH Agent確認
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# 接続テスト
ssh -T git@github.com
```

### 共通の問題

#### Ansibleの依存関係エラー
```bash
# Ansible再インストール
sudo apt update
sudo apt install ansible

# または macOS
brew install ansible
```

#### 権限エラー
```bash
# sudoパスワードを確認して再実行
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass
```

#### Git設定の問題
```bash
# 設定を手動確認
git config --global --list

# 設定をリセット（必要な場合）
git config --global --unset-all user.name
git config --global --unset-all user.email
```

## 📁 プロジェクト構造

```
dev-env-ansible/
├── README.md                    # このファイル
├── playbook.yml                 # メインプレイブック
├── inventory.yml                # インベントリ設定
├── vars.yml.example            # 設定例
├── group_vars/                  # グループ変数
│   ├── all.yml                 # 全環境共通
│   ├── wsl.yml                 # WSL専用
│   └── macos.yml               # macOS専用
└── roles/                       # 機能別ロール
    ├── common/                 # 共通機能
    │   ├── tasks/              # タスク定義
    │   └── templates/          # 設定テンプレート
    ├── wsl/                    # WSL専用機能
    └── macos/                  # macOS専用機能
```

## 📝 ライセンス

MIT License
