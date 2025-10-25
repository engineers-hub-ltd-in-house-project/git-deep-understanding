# 第 31 章: Part5 実践演習: チーム開発シミュレーション

---

この演習では、Part5 で学んだリモートリポジトリの操作と GitHub ワークフローの知識を総動員し、2 人の開発者 (`Alice` と `Bob`) の役割を一人で演じることで、チーム開発の典型的な流れをシミュレートします。

**シナリオ**:
簡単な Web ページの共同開発を行います。
- `Alice`: ページの基本となる `index.html` を作成し、`<h1>` タグを追加する。
- `Bob`: `Alice` の作業と並行して、ページのスタイルを定義する `style.css` を作成し、リンクする。
- 最終的に、`Alice` と `Bob` の両方の作業を `main` ブランチに統合します。

---
## 31.1 準備: リモートリポジトリと 2 人のローカルリポジトリ

まず、シミュレーションの舞台となるリポジトリを準備します。

```bash
# 1. 中央リポジトリ (GitHubの代わり) を作成
git init --bare ../team-project.git

# 2. Aliceのリポジトリをclone
git clone ../team-project.git alice-repo
cd alice-repo
git config user.name "Alice"
git config user.email "alice@example.com"
cd ..

# 3. Bobのリポジトリをclone
git clone ../team-project.git bob-repo
cd bob-repo
git config user.name "Bob"
git config user.email "bob@example.com"
cd ..
```
これで、`team-project.git` というリモートリポジトリと、`Alice`, `Bob` それぞれの作業場ができました。

---
## 31.2 Step1: Alice の作業 (Pull Request #1)

`Alice` の役割を演じます。

```bash
cd alice-repo

# 1. mainブランチで最初のファイルを作成し、push
touch index.html
git add .
git commit -m "Initial commit: Add index.html"
git push origin main

# 2. フィーチャーブランチを作成
git switch -c feature/add-title

# 3. HTMLに見出しを追加してコミット
echo "<h1>Welcome to our page!</h1>" > index.html
git add .
git commit -m "feat: Add H1 title to index"

# 4. リモートにpush
git push -u origin feature/add-title
```
`Alice` は GitHub (のつもり) で、`feature/add-title` から `main` への Pull Request を作成しました。

---
## 31.3 Step2: Bob の作業 (Pull Request #2)

次に `Bob` の役割を演じます。`Bob` は `Alice` の作業を知りません。

```bash
cd ../bob-repo

# 1. 最新のmainブランチを取得
# (重要: AliceがpushしたInitial commitをローカルに反映させる)
git pull origin main

# 2. フィーチャーブランチを作成
git switch -c feature/add-stylesheet

# 3. CSSファイルを作成し、コミット
echo "h1 { color: blue; }" > style.css
git add .
git commit -m "feat: Add stylesheet for H1"

# 4. HTMLファイルにCSSをリンクし、コミット
# (注意: ここで将来のコンフリクトの火種が生まれる)
echo "<link rel=\"stylesheet\" href=\"style.css\">" > index.html
git add .
git commit -m "feat: Link stylesheet in index.html"

# 5. リモートにpush
git push -u origin feature/add-stylesheet
```
`Bob` も GitHub で、`feature/add-stylesheet` から `main` への Pull Request を作成しました。

---
## 31.4 Step3: Alice の PR をマージする

`Alice` の役割に戻ります。`Alice` の PR は `main` とコンフリクトしていないので、レビュー後にマージできます。

```bash
cd ../alice-repo

# GitHub上でAliceのPRがマージされたと仮定する
# (ここでは手動でシミュレート)
git switch main
git merge --no-ff feature/add-title -m "Merge pull request #1 from feature/add-title"
git push origin main

# 不要になったブランチを削除
git branch -d feature/add-title
git push origin --delete feature/add-title
```
これで `main` ブランチには `Alice` の変更 (`<h1>` タグ) が取り込まれました。

---
## 31.5 Step4: Bob の PR のコンフリクトを解決する

`Bob` の役割に戻ります。`Bob` が PR を見ると、`Alice` の変更が先にマージされたため、「This branch has conflicts that must be resolved」と表示されています。
前の章で学んだ手順に従って、`Bob` はコンフリクトを解決します。

```bash
cd ../bob-repo

# 1. ローカルのmainブランチを最新化する
git switch main
git pull origin main

# 2. フィーチャーブランチでrebaseを実行
git switch feature/add-stylesheet
git rebase main
```
`index.html` でコンフリクトが発生し、`rebase` が一時停止します。
ファイルを開くと、このようになっています。
```html
<<<<<<< HEAD
<h1>Welcome to our page!</h1>
=======
<link rel="stylesheet" href="style.css">
>>>>>>> feat: Link stylesheet in index.html
```
`HEAD` (`main` 側の変更) には `h1` タグが、`Bob` のブランチ側の変更には `link` タグがあります。
`Bob` は両方の変更が必要だと判断し、ファイルを以下のように手動で編集します。

```html
<link rel="stylesheet" href="style.css">
<h1>Welcome to our page!</h1>
```

ファイルを修正したら、`rebase` を続行します。
```bash
git add index.html
git rebase --continue
```
`rebase` が正常に完了しました。

### 31.6 Step5: 解決したブランチを Push してマージする

`rebase` によって `Bob` のローカルブランチの歴史は書き換えられたので、`--force-with-lease` を使って `push` します。
```bash
git push --force-with-lease origin feature/add-stylesheet
```
これで GitHub 上の `Bob` の PR はコンフリクトが解消され、マージ可能になります。

最後に、`Alice` の時と同様に、`Bob` の PR も `main` にマージします。
```bash
# GitHub上でBobのPRがマージされたと仮定する
git switch main
git merge --no-ff feature/add-stylesheet -m "Merge pull request #2 from feature/add-stylesheet"
git push origin main

# ブランチの削除
git branch -d feature/add-stylesheet
git push origin --delete feature/add-stylesheet
```

---
**演習完了**
お疲れ様でした！ これで `Alice` と `Bob` 両方の変更が、コンフリクトを乗り越えて `main` ブランチに無事統合されました。
`alice-repo` と `bob-repo` の両方で `git pull` を実行すれば、最終的な状態が同期されていることを確認できます。

この演習は、チーム開発で日常的に発生するブランチの作成、`push`、コンフリクトの解決、マージという一連のプロセスを凝縮したものです。この流れをマスターすることが、チームの一員としてスムーズに開発を進めるための鍵となります。

最後に演習用ディレクトリを削除しておきましょう。
```bash
cd ..
rm -rf team-project.git alice-repo bob-repo
```
