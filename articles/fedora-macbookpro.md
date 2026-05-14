---
title: FedoraをMacBook Pro (13 inch, Mid 2014)でデュアルブート
emoji: 👏
type: tech
topics:
  - fedora
  - Macbook
published: false
---
- なぜLinux?
	- 私の使っているMacBook Pro Mid 2014で動く最新のmacOSである11 Big Surを使っていたが，12以降でないと動かない開発ツールが多くなった
	- 高校生時代（2022-3頃）から既にNode 18系までしか動かずNode 20系での開発がうまく行かなかった気がする
```sh
dyld: Symbol not found: __ZNSt3__113basic_filebufIcNS_11char_traitsIcEEE4openEPKcj

Referenced from: /Users/osamukigoshi/.nvm/versions/node/v24.13.0/bin/node (which was built for Mac OS X 13.5)

Expected in: /usr/lib/libc++.1.dylib


[1] 29870 abort node -v 
```
```sh
pnpm dev # in Hono Project
# Cloudflare Workers runtime cannot run on the current version of macOS
```


- なぜFedora?
	- なんとなく以下の条件があった
		- 私の中ではなるべくセットアップの負担が少なく，MacBook
- 参考になったブログ
	- 誰かがMacBook Air 12だかでFedoraをインストールしているブログが参考になった
	- dnfでOS含め一括アップデートしてから
- 安易に「AIすごい」とは言いたくないけど，Linuxの導入に関してはGoogle検索と，その元になったブログ投稿，Reddit Postsが役立ったと言わざるを得ない．

- https://etcher.balena.io/
- 必要な物: 
  - インターネット（Android, 携帯通信容量に余裕のあるiPhone, Ethernet）
    - おじいちゃん
  - パソコンオタク（Geek）
- Geminiから拾う

## よかったこと
## わるいこと
- WiFiの接続がよく切れる
	- 住んでいる寮のWiFiが不安定なのもその理由の一つだろうが，どのWiFiにも接続詞なくなって，再起動するまで治らないことがある．
		- 個人的な仮説としては，寮のWiFi接続が切れるとFedora側でどうしていいか分からなくなって結果的にどこにも接続できなくなるんじゃないかと思う
	- FedoraのアップデートでWiFiが使えなくなる事が一度あった．本来自動でビルドされるはずのwlというWiFiドライバがビルドされないかまだアップデートされていないかで使えない
		- そもそもlinuxを使うならbroadcomでなくIntel製

---
## メモ

- なぜ？
    - 開発できない
    - セキュリティ懸念
    - 結果として大学もこれでいけた
    - Nix (面倒; 日本語フォントが汚い), Ubuntu (デフォルトデスクトップが可愛くない), Zorin (有料版があるのが引かれないかも,だったらUbuntuで良くない？) 
    - Wranglerが動かない
- デュアルブート
    - パーティション作成 (確か800MBくらいの1領域余計に必要)
    - SDカードから起動
- ドライバ
    - Wi-Fi: Broadcom-wl
        - 友達のiPhone借りた
        - Windowsを同期していたおじいちゃんがThunderbolt-Ethernetを持っていたのはこういうときのためだったんだな
        - sudo dnf update
        - sudo dnf install broadcom-wl
    - Camera: FaceTime HDのドライバ
- Mailer: Evolution
- JetBrains Toolbox
- FireFox: BitWarden
- 日本語入力: Anthyの設定から"hangul"と"hangul_hanji"を適切にかな英数キーに割り当てる
- keyd: 別記事
- Tweaks: Caps LockをCtrlとしてつかう

- Macでしかできないこと
  - iPod更新

- Linuxの嫌なところ（日本語とWi-Fi）
	- 日本語環境
		- IBUS-Anthyが馬鹿（文の途中から打つとデタラメになる）, ATOKが恋しい
		- オフライン辞書 (Mac辞書, 物書堂; 国語, 英和)が使いたい
	- 良い音楽プレイヤー (編集, iPodとの同期)が無い (Rhythmbox, GNOME Music)
	- 良いカレンダー (タイムゾーンの変更): GNOME Calendar
	- Flatpakが結構権限関係で詰まる; でも環境は汚れない
	- いけるところ
		- Mailer: Thunderbird
		- Browser: Zen (Firefox)
		- Development: Zed, JetBrains; Linux-Firstな分野, 圧倒的な優位性
		- 画像: Inkskape, GIMP; 慣れたら使える
		- 動画: KDENLive; 慣れたら使える
		- PDF: Document Viewer, ページなら並び替えなどは限定的だがまあ十分
		- Office: ほとんど使えないが他人から送られてきたものを開く用途ではLibreOfficeで十分．
			- ってか世の中の人はなぜLibreOfficeを使わないのか？OSに同梱されてないから，とかならクソ商法過ぎるだろ．
			- GUIで操作できる範囲を広げて，Linux by Defaultを特にパソコンをさわり立ての子どもなどに広めていかなければ．
			- ChromeOSは好例だが依然としてGoogleのベンダーロックイン

https://www.schabell.org/2025/01/installing-fedora-41-on-macbook-pro-13-inch-late-2011.html
https://www.cyberciti.biz/faq/fedora-linux-install-broadcom-wl-sta-wireless-driver-for-bcm43228/
https://www.gentoo.org/彼はGentoo Linuxを使っているようです
https://www.thetestspecimen.com/posts/broadcom-wifi-modules-fedora/
https://alex.dzyoba.com/blog/macbook-air-linux/