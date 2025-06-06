# WSL Git, SSH Agent, Zsh and Neovim Setup

WSL環境でGit、Bitwarden SSH Agent、Zsh（Prezto）、Neovimを自動セットアップするためのAnsibleプレイブックです。

## 概要

このプロジェクトは以下を自動化します：

- 必要なパッケージのインストール（Git、socat、npirelay、zshなど）
- Neovimの最新版インストール
- Zshのインストールとデフォルトシェル変更
- Preztoフレームワークの設定
- Bitwarden SSH Agent bridgeの設定
- GitHubへのSSH接続設定
- Git全体設定
- Neovim設定の自動クローン・適用

## 前提条件

- WSL2環境（Ubuntu推奨）
- Ansibleがインストール済み
- Windows側でBitwarden SSH Agentが動作している
- sudoアクセス権限

## インストール

### 0. Ansibleのインストール

```bash
# システムパッケージの更新
sudo apt update

# Ansibleのインストール
sudo apt install -y ansible

# バージョン確認
ansible --version
```

### 1. リポジトリのクローン

```bash
git clone https://github.com/kopug/wsl-git-ssh-setup.git
cd wsl-git-ssh-setup
```

### 2. 設定ファイルの準備

```bash
# vars.yml.exampleをコピーして設定
cp vars.yml.example vars.yml

# 自分の情報に編集
nano vars.yml
```

**vars.yml例:**
```yaml
git_user_name: "Your Name"
git_user_email: "your-email@example.com"
```

### 3. Ansibleの実行

```bash
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass
```

## 検証手順

### 1. WSL再起動
```bash
exit
# Windows PowerShellで
wsl --shutdown
wsl
```

### 2. SSH Agent確認
```bash
ssh-add -l
# 期待する出力: 4096 SHA256:... github - kopug (RSA)
```

### 3. GitHub接続テスト
```bash
ssh -T git@github.com
# 期待する出力: Hi username! You've successfully authenticated...
```

### 4. Git操作テスト
```bash
# テストリポジトリのクローン
git clone git@github.com:yourusername/testrepo.git
```

### 5. Zsh & Prezto動作確認
```bash
# デフォルトシェルがzshに変更されていることを確認
echo $SHELL
# 期待する出力: /bin/zsh

# Preztoが動作していることを確認
# プロンプトがカラフルに表示され、補完機能が有効になっている
```

### 6. Neovim動作確認
```bash
# Neovimを起動
nvim

# 初回起動時にLazy.nvimが自動でプラグインをインストール
# :checkhealth でNeovimの設定状況を確認可能
```

## 重要な注意点

### Windows PATHの無効化について

このプレイブックは、zshの補完速度を大幅に改善するため、`/etc/wsl.conf`に以下の設定を追加します：

```ini
[interop]
appendWindowsPath = false
```

**これにより**：
- ✅ zshの補完が高速化される
- ✅ Preztoの動作が軽快になる
- ❌ WSLからWindows側のコマンド（`code.exe`, `explorer.exe`など）が直接実行できなくなる

**Windows側のコマンドを使用したい場合**：
```bash
# フルパスで実行
/mnt/c/Windows/System32/notepad.exe

# またはエイリアスを設定
alias notepad='/mnt/c/Windows/System32/notepad.exe'
alias code='/mnt/c/Users/[username]/AppData/Local/Programs/Microsoft\ VS\ Code/Code.exe'
```

この設定を無効にしたい場合は、`/etc/wsl.conf`を削除またはコメントアウトしてWSLを再起動してください。

## セットアップされる内容

### パッケージ
- Git
- Neovim（最新版）
- Zsh + Prezto フレームワーク
- socat, npiperelay（SSH Agent Bridge用）
- 開発に必要な基本ツール

### シェル設定
- デフォルトシェルをzshに変更
- Preztoフレームワークの自動設定
- 高機能な補完とプロンプト
- Git統合とシンタックスハイライト

### WSL設定
- `/etc/wsl.conf`でWindows PATHを無効化
- zsh補完の高速化（Windowsパス検索を回避）
- WSL固有の最適化設定

### SSH設定
- Bitwarden SSH Agent bridgeの自動設定
- GitHub用SSH設定
- WSL再起動時の自動再接続

### Git設定
- ユーザー名・メールアドレス
- カラー設定、エディタ設定
- 豊富なエイリアス設定
- グローバル.gitignore

### Neovim設定
- [kopug/neovim](https://github.com/kopug/neovim)からの設定自動取得
- Lazy.nvimプラグインマネージャー
- Treesitterによるシンタックスハイライト
- 開発効率向上のための設定

## ファイル構成

```
├── inventory.yml       # Ansibleインベントリ
├── playbook.yml        # メインプレイブック
├── gitconfig.template  # Git設定テンプレート
├── vars.yml.example    # 設定例ファイル
├── vars.yml           # 個人設定（.gitignoreで除外）
└── README.md          # このファイル
```

## 設定される機能

### Zsh & Prezto機能
- **高機能補完**: コマンド、ファイル、Git補完
- **美しいプロンプト**: Git状態の表示
- **シンタックスハイライト**: コマンドライン上でのリアルタイム色付け
- **履歴検索**: 高度なコマンド履歴機能
- **エイリアス**: 便利なコマンドエイリアス
- **モジュール系**: 必要に応じてモジュールを追加可能

### Neovim機能
- **Lazy.nvim**: 高性能プラグインマネージャー
- **Treesitter**: 高度なシンタックスハイライト
- **基本設定**: 行番号、タブ設定、検索設定など
- **クリップボード連携**: WSLとWindowsのクリップボード共有
- **エディタエイリアス**: `vi`, `vim`コマンドで`nvim`を起動

### Git エイリアス例
```bash
git st          # status -s（短縮ステータス）
git co          # checkout
git cb          # checkout -b（新ブランチ作成）
git ci          # commit
git up          # pull --rebase
git log-graph   # グラフ付きログ表示
```

## トラブルシューティング

### SSH Agentが動作しない

```bash
# プロセス確認
ps aux | grep socat
ps aux | grep npiperelay

# 手動再起動
pkill socat 2>/dev/null
pkill npiperelay.exe 2>/dev/null
export SSH_AUTH_SOCK=$HOME/.ssh/agent.sock
rm -f $SSH_AUTH_SOCK
(setsid socat UNIX-LISTEN:$SSH_AUTH_SOCK,fork EXEC:"/usr/local/bin/npiperelay.exe -ei -s //./pipe/openssh-ssh-agent",nofork &) >/dev/null 2>&1
```

### Neovimプラグインエラー

```bash
# Neovim内で以下を実行
:Lazy sync          # プラグインの同期
:checkhealth        # 健康状態チェック
:TSUpdate           # Treesitterの更新
```

### GitHub接続エラー

1. **Bitwarden SSH Agentが動作しているか確認**
   ```powershell
   # Windows PowerShellで
   ssh-add -l
   ```

2. **GitHubにSSHキーが登録されているか確認**
   - GitHub Settings → SSH and GPG keys
   - Bitwardenの公開鍵が登録されていることを確認

3. **SSH設定確認**
   ```bash
   cat ~/.ssh/config
   # IdentitiesOnly no が設定されていることを確認
   ```

## カスタマイズ

### Neovim設定の更新

設定を更新したい場合：

```bash
cd ~/.config/nvim
git pull origin main
nvim
# :Lazy sync でプラグインを更新
```

### zsh補完が遅い場合

通常は自動で設定されますが、もし補完が遅い場合：

```bash
# /etc/wsl.confの内容を確認
cat /etc/wsl.conf

# 以下が設定されていることを確認
# [interop]
# appendWindowsPath = false

# 設定後は必ずWSLを完全に再起動
exit
# PowerShellで
wsl --shutdown
wsl
```

### Preztoのカスタマイズ

Preztoの設定は`~/.zpreztorc`で行います：

```bash
# 使用するモジュールを設定
zstyle ':prezto:load' pmodule \
  'environment' \
  'terminal' \
  'editor' \
  'history' \
  'directory' \
  'spectrum' \
  'utility' \
  'completion' \
  'prompt' \
  'syntax-highlighting' \
  'history-substring-search' \
  'git'

# プロンプトテーマを変更
zstyle ':prezto:module:prompt' theme 'sorin'
```

利用可能なテーマ一覧：
```bash
prompt -l
```

### 独自のNeovim設定を使用

`playbook.yml`の以下の部分を編集：

```yaml
- name: Clone Neovim configuration repository
  git:
    repo: https://github.com/yourusername/your-neovim-config.git  # <- 変更
    dest: "{{ target_home }}/.config/nvim"
    force: yes
```

### 異なるGitサービスの追加

`.ssh/config`に他のGitサービスの設定を追加：

```bash
Host gitlab.com
    HostName gitlab.com
    User git
    Port 22
    IdentitiesOnly no
    PreferredAuthentications publickey
```

## ライセンス

MIT License

## 貢献

バグ報告や機能改善の提案はIssueまたはPull Requestでお願いします。

## 参考

- [Bitwarden SSH Agent](https://bitwarden.com/help/ssh-agent/)
- [npiperelay](https://github.com/jstarks/npiperelay)
- [WSL SSH Agent Bridge](https://docs.microsoft.com/en-us/windows/wsl/tutorials/wsl-git)
- [Neovim](https://neovim.io/)
- [Lazy.nvim](https://github.com/folke/lazy.nvim)
- [Prezto](https://github.com/sorin-ionescu/prezto)
- [Zsh](https://www.zsh.org/)
