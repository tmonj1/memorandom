---
title: "Git submodule の使い方"
date: 2018-12-31T09:27:45+09:00
draft: true
---

## submodule とは

submodule とは、他のリポジトリを自分のリポジトリの一部として取り込む仕組みです。

submodule は他のリポジトリの特定のコミットへのリンク (gitlink) です。submodule として指定された側のリポジトリは特に制限なく、引き続き独立したリポジトリとして利用可能です。

submodule を取り込む側のリポジトリ (これを superproject と言います) は、参照する submodule を追加・削除したり、参照するコミットを変更したりできます (たとえば submodule 側が更新された場合、submodule の HEAD を参照するように更新できます)。

submodule と superproject は互いに独立なので、コミットログも別々に管理されます (混在することはありません)。submodule を追加すると submodule のディレクトリとファイルが superproject に追加されますが、Git はこれを submodule のリンクの追加という形で変更を認識します。変更をリポジトリに登録するには、通常どおり commit を実行します。

submodule と superproject の関係は多対多です。一つの  リポジトリに複数の submodule を登録できますし、逆に一つのリポジトリを submodule とする superproject を複数作ることもできます。また、submodule はネストできます。

## submodule の仕組み

上述のとおり submodule はリンク (gitlink) なので、superproject のリポジトリが保持しているのは取り込んだ時点でのリンク先の情報です。

git submodule add コマンドで submodule を追加すると、指定した submodule のファイル一式が取り込まれるだけでなく、.gitmodules ファイル  (なければ作られる ) にリンクの情報が登録されます。以下に例を示します。

```bash:
$ ls -a
.git/ index.js  README.md
$ # git module add コマンドで submodule を追加
$ git submodule add http://repo1.mygit.com/sub1.git
$ ls -a
.git/ .gitmodules index.js README.md sub1/
$ cat .gitmodules
[submodule "sub1"]
  path = sub1
  url = http://repo1.mygit.com/sub1.git
$ # 追加したサブモジュールをコミット
$ git commit -m "add submodule sub1"
```

上記の例では、submodule の  最新の (= HEAD) のファイル一式が sub1 フォルダ以下に展開され、同時に .gitmodules ファイルが追加されます。

.gitmodules  には追加した submodule の URL とパスの情報が格納されます。submodule リポジトリで新たに変更が push されても、superproject 側に自動で反映されるわけではありません (submodule を add した時点での HEAD のファイル一式のままです)。変更を反映するには、--remote オプションをつけて git submodule update コマンドを実行します。

```bash:
$ # sub1 を最新 (= HEAD) に更新
$ git submodule update --remote
```

"--remote" をつけない場合、superproject に記録された最新の状態に submodule が復元されます。たとえば、submodule を完全に削除して git submodule update を実行すると、削除前の状態を復元できます。

```bash:
$ # sub1 ディレクトリを試しに削除
$ rm -rf sub1
$ git submodule update
$ ls
# sub1 ディレクトリが復元される
sub1/
```

HEAD ではなく、特定の commit にリンクしたいときは、submodule フォルダに移動して通常どおり git checkout &lt;commit id> を実行すればよいです (その後 superproject に cd して git add, git commit, git push する)。

なお、submodule のどのコミットにリンクしているかは .gitmodules には書かれていません。これは commit 自体に記録されているので、submodule を取り込んだときのコミット ID を指定して git show を実行すれば確認できます。

## コマンド一覧 (簡易版)

submodule を扱うときによく使うコマンドは以下です。詳細なオプションについては  リファレンスを参照してください。

### (1) git submodule add &lt;repo-url> &lt;dirname>

指定したリポジトリの HEAD を取得し、指定したフォルダに格納します。

&lt;repo-url> には "https://repo.mygit.com/sub.git" のように  submodule として追加するリポジトリの URL を指定します。

URL は　 .gitmodules にも格納されます。

&lt;path> を省略した場合、リポジトリ名のカノニカル部分 ("sub.git" であれば  "sub") が使用されます。

取得した submodule は変更セットとして取り込まれるため、カレントリポジトリに反映するためには commit する必要があります。

カレントリポジトリ (submodule から見て superproject と呼ばれる) のリモートリポジトリからの相対パスを指定することもできます。たとえば、カレントリポジトリのリモートリポジトリが "https://repo.mygit.com/main.git" の場合、"../sub1"  と指定  できます。

### (2) git submodule status

すべての submodule のコミットと  (superproject 側の ) 変更状況を表示します。

### (3) git submodule init

.gitmodules  の URL を .git/config にコピーします。

```bash
$ cat .git/config
:
: (途中省略)
[submodule]
url = http://repo.mygit.com/sub1
```

git submodule update コマンドは実行時に  .gitmodules ではなくこちらの URL を参照します。

このため自分のローカル環境だけ submodule が別のところを指すようにしたいときは .git/config の URL を  手で編集して書き換えます (.gitmodules の URL を直接書き換えるのは **公式**の URL を上書きしてしまうことになるのでよろしくありません)。

### (4) git submodule update

.git/config に記載された URL から submodule の  HEAD を取得します。

### (5) git submodule summary

各 submodule の変更状況を表示します。

### (6) git submodule deinit &lt;dirname>

指定した submodule を削除します。ワークツリーを削除し、.git/config のエントリも削除します。

## submodule を使うときの注意点

submodule は便利な機能ですが、いくつか注意が必要です。

### (1) branch を切り替えたとき、自動で中身が切り替わらない

たとえば devlop ブランチから topic ブランチに切り替えたとき 、topic ブランチの submodule が古いバージョンを指していたとしても、submodule フォルダの中身は自動では切り替わりません。これは git pull してきたときも同様です。

ブランチを切り替えたときや git pull してきたときは明示的に git submodule update を実行して同期させる必要があります。

### (2) conflict が発生したら手動でマージする必要がある

submodule も同時に 2 人が編集すれば conflict する可能性があります。Git は自動マージできないので、手動で解決する必要があります。

### (3)  更新は submodule 側のみ、superproject は参照のみとしたほうがよい

superproject 側で更新すると、superproject だけでなく submodule にも更新をコミットする必要がありますが、これは煩雑なだけでなく操作を忘れやすいです。このため、superproject 側では更新しないルールがおすすめです。
