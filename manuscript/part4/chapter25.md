# 第 25 章: Part4 実践演習: Rebase で美しい歴史を作る

---

この演習では、Part4 で学んだ `rebase` と `rebase -i` の知識を総動員して、乱雑な作業履歴を、まるで一本の美しい線で描かれたかのように整形するプロセスを体験します。

**シナリオ**:
あなたは電卓アプリに新しい `divide` (割り算) 機能を追加するタスクを担当しています。開発の途中経過は、試行錯誤やタイポ修正などを含み、少し乱雑なコミット履歴になっています。最終的に `main` ブランチにマージする前に、この履歴を誰が見ても分かりやすい、論理的な形に整えましょう。

---
## 25.1 演習の準備

まず、演習用のリポジトリと、整形前のごちゃっとしたコミット履歴を作成します。

```bash
# 演習用ディレクトリの作成
mkdir git-rebase-exercise && cd git-rebase-exercise
git init
git config user.name "Your Name"
git config user.email "your@email.com"

# mainブランチのセットアップ
echo 'puts 1 + 1' > calculator.rb
git add .
git commit -m "feat: Add add function"

# divide機能開発ブランチの作成
git switch -c feature/divide

# --- ここからが整形対象のコミット履歴 ---

# 1. 機能の基本ロジックを追加 (あとでタイポに気づく)
echo 'puts 10 / 2' > temp_divide.rb
git add .
git commit -m "wip: add devide logic" # "devide" はタイポ

# 2. 作業途中のコミット
echo '# temporary file' >> temp_divide.rb
git add .
git commit -m "wip"

# 3. 最終的なファイルを正しい名前で作成
mv temp_divide.rb divide.rb
git add .
git commit -m "feat: Add divide function"

# 4. mainブランチの変更を取り込む準備として、mainブランチも進めておく
git switch main
echo 'puts 2 - 1' > subtract.rb
git add .
git commit -m "feat: Add subtract function"

# 再び開発ブランチに戻る
git switch feature/divide
```

`git log --oneline --graph --all` で全体の状況を確認しましょう。
```
* <hash3> (HEAD -> feature/divide) feat: Add divide function
* <hash2> wip
* <hash1> wip: add devide logic
| * <hash_sub> (main) feat: Add subtract function
|/
* <hash_add> feat: Add add function
```
`feature/divide` ブランチには 3 つのコミットがあり、その間に `main` ブランチも進んでいます。
私たちのゴールは、この `feature/divide` ブランチの 3 つのコミットを、**「feat: Add divide function」という 1 つの綺麗なコミットにまとめ**、かつ、**最新の `main` ブランチの上に乗せる**ことです。

---
## 25.2 Step1: インタラクティブリベースでコミットをまとめる

まず、`feature/divide` ブランチ内の 3 つのコミットを整理します。
これらのコミットは、`main` ブランチから分岐した `add` 関数のコミット以降に追加されたものです。
`git rebase -i main` を実行して、インタラクティブモードを開始します。

```bash
git rebase -i main
```

エディタが開かれ、以下の内容が表示されます。
```
pick <hash1> wip: add devide logic
pick <hash2> wip
pick <hash3> feat: Add divide function
```

これを、以下のように編集します。
1.  最初のコミット (`<hash1>`) のメッセージを修正したいので `reword` に変更します。
2.  `wip` コミット (`<hash2>`) は、メッセージも内容も最初のコミットに含めたいので `fixup` にします。
3.  3 つ目のコミット (`<hash3>`) も、内容は必要ですがメッセージは不要なので `fixup` にします。

**編集後のファイル**:
```
reword <hash1> wip: add devide logic
fixup <hash2> wip
fixup <hash3> feat: Add divide function
```

ファイルを保存して閉じると、rebase が始まります。
最初に、`reword` を指定したコミットのメッセージ編集画面が開きます。
ここで、タイポを修正し、最終的なコミットメッセージとしてふさわしいものに書き換えます。

**修正後のコミットメッセージ**:
```
feat: Add divide function
```

メッセージを保存してエディタを閉じると、残りの `fixup` が自動的に実行され、rebase が完了します。
`git log --oneline` で `feature/divide` ブランチの歴史を確認しましょう。
```
* <new_hash> (HEAD -> feature/divide) feat: Add divide function
* <hash_add> feat: Add add function
```
3 つのコミットが、1 つの綺麗なコミットにまとまりました！

---
## 25.3 Step2: 通常の Rebase でブランチの土台を更新する

コミットは綺麗になりましたが、まだ `feature/divide` ブランチの土台は古い `main` (`add` 関数のコミット) のままです。
`git log --oneline --graph --all` を見ると、歴史はまだ分岐しています。

```
* <new_hash> (HEAD -> feature/divide) feat: Add divide function
| * <hash_sub> (main) feat: Add subtract function
|/
* <hash_add> feat: Add add function
```

これを最新の `main` の上に移動させるために、通常の `rebase` を実行します。

```bash
git rebase main
```

コンフリクトがなければ、rebase は成功し、`feature/divide` ブランチの土台が `subtract` 関数のコミットの上に移動します。
再度 `git log --oneline --graph --all` で確認します。
```
* <new_hash2> (HEAD -> feature/divide) feat: Add divide function
* <hash_sub> (main) feat: Add subtract function
* <hash_add> feat: Add add function
```
歴史が一直線になりました！

---
## 25.4 Step3: main ブランチにマージする

これで、`feature/divide` ブランチは `main` ブランチにマージする準備が万端です。
`main` ブランチに切り替えて、マージを実行しましょう。

```bash
git switch main
git merge feature/divide
```

`feature/divide` は `main` の直系の歴史になっているため、このマージは必ず **Fast-forward** マージになります。
`main` ブランチのポインタが、`feature/divide` の先端に移動するだけです。

最終的な `git log --oneline --graph` の結果は、一直線で、かつ論理的な単位でコミットが並んだ、非常に美しいものになっているはずです。
```
* <new_hash2> (HEAD -> main, feature/divide) feat: Add divide function
* <hash_sub> feat: Add subtract function
* <hash_add> feat: Add add function
```

---
**まとめ**

この演習を通して、以下のワークフローを実践しました。
1.  フィーチャーブランチでの作業中、コミットは自由な粒度で行う。
2.  `main` ブランチにマージする直前に、`git rebase -i` を使って、関連するコミットを一つの論理的な単位にまとめる (`reword`, `squash`, `fixup`)。
3.  `main` ブランチがその間に進んでいた場合、`git rebase main` を使って、フィーチャーブランチの土台を最新の状態に更新する。
4.  Fast-forward マージで `main` ブランチに統合する。

このワークフローは、`main` ブランチの歴史を常にクリーンで追跡しやすい状態に保つための、非常に強力で実践的なテクニックです。

最後に演習用ディレクトリを削除しておきましょう。
```bash
cd ..
rm -rf git-rebase-exercise
```
