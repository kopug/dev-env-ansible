# 🛠️ Development Environment Setup with Ansible

このリポジトリは、Ansible を使用して開発環境を自動セットアップするためのプレイブックです。macOS と WSL (Windows Subsystem for Linux) の両方に対応し、Docker と Volta（Node.js管理）も含む完全な開発環境を構築できます。

## ✨ 特徴

- **🖥️ クロスプラットフォーム対応**: macOS と WSL の両方をサポート
- **🐳 Docker CE 統合**: WSL環境での完全なDocker開発環境
- **⚡ Volta統合**: 高速なNode.jsバージョン管理
- **🔧 systemd サポート**: WSLでのsystemd有効化とサービス管理
- **🔐 GitHub CLI 統合**: WSL環境での認証を自動化
- **⚡ 完全自動化**: 一度のコマンドで開発環境が整う
- **📝 Git設定の完全管理**: エイリアス、設定を含む完全な`.gitconfig`
- **🧩 モジュラー設計**: 機能別に分離されたタスク構造
- **🔒 冪等性保証**: 何度実行しても安全

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
- **Git**: 完全な設定とエイリアス付き
- **Neovim**: エディタとプラグイン環境
- **Zsh + Prezto**: 強化されたシェル環境
- **開発ツール**: 基本的なコマンドラインツール
- **Volta**: 高速なJavaScript toolchain管理（Node.js LTS版、npm、プロジェクト自動切り替え）

### WSL 固有
- **systemd**: サービス管理とDocker統合
- **Docker CE**: コンテナ開発環境（非rootユーザー対応）
- **Docker Compose**: マルチコンテナアプリケーション管理
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
  # - rust
  # - java

# カスタムgitignoreパターン（オプション）
custom_gitignore_patterns:
  - "# Project specific"
  - "*.local" 
  - ".env.local"

# 機能フラグ（オプション - デフォルト値を上書きする場合のみ設定）
features:
  github_cli: false    # GitHub CLI を無効化
  docker: false        # Docker インストールを無効化
  systemd: false       # systemd 設定を無効化
```

## 🎯 セレクティブ実行

特定の機能のみ実行する場合：

```bash
# Git設定のみ
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass --tags git

# Volta設定のみ
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass --tags volta

# Docker設定のみ (WSL)
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass --tags docker

# GitHub CLI設定のみ (WSL)
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass --tags wsl

# Neovim設定のみ
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass --tags neovim

# シェル設定のみ
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass --tags shell
```

## ⚡ Volta の使用方法

### 基本的な使用方法
セットアップ後、Voltaで管理されたNode.jsとnpmが利用できます：

```bash
# バージョン確認
volta --version
node --version   # LTS版がインストール済み
npm --version

# 特定バージョンをインストール
volta install node@18.17.0
volta install node@20.12.0

# インストール済みのツールを確認
volta list
```

### プロジェクトでのバージョン管理
```bash
# プロジェクトディレクトリで特定バージョンを固定
cd your-project
volta pin node@18.17.0
volta pin npm@9.6.7

# package.jsonに設定が自動保存される
cat package.json
# {
#   "volta": {
#     "node": "18.17.0",
#     "npm": "9.6.7"
#   }
# }
```

### 自動バージョン切り替え
プロジェクトディレクトリに移動すると、Voltaが自動的に適切なバージョンに切り替えます：

```bash
cd ~/project-a  # Node.js 18.17.0に自動切り替え
node --version  # v18.17.0

cd ~/project-b  # Node.js 20.12.0に自動切り替え
node --version  # v20.12.0
```

## 🐳 Docker 環境（WSL）

### Docker の使用方法
セットアップ後、非rootユーザーでDockerを使用できます：

```bash
# コンテナ実行テスト
docker run hello-world

# 実行中のコンテナ確認
docker ps

# Docker Compose使用
docker compose version

# サービス状態確認
sudo systemctl status docker
```

### systemd 統合
- Docker は systemd により自動起動
- WSL 起動時に自動的に利用可能
- `systemctl` でサービス管理可能

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

### Volta関連の問題

#### Voltaが認識されない場合
```bash
# シェルを再起動
exec $SHELL

# または、手動でPATHを設定
export VOLTA_HOME="$HOME/.volta"
export PATH="$VOLTA_HOME/bin:$PATH"

# 設定の永続化確認
echo $PATH | grep volta
```

#### Node.js/npmのバージョン確認
```bash
# Voltaで管理されているツールを確認
volta list

# 現在のバージョン確認
volta --version
node --version
npm --version

# どのバイナリが使用されているか確認
which node
which npm
```

#### npmが見つからない場合
Node.jsにはnpmが同梱されているため、通常は追加インストール不要です：

```bash
# Node.jsと同梱のnpmを使用
node --version  # これが表示されればnpmも利用可能
npm --version   # npmバージョン確認

# 必要に応じてVoltaでnpmを明示的にインストール
volta install npm
```

#### プロジェクトでVoltaが効かない場合
```bash
# package.jsonのvolta設定を確認
cat package.json | grep -A5 "volta"

# 設定されていない場合は手動で追加
volta pin node@18.17.0

# 強制的にバージョンを確認
volta run node --version
```

### WSL環境での問題

#### systemd が無効の場合
```bash
# systemd状態確認
systemctl --version

# features.systemd: true に設定してプレイブック実行
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass --tags wsl

# WSL再起動が必要
# PowerShell/Command Prompt で: wsl --shutdown
```

#### Docker 関連のエラー
```bash
# Docker グループ確認
groups | grep docker

# Docker サービス確認
sudo systemctl status docker

# 権限エラーの場合（ログアウト・ログインが必要）
newgrp docker
```

#### GitHub CLI認証エラー
```bash
# ブラウザが開かない場合
sudo apt install wslu
export BROWSER="wslview"

# 手動認証の場合
# → https://github.com/login/device にアクセス
# → 表示されたワンタイムコードを入力
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

## 📁 プロジェクト構造

```
dev-env-ansible/
├── README.md                    # このファイル
├── playbook.yml                 # メインプレイブック
├── inventory.yml                # インベントリ設定
├── vars.yml.example            # 設定例

├── group_vars/                  # グループ変数
│   ├── all.yml                 # 全環境共通設定
│   ├── wsl.yml                 # WSL専用設定
│   └── darwin.yml              # macOS専用設定

└── roles/                       # 機能別ロール
    ├── common/                 # 共通機能
    │   ├── tasks/              # タスク定義
    │   │   ├── main.yml        # メインタスク
    │   │   ├── setup_*.yml     # 機能別設定タスク
    │   │   └── install_*.yml   # インストールタスク
    │   └── templates/          # 設定テンプレート
    ├── wsl/                    # WSL専用機能
    ├── darwin/                 # macOS専用機能
    └── docker/                 # Docker CE インストール
        ├── tasks/
        └── handlers/
```

## 🔧 システム要件

### WSL (推奨)
- Windows 10/11 with WSL2
- Ubuntu 20.04+ または互換ディストリビューション
- systemd サポート
- 最低 4GB RAM (Docker使用時は 8GB 推奨)

### macOS
- macOS 10.15 (Catalina) 以上
- Xcode Command Line Tools
- Homebrew (自動インストール)

## 🔄 アップデート

```bash
# リポジトリ更新
git pull origin main

# 新機能を適用
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass
```

## 📝 ライセンス

MIT License

---

**注意**: このセットアップには管理者権限が必要です。信頼できる環境でのみ実行してください。
