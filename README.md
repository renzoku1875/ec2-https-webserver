# EC2 + HTTPS サーバ構築ポートフォリオ

## 🔧 Webサイト作成・管理（HTTPS対応）

LPICを勉強して、実際にAWSのサーバ（EC2）でWebサイトを作ってみた記録です。
できるだけ実務に近い構成を自分で調べながら作りました。

---

## 🛠 開発環境

* Amazon EC2（Amazon Linux 2023）
* Apache（Webサーバ）
* お名前.com で取得した独自ドメイン
* Certbot（Let's Encrypt）でHTTPS化 → cronで自動更新済み
* Basic認証（.htpasswd）でログイン制限 → `/secret` のみに適用
* fail2ban（SSHの攻撃対策）→ 5回ログイン失敗で一時BAN
* goaccess（アクセスログのグラフ表示）
* GitHub（コードと設定を管理）

---

## 🌐 サイトURL

* [https://mytest-portfolio.xyz](https://mytest-portfolio.xyz)
* [https://mytest-portfolio.xyz/secret（Basic認証あり）](https://mytest-portfolio.xyz/secret（Basic認証あり）)

---

## 🗂 サーバ構成図

以下は、Amazon EC2 上に構築した Web サーバの全体構成図です。

![EC2構成図](./images/ec2-architecture.png)

---

## 🔐 セキュリティの工夫

* HTTPアクセスはすべてHTTPSへ自動リダイレクト
* `.htpasswd` を使って特定ページにログイン制限を設定
* fail2banでSSH攻撃対策（5回失敗でIPブロック）
* セキュリティグループ設定：22, 80, 443番ポートのみ開放

---

## 📊 アクセスログの見える化

`goaccess` を使って、ApacheのアクセスログをHTMLでグラフ化しました。
Webブラウザからアクセス解析の可視化が可能です。

---

## 📁 フォルダ構成（設定・Web・セキュリティ）

```plaintext
/var/www/html/
├── index.html          # トップページ
├── report.html         # サンプルページ
└── secret/             # Basic認証エリア（.htaccessで制限）

/etc/httpd/
├── conf/               # Apacheのメイン設定（httpd.conf）
├── conf.d/             # certbot の追加設定（httpd-le-ssl.conf）
├── conf.modules.d/     # Apacheモジュール構成

/etc/letsencrypt/
├── live/               # 実際に使われる証明書（fullchain.pem, privkey.pem）
├── archive/            # 証明書の履歴（過去分）
├── renewal/            # 自動更新の設定ファイル
├── cli.ini             # certbot CLI設定

/etc/fail2ban/
├── jail.d/             # jailルール設定（例：sshd.conf）
├── filter.d/           # フィルタ定義（ログのパターン）
├── fail2ban.conf       # 本体設定
```

---
