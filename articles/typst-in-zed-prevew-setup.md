---
title: ZedでTypstの執筆・プレビュー環境を整える方法
emoji: 🪙
type: tech
topics:
  - typst
  - zed
  - fedora
  - rust
  - cargo
published: false
---
## 前提

### Typstとは

- 英語Docs
- 日本語Docs

- LaTeXのalternative
	- 環境設定がとにかく大変な印象
- オープンソース

- Webエディタも存在する
	- LaTeXにおけるOverleafみたいな
- オフラインでも作業したいし，ローカルでやれば容量制限もないのでっローカルで使う

### Zedとは
- とにかく軽量なエディタです．
- 筆者は近年のMicrosoft製品（Windows, GitHub, VSCode）における一連のCopilot製品群の押し付けや，そもそもデザインの使いにくさからVSCode (VSCodium)の使用を控えています．
- 
## Typstのセットアップ方法
### Rust/Cargoのインストール
- TypstはRustという言語で製作されているので，Typstのインストール前にRustとCargo（Rustに関連するファイル群の管理をするツール; パッケージマネージャ）をインストールします．
- Rustなしでのインストールもある

インストール方法はドキュメント「[Rust をインストール - Rustプログラミング言語](https://rust-lang.org/ja/tools/install/)」をご参照ください．

筆者の使用するFedora Linuxでは
```sh
# Fedoraの場合
sudo dnf install rust cargo
```
でインストールできます．



### Typstのインストール

Rust/Cargo 経由で Typst をインストール:
```sh
cargo install --locked typst-cli
```
### Zedでの設定
ZedでTypstに関連する機能を提供するTinymistをインストールします．
`Zed > Extensions` (Ctrl-Shift-X) を開き「Typst」と検索し，[Typst Extension](https://github.com/zed-extensions/typst)をインストールします．
![](/images/typst-extension-zed.webp)
次に，自動的にPDFでプレビューできるようにZedの設定を変更します．
```jsonc:~/.config/zed/settings.json
{
  // 他の設定
  "lsp": {
    "tinymist": {
      "settings": {
        "exportPdf": "onSave",
        "outputPath": "$dir/$name",
      },
    },
}
```
上記の設定ではPDFファイルがTypstファイルと同じフォルダに生成されます．プロジェクトのルートディレクトリにPDFを書き出したい場合は`"outputPath": "$root/$name"`と書き換えます．

これで，`ファイル名.typ`というファイルを作ると，同じディレクトリ内に`ファイル名.pdf`が生成されます．