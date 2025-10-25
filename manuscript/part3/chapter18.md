# 第18章: コンフリクトの解決方法

前章で、私たちは意図的にコンフリクトを発生させ、Gitがマージを中断した状態にあります。`git status` を実行すると、現在の状況を詳しく教えてくれます。

```bash
# git-conflict ディレクトリにいることを確認
git status
```
```
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
	both modified:   greeting.txt

no changes added to commit (use "git add" and/or "git commit -a")
```
Gitは非常に親切です。「コンフリクトを修正して `git commit` してください」と、次に行うべき操作を明確に指示してくれています。この指示に従って、コンフリクトを解決していきましょう。

---
## 18.1 ステップ1: ファイルを手動で編集する

コンフリクト解決の最初のステップは、コンフリクトマーカーが挿入されたファイルを直接編集し、最終的にあなたが望む形に修正することです。

現在の `greeting.txt` の中身を再確認します。
```
<<<<<<< HEAD
Hola, Mundo!
=======
こんにちは、世界！
>>>>>>> feature
```
ここで決断が必要です。
-   `master`ブランチの変更（スペイン語）を採用するのか？
-   `feature`ブランチの変更（日本語）を採用するのか？
-   あるいは、両方を残す、全く新しい内容にする、といった第三の道を選ぶのか？

決定権は完全にあなたにあります。今回は、両方の挨拶を残すという形で解決してみましょう。エディタで `greeting.txt` を開き、コンフリクトマーカーをすべて削除し、以下のように内容を書き換えます。

```
Hola, Mundo! / こんにちは、世界！
```
これで、ファイルはあなたが望む最終的な形になりました。

---
## 18.2 ステップ2: `git add` で解決を伝える

ファイルを修正しただけでは、まだコンフリクトは解決していません。Gitに「このファイルのコンフリクトは、私が手動で解決しました。この内容で確定してください」と伝える必要があります。

そのためのコマンドが `git add` です。

```bash
git add greeting.txt
```
このコマンドを実行すると、Gitの内部では何が起こるのでしょうか？
前章で見た、`index`ファイル内のステージ`1, 2, 3`に分かれていた`greeting.txt`の情報がクリアされ、あなたが作成した解決済みのファイルの内容（`Hola, Mundo! / こんにちは、世界！`）が、単一のステージ`0`のエントリとして新しく登録されます。

`git ls-files --stage` で確認してみましょう。
```
100644 <new_blob_hash> 0	greeting.txt
```
インデックスがコンフリクトのない正常な状態に戻りました。`git status` を見ても、表示が変わっていることがわかります。
```
On branch master
All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:
	modified:   greeting.txt
```
「すべてのコンフリクトは修正されましたが、まだマージの途中です」と表示され、`greeting.txt`がコミット対象としてステージングされていることがわかります。

---
## 18.3 ステップ3: `git commit` でマージを完了する

インデックスの状態が正常に戻れば、あとはマージを完了させるだけです。そのためのコマンドはお馴染みの `git commit` です。

```bash
git commit
```
このとき、コミットメッセージを尋ねるエディタが開きますが、すでに `.git/MERGE_MSG` に保存されていたデフォルトのメッセージ（`Merge branch 'feature'`）が入力されています。そのまま保存してエディタを閉じれば、マージコミットが作成され、コンフリクトの解決プロセスはすべて完了です。

`git log --oneline --graph --all` で最終的な歴史を確認しましょう。
```
*   <hash_merge> (HEAD -> master) Merge branch 'feature'
|\
| * <hash_feature> (feature) Translate greeting to Japanese
* | <hash_master> Translate greeting to Spanish
|/
*   <hash_initial> Initial commit with a greeting
```
コンフリクトを乗り越え、分岐していた歴史が無事に一つに統合されました。

---
## 18.4 (補足) どちらか一方の変更を完全に採用する場合

手動で編集する代わりに、「`master`の変更を正とする」「`feature`の変更を正とする」と完全に決まっている場合は、`git checkout`コマンドでより簡単に解決できます。

-   **`HEAD`（自分）の変更を採用する場合:**
    ```bash
    git checkout --ours greeting.txt
    ```
-   **マージ対象（相手）の変更を採用する場合:**
    ```bash
    git checkout --theirs greeting.txt
    ```
このコマンドは、指定した側のバージョンで作業ディレクトリのファイルを上書きします。その後は同様に `git add greeting.txt` を実行し、`git commit` することでマージを完了できます。

---
**まとめ**

この章では、コンフリクトを解決するための普遍的な3ステップを学びました。

1.  **ファイルを編集する**: コンフリクトマーカーを削除し、コードをあるべき姿に修正する。
2.  **`git add <file>`**: 修正したファイルをステージングし、Gitに「解決済み」と伝える。これにより、`index`ファイルが正常な状態に戻る。
3.  **`git commit`**: マージコミットを作成し、中断していたマージプロセスを完了させる。

この手順さえ覚えておけば、どんなコンフリクトも怖くありません。コンフリクトは、Gitが開発者に意思決定を求めているサインに過ぎないのです。

最後に演習用ディレクトリを削除しておきましょう。
```bash
cd ..
rm -rf git-conflict
```
