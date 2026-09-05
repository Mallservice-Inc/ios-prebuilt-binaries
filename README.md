# tflite-ios-binaries

TensorFlow Lite の iOS 向け二進を、**Swift Package Manager が取得できる形**に詰め替えて置いています。

## なぜ在るのか

CocoaPods の trunk は **2026-12-02 に読み取り専用**になります。TensorFlow Lite の
iOS 向け配布は CocoaPods だけで、SPM 用の配布がありません
（Google の `google-ai-edge/LiteRT` には `Package.swift` が在りますが、
そこが参照する `prebuilt/*.xcframework.zip` は 2026-09 時点でどのタグにも公開されていません）。

SPM の `binaryTarget(url:checksum:)` は **zip しか受け付けない**のに対し、
Google 公式の配布は `.tar.gz` です。そのため詰め替えが要ります。

## 中身

Google 公式の配布物から取り出した二進**そのもの**です。中身は書き換えていません。

⚠️ ただし**入れ物だけ組み直しています**。公式の配布は「静的フレームワーク」形式で、
そのまま Swift Package Manager に載せると 🔴 **Xcode が動的フレームワークと誤認して
アプリに埋め込もうとし、`did not contain an Info.plist` でビルドが止まります**。
`libtool -static` で `.a` にし、`xcodebuild -create-xcframework -library` で
「ライブラリ形式」の xcframework に詰め直してあります。機械語は同じものです。

| 元 | |
|---|---|
| 配布元 | `https://dl.google.com/tflite-release/ios/prod/tensorflow/lite/release/ios/release/25/20230414-162712/TensorFlowLiteC/2.12.0/bf93fad9a2465150/TensorFlowLiteC-2.12.0.tar.gz` |
| tar.gz の sha256 | `88e623a406e06c3bec43ff61637a19a6381dff3031f39f96d00e2be8917bf8cb` |

`TensorFlowLiteC.framework` 本体（ios-arm64）の sha256 は
`d57db1be4d722fea6023efd338b8ea218533b4ec38abeacda186c0428ebf419a` で、
CocoaPods が入れる物と**1 バイトも違いません**（照合済み）。

## 作り直す手順

```sh
curl -L -o tflite.tar.gz "<上の配布元 URL>"
shasum -a 256 tflite.tar.gz          # 上の値と一致することを確かめる
tar xzf tflite.tar.gz

for f in TensorFlowLiteC TensorFlowLiteCCoreML TensorFlowLiteCMetal; do
  args=()
  for slice in ios-arm64 ios-arm64_x86_64-simulator; do
    src="TensorFlowLiteC-2.12.0/Frameworks/$f.xcframework/$slice/$f.framework"
    mkdir -p "out/$f/$slice"
    libtool -static -o "out/$f/$slice/lib$f.a" "$src/$f"
    args+=(-library "$PWD/out/$f/$slice/lib$f.a")
    [ -d "$src/Headers" ] && cp -R "$src/Headers" "out/$f/$slice/Headers" \
      && args+=(-headers "$PWD/out/$f/$slice/Headers")
  done
  xcodebuild -create-xcframework "${args[@]}" -output "$f.xcframework"
  ditto -c -k --sequesterRsrc --keepParent "$f.xcframework" "$f.xcframework.zip"
  swift package compute-checksum "$f.xcframework.zip"
done
```

## ライセンス

Apache License 2.0（TensorFlow）。`LICENSE` を同梱しています。
