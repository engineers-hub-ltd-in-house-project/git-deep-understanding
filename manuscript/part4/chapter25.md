# 第25章: 履歴の書き換え実践演習

この章は、第4部「リベースと履歴の書き換え」の集大成です。`rebase`、`rebase -i`、`cherry-pick`という強力なツールを組み合わせ、実践的なシナリオでコミット履歴を美しく整形する一連のワークフローを体験します。

この演習のゴールは、**Pull Requestをレビューしてもらう前に、同僚が理解しやすいようにコミット履歴を整える**スキルを身につけることです。

---
## 25.1 演習シナリオと準備

あなたは電卓アプリケーションの開発をしています。現在、`main`ブランチには足し算と引き算の機能が実装されています。あなたのタスクは、割り算機能を追加することです。しかし、開発の途中で様々な試行錯誤があり、コミット履歴は少し乱雑になってしまいました。

まず、演習用のリポジトリを準備します。
```bash
mkdir calculator-app && cd calculator-app
git init

# mainブランチの準備
git config --local user.name "You"
git config --local user.email "you@example.com"

echo "const add = (a, b) => a + b;" > calculator.js
git add .
git commit -m "feat: Add 'add' function"

echo "const subtract = (a, b) => a - b;" >> calculator.js
git add .
git commit -m "feat: Add 'subtract' function"

# 別の開発者がREADMEを更新したと仮定
echo "# Calculator App" > README.md
git add .
git commit -m "docs: Add README file"
```
これで`main`ブランチの準備ができました。

次に、あなたが割り算機能を開発するために作成した、少し乱雑な`feature/divide`ブランチの状況を再現します。
```bash
# 2つ前のコミットからブランチを作成
git switch -c feature/divide HEAD~1

# 試行錯誤のコミット履歴
echo "// divide function" >> calculator.js && git add . && git commit -m "wip: start divide"
echo "const divide = (a, b) => a / b;" >> calculator.js && git add . && git commit -m "feat: add basic divide logic"
# コミットメッセージのタイポに注目
echo "// Add zero check" >> calculator.js && git add . && git commit -m "fix: Add devide by zero check"
sed -i 's/a \/ b/b === 0 ? "Error" : a \/ b/' calculator.js && git add . && git commit -m "refactor: Improve zero check"
```
`git log --oneline --graph`で`feature/divide`ブランチの現状を確認します。
```
* <hash4> refactor: Improve zero check
* <hash3> fix: Add devide by zero check
* <hash2> feat: add basic divide logic
* <hash1> wip: start divide
* <hash_subtract> feat: Add 'subtract' function
...
```
この履歴はレビューしてもらうには少し不親切です。これをきれいにしていきましょう。

---
## 25.2 ステップ1: インタラクティブリベースで履歴を整形する

最初のステップは、`feature/divide`ブランチのローカルなコミットを整理することです。`main`ブランチとの分岐点（`subtract`コミット）以降の4つのコミットを編集対象とします。
```bash
git rebase -i main
```
エディタが開いたら、以下の計画に従って編集します。
1.  最初の`wip`コミットを、次の`basic divide logic`に`fixup`で統合する。
2.  `devide`のタイポを`reword`で修正する。
3.  `Improve zero check`を、直前のコミットに`squash`で統合し、メッセージを一つにまとめる。

編集後のファイルは以下のようになります。
```
pick <hash1> wip: start divide
fixup <hash2> feat: add basic divide logic
reword <hash3> fix: Add devide by zero check
squash <hash4> refactor: Improve zero check
```
おっと、計画と少し違いますね。`fixup`は`pick`の後に来る必要があります。また、`wip`を`pick`する意味はありません。順序を入れ替えて、コマンドを修正しましょう。

```
pick <hash2> feat: add basic divide logic
fixup <hash1> wip: start divide
reword <hash3> fix: Add devide by zero check
squash <hash4> refactor: Improve zero check
```
これを保存してエディタを閉じると、リベースが始まります。
1.  Gitはまず`hash2`と`hash1`を`fixup`でまとめます。
2.  次に、`hash3`の`reword`のため、エディタが開きます。メッセージを`fix: Add divide by zero check`に修正します。
3.  最後に、`hash4`を`squash`するため、再度エディタが開きます。2つのコミットメッセージが表示されるので、1つのクリーンなメッセージ「feat: Add 'divide' function with zero check」にまとめます。

すべてのプロセスが完了すると、`git log --oneline --graph`の歴史は以下のようになります。
```
* <new_hash> feat: Add 'divide' function with zero check
* <hash_subtract> feat: Add 'subtract' function
...
```
4つのコミットが、意味のある1つのコミットにまとまりました。

---
## 25.3 ステップ2: 最新の`main`にリベースする

コミットの整理が終わりましたが、作業中に`main`ブランチは`docs: Add README file`というコミットで先に進んでいました。このままマージするとマージコミットができてしまいます。歴史を一直線に保つため、整理した`feature/divide`ブランチの土台を、最新の`main`に付け替えます。
```bash
# feature/divideブランチにいることを確認
git rebase main
```
このコマンドは、前章で学んだ基本的なリベースです。`feature/divide`ブランチにだけ存在するコミット（先ほどまとめた1つのコミット）が、最新の`main`の先端に移動します。

`git log --oneline --graph --all`で全体の歴史を見てみましょう。
```
* <even_newer_hash> (HEAD -> feature/divide) feat: Add 'divide' function with zero check
* <hash_readme> (main) docs: Add README file
* <hash_subtract> feat: Add 'subtract' function
* <hash_add> feat: Add 'add' function
```
`main`ブランチの歴史の上に、私たちの機能追加が一直線に繋がりました。

---
## 25.4 ステップ3: マージして完了

これで、レビューの準備が整った美しいブランチが完成しました。`main`ブランチにマージしましょう。
```bash
git switch main
git merge feature/divide
```
`feature/divide`は`main`の歴史を完全に取り込んでいるため、このマージは必ず**Fast-forward**になります。マージコミットは作られず、`main`のポインタが前に進むだけです。

最終的な`main`の歴史は、クリーンで一直線、そして各コミットが意味のある単位でまとまっており、非常に追跡しやすくなっています。

---
**まとめ**

お疲れ様でした！この実践演習を通して、あなたは以下のプロフェッショナルなワークフローを完遂しました。

1.  ローカルの作業ブランチで、試行錯誤の結果生まれた乱雑なコミットを作成した。
2.  `git rebase -i`を駆使して、これらのコミットを意味のある単位にまとめ、メッセージを洗練させた。
3.  `git rebase`で、整形したブランチを最新の`main`ブランチの先端に移動させ、コンフリクトの可能性を事前に解消した。
4.  最後にFast-forwardマージでクリーンに本流へ統合した。

この流れは、`rebase`を安全かつ効果的に使うための理想的なモデルです。

最後に演習用ディレクトリを削除しておきましょう。
```bash
cd ..
rm -rf calculator-app
```
