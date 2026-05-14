---
title: GitのSSH署名/リモート接続にBitwarden SSHエージェントを使う
emoji: 💨
type: tech
topics:
  - Bitwarden
  - SSH
  - Git
  - GitLab
  - GitHub
published: false
---
## Bitwardenとは
Bitwardenはオープンソースのパスワードマネージャで，Bitwardenが用意したフリーミアムプランを使用すれば基本無料で使用できます．組織向けのプランも存在するようですがここでは個人利用を想定します．
https://bitwarden.com/

### 余談: Why Bitwarden?
筆者はmacOSからLinuxへ乗り換えるときにiCloud Keychainからパスワードマネージャを移行する必要に迫られ，無料でiOSとLinuxで使える点，そしてオープンソースである点にも惹かれてBitwardenを使い始めました．サービスには満足しています．一方で，2026年4月には[Bitwarden CLIへのサプライチェーン攻撃](https://community.bitwarden.com/t/bitwarden-statement-on-checkmarx-supply-chain-incident/96127)が発生しており，秘匿情報を扱う以上，読者の皆様がパスワードマネージャ移行を検討する際には慎重に比較なさることをおすすめします．とはいえ，私は今後もBitwardenを使用するつもりです．

## BitwardenのSSHエージェント
Bitwardenは，SSH鍵を同ソフトウェア内で管理する方法を2025年1月より提供しています．

https://bitwarden.com/help/ssh-agent/
同機能を使うことで，通常`ssh-keygen`で作成する時のように`~/.ssh`ディレクトリに直接秘密鍵が保存されることなく，Bitwardenが起動している間のみSSH鍵が使用できるようにできます．

これで，デバイスの盗難やハッキング時にも，Bitwardenさえロックされていれば秘密鍵を盗まれなくなる，はずです．少なくとも`~/.ssh`をコピーされることはないでしょう．
より厳格なセキュリティを求めるなら，外部へ秘密鍵が書き出し不可な[YubiKey](https://www.yubico.com/yubikey/)など外部セキュリティキーを使うべきなのでしょうか．俄然興味が湧きます．


## GitにおけるSSH
バージョン管理ツールであるGitは，デフォルトで[SSH（またはGPG）によるコミットへの署名](https://git-scm.com/book/en/v2/Git-Tools-Signing-Your-Work)を提供しています．また，[GitLab](https://docs.gitlab.com/user/ssh/)や[GitHub](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)などのGitホスティングサービスもリポジトリへのアクセス時にSSH接続を提供しています．

今回はこの

1. コミット署名用SSH鍵: コミットが本人によるものであることを証明
2. SSH接続用SSH鍵: GitLab/GitHubへの接続時の認証

をそれぞれBitwardenで管理します．

## BitwardenでのSSH鍵作成
ここで説明する内容は，[公式ヘルプ](https://bitwarden.com/help/ssh-agent)（英語）が詳しいです．併せてご参照ください．

まず，Bitwardenを開きます．私はFedoraのFlathub版を使っていますが，きっとあなたが[お使いのOSに対応したBitwardenクライアント](https://bitwarden.com/download/)が存在することでしょう．
![Bitwardenのスタート画面](/images/using-ssh-in-git-with-bitwarden/bitwarden-startup.webp)
次に，`+`ボタンを押して"SSH Key"を選択し，名前をつけてSaveすれば...
![Bitwarden SSH Key Generation](/images/using-ssh-in-git-with-bitwarden/bitwarden-ssh-key-generation.webp)
完成です！

私はGit署名，GitHub接続, GitLab接続をそれぞれ別の鍵で管理したいので，3つ作成しました．

## PATHを通す
この状態ではshellで鍵を見ようとして`ssh-add -L`しても，`Error connecting to agent: Connection refused`とエラーが出ます．
Shellに鍵を認識してもらうために，`nano ~/.bashrc` (または`nano ~/.zshrc`)して，
```text:~/.bashrc
# Flatpak版の場合
export SSH_AUTH_SOCK=/home/<あなたのユーザ名>/.var/app/com.bitwarden.desktop/data/.bitwarden-ssh-agent.sock
```
を書き加えます．
各OS/アプリの種類によりPATHが異なります．[公式ヘルプの該当箇所](https://bitwarden.com/help/ssh-agent/#configure-bitwarden-ssh-agent)を参照してください．

これで，`ssh-add -L`すればBitwardenで作成した鍵が見えることでしょう．

## Git署名に設定する
Gitのコミット署名にSSH鍵を使うために，まず署名の形式がSSHであることをGitに伝えます．
```sh
git config --global gpg.format ssh
```

あなたのSSHの公開鍵を事前にBitwardenからコピーしておき，
```sh
git config --global user.signingkey "<ssh-ed25519から始まるあなたの署名用SSH公開鍵>"
```
で公開鍵をGitに伝えます．

任意で，自動でGitコミットへの署名を有効にするために
```sh
git config --global commit.gpgsign true
```
します．

さらに，ローカルのGitにこの鍵を信頼させるために
```sh
# 信頼できる鍵一覧の作成
touch ~/.ssh/allowed_signers
```
でファイルを作成し，`nano ~/.ssh/allowed_signers`でその中に
```text
# 例: email@example.com ssh-ed25519 XXXXXXXX
<あなたがGit Commitに使用しているEmail> "<ssh-ed25519から始まるあなたの署名用SSH公開鍵>"
```
を書き込みます．あとは，
```sh
git config --global gpg.ssh.allowedSignersFile "$HOME/.ssh/allowed_signers"
```
で信頼できる鍵一覧をGitに伝えれば，ローカルのGitがあなたの鍵を信頼できるものと認識してくれます．

## GitLab，GitHubへの公開鍵登録
このままではGitLab/GitHub側はあなたのSSH鍵を知る由もありません．伝えてあげましょう．
それぞれのサイトにログインした状態で，
- GitLab: `User Settings > Access > SSH Keys`
- GitHub: `Setting > SSH and GPG Keys > New SSH Key`
で鍵を登録します．
GitHubは鍵追加時にその鍵が認証（`ssh git@github.com`）用かGit署名用かを選ぶ必要があります．私は異なる2つの鍵をそれぞれの用途に作成し追加しました．

![SSH Keys on GitLab](/images/using-ssh-in-git-with-bitwarden/gitlab-ssh-keys.webp)
*GitLabに登録されたSSH鍵*
![SSH Keys on GitHub](/images/using-ssh-in-git-with-bitwarden/github-ssh-keys.webp)
*GitHubに登録されたSSH鍵*

これで，`git clone`時などにこのSSH Keyを使うことが出来ます．

試しに接続してみると，
```sh
# GitLab.com
ssh -T git@gitlab.com
# Welcome to GitLab, @<あなたのユーザーネーム>!

# GitHub
ssh -T git@github.com
# Hi <あなたの名前>! You've successfully authenticated, but GitHub does not provide shell access.
```
のようにフレンドリーな返答が期待できます．やったね．

## Closing👋
これで，Bitwardenを用いたSSH接続並びにコミット署名の準備が完了しました．デフォルトでコミットに署名するように設定していれば，これからあなたの端末で作成されるコミットには自動で署名が付きます．

GitLab/GitHubに署名されたコミットをpushすると画像のように`verified`と，検証済みのコミット署名であることが表示されます．
![Verified Commits on GitLab](/images/using-ssh-in-git-with-bitwarden/signed-commits-on-gitlab.webp)
*GitLabに表示される署名済みマーク*

![Verified Commits on GitHub](/images/using-ssh-in-git-with-bitwarden/signed-commits-on-github.webp)
*GitHubに表示される署名済みマーク*

みなさまも快適な鍵生活を送られんことを願っています．
