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

現在 `REPLACE_WITH_*_PLAY_APP_SIGNING_SHA256` のプレースホルダ。**Google Play への公開前後に
実際の証明書フィンガープリントへ差し替えるまで、Android の App Links 検証は成立しない**
（その場合はリンクがブラウザで開き、中間ページからストアへ誘導される＝動作としては劣化のみ）。

取得方法（いずれか）:

- Play Console → 対象アプリ → テストとリリース → アプリの整合性 → アプリ署名鍵証明書の SHA-256
  （Play App Signing を使う場合はこちらが正。アップロード鍵ではない）
- `eas credentials`（プラットフォーム Android）→ 表示される Keystore の SHA-256 Fingerprint

`AA:BB:...:FF` 形式（コロン区切り・大文字）でそのまま貼る。

### 各招待ページの `APP_STORE_ID`

`invite/hajimari/index.html` / `invite/okaeri/index.html` の JS 冒頭。App Store 公開時に採番される
数値 ID を入れる。空の間は App Store の検索結果ページへ誘導する（リンク切れ回避）。

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
