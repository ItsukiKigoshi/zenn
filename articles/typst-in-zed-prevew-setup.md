---
title: ZedでTypstの執筆・プレビュー環境を整える方法
emoji: 🐂
type: tech
topics:
  - typst
  - zed
  - fedora
  - rust
published: true
---
## 🥬本日使用するお材料の紹介

### 🥕Typstとは
[Typst](https://typst.app/)とは，2023年にリリースされたRust製の新興組版エンジンです．

より簡単に執筆できるように設計された[LaTeX](https://www.latex-project.org/)のようなもので，数式や表を含むドキュメント/スライド等をシンプルな関数で記述できます．

今回紹介した方法の他に，[Typst Webエディタ](https://typst.app/)（LaTeXで言うところの[Overleaf](https://www.overleaf.com/)）を使う方法があります．気軽に試してみたい方はまずそちらをどうぞ．

https://typst.app/

#### 余談: Why Typst?
- 私は数学を専攻している大学生で，セミナーの発表資料で数式入りのPDFファイルを作成する必要に迫られ，普段使っているObsidianのインライン数式+PDF書き出しでは流石に限界を感じたので，卒業論文執筆まで使える執筆環境を検討しました．
- LaTeXはサイズが大きい印象があり，またLaTeX冒頭に書かなくてはならない呪文が嫌いなので代替策を探し，Typstを（再）発見しました．
- TypstはフリーミアムのWebエディタが存在するのでてっきりプロプライエタリ（有償/非公開）ソフトウェアかと思って敬遠していましたが，組版エンジンはオープンソースで開発されており^[[https://typst.app/open-source/](https://typst.app/open-source/)を参照]，自分のパーソナルコンピュータ上だけで執筆環境を整えられます．
- オフラインかつお気に入りのエディタで執筆できて素敵ですね．

### 🧅Zedとは
- [Zed](https://zed.dev/)は，これまたRustで開発されている，とにかく軽量なエディタです．
  - 筆者は近年のMicrosoft製品（Windows, GitHub, VSCode）における一連のCopilot製品群押し付けや，そもそもどこに何の機能があるのか直感的に分からないデザインの使いにくさからVSCode (VSCodium)の使用を控えています．GitHubはまだしばらく権威がありそうだけれど．
  - しかし，今回導入するTypst環境を含め，Playwright（Webのテスト環境）などVSCodeにしかない/充実しているプラグインがある点が口惜しいです．

https://zed.dev/

## 🫕ZedでTypstプレビュー環境設定
### 🧈Typst Extensionのインストール
ZedでTypst関連機能を提供する[Tinymist](https://github.com/Myriad-Dreamin/tinymist)をインストールします．
`Zed > Extensions` (Ctrl-Shift-X) を開き「Typst」と検索し，[Typst Extension](https://github.com/zed-extensions/typst)をインストールします．
![Zed拡張機能で「Typst」の検索結果](/images/typst-in-zed-prevew-setup/typst-extension-zed.webp)
いよいよお料理の始まりです．バターはごま油でも代用できます．

### 🧂Tinymistの設定
次に，自動的にPDFが出力されるようにZedの設定を変更します．
塩コショウが効いて味がつきますね．
```jsonc:~/.config/zed/settings.json
{
  // 他の設定
  "lsp": {
    "tinymist": {
      "settings": {
        "exportPdf": "onSave",
        "outputPath": "$root/$dir/$name",
      },
    },
  },
}
```
- 上記の設定では，`.typ`ファイルの保存時にTypstファイルと同じフォルダにPDFファイルが出力されます．
  - `"exportPdf": "onType"`とすることで，エディタで`.typ`ファイルを編集するなりPDFが出力されるようになります．
  - `"outputPath": "$root/$name"`とすることで，PDFの出力先をプロジェクトのルートディレクトリにできます．
- 他のオプションについては[Tinymist Documentation](https://myriad-dreamin.github.io/tinymist/feature/preview.html)をお読みください．

これで，`ファイル名.typ`というTypstファイルを作ると，同じディレクトリ内に`ファイル名.pdf`というPDFが出力されるようになります．

![Zed上でTypstファイルから自動で書き出されたPDFファイル](/images/typst-in-zed-prevew-setup/tinymist-pdf-exported.webp)

素晴らしい！

### 🥢味見: 実際に見てみましょう．


出力されたPDFをお気に入りのプレビューアプリで開き，Zedの横に並べれば...
![ZedとPDFプレビューを並べた最高の執筆環境](/images/typst-in-zed-prevew-setup/typst-preview.webp)
この通り！プレビュー環境が整いました．
味が足りない場合はお好みでホットソースでもかけてお召し上がりください．

### 😑その後...

せっかく作った料理なのに，家族が帰ってきません...
いつ帰ってきても温められるようにNEWSの「[チャンカパーナ](https://www.youtube.com/watch?v=Ou1UG_PC43Q)」でも聴きながら待ちましょう．早く帰って来ないかなあ．

お料理終わり．

木越 斎 (2026/05/10; Provided with [CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.en))


:::details 変更履歴
- 2026/05/10T22:03UTC Zedでのプレビュー環境構築にtypst-cliのインストールは不要だったため，当該記述を削除しました．
- すべての変更履歴は[GitHub](https://github.com/ItsukiKigoshi/zenn.dev/commits/main/articles/typst-in-zed-prevew-setup.md)で確認できます．
:::
