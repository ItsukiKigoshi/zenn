---
title: Fedora LinuxをMacBook Pro (Mid 2014)でデュアルブートしよう
emoji: 👏
type: tech
topics:
  - fedora
  - Macbook
published: false
---
## 概要
- MacBook Proが古くて開発に支障が...
- パソコンを買い直すのも面倒だし，Linux入れてみるか！（Geminiにそう提案された）
- 結果として大学もこれでいけた

## MacBook Pro (13 inch, Mid 2014)の魅力
- HDMI
- 光る
- SDカード
- MagSafe
- 今とそんなに重さが変わらない
- Intel (Linuxがブートしやすい)
  - Apple SiliconにはAsahi Linuxがひつようでちょっとめんどいっぽい

## なぜLinux?
- 私の使っているMacBook Pro Mid 2014で動く最新のmacOSである11 Big Surを使っていたが，12以降でないと動かない開発ツールが多くなった
  - Wranglerが動かない
  - 高校生時代（2022-3頃）から既にNode 18系までしか動かずNode 20系での開発がうまく行かなかった気がする
- セキュリティ懸念
```sh
dyld: Symbol not found: __ZNSt3__113basic_filebufIcNS_11char_traitsIcEEE4openEPKcj

Referenced from: /Users/user/.nvm/versions/node/v24.13.0/bin/node (which was built for Mac OS X 13.5)
```

```sh
pnpm dev # in Hono Project
# Cloudflare Workers runtime cannot run on the current version of macOS
```

## なぜFedora Linux?
- なんとなく以下の条件があった
	- 私の中ではなるべくセットアップの負担が少ない
	- クール
- Fedora
  - PureなGNOMEが綺麗
  - 開発環境として適している
  - Just works （面倒な設定無しでも結構動く）
    - Linusも使っている
- Nix (試した; 面倒; 日本語フォントが汚い), Ubuntu (デフォルトデスクトップが可愛くない), Zorin (試した; 有料版があるのが引かれないかも,だったらUbuntuで良くない？) 

## Fedora Linuxをデュアルブートする手順
- macOSを使う可能性があるので（実際に使った），Macは残す

- 必要な物: 
  - インターネット（Android, 携帯通信容量に余裕のあるiPhone, Ethernet）
    - おじいちゃんが持っていた理由がわかった，有線結局最強の安定性
    - 家帰ったらルーターから有線を引けるようにしよう
  - パソコンオタク（Geek）
    - インターネットつながらないって言ったら色々いじってくれた
    - ダンボール
    - 洗濯機改造
    - https://www.gentoo.org/ 彼はGentoo Linuxを使っているようです
    - Ian "He is a computer hucker"

https://www.youtube.com/watch?v=nqioOaFAfHw
https://www.youtube.com/watch?v=p4lu-_6nY6Q
1. バックアップ
1. パーティションを作成
  1. 確か800MBくらいの1領域余計に必要
1. インストールメディアを作成
  1. https://etcher.balena.io/ : Media Writerの方がいいかも
1. SDカードから起動
  1. 多分USBの方が良い（？）
1. Fedoraを作った容量にインストール
1. Fedoraを起動 (altキー)
- 参考になったブログ
	- 誰かがMacBook Air 12だかでFedoraをインストールしているブログが参考になった
	- dnfでOS含め一括アップデートしてから
- 安易に「AIすごい」とは言いたくないけど，Linuxの導入に関してはGoogle検索と，その元になったブログ投稿，Reddit Postsが役立ったと言わざるを得ない．

この際だから調べよう:
- [ ] FAT32などのファイルシステムの違い

## Fedoraのセットアップ
- [ ] どんな感じになったか積極的にスクリーンショットを載せよう

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

## Linuxの世界
- オープンソースカルチャーとの強い結びつき
  - Wikipedia
  - Internet ArchiveにもLinuxロゴがあったな
- カルチャー
  - "Linuxちょっとできる"
  - Linuxのミームたち
---
## メモ
https://www.schabell.org/2025/01/installing-fedora-41-on-macbook-pro-13-inch-late-2011.html
https://www.cyberciti.biz/faq/fedora-linux-install-broadcom-wl-sta-wireless-driver-for-bcm43228/
https://www.thetestspecimen.com/posts/broadcom-wifi-modules-fedora/
https://alex.dzyoba.com/blog/macbook-air-linux/
