# 第22章: インタラクティブリベース

`git rebase`は、ブランチの土台を付け替えるだけでなく、 `--interactive` (または `-i`) オプションを使うことで、コミットの歴史を自在に編集する強力なツールに変身します。これを**インタラクティブリベース**と呼びます。

インタラクティブリベースを使うと、まるでタイムマシンに乗って過去に戻り、歴史を思い通りに修正するような体験ができます。具体的には、以下のような操作が可能です。

-   コミットメッセージの修正 (`reword`)
-   複数のコミットを1つにまとめる (`squash`, `fixup`)
-   コミットの順序を入れ替える
-   コミットを削除する (`drop`)
-   コミットの中身を修正する (`edit`)

この章では、これらの操作を実践し、乱雑なコミット履歴を美しく整形するテクニックを学びます。

---
## 22.1 演習の準備: 整形前の乱雑な歴史

まず、インタラクティブリベースの威力を実感するために、意図的に少し乱雑なコミット履歴を作ります。

```bash
# 実験用ディレクトリを作成
mkdir git-interactive-rebase && cd git-interactive-rebase
git init

# 最初のファイルをコミット
echo "Feature content" > feature.txt
git add .
git commit -m "feat: Start feature development"

# 作業途中のコミット
echo "More content" >> feature.txt
git add .
git commit -m "wip" # wip = Work In Progress (作業中)

# 機能の一部を追加
echo "Final touches" >> feature.txt
git add .
git commit -m "feat: Add final touches"

# 最初のコミットメッセージにタイポがあったことに気づく
# (本来なら git commit --amend を使う場面だが、ここでは演習のため新しいコミットとする)
git commit --allow-empty -m "fix: Correct typo in first commit message"

# 不要だったかもしれない変更
echo "Temporary change" >> temp.txt
git add .
git commit -m "feat: Add temporary file"
```

`git log --oneline`で現在の歴史を見てみましょう。
```
<hash5> feat: Add temporary file
<hash4> fix: Correct typo in first commit message
<hash3> feat: Add final touches
<hash2> wip
<hash1> feat: Start feature development
```
この歴史にはいくつかの問題があります。
-   `wip`のような意味のないコミットメッセージがある。
-   本来一つであるべき機能追加が複数のコミットに分かれている。
-   タイポ修正のためのコミットは、元のコミットに含めるべき。
-   最後の`temporary file`は最終的に不要かもしれない。

この乱雑な歴史を、インタラクティブリベースで整形していきます。

---
## 22.2 インタラクティブリベースの開始

インタラクティブリベースは `git rebase -i <commit>` の形で実行します。この`<commit>`には、「**どこからどこまでのコミットを編集したいか**」の**始点（の1つ前）** を指定します。

今回はすべてのコミットを編集対象にしたいので、最初のコミットの親である`HEAD~5`を指定します。
```bash
git rebase -i HEAD~5
```
このコマンドを実行すると、Gitはテキストエディタを起動し、以下のようなファイルを表示します。

```
pick <hash1> feat: Start feature development
pick <hash2> wip
pick <hash3> feat: Add final touches
pick <hash4> fix: Correct typo in first commit message
pick <hash5> feat: Add temporary file

# Rebase <hash_base>..<hash5> onto <hash_base> (5 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
```
上半分には編集対象のコミットが**古い順に**リストアップされ、下半分には利用可能なコマンドの説明が書かれています。Gitはこのリストを上から順番に処理していきます。

---
## 22.3 歴史の編集

このリストを編集して、私たちの望む歴史の形をGitに指示します。今回は以下のように整形したいと考えます。
1.  最初のコミットのメッセージを修正する (`reword`)
2.  `wip`と`feat: Add final touches`を最初のコミットと一つにまとめる (`squash`)
3.  タイポ修正のコミットは、内容もメッセージも不要なので、`fixup`で完全に溶け込ませる。
4.  `temporary file`のコミットは完全に削除する (`drop`)

この計画に従って、エディタでファイルを以下のように書き換えます。

```
reword <hash1> feat: Start feature development
squash <hash2> wip
squash <hash3> feat: Add final touches
fixup <hash4> fix: Correct typo in first commit message
drop <hash5> feat: Add temporary file
```
ファイルを保存してエディタを閉じると、リベースのプロセスが始まります。Gitは書き換えた指示書を上から順番に実行していきます。

1.  **`reword <hash1>`**: 最初のコミットで`reword`を指定したので、Gitはコミットメッセージを編集するためのエディタを開きます。メッセージを「feat: Implement the main feature」のように分かりやすく修正して保存します。
2.  **`squash <hash2>` & `squash <hash3>`**: 次に`squash`が2つ連続しています。Gitはこれら3つのコミット（元の`hash1`, `hash2`, `hash3`）を一つにまとめる処理を行い、統合後の新しいコミットメッセージを編集するためのエディタを開きます。3つのコミットメッセージが結合された状態で表示されるので、不要な行を削除し、最終的なメッセージを一つにまとめて保存します。
3.  **`fixup <hash4>`**: `fixup`は`squash`と似ていますが、このコミットのメッセージは完全に破棄されます。Gitは自動的にこのコミットを前のコミットに溶け込ませます。
4.  **`drop <hash5>`**: `drop`を指定したので、このコミットは完全に削除されます。

すべての処理が完了すると、リベース成功のメッセージが表示されます。

---
## 22.4 整形後の歴史

`git log --oneline`で最終的な歴史を確認してみましょう。
```
<new_hash> feat: Implement the main feature
```
あれだけ乱雑だった5つのコミットが、意味のある1つの美しいコミットに生まれ変わりました。これこそがインタラクティブリベースの力です。

---
**まとめ**

この章では、インタラクティブリベースを使ってコミットの歴史を自在に編集する方法を学びました。

-   `git rebase -i <commit>`で、指定したコミット以降の歴史を編集対象とするインタラクティブモードを開始できる。
-   表示された編集画面で、`pick`, `reword`, `squash`, `fixup`, `drop`などのコマンドを指示することで、歴史を思い通りに再構築できる。
-   コミットの並べ替えも、編集画面の行を入れ替えるだけで可能。
-   インタラクティブリベースは、Pull Requestを出す前にローカルのコミット履歴を整理し、レビューしやすい形にするための非常に強力なテクニックである。

もちろん、これも**歴史を書き換える**操作です。前章で学んだ黄金律「共有されたブランチはリベースするな」は、インタラクティブリベースにも同様に適用されます。

最後に演習用ディレクトリを削除しておきましょう。
```bash
cd ..
rm -rf git-interactive-rebase
```
