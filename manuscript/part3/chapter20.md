# 第 20 章: 実践演習（第 3 部）

第 3 部では、分岐した歴史を合流させる `git merge` の世界を探検しました。Fast-forward マージのシンプルさから、Three-way マージの賢い仕組み、そして避けられないコンフリクトの原理と解決方法まで、チーム開発における必須スキルを学びました。

この章では、第 3 部で得た知識を総動員し、架空のチーム開発シナリオを通して、様々なマージの状況を実践的に体験します。

---
## 20.1 シナリオ: 会社のウェブサイト共同開発

あなたは企業のウェブサイトを開発するチームの一員です。`main` ブランチが本番のウェブサイトの内容を管理しており、各開発者は担当のページをフィーチャーブランチで開発し、`main` にマージしていくというルールです。

**演習の登場人物**:
- **あなた**: 「会社概要 (about.html)」ページの作成を担当。
- **同僚**: あなたと並行して「製品情報 (products.html)」ページの作成を担当。（同僚の作業は、私たちが `main` ブランチで直接コミットすることでシミュレートします。）

### プロジェクトのセットアップ
```bash
# 演習用ディレクトリを作成して移動
mkdir website-project && cd website-project
git init

# サイトのホームページを作成
echo "<h1>Welcome to Our Company</h1>" > index.html
git add .
git commit -m "Initial commit: Add homepage"
```
これが、あなたと同僚が開発を始める前の、共通の出発点です。

---
## 20.2 あなたの作業: Fast-forward マージを体験

まず、あなたは自分の担当である「会社概要」ページを作成します。
```bash
# aboutページ用のブランチを作成
git switch -c feature/about-page

# ページを作成してコミット
echo "<h1>About Us</h1>" > about.html
git add .
git commit -m "feat: Create about page"
```
作業が完了したので、`main` ブランチにマージします。この時点では、まだ同僚は作業を終えていない（`main` が更新されていない）ので、Fast-forward マージになるはずです。
```bash
git switch main
git merge feature/about-page
```
出力に "Fast-forward" と表示され、歴史が一直線に進んだことを確認しましょう。`git log --oneline --graph` でも分岐がないことがわかります。

---
## 20.3 同僚の作業とあなたの次の作業: Three-way マージを体験

あなたが `about.html` をマージした後、同僚が `products.html` の作業を完了し、`main` にマージしました。（この作業をシミュレートします）
```bash
# 同僚の作業をシミュレート
echo "<h1>Our Products</h1>" > products.html
git add .
git commit -m "feat: Create products page"
```
`main` ブランチが、あなたが `feature/about-page` を分岐させた時点から一つ進みました。

さて、あなたは次に `about.html` に「沿革」セクションを追加する作業を始めます。
```bash
# 新しい作業のためにブランチを作成
git switch -c feature/add-history

# 沿革セクションを追加
echo "<h2>Our History</h2>" >> about.html
git add .
git commit -m "feat: Add history section to about page"
```
作業が完了したので、`main` にマージしようとします。しかし、今回は `main` の歴史が分岐後（同僚の作業によって）に進んでいるため、Three-way マージが発生します。
```bash
git switch main
git merge feature/add-history
```
エディタが立ち上がり、マージコミットのメッセージを求められるはずです。そのまま保存してマージを完了させましょう。`git log --oneline --graph` で、歴史が分岐し、マージコミットによって合流したことを確認してください。

---
## 20.4 共同作業の宿命: コンフリクトの解決

最後に、あなたと同僚が**同じファイル**を同時に編集してしまったケースを体験します。

`main` ブランチの `index.html` の見出しを、同僚がより具体的に変更しました。
```bash
# 同僚の作業をシミュレート
sed -i 's/Our Company/Our Awesome Company/' index.html
git add .
git commit -m "docs: Update main heading"
```
時を同じくして、あなたも `index.html` の見出しを、マーケティング部の依頼で別の文言に変更する作業をブランチで行っていました。
```bash
# あなたの作業ブランチを作成
git switch -c feature/update-heading

# 見出しを変更
sed -i 's/Our Company/The Best Company Inc./' index.html
git add .
git commit -m "feat: Update heading based on marketing feedback"
```
これでコンフリクトの準備が整いました。`main` にあなたの変更をマージしようとすると、必ずコンフリクトが発生します。
```bash
git switch main
git merge feature/update-heading
```
案の定、コンフリクトが発生しました。
`index.html` を開くと、コンフリクトマーカーが表示されています。
```html
<<<<<<< HEAD
<h1>Welcome to Our Awesome Company</h1>
=======
<h1>Welcome to The Best Company Inc.</h1>
>>>>>>> feature/update-heading
```
チームで話し合った結果、「The Best Company Inc.」を採用し、さらに感嘆符を追加することになりました。ファイルを以下のように編集します。
```html
<h1>Welcome to The Best Company Inc.!</h1>
```
ファイルを修正したら、コンフリクト解決の 3 ステップを思い出してください。
```bash
# ステップ2: git add で解決を通知
git add index.html

# ステップ3: git commit でマージを完了
git commit
```
コミットメッセージを編集・保存して、マージを完了します。

---
**第 3 部のまとめ**

この演習を通して、私たちはチーム開発で日常的に発生する 3 つのマージパターンを体験しました。
- 自分の作業中にベースとなるブランチが進んでいない場合の **Fast-forward マージ**。
- 他のメンバーと並行して開発を進めた結果としての **Three-way マージ**。
- 同じ箇所を編集してしまった場合の**コンフリクトとその解決**。

これらの操作が、`.git` 内部のコミットオブジェクトや `parent` ポインタ、参照、インデックスにどのような影響を与えているのかを意識できましたか？ この内部構造の理解こそが、複雑な状況でも冷静に Git を操作するための鍵となります。

第 4 部では、歴史をより美しく、分かりやすく書き換えるための強力なツール、`git rebase` の世界に足を踏み入れます。

最後に演習用ディレクトリを削除しておきましょう。
```bash
cd ..
rm -rf website-project
```
