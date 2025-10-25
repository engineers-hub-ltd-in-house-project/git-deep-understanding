# 第31章: チーム開発の実践

おめでとうございます！これで第5部の最終章です。この章は、これまでに学んだリモートリポジトリの操作とPull Requestのワークフローを組み合わせた、総合的な実践演習です。

2人の開発者、あなた（Developer A）と同僚（Developer B）が、簡単なブログの記事を追加していくというシナリオで進めます。この演習を通じて、あなたは日々の開発業務で遭遇するであろう一連の流れを、一人でシミュレートし、体験することができます。

---
## 31.1 準備: 共有リポジトリと2人の開発者

まず、シミュレーションの舞台を整えましょう。
-   `shared-blog.git`: GitHubの役割を果たす中央リポジトリ。
-   `dev-a-repo`: あなた（Developer A）のローカルリポジトリ。
-   `dev-b-repo`: 同僚（Developer B）のローカルリポジトリ。

```bash
# 共有リポジトリ
git init --bare ../shared-blog.git

# Aさん（あなた）のリポジトリ
git clone ../shared-blog.git dev-a-repo
cd dev-a-repo
git config user.name "Developer A"
git config user.email "a@example.com"
echo "# Our Team Blog" > README.md
git add . && git commit -m "Initial commit"
git push origin main
cd ..

# Bさん（同僚）のリポジトリ
git clone ../shared-blog.git dev-b-repo
cd dev-b-repo
git config user.name "Developer B"
git config user.email "b@example.com"
cd ..
```

---
## 31.2 シナリオ開始: 2つの記事の並行執筆

あなた（A）とBさんは、それぞれ別のブログ記事を執筆することになりました。
-   Aさん: `git-is-fun.md` を執筆
-   Bさん: `github-is-useful.md` を執筆

**ステップ1: Bさんが記事を書き上げ、PRをマージする**

まず、Bさんが先行して作業を完了させます。
```bash
cd dev-b-repo
git pull origin main # 念のため最新化
git switch -c feature/github-article
echo "GitHub is a useful platform for collaboration." > github-is-useful.md
git add . && git commit -m "feat: Add GitHub article"
git push origin feature/github-article
```
BさんはGitHub上でPRを作成し、それはすぐにレビューされ、`main`ブランチにマージされたとします。（このマージ操作は、中央リポジトリで直接行うことでシミュレートします）
```bash
cd ../shared-blog.git
git switch main # shared-blogはベアリポジトリなので本当はswitchできないが、概念的な操作
git merge --no-ff remotes/origin/feature/github-article -m "Merge pull request #1 from feature/github-article"
cd ../
```
これで、`main`の歴史はBさんの記事追加によって一つ先に進みました。

**ステップ2: あなたが記事を執筆し、PRを作成する**

その間に、あなた（A）も自分の記事を執筆していました。
```bash
cd dev-a-repo
git pull origin main # 作業開始前に最新化
git switch -c feature/git-article
echo "Git is a fun version control system." > git-is-fun.md
git add . && git commit -m "feat: Add Git article"
git push origin feature/git-article
```
あなたもGitHub上でPRを作成しました。しかし、このPRは`main`ブランチとコンフリクトは起こしていません。なぜなら、Bさんとあなたは**別のファイル**を追加しただけだからです。

このPRもレビューされ、マージの準備が整いました。

---
## 31.3 `main`ブランチの更新とコンフリクトの発生

ここで状況が変わります。Bさんが、すべての記事にフッターを追加する、という共通の変更を`main`ブランチに直接追加しました。（小規模な変更のためPRは作りませんでした）
```bash
cd dev-b-repo
git pull origin main
echo "---" >> github-is-useful.md
echo "Footer" >> github-is-useful.md
git add . && git commit -m "docs: Add footer to GitHub article"
git push origin main
cd ..
```
`main`ブランチはさらに先に進みました。

あなたのPR（`feature/git-article`）は、まだこのフッターの変更を知りません。このままでは、あなたの記事だけフッターがない、という中途半端な状態でマージされてしまいます。

レビュアーが言います。「Aさん、あなたのブランチが古くなっています。`main`の最新の変更を取り込んでからマージしてください。」

---
## 31.4 リベースによるPRの更新とコンフリクト解決

レビュアーの指示に従い、あなたの`feature/git-article`ブランチを`main`の最新版に更新します。今回は、クリーンな歴史を保つためにリベース戦略を選択します。

**ステップ1: 最新の`main`を取得し、リベースする**
```bash
cd dev-a-repo
git switch feature/git-article
git fetch origin

git rebase origin/main
```
`rebase`は成功し、あなたの`git-is-fun.md`を追加するコミットは、Bさんがフッターを追加したコミットの上に移動しました。

**ステップ2: 自分の記事にもフッターを追加する**

リベースしただけでは、あなたの記事にはまだフッターがありません。`main`の変更に追従し、自分の記事にもフッターを追加します。
```bash
echo "---" >> git-is-fun.md
echo "Footer" >> git-is-fun.md
git add .
# --amend を使って、直前のコミットにこの変更を含めてしまう
git commit --amend --no-edit
```
`--amend`と`--no-edit`オプションを使うことで、新しいコミットを作らずに、直前の「feat: Add Git article」のコミットを更新しました。これで、あなたのコミットは「Gitの記事ファイルを追加し、フッターもつける」という完全な一つの単位になりました。

**ステップ3: PRを強制Pushで更新する**

リベースと`--amend`で歴史を書き換えたので、`--force-with-lease`を使ってPRを更新します。
```bash
git push origin feature/git-article --force-with-lease
```
これでPRが更新され、コンフリクトもなく、`main`の最新の変更も反映された状態になりました。レビュアーはこれに満足し、PRは無事にマージされます。

---
## 31.5 演習完了と後片付け

お疲れ様でした！これでチーム開発のシミュレーションは完了です。最後に、ローカル環境をきれいにしましょう。
```bash
cd dev-a-repo
git switch main
git pull origin main
git branch -d feature/git-article
```

---
**まとめ**

この実践演習を通して、あなたは以下の重要なスキルを体験しました。
-   他の開発者の作業と並行して、安全に自分の作業を進めるブランチ戦略。
-   Pull Requestが進行中に、本流の`main`ブランチが更新された場合の対応方法。
-   `git fetch`と`git rebase`を使って、PRを`main`の最新版にクリーンに追従させるテクニック。
-   `--amend`を使って、コミットをより適切な単位にまとめる方法。
-   `--force-with-lease`を使って、書き換えた歴史を安全にリモートに反映させる方法。

これで、第5部の学習はすべて完了です。あなたはもう、GitとGitHubを使ったチーム開発の波に乗りこなす準備ができています。

最後に演習用ディレクトリを削除しておきましょう。
```bash
cd ..
rm -rf shared-blog.git dev-a-repo dev-b-repo
```
