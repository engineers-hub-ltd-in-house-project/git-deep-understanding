# 第29章: Pull Requestの完全ワークフロー

これまでの章で、Gitのローカルでの操作、そしてリモートリポジトリとの通信方法を学びました。理論上は、`main`ブランチで作業し、それを`git push`すればチームとコードを共有できます。しかし、なぜ実際の開発現場では、わざわざ`feature`ブランチを作り、**Pull Request（プルリクエスト、PR）** という一手間をかけるのでしょうか？

答えは**品質とコミュニケーション**のためです。Pull Requestは、あなたの変更点を本流にマージする前に、
-   チームメンバーに変更内容を通知し、
-   コードレビューを通じて潜在的なバグや設計の問題点を議論し、
-   自動テストをパスしていることを確認する

ための「関所」の役割を果たします。これにより、`main`ブランチは常に安定した、品質の高い状態に保たれます。Pull Requestは、Gitのコマンドではなく、GitHubやGitLabといったプラットフォームが提供する、チーム開発を円滑にするための**文化**であり**プロセス**なのです。

この章では、架空のシナリオを通じて、Pull Requestが作成され、レビューされ、マージされるまでの一連の流れを追いかけます。

---
## 29.1 Pull Requestのライフサイクル

Pull Requestの典型的なライフサイクルは、以下のステップで構成されます。
1.  **ブランチの作成とPush**: ローカルで`feature`ブランチを作成し、変更をコミットしてリモートに`push`する。
2.  **Pull Requestの作成**: GitHub上で、`feature`ブランチから`main`ブランチへの変更の取り込みを依頼する。
3.  **レビューとディスカッション**: レビュアーがコードを確認し、コメントや修正依頼を行う。
4.  **修正と再Push**: PRの作成者がフィードバックを元にコードを修正し、同じ`feature`ブランチに`push`する。（PRは自動的に更新される）
5.  **承認とマージ**: レビューが完了し、変更が承認されたら、リポジトリの管理者がPRを`main`ブランチにマージする。
6.  **後片付け**: マージ済みの`feature`ブランチをローカルとリモートで削除する。

---
## 29.2 実践ワークフロー

それでは、このライフサイクルを実際に体験してみましょう。

### ステップ1: ブランチの作成とPush (PR作成者)

まず、リモートリポジトリと、あなたのローカルリポジトリを準備します。
```bash
# リモートリポジトリ
git init --bare ../pr-workflow.git

# ローカルリポジトリ
git clone ../pr-workflow.git my-repo && cd my-repo
echo "Initial file" > file.txt && git add . && git commit -m "Initial"
git push origin main
```
あなたは新しい機能「hello world機能」を実装することになりました。まず、最新の`main`から`feature/hello`ブランチを作成します。
```bash
git switch -c feature/hello main
echo "console.log('hello world');" > hello.js
git add .
git commit -m "feat: Add hello world function"
```
ここで重要なのは、**Pushする前に履歴をきれいにすること**です。第4部で学んだインタラクティブリベースを使い、コミットを意味のある単位にまとめ、メッセージを分かりやすくしておきましょう。（今回は1コミットなので不要です）

準備ができたら、このブランチをリモートに`push`します。
```bash
git push origin feature/hello
```
`push`が成功すると、ターミナルに「Create a pull request for 'feature/hello' on GitHub by visiting: ...」というURLが表示されることがあります。このURLにアクセスするのがPR作成の近道です。

### ステップ2: Pull Requestの作成 (PR作成者)

GitHubのサイトに行き、リポジトリのページを開くと、「`feature/hello` had recent pushes」という通知と共に「Compare & pull request」という緑色のボタンが表示されています。これをクリックします。

PR作成画面では、以下の項目を記述します。
-   **Title**: PRの目的が簡潔にわかるタイトル。（例: `feat: HELLO WORLD機能を追加`）
-   **Description**: なぜこの変更が必要なのか、何をしたのか、どうやってテストしたのか、レビューしてほしい観点などを詳しく記述します。良いPRの डिस्क्रिप्शンは、レビューの質を大きく左右します。

記述したら、「Create pull request」ボタンを押します。これで、あなたの変更提案がチームに公開されました。

### ステップ3 & 4: レビューと修正 (レビュアー & PR作成者)

チームメンバーは、このPRの通知を受け取ります。GitHubの「Files changed」タブを開き、あなたのコードの変更点一行一行に対してコメントをすることができます。

**レビュアー**: 「`console.log`ではなく、`return`で文字列を返す関数にして、テストしやすくしませんか？」

あなたはこのフィードバックを受け、ローカルの`feature/hello`ブランチでコードを修正します。
```bash
# feature/hello ブランチにいることを確認
# hello.js を以下のように修正
# const hello = () => 'hello world';
# module.exports = hello;

git add hello.js
git commit -m "refactor: Return a string instead of logging"
```
修正が完了したら、**同じブランチに再度push**します。
```bash
git push origin feature/hello
```
これにより、既存のPRが自動的に新しいコミットを含んだ状態で更新されます。レビュアーはあなたの修正を再度確認できます。

### ステップ5: 承認とマージ (リポジトリ管理者)

レビュアーが「LGTM (Looks Good To Me)」とコメントし、変更をApprove（承認）します。
すべてのレビューが完了したら、リポジトリの管理者が緑色の「Merge pull request」ボタンを押します。

GitHubのマージボタンには通常3つの選択肢があります。
-   **Create a merge commit**: 通常の`git merge`と同じ。`feature`ブランチの歴史をマージコミットとして`main`に残す。
-   **Squash and merge**: PRの全コミットを1つにまとめてから`main`にマージする。`main`の歴史がクリーンに保たれる。
-   **Rebase and merge**: `feature`ブランチの全コミットを`main`の先端にリベースしてからマージする。`main`の歴史が一直線になる。

チームのルールに従ってマージ方法を選択し、「Confirm merge」を押せば、あなたの変更が晴れて`main`ブランチに統合されます。

### ステップ6: 後片付け (全員)

マージが完了すると、GitHubは不要になったリモートの`feature/hello`ブランチを削除するボタンを表示してくれるので、クリックして削除します。

最後に、あなたのローカル環境もきれいにします。
```bash
# mainブランチに切り替え
git switch main

# リモートの最新のmain（あなたの変更が含まれている）を取得
git pull origin main

# 不要になったローカルのfeatureブランチを削除
git branch -d feature/hello
```
これで、一つの機能開発サイクルが完了しました。

---
**まとめ**

この章では、GitHubを舞台としたPull Requestの完全なワークフローを体験しました。
-   Pull Requestは、コードの品質を担保し、チームのコミュニケーションを促進するための中心的なプロセスである。
-   **Branch -> Commit -> Push -> Create PR -> Review -> Update -> Merge -> Cleanup** という一連のサイクルで開発が進む。
-   PRへの修正は、同じブランチに新しいコミットを追加して`push`するだけで自動的に反映される。
-   PRは、Gitのコマンドの上に構築された、より高度な共同作業のための文化であり、現代の開発に不可欠なスキルである。

最後に演習用ディレクトリを削除しておきましょう。
```bash
cd ..
rm -rf pr-workflow.git my-repo
```
