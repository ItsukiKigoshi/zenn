---
title: 国際基督教大学の履修計画アプリをAstro + Cloudflare Workers + D1でつくりました
emoji: 🐦‍🔥
type: tech
topics:
  - Astro
  - Cloudflare
  - cloudflareworkers
  - d1
  - 大学
published: true
---
## "[What's This?](https://www.youtube.com/watch?v=o36k8upu3Ks)": 「ICUのじかんわり」
私が通う国際基督教大学（ICU; 東京・三鷹）の履修計画/時間割アプリをつくりました．

https://timetable.icu/

### できること
1. 全コースを検索
    - 大学が一般公開しているシラバスデータなどから構築された授業情報を表示・検索できます．
1. 時間割の表示・同期
    - ICU独自のスケジュールに対応した時間割が作れるほか，同時間帯に複数の授業を入れられるので迷っているコースらを吟味することが出来ます．
    - 大学アカウントでログインすると，デバイス間で最新の時間割を同期できます．
1. 日英両言語対応

以前にも[似たようなアプリ](https://itsukikigoshi.github.io/icu-catalogue/)をつくりましたが，当時データ保存は[localStorage](https://developer.mozilla.org/ja/docs/Web/API/Window/localStorage)での管理に留まり，また，ICU独自の[授業同士が食い込む時間割](https://www.icu.ac.jp/news/2406181000.html)の表示にも対応していませんでした．

今回製作したWebアプリケーションではコース検索とICUの独自スケジュール表示に加え，デバイス間同期などの機能も実装しましたのでその開発の全体像をお見せします．

本アプリのソースコードは[こちら](https://gitlab.com/ItsukiKigoshi/timetable.icu)で公開しており，どなたでも[問題を報告](https://gitlab.com/ItsukiKigoshi/timetable.icu)できます．主に学内者からのコントリビュートを想定しておりますが，それ以外の方でもお気づきの点がありましたらお知らせ願えますと幸いです．
## なぜ自作?
既に世の中に数多ある時間割アプリの自作を決意するに至った理由は以下に主に3つです．
1. **友人からのリクエスト**
	- 一番大きな理由&モチベーションです．
	- その友人は過去に私が作った前述の時間割アプリをいたく気に入ってくれていたようですが，そのまた友人に同アプリを薦めたところバグに遭遇しうまく使えなかったとのことでした．
	- 同アプリは大学1年次の2023年秋に製作して以降放置しており，サーバでのデータ管理なども検討しましたが何度も挫折しました（FastAPI, NestJS, Supabase, Prisma, Flutter など検討しては面倒だと言って諦めて...）．
	- なによりも現状アプリは動いている（+下記コラム1の通り既存プレイヤーが多くいる）ので新機能開発のモチベーションは低めでした．
	- しかし，この友人の一言がきっかけで（当時ちょうど別のことがやりたくてHono, 今回は未採用, + Cloudflare Pages + Workers + D1 +Better Authでプログラミングしていたのもあって）新しいアプリを作ることにしました．
2. **ICUの制度が他大学と大きく異なり，汎用品では代替が難しかった**
	- 一例として， 本アプリの訴求ポイントである「とりあえず気になるコースをすべて保存して後から絞る」，というユースケースが挙げられます．
		- 下記コラム1で言及した既存アプリのほとんどが，一コマあたり一授業しか設定できない仕様です．
		- 想像するに，他大学では必修科目が多くコマ選択の余地が少ないためにそのような機能が不要なのかもしれません．
		- 一方，ICUでは1年次の言語/体育+キリスト教概論以外に必修はないので多くのコースの中から自分の専攻に必要な科目を選び取る必要があります．
		- こうした機能は私とその友人を含め確かにニーズがありそうでした．
	- Google Spreadsheetを使う履修計画方法が大学公式のアカデミックサポートでも紹介されていますが，授業情報一覧からスケジュールに落とし込む作業はミスしやすく面倒で，機械にやらせた方がいいと思いました．
 3. **大規模言語モデル（LLM）の発達によって実装の心理的負担や学習障壁が小さくなった**
	 - LLMの発達によって，2023年秋に前アプリに取り組んだ時と比べて楽しく短い期間で開発出来そうでした．
	- 一方で，すべてをClaude Codeに任せるような"Vibe Coding"は.envをgit commitしてしまったり自身の理解が追いつかなくなる+お金がかかるので，IDE/エディタにはエージェントを入れず，大学が管理するGeminiで質問したりコード出力を行いました（大学が管理していれば安全か，という問題はさておき）．
		- 大規模言語モデルの使用がなければこのアプリが出来ていなかったであろうが，正直に言って，実装に関して自分でも不確かな部分があるのが心配です．
		- Securityに関する部分の不安はなるべく取り除いているが，著者の能力には限界がある．
		- 一方で，この構成のアプリケーションを（私程度の技量しかない）人間が，（ある程度の質の）大規模言語モデルなしで書いたとすればそちらの方がsecurity holeが多くなることは目に見えているので，必要なのは人間側のセキュリティに関する知識と実際のコードでその脆弱性を見つけられる審美眼だろう．精進します．
		- 環境変数やユーザデータ等の機密情報を除きLLMにかかれば脆弱性が一気に露見してしまうだろうという諦めも，積極的にコードを公開して修正を可能にしている理由です．
	- LLMの利用（モデルの学習/推論）で消費されるエネルギーの莫大さにもう少し自覚的である必要があるかも知れません．私は今回の開発中に使用した電力に見合うだけの活動が出来ているのでしょうか？
:::details コラム1: 既存の履修登録比較アプリの比較
| 名前 | [Timetable4ICU](https://www.timetable4icu.com/) | [ICUrriculum](https://icu-courses.com/) |[すごい時間割](https://www.sugojika.com/) | [Penmark](https://penmark.jp/) |
| ---- | ---- | ---- | ---- |----  |
| 開発元 | Kohshi Yamaguchi | 非公開| リクルート | Penmark |
| 動作環境 | iOS | iOS, Web| iOS, Android| iOS, Android |
| 特徴 | ICU内の目測で圧倒的なシェア（だった）．2026年度版には今のところ更新されていない． | 2026年初頭に公開されたサービス．アプリ内で表示されるユーザ数は90人超．大学が数千人規模であることを考えると既にある程度ののシェアを持っている．| リクルートIDログインが必須で規約に求人目的でのデータ利用が明記されている．個人的にはちょっと嫌かも．時間割見るのに私のPontaポイントの残高を知られる必要なくない？ | 公式シラバスからのデータ取得を謳っている．この4つの中で唯一同時間帯への複数授業追加に対応 |
|ICU独自スケジュール|O|O|X|X|
|技術スタック|Swift|VanillaJS(?), Express, Render \[APIレスポンスヘッダーより推測\]|不明|Flutter+Firebase\[[出典](https://note.com/penmarkjp/n/n64919ce034b1)\]|
:::
::: details コラム2: ICUの時間割アプリ今昔
- 1代目「ICUTTABLE」
    - 作者: 不明
    - 2代目の記事で言及あり
- 2代目「ICUTTABLE#2」: [ICU生向け時間割アプリ「ICUTTABLE#2」開発者インタビュー](http://weeklygiants.co/?p=1805)
    - 作者: 新井友朗さん（ID,卒業年のこと,  17-18か？）
- 3代目「Timetable For ICU」: [私の芽―ICU時間割アプリを作る](http://weeklygiants.co/?p=8872)
    - 作者: 千葉彌平さん（ID19; お会いしたことある）
        - 課題: Apple Developers Programの登録料を賄えない
    - プラットフォーム: iOS, Android 
    - [GitHub](https://github.com/YasuChiba/TimeTableForICU-ios-public)
    - [Twitter](https://x.com/TimeTableForICU)
    - [Facebook](https://www.facebook.com/timetableforicu/)
- 4代目 : [Timetable4ICU - 国際基督教大学の時間割アプリ](https://www.timetable4icu.com/)
    - Kohshi Yamaguchi (ID24-25?; お会いしたことある)
    - プラットフォーム: iOS
    - [GitHub](https://github.com/kohshi54/Timetable4ICU)
    - [Twitter](https://x.com/timetable4icu)
あくまでも独自調査に基づくものです．「○代目」も独自調査に基づき，網羅性は担保しません．
:::
## 👀Take a Closer Look at 技術選定
### 技術構成まとめ
- [Astro](https://astro.build/) : フロントエンド
	- [React](https://react.dev/) (詳述しない)
- [Cloudflare Workers](https://www.cloudflare.com/ja-jp/developer-platform/products/workers/)+[D1](https://www.cloudflare.com/ja-jp/developer-platform/products/d1/) (SQLite): サーバ/DB
- [DrizzleORM](https://orm.drizzle.team/): ORM
- [BetterAuth](https://better-auth.com/): 認証
- [Bun](https://bun.sh/)（おまけ）: JSランタイム
- Coudflare Registrar（おまけ）: ドメイン登録業者	
### Why Astro?
- ずっとコードの手入れをしてこなかった旧プロジェクト（Next.js）は忘れてAstroに出来たのは良かった．
	- 開発者として新たなことが学べ，コード負債を一掃できる．
- Webアプリという性質上Next.jsでSPAという選択肢もよかったのだろうが，Astroに乗り換えることで商用有料で課金最低額が20\$のVercelではなく，課金最低額が5\$で無料枠が太っ腹なCloudflareが現実的な選択肢になった点は良かった．
- [Landing Page](https://timetable.icu/)などJavaScriptが不要なページの表示速度がultra fast（PageSpeedInsights>=98とか）なのはおそらくAstroのおかげ．
	- 私がロジックを書くことよりもUIデザインに興味があるのもあって，見た目や体験の開発に集中出来たのはAstroのおかげだろう．
	- JS不要なところでJSXを書く必要がない．
- 難点は/timetable, /exploreといったページ切り替えで毎回ロードが発生すること．
	- 各ページの読込速度が短いので通信状況が良ければそこまで気にならない．
- スタイリングには[Tailwind CSS](https://tailwindcss.com/) + [DaisyUI](https://daisyui.com/)を使った．
	- LLMが出力したコードには大量の余計なスタイリングと絵文字が付いてくる．
		- 影とか，ボーダーとか，彼らは他の要素の統一なんて気にせずスタイリング付けてくる．
		- 絵文字は，文字コードでアイコンらしい図形が出力されるので彼らはよく出力するが，これもAIらしさを増すだけ．私は[Lucide Icons](https://lucide.dev/)を使った．美しいアイコンの作者の皆様に感謝．もちろん絵文字も美しい，ただ，AIがそれを乱用しているというだけで．
	- AIらしさは作品の魅力を損なうだけだ．たとえパッと見が良くてもそこに人間の手が加わっていないと分かると私はとても幻滅する．チェーン店のメニューに監修したシェフの顔が描いてあるのも，イトーヨーカドーの野菜に生産者の顔を載せるのも，人間らしさが信頼感を生むというコミュニケーションの一要素ではないか？
	- だからこそ私はDaisyUIを用いることで全体の配色と各コンポーネントの統一感を持たせた．
	- また，キャンパスで撮影された写真や生産者の顔をランディングページに載せた．これこそが現実世界にいる私たちの特権！
		- 彼らは写真を"生成"するが，人間であるところの我々には生成せずともそこに世界があり，ベクトル演算なしでフィルムに焼けばその景色を"写した"ことになるのだから！
### Why Cloudflare Workers + D1?
- 無料で運用できる先として魅力的だった（実際今まで無料で運用している）．
- 当初はAPI部分をWorkers，Static部分をPagesとしてdeployするものと思っていたがずいぶん前から[統合がアナウンス](https://blog.cloudflare.com/ja-jp/pages-and-workers-are-converging-into-one-experience/)されていたのでWorkersにDeployした.
- Vercelを以前の時間割アプリプロジェクトで使ったことがあって，Vercel有料プランが20\$/月スタートだった点が私のお財布には痛く，殆ど儲からないアプリに運用費をかけたくなかったのでもう少し柔軟なデプロイ先を探していた．
- あと，純粋にCloudflareを触ってみたかった．
	- あらゆる場所で目にする[CAPTCHA代替技術](https://www.cloudflare.com/application-services/products/turnstile/)生んだり，[Hono](https://hono.dev/)開発者が所属している会社という認識でした．
	- GoogleのreCAPTCHAに関しては，なぜ私たちが無給でWaymoの画像認識精度向上をさせられている（かもしらない，Googleは公式に認めていないので推測に過ぎない）のか分からない．
		- OpenAIがアフリカの英語話者にGPTの訓練を行わせたために当地の英語のくせChatGPTが残った\[[出典](https://www.nikkei.com/article/DGXZQOUC190OK0Z10C24A5000000/)\]という話があったが，それでも彼らは給料をもらっていたはずだ．それをなぜGoogleに対しては無料で私たちが？いつもあの横断歩道を選ばされる時にそのことを思うと嫌な気分になる．
		- しかし，[よく分からん記号がどの角度で置いてあるかを選ぶあの難しすぎる認証](https://blog.lycolia.info/0212)に比べればよっぽどマシかも知れない．もはや知能レベルだけで人間を見分けるのは殆ど不可能なのではないかしら．
![Cloudflare本社 in San Francisco|471](https://static.zenn.studio/user-upload/973f9523ab1c-20260424.webp =500x)
*せっかくSan Frainciscoが近いのでCloudflare本社を見に行きました．*
### Why DrizzleORM?
- [Prisma](https://www.prisma.io/)をずいぶん前に入門してよーわからんかった．特にスキーマ定義に使う独自言語．
	- Drizzleはスキーマ定義をTypeScriptの構文で読めるので読みやすい．
###  Why BetterAuth?
- 個人的にPasskeyが必須だった．
	- 現在使っているのは私だけみたい．多くのユーザがOAuthで事足りているか，Passkeyの説明が足りていないか．
	- ログイン手段が多くなることはセキュリティリスクもあげるので（特に学外者の侵入; 現状はほとんど秘匿情報を扱っておらず大きな問題にならないが，今後問題になる可能性）
	- 私の利便性を犠牲にしてでもOAuth1本に絞るべき？一方で，Google OAuthはアプリに返ってくるまでのクリック数が多い（ユーザが多いから仕方ない）ので好きじゃない．
- Supabase Authのrate limitやロックインが嫌だし，Auth.jsがBetterAuthに吸収されたことを踏まえて現代的な選択肢だった
### Why Bun?（おまけ）
- とにかく速いから．
	- npm, yarn, pnpm, bunをすべて使ってきて正直機能面での違いはよく分からなかったけれどインストール速度は一目瞭然だった．雷かな？ってくらいの速度でインストールされる（厳密にはグローバルに置いてあるキャッシュを読んでいるんだろうけど）
	- 一方で，プロジェクトが大きいときやNode.jsとの互換性が要るとき，特に高いセキュリティが求められている場面ではpnpmの方が有利なのかも．
	- bunのキャラクターを見る度においしそうでbunを食べに行った．OSSのロゴって可愛くないものも多いと感じるけど，ロゴが可愛いって大事だと思う．![豚まん|50](https://static.zenn.studio/user-upload/5e5d359b89ff-20260424.png =250x)*おいしそうな豚まん (pork bun) in Berkeley, CA*
### Why Cloudflare Registrar?（おまけ）
- お名前.comと違って，頻繁な「セキュリティが危ない」みたいな煽りメールもないし（危ないと思うなら最初からその機能付けなーね），ダッシュボードが操作しにくく気づいたら勝手に要らんオプションつけられてることもないので，とても良い．
	- 現代アートを愛する私からすればお名前com運営のGMOは[美術手帖オンライン](https://bijutsutecho.com/)に投資したり渋谷本社にアートペース（Cf. 下記"ちょっと寄り道"）があったり現代美術に理解があって好意的だったのに，お名前.comの体験が悪すぎてそれだけでフィランソロピーでの好印象が吹っ飛ぶほどである．
	- 国内ユーザメインの今回のようなアプリを作るときはさくらとかGMOとか日本のサービスを使っていきたいが，こうユーザ体験が悪くっちゃ使っていられない．
	- ::: details ちょっと寄り道: GMOの美術投資
		- [GMO美術館](https://banksy.tokyo/): 渋谷のGMO本社が入るビルに入居するアートスペース．博物館法の定める美術館ではなさそう．私はここに行ったことなかったけど，ここではずっとバンクシーを展示しているみたい．
		- バンクシーは自身の作品がこうやって[無断展示されることに抗議](https://bijutsutecho.com/magazine/insight/24571)してたはず．二次流通で高値が付くことを毛嫌いしてサザビーズでシュレッダーにかけたんじゃなかったっけ？
		- 結局GMOのアート投資もNFT流行の一端でIT企業のお金儲けに過ぎないのかしら．もちろん上場企業が投資家に対して投資対象が自社にもたらす利益を説明する責任があることがわかるけど．（もちろん美術手帖の支援は大事です！図書館で気軽に読める現代アート専門誌って，仏像特集など現代美術ではない特集も組まれる芸術新潮を除けばほぼこの1誌だけなので！！）
		- Banksyを含めアートマーケットについて書かれた本としては[小崎哲哉『現代アートとは何か』](https://calil.jp/book/4309279295)（河出書房新社）がおすすめです．
		:::

## 今後の課題
### ユーザ数の確保
- 友達1人のために作ったものの，せっかく作ったから多くの人に使ってもらいたいのが人情．
	- ユーザが増えないなら，その友達さえ卒業すればサービスは閉じる．
- 宣伝するためにやったこと
	- 友達に宣伝した
		- Twitterで友達が宣伝してくれた
		- 公式LINEで友達が宣伝してくれた
	- 学内にポスターを貼る（大学の許可待ち）
### 保守/点検
- 機密情報
	- ほとんどが大学から公開されている情報
- 攻撃
	- 噂に聞いていたとおりWordPress，Laravel, PHPなどの脆弱性や.env/.gitなど機密情報窃取を狙う侵略者が頻繁にやっていることが[Workers Observability](https://developers.cloudflare.com/workers/observability/)で観測できて興味深い．
		- きちんとそうした穴を閉じる対策を講じる良い経験になっている．
		- 世の中にある攻撃手法の一端を知れておもしろい．
		- 彼らはTwitterのリンクを踏んでいたり，あるいは自身がGoogleであると偽装したり色々な特徴があっておもしろい．
<!--
- テスト
	- ユーザ以外からアクセスできないことの保証
		- LocalStorageはクロスサイトスクリプティングに弱いが，機密情報ではないとの扱い
	- 漏れてはいけないのはメール->名前->プロフィール写真（ほとんどが顔写真ではない）->履修情報（ユーザ情報と結びついたら居場所が漏れて危険）
-->
- プライバシーと運用の効率化のバランス
	- ユーザであれば読み飛ばすような読みにくい[プライバシーポリシー](https://timetable.icu/privacy)を自分でも書いてしまうことに罪悪感を覚える．
		- 最低限，我々が運営のために保存しているデータについては明示している．
		- ブロックに分けたり要約するなどして，読みやすい規約（[Creative Commons](https://creativecommons.org/)のように[法的な文書](https://creativecommons.org/licenses/by/4.0/legalcode.en)と[一般向け文書](https://creativecommons.org/licenses/by/4.0/)を分けるべきかもしれません）を作っていくのが今後の課題．
	- 今後管理者が増えたときにどうやってガバナンスを確保していくのかも課題．
### 引き継ぎ！
- 私はもうすぐ4年生なので卒業しなくてはなりません．
- 帰国次第ハッカソンやワークショップでも開いてWebプログラミングに興味がある人の間口を学内で広めにゃあきません．
- 将来的には，[IBS](https://office.icu.ac.jp/ctl/aps/ibs.html)と呼ばれる公式に履修相談を受けている学生団体や学修・教育センター（[CTL](https://office.icu.ac.jp/ctl/)）などとコラボしたりこのアプリケーションの開発を引き継げたりしたら本望です．
	- 学生の履修にきっと役に立つものであることは保証出来るので，今後も透明性・責任のある運営と低コストの維持に努めます．
### 🌸We are Hiring!
あなたは ICU生ですか？
Zennを読んでいるということはこうした技術に少しは興味があるということですね？
充分過ぎる人材です！！
ぜひ私たちと一緒に時間割アプリを作りましょう．
10人ほどのDiscordサーバで開発や広報について議論していますのでぜひ[ご参加ください](https://discord.gg/2gmKTs4ezk)！
もしくは開発者代表・木越 斎 (`c271155i`[at]`icu.ac.jp`)までどうぞ～
### 参考事例
日本国内外を問わず大学生が時間割アプリを作るのはあるあるのようです．以下，特に参考にした事例たちです．
- 北海道大学: [Hupass](https://hupass.hu-jagajaga.com/)
- 筑波大学: [Twin:te](https://www.twinte.net/)
	- 透明性の高い資金運用やチーム開発・OAuthトークンの取得のみで学籍番号などを保持しないprivacy by designなど，参考にするべき点は多い．
- University of California, Berkeley: [Berkleytime](https://berkeleytime.com/)
- University of Singapore: [NUSMods](https://nusmods.com/)
- [芝浦工業大学](https://qiita.com/koheisato/items/7f2e604233372af35b41)
- [大阪公立大学](https://zenn.dev/omu_tryangle/articles/d3dfb4f369403d)
## Who Wrote This?: It's Me! 木越 斎. 
![Itsuki Kigoshi|181](https://cimetier.org/_astro/profile.CR0s2a3J_ZxUKlS.webp =250x)
*[Itsuki Kigoshi/木越 斎](https://cimetier.org/)*
この記事を書いたのは，国際基督教大学3年生の**木越 斎**（きごし いつき）です．
今は交換留学でUC Berkeleyにいます．数学を学んでいます，と言いたいところですが地理や天文学など興味が広がり過ぎて実力が追いつかず途方にくれています．
天文や地球科学（+海洋学??）と数理が関わるような分野で大学院に進学したいと考えていますが，私はこの先何を学んでいけば...
登山と現代美術鑑賞が趣味です．

- MIT Media Lab: https://www.media.mit.edu/
- OIST: https://www.oist.jp/

ご機嫌よう，
**[木越 斎](https://cimetier.org/)**

P.S. この記事にいただいたチップは「ICUのじかんわり」の運営資金に充当します．
また，[GitHub Sponsors](https://github.com/sponsors/ItsukiKigoshi)でのサポートも受け付けております．皆様からの暖かいご支援をお待ちしております．
