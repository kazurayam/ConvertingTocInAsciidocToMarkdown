# 目次つきのREADMEを作る ただしローカルでAsciidocからMarkdownへ変換する方法で

kazurayam
v1, 2022-01-23
v2, 2022-04-12

## Problem to solve

GitHubプロジェクトを自作したらREADMEドキュメントをかならず書く。READMEドキュメントはMarkdownの構文で書くのが普通だ。Markdownはシンプルで短い説明を書くにはとても使いやすいが、長い文章を書くにはいささか非力だ。例えばinclude命令が無いのでプログラムのソースコードをREADME文書に引用することができない。いちいちプログラムのソースをREAMDE.mdにコピペしなければならないのがつらい。

Asciidocの構文でREADMEの原稿を書いてMarkdownに変換する方法を見つけた。そのスクリプトは下記のURLに公開されていたものだ。

-   &lt;<https://github.com/github/markup/issues/1095>

ここまでの次第を私は以前Qiitaに投稿した。

-   <https://qiita.com/kazurayam/items/361fdcd6846bba7bd03a>

さてAsciidocでドキュメントの原稿を書いてMarkdownに変換したものをpublishするやり方はとても楽なので、ついつい長くて懇切丁寧な文章を書いてしまう。すると目次 (TOC, Table of contents) が欲しくなる。TOCを自動的に作りたいがどうやればいいか？

## 解決方法

### 今まで使ってきた方法と課題

GitHubで公開するREADMEドキュメントにTOCを自動的につける方法がネット上にいくつも提案され実用されている。わたしも下記のツールを何年も使ってきた。

-   <https://github.com/technote-space/toc-generator>

このツールはちゃんと動くし便利だ。このツールは GitHub Action として動作する。

しかしこのツールを使っていて不便に思う場面があった。わたしがREADME.mdファイルを編集してgit addしてgit commitしてgit pushする。するとpushのインベントを受けてGitHub Actionが起動され、README.mdファイルにTOCを挿入する。するとリモートレポジトリの中に格納されたREADME.mdファイルはわたしの手元にあるREADME.mdよりも1回分コミットが進んだ状態になる。だから私は次にREADME.mdを編集する前にgit pullしなければならない。さもないとローカルのREADME.mdファイルとリモートのREADME.mdファイルが同期しなくなってあとでmerge conflictが発生する。いったん発生させてしまったconflictをもみ消す作業は厄介だ。confictを数回経験してわたしは嫌になってしまった。

ちょっと待てよ。わたしはAsciidocで原稿を書いてそれを入力としてMarkdownを生成するというアクションをbashシェルスクリプトとして記述し、手元のPC上で実行している。ならば目次を生成するのをローカルでシェルによって実行できるのではないか？GitHub Actionに頼るのではなく。

調べたら出来ました。

## 代替的方法

### pandocにTOCを生成させる

bashシェルスクリプト `readmeconv.sh` を次のように記述した

    #!/usr/bin/env bash

    # Convert all the files with name ending with `*.adoc` into `*.md`.
    # `*.adoc` is an Asciidoc document file, `*.md` is a Mardown document file.
    # E.g, `readme_.adoc` will be converted into `readme_.md`
    # Except ones with `_` as prefix.
    # E.g, `_readme.adoc` is NOT processed by this script, will be left unprocessed.
    #
    # How to active this: in the command line, just type
    # `> ./readmeconv.sh`
    #
    # Can generate Table of content in the output *.md file by specifying `-t` option
    # `> ./readmeconv.sh -t`

    requireTOC=false

    optstring="t"
    while getopts ${optstring} arg; do
        case ${arg} in
            t)
                requireTOC=true
                ;;
            ?)
                ;;
        esac
    done

    find . -iname "*.adoc" -type f -maxdepth 1 -not -name "_*.adoc" | while read fname; do
        target=${fname//adoc/md}
        xml=${fname//adoc/xml}
        echo "converting $fname into $target"
        # converting a *.adoc into a docbook
        asciidoctor -b docbook -a leveloffset=+1 -o - "$fname" > "$xml"
        if [ $requireTOC = true ]; then
          # generate a Markdown file with Table of contents
          cat "$xml" | pandoc --standalone --toc --markdown-headings=atx --wrap=preserve -t markdown_strict -f docbook - > "$target"
        else
          # without TOC
          cat "$xml" | pandoc --markdown-headings=atx --wrap=preserve -t markdown_strict -f docbook - > "$target"
        fi
        echo deleting $xml
        rm -f "$xml"
    done

    # if we find a readme*.md (or README*.md),
    # we rename all of them to a single README.md while overwriting,
    # effectively the last wins.
    # E.g, if we have `readme_.md`, it will be overwritten into `README.md`
    find . -iname "readme*.md" -not -name "README.md" -type f -maxdepth 1 | while read fname; do
        echo Renaming $fname to README.md
        mv $fname README.md
    done

pandocコマンドに `--standalon --toc` というオプションを指定していることが肝心。コマンドラインで `readmeconv.sh` を実行するとき、`-t` オプションを付ける。するとTOCが生成されて `README.md` ファイルの先頭に挿入される。

    $ ./readmeconv.sh -t

もちろん `-t` オプションを指定しなればTOC無しで `README.md` ファイルが生成される。

    $ ./readmeconv.sh

目次が付加された `REAMDE.md` はこんな姿形になった。

![README\_with\_TOC](https://kazurayam.github.io/AsciidocToMarkdownConversionWithTOC/images/README_with_TOC.png)

#### ツールのバージョン

わたしは自分の環境に下記のツールをインストールした。

-   [asciidoctor](https://asciidoctor.org/)

-   [pandoc](https://pandoc.org/)

バージョンは下記のとおりだった。

    $ asciidoctor -v
    Asciidoctor 2.0.16 [https://asciidoctor.org]
    Runtime Environment (ruby 2.6.8p205 (2021-07-07 revision 67951) [universal.x86_64-darwin21]) (lc:UTF-8 fs:UTF-8 in:UTF-8 ex:UTF-8)
    :~/github/AsciidocToMarkdownConversionWithTOC (master *)
    $ pandoc -v
    pandoc 2.16.1
    Compiled with pandoc-types 1.22.1, texmath 0.12.3.2, skylighting 0.12.1,
    citeproc 0.6, ipynb 0.1.0.2
    User data directory: /Users/kazurayam/.local/share/pandoc
    Copyright (C) 2006-2021 John MacFarlane. Web:  https://pandoc.org
    This is free software; see the source for copying conditions. There is no
    warranty, not even for merchantability or fitness for a particular purpose.

### 問題あり 目次から本文へのリンクが切れていた

pandocが生成した\`README.md\`をGitHubにpushした。ブラウザで開いた。そして目次の中のリンクをクリックした。本文の該当箇所へジャンプすることを期待していたが、ジャンプしなかった。なぜだ？

ファイルとHTMLソースコードを解読したところ、目次の中のリンクが切れていることが判明した。

TOC内のリンク部分のHTMLコードがこれ:

    <a href="#_my_previous_solution">My previous solution</a>

ジャンプ先のHTMLコードがこれ:

    <a id="my-previous-solution" ...>

よく見ると href="#\_my\_previous\_solution" と id="my-previous-solution" とが対応していない。 \_ と - とは違う文字だもの。だからリンク切れするのは当然だ。どうしてこうなってしまったのだろうか？pandocが生成したREAMD.mdファイルのコードをよく見るとこうなっていた。

      -   [My previous solution](#my-previous-solution)

このコードは誤りなのではないか？下記のようなコードをpandocが生成してくれたら解決するだろう。

      -   [My previous solution](#my-previous-solution)

この推測にもとづき README.md ファイルを手書きで修正してGitHubにpushして見た。するとリンクが正しく動いた。

pandocがどうしてこのように動作するのか、わたしは知らない。バグなのかどうかも判断できない。まあ、それはいいことにしよう。わたしがいま望むのはGitHubにアップしたREADME文書のTOCから本文へのリンクが正しく動作することだ。そのためにツールを開発して、pandocが生成したREADME.mdファイルを書き換えることにした。

#### mdTocFilter.groovy

Groovy言語でスクリプトを書いた。

    /**
     * mdTocFilter.groovy
     */
    import java.util.regex.Pattern
    import java.util.regex.Matcher
    Pattern pattern = Pattern.compile('^(.*)\\(#(.*)$')
    def stdin = System.in.newReader()
    String line
    while ((line = stdin.readLine()) != null) {
      Matcher m = pattern.matcher(line)
      if (m.matches()) {
        /*
        println "does match"
        println "groupCount=" + m.groupCount()
        for (int i = 0; i <= m.groupCount(); i++) {
          println "group[" + i + "]=" + m.group(i)
        }
        */
        println m.group(1) + '(#' + m.group(2).replace('_', '-')
      } else {
        //println "no match"
        println line
      }
    }

このスクリプトがしているのは単純だ。REAMDE.mdファイルの全行をスキャンし、`(#_` を含む行を選んで書き換える。 `(#my-previous-solution)` を `(#my-previous-solution)` に置換する。

そして [readmeconv.sh](readmeconv.sh) の末尾に下記の数行を追加した。

    cat README.md | groovy mdTocFilter.groovy > temp.md
    cat temp.md > README.md
    rm temp.md

このフィルタ処理を経た README.md ファイルをGitHubにpushした。TOCから本文へのジャンプが正しく動作するようになった。

わたしは最初このフィルタをシェルのsedコマンドやawkで作ろうと試みたがちょっと難しかった。自分のPCにたまたまGroovy環境が整っていたのでフィルタをGroovyで実装した。Perl、Python、Ruby、Nodeでも同等のフィルタを実装できるだろう。

## 結論

わたしはGitHubプロジェクトのREADMEドキュメントをAsciidocで書いている。プログラムのソースコードを文中にincludeするのを自動化したかったから。READMEが長文になったので目次をつけたくなった。pandocコマンドに\`--toc\`オプションを指定して目次を生成したが、残念なながらリンク切れになってしまった。pandocが生成したREADME.mdファイルをほんの少し書きかえるフィルタを自作して、リンク切れを解消することができた。
