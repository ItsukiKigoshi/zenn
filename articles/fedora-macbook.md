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

## 【概要】
旧式のMacBook Pro (Retina, 13-inch, Mid 2014) を現役で使用している私は，年初（2026年1月）に開発環境としてのmacOS 11 (Big Sur)に限界を感じ，Linux導入に踏み切りました．
ここでは10年以上前に発売されたMacBook ProにFedora Linuxをデュアルブートした手順と，Linux移行に係る所感とAppleやLinuxへの想いを綴ります．

![日常に溶けるFedora](/images/fedora-macbook/fedora-daily-life.webp)*日常に溶けるFedora*

## 【本題1】動機

私は，10年以上前に発売されたMacBook Pro ([Retina, 13-inch, Mid 2014](https://support.apple.com/ja-jp/111942)) を主たるコンピュータとして使用しています（まさかこんなに長く使うことになるなんて！）．
このMacBook Proは，2018年に亡くなった[祖父](https://ja.wikipedia.org/wiki/木越治)の形見として，彼の息子になる私の父親から譲り受けました．それ以来ですから，私が中学生だった2018年からかれこれ8年近く使用していて，今私は大学生ですがまだ現役で活躍しています．
しかし，同MacBook Proへインストール可能な最新OSの[macOS Big Sur](https://ja.wikipedia.org/wiki/MacOS_Big_Sur) (v11; 2020年)は私の趣味のWeb開発に関連する最新の開発環境を導入することは殆ど不可能です．高校生 (2022年あたり)からNode.js v18までしか動作せず，当時のLTSである20系を使えなかった記憶があります^[過去に関わっていたプロジェクトではNode v16を使っているので，ひょっとしたらv18すら動かなかったのかもしれない ([GitHub](https://github.com/hibiya-itchief/2024-quaint-app/blob/d0f200655489b32d398071594d55316d2a31afc8/.github/workflows/ci.yml#L22))]^[少なくともNode.js v20のサポートはmacOS 12以上でありそうです ([Github Discussion](https://github.com/nodejs/node/issues/47067#issuecomment-1474810592))]．

### コラム: 技術的な限界の詳細
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

## 記録: macOSで開発環境を整えるための試行錯誤

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

具体的には，学生認証で私が交換留学生として滞在しているUniversity of California, Berkeleyのアカウント (`itsuki [at] berkeley.edu`)を使おうとしたところ，私みたいなNon-degree studentは受け入れないようになっているのか，ただの`berkeley.edu`ではなく`*.berkeley.edu`という，どこかの学科に属していることが求められるドメインでないと登録不可でした．

私が所属する国際基督教大学のドメイン（`icu.ac.jp`）も，私が日本からアクセスしていない，という理由ではじかれました．

紛れもなく私は学生であるのに，その証明がこんなに大変だなんて，ちょっとしたアイデンティティクライシスを覚えました．

### Linuxのデュアルブート [採用]

ここまで試して，私に残された選択肢は殆どこれ1つになっていました．データが飛ぶリスクがあったので怖かったのですが，腹を括ってデュアルブートに踏みきることにしました．
そもそも，アクティブなサポートが殆ど切れているようなmacOSはセキュリティの観点からもあまり褒められたものではありません．
macOSから離れる機会をくれてありがとう．

---

### コラム: 厳格なWindows家系に生まれた私のAppleに対する想い

#### 家族はみーんなWindows.
私は，母方の祖父母がWindows 95だか98のパソコン教室を自宅で開いていたり，父方の祖父は昔からMS-DOSを使っていて，父親もMS-DOSを子供の頃に習ったという厳格なWindows家系に生まれたました．

父方の祖父は私が今使っているMacBookでWindows on Parallel Desktopを使っていました．支給されたのがMacBookだったからそれを使っていただけという理由らしい．多分一番使っていたのはワープロソフト[一太郎](https://www.justsystems.com/jp/products/ichitaro/) (Windowsでしか動作しない)だし．わけわかめ！たしかその祖父は生前にmacOSのライブかな漢字変換が嫌いとか，Macに対して良い事は言っていなかったような...

私の父親も，[Let's Note](https://panasonic.jp/cns/pc/products/lineup/)/[VAIO](https://vaio.com/)/Surfaceを使っていて，[秀丸エディタ](https://hide.maruo.co.jp/software/hidemaru.html)（Windows専用のフリーエディタ）でメモとか研究メモの執筆をしています．

うちの母親もバックオフィス系の仕事をしているので，自宅でHPを使っているし．

ほんまに誰一人として非Windowsユーザが家族にいない！

![2021年のMacBook](/images/fedora-macbook/mac-in-room.webp =300x)*Macと，横に見える親のSurface (2021)*

#### Macへの羨望
私が生まれ育った山口県山口市には，[山口情報芸術センター](https://www.ycam.jp/)（YCAM）という，最高にクールなメディアアートの開発・展示拠点があります．
https://www.ycam.jp/
今思い返しても，あんな田舎（ちょっと失礼）に世界からメディアアーティストがやってきたり（時に故・[坂本龍一](https://www.ycam.jp/archive/profile/ryuichi-sakamoto/)，[大友良英](https://www.ycam.jp/archive/profile/yoshihide-otomo/)，[ダムタイプ](https://www.ycam.jp/archive/profile/dumb-type/)），常駐の開発集団・[インターラボ](https://www.ycam.jp/aboutus/interlab/)のクールなみなさんが子どもだった私と遊んでくれたりしたあの環境は，紛れもなく山口市の財産だと思います．ありがとうYCAM．

私の趣味の現代アート鑑賞も，プログラミング（YCAMにはArduinoが転がっていた）やオープンカルチャー（作品は積極的にCreative Commonsライセンスで頒布されていた (例: [gonzoCam](https://www.ycam.jp/archive/software-hardware/gonzocam/))）への興味も間違いなくYCAMに根ざしています．

まあ小学生当時の私はコンピュータというより[レゴブロックのこまどり撮影](https://youtube.com/playlist?list=PLVxEAWkQVBq7ZLbCohJz4OtNNkQ6UMr0c)にはまっていたので，全然コンピュータ関連に明るかった訳ではなかったですが．そういえばYouTubeへ動画をアップロードする方法を小学3-4年生の私に教えてくれたのも[深澤孝史](https://www.ycam.jp/archive/profile/takafumi-fukasawa/)さんという美術家の方でした．失礼極まりなかったであろう世間知らず・木越斎少年のガキに色々なことを教えてくれたことには感謝しかありません．今も元気にしているかしら．最後に会ったのは確かコロナ禍が始まってすぐの2020年で，Zoom越しの対面でした．

#### アートとMac

さて，脱線が過ぎましたが，彼らが使っていたのが，他ならぬMacintoshだったのです．メディアアートに限らずMacユーザが多いのはアーティストあるあるなのでしょう．当時（2014年）のMacは，今よりも一層「洗練された人が使っているもの」という印象が強かったと思います．山口市という田舎に住んでいたのできっと余計に．りんごマークが光るMacBook Proはもちろん，Mac Pro 
（2013; "ゴミ箱"）を見たときなんて感動しました．
[![ゴミ箱Mac Pro](https://upload.wikimedia.org/wikipedia/commons/d/d6/New_Mac_Pro_%2812093123884%29.jpg?utm_source=commons.wikimedia.org&utm_campaign=index&utm_content=original  =250x)
*Paul Hudson from United Kingdom, CC BY 2.0, via Wikimedia Commons*
](https://commons.wikimedia.org/wiki/File:New_Mac_Pro_(12093123884).jpg)

こんな昔語りができるほど，2004年生まれの私も歳を取りました．
そういえば小さい頃に通っていた[英語教室](http://redrobini.com/)の室長先生もMacが好きで，MacBook ProやiMac，それに発売されたばかりのApple Watchを使っていました．
まだスキューモーフィズムの面影漂う[OS X Mavericks](https://ja.wikipedia.org/wiki/OS_X_Mavericks)でしたね（"OS X"！なんて懐かしい響き！！）．そういえば室長の彼（"katsu-sensei"）は当時New York旅行でタイムズスクエアに行ったときの写真を見せてくれましたが，その約10年後に私も初めてそこを訪れるわけです．あの英語教室に行っていなかったら私は国際基督教大学に通っていないかもしれないし，UC Berkeleyに留学していなかったかもしれません，ふーん，振り返って見ると色々なものがつながっているのです．[Connecting the Dots](https://youtu.be/UF8uR6Z6KLc)^[こんなに喋って，私はただのApple Fanではありませんか！否定は出来ないけど安直ですみません]^[日本語字幕が関西弁でふざけていますね...今気づきました.]...ってか私の中を流れるこのコンピュータへの関心から今年友達が通うスタンフォードにも訪れたわけで，そこでSteve Jobsは20年前に講演をしたわけで．つーか今日ちょうどBerkeleyの卒業式で，ガウンを着た人がいっぱいいるカフェでこれを書いています．スタンフォードのライバルのBerkeleyで．ちょっとこじつけが過ぎますね．


#### iMacを買ってもらった

[![iMac Retina](https://upload.wikimedia.org/wikipedia/commons/e/e6/IMac_vector.svg?utm_source=commons.wikimedia.org&utm_campaign=index&utm_content=original =500x)*Rafael Fernandez, CC BY-SA 3.0, via Wikimedia Commons*](https://commons.wikimedia.org/wiki/File:IMac_vector.svg)

小学4年生の木越少年は当時，家にあった4万円の激安NECパソコン（Windows 7）にインストールした[Windows ムービーメーカー](https://ja.wikipedia.org/wiki/Windows_ムービーメーカー)で，図書館から借りたハンズオン本を片手に動画編集を覚えました．しかし，同ソフトウェアの機能は限定的で，たしか1クリップあたりに細かい継続時間指定が出来ないなど，限界も感じ始めていました．
私の父親は子どもの潜在的なクリエイティビティに対する投資は惜しまない方針だったようで，iMovieが標準搭載されているiMacを近くのヤマダ電機だかビックカメラで一緒に買ってくれました．
その後，そのiMacはMacBook Proとともに外付けDVDドライブを指したら電源系統のトラブルで（多分私が適当な外部電源をDVDドライブに差したのでショートした）起動しなくなり，数万円で修理したものの8GBのメモリでの作業は辛く，結局私が高校生の頃だったかPCリサイクルされました．ユニファイドボディは美しいですが，「ディスプレイだけは再利用しよう」みたいことがやりにくいですよね．それはまた別のお話．

#### "Yosemite"

ところで，当時の最新OSはOS X Yosemiteで，子どもの私はその美しいデザインと[その発表方法](https://youtu.be/w87fOAG8fjk?t=812)に感動しました．多分人生で初めて見たWWDC Keynote (WWDC 2014)です．Craig Federighiの冗談と美しいデザイン哲学のミックスは今見ても美しい．Yosemiteは10年以上前だけれど，今見ても見劣りがしないどころか，Liquid Glassよりも洗練されているようにさえ思えます．思い出補正マシマシですが．

脱線の脱線ですがBerkeleyに来たからにはYosemite国立公園に行きました．小4の私はカリフォルニア州がどこにあるかも知らなかったわけなのですが，10年の時を経た聖地巡礼です．しかも，大学生になって始めたクライミングの聖地でもあるという，2つの意味での聖地を同時に巡るわけですね．

![El Capitan](/images/fedora-macbook/elcap.webp =400x)
*Yosemiteの次にOS Xの名称になった，ヨセミテ国立公園の象徴: エル・キャピタン岩．私の滞在中には[ユージ・ヒラヤマ](https://yuji-hirayama.com/)がこの壁を登っていたらしい^[実は私が高校生の頃に参加した森美術館のワークショップで彼の娘さんとダンスしており，彼からクライミング講習も受けたことがあることを，ヨセミテに行った後思い出しました．縁過ぎる．[![森美術館のワークショップでユージ・ヒラヤマのジムを訪れたときの写真](https://live.staticflickr.com/65535/51756727258_f07d7ca887.jpg)](https://flic.kr/p/2mRyKQw)*©Mori Art Museum. All Rights reserved. Provided thru [Flickr](https://flic.kr/p/2mRyKQw). Another Energy-Related Community Engagement Program “Art Camp for under 22, Vol. 7 Human Begin: What Are We Doing Tomorrow?” Session #2: Saturday, July 10, 2021 Venue: “Climb Park Base Camp” (b-camp.jp/) in Iruma City, Saitama Prefecture Photo: Tayama Tatsuyuki*]．映画『[フリーソロ](https://ja.wikipedia.org/wiki/フリーソロ)』でアレックスオノルドが綱なしで登ったことも有名ですね．一枚岩の花崗岩としては世界最大であると言われています．*

![壁を登るクラブメイト](/images/fedora-macbook/climbing.webp =400x)
*写真に写る彼女は，（リアルライフでも）パートナーである少し上にいる彼と一緒にフリークライミングしていました．一緒にトラッドクライミングできるカップルなんて素敵過ぎる．日本帰国後の私の1つの目標はフリークライミングの技術を身につけることかな．*

#### 他にもApple...
小学生の頃に買ってもらったiPod nanoも，人生で初めて買った携帯電話のiPhone 12 miniもまだ現役．両親が友人の結婚式でもらったか何かの初代iPod Shuffle（容量1GB!）も以前は使っていました．
![Apple Decices Still Workin'](/images/fedora-macbook/apples.webp =250x)
*iPod nano (6th Gen), MacBook Pro, iPhone 12 mini*


#### MacBook Pro (Retina, 13-inch, Mid 2014)の魅力
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

## 【本題2】なぜFedora Linux?
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


## 【本題3】Fedora Linuxをデュアルブートする
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


## 【本題4】Fedoraのセットアップ
これでFedoraが起動するようになりました！しかし，これだけでは日常使いができません．最大の壁は①WiFi接続と②日本語入力です．ついでに，カメラのセットアップもしておくべきでしょう．

ここで必要になってくるのが有線インターネットです．この時代のMacBookにはBroadcom製のWi-Fiチップが搭載されており，それを使った無線通信には，専用Wi-Fiドライバをインストールする必要があります．というのも，BroadcomのWi-Fiドライバはプロプライエタリである（オープンソフトウェアでない）ため，Linuxに同梱されません．

Ethernet-Thunderbolt^[おじいちゃんがこのMacBookと一緒に持っていて，何に使うんだろうと思っていましたが，Linuxのインストールに必要だったのですね！]でEthernetにつなぐか，Wi-FiにつながったAndroid，または携帯通信容量に余裕のあるiPhone^[Androidと異なり，iPhoneは自分が接続中のWi-Fiを別のデバイスに中継することができません．よって，iPhoneをこの作業に使う場合は携帯通信を使うことになります．]をUSBでMacと繋ぎます．私は容量無制限プランの友達のiPhoneを借りました．初めは無制限であると知らなかったので，最初のインストール時に数GBまで服欄でどうしようかと思いました．Ethernetか，Androidを使いましょう．Linuxを使っていると何かとAndroidの方が都合が良い時がありますが^[他には，ObsidianのDropbox経由の同期とか]，そのうちの1つです．イヤホンジャック返して．

### broadcom-wl: Wi-Fiセットアップ
ここからの工程は，私がやったときと同じく[こちらのブログ](https://www.schabell.org/2025/01/installing-fedora-41-on-macbook-pro-13-inch-late-2011.html#:~:text=Updating%20the%20installation)を参照しています．
インターネットにつながったら，まずは既存のライブラリを一括アップデートします．

```sh
# 一括アップデート
sudo dnf --refresh update
# broadcomのドライバをインストール
sudo dnf install -y broadcom-wl
# なんやわからんけどそれをビルド
sudo akmods
```

これで
```sh
$ lsmod | grep wl

wl                   6529024  0
cfg80211             1601536  1 wl
```
とすればインストールされたwlモジュールが見つかるはずです！これでネットワークメニューにWi-Fiが表示されませんか？もし見つからない場合は再起動してみてください．

### FaceTime HDカメラ
[こちらのディスカッション](https://discussion.fedoraproject.org/t/mulderje-intel-mac-rpms/130045)の通り，以下を実行してみてください．
```sh
$ sudo dnf copr enable mulderje/facetimehd-kmod 
$ sudo dnf install facetimehd-kmod
```

### 日本語入力
- 私は初期設定でIBUS-[Anthy](https://github.com/fujiwarat/anthy-unicode)を選びました，
- Macの「かな」「英数」キーをその通りに当てはめるには一工夫必要で，Anthy設定内のキー設定で`latin_mode`に`Hangul_Hanja`（「英数」キーに割り当てられた名前）, `hiragana_mode`に`Hangul`（「かな」キーに割り当てられた名前）を設定します．![Anthyで英数・かなキーを設定](/images/fedora-macbook/anthy.webp)
  - どのキーかを判定するのにものすごく時間がかかりました．みなさんはこれで簡単に設定できる事を祈ります．
- Anthyのかな漢字変換はお世辞にも賢いと言えません．文の途中から打ったときに文節を区切るのが苦手で，あと「そんな熟語いつ使うん？」みたいなんがしょっちゅうでてきます．こればかりは[ATOK](https://www.atok.com/)が恋しい．
  - 辞書と構文解析エンジンは今の時代に併せて改善の余地がありそうです．
  - Anthy初代が未踏発なら，新しいプロジェクトも未踏としてプロポーザル出せるかしら．
  - IBUS-SKKも試しましたが，`l`で英数モードにする仕様が変更できず，文章中に大量の"l"が登場して耐えられず辞めました．

ここまでできれば大体パソコンとして使えるようになります！

### キー設定
- [GNOME Tweaks](https://wiki.gnome.org/Apps/Tweaks)をインストールすることで， Caps LockをCtrlとして使うなどの簡単なキーマップ変更が出来ます
- さらに細かいキー設定には[keyd](https://github.com/rvaiya/keyd)がおすすめです．
- 筆者は大西配列にして使っています．
https://zenn.dev/itsukikigoshi/articles/d550cd8fe41fcd

### その他
- DropboxやJetBrains Toolboxなど，トップバーに表示されるインジケータの表示には，[こちらのGNOME Extension](https://extensions.gnome.org/extension/615/appindicator-support/)を入れてください！困難最初からあってもいいと思うですが，どうなってまんねやろなあ．

### メモ: いつからFedora?
Fedoraを使い始めたのは2026/1/31のようです．
```sh
(base) itsukikigoshi@fedora:~$ sudo stat / 
# ...
Créé : 2026-01-31 01:26:32.065996572 -0800
```


## コラム: ソフトウェア
使っているソフトウェアを紹介します．macOSの頃は大抵Apple標準で済んでいましたが，GNOMEは標準だと機能が物足りない場合があります．

- Mailer: [Thunderbird](https://www.thunderbird.net/fr/)
  - 言わずと知れた．
- Browser: [Zen](https://zen-browser.app/)
  - Arcみたいな新興ブラウザで，Gecko/FireFoxベースです．モダンなブラウザが欲しいけどChromiumベースに抵抗感がある私にピッタリです．
- Development: [Zed](https://zed.dev/), [PyCharm](https://www.jetbrains.com/pycharm/) (重いけどJupyter Notebookが使いやすい)
  - 開発はLinux-Firstな分野で，Linuxに圧倒的な優位性があります．XCcde以外は基本全部動く．
  - 逆に，最新のmacOSがないとビルドできないXcodeって悪目立ちしすぎでは？もうちょっとインクルーシブになってください．
- パスワード管理: [Bitwarden](https://bitwarden.com/)
- 画像: [Inkskcpe](https://inkscape.org/), [GIMP](https://www.gimp.org/)
  - 老舗たちです．癖はあるけど慣れたら使えます．
  - シンプルさが欲しいならブラウザでFigmaやCanvaも使えますね．
- 動編集画: [KdenLive](https://kdenlive.org/)
  - 慣れたら使えます
  - Flatpak版じゃない（RPM版など）とプロプライエタリなコーデック（.movなど）が読めない
  - 書き出し速度はmacOSでFinal Cut Proを使ったときより流石に劣ると思います．
- PDF: [Document Viewer](https://apps.gnome.org/fr/Papers/)（標準）
  - ページ並び替えなどは限定的だがまあ十分
  - たまに，ページを縮小しようとすると辺なページに飛ぶバグが地味に辛いです．
- Office: [LibreOffice](https://www.libreoffice.org/)
  - ほとんど使わないが，他人から送られてきたものを開く用途ではLibreOfficeで十分．
	- ってか世の中の人はなぜLibreOfficeを使わないのでしょうか？多分OSに同梱されてないからです．Microsoft Officeなどなくても殆どの用途はLibreOfficeで済む（しかも無料である）のに，普及していないのが悲しすぎます．
	- パソコンをさわり立ての子どもなどに，より簡単に使えるLinuxや自由ソフトウェアを広めていかなければ，子どもの世代まで不要なMS365に高額を払わせることになってしまいます...ここで負の連鎖は終わらせましょう．
- [LocalSend](https://localsend.org/): AirDorpの代替
  - 同じWiFiに接続しなくてはいけないという制限がありながら，ほとんどのケースでこれで事足りる
 	- AirDropの一番の使いどころは自分のPC↔iPhoneであることに気づいたので，その用途が満たせれば十分．
- [FreeFileSync](https://freefilesync.org/)
  - デジタルペーパやiPhoneのVLCとパソコンを同期できます．便利．
- この辺のソフトウェアには寄付せなあかんかも知れんです．
- Flatpakというアプリの配布形式はmacOSでいう.appみたいなもので，サンドボックス化と言ってOSの他の部分に侵食し内容に設計されています．たまに権限で詰まるので[Flatseal](https://flathub.org/en/apps/com.github.tchx84.Flatseal)などを使って解いてあげます．

## おまけ: Computer Hacker - Ian
![computer hacker ian](/images/fedora-macbook/munchausen-by-proxy.webp)*Munchausen By Proxy - From the Movie "Yes Man" (2008)*

私が交換留学しているUC Berkeleyで住んでいる寮（正確には[Coop](https://www.bsc.coop/housing/our-houses-apartments/cloyne-court)）でいつもパソコンを触っている細い男の子がLinuxに詳しいという話を聞いたので，Wi-Fiがつながらないと言ったら相談に乗ってくれました．

![自作のダンボールパソコン](/images/fedora-macbook/cardboard-laptop.webp =500x)*彼の部屋にあった自作の段ボールラップトップ*
彼は，アプリのインストール時に1からソフトウェアをビルドをする[Gentoo Linux](https://www.gentoo.org/)というディストリビューションを使っていて，部屋には自作の段ボールパソコンがあります．夏休みに開発を進めるらしい．
![Gentoo is not easy](/images/fedora-macbook/gentoo.webp =500x)*Ianはレポート提出前にFireFoxをアンインストールしてしまい，再インストールとコンパイルに30時間かかったことがあるらしい．全然LOLじゃなさすぎる．*

彼は，寮内の洗濯機を無料にしようとしてました．多分試みは上手くいっていないのだけれど，あそこを分解しようとするだけで充分crazy(褒めてる)．

![Fix Laundry Machine](/images/fedora-macbook/hacking-laundry-machine.webp =500x)*洗濯機を無料にするために改造．ちなみにこれとは関係なく来学期から無料になるらしい*

## 【本題5】Fedoraにしてみて...
### よかったこと
- 開発体験が向上しました．
  - wranglerもpnpmもnodeもbunも最高にサクサク動きます．
  - 開発が止まってた原因は，Mac自体の性能ではなくmacOSのせいでした．
### よくないところ
- WiFiの接続がよく切れる
	- 住んでいる寮のWiFiが不安定なのもその理由の一つだろうが，どのWiFiにも接続になくなって，再起動するまで治らないことがあります．
 	- そもそもLinuxを使うならbroadcomでなくIntel製やMediaTek製のワイヤレスチップが推奨されるようです．Broadcom-wlドライバはそもそもが非推奨なので切れても仕方ない．
- 日本語
		- かな漢字変換は，ATOKが恋しいです．ATOKで打っていて漢字変換について考える事なんてほとんど無かったのにい
		- オフライン辞書 (Mac辞書, 物書堂; 国語, 英和)が使いたい
      - Mac標準の辞書や物書堂のような引きやすい辞書が欲しいです．[GoldenDict](https://www.goldendict.org/)など，あるにはあるようですが辞書ファイルの追加方法が分かりません．
- 標準アプリが（これはFedoraではなくGNOME）限定的
  - Apple Musicに代わる良い音楽プレイヤー (編集, iPodとの同期)が無い
    - Rhythmboxはカバーアートが出ない
    - GNOME Musicも初回起動に時間がかかる，同じアルバムなのに別物と認識されるなど
	- Apple Calendarに代わる良いカレンダー
  	- GNOME Calendarはタイムゾーンの変更ができず困ります
  - いかにmacOS標準アプリがよくできているかを実感しました．ここはプロプライエタリさまさまです．
- 使えないアプリもまあまあある
  - 個人用ストレージサービスのローカル連携は，基本Dropboxしか使えません．iCloud, OneDrive, Google Driveはローカル同期用の公式アプリが存在しないようです．
  - iMessage: iPhoneでやれば意外と困りません
    - RCSが普及すれば，よりiMessageである理由はなくなると思います．普及しろ～
  - FaceTime: iPhoneでやれば意外と困らない
  - iCloud写真: iPhone写真のバックアップは日本帰ったらやらなきゃです
  - Final Cut Pro: GPUの扱い方はやはりmacOSの方がうまそうです


## コラム: 次にPCを買うなら?
- Fedora on [Let's Note](https://panasonic.jp/cns/pc/products/lineup/)
- Fedora on [Lenovo](https://www.lenovo.com/jp/ja/)
- Fedora on [Framework](https://frame.work/): 修理が容易なPC．日本未発売．
  - アメリカで実際に使っている学生を2例見かけました![カフェで観測したFramework](/images/fedora-macbook/framework.webp =300x)
- [MacBook Pro](https://www.apple.com/jp/macbook-pro/)

次にPCを買い換えるなら，現在のWindows PCはCopilotマークがおダサなので，それがないFrameworkかMacBook Proを買います．実はFrameworkもあまり安くない．

今年のWWDC (2026)で，macOSのLiquid Glassがもう少し落ち着いたデザインになる事を願います．

## Linuxの世界

macOSで開発が出来ないので代替手段を得るという消極的な理由でしたが，今（そしてアメリカ西海岸で）Linuxに出会えたことはよかったと思います．Internet Archiveを訪問した時もLinuxステッカーが貼ってあったし，この地域のフリーカルチャーとLinuxは密接に結びついてそうです．
Linuxを使ったことで初めて「プロプライエタリ」という言葉を口にしたし，フリーソフトウェアコミュニティの厚さも実感したし，コマンドラインに抵抗感がなくなりました．

このMacBook Proが壊れるまでは，ネジも付け直したりしながらFedoraと共に勉強や物作りに励みたいと思います．

## 結びに代えて: これから何するの？
2026年5月に交換留学を終えて日本に帰国し，ICUで卒業するミッションが控えているのですが，その先に宇宙科学も興味あるし登山をするので地学にも興味があるし，またモデリングなどの数値解析でコンピューティングが関わるような研究や仕事もしてみたいし，何をしていいか定まらず困っています．

### 2026年夏は槍ヶ岳に遊びにきてね
さしあたり今夏は長野県は北アルプス・槍ヶ岳の山小屋で勤務予定なので，お近くをご通行の際はご一報ください．
山の上で働きながら，たまに縦走登山する夏にします．
遊びに来てね．

https://www.enzanso.co.jp/hutte-ooyari

木越 斎 (2026/05/17; Provided with [CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.en))