# 第 22 章: インタラクティブリベース `-i`

---

前章で学んだ `git rebase <branch>` は、ブランチの土台を一括で変更するものでした。しかし `rebase` の真の力は、`--interactive` (または `-i`) オプションを付けたときに発揮されます。

**インタラクティブリベース (`git rebase -i`)** を使うと、ブランチに含まれる一連のコミットを、まるでビデオ編集のように自由自在に操作できます。

- コミットメッセージを修正する (`reword`)
- 複数のコミットを一つにまとめる (`squash`, `fixup`)
- コミットを並べ替える (順番の変更)
- コミットを削除する (`drop`)
- コミットを編集して変更内容を追加する (`edit`)

これにより、他人に見せる前のブランチの歴史を、非常に分かりやすく、論理的に整えることができます。

---
## 22.1 インタラクティブリベースの開始

インタラクティブリベースは、「どこからどこまでのコミットを操作対象にするか」を指定して開始します。一般的には、「現在のブランチと、分岐元のブランチとの差分」を対象にします。

`feature` ブランチ上で `main` ブランチから分岐して以降のコミットを全て操作したい場合は、以下のように実行します。

```bash
git rebase -i main
```

また、`HEAD` から遡って特定の数のコミットを対象にすることもできます。例えば、最新の 3 つのコミットを操作したい場合はこうです。
```bash
git rebase -i HEAD~3
```

このコマンドを実行すると、Git はデフォルトのエディタを起動し、操作対象のコミットリストと、実行可能なコマンドのリストを表示します。

```text
pick f7f3f6d Add feature A
pick 310154e Add feature B
pick a5f4a0d Fix typo in feature B

# Rebase 710f0f8..a5f4a0d onto 710f0f8 (3 commands)
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

この画面がインタラクティブリベースの中心です。
- 上半分には、古い順にコミットがリストアップされています。
- 各行の先頭にある `pick` が、そのコミットに対して実行される**コマンド**です。
- 下半分には、利用可能なコマンドの解説が書かれています。

私たちは、この**テキストファイルを編集して保存する**ことで、Git にこれから実行してほしい歴史の書き換えシナリオを指示します。

---
## 22.2 一般的なユースケース

ここでは、インタラクティブリベースの一般的な使い方をいくつか見ていきましょう。

### a) コミットメッセージの修正 (`reword`)

`Add feature A` のコミットメッセージを、もっと分かりやすく `Add login feature` に変更したいとします。
`f7f3f6d` の行の `pick` を `reword` (または `r`) に書き換えて保存します。

```text
reword f7f3f6d Add feature A
pick 310154e Add feature B
pick a5f4a0d Fix typo in feature B
```

ファイルを保存して閉じると、Git はすぐに新しいエディタを開き、コミットメッセージの編集を促します。
メッセージを修正して保存すれば、そのコミットのメッセージが新しくなります。

### b) コミットの結合 (`squash`, `fixup`)

`Add feature B` と `Fix typo in feature B` の 2 つは、別々のコミットにする必要はありません。一つの「feature B の追加」コミットにまとめたいです。
この場合、まとめられる側のコミット (`a5f4a0d`) のコマンドを `squash` (または `s`) に変更します。

```text
pick f7f3f6d Add feature A
pick 310154e Add feature B
squash a5f4a0d Fix typo in feature B
```

`squash` は、そのコミットを**直前のコミットに**統合します。
ファイルを保存すると、Git は新しいエディタを開き、結合後の新しいコミットメッセージをどうするか聞いてきます。デフォルトでは両方のメッセージが含まれているので、これを分かりやすく一つにまとめます。

もし、タイポ修正のコミットメッセージ (`Fix typo in feature B`) が不要で、単に `Add feature B` のメッセージだけを残したい場合は、`squash` の代わりに `fixup` (または `f`) を使います。`fixup` は、コミットを統合し、かつそのコミットのメッセージを破棄するコマンドです。この場合、メッセージ編集のステップはスキップされます。

### c) コミットの削除 (`drop`)

もし `Add feature A` のコミットがやっぱり不要になった場合、その行全体を**削除**するか、コマンドを `drop` (または `d`) に変更します。

```text
# f7f3f6d の行を削除
pick 310154e Add feature B
pick a5f4a0d Fix typo in feature B
```
または
```text
drop f7f3f6d Add feature A
pick 310154e Add feature B
pick a5f4a0d Fix typo in feature B
```

ファイルを保存すれば、そのコミットは歴史から消え去ります。

### d) コミットの順序変更

`Add feature B` を `Add feature A` より先に適用した歴史にしたい場合、単純に行の順序を入れ替えるだけです。

```text
pick 310154e Add feature B
pick a5f4a0d Fix typo in feature B
pick f7f3f6d Add feature A
```
ファイルを保存すれば、Git は上から順にコミットを再適用し、歴史を並べ替えてくれます (ただし、互いに依存関係がなく、コンフリクトが発生しない場合に限ります)。

---
## 22.3 Rebase 中のコンフリクト

コミットを並べ替えたり、内容を編集したりすると、マージの時と同じように**コンフリクト**が発生することがあります。
例えば、後のコミットが、前のコミットで変更した箇所をさらに変更している場合などです。

Rebase 中にコンフリクトが起きると、プロセスは一時停止し、コンフリクトの解決を求められます。
```
CONFLICT (content): Merge conflict in file.txt
error: could not apply a5f4a0d... Fix typo in feature B
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit with "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
```
やるべきことは Merge の時と同じです。
1.  コンフリクトマーカー (`<<<<<<<`, `=======`, `>>>>>>>`) が入ったファイルを修正する。
2.  `git add <file>` で、解決したことを Git に伝える。
3.  `git rebase --continue` を実行して、rebase プロセスを再開する。

もし解決が難しい場合は `git rebase --abort` で、rebase を開始する前の状態に安全に戻ることができます。

---
**まとめ**

- `git rebase -i` (インタラクティブリベース) は、コミットの歴史を対話的に編集するための強力なツールである。
- コマンドリスト (テキストファイル) を編集することで、コミットメッセージの修正 (`reword`)、コミットの結合 (`squash`, `fixup`)、削除 (`drop`)、順序変更などが可能。
- 複数の細かい "作業中" コミットを、一つの論理的な単位にまとめることで、プルリクエスト前のブランチを綺麗に整えることができる。
- Rebase 中にコンフリクトが発生した場合は、ファイルを修正し、`git add` してから `git rebase --continue` で続行する。
- Rebase は歴史を書き換えるため、前章で学んだ黄金律 (公共のブランチは rebase しない) はここでも同様に適用される。
