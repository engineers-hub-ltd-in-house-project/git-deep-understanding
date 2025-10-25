# 第 23 章: チェリーピック `cherry-pick`

---

`git rebase` がブランチ全体のコミットを移動・再適用するのに対し、**`git cherry-pick`** は、**特定のコミットだけを一つずつ摘み取って (pick)、現在のブランチに適用する**ためのコマンドです。

果物の木から、熟した美味しいサクランボ (cherry) だけを選んで摘み取る様子に似ていることから、この名前がついています。

---
## 23.1 チェリーピックのユースケース

`cherry-pick` は、以下のような状況で非常に役立ちます。

- **緊急のバグ修正**: `develop` ブランチで作業していたバグ修正コミットを、急遽 `main` ブランチにも適用する必要が出た。ブランチ全体をマージするのではなく、そのバグ修正コミットだけを `main` に持ってきたい。
- **機能の一部だけを先行リリース**: ある `feature` ブランチで複数の機能を開発していたが、そのうちの一つの機能だけを先にリリースすることになった。その機能に関するコミットだけをリリース用のブランチに適用したい。
- **間違ったブランチへのコミット**: 本来 `feature-A` ブランチで行うべきだったコミットを、間違って `feature-B` ブランチにしてしまった。そのコミットだけを `feature-A` に移動させたい。

---
## 23.2 `cherry-pick` の実行

`cherry-pick` の基本的な使い方は非常にシンプルです。
まず、適用したいコミットがあるブランチ (`bug-fix` ブランチなど) のコミットハッシュを `git log` で確認します。

```bash
# 実験用ディレクトリを作成
mkdir cherry-pick-practice && cd cherry-pick-practice
git init

# mainブランチでコミット
echo "v1" > file.txt && git add . && git commit -m "v1"

# bug-fixブランチで2つのコミット
git switch -c bug-fix
echo "fix!" >> file.txt && git add . && git commit -m "Fix critical bug"
echo "refactor" >> file.txt && git add . && git commit -m "Refactor code"

# git logでコミットハッシュを確認
git log --oneline
# <hash_refactor> Refactor code
# <hash_fix> Fix critical bug
# <hash_v1> v1
```

今、`main` ブランチには `v1` しかありません。この `main` ブランチに、`bug-fix` ブランチの `Fix critical bug` コミットだけを適用したいとします。

**手順**:
1. コミットを適用したい先のブランチ (`main`) に移動します。
2. `git cherry-pick <コミットハッシュ>` を実行します。

```bash
git switch main
git cherry-pick <hash_fix> # "Fix critical bug" のコミットハッシュを指定
```

実行後、`git log --oneline` で `main` ブランチの歴史を確認してみましょう。
```
* <hash_fix_prime> (HEAD -> main) Fix critical bug
* <hash_v1> v1
```
`main` ブランチに `Fix critical bug` コミットが適用されました。
`bug-fix` ブランチの `Refactor code` コミットは適用されていません。

---
## 23.3 `cherry-pick` の内部動作と注意点

`cherry-pick` の内部動作は、`rebase` と非常によく似ています。

指定されたコミット (`<hash_fix>`) とその親コミット (`<hash_v1>`) の差分 (patch) を計算し、その差分を現在のブランチ (`main`) の `HEAD` に適用して、**新しいコミットを作成します**。

そのため、`main` ブランチに作られた `<hash_fix_prime>` は、元の `<hash_fix>` と**コミットハッシュが異なります**。変更内容は同じですが、親コミットが異なるため、別のコミットとして扱われます。

この性質のため、後で `bug-fix` ブランチを `main` ブランチにマージしようとすると、同じ変更が二重に適用されているため、Git は歴史の解釈に混乱し、不要なコンフリクトを引き起こす可能性があります。

`cherry-pick` は便利な反面、歴史を複雑にする可能性があるため、緊急時や限定的な状況で慎重に使うべきコマンドと言えます。

---
## 23.4 コンフリクトの解決

`cherry-pick` しようとした変更が、現在のブランチの内容と衝突する場合、`rebase` や `merge` と同様にコンフリクトが発生します。

```
error: could not apply <hash_fix>... Fix critical bug
hint: after resolving the conflicts, mark the corrected paths
hint: with 'git add <paths>' or 'git rm <paths>'
hint: and commit the result with 'git commit'
```
このメッセージは少し不親切で、`git commit` を実行するように指示していますが、`cherry-pick` のコンフリクト解決後は、通常 `git cherry-pick --continue` を使います。

**解決手順**:
1. コンフリクトが発生したファイルを修正する。
2. `git add <file>` で解決したことを Git に伝える。
3. `git cherry-pick --continue` でプロセスを続行する。

もし解決を中断したい場合は `git cherry-pick --abort` を、コンフリクトしたコミットの適用を諦める場合は `git cherry-pick --skip` を使うことができます。

---
**まとめ**

- `git cherry-pick` は、他のブランチにある特定のコミットだけを選んで、現在のブランチに適用するコマンドである。
- 緊急のバグ修正など、ブランチ全体をマージせずに一部の変更だけを取り込みたい場合に便利。
- 内部的には、指定されたコミットの差分を適用して新しいコミットを作成するため、元のコミットとはハッシュ値が異なる。
- 歴史を複雑にする可能性があるため、利用は慎重に行うべきである。
- コンフリクトが発生した場合は、ファイルを修正し、`git add` してから `git cherry-pick --continue` で続行する。

最後に実験用ディレクトリを削除しておきましょう。
```bash
cd ..
rm -rf cherry-pick-practice
```
