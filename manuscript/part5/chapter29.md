# 第 29 章: GitHub ワークフロー: Pull Request の実践

---

ここまでの章で学んだリモートリポジトリの仕組み (`remote`, `fetch`, `push`) は、**GitHub** に代表されるプラットフォーム上で、チームが効果的に協力するための技術的な土台となります。

現代の多くの開発チームでは、**Pull Request (プルリクエスト)** または **Merge Request (マージリクエスト)** と呼ばれる仕組みをワークフローの中心に据えています。これは、単にコードをマージするだけでなく、**コードレビュー**や**自動テスト**、**議論**といったコラボレーションのプロセスを円滑に進めるための非常に強力な機能です。

この章では、GitHub を使った最も一般的で基本的なワークフローを、具体的な手順に沿って実践します。

---
## 29.1 Pull Request ワークフローの全体像

GitHub ワークフローは、一般的に以下のステップで構成されます。

1.  **Issue (課題) の確認**: これから取り組むタスク (機能追加、バグ修正など) が Issue として GitHub 上に登録されていることを確認します。
2.  **ブランチの作成**: `main` (または `develop`) ブランチから、作業内容が分かりやすい名前の**フィーチャーブランチ**をローカルに作成します。(例: `feature/add-login-page`)
3.  **実装とコミット**: ローカルでコードを書き、小さな単位でコミットを積み重ねます。
4.  **Push**: 作成したフィーチャーブランチを、ローカルからリモート (GitHub) に `push` します。
5.  **Pull Request の作成**: GitHub 上で、`push` したフィーチャーブランチから `main` ブランチへのマージを依頼する Pull Request (PR) を作成します。
6.  **コードレビューと議論**: 他のチームメンバーが PR を確認し、コードに対するコメントや質問、改善提案などを行います。必要に応じて、指摘を修正し、再度 `push` します。
7.  **マージ**: レビューで承認 (Approve) されたら、PR を `main` ブランチにマージします。多くの場合、GitHub 上のボタンをクリックするだけで安全にマージが実行されます。
8.  **ブランチの削除**: マージが完了したら、不要になったフィーチャーブランチをリモートとローカルの両方から削除し、クリーンな状態を保ちます。

---
## 29.2 実践: はじめての Pull Request

それでは、簡単なシナリオに沿って、このワークフローを体験してみましょう。

**シナリオ**: プロジェクトの `README.md` に、プロジェクトの概要説明を追加する。

### Step 1 & 2: ブランチの作成

まず、最新の `main` ブランチから作業用のブランチを作成します。

```bash
# 最新のリモートの状態を取得
git fetch origin

# mainブランチを最新の状態に更新
git switch main
git merge origin/main

# フィーチャーブランチを作成して移動
git switch -c feature/add-readme-description
```

### Step 3: 実装とコミット

`README.md` ファイルを編集して、説明文を追加します。
```markdown
# My Awesome Project

This is a sample project to demonstrate the GitHub workflow.
```
変更をコミットします。
```bash
git add README.md
git commit -m "docs: Add project description to README"
```

### Step 4: Push

作成したブランチをリモートリポジトリに `push` します。
初めてそのブランチを `push` する際は、`--set-upstream` (または `-u`) オプションを付けると、ローカルブランチとリモートブランチが紐付けられ、次回から `git push` だけで `push` できるようになります。

```bash
git push --set-upstream origin feature/add-readme-description
```

### Step 5: Pull Request の作成

`push` が成功すると、ターミナルに以下のようなメッセージが表示されることが多いです。
```
remote: Create a pull request for 'feature/add-readme-description' on GitHub by visiting:
remote:   https://github.com/your-username/your-repo/pull/new/feature/add-readme-description
```
この URL にアクセスするか、GitHub のリポジトリページに行くと、「`feature/add-readme-description` had recent pushes」という通知と共に "Compare & pull request" ボタンが表示されています。

このボタンをクリックすると、Pull Request 作成画面が開きます。
- **Title**: PR の目的が簡潔にわかるタイトルを付けます。(通常はコミットメッセージから自動入力されます)
- **Description**: なぜこの変更が必要なのか、どのような変更を加えたのか、レビューしてほしい点などを詳しく記述します。関連する Issue があれば、`#<issue-number>` のように書くとリンクできます。

内容を記述したら、"Create pull request" ボタンをクリックします。

### Step 6 & 7: レビューとマージ

PR が作成されると、チームメンバーに通知が飛びます。
メンバーは "Files changed" タブでコードの差分を確認し、特定の行にコメントを残すことができます。

もし修正依頼があれば、ローカルの `feature/add-readme-description` ブランチでコードを修正し、再度コミットして `push` します。`push` されたコミットは自動的に同じ PR に追加されます。

議論が終わり、レビュアーから "Approve" (承認) をもらったら、"Merge pull request" ボタンをクリックしてマージを実行します。

### Step 8: ブランチの削除

マージ後、GitHub 上でブランチを削除するボタンが表示されるので、クリックしてリモートブランチを削除します。
その後、ローカルでも不要になったブランチを削除し、次の作業に備えます。

```bash
# mainブランチに戻る
git switch main

# リモートで削除されたブランチの情報をローカルに反映
git fetch --prune

# ローカルブランチを削除
git branch -d feature/add-readme-description
```
`--prune` オプションは、リモートで既に存在しない追跡ブランチをローカルから削除してくれる便利なオプションです。

---
**まとめ**

- Pull Request は、コードの変更をチームのメインブランチに統合するための、**提案**であり、**コミュニケーションの場**である。
- `main` ブランチを常に安定した状態に保ちつつ、フィーチャーブランチで安全に開発を進めることができる。
- コードレビューのプロセスを挟むことで、コードの品質向上、知識の共有、チーム全体のスキルアップに繋がる。
- この **Branch -> Push -> Pull Request -> Review -> Merge** という一連の流れは、現代的なチーム開発における最も標準的で効果的なワークフローである。
