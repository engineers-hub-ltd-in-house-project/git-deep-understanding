# Git & GitHub 実践書籍企画書

## 書籍タイトル (案)

**『現場で本当に使える Git & GitHub 実践ガイド』**

副題: 「.git の内部から理解する、コンフリクトに怯えない開発者への道」

**タイトルの特徴:**
- ターゲット読者が明確 (実務経験 2 年以上の開発者)
- 「内部から理解する」アプローチを明示
- 「コンフリクトに怯えない」という実用性
- 「実践」というキーワード
- 現場でよく使うコマンドを深く理解して使いこなす一冊

---

## キャッチコピー案

```
・「マージとリベースの違いが分からない」を解消
・「コンフリクトが怖い」を解消
・「.git の中身が謎」を解消
・「なんとなく使える」で終わらせない深い理解
・1 章 1 概念で確実にステップアップ

こんな人におすすめ:
・Git を使っているが、いざという時に怖い
・マージやリベースが何をしているか分からない
・コンフリクトが発生すると対処できない
・チームに Git を教える立場になった
・「雰囲気で」使うのではなく、本質を理解したい
```

---

## 1. 企画の背景・目的

### 1.1 Git の現状
- バージョン管理システムのデファクトスタンダード
- GitHub を中心とした開発フローが主流
- すべての開発者が使うツールだが、深く理解している人は少ない
- 「なんとなく動いているが、トラブル時に対処できない」という声が多い

### 1.2 既存の Git 入門書の問題点

現在市場にある Git 入門書の多くは、以下の深刻な問題を抱えています:

#### 問題 1: .git の内部構造を説明しない

**具体例: ブランチの説明で**
多くの書籍: 「ブランチは開発の枝分かれです」
図: 棒線が分岐している絵

**問題点:**
- ブランチの実体が何かを説明していない
- `.git/refs/heads/main` がただのテキストファイルであることに触れない
- コミットハッシュを指しているだけという本質を隠蔽
- objects ディレクトリの中身を見せない

**結果:**
- 抽象的な理解しかできない
- トラブル時に対処できない
- 「魔法のツール」として扱うしかない

---

#### 問題 2: コマンドの挙動を .git の変化と結びつけない

**具体例: git add の説明で**

既存書籍の説明:
```bash
git add file.txt  # ステージングエリアに追加
```

「ステージングエリアに追加されました」（図: 四角に矢印）

**問題点:**
- `.git/index` ファイルが更新されることに触れない
- `.git/objects` にブロブオブジェクトが作成されることを説明しない
- `git cat-file` などの配管コマンドで確認する方法を教えない
- 実際に何が起きているか体験できない

**結果:**
- 「おまじない」として覚えるだけ
- トラブル時に調査できない
- 内部の仕組みを理解できない

---

#### 問題 3: マージの種類を曖昧に説明

**具体例: マージの章で**

同じページ (見開き 2 ページ) で以下すべてを説明:
1. Fast-forward マージ
2. Three-way マージ
3. マージコミット
4. Squash マージ
5. リベース
6. **さらに Cherry-pick まで**

**問題点:**
- それぞれがいつ発生するか不明確
- .git の状態変化を説明しない
- 実際にどのコミットグラフになるか体験できない
- 使い分けの基準が曖昧

**結果:**
- すべてが中途半半端な理解
- 現場で適切な方法を選べない
- マージが怖くなる

---

#### 問題 4: コンフリクト解決を表面的に説明

**具体例: コンフリクトの章で**

```bash
$ git merge feature
Auto-merging file.txt
CONFLICT (content): Merge conflict in file.txt
Automatic merge failed; fix conflicts and then commit the result.
```

説明: 「`<<<<<<<` と `=======` と `>>>>>>>` の間を編集してください」

**問題点:**
- なぜコンフリクトが発生したか説明しない
- Three-way マージのアルゴリズムを説明しない
- ベースコミット、ours、theirs の概念を教えない
- `.git/MERGE_HEAD` などの内部状態を見せない
- 実際の修正判断基準を教えない

**結果:**
- コンフリクトが怖い
- 適当に選んでバグを生む
- 自信を持って解決できない

---

#### 問題 5: GUI ツールに頼りすぎる

**具体例: SourceTree の説明に多くのページを割く**

「SourceTree を使えばコマンドを覚えなくても大丈夫！」

**問題点:**
- コマンドラインの理解が疎かになる
- CI/CD、サーバー上での作業ができない
- トラブル時にコマンドで対処できない
- 内部の仕組みを理解する機会を奪う

**結果:**
- GUI に依存しすぎて応用が効かない
- 現場で必要なスキルが身につかない
- 深い理解に到達できない

---

#### 問題 6: GitHub の実践的な使い方を説明しない

**具体例: Pull Request の章で**

「Pull Request を作成しましょう」
スクリーンショット: GitHub の UI

**問題点:**
- PR の目的を説明しない (コードレビュー、CI チェック)
- レビューの具体的な方法を教えない
- レビューコメントへの対応方法を説明しない
- PR のベストプラクティスを示さない
- ローカルでの PR ブランチの操作を説明しない

**結果:**
- PR を作れても、その後が分からない
- レビューができない
- チーム開発で困る

---

#### 問題 7: 「よくあるトラブル」を後回し

**具体例:**
- 第 15 章: Git の基本
- ...
- **第 30 章: トラブルシューティング（最後！）**

**問題点:**
- トラブルは学習の最初から発生する
- エラーメッセージの意味が分からず挫折
- 「後で」では遅すぎる
- 各章でトラブル対処を説明すべき

**結果:**
- 挫折

---

### 1.3 本書の目的

**「なんとなく使える」ではなく「深い理解」を提供し、現場で自信を持って Git を使いこなせる開発者を育成する**

---

### 1.4 市場機会と本書の独自の提案

#### 「習熟した初心者」の壁：開発者スキル市場における重大なギャップ

現代のソフトウェア開発において、Gitは不可欠なツールです。その結果、市場には開発者がGitを使い始めるための無数のリソースが存在します。しかし、これらの教材の大部分は、表層的でコマンド中心のアプローチに終始しています。これは、業界で共通の現象、すなわち「習熟した初心者（Proficient-Beginner）」を生み出す原因となっています。彼らは単純なワークフローには従えますが、実世界のチームコラボレーションで必然的に発生する複雑な状況を乗り切るための基礎知識を欠いています。

この問題の根源は、既存の学習教材が提供する知識の構造にあります。開発者はまず、一連のコマンドを「レシピ」として学びます。しかし、チームに参加し、複雑なリベースのコンフリクト、コミット履歴の整理要求、あるいは誤った`git reset --hard`といった問題に直面した途端、そのレシピベースの知識は破綻します。なぜなら、彼らは問題の診断に必要な、根底にあるメンタルモデルを欠いているからです。

#### 本書の中心的な主張

本書は、「初心者」から「真の中級者/上級者」への飛躍は、より多くのコマンドを学ぶことではなく、**システムを理解すること**にあると主張します。真のGitマスタリーは、有向非巡回グラフ（DAG）を視覚化し、すべてのコマンドが`.git`ディレクトリ内のオブジェクトと参照をどのように操作するかを理解することから生まれる、という前提に基づいています。

---

### 1.5 競合分析とニッチの定義

本書の独自のセールスプロポジション（USP）は、**理論と実践の緊密かつ継続的な統合**です。各章は明確なパターンに従います。

1. **現実的な問題を提示する**（例：「あなたのフィーチャーブランチには15個の乱雑な『WIP』コミットがあります。チームリーダーはマージ前に1つのクリーンなコミットを求めています」）。
2. リポジトリの状態を理解するために**内部構造を探る**。
3. 問題を解決するツールとして**コマンドを紹介する**（例：`git rebase -i`）。
4. **コマンドを実行**し、直後に**内部構造を再調査**して、何がどのように変化したかを正確に確認する。
5. 学習を定着させるための**ハンズオン演習で締めくくる**。

#### 表1：Git技術書の比較分析

| 特徴 | 本書（提案） | 『エンジニアのためのGitの教科書［上級編］』 | 『Pro Git』（公式） | 『動かして学ぶ！ Git入門』 |
| :---- | :---- | :---- | :---- | :---- |
| **対象読者** | 経験豊富な初心者（実務2年以上） | 上級者 | 包括的（全レベル） | 初心者〜中級者 |
| **教育手法** | 問題駆動型、体験的 | 解説的、理論的 | 包括的なリファレンス | コマンド中心、ハンズオン |
| **.git内部構造の解説** | **深く統合的：** 全ての章で解説の核となる | **深く基礎的：** 初期章で優れた解説 | **深いが分離的：** 第10章が内部構造専門 | 表層的 |
| **低レベルコマンドの活用** | **高：** 高レベルコマンドの効果検証に多用 | **中：** オブジェクト生成の説明に使用 | **高：** 内部構造専門の章で使用 | 低/皆無 |
| **実世界のシナリオ** | **高：** 各章が開発上の課題を中心に構成 | **中：** シナリオよりメカニズムに焦点 | **中：** 例はあるがリファレンス形式 | **中：** 基本的なワークフローに焦点 |
| **GitHub/共同作業** | **高：** PR、レビュー、チーム戦略に専門セクション | 低 | 中 | 中 |

---

## 2. ターゲット読者

### 2.1 メインターゲット
- **実務経験 2 年以上の開発者**
  * Git を使っているが、深く理解していない
  * コンフリクトやトラブルに不安がある
  * チームで Git を使っている

- **Git で困った経験がある人**
  * 「なんとなく」使っているが自信がない
  * マージ、リベースで失敗した経験がある
  * もっと深く理解したい

- **チームリーダー・教育担当**
  * メンバーに Git を教える必要がある
  * ベストプラクティスを確立したい
  * トラブル対応できるようになりたい

---

## 3. 本書の特徴 (差別化ポイント)

### 3.1 .git の内部構造を徹底的に可視化

**すべてのコマンドで .git の変化を確認**

**git commit の例:**
```bash
# git commit を実行
$ git commit -m "Add feature"
# コマンド実行後の状態を確認
$ tree .git/objects
# commit オブジェクトの中身を確認
$ git cat-file -p <commit_hash>
# ブランチの実体を確認
$ cat .git/refs/heads/main
```
**効果:**
- コマンドが「何をしているか」が目で見える
- 抽象概念ではなく、具体的なファイルとして理解
- トラブル時に .git を直接調査できる
- 「魔法」ではなく「仕組み」として理解

---

### 3.2 1 章 1 概念の徹底

**絶対に詰め込まない**

**各章の構成:**
1. **なぜこの概念が必要なのか** (動機付け)
2. **実際にやってみる** (ハンズオン)
3. **.git の状態を確認** (内部構造)
4. **前の方法との違い** (比較表)
5. **使い分けの基準** (明確な基準)
6. **よくあるエラーと解決法** (トラブル対処)
7. **練習問題** (手を動かす)

**効果:**
- 確実な理解
- 段階的なスキルアップ
- トラブルに対応できる
- 自信を持って使える

---

### 3.3 配管コマンド (Plumbing Commands) の活用

**「磁器コマンド」だけでなく「配管コマンド」も教える**

本書で使う配管コマンド:
- `git cat-file`: オブジェクトの内容を確認
- `git ls-files`: インデックスの内容を確認
- `git rev-parse`: 参照の解決
- `git update-ref`: 参照の更新
- `git hash-object`: オブジェクトのハッシュ計算

**効果:**
- Git の仕組みが理解できる
- トラブル時に調査できる
- 「魔法」ではなくなる
- 深い理解に到達

---

## 4. 目次構成案

### 第 1 部: Git の内部構造を理解する (第 1-7 章)

**第 1 章: Git とは何か**
- **目的:** Gitの3つの核となるオブジェクト（blob, tree, commit）の強固なメンタルモデルを、一つの変更のライフサイクルを手動で追跡することによって構築する。
- **演習：「Git探偵」:** 読者には単純なリポジトリが与えられ、低レベルな「plumbing」コマンド（cat-file, ls-files）のみを使用して、最後のコミットの内容、それに含まれるファイル、そして作者名を特定してもらいます。

**第 2 章: .git ディレクトリの全体像**
- **目的:** ブランチがコードベースの重厚なコピーであるという一般的な誤解を解体する。ブランチが単純で軽量なポインタであることを明らかにする。
- **演習：「ブランチジャグリング」:** 読者は複数のブランチを作成し、コミットを行い、その後.git/refs/headsと.git/HEADのファイルを見るだけでコミットグラフを描き、現在アクティブなブランチを特定します。

**第 3 章: Git オブジェクト（blob）**
**第 4 章: Git オブジェクト（tree）**
**第 5 章: Git オブジェクト（commit）**
**第 6 章: 参照（refs）とブランチ**
**第 7 章: ステージングエリアとインデックス**

---

### 第 2 部: 基本操作を内部から理解する (第 8-14 章)

**第 8 章: git add の完全理解**
**第 9 章: git commit の完全理解**
**第 10 章: git status の読み方**
**第 11 章: git log で履歴を確認**
**第 12 章: ブランチの作成と切り替え**
**第 13 章: git diff で差分を確認**
**第 14 章: 実践演習（第 1 部 + 第 2 部）**

---

### 第 3 部: マージとコンフリクト (第 15-20 章)

- **目的:** 3つの主要なマージ戦略について、明確で視覚的、かつ実践的なガイドを提供し、開発者が状況に応じて適切な戦略を選択できるようにする。
- **演習：「マージマスター」:** 3つのフィーチャーブランチを持つリポジトリを提供します。読者は1つをファストフォワード、もう1つを3方向マージ、3つ目をスカッシュ＆マージで統合し、結果として生じる`git log --graph`の違いを説明しなければなりません。

**第 15 章: Fast-forward マージ**
**第 16 章: Three-way マージ**
**第 17 章: コンフリクトの発生原理**
**第 18 章: コンフリクトの解決方法**
**第 19 章: マージツールの使い方**
**第 20 章: マージの実践演習**

---

### 第 4 部: リベースと履歴の書き換え (第 21-25 章)

- **目的:** クリーンで直線的な歴史を作成するためのツールとしてrebaseを解明し、同時にリポジトリを破壊するようなミスを防ぐための「黄金律」を強く強調する。
- **演習：「PRクリーンアップ」:** 読者には乱雑な歴史を持つフィーチャーブランチと、要求される変更のチェックリストが与えられます。彼らは「レビューの準備が整う」前に、インタラクティブリベースを使ってブランチをクリーンアップしなければなりません。

**第 21 章: リベースの基本**
**第 22 章: インタラクティブリベース**
**第 23 章: Cherry-pick**
**第 24 章: リベースの注意点**
**第 25 章: 履歴の書き換え実践演習**

---

### 第 5 部: リモートリポジトリと GitHub (第 26-31 章)

**第 26 章: リモートリポジトリの仕組み**
**第 27 章: fetch vs pull**
**第 28 章: push の仕組み**
**第 29 章: Pull Request の完全ワークフロー**
**第 30 章: PR でのコンフリクト解決**
**第 31 章: チーム開発の実践**

---

### 第 6 部: トラブルシューティングと高度なトピック (第 32-36 章)

- **目的:** reflogを究極のローカルセーフティネットとして紹介し、開発者がほとんどすべてのミスから回復できるという自信を持って実験できるようにする。
- **演習：「災害復旧」:** 読者は意図的にローカルリポジトリをいくつかの方法で「破壊」します。その後、git reflogのみを使用してリポジトリを元の状態に復元しなければなりません。

**第 32 章: よくあるエラーと解決法**
**第 33 章: コミットの修正・取り消し**
**第 34 章: ファイルの履歴を辿る**
**第 35 章: サブモジュールとサブツリー**
**第 36 章: Git の高度な機能**

---

## 5. 執筆の鉄則

### 5.1 絶対にやってはいけないこと
1. **内部構造を見せない**: 必ず `.git` の変化を確認
2. **複数の概念を同時に説明**: 1 章 1 概念
3. **GUI ツールに頼る**: コマンドラインが基本
4. **「雰囲気で」「おまじない」**: すべてに「なぜ」を説明
5. **エラー対処を後回し**: 各章で「よくあるエラー」セクション

### 5.2 必ずやること
1. **.git の変化を必ず確認**
2. **なぜその概念が必要か説明**
3. **使い分けの基準を明示**
4. **豊富な練習問題**
5. **エラーメッセージの解説**

---
## 6. 引用文献

1. Gitの学習におすすめの本12選｜自分に合う本を見つけるためのポイントを解説, 10月 23, 2025にアクセス、 [https://web-camp.io/magazine/archives/110419/](https://web-camp.io/magazine/archives/110419/)
2. 【2023年】Git/GitHub本「人気20冊と高評価・おすすめの5冊」 | Techs Life, 10月 23, 2025にアクセス、 [https://freelifetech.com/git-books/](https://freelifetech.com/git-books/)
3. 【初学者の私が選ぶ、学習の味方】Git\&GitHub ２冊の教本を比較してみた - Zenn, 10月 23, 2025にアクセス、 [https://zenn.dev/pechan/articles/6e480f90542773](https://zenn.dev/pechan/articles/6e480f90542773)
4. Gitの基本がまるっと分かる！おすすめ学習書籍6選【熟練度別に紹介】 - 侍エンジニア, 10月 23, 2025にアクセス、 [https://www.sejuku.net/blog/258566](https://www.sejuku.net/blog/258566)
5. 「エンジニアのためのgit教科書」からgitの内部構造を学ぶ 実践編 ..., 10月 23, 2025にアクセス、 [https://blog.chaspy.me/entry/2017/08/10/120000](https://blog.chaspy.me/entry/2017/08/10/120000)
6. Removing sensitive data from a repository - GitHub Docs, 10月 23, 2025にアクセス、 [https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository)
7. 「Git・GitHub」おすすめ書籍6選＋α【初心者用4冊＆実践用２冊】, 10月 23, 2025にアクセス、 [https://howahowablog.com/git_github_books_recommendation6/](https://howahowablog.com/git_github_books_recommendation6/)
8. Gitの内部構造をよく理解して、うまく使おう【基本の仕組みを解説 ..., 10月 23, 2025にアクセス、 [https://codezine.jp/article/detail/17328](https://codezine.jp/article/detail/17328)
9. Squash, Merge, or Rebase? - Matt Rickard, 10月 23, 2025にアクセス、 [https://mattrickard.com/squash-merge-or-rebase](https://mattrickard.com/squash-merge-or-rebase)
10. Your team's PR merge strategy: rebase, squash, merge? Why? : r/developpeurs - Reddit, 10月 23, 2025にアクセス、 [https://www.reddit.com/r/developpeurs/comments/1j40gx8/votre\_strat%C3%A9gie\_de\_merge\_de\_prs\_dans\_votre\_%C3%A9quipe/?tl=en](https://www.reddit.com/r/developpeurs/comments/1j40gx8/votre_strat%C3%A9gie_de_merge_de_prs_dans_votre_%C3%A9quipe/?tl=en)
11. What is the difference between merge --squash and rebase? - Stack Overflow, 10月 23, 2025にアクセス、 [https://stackoverflow.com/questions/2427238/what-is-the-difference-between-merge-squash-and-rebase](https://stackoverflow.com/questions/2427238/what-is-the-difference-between-merge-squash-and-rebase)
12. git rebase | Atlassian Git Tutorial, 10月 23, 2025にアクセス、 [https://www.atlassian.com/git/tutorials/rewriting-history/git-rebase](https://www.atlassian.com/git/tutorials/rewriting-history/git-rebase)
13. Git Rebase: A Beginner's Guide to Streamlined Version Control - DataCamp, 10月 23, 2025にアクセス、 [https://www.datacamp.com/tutorial/git-rebase](https://www.datacamp.com/tutorial/git-rebase)
14. Beginner's Guide to Interactive Rebasing | HackerNoon, 10月 23, 2025にアクセス、 [https://hackernoon.com/beginners-guide-to-interactive-rebasing-346a3f9c3a6d](https://hackernoon.com/beginners-guide-to-interactive-rebasing-346a3f9c3a6d)
15. Using Git rebase on the command line - GitHub Docs, 10月 23, 2025にアクセス、 [https://docs.github.com/en/get-started/using-git/using-git-rebase-on-the-command-line](https://docs.github.com/en/get-started/using-git/using-git-rebase-on-the-command-line)
16. Merging vs. Rebasing | Atlassian Git Tutorial, 10月 23, 2025にアクセス、 [https://www.atlassian.com/git/tutorials/merging-vs-rebasing](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)
17. Git Cherry Pick | Atlassian Git Tutorial, 10月 23, 2025にアクセス、 [https://www.atlassian.com/git/tutorials/cherry-pick](https://www.atlassian.com/git/tutorials/cherry-pick)
18. Mastering Git Cherry Pick: A Comprehensive Guide to Selective Commit Management, 10月 23, 2025にアクセス、 [https://algocademy.com/blog/mastering-git-cherry-pick-a-comprehensive-guide-to-selective-commit-management/](https://algocademy.com/blog/mastering-git-cherry-pick-a-comprehensive-guide-to-selective-commit-management/)
19. The Git Week: A Guide to Git Revert, Reset and Checkout | by Lorenzo Uriel - Medium, 10月 23, 2025にアクセス、 [https://medium.com/@lorenzouriel/the-git-week-a-guide-to-git-revert-reset-and-checkout-da103b119b17](https://medium.com/@lorenzouriel/the-git-week-a-guide-to-git-revert-reset-and-checkout-da103b119b17)
20. Difference Between Git Revert, Checkout and Reset - GeeksforGeeks, 10月 23, 2025にアクセス、 [https://www.geeksforgeeks.org/git/git-difference-between-git-revert-checkout-and-reset/](https://www.geeksforgeeks.org/git/git-difference-between-git-revert-checkout-and-reset/)
21. Resetting, Checking Out & Reverting | Atlassian Git Tutorial, 10月 23, 2025にアクセス、 [https://www.atlassian.com/git/tutorials/resetting-checking-out-and-reverting](https://www.atlassian.com/git/tutorials/resetting-checking-out-and-reverting)
22. The Difference Between Git Reset, Git Checkout, and Git Revert - GreenGeeks, 10月 23, 2025にアクセス、 [https://www.greengeeks.com/tutorials/the-difference-between-git-reset-git-checkout-and-git-revert/](https://www.greengeeks.com/tutorials/the-difference-between-git-reset-git-checkout-and-git-revert/)
23. What's the difference between Git Revert, Checkout and Reset? - Stack Overflow, 10月 23, 2025にアクセス、 [https://stackoverflow.com/questions/8358035/whats-the-difference-between-git-revert-checkout-and-reset](https://stackoverflow.com/questions/8358035/whats-the-difference-between-git-revert-checkout-and-reset)
24. What is the Git Reflog? | Learn Version Control with Git - Git Tower, 10月 23, 2025にアクセス、 [https://www.git-tower.com/learn/git/faq/what-is-git-reflog](https://www.git-tower.com/learn/git/faq/what-is-git-reflog)
25. Git Reflog: Understanding and Using Reference Logs in Git - DataCamp, 10月 23, 2025にアクセス、 [https://www.datacamp.com/tutorial/git-reflog](https://www.datacamp.com/tutorial/git-reflog)
26. A Comprehensive Guide to Git's Reflog Command - StarAgile, 10月 23, 2025にアクセス、 [https://staragile.com/blog/git-reflog](https://staragile.com/blog/git-reflog)
27. git-filter-branch Documentation - Git, 10月 23, 2025にアクセス、 [https://git-scm.com/docs/git-filter-branch](https://git-scm.com/docs/git-filter-branch)
28. Remove a file from the repository history when there are multiple files with the same name in Bitbucket Cloud - Atlassian Support, 10月 23, 2025にアクセス、 [https://support.atlassian.com/bitbucket-cloud/kb/how-to-remove-a-file-from-the-repository-history-when-there-are-multiple-files-with-the-same-name/](https://support.atlassian.com/bitbucket-cloud/kb/how-to-remove-a-file-from-the-repository-history-when-there-are-multiple-files-with-the-same-name/)
29. Rewriting a Git repo to remove secrets from the history - Simon Willison: TIL, 10月 23, 2025にアクセス、 [https://til.simonwillison.net/git/rewrite-repo-remove-secrets](https://til.simonwillison.net/git/rewrite-repo-remove-secrets)
30. 作成拉取請求- GitHub 文档, 10月 23, 2025にアクセス、 [https://docs.github.com/zh/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request](https://docs.github.com/zh/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request)
31. 关于拉取请求- GitHub 文档, 10月 23, 2025にアクセス、 [https://docs.github.com/zh/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests](https://docs.github.com/zh/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests)
32. 拉取請求文档 - GitHub Docs, 10月 23, 2025にアクセス、 [https://docs.github.com/zh/pull-requests](https://docs.github.com/zh/pull-requests)
33. git-rebase Documentation - Git, 10月 23, 2025にアクセス、 [https://git-scm.com/docs/git-rebase](https://git-scm.com/docs/git-rebase)
34. 実践的なGitブランチ戦略入門 〜チーム開発を円滑に進めるための ..., 10月 23, 2025にアクセス、 [https://minna-systems.co.jp/blogs/3043/](https://minna-systems.co.jp/blogs/3043/)
35. Gitflow 実践ガイド：安定したリリースと効率的な開発を実現するブランチ戦略 - izanami, 10月 23, 2025にアクセス、 [https://izanami.dev/post/16113b2f-906f-4b30-8ecb-59400090b906](https://izanami.dev/post/16113b2f-906f-4b30-8ecb-59400090b906)
