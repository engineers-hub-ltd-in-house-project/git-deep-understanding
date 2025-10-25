# 第11章: `git log` で履歴を確認

`git add` でインデックスを更新し、`git commit` でリポジトリに歴史を刻み、`git status` で現在地を確認する。このサイクルを回していくと、リポジトリにはコミットが積み重なっていきます。この積み重なった歴史を旅するためのコマンドが `git log` です。

`git log` は、内部的には非常にシンプルな動きをしています。

1.  指定された始点（デフォルトでは `HEAD`）からスタートします。
2.  そのコミットの情報を表示します。
3.  そのコミットが持つ `parent` ポインタを辿り、親コミットに移動します。
4.  歴史の始まり（親を持たない最初のコミット）にたどり着くまで、2と3を繰り返します。

この章では、この基本的な動きを理解した上で、`git log` をより強力な歴史分析ツールに変えるための様々なオプションを見ていきましょう。

---

## 11.1 `log` の出力を整形する

まずは、ログの出力を整形し、見やすくするためのオプションです。実験用に、いくつかコミットがあるリポジトリを準備しましょう。

```bash
# 実験用ディレクトリを作成して移動
mkdir git-log-practice && cd git-log-practice
git init

# 3つのコミットを作成
echo "v1" > file.txt && git add . && git commit -m "v1"
echo "v2" > file.txt && git add . && git commit -m "v2"
echo "v3" > file.txt && git add . && git commit -m "v3"
```

### 基本の `git log`

オプションなしで実行すると、各コミットの詳細情報が表示されます。
```bash
git log
```
出力結果（例）:
```
commit a1b2c3d... (HEAD -> master)
Author: Your Name <you@example.com>
Date:   ...

    v3

commit e4f5g6h...
Author: Your Name <you@example.com>
Date:   ...

    v2

...
```
これらはすべて、第5章で学んだ `commit` オブジェクトのメタデータそのものです。

### `--oneline` と `--graph`

最もよく使われるオプションの一つが `--oneline` です。コミットを1行に要約して表示してくれます。
```bash
git log --oneline
```
出力結果（例）:
```
a1b2c3d (HEAD -> master) v3
e4f5g6h v2
i7j8k9l v1
```

ブランチが分岐・合流している場合は `--graph` オプションが非常に役立ちます。歴史の流れをアスキーアートで視覚化してくれます。
```bash
# featureブランチを作成してコミット
git branch feature
git checkout feature
echo "feature" > feature.txt && git add . && git commit -m "add feature"

# masterに戻ってマージ
git checkout master
git merge --no-ff feature -m "merge feature"

# グラフ付きでログを表示
git log --oneline --graph
```
出力結果（例）:
```
*   b1c2d3e (HEAD -> master) merge feature
|\  
| * f2g3h4i (feature) add feature
|/  
* a1b2c3d v3
* e4f5g6h v2
* i7j8k9l v1
```

### `--pretty=format:"..."` (カスタムフォーマット)

`--pretty=format` を使えば、出力を完全に自作できます。これは、Gitの内部データを直接扱っている感覚を味わえる強力なオプションです。

```bash
git log --pretty=format:"%h %an, %ar: %s"
```
出力結果（例）:
```
b1c2d3e Your Name, 3 minutes ago: merge feature
f2g3h4i Your Name, 5 minutes ago: add feature
a1b2c3d Your Name, 7 minutes ago: v3
...
```
- `%h`: 短縮されたコミットハッシュ
- `%an`: Author名
- `%ar`: Authorの日付 (相対的)
- `%s`: コミットメッセージの件名

このように、`commit` オブジェクトが持つ情報を自由に組み合わせて表示できます。

---
## 11.2 ログをフィルタリングする

`git log` は、ただ表示するだけでなく、特定の条件に合致するコミットだけを絞り込んで表示することもできます。

### `-n <数>` (数で絞り込む)
直近のN件のコミットだけを表示します。
```bash
git log -n 3
```

### 時間で絞り込む
`--since` (または `--after`) と `--until` (または `--before`) を使って、期間を指定できます。
```bash
# 2週間前からのログを表示
git log --since="2 weeks"

# 特定の日付範囲のログを表示
git log --since="2023-01-01" --until="2023-01-31"
```

### `--author` と `--grep` (人で絞り込む / 内容で絞り込む)
特定の作者や、コミットメッセージに特定の文字列を含むコミットを検索できます。
```bash
# 特定の作者のコミットを表示 (部分一致)
git log --author="Yusuke"

# コミットメッセージに "fix" を含むものを検索
git log --grep="fix"
```

### `<ファイルパス>` (パスで絞り込む)
`git log` の最後にファイルパスやディレクトリパスを指定すると、そのパスに変更を加えたコミットだけが表示されます。
```bash
# file.txt に関連するコミットだけを表示
git log file.txt

# src/ ディレクトリに関連するコミットだけを表示
git log src/
```
これは、特定のファイルの変更履歴を追跡する際に非常に便利です。Gitは各コミットの`tree`とその親の`tree`を比較することで、どのファイルが変更されたかを判断し、このフィルタリングを実現しています。

---
**まとめ**

この章では、`git log` がコミットの `parent` ポインタを辿ることで歴史を表示していること、そして多彩なオプションによってその出力を自在に操れることを学びました。

-   `git log` は `HEAD` から始まり、親を辿って歴史を遡る。
-   `--oneline` や `--graph` は、ログを視覚的に分かりやすくする。
-   `--pretty=format` を使えば、`commit` オブジェクトの情報を元に自由なフォーマットを定義できる。
-   時間、作者、内容、パスなど、様々な条件でコミットをフィルタリングし、必要な情報だけを効率的に抽出できる。

`git log` を使いこなすことは、過去の変更の意図を理解し、バグの原因を特定し、チームメンバーの作業を把握するための必須スキルです。

次の章では、ブランチの作成と切り替えを、内部構造と結びつけながらさらに深く探求します。

最後に実験用ディレクトリを削除しておきましょう。
```bash
cd ..
rm -rf git-log-practice
```
