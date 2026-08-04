# .well-known — アプリのディープリンク検証ファイル

みまもりアプリの招待URL（`https://class-orange.com/invite/{hajimari,okaeri}/?t={TOKEN}`）を
インストール済み端末でアプリが直接開くための設定。anshin-platform の issue #167 で追加した。

> ⚠️ リポジトリ直下の `.nojekyll` が必須。GitHub Pages は既定で Jekyll が動き、
> ドット始まりのディレクトリ（`.well-known/`）を出力から除外するため、
> `.nojekyll` が無いと 404 になる。

| ファイル | 対象 | アプリ側の対応設定 |
|---|---|---|
| `apple-app-site-association` | iOS Universal Links | `app.json` の `ios.associatedDomains: ["applinks:class-orange.com"]` + `ios/app/app.entitlements` |
| `assetlinks.json` | Android App Links | `app.json` の `android.intentFilters`（`autoVerify: true`）+ `android/app/src/main/AndroidManifest.xml` |

## リリース時に埋める値（TODO）

### `assetlinks.json` の `sha256_cert_fingerprints`

**はじまりの刻（`com.norizooo.hajimari`）は設定済み。** おかえりナビは Play 未公開で
フィンガープリントを取得できないため、**エントリごと未記載**にしてある。不正な値の
エントリを置くと statement 全体の解釈に影響しうるので、プレースホルダは残さない方針。
おかえりナビを公開する際に、はじまりの刻と同じ形のオブジェクトを配列に追加すること。

未設定の間はそのアプリの App Links 検証が成立しないだけで、リンクはブラウザで開き
中間ページからストアへ誘導される（動作としては劣化のみ）。

取得方法:

Play Console → 対象アプリ → **Google Play による保護** → 「Google Play ストアの保護」を展開
→ `アプリ署名鍵の保護` の行の **「Play アプリ署名の管理」** ボタン
→ **「アプリ署名鍵証明書」** の SHA-256

⚠️ 同じページの「アップロード鍵証明書」ではない。Play App Signing で再署名された後の鍵でないと
検証は通らない。`eas credentials` が表示するのもアップロード鍵なので使えない。

⚠️ Play Console の画面構成は頻繁に変わる（旧: テストとリリース → アプリの完全性）。
メニューで見つからない場合は URL の末尾を `/keymanagement` に差し替える。

`AA:BB:...:FF` 形式（コロン区切り・大文字）でそのまま貼る。

### 各招待ページの `APP_STORE_ID`

`invite/hajimari/index.html` / `invite/okaeri/index.html` の JS 冒頭。App Store 公開時に採番される
数値 ID を入れる。空の間は App Store の検索結果ページへ誘導する（リンク切れ回避）。

**はじまりの刻の ID は判明済み: `6767998433`**（2026-08-02・TestFlight 配信時に確認）。
App Store Connect のアプリ ID と App Store の URL の数値は同一で、
`https://appstoreconnect.apple.com/apps/6767998433/testflight/ios` の `apps/` 直後に現れる。
App Store Connect → App 情報 → 一般情報 → Apple ID でも確認できる。

> ⚠️ **公開までは埋めないこと。** `https://apps.apple.com/jp/app/id{ID}` は
> アプリが App Store で公開されるまで **404** を返す。値を入れた時点でこの URL が使われるため、
> 未公開のまま埋めるとリンク切れになる。空のまま検索ページへ誘導する現状の方が正しく機能する。
> **App Store 公開と同時に埋める。**

おかえりナビはリリース見送り中のため未採番（`assetlinks.json` のエントリ未記載と同じ理由）。

## 検証方法

```bash
# 配信されているか（HTTP 200・リダイレクトなしであること）
curl -sI https://class-orange.com/.well-known/apple-app-site-association
curl -s  https://class-orange.com/.well-known/assetlinks.json

# Android App Links の検証状態（実機・アプリインストール後）
adb shell pm get-app-links com.norizooo.hajimari
```

iOS は実機にインストール後、招待URLをメール等からタップしてアプリが直接開けば成立。
（Safari のアドレスバーに直接入力した場合は Universal Links が発動しない仕様）
