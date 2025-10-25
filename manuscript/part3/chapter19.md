# 第19章: マージツールの使い方

手動でのコンフリクト解決は、Gitの基本を理解する上で非常に重要です。しかし、実務ではより効率と正確性が求められます。特に、複数のファイルにまたがる大規模なコンフリクトや、一つのファイル内でも変更箇所が複雑に絡み合っている場合、テキストエディタだけで対応するのは困難です。

そこで登場するのが**マージツール**です。マージツールは、コンフリクトを解決するために特化したGUIツールで、多くの場合、以下の3つのファイルを並べて表示してくれます。

-   **BASE**: 共通の祖先のバージョン。
-   **LOCAL (Ours)**: `HEAD`、つまり自分がいるブランチのバージョン。
-   **REMOTE (Theirs)**: マージしようとしている相手ブランチのバージョン。

そして、これらの差分を見ながら、最終的にどのような結果（**RESULT**）にしたいかをボタン一つで選択・編集できます。これにより、コンフリクトマーカーを手で削除する作業から解放され、より迅速かつ正確にコンフリクトを解決できます。

---
## 19.1 マージツールの設定

Gitは多くのマージツールに対応しています。`git mergetool --tool-help` で利用可能なツールの一覧を確認できます。ここでは、多くの開発者が利用しているVisual Studio Code (VS Code) をマージツールとして設定する方法を解説します。

まず、Gitに対して、マージツールとしてVS Codeを使うことを教える必要があります。以下のコマンドをターミナルで実行してください。

```bash
# マージツールとして "vscode" を設定
git config --global merge.tool vscode

# VS Codeをマージツールとして使うための設定
git config --global mergetool.vscode.cmd "code --wait $MERGED"

# マージツール起動時に確認のプロンプトを表示しないようにする（お好みで）
git config --global mergetool.prompt false
```

-   `merge.tool vscode`: マージに使うツールの名前を`vscode`と定義します。
-   `mergetool.vscode.cmd`: `git mergetool`を実行した際に、実際に呼び出されるコマンドを設定します。`code --wait`は、VS Codeのウィンドウが閉じられるまでGitのプロセスを待機させるための重要なオプションです。`$MERGED`はGitが自動的にコンフリクト中のファイルパスに置き換えてくれます。
-   `mergetool.prompt false`: `git mergetool`を実行するたびに「ツールを起動しますか？」と聞かれるのを省略します。

これで設定は完了です。

---
## 19.2 `git mergetool` の実行

それでは、実際にマージツールを使ってみましょう。第17章と同様のコンフリクトを再度発生させます。

```bash
# 実験用ディレクトリを作成
mkdir git-mergetool-practice && cd git-mergetool-practice
git init

# 共通の祖先
echo "Hello, World!" > greeting.txt && git add . && git commit -m "Initial"

# featureブランチでの変更
git switch -c feature
echo "こんにちは、世界！" > greeting.txt && git add . && git commit -m "Japanese"

# masterブランチでの変更
git switch master
echo "Hola, Mundo!" > greeting.txt && git add . && git commit -m "Spanish"

# コンフリクトを発生させる
git merge feature
```
コンフリクトが発生しました。ここで、前回のようにファイルを直接編集する代わりに、`git mergetool`コマンドを実行します。

```bash
git mergetool
```
このコマンドを実行すると、Gitは設定に従ってVS Codeを起動し、マージ専用のUIで`greeting.txt`を開きます。

画面には、多くの場合、以下のようなビューが表示されます。
-   左側: **Incoming (Theirs)** - `feature`ブランチの変更内容（こんにちは、世界！）
-   右側: **Current (Ours)** - `master`ブランチの変更内容（Hola, Mundo!）
-   下側: **Result** - 最終的なマージ結果を編集するビュー

そして、それぞれの変更箇所の上には「Accept Current Change」「Accept Incoming Change」「Accept Both Changes」といったアクションボタンが表示されます。これらのボタンをクリックすることで、コンフリクトマーカーを手動で編集することなく、直感的にマージ結果を作成できます。

今回は「Accept Both Changes」に相当する操作を行い、Resultビューが `Hola, Mundo! / こんにちは、世界！` のようになるように編集し、ファイルを保存してVS Codeのウィンドウを閉じます。

---
## 19.3 マージの完了

マージツールを終了すると、Gitは作業ディレクトリのファイルがツールによって更新されたことを検知します。また、マージに成功した元のコンフリクトファイルは、`.orig`という拡張子でバックアップとして残されることがあります。（これは設定によります）

マージツールでの作業は、あくまで「ステップ1: ファイルを手動で編集する」を効率化したに過ぎません。したがって、その後の手順は通常の手動解決と全く同じです。

1.  **`git add` で解決を伝える**
2.  **`git commit` でマージを完了する**

```bash
# ステージングして解決を伝える
git add greeting.txt

# マージを完了する
git commit
```

これで、マージツールを使ったコンフリクトの解決が完了しました。

---
**まとめ**

この章では、コンフリクト解決の強力な助っ人であるマージツールの使い方を学びました。

-   マージツールは、コンフリクトした3つのバージョン（BASE, LOCAL, REMOTE）を視覚的に比較し、効率的な解決を支援するGUIツールである。
-   `git config`コマンドで、VS Codeなどの好みのツールをGitに登録できる。
-   コンフリクトが発生したら、`git mergetool`コマンドでツールを起動する。
-   ツールでの編集・保存が完了したら、通常通り `git add` と `git commit` を実行してマージを完了させる必要がある。

複雑なコンフリクトに直面した際は、マージツールを積極的に活用することで、解決にかかる時間と精神的な負担を大幅に削減できます。

最後に演習用ディレクトリを削除しておきましょう。
```bash
cd ..
rm -rf git-mergetool-practice
```
