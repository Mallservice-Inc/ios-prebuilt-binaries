# ios-prebuilt-binaries

iOS 向けの二進を、**Swift Package Manager が取得できる形**で置いています。

⚠️ **公開でなければなりません。** SPM は `binaryTarget` の二進を
**認証なしで**取りに行くため、非公開だと開発機と CI の両方で失敗します。
置いてあるのは再配布可能な二進だけで、自社のコードは入っていません。

## いま置いてあるもの

### Isar Core 3.1.0 (iOS) — `isar-3.1.0`

pub.dev の `isar_flutter_libs 3.1.0+1` が同梱している `isar.xcframework`
**そのもの**（無改変）。

上流（`isar` / `isar-community`）は Swift Package Manager に未対応で、
しかも🔴 **repo に二進が入っていない**（公開時に取りに行く作り）ため、
git 参照では取れません。だからここに置いています。

libisar.a（ios-arm64）の公開記号: 92 個
SPM の checksum: `20adf74a0eb8e13987eea63732005a6053d3026b6a5a55d91bc7c62ca5f4c55f`

## 消したもの

### TensorFlow Lite 2.12.0 (iOS) — 2026-09-05 に削除

⚠️ **要らなくなりました。** iOS の顔照合・なりすまし検知を
**Core ML（OS 内蔵）**へ移したためです。Android は Maven の
Google 公式（`org.tensorflow:tensorflow-lite`）を使います。

同じ理由で ML Kit も **Vision（OS 内蔵）**へ移しました。

経緯は本体 repo の `docs/operations/COCOAPODS_READONLY_MIGRATION.md`。

## なぜ在るのか（背景）

CocoaPods の trunk は **2026-12-02 に読み取り専用**になります。
`pod install` はその後も動きますが、🔴 **新しい版が二度と来なくなります**。

そこで CocoaPods でしか解決できないプラグインを無くしました。
残っているのは Isar だけで、上流が SPM に対応したらこの repo ごと不要になります。
