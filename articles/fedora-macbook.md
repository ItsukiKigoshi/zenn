---
title: MacBook Pro (Mid 2014)でFedora Linuxをデュアルブートしよう
emoji: 🍋
type: tech
topics:
  - fedora
  - linux
  - macbook
  - macbookpro
  - macos
published: false
---

## 概要
旧式のMacBook Pro (Retina, 13-inch, Mid 2014) を現役で使用している私は，年初（2026年1月）に開発環境としてのmacOS 11 (Big Sur)に限界を感じ，Linux導入に踏み切りました．
ここでは10年以上前に発売されたMacBook ProにFedora Linuxをデュアルブートした手順と，Linux移行に係る所感とLinuxあるいはオープンカルチャーへの想いを綴ります．

![日常に溶けるFedora](/images/fedora-macbook/fedora-daily-life.webp)*日常に溶けるFedora*

## 動機

私は，10年以上前に発売されたMacBook Pro ([Retina, 13-inch, Mid 2014](https://support.apple.com/ja-jp/111942)) を主たるコンピュータとして使用しています（まさかこんなに長く使うことになるなんて！）．
このMacBook Proは，2018年に亡くなった[祖父](https://ja.wikipedia.org/wiki/木越治)の形見として，彼の息子になる私の父親から譲り受けました．それ以来ですから，私が中学生だった2018年からかれこれ8年近く使用していて，今私は大学生ですがまだ現役で活躍しています．
しかし，同MacBook Proへインストール可能な最新OSの[macOS Big Sur](https://ja.wikipedia.org/wiki/MacOS_Big_Sur) (v11; 2020年)は私の趣味のWeb開発に関連する最新の開発環境を導入することは殆ど不可能です．高校生 (2022年あたり)からNode.js v18までしか動作せず，当時のLTSである20系を使えなかった記憶があります^[過去に関わっていたプロジェクトではNode v16を使っているので，ひょっとしたらv18すら動かなかったのかもしれない ([GitHub](https://github.com/hibiya-itchief/2024-quaint-app/blob/d0f200655489b32d398071594d55316d2a31afc8/.github/workflows/ci.yml#L22))]^[少なくともNode.js v20のサポートはmacOS 12以上でありそうです ([Github Discussion](https://github.com/nodejs/node/issues/47067#issuecomment-1474810592))]．

### 技術的な限界の詳細
実際，実験的にHonoのプロジェクトを触ってみようとしたところ，`$pnpm dev`で`Cloudflare Workers runtime cannot run on the current version of macOS`と言われてしまう始末...

```sh
$ wrangler dev
Warning: Unsupported macOS version detected (11.6.0).
The Cloudflare Workers runtime may not work correctly on macOS versions below 13.5.0.
Consider upgrading to macOS 13.5.0+ or using a DevContainer setup with a supported version of Linux (glibc 2.35+ required). 
```

また，Nodeに関連するプロジェクトを開発しようとすると，事あるごとに
```sh
dyld: Symbol not found: __ZNSt3__113basic_filebufIcNS_11char_traitsIcEEE4openEPKcj
Referenced from: /Users/user/.nvm/versions/node/v24.13.0/bin/node (which was built for Mac OS X 13.5)
```
や
```sh
$ pkgx npm create hono@latest app
dyld: Symbol not found: __ZNKSt3__115basic_stringbufIcNS_11char_traitsIcEENS_9allocatorIcEEE3strEv
Referenced from: /Users/user/.pkgx/nodejs.org/v25.5.0/bin/node (which was built for Mac OS X 13.5)
Expected in: /usr/lib/libc++.1.dylib 
```
というようなエラーを目にするようになりました．

まあ，macOSのメジャーアップデートが毎年やってきて，基本的に開発環境は最新OSに追従していることを前提とするならば当然かもしれません...

でも，文章を書いたりYouTubeを見るためにはこのMacで充分なので，これだけでPCを買い換えるのはもったいない...

![カスタムCommand Line](/images/fedora-macbook/cli.webp =400x)*関係ないけど，Berkeleyの寮共用パソコンにインストールされているカスタムCLI*

## macOSで開発環境を整えるための試行錯誤

開発環境を整える上で，まずはmacOS上で動く環境構築を模索しました．

### Docker / OrbStack / Colima [不可]

仮想環境構築方法の常套手段であろう[Docker](https://www.docker.com/)と，そのmacOS向け軽量代替手段である[OrbStack](https://orbstack.dev/)/[Colima](https://colima.run/)を試しましたが，大抵がmacOS 12以上を想定していて，そもそもmacOS 11には対応していないので諦めました．

https://www.docker.com/
https://orbstack.dev/
https://colima.run/

### UTM [不可]

macOS上でLinuxを立ち上げる選択肢として，[UTM](https://mac.getutm.app/)を検討しました．しかし，macOS上でLinuxを起動する以上，マシンの性能を最大限生かしきれず動作がとても実用に耐えるものではありませんでした．

https://mac.getutm.app

### GitHub Codespace [不可]

自身のOSと関係なく環境を構築する選択肢として，[GitHub Codespace](https://github.com/features/codespaces)も検討しました．
学生なら[Student Developer Pack](https://education.github.com/pack)を使えば無料で使えることを高校生の頃に使った経験から覚えていましたが，学生認証ができないのでCodespaceの使用は諦めました．そもそもオンラインでしか開発サーバを立ち上げられないのは不便だし...

具体的には，学生認証で私が交換留学生として滞在しているUniversity of California, Berkeleyのアカウント (`itsuki [at] berkeley.edu`)を使おうとしたところ，私みたいなNon-degree studentは受け入れないようになっているのか，ただの`berkeley.edu`ではなく`*.berkeley.edu`という，どこかの学科に属していることが求められ入るドメインでないと登録不可でした．

私が所属する国際基督教大学のドメイン（`icu.ac.jp`）も，私が日本からアクセスしていない，という理由ではじかれました．

紛れもなく私は学生であるのに，その正銘がこんなに大変だなんて，ちょっとしたアイデンティティクライシスを覚えました．

### Linuxのデュアルブート [採用]

ここまで試して，私に残された選択肢は殆どこれ1つになっていました．データが飛ぶリスクがあったので怖かったのですが，腹を括ってデュアルブートに踏みきることにしました．
そもそも，アクティブなサポートが殆ど切れているようなmacOSはセキュリティの観点からもあまり褒められたものではありません．
macOSから離れる機会をくれてありがとう．

## コラム: 厳格なWindows家系に生まれた私のAppleに対する想い

![2021年のMacBook](/images/fedora-macbook/mac-in-room.webp =300x)*Macと，横に見える親のSurface (2021)*

### 家族はみーんなWindows.
私は，母方の祖父母がWindows 95だか98のパソコン教室を自宅で開いていたり，父方の祖父は昔からMS-DOSを使っていて，父親もMS-DOSを子供の頃に習ったという厳格なWindows家系に生まれたました．

父方の祖父は私が今使っているMacBookでWindows on Parallel Desktopを使っていました．支給されたのがMacBookだったからそれを使っていただけという理由らしい．多分一番使っていたのはワープロソフト[一太郎](https://www.justsystems.com/jp/products/ichitaro/) (Windowsでしか動作しない)だし．わけわかめ！たしかその祖父は生前にmacOSのライブかな漢字変換が嫌いとか，Macに対して良い事は言っていなかったような...

私の父親も，[Let's Note](https://panasonic.jp/cns/pc/products/lineup/)/[VAIO](https://vaio.com/)/Surfaceを使っていて，[秀丸エディタ](https://hide.maruo.co.jp/software/hidemaru.html)（Windows専用のフリーエディタ）でメモとか研究メモの執筆をしています．

うちの母親もバックオフィス系の仕事をしているので，自宅でHPを使っているし．

ほんまに誰一人として非Windowsユーザが家族にいない！

### Macへの羨望
私が生まれ育った山口県山口市には，[山口情報芸術センター](https://www.ycam.jp/)（YCAM）という，最高にクールなメディアアートの開発・展示拠点があります．
https://www.ycam.jp/
今思い返しても，あんな田舎（ちょっと失礼）に世界からメディアアーティストがやってきたり（時に故・[坂本龍一](https://www.ycam.jp/archive/profile/ryuichi-sakamoto/)，[大友良英](https://www.ycam.jp/archive/profile/yoshihide-otomo/)，[ダムタイプ](https://www.ycam.jp/archive/profile/dumb-type/)），常駐の開発集団・[インターラボ](https://www.ycam.jp/aboutus/interlab/)のクールなみなさんが子どもだった私と遊んでくれたりしたあの環境は，紛れもなく山口市の財産だと思います．ありがとうYCAM．

私の趣味の現代アート鑑賞も，プログラミング（YCAMにはArduinoが転がっていた）やオープンカルチャー（作品は積極的にCreative Commonsライセンスで頒布されていた (例: [gonzoCam](https://www.ycam.jp/archive/software-hardware/gonzocam/))）への興味も間違いなくYCAMに根ざしています．

まあ小学生当時の私はコンピュータというより[レゴブロックのこまどり撮影](https://youtube.com/playlist?list=PLVxEAWkQVBq7ZLbCohJz4OtNNkQ6UMr0c)にはまっていたので，全然コンピュータ関連に明るかった訳ではなかったですが．そういえばYouTubeへ動画をアップロードする方法を小学3-4年生の私に教えてくれたのも[深澤孝史](https://www.ycam.jp/archive/profile/takafumi-fukasawa/)さんという美術家の方でした．失礼極まりなかったであろう世間知らず・木越斎少年のガキに色々なことを教えてくれたことには感謝しかありません．今も元気にしているかしら．最後に会ったのは確かコロナ禍が始まってすぐの2020年で，Zoom越しの対面でした．

### アートとMac

さて，脱線が過ぎましたが，彼らが使っていたのが，他ならぬMacintoshだったのです．メディアアートに限らずMacユーザが多いのはアーティストあるあるなのでしょう．当時（2014年）のMacは，今よりも一層「洗練された人が使っているもの」という印象が強かったと思います．山口市という田舎に住んでいたのできっと余計に．りんごマークが光るMacBook Proはもちろん，Mac Pro 
（2013; "ゴミ箱"）を見たときなんて感動しました．
[![ゴミ箱Mac Pro](https://upload.wikimedia.org/wikipedia/commons/d/d6/New_Mac_Pro_%2812093123884%29.jpg?utm_source=commons.wikimedia.org&utm_campaign=index&utm_content=original  =250x)
*Paul Hudson from United Kingdom, CC BY 2.0, via Wikimedia Commons*
](https://commons.wikimedia.org/wiki/File:New_Mac_Pro_(12093123884).jpg)

こんな昔語りができるほど，2004年生まれの私も歳を取りました．
そういえば小さい頃に通っていた[英語教室](http://redrobini.com/)の室長先生もMacが好きで，MacBook ProやiMac，それに発売されたばかりのApple Watchを使っていました．
まだスキューモーフィズムの面影漂う[OS X Mavericks](https://ja.wikipedia.org/wiki/OS_X_Mavericks)でしたね（"OS X"！なんて懐かしい響き！！）．そういえば室長の彼（"katsu-sensei"）は当時New York旅行でタイムズスクエアに行ったときの写真を見せてくれましたが，その約10年後に私も初めてそこを訪れるわけです．あの英語教室に行っていなかったら私は国際基督教大学に通っていないかもしれないし，UC Berkeleyに留学していなかったかもしれません，ふーん，振り返って見ると色々なものがつながっているのです．[Connecting the Dots](https://youtu.be/UF8uR6Z6KLc)^[こんなに喋って，私はただのApple Fanではありませんか！否定は出来ないけど安直ですみません]^[日本語字幕が関西弁でふざけていますね...今気づきました.]...ってか私の中を流れるこのコンピュータへの関心から今年友達が通うスタンフォードにも訪れたわけで，そこでSteve Jobsは20年前に講演をしたわけで．つーか今日ちょうどBerkeleyの卒業式で，ガウンを着た人がいっぱいいるカフェでこれを書いています．スタンフォードのライバルのBerkeleyで．ちょっとこじつけが過ぎますね．


### iMacを買ってもらった

[![iMac Retina](https://upload.wikimedia.org/wikipedia/commons/e/e6/IMac_vector.svg?utm_source=commons.wikimedia.org&utm_campaign=index&utm_content=original =500x)*Rafael Fernandez, CC BY-SA 3.0, via Wikimedia Commons*](https://commons.wikimedia.org/wiki/File:IMac_vector.svg)

小学4年生の木越少年は当時，家にあった4万円の激安NECパソコン（Windows 7）にインストールした[Windows ムービーメーカー](https://ja.wikipedia.org/wiki/Windows_ムービーメーカー)で，図書館から借りたハンズオン本を片手に動画編集を覚えました．しかし，同ソフトウェアの機能は限定的で，たしか1クリップあたりに細かい継続時間指定が出来ないなど，限界も感じ始めていました．
私の父親は子どもの潜在的なクリエイティビティに対する投資は惜しまない方針だったようで，iMovieが標準搭載されているiMacを近くのヤマダ電機だかビックカメラで一緒に買ってくれました．
その後，そのiMacはMacBook Proとともに外付けDVDドライブを指したら電源系統のトラブルで（多分私が適当な外部電源をDVDドライブに差したのでショートした）起動しなくなり，数万円で修理したものの8GBのメモリでの作業は辛く，結局私が高校生の頃だったかPCリサイクルされました．ユニファイドボディは美しいですが，「ディスプレイだけは再利用しよう」みたいことがやりにくいですよね．それはまた別のお話．
### "Yosemite"

ところで，当時の最新OSはOS X Yosemiteで，子どもの私はその美しいデザインと[その発表方法](https://youtu.be/w87fOAG8fjk?t=812)に感動しました．多分人生で初めて見たWWDC Keynote (WWDC 2014)です．Craig Federighiの冗談と美しいデザイン哲学のミックスは今見ても美しい．Yosemiteは10年以上前だけれど，今見ても見劣りがしないどころか，Liquid Glassよりも洗練されているようにさえ思えます．思い出補正マシマシですが．

脱線の脱線ですがBerkeleyに来たからにはYosemite国立公園に行きました．小4の私はカリフォルニア州がどこにあるかも知らなかったわけなのですが，10年の時を経た聖地巡礼です．しかも，大学生になって始めたクライミングの聖地でもあるという，2つの意味での聖地を同時に巡るわけですね．

![El Capitan](/images/fedora-macbook/elcap.webp =400x)
*Yosemiteの次にOS Xの名称になった，ヨセミテ国立公園の象徴: エル・キャピタン岩．私の滞在中には[ユージ・ヒラヤマ](https://yuji-hirayama.com/)がこの壁を登っていたらしい^[実は私が高校生の頃に参加した森美術館のワークショップで彼の娘さんとダンスしており，彼からクライミング講習も受けたことがあることを，ヨセミテに行った後思い出しました．縁過ぎる．[![森美術館のワークショップでユージ・ヒラヤマのジムを訪れたときの写真](https://live.staticflickr.com/65535/51756727258_f07d7ca887.jpg)](https://flic.kr/p/2mRyKQw)*©Mori Art Museum. All Rights reserved. Provided thru [Flickr](https://flic.kr/p/2mRyKQw). Another Energy-Related Community Engagement Program “Art Camp for under 22, Vol. 7 Human Begin: What Are We Doing Tomorrow?” Session #2: Saturday, July 10, 2021 Venue: “Climb Park Base Camp” (b-camp.jp/) in Iruma City, Saitama Prefecture Photo: Tayama Tatsuyuki*]．映画『[フリーソロ](https://ja.wikipedia.org/wiki/フリーソロ)』でアレックスオノルドが綱なしで登ったことも有名ですね．一枚岩の花崗岩としては世界最大であると言われています．*

![壁を登るクラブメイト](/images/fedora-macbook/climbing.webp =400x)
*写真に写る彼女は，（リアルライフでも）パートナーである少し上にいる彼と一緒にフリークライミングしていました．一緒にトラッドクライミングできるカップルなんて素敵過ぎる．日本帰国後の私の1つの目標はフリークライミングの技術を身につけることかな．*

### 他にもApple...
小学生の頃に買ってもらったiPod nanoも，人生で初めて買った携帯電話のiPhone 12 miniもまだ現役．両親が友人の結婚式でもらったか何かの初代iPod Shuffle（容量1GB!）も以前は使っていました．
![Apple Decices Still Workin'](/images/fedora-macbook/apples.webp =250x)
*iPod nano (6th Gen), MacBook Pro, iPhone 12 mini*


### MacBook Pro (Retina, 13-inch, Mid 2014)の魅力
さて，祖父の形見であるMacBook Pro (Retina, 13-inch, Mid 2014)を今でも使っている理由に，ひとつは形見であることもありますが，それが製品として優れているからというのも理由のひとつです．なんせ，
- このMacBook Proには**HDMIポート**がついています
- このMacBook Proには**SDカードスロット**がついています
- このMacBook Proには**MagSafe** (磁石で取り外し可能な充電端子)がついています

上記の3機能は2016年モデルで廃止され，その後また戻ってきた機能たちです．つまり，必要不可欠な機能はこの時代のMacBook Proにもう揃っていたのです！

- 最新のMacBook Proと重量が殆ど変わりません！
  - 最新の再軽量MacBook Pro: [1.55kg](https://www.apple.com/macbook-pro/specs/#:~:text=1%2E55%20kg) 
  - 私のMacBook Pro: [1.57kg](https://support.apple.com/ja-jp/111942#:~:text=1%2E57%20kg2)
- りんごが光ります ![光るりんご](/images/fedora-macbook/shining.webp =500x)
- このMacBook Proは**Intel製チップ**で動作しています
  - 性能面の優劣はさしおいて，Linuxがブートしやすいです．
  - Apple SiliconでのLinuxにはAsahi Linuxという前提OS(?)が必要でちと手間がかかりそう．


長くなりましたが，10年前のAppleが私を形作ったであろうお話でした．

**（コラムおわり）**

## 【本題1】なぜFedora Linux?
結果を先に言えば，私はLinuxディストリビューション^[macOSやWindowsと違って，LinuxではOSの核となる"Linuxカーネル"を元に派生した様々な種類のOSが存在します]として[Fedora Workstation](https://fedoraproject.org/workstation/)を選びました．ここでは簡単にその理由を書きます．この話には議論・闘争が付きものです．以下のミームが物語っています．だから深く立ち入りません．
![Linux is Scary](/images/fedora-macbook/scary-linux.webp =300x)
*"私，猫ミームが見たいだけやねんけど"*

私には，なんとなくですが以下の条件がありました．
- **セットアップの負担が少ない**: 本業が大学生なので，レポートや課題がこなしやすい環境であることが絶対条件でした．インターネットブラウズや印刷，Zoom参加などです．
- **開発環境として不自由しない**: そもそものモチベーションが開発環境を整えることだったので，最新のパッケージが動作することが条件でした

### 検討したディストリビューション
- **[Zorin](https://zorin.com/os/)**
  - 導入したそばから使えそうなOSの筆頭でした．
  - 一度ライブディスクから起動して試しましたが良い感じでした．
  - しかし，後述のFedoraと比べて私に撮っての利点が少なく有償バージョンが存在することが少し嫌で辞めました．
  - FedoraがなければZorinにしてたかも．
- **[NixOS](https://nixos.org/)**
  - よく聞くのでライブディスクで試しました．理想はいいのですが，日常使用には考えることが多すぎると感じました．
  - あと，デフォルトの日本語フォントがガビガビなのがテンション下がりました．
  - ここで作成したNixOSのインストールメディアは，のちのちFedoraのパーティション拡張に使いました．ありがとうNixOS!
- **[Ubuntu](https://ubuntu.com/desktop)**
  - 小学生の頃に[日経Linux](https://www.fujisan.co.jp/product/1281679734/b/1204373/)を買って読んでいたくらいなので当然Ubuntuは検討しました．
  - しかし，本当に単純な理由で，デフォルトデスクトップの色味が怖い（なぜ紫色で，おまけに不気味なマークなのですか？怖すぎる...）のでやめました．これが動物の写真だったらUbuntuを使っていたことでしょう．
  - 試してすらいないです...試すべきだったかな．
- **[openSUSE](https://www.opensuse.org/)**/**[Linux Mint](https://www.linuxmint.com/)**
  - KDEやXfce環境はWindowsっぽく馴染めそうになかったので考慮しませんでした．
- **[Fedora Workstation](https://fedoraproject.org/workstation/)**: *君に決めた！*
  - Fedora Workstationに搭載されているピュアな[GNOME](https://www.gnome.org/fr/)がただ単純に美しかったです．
  - また，Fedoraはパッケージ更新に積極的で，最新の開発環境を手に入れる文脈からも適していました．
  - FedoraはIBM傘下の商業向けLinuxディストリビューションベンダー・Red Hatが開発しているオープンソースでフリーOSです．Red "Hat"に対するFedora帽子ですね．おしゃれ．
  - Just worksすることが特徴です．
    - 面倒な設定無しでもそこそこ動きます．
    - これはFedoraを使い始めた後に知ったことですが，Linuxの生みの親・LinusもFedora使っているようです![Linus uses Fedora](/images/fedora-macbook/linus.webp)*From [Linus Tech Tips on YouTube](https://youtu.be/mfv0V1SxbNA?t=2722)*
    - "チョットデキル"あの人です．この写真の頃と比べると，だいぶお年を召されたよね．![ちょっとできる](/images/fedora-macbook/chotto.webp)*チョットデキル!*


## 【本題2】Fedora Linuxをデュアルブートする
macOSを使う可能性があるのでmacOSは残しつつ，FedoraをMacBookでデュアルブートしていきます．後述しますが，実際にFinal Cut ProやiTunes (Music)が必要な場面が出てきてmacOSも使いました，しかし，普段macOSの出番は殆どありません．

### 必要な物
- **有線インターネット**（もしくはAndroidか，携帯通信容量に余裕のあるiPhone）
  - おじいちゃんが持っていた理由がわかった，有線結局最強の安定性
  - 家帰ったらルーターから有線を引けるようにしよう
- **USB** (インストールメディア用)
  - 私は無かったのでSDカードで代用しました..
  - しかしSDカードは比較的衝撃に弱かったりするそうなのでおすすめしません
- **パソコンオタク**（"Geek"; もし近くにいればきっと助けてくれるでしょう）
  - 「おまけ」で後述


### 手順
本手順は，作業の全体像を示すために概略を掲載したものです．
詳しい手順については，[こちらの動画](https://www.youtube.com/watch?v=p4lu-_6nY6Q)などを参照してください．

1. macOSの掃除・バックアップ
    - Fedoraをインストールする容量を確保するために，macOSのストレージ内を整理し，消えてほしくないファイルはすべて外部に書き出します．
    - 私は外部ストレージが無かったので友人のUSBを借りました．
    - 理想的には，macOSの[Time Machine](https://support.apple.com/ja-jp/104984)バックアップをすると，いざというときにも復元できてよいでしょう．
    - 容量を削る際には，[GrandPerspective](https://grandperspectiv.sourceforge.net/)というフリーソフトがディスクが何に使われているのかを見る上で役に立ちます． [![GrandPerspective](https://grandperspectiv.sourceforge.net/ScreenShots/3_4-FoldersBujumbura.png =500x)*各ファイルの容量が面積に対応して表示されます*](https://grandperspectiv.sourceforge.net/)
    - 私の場合は，再インストール可能なXCodeやFinal Cut Pro関連の重たいファイルを消し，写真や動画，音楽を外に出しました．
    - 整理していたら，おじいちゃんが撮影した幼少期の私の動画が出てきました，ちゃんとMacの中に取っておいてくれたのだと思い胸が熱くなりました．![パンタグラフに見入る](/images/fedora-macbook/pantograph.webp =500x)*鉄道博物館でパンタグラフの動きに刮目する斎少年* ![治・講談](/images/fedora-macbook/osamu.webp =500x)*祖父が講談をやっている動画なんかも見つかりました*
1. パーティションを作成
    - macOSに標準搭載されているディスクユーティリティでFedora用の容量を空けていきます．
    - [こちらの記事](https://alex.dzyoba.com/blog/macbook-air-linux/#:~:text=Make%20partition%20for%20Linux)なんかに該当の作業が紹介されてございます．
    - この工程は非常に繊細で，おそらく失敗するとmacOSが起動しなくなり初期化が必要になるなど注意を要する工程です．ご自身でよく確認なさって，さらにバックアップなどで壊れても大丈夫な予防線を十分張った上で，「えいやっ」としてください（えいやっ，は良くないけど私はそうしました...背水の陣）．![ディスクユーティリティ](/images/fedora-macbook/partition.webp =500x)*作業中の写真; この後Fedoraの容量は300GBまで増やしました*![最終的なパーティション](/images/fedora-macbook/partition-latest.webp =500x)*最終的なパーティション分割はこの通りで，macOS 200GB, Fedora 300GBになっています*
1. インストールメディアを作成
    - USB（私はSDカード）にFedoraのOS Imageを書き込みます
    - 私は[Fedoraの公式サイト](https://fedoraproject.org/ja/workstation/download/)から.isoをダウンロードし，[Balena　Etcher](https://etcher.balena.io/)でSDカードに書き込みました．
    - Fedoraだけをインストールするなら，[Fedora Media Writer](https://docs.fedoraproject.org/en-US/fedora/latest/preparing-boot-media/#_fedora_media_writer)というより簡単な方法が提供されています．
1. USBから起動し，2で作成したパーティションにFedoraをインストールします
  - 間違えてmacOSの方を消さないでください．
1. 晴れて，MacBookを`alt`キーを押しながら起動すると，Fedoraが選べるようになります！
  - デフォルトの起動ディスクは，[macOSのシステム環境設定](https://support.apple.com/ja-jp/guide/mac-help/mchlp1034/mac)から選べます． 


## 【本題3】Fedoraのセットアップ
- ドライバ
    - Wi-Fi: Broadcom-wl
        - 友達のiPhone借りた
        - Windowsを同期していたおじいちゃんがThunderbolt-Ethernetを持っていたのはこういうときのためだったんだな
        - sudo dnf update
        - sudo dnf install broadcom-wl
    - Camera: FaceTime HDのドライバ
- 日本語入力: Anthyの設定から"hangul"と"hangul_hanji"を適切にかな英数キーに割り当てる
- Tweaks: Caps LockをCtrlとしてつかう
  - 細かいキー設定はkeyd: 別記事
- https://github.com/ubuntu/gnome-shell-extension-appindicator を入れな！
- ソフトウェア
	- Mailer: Thunderbird
	- Browser: Zen (Firefox)
	- Development: Zed, JetBrains; Linux-Firstな分野, 圧倒的な優位性
	- Bitwarden
	- 画像: Inkskape, GIMP; 慣れたら使える
	- 動画: KDENLive; 慣れたら使える
	- PDF: Document Viewer, ページなら並び替えなどは限定的だがまあ十分
	- Office: ほとんど使えないが他人から送られてきたものを開く用途ではLibreOfficeで十分．
		- ってか世の中の人はなぜLibreOfficeを使わないのか？OSに同梱されてないから，とかならクソ商法過ぎるだろ．
		- GUIで操作できる範囲を広げて，Linux by Defaultを特にパソコンをさわり立ての子どもなどに広めていかなければ．
		- ChromeOSは好例だが依然としてGoogleのベンダーロックイン
	- LocalSend: 同じWiFiに接続しなくてはいけないという制限がありながら，ほとんどのケースでこれで事足りる
  	- AirDropの一番の使いどころは
  - FreeFileSync
	- この辺のソフトウェアには寄付せなあかんかも知れん
	- Flatpakは.appみたいなもの．たまに権限で詰まる


- 参考になったブログ
	  - 誰かがMacBook Air 12だかでFedoraをインストールしているブログが参考になった
	  - dnfでOS含め一括アップデートしてから

https://www.schabell.org/2025/01/installing-fedora-41-on-macbook-pro-13-inch-late-2011.html
https://www.cyberciti.biz/faq/fedora-linux-install-broadcom-wl-sta-wireless-driver-for-bcm43228/
https://www.thetestspecimen.com/posts/broadcom-wifi-modules-fedora/
https://alex.dzyoba.com/blog/macbook-air-linux/


### おまけ: Computer Hacker - Ian
![computer hacker ian](/images/fedora-macbook/munchausen-by-proxy.webp)*Munchausen By Proxy - From the Movie "Yes Man" (2008)*

![](/images/fedora-macbook/cardboard-laptop.webp =500x)
![](/images/fedora-macbook/hacking-laundry-machine.webp =500x)

- インターネットつながらないって言ったら色々いじってくれた
- ダンボール
- 洗濯機改造
- https://www.gentoo.org/ 彼はGentoo Linuxを使っているようです

![Gentoo is not easy](/images/fedora-macbook/gentoo.webp =500x)*レポート提出しようと思ったらFireFoxをアンインストールしてしまい，再インストールとコンパイルに30時間かかったらしい．全然LOLじゃなさすぎる．*
```sh
(base) itsukikigoshi@fedora:~$ sudo stat / 
  Fichier : /
   Taille : 190       	Blocs : 0          Blocs d'E/S : 4096   répertoire
Périphérique : 0/37	Inœud : 256         Liens : 1
Accès : (0555/dr-xr-xr-x)  UID : (    0/    root)   GID : (    0/    root)
Contexte : system_u:object_r:root_t:s0
 Accès : 2026-05-16 00:25:47.727713744 -0700
Modif. : 2026-05-09 18:50:02.739655681 -0700
Changt : 2026-05-09 18:50:02.739655681 -0700
  Créé : 2026-01-31 01:26:32.065996572 -0800
```
使い始めたのは2026/1/31だとよ


## Fedoraにしてみて...
### よかったこと
- 開発体験の向上
- OSのバージョンによる制約を受けない
### よくないところ
- WiFiの接続がよく切れる
	- 住んでいる寮のWiFiが不安定なのもその理由の一つだろうが，どのWiFiにも接続詞なくなって，再起動するまで治らないことがある．
	- 個人的な仮説としては，寮のWiFi接続が切れるとFedora側でどうしていいか分からなくなって結果的にどこにも接続できなくなるんじゃないかと思う
  - FedoraのアップデートでWiFiが使えなくなる事が一度あった．本来自動でビルドされるはずのwlというWiFiドライバがビルドされないかまだアップデートされていないかで使えない
 	- そもそもlinuxを使うならbroadcomでなくIntel製
- 日本語入力
		- IBUS-Anthyが馬鹿（文の途中から打つとデタラメになる）, ATOKが恋しい
		- オフライン辞書 (Mac辞書, 物書堂; 国語, 英和)が使いたい
- 標準アプリが（これはFedoraではなくGNOME）限定的
  - Apple Musicに代わる良い音楽プレイヤー (編集, iPodとの同期)が無い (Rhythmboxはカバーが出ない, GNOME Musicも初回起動に時間がかかる，同じアルバムなのに別物と認識)
	- Apple Calendarに代わる良いカレンダー (タイムゾーンの変更): GNOME Calendar
  - いかにmacOS標準アプリがよくできているかを実感できる．ここはプロプライエタリさまさま
- 使えないアプリもまあまあある
  - ドライブ系のローカル連携: 基本Dropboxしか使えない．iCloud, OneDrive, Google Driveはローカル同期用の公式アプリが存在しない（本当に？）
  - Evernote -> Obsidian+Dropboxに乗り換えた（iPhoneと同期できない，やはりAndroid?でもGoogle好きじゃない）
- Apple製品は使えない（そこまで問題ではない）
  - iPodの更新: macOSを立ち上げる
  - iMessage: iPhoneでやる（意外と困らない）
    - RCSが普及すれば，よりiMessageである理由はなくなると思う．普及しろ～
  - FaceTime: iPhoneでやる（意外と困らない）
  - iCloud写真（iPhone写真のバックアップは日本帰ったらやらなきゃ）
  - Final Cut Pro (GPUの扱い方はやはりmacOSの方がうまそう？)

## コラム: 次に買うなら?
私もLet's Noteのゴツい見た目と，ソニー/NECと並んで数少ない日本メーカーである^[富士通のPC事業子会社である富士通クライアントコンピューティングはレノボに買収されたし...]点が好きなので，次に買うならFedora on Let's Note?
- Lenovo
- Framework（アメリカではよく観測した）
![](/images/fedora-macbook/framework.webp)

## Linuxの世界
- オープンソースカルチャーとの強い結びつき
  - Wikipedia
  - Internet ArchiveにもLinuxロゴがあったな
- カルチャー
  - "Linuxちょっとできる"
  - Linuxのミームたち
- アプリストアにフリーソフトウェアが充実しているのは，コミュニティとともに，参入障壁が低いこと（Apple Developers Program）
  - せっかく写真ビューワアプリApollo Oneを買ったのに...
- Arch Wiki
- 源流志向, できるだけ源流をたどる(source of truth)
  - 大抵英語で辛いけど，Docsを読むのがかなり利く．Tech Blogは入り口に，最終的に参照するのは公式ドキュメント
- CLIに抵抗はだいぶなくなった
  - なぜCLIはコマンドを知らないと何もできないんだろう

これから何するの？

---
## メモ

