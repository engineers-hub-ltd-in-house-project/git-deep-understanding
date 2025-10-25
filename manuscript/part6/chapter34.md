# 第34章: ファイルの履歴を辿る

バグの修正や機能の仕様変更を調査する際、「このコードはいつ、誰が、なぜ変更したのか？」を知ることは非常に重要です。Gitは、プロジェクト全体の歴史だけでなく、ファイル一つ一つの変遷も詳細に記録しています。

この章では、特定のファイルの歴史を深く掘り下げるための3つの主要なコマンド、`git log`, `git blame`, `git checkout`を学びます。これらのツールは、コードの考古学者として、過去の変更の意図や経緯を解明するための強力な武器となります。

---
## 34.1 `git log -- <file>`: 特定のファイルの変更履歴を見る

`git log`は、コミット履歴を表示するコマンドですが、末尾にファイルパスを指定することで、そのファイルに関連するコミットだけを絞り込んで表示することができます。

さらに、`-p` (または `--patch`) オプションを付けると、各コミットでそのファイルにどのような変更（差分）が加えられたのかを具体的に見ることができます。

**シナリオ: `config.yml`の変更履歴を調査する**
```bash
# 演習用リポジトリ
git init
echo "version: 1.0" > config.yml && git add . && git commit -m "feat: Initial config"
echo "debug: false" >> config.yml && git add . && git commit -m "feat: Add debug flag"
sed -i 's/1.0/1.1/' config.yml && git add . && git commit -m "release: Bump version to 1.1"
```
`config.yml`が3回変更されました。このファイルの歴史だけを見てみましょう。
```bash
git log -p -- config.yml
```
出力結果 (一部):
```
commit <hash3> (HEAD -> main)
Author: Your Name <you@example.com>
Date:   ...

    release: Bump version to 1.1

diff --git a/config.yml b/config.yml
--- a/config.yml
+++ b/config.yml
@@ -1,2 +1,2 @@
-version: 1.0
+version: 1.1
 debug: false

commit <hash2>
Author: Your Name <you@example.com>
Date:   ...

    feat: Add debug flag
...
```
このように、`config.yml`に加えられた変更が、コミット単位で時系列に表示されます。これにより、「いつバージョンが1.1になり、いつdebugフラグが追加されたのか」が一目瞭然になります。

---
## 34.2 `git blame <file>`: 行単位の変更履歴を見る

`git log`がファイル全体の歴史を教えてくれるのに対し、`git blame`は**ファイル内の各行が、どのコミットで、誰によって最後に変更されたか**を教えてくれます。コマンド名が少し物騒ですが、「誰かを責める(blame)」ためではなく、「誰に聞けばこのコードの意図が分かるか」を見つけるためのコマンドです。

**シナリオ: `config.yml`の`debug`フラグは誰が`false`にした？**

プロジェクトで問題が発生し、`debug`フラグが`false`になっていることが原因かもしれないと疑っています。この設定がいつ、誰によって行われたのかを調査します。
```bash
git blame config.yml
```
出力結果:
```
^<hash1> (Your Name ... 1) version: 1.1
<hash2> (Your Name ... 2) debug: false
```
-   `^<hash1>`: `^`は、その行が最初のコミット(`hash1`)で追加されたことを示す。
-   `<hash2>`: `debug: false`の行は、`hash2`のコミットで変更された。
-   `(Your Name ...)`: 変更者の名前と日時。
-   `1)`, `2)`: ファイルの行番号。

この結果から、`debug: false`という行は`hash2`のコミット（`feat: Add debug flag`）で追加されたことがわかります。さらに詳しく知りたければ、`git show <hash2>`でそのコミットの詳細を確認したり、コミットメッセージや関連するPull Requestを探したりすることで、変更の背景にある文脈を深く理解できます。

---
## 34.3 `git checkout <commit> -- <file>`: 過去のファイルを取り出す

時には、ファイルの変更履歴を見るだけでなく、**過去のある時点のファイルそのもの**が必要になることがあります。「間違えて消してしまった設定を復活させたい」「リファクタリング前のロジックと比較したい」といったケースです。

`git checkout`にコミットハッシュとファイルパスを渡すことで、作業ディレクトリにその時点のファイルを復元できます。

**シナリオ: バージョンを`1.0`に戻したい**

最新の`config.yml`ではバージョンが`1.1`になっていますが、問題があったため一時的に`1.0`の状態に戻す必要が出てきました。
`git log -- config.yml`で、バージョンが`1.0`だった最後のコミットを探します。それは`hash2` (`feat: Add debug flag`) です。

**解決策: `checkout`で過去のバージョンを復元**
```bash
git checkout <hash2> -- config.yml
```
このコマンドを実行すると、作業ディレクトリにある`config.yml`の中身が、`hash2`のコミット時点の
```
version: 1.0
debug: false
```
という状態に上書きされます。`git status`を見ると、`config.yml`が変更された状態としてステージング待ちになっていることがわかります。

あとは、この復元したファイルをコミットすれば完了です。
```bash
git add config.yml
git commit -m "revert: Revert version to 1.0 due to issues"
```
これにより、`reset`や`revert`でプロジェクト全体を巻き戻すことなく、特定のファイルだけを安全に過去の状態に戻すことができました。

---
**まとめ**

| コマンド | 何をするか？ | 主な用途 |
| :--- | :--- | :--- |
| `git log -p -- <file>` | ファイルの変更履歴と差分を一覧する | ファイル全体の変遷を時系列で理解する |
| `git blame <file>` | 各行の最終変更者とコミットを特定する | 特定のコード行の背景や意図を調査する |
| `git checkout <commit> -- <file>` | ファイルを過去の特定の状態に復元する | 過去のバージョンの参照、またはファイルの巻き戻し |

これらのコマンドは、コードの歴史という広大なデータベースを探索するための強力なツールです。使いこなすことで、デバッグやコードリーディングの効率を飛躍的に向上させることができます。
