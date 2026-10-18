# dnsmasq で学ぶ DHCP と DNS のラボ：1 台の VM で動く、授業向けのネットワークラボ

[English](README.md) | [Français](README.fr.md) | [日本語](README.ja.md) | [Español](README.es.md)

[![check](https://github.com/mitsurutech/lab-dhcp-dns/actions/workflows/check.yml/badge.svg)](https://github.com/mitsurutech/lab-dhcp-dns/actions/workflows/check.yml)

ネットワークがアドレスを割り当て、名前に答える仕組みを教えるための、そのまま使える **DHCP と DNS のラボ**です。学校や学校法人が、生徒一人ひとりに専用の Ubuntu VM とブラウザー上のターミナルを用意できる [MitsuruTech ラーニングプラットフォーム](https://mitsurutech.co.uk/ja/learning)から提供しています。生徒はそれぞれ、自分のマシンの中に作られた小さなネットワークを使います。サーバー 1 台と、クライアント 2 台（`pc1` と `pc2`）で、クライアントは**アドレスをまったく持たない状態**で起動します。生徒は、家庭用ルーターやラボのあちこちにある小さな DHCP・DNS サーバーの **dnsmasq** を設定します。サーバーが範囲の中からアドレスを貸し出し、クライアントにルーターと DNS サーバーを伝え、片方のクライアントに **MAC アドレスで固定したアドレス**を与え、ラボ独自の名前に答えるようにします。生徒のコンピューターには何もインストールされません。

サーバーとクライアントは、1 つの仮想スイッチにつながった**ネットワーク名前空間**で、実際のツールが仕事をします。サーバーでは `dnsmasq`、クライアントでは `dhclient`、観察には `dig`、`ip`、`tcpdump` を使います。それがねらいです。

## ラボ

| ラボ | ファイル | 生徒がすること | ヘルプページ |
| --- | --- | --- | --- |
| DHCP と DNS | [`dhcp-dns.lab.yaml`](dhcp-dns.lab.yaml) | クライアントが範囲内のアドレスと正しいルーター、DNS サーバーを受け取り、片方のクライアントは常に同じアドレスになり、ラボの 2 つの名前だけが名前解決でき、ほかの名前は解決できないように、dnsmasq の設定を書く | [DHCP と DNS ラボ](https://mitsurutech.co.uk/ja/help/lab-dhcp-dns) |

ラボは、生徒のホームフォルダーに README を置きます。何をするかは書かれていますが、答えは書かれていません。ラボには**チェック**があります。これは、教員が読まなくても、提出された VM を採点するスクリプトです。新しく起動した直後の、保存済みマシンのコピーの上で実行され、動いていた dnsmasq を止めて、`/etc/dnsmasq.d` に保存されたファイルから新しく起動し、2 台のクライアントを初期化して、もう一度アドレスを要求させます。そのため、実際の動作が正しい場合にだけ合格します。クライアントが範囲内のアドレスを、正しいルーター、DNS サーバー、リース時間とともに受け取ること、2 台目のクライアントが固定アドレスを受け取ること、2 つの名前が正しいアドレスに解決されること、そして未知の名前には答えないことです。クライアントに手で入力したアドレス、すべての名前に答えるワイルドカード、`/etc/hosts` に書いた名前では合格しません。

## ファイルの中身

`dhcp-dns.lab.yaml` は、プラットフォームがそのまま実行する、**VM ラボの定義**一式です。

| キー | 役割 |
| --- | --- |
| `cpus`、`memory`、`disk` | 生徒それぞれの VM の大きさ：CPU 1 つ、1024 MB、10 GB |
| `open` | ブラウザーで開くポート：なし。ラボはターミナルです |
| `setup` | 生徒それぞれの VM の初回起動時に root として一度だけ実行されるスクリプト。dnsmasq、DHCP クライアント、`dig`、`tcpdump` を Ubuntu のアーカイブからインストールし、`dnslab` コマンド、ラボの README、起動時にネットワークを作り直すユニットを書き込みます |
| `check` | 採点スクリプト。新しく起動した直後の、保存済みディスクのコピーの上で root として実行されます。終了ステータスが結果で、`0` が合格です |

インストールされる `dnslab` コマンド：`dnslab status` でマシンを一覧し、`dnslab restart` で保存済みの設定から dnsmasq をもう一度実行し、`dnslab renew pc1` でクライアントにリースを要求させ、`dnslab shell pc1` でマシンの中のシェルを開きます。

## プラットフォームで使う

**クラスの課題として出すには：**

1. モジュールで課題を追加し、**Ubuntu VM ラボ**のカードを選びます。
2. サンプルを押すか、ここにあるファイルの内容を貼り付けます。サンプルを使うとテキストが置き換わりますが、元に戻すで自分のものに戻せます。
3. 公開し、まず自分のコピーで試します。生徒にはそれぞれ専用の VM が用意され、セッションをまたいで保持されます。
4. ほかの課題と同じように配布します。提出されたら、クラス一覧の生徒の行でチェックを実行します。その生徒の保存済みマシンのコピーを、保存し戻さない形で起動し、合格したかどうかと、出力された内容を表示します。

マニュアルの [Ubuntu VM ラボのガイド](https://mitsurutech.co.uk/ja/help/labs)と[サンプルラボのページ](https://mitsurutech.co.uk/ja/help/sample-labs)に、そのほかのことが載っています。このラボの基本レッスン **DHCP and DNS Basics** は、Public college と、すべての新しい学校にあります。ラボを変更するには、ファイルをコピーし、`setup` と `check` のアドレスと名前をあわせて編集してください。

学校のアカウントはまだありませんか？[最初の 1 年は無料](https://mitsurutech.co.uk/ja/pricing)で、生徒 50 人まで使えます。[登録](https://mitsurutech.co.uk/ja/signup)にはメールアドレスだけが必要です。

## ファイルの検証

push のたびに、`.github/workflows/check.yml` が、ファイルが上の 5 つのキーを持つ有効な YAML であること、そして `setup` と `check` がそれぞれプラットフォームの上限である 16 KB 以内であることを確認します。VM は起動しません。それは、プラットフォーム自身のマシンで行われます。

## ラボでできないこと

VM が届くのは Ubuntu のパッケージアーカイブ、PyPI、npm、GitHub だけなので、ラボをインターネット上のホストに向けることはできず、ラボの DNS サーバーには名前を転送する先がありません。VM は 1 台で、サーバーとクライアントはその中の名前空間なので、クライアントは別々のマシンではなく、競合するもう 1 台の DHCP サーバーもありません。

## この演習について

[Mitsuru Technologies](https://mitsurutech.co.uk) が、[MitsuruTech ラーニングプラットフォーム](https://mitsurutech.co.uk/ja/learning)のために作りました。学校向けに、Python、Java、Go、Node、SQL、Linux をブラウザーで使えるようにし、コンテナーラボ、VM ラボ、GitHub から公開する Web アプリ、自動採点されるクイズも備えています。アカウントなしでコードを試せる[プレイグラウンド](https://mitsurutech.co.uk/ja/playground)もあります。

同じ提供元のほかのラボ：[lab-netlab](https://github.com/mitsurutech/lab-netlab)、[lab-juice-shop](https://github.com/mitsurutech/lab-juice-shop)、[lab-dvwa](https://github.com/mitsurutech/lab-dvwa)、[lab-webgoat](https://github.com/mitsurutech/lab-webgoat)、[lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin)、[lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea)、[lab-sample](https://github.com/mitsurutech/lab-sample)。

## ライセンス

この作品は [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) でライセンスされています。Mitsuru Technologies Ltd のクレジットを表示し、商用に使わず、これをもとに作ったものを同じライセンスで共有するなら、学習と授業のために共有・改変できます。全文は `LICENSE` ファイルにあります。

このラボがインストールするアプリケーションは、それぞれのプロジェクトのライセンスに従います。[dnsmasq](https://thekelleys.org.uk/dnsmasq/doc.html) は GPL-2.0 または GPL-3.0 でライセンスされ、ISC DHCP クライアントは MPL-2.0 でライセンスされています。どちらも VM の起動時に Ubuntu のアーカイブからインストールされ、ここでは配布していません。
