# Cross-Platform Development Environment Setup

WSL、macOS、Linux環境でGit、SSH Agent、Zsh、Neovimを自動セットアップするためのAnsibleプレイブックです。

## 新しいアーキテクチャ

```
├── playbook_new.yml           # メインプレイブック
├── inventory_new.yml          # インベントリ
├── group_vars/               # グループ変数
│   ├── all.yml               # 全プラットフォーム共通
│   ├── wsl.yml               # WSL固有
│   └── macos.yml             # macOS固有
├── templates/                # 共通テンプレート
│   ├── ssh_config.j2         # SSH設定
│   ├── wsl.conf.j2           # WSL設定
│   └── zshrc_additions.j2    # Zsh追加設定
├── roles/                    # ロール
│   ├── common/               # 共通設定
│   ├── wsl/                  # WSL固有
│   └── macos/                # macOS固有
└── vars.yml                  # ユーザー固有設定
```

## 使用方法

### 1. 設定ファイルの準備

```bash
cp vars.yml.example vars.yml
nano vars.yml
```

### 2. 実行

```bash
# 新しいプレイブック
ansible-playbook -i inventory_new.yml playbook_new.yml --extra-vars "@vars.yml" --ask-become-pass

# 従来のプレイブック（後方互換性）
ansible-playbook -i inventory.yml playbook.yml --extra-vars "@vars.yml" --ask-become-pass
```

## プラットフォーム対応

- **WSL**: Windows PATH無効化、npiperelay、SSH Agent Bridge
- **macOS**: Homebrew、macOS固有のSSH Agent
- **Linux**: 標準的なLinux環境

## カスタマイズ

各プラットフォームの設定は `group_vars/` で管理されています：

- `group_vars/all.yml`: 全プラットフォーム共通
- `group_vars/wsl.yml`: WSL固有
- `group_vars/macos.yml`: macOS固有

テンプレートファイルで条件分岐により、プラットフォーム固有の設定を適用します。
