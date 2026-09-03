# 悠三堂 有機JAS記録 / Yusando Organic JAS Records

栽培・加工・格付の記録と台帳を、スマホから記録してチームで共有するアプリです。
Cloudflare Workers + D1 で動き、圏外でも記録できます(電波が戻ると自動で同期)。

## 公開手順(初回のみ・約10分)

### 1. GitHub にこのフォルダを置く
1. https://github.com/new で新しいリポジトリを作る(例: `yusando-jas`、**Private** でよい)
2. このフォルダの中身をすべてアップロードする
   (`wrangler.toml`, `package.json`, `schema.sql`, `src/`, `public/`, `.gitignore`, `README.md`)

### 2. Cloudflare でリポジトリをつなぐ
1. Cloudflare ダッシュボード → **Workers & Pages** → **Create** → **Workers** → **Import a repository**
2. GitHub と連携し、作ったリポジトリを選ぶ
3. 設定はそのまま(ビルドコマンド不要)。**Deploy** を押す
4. 数十秒で `https://yusando-jas.<アカウント名>.workers.dev` のURLができる

データベース(D1 `yusando-jas`)はすでに作成済みで、`wrangler.toml` に紐づいています。
テーブルも作成済みです。追加の設定は要りません。

### 3. はじめて開いたとき
最初に開いた人が **チームの合言葉**(6文字以上)を決めます。
あとの2人には同じ合言葉を伝えてください。名前は各自が入れます。

### 4. スマホのホーム画面に追加(推奨)
- iPhone: Safariで開く → 共有 → 「ホーム画面に追加」
- Android: Chromeで開く → メニュー → 「ホーム画面に追加」

これで圏外の畑でも開けます。

## 更新するとき
GitHub のファイルを更新すれば、Cloudflare が自動で再デプロイします。

## Googleカレンダーに表示する
台帳タブ →「Googleカレンダーに表示する」のURLをコピーし、
パソコンの Googleカレンダー → 他のカレンダー「＋」→「URLで追加」に貼り付けます。
栽培・加工・格付の記録が終日の予定として並びます(読み取り専用)。
Google が読みに来る間隔は数時間〜1日程度です。

## バックアップ
台帳タブ →「全データのバックアップ」で、全記録を JSON でダウンロードできます。
月に一度はダウンロードして保管してください。

## 独自ドメインにしたいとき
Cloudflare ダッシュボード → このWorker → Settings → Domains & Routes → Add
で `kiroku.yusando.com` のようなドメインを付けられます(yusando.com が Cloudflare 管理なら数クリック)。

## 開発者向け
```
npm install
npx wrangler d1 execute yusando-jas --local --file=schema.sql
npx wrangler dev
```
