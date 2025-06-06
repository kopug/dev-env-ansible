# Development Environment Setup with Ansible

自動化された開発環境セットアップツール。WSL、Linux、macOSに対応したAnsibleプレイブックです。

## 🚀 クイックスタート

```bash
# リポジトリをクローン
git clone https://github.com/kopug/dev-env-ansible.git
cd dev-env-ansible

# 設定ファイルをコピー
cp vars.yml.example vars.yml

# vars.ymlを編集（GitHubユーザー名、メールアドレスなど）
vim vars.yml

# 開発環境をセットアップ
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass
```

## 📋 機能

### 共通機能 (All Platforms)
- **Git設定**: ユーザー情報、エイリアス、グローバルgitignore
- **SSH設定**: GitHub用SSH設定
- **Neovim**: 最新版のインストールと設定
- **Zsh + Prezto**: モダンなシェル環境
- **開発パッケージ**: curl, wget, unzip, git, vim等

### WSL固有機能
- **WSL設定**: systemd無効化、ネットワーク設定
- **SSH Agent Bridge**: npiperelayによるWindows連携
- **パッケージ**: socat等のWSL必須パッケージ

### macOS固有機能
- **Homebrew**: パッケージマネージャー
- **macOS最適化**: Homebrew経由でのツールインストール

## 🔧 部分実行

特定の機能のみを実行したい場合は、タグを使用できます：

```bash
# Git設定のみ
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass --tags git

# SSH設定のみ
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass --tags ssh

# Neovim関連のみ
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass --tags neovim

# WSL設定のみ
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass --tags wsl

# 利用可能なタグ一覧を確認
ansible-playbook -i inventory.yml playbook.yml --list-tags
```

## 📁 プロジェクト構造

```
dev-env-ansible/
├── group_vars/          # グループ変数
│   ├── all.yml         # 全プラットフォーム共通設定
│   ├── wsl.yml         # WSL固有設定
│   └── macos.yml       # macOS固有設定
├── roles/              # Ansibleロール
│   ├── common/         # 共通機能
│   │   ├── tasks/
│   │   │   ├── main.yml
│   │   │   ├── setup_shell.yml
│   │   │   ├── install_neovim_linux.yml
│   │   │   └── install_neovim_darwin.yml
│   │   └── templates/
│   │       ├── ssh_config.j2
│   │       ├── gitignore.j2
│   │       └── zshrc_additions.j2
│   ├── wsl/            # WSL固有機能
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   └── templates/
│   │       └── wsl.conf.j2
│   └── macos/          # macOS固有機能
│       ├── tasks/
│       │   └── main.yml
│       └── templates/
├── playbook.yml        # メインプレイブック
├── inventory.yml       # インベントリファイル
├── vars.yml           # ユーザー設定変数
└── vars.yml.example   # 設定例
```

## ⚙️ 設定

### vars.yml の設定例

```yaml
# Git設定
git_user_name: "Your Name"
git_user_email: "your-email@example.com"

# GitHub設定
github_username: "your-github-username"

# リポジトリ設定
neovim_config_repo: "https://github.com/your-username/neovim.git"

# 開発環境（gitignoreに影響）
development_environments:
  - node
  - python
  - go
  # - rust
  # - java

# WSL設定
wsl_systemd: false
npiperelay_url: "https://github.com/jstarks/npiperelay/releases/latest/download/npiperelay_windows_amd64.zip"
```

## 🔍 トラブルシューティング

### 構文チェック
```bash
ansible-playbook --syntax-check -i inventory.yml playbook.yml
```

### ドライラン（変更せずに確認）
```bash
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --check --diff --ask-become-pass
```

### 詳細ログで実行
```bash
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass -v
```

### WSL環境の確認
```bash
# WSL検出確認
cat /proc/version | grep -i microsoft && echo "This is WSL" || echo "This is not WSL"

# SSH Agent確認
ssh-add -l

# GitHub接続テスト
ssh -T git@github.com
```

## 📋 セットアップ後の確認

```bash
# 新しいシェルセッション開始
zsh

# 環境変数確認
echo $SHELL    # /usr/bin/zsh
echo $EDITOR   # nvim

# Git設定確認
git config --list --global

# Neovim確認
nvim --version

# SSH設定確認
cat ~/.ssh/config
```

## 🔄 新しいマシンでの使用

1. このリポジトリをクローン
2. `vars.yml.example`を`vars.yml`にコピーして編集
3. `ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass`を実行

数分で同じ開発環境が構築されます！

## 🤝 カスタマイズ

### 新しい開発言語の追加
`group_vars/all.yml`の`development_environments`に言語を追加し、`roles/common/templates/gitignore.j2`にパターンを追加してください。

### プラットフォーム固有設定の追加
各プラットフォーム用のロール（`roles/wsl/`, `roles/macos/`）にタスクを追加してください。

### パッケージの追加
`group_vars/`の各ファイルでパッケージリストを編集してください。

## 📝 ライセンス

MIT License

## 🙋‍♂️ サポート

質問や問題がある場合は、GitHubのIssuesでお知らせください。
