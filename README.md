# Minecraft用 Discordユーザー申請管理システム

DiscordとローカルWeb UIを連携させた、安全で効率的なMinecraftユーザーの申請・承認管理システムです。

## ✨ 概要

Discordボットを介してユーザーからの登録申請を受け付け、管理者がローカルで動作するWebインターフェースを通じて申請内容を確認・承認します。申請データは **Firebase (Firestore)** に暗号化して一時保管され、承認されたデータのみがローカルの **SQLite** データベースに永続的に保存されるため、安全なユーザー管理を実現します。

## 🚀 主な機能

- **🤖 Discordからの簡単申請**: ユーザーはDiscordの `/create` コマンドから直感的に登録申請を行えます。
- **🔒 安全なデータ転送**: 申請データはマスターキーで暗号化され、安全に一時保管されます。
- **🌐 ローカルWeb管理画面**: 管理者は自身のPC上でのみ動作するWebページで、以下の操作を実行できます。
  - 申請内容の確認と、承認・否認の実行
  - 承認済みユーザーの一覧表示
  - 名前やIDによるユーザー検索
  - 各項目での並べ替え（ソート）
  - 承認済みユーザーの削除（カスタム確認アラート付き）
- **🔗 QRコードによる承認フロー**: ボットが管理者向けの承認用URLを含んだQRコードを生成し、スムーズな連携を実現します。

## ⚙️ システム構成

本システムは、以下の4つのコンポーネントが連携して動作します。

1.  **Discordボット (Node.js)**
    - ユーザーからの申請を受け付ける窓口。入力された情報を暗号化し、**Firebase**へ一時保存します。
2.  **Firebase (Firestore)**
    - 承認待ちの申請データを一時的に保管するクラウドデータベース。データは暗号化されているため安全です。
3.  **ローカルWebサーバー (Node.js / Express)**
    - 管理者のみが自身のPCでアクセスするWebインターフェースを提供します。Firebaseから申請データを取得・復号し、承認処理を実行します。
4.  **SQLite**
    - 承認されたユーザーデータを永続的に保存するローカルデータベースファイルです。

### 🔄 データフロー
```
[👤 ユーザー]  ==(① 申請)==>  [🤖 Discordボット]
|
V  (② 暗号化)
[🔥 Firebase]
|
V  (③ 承認依頼)
[👑 管理者]  ==(④ 確認)==>  [💻 ローカルWebサーバー]
|
V  (⑤ 復号 & 承認)
[🗃️ SQLite]
```

## 🛠️ 必要なもの

- [Node.js](https://nodejs.org/) (v18.x 以上を推奨)
- [Discordアカウント](https://discord.com/)と、作成済みのDiscordボットアプリケーション
- [Googleアカウント](https://accounts.google.com/)と、作成済みのFirebaseプロジェクト

## 🚀 セットアップ手順

1.  **リポジトリのクローンと移動**
    ```bash
    git clone <リポジトリのURL>
    cd <リポジトリ名>
    ```

2.  **依存パッケージのインストール**
    ```bash
    npm install
    ```

3.  **`.env` ファイルの作成と設定**
    プロジェクトのルートに `.env` という名前のファイルを作成し、以下の内容を記述・設定してください。

    ```env
    # Discordボットのトークン
    DISCORD_TOKEN=your_discord_bot_token

    # Firebaseサービスアカウント情報 (JSONを一行に圧縮して貼り付け)
    FIREBASE_SERVICE_ACCOUNT_JSON='{"type": "service_account", ...}'

    # 暗号化マスターキー (非常に重要)
    MASTER_ENCRYPTION_KEY="ここに非常に強力でランダムな秘密の文字列を生成して貼り付け"
    
    # Webサーバーへの接続を許可するIPアドレス (複数許可する場合はカンマ区切り。空欄の場合は制限なし)
    ALLOWED_IPS=127.0.0.1

    # ローカルWebサーバーのポート番号 (通常は3000のままでOK)
    PORT=3000
    ```

    - **`DISCORD_TOKEN`**: [Discord Developer Portal](https://discord.com/developers/applications)で、自身のボットのページから取得します。
    - **`FIREBASE_SERVICE_ACCOUNT_JSON`**:
        1.  [Firebase Console](https://console.firebase.google.com/)で自身のプロジェクトを開きます。
        2.  `プロジェクトの設定` > `サービスアカウント` タブへ移動します。
        3.  「新しい秘密鍵の生成」ボタンを押し、キーファイル（JSON）をダウンロードします。
        4.  ダウンロードしたJSONファイルの中身を**すべてコピーし、改行を削除して一行の文字列にしたもの**を、`''`の中に貼り付けます。
    - **`MASTER_ENCRYPTION_KEY`**:
        > **⚠️【最重要】このキーはシステムの心臓部です。**
        > 一度設定したら絶対に紛失・変更しないでください。変更した場合、過去の申請データがすべて復号できなくなります。パスワード生成ツール等で、64文字以上の英数記号を混ぜた、推測不可能な文字列を設定してください。
    - **`ALLOWED_IPS`**: 管理画面にアクセスできるIPアドレスを制限します。自分のIPアドレスなどを設定してください。複数指定する場合はカンマ(`,`)で区切ります。制限しない場合は空欄にします。
    - **`PORT`**: ローカルWebサーバーが使用するポート番号です。通常は`3000`のままで問題ありません。

4.  **アプリケーションの起動**
    以下のいずれかの方法でアプリケーションを起動できます。
    
    - **Node.jsで直接起動:**
    ```bash
    node index.js
    ```
    コンソールに「ボット起動完了」と「Webサーバー起動完了」のメッセージが表示されれば成功です。
    
    - **`start.sh` スクリプトで起動 (推奨):**
    本システムは、start.sh というシェルスクリプトを提供しており、これを使用することで簡単にアプリケーションを起動できます。このスクリプトは、Node.jsアプリケーションをバックグラウンドで実行し、エラーが発生した場合に自動的に再起動するなどの機能を提供する場合があります。
    ```bash
    ./start.sh
    ```
    このコマンドを実行すると、DiscordボットとローカルWebサーバーの両方が起動します。ターミナルにログが表示され、ボットとWebサーバーが正常に動作していることを確認できます。
    
## 🎮 使用方法

### ユーザー向け

1.  Discordサーバーで `/create` コマンドを実行します。
2.  表示されたモーダルに、名前、読み仮名、Minecraft IDを入力して送信します。
3.  ボットから、管理者への申請用QRコードとURLが非表示メッセージで送られてきます。
4.  そのQRコードまたはURLを管理者に提示してください。

### 管理者向け

1.  **申請の承認**
    - ユーザーから提示されたQRコードを読み取るか、URLをブラウザで開きます。
    - ローカルWebサーバーが起動していれば、`http://localhost:3000/approve?code=...` のような承認ページが表示されます。
    - 表示された申請内容を確認し、「承認して登録」または「否認する」ボタンを押します。

2.  **承認済みユーザーの管理**
    - ブラウザで `http://localhost:3000/admin` にアクセスします。
    - SQLiteに保存されている、承認済みの全ユーザーリストが表示されます。
    - **検索**: 右上の検索ボックスでユーザーを絞り込めます。
    - **並べ替え**: テーブルのヘッダー（「ID」や「名前」など）をクリックしてリストをソートできます。
    - **削除**: 各ユーザーの行にある「削除」ボタンで、確認後にユーザーを完全に削除できます。

## 📁 ファイル構成
```
.
├── commands/
│   └── create.js         # Discordスラッシュコマンドの定義
├── utils/
│   └── encryption.js     # 暗号化・復号ロジック
├── views/
│   ├── admin.ejs         # 管理ダッシュボードのWebページ
│   └── approve.ejs       # 申請承認用のWebページ
├── .env                  # 環境変数の設定ファイル (重要)
├── database.js           # SQLiteデータベースの初期化
├── firebase.js           # Firebase Admin SDKの初期化
├── index.js              # ボットとWebサーバーを起動するメインファイル
├── package.json
└── approved_users.db     # 承認済みユーザーが保存されるDBファイル (自動生成)
```

## 📄 ライセンス

[MIT License](LICENSE)
