# 第 19 章: マージツールによるコンフリクト解決

前章で学んだテキストエディタで手動でコンフリクトを解決する方法は、Git の基本であり、どんな環境でも使える普遍的なスキルです。しかし、コンフリクトが複数ファイルにまたがっていたり、変更箇所が複雑だったりすると、コンフリクトマーカーを一つ一つ編集していくのは非常に手間がかかり、ミスも起こりやすくなります。

幸いなことに、Git は外部のグラフィカルなツールと連携して、より視覚的にコンフリクトを解決する仕組みを提供しています。それが `git mergetool` コマンドです。

---
## 19.1 マージツールの設定

`git mergetool` を使うには、まずどのツールを使用するかを Git に設定する必要があります。多くの開発用エディタや専用の差分比較ツールが利用可能です。ここでは、多くの開発者に利用されている **Visual Studio Code (VS Code)** をマージツールとして設定する方法を例に挙げます。

VS Code をマージツールとして設定するには、以下のコマンドを実行します。
```bash
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd "code --wait $MERGED"
git config --global mergetool.keepBackup false
```
- `merge.tool vscode`: デフォルトのマージツールとして `vscode` という名前のカスタム設定を使うことを宣言します。
- `mergetool.vscode.cmd "code --wait $MERGED"`: `git mergetool` が実行されたときに、VS Code の CLI (`code`) を特定のオプション付きで起動するように設定します。`--wait` は VS Code での作業が終わるまでターミナルの処理を待たせるために重要で、`$MERGED` はコンフリクトしているファイルパスに置き換えられます。
- `mergetool.keepBackup false`: マージツールが生成するバックアップファイル (`.orig`) を作成しないように設定します。これはリポジトリを綺麗に保つのに役立ちます。

---
## 19.2 `git mergetool` の使い方

設定が完了したら、実際にマージツールを使ってみましょう。
前章と同様のコンフリクト状態を再度作り出します。

```bash
# 実験用ディレクトリを作成して移動
mkdir mergetool-practice && cd mergetool-practice
git init

# 共通の祖先
echo "Hello, world!" > greeting.txt && git add . && git commit -m "Initial"

# featureブランチでの変更
git switch -c feature
echo "Hello, feature!" > greeting.txt && git add . && git commit -m "Feat"

# mainブランチでの変更
git switch main
echo "Hello, main!" > greeting.txt && git add . && git commit -m "Main"

# コンフリクトを発生させる
git merge feature
```

このコンフリクト状態で、`git mergetool` コマンドを実行します。
```bash
git mergetool
```
すると、Git は設定された VS Code を起動し、コンフリクトを解決するための特別な UI を表示します。

VS Code のマージエディタ画面は、典型的には以下のような構成になっています。
- **左側 (Incoming)**: マージしようとしているブランチ (`feature`) での変更内容。`--theirs` に相当。
- **右側 (Current)**: 現在のブランチ (`main`) での変更内容。`--ours` に相当。
- **下側 (Result)**: 解決後の最終的なファイル内容。

エディタ上には、それぞれの変更を採用するためのボタン（"Accept Incoming Change", "Accept Current Change", "Accept Both Changes" など）が表示されます。開発者は、これらのボタンをクリックしたり、下側の Result ペインを直接編集したりすることで、直感的にコンフリクトを解決できます。

作業が完了したら、ファイルを保存して VS Code のタブを閉じます。`--wait` オプションのおかげで、タブを閉じるとターミナルに制御が戻ります。

### マージツールの内部動作

`git mergetool` が行っていることは、実は前章で学んだ手動での解決プロセスを、ツールを使って効率化しているだけです。
1.  **ファイルを編集する**: このステップを GUI ツールが支援してくれます。ユーザーが GUI 上で変更を確定すると、ツールがコンフリクトマーカーのない最終的なファイルをワーキングディレクトリに保存します。
2.  **`git add` する**: 多くのマージツールは、解決が完了すると自動的に `git add` を実行してくれるか、それに相当する処理を行います。これにより、インデックスはステージ 0 の解決済み状態に更新されます。
3.  **`git commit` する**: この最後のステップは、通常どおり開発者が手動で実行する必要があります。マージツールがマージコミットまで自動で行うことは稀です。

ツールが終了した後、`git status` を確認すると、ファイルがすでにステージングされている（`Changes to be committed` になっている）ことが分かります。あとは `git commit` を実行すればマージは完了です。

---
**まとめ**

- `git mergetool` は、外部の GUI ツールと連携してマージコンフリクトを視覚的に解決するためのコマンドである。
- 使用するには、`git config` で使用するツールとその起動コマンドを事前に設定する必要がある。
- マージツールは、「ファイルを編集し、`git add` する」というコンフリクト解決の最初の 2 ステップを効率化してくれる。
- 最終的なマージの完了には、開発者自身が `git commit` を実行する必要がある。

複雑なコンフリクトに直面したとき、マージツールは強力な味方になります。しかし、その裏で Git がどのように動いているか（インデックスの状態変化など）を理解していることで、ツールが予期せぬ挙動をした場合でも冷静に対処できるようになります。

最後に実験用ディレクトリを削除しておきましょう。
```bash
cd ..
rm -rf mergetool-practice
```
