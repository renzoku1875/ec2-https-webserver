# EC2 + HTTPS サーバ構築ポートフォリオ

## 🔧 Webサイト作成・管理（HTTPS対応）

LPICを勉強して、実際にAWSのサーバ（EC2）でWebサイトを作ってみた記録です。
できるだけ実務に近い構成を自分で調べながら作りました。

---

### 🌐 サイトURL

* [https://mytest-portfolio.xyz](https://mytest-portfolio.xyz)
* [https://mytest-portfolio.xyz/secret](https://mytest-portfolio.xyz/secret)  - (Basic認証あり）
* [https://mytest-portfolio.xyz/report.html](https://mytest-portfolio.xyz/report.html) - サンプルレポートページ
  
---

## 🗂 サーバ構成図

以下は、Amazon EC2 上に構築した Web サーバの全体構成図です。

![EC2構成図](./images/ec2-architecture.png)

---

### 🔐 セキュリティの工夫

* HTTPアクセスはすべてHTTPSへ自動リダイレクト
* `.htpasswd` を使って特定ページにログイン制限を設定
* fail2banでSSH攻撃対策（5回失敗でIPブロック）
* セキュリティグループ設定：22, 80, 443番ポートのみ開放
* `/etc/ssh/sshd_config` にて `PermitRootLogin no` 設定済み（rootログイン無効化）
* パスワードログインを無効化（`PasswordAuthentication no`）し、鍵認証のみに限定
* IP直打ちアクセスは403で拒否（FQDNアクセスのみ許可）

---

### 📊 アクセスログの見える化

`goaccess` を使って、ApacheのアクセスログをHTMLでグラフ化しました。  
Webブラウザからアクセス解析の可視化が可能です。

さらに、cronジョブにより `goaccess` を **10分ごとに自動実行**しており、  
`report.html` は常に最新のログ情報に基づいて自動更新されています。

---

## 🔹 手動構築構成（LPICスキル活用）

### 🛠 開発環境

* Amazon EC2（Amazon Linux 2023）
* Apache（Webサーバ）
* お名前.com で取得した独自ドメイン
* Certbot（Let's Encrypt）でHTTPS化 → 自動更新はcronで実施
* Basic認証（.htpasswd）でログイン制限 → `/secret` のみに適用
* fail2ban（SSHの攻撃対策）→ 5回ログイン失敗で一時BAN
* goaccess（アクセスログのグラフ表示）
* GitHub（コードと設定を管理）

### 📁 フォルダ構成（設定・Web・セキュリティ）

```plaintext
/var/www/html/
├── index.html          # トップページ
├── report.html         # サンプルページ
└── secret/             # Basic認証エリア（.htaccessで制限）

/etc/httpd/
├── conf/               # Apacheのメイン設定（httpd.conf）
├── conf.d/             # HTTPSバーチャルホスト設定（httpd-le-ssl.conf: 443, httpd-redirect.conf: 80）
├── conf.modules.d/     # Apacheモジュール構成

/etc/letsencrypt/
├── live/               # 実際に使われる証明書
├── archive/            # 証明書の履歴（過去分）
├── renewal/            # 自動更新の設定ファイル
├── cli.ini             # certbot CLI設定

/etc/fail2ban/
├── jail.d/             # jailルール設定（例：sshd.conf）
├── filter.d/           # フィルタ定義（ログのパターン）
├── fail2ban.conf       # 本体設定
```

---

## ⚙️ 自動化構成（Ansible + Route 53）

Ansibleでサーバ構築を自動化し、Route 53でDNS設定も一元管理しています。

### 🛠 使用技術

* **Ansible**：Apache／fail2ban／goaccess／Basic認証を自動構成
* **Route 53**：Aレコード・NS・SOAなどを自動で管理し、ドメインとEC2を紐づけ

### ✅ 自動化で実現できること

* Apacheの自動インストール・HTTPS対応（Let's Encrypt）
* fail2banの導入と有効化
* Basic認証の設定（.htpasswd生成）
* goaccessをソースからビルドして可視化ページ生成
* HTTP→HTTPS リダイレクト設定

---

## ✅ Ansible 実行ログ

構成はすべてAnsibleにより自動化されています。  
実行結果の全体ログを以下にまとめています：

📄 [▶ Ansible 実行ログを見る](./ansible_ec2_setup/ansible-output.txt)

---

### 📁 ansibleディレクトリ構成例

```plaintext
ansible_ec2_setup/
├── site.yml
├── inventory                    # インベントリ（localhost指定）
├── tasks/
│   ├── setup_apache.yml
│   ├── setup_certbot.yml
│   ├── setup_fail2ban.yml
│   ├── setup_htpasswd.yml
│   ├── setup_https_redirect.yml
│   ├── setup_goaccess.yml
│   └── setup_sshd_config.yml
├── files/
├── ansible-output.txt

.github/
└── workflows/
    └── deploy.yml              # GitHub Actionsワークフロー定義

---

## 🚀 CI/CD（継続的インテグレーション & デプロイ）

GitHub Actions を使って、以下のような **CI/CDパイプライン**を構築しています。

### ✅ 流れ

1. GitHub の `main` ブランチにコードが push されると、自動でワークフローが起動（CI）
2. GitHub Actions が構成エラー（Ansibleの構文や設定ミス）をチェック
3. 問題がなければ EC2 に SSH 接続（CD）
4. `ansible-playbook` を実行し、構成が本番サーバに即時反映

### ✅ 特徴

- `.github/workflows/deploy.yml` により自動実行
- `ansible-playbook` によって構成の検証と反映を一括で処理
- GitHub Secrets により SSH鍵・接続先情報を安全に管理
- 手動ログイン（TeraTerm）とも併用可能な安全設計
- CI（構成検証）＋ CD（自動反映）の両方を一貫して実現

### ✅ 使用技術

- GitHub Actions（CI/CDパイプライン）
- Ansible（構成管理）
- Amazon EC2（本番環境）
- SSH秘密鍵（Secrets管理で安全な接続）

---
