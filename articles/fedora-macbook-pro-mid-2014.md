---
title: ""
emoji: "👏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---
- なぜLinux?
	- 私の使っているMacBook Pro Mid 2014で動く最新のmacOSである11 Big Surを使っていたが，12以降でないと動かない開発ツールが多くなった
	- 高校生時代（2022-3頃）から既にNode 18系までしか動かずNode 20系での開発がうまく行かなかった気がする
- なぜFedora?
	- なんとなく以下の条件があった
		- 私の中ではなるべくセットアップの負担が少なく，MacBook
- 参考になったブログ
	- 誰かがMacBook Air 12だかでFedoraをインストールしているブログが参考になった
	- dnfでOS含め一括アップデートしてから
- 安易に「AIすごい」とは言いたくないけど，Linuxの導入に関してはGoogle検索と，その元になったブログ投稿，Reddit Postsが約だったと言わざるを得ない．
- 

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
