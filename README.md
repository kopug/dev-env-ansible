# WSL Git and SSH Agent Setup

WSL環境でGitとBitwarden SSH Agentを自動セットアップするためのAnsibleプレイブックです。

## 概要

このプロジェクトは以下を自動化します：

- 必要なパッケージのインストール（Git、socat、npiperelayなど）
- Bitwarden SSH Agent bridgeの設定
- GitHubへのSSH接続設定
- Git全体設定

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

## ファイル構成

```
├── inventory.yml       # Ansibleインベントリ
├── playbook.yml        # メインプレイブック
├── vars.yml.example    # 設定例ファイル
├── vars.yml           # 個人設定（.gitignoreで除外）
└── README.md          # このファイル
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

### 複数のSSHキーの使用

特定のリポジトリで異なるキーを使用する場合：

```bash
Host github-work
    HostName github.com
    User git
    Port 22
    IdentitiesOnly yes
    IdentityFile ~/.ssh/work_key
```

## ライセンス

MIT License

## 貢献

バグ報告や機能改善の提案はIssueまたはPull Requestでお願いします。

## 参考

- [Bitwarden SSH Agent](https://bitwarden.com/help/ssh-agent/)
- [npiperelay](https://github.com/jstarks/npiperelay)
- [WSL SSH Agent Bridge](https://docs.microsoft.com/en-us/windows/wsl/tutorials/wsl-git)
