# 悠三堂 有機JAS記録アプリ（kiroku）

畑での作業・加工・格付を記録し、有機JASの認証と検査に使う記録を残すためのアプリ。
利用者は3人（礒﨑・ジェフ・ヴィンセント）。主にスマホから、圏外の茶畑でも使う。

- 本番: https://kiroku.isozaki-f67.workers.dev
- Cloudflare Worker 名: `kiroku` / D1: `yusando-jas`（id `7fa0a30a-d63d-47dd-8312-e633b69481b7`, APAC）
- リポジトリ: Yusandonatural/kiroku（main にpushすると Cloudflare が自動デプロイ）

## 構成

ビルド工程なし。`wrangler.toml` の assets 設定で `public/` をそのまま配信し、`/api/*` だけ Worker が処理する。

```
src/worker.js   API（認証・同期・iCal・バックアップ）。依存ライブラリなし
public/index.html  アプリ本体。HTML/CSS/JS を1ファイルに同梱（約4,500行）
public/sw.js    Service Worker。リリース時に VERSION を上げる
schema.sql      D1 のテーブル定義（records / masters / settings / audit）
```

## データの考え方

記録は4種類。`records` テーブルに `kind` と JSON の `body` で入る。

| kind | 内容 | ロット番号 |
|---|---|---|
| log | 日々の作業（収穫量は圃場ごと） | H-/U-（未申請圃場はU） |
| proc | 加工（荒茶→仕上げ→選別→粉砕） | A- / S- / P- / M-（有機でないものは AU- のようにUが入る） |
| grade | 格付・小分け | G- |
| clean | 洗浄・切替（加工の認証に必要） | なし |

圃場・機械・工場・作業者は `masters` の `all` に1行のJSONで入る。
ロットは「1圃場=1ロット」で、収穫→荒茶→仕上げ→格付までロット番号でつながる。

## 気をつけること

- **MASTER_REV を安易に上げない。** 上げると `load()` が SEED から圃場・機械を入れ直すため、
  本番にある19筆（移行時にSQLで追加した分を含む）が7筆に戻り、それが同期で本番へ書き戻される。
  ラベルだけ更新したい場合は mastersDirty を立てる方法を使う。
- 日本語と英語を必ず併記する（`<span class="ja">`/`<span class="en">` のペア）。海外のスタッフが使う。
- 外部ライブラリを足さない。オフラインで動くことが要件。Excel書き出しも自前のZIP+XMLで作っている。
- 同期は updated_at の新しい方が勝つ（last-writer-wins）。サーバ側で古い更新は弾く。
- 認証はチーム共通の合言葉（PBKDF2）+ 名前。セッションはHMAC署名のcookieで180日。
- リリースのたびに `public/sw.js` の `VERSION` を上げる（上げないと古い画面が残る）。

## 確認のしかた

ローカルに静的サーバを立てて Playwright で見るのが早い。`/api/*` はスタブで返す。

```
cd public && python3 -m http.server 8931
# 別シェルで Playwright から http://localhost:8931/index.html を開き、
# page.route('**/api/**', ...) で {setup:true, member:'...', ok:true, records:[], masters:{}} を返す
```

D1 の本番データは Cloudflare の MCP か `npx wrangler d1 execute yusando-jas --remote` で確認できる。

## いまの状況（2026-10）

- 加工（製茶）の有機JAS認証はこれから申請予定。「洗浄」タブと申請書の加工シートはそのために作ってある。
- 台帳タブの「継続申請書の下書き」は、申請書A-Jと同じセル位置でExcelを書き出す。
- record.yusando.com を当てたいが、yusando.com のDNSが Route 53 のため Worker のカスタムドメインが付けられない。
  使うなら yusando.com のDNSを Cloudflare に移す必要がある。
- 2025年度の申請書では小麦奥（BTF①-A）の収穫量が空欄だが、作業日誌には2025年5月に548kgの記録がある。要確認。
