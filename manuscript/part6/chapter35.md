# 第 35 章: ファイルの履歴を調査する (`log`, `blame`, `checkout`)

バグの原因調査や仕様変更の経緯を追う際、「このコードはいつ、誰が、なぜ変更したのか？」という情報は極めて重要です。Git はプロジェクト全体の歴史だけでなく、ファイル一つ一つの変遷も詳細に記録しています。

この章では、特定のファイルの歴史を深く掘り下げるための 3 つのコマンド、`git log`, `git blame`, `git checkout` を学びます。これらはコードの考古学者として、過去の変更の意図を解明するための強力な武器となります。

---
## 35.1 `git log -- <file>`: 特定のファイルの変更履歴を見る

`git log` はコミット履歴を表示するコマンドですが、末尾にファイルパスを指定することで、そのファイルに関連するコミットだけを絞り込んで表示できます。

さらに `-p` (または `--patch`) オプションを付けると、各コミットでそのファイルにどのような差分が加えられたのかを具体的に確認できます。

**シナリオ**: `config.yml` の変更履歴を時系列で追いたい。

```bash
# 演習用リポジトリ
git init
echo "version: 1.0" > config.yml && git add . && git commit -m "feat: Initial config"
echo "debug: false" >> config.yml && git add . && git commit -m "feat: Add debug flag"
sed -i 's/1.0/1.1/' config.yml && git add . && git commit -m "release: Bump version to 1.1"

# config.yml の履歴と差分を表示
git log -p -- config.yml
```

出力結果 (一部抜粋):
```
commit <hash3> (HEAD -> main)
Author: Your Name <you@example.com>
...
    release: Bump version to 1.1

diff --git a/config.yml b/config.yml
--- a/config.yml
+++ b/config.yml
@@ -1,2 +1,2 @@
-version: 1.0
+version: 1.1
 debug: false

commit <hash2>
...
```
このように、`config.yml` に加えられた変更がコミット単位で時系列に表示され、「いつバージョンが 1.1 になり、いつ debug フラグが追加されたのか」が一目瞭然です。

---
## 35.2 `git blame <file>`: 行単位の変更責任者を特定する

`git log` がファイル全体の歴史を教えてくれるのに対し、`git blame` は**ファイル内の各行が、どのコミットで、誰によって最後に変更されたか**を特定します。これは誰かを「責める (blame)」ためではなく、「このコードの意図を誰に聞けばわかるか」を見つけるためのコマンドです。

**シナリオ**: `config.yml` の `debug` フラグは誰が `false` にしたのか？

```bash
git blame config.yml
```

出力結果:
```
^<hash1> (Your Name ... 1) version: 1.1
<hash2> (Your Name ... 2) debug: false
```
-   `^<hash1>`: `^` は、その行が最初のコミット (`<hash1>`) で追加されたことを示す。
-   `<hash2>`: `debug: false` の行は、`<hash2>` のコミットで変更された。
-   `(Your Name ...)`: 変更者の名前と日時。
-   `1)`, `2)`: ファイルの行番号。

この結果から、`debug: false` という行は `<hash2>` のコミット (`feat: Add debug flag`) で追加されたことがわかります。さらに詳しく知りたければ `git show <hash2>` でコミットの詳細を確認し、変更の背景にある文脈を深く理解できます。

---
## 35.3 `git checkout <commit> -- <file>`: 過去のファイルバージョンを復元する

時には、履歴を見るだけでなく、**過去のある時点のファイルそのもの**が必要になることがあります。「間違えて消してしまった設定を復活させたい」「リファクタリング前のロジックと比較したい」といったケースです。

`git checkout` にコミットハッシュとファイルパスを渡すことで、ワーキングディレクトリにその時点のファイルを復元できます。

**シナリオ**: バージョンを `1.0` に戻したい。

`git log -- config.yml` で、バージョンが `1.0` だった最後のコミットは `<hash2>` (`feat: Add debug flag`) だとわかっています。

```bash
# <hash2> の時点の config.yml をワーキングディレクトリに復元
git checkout <hash2> -- config.yml
```

このコマンドを実行すると、ワーキングディレクトリの `config.yml` の中身が `<hash2>` の時点の状態に上書きされます。`git status` を見ると、`config.yml` が変更された状態 (`modified`) になっていることがわかります。

あとは、この復元したファイルをコミットすれば完了です。

```bash
git add config.yml
git commit -m "revert: Revert version to 1.0 due to issues"
```

これにより、プロジェクト全体を巻き戻すことなく、特定のファイルだけを安全に過去の状態に戻すことができました。

---
**まとめ**

| コマンド                            | 何をするか？                                 | 主な用途                                     |
| :---------------------------------- | :------------------------------------------- | :------------------------------------------- |
| `git log -p -- <file>`              | ファイルの変更履歴と差分を一覧する           | ファイル全体の変遷を時系列で理解する         |
| `git blame <file>`                  | 各行の最終変更者とコミットを特定する         | 特定のコード行の背景や意お図を調査する       |
| `git checkout <commit> -- <file>`   | ファイルを過去の特定の状態に復元する         | 過去のバージョンの参照、またはファイルの巻き戻し |

これらのコマンドは、コードの歴史という広大なデータベースを探索するための強力なツールです。使いこなすことで、デバッグやコードリーディングの効率を飛躍的に向上させることができます。
