-   [AsciidocドキュメントからMarkdownドキュメントを生成する、ただし目次つきで](#_asciidocドキュメントからmarkdownドキュメントを生成するただし目次つきで)
    -   [解決すべき問題](#_解決すべき問題)
    -   [解決方法](#_解決方法)
        -   [今までわたしがどうやってきたかというと](#_今までわたしがどうやってきたかというと)
        -   [新たな疑問](#_新たな疑問)
    -   [Description](#_description)

# AsciidocドキュメントからMarkdownドキュメントを生成する、ただし目次つきで

kazurayam &lt;<jedi@kazurayam.com>&gt;
v0.1, 2022-01-23

## 解決すべき問題

GitHubプロジェクトを自作したらREADMEドキュメントをかならず書く。READMEドキュメントはMarkdownの構文で書くのが普通だ。Markdownはシンプルで短い説明を書くにはとても使いやすいが、長い文章を書くにはいささか非力だ。特にプログラムのソースコードをREADMEも文中に引用することができないのがつらい。いっぽう、Asciidocの構文でREADMEの原稿を書いてMarkdownに変換する方法を見つけた。そのスクリプトは下記のURLに公開されていたものだ。

-   <https://github.com/github/markup/issues/1095> 9th June 2021に chevdoor が投稿したコード

…​と言うことを私は以前Qiitaに投稿した。

-   <https://qiita.com/kazurayam/items/361fdcd6846bba7bd03a>

Asciidocでドキュメントの原稿を書いてMarkdownに変換したものをpublishするやり方はとても楽なので、ついつい長くて懇切丁寧な文章を書いてしまう。すると目次 (Table of contents) が必要になってきた。TOCをどうやって作ろうか？

## 解決方法

### 今までわたしがどうやってきたかというと

ネットを調べるとGitHubで公開するREADMEドキュメントにTOCを自動的につける方法がいくつも提案してされ実用されている。わたしも下記のツールを何年も使ってきた。

-   <https://github.com/technote-space/toc-generator>

このツールはちゃんと動くし便利だ。このツールは GitHub Action として動作する。

### 新たな疑問

前述のツールを使っていて少し不便に思うことがあった。わたしがREADME.mdファイルを編集してgit addしてgit commitしてgit pushする。するとpushのインベントを受けてGitHub Actionが起動され、README.mdファイルにTOCを挿入する。するとリモートレポジトリの中に格納されたREADME.mdファイルはわたしの手元にあるREADME.mdよりも1回分コミットが進んだ状態になる。だから私は次にREADME.mdを編集する前にgit pullしなければならない。さもないとローカルのREADME.mdファイルとリモートのREADME.mdファイルが同期しなくなってあとでmerge conflictが発生する。厄介だ。

ちょっと待てよ。わたしはAsciidocで原稿を書いてそれを入力としてMarkdownを生成するというアクションをbashシェルスクリプトとして記述し、手元のPC上で実行している。ならば目次を生成するのをローカルで実行できるのではないか？GitHub Actionに頼るのではなく。…​ 調べたら、出来ました。

## Description
