# 第6部: トラブルシューティングと高度なトピック

---

# 第32章: よくあるエラーと解決法

Gitを使っていると、意図しない操作によって予期せぬ状態に陥ることがあります。「ブランチを間違えて消してしまった」「`reset --hard`で作業が消えた」「`detached HEAD`って何？」など、パニックになりがちな状況は数多く存在します。

しかし、Gitには強力な安全装置と回復機能が備わっています。その中でも最強のツールが**`reflog`**です。`reflog`は、あなたのリポジトリで`HEAD`が移動したすべての記録を、一定期間保持しています。これは、あなたのローカルリポジトリ専用の「操作履歴」であり、致命的なミスからあなたを救う最後の命綱となります。

この章では、`reflog`を中心に、よくあるトラブルとその回復手順を学びます。

---
## 32.1 「しまった！ブランチを強制削除してしまった！」 -> `reflog`で復活

`git branch -d`はマージ済みのブランチしか削除できませんが、`-D`オプションを使えば、マージされていないブランチも強制的に削除できます。もし間違えて、まだ必要な作業が残っているブランチを`-D`で消してしまったらどうなるでしょうか？

**シナリオ: 重要なブランチを誤って削除**
```bash
# 演習用リポジトリ
git init && cd .git && touch an-empty-file-to-make-main-branch-work && cd .. && git add . && git commit -m "Initial"

# 重要な作業ブランチを作成
git switch -c very-important-feature
echo "Secret formula" > secret.txt && git add . && git commit -m "feat: Add secret formula"

# mainに戻る
git switch main

# 誤ってブランチを強制削除！
git branch -D very-important-feature
```
`git branch`コマンドで確認しても、`very-important-feature`はどこにも見当たりません。コミットは永遠に失われてしまったのでしょうか？

**解決策: `git reflog`でコミットを探し、ブランチを再作成**

ここで`git reflog`の出番です。このコマンドは、あなたの`HEAD`の移動履歴を表示します。
```bash
git reflog
```
出力結果 (一部抜粋):
```
<hash_main> HEAD@{0}: checkout: moving from very-important-feature to main
<hash_secret> HEAD@{1}: commit: feat: Add secret formula
<hash_initial> HEAD@{2}: checkout: moving from main to very-important-feature
...
```
`reflog`は、「いつ、どこからどこへ移動したか」を詳細に記録しています。`HEAD@{1}`の行に、失われたはずの「feat: Add secret formula」コミットのハッシュ値(`<hash_secret>`)が残っているのがわかります。

コミットさえ見つかれば、そこからブランチを復活させるのは簡単です。
```bash
git branch very-important-feature <hash_secret>
```
これで、`very-important-feature`ブランチが、失われる直前の状態で完全に復活しました。`reflog`のおかげで、致命的なミスは完全に回避されました。

---
## 32.2 「しまった！`reset --hard`でコミットが消えた！」 -> `reflog`で時間を巻き戻す

`git reset --hard`は、指定したコミットの状態にインデックスと作業ディレクトリを完全に一致させる、強力かつ危険なコマンドです。もし間違ったコミットに戻してしまったら、それ以降のコミットは歴史から消え去ったように見えます。

**シナリオ: `reset --hard`で未来のコミットを消してしまう**
```bash
git init
echo "C1" > file.txt && git add . && git commit -m "C1"
echo "C2" >> file.txt && git add . && git commit -m "C2"

# この時点で C1 -> C2 という歴史

# 誤って最初のコミットにハードリセットしてしまう
git reset --hard HEAD~1
```
`git log`を見ると`C1`のコミットしか見えません。`C2`はどこに行ったのでしょうか？

**解決策: `reflog`でリセット前の状態に戻る**
```bash
git reflog
```
出力結果:
```
<hash_c1> HEAD@{0}: reset: moving to HEAD~1
<hash_c2> HEAD@{1}: commit: C2
<hash_c1> HEAD@{2}: commit (initial): C1
```
`HEAD@{1}`に、`reset`する直前の`C2`の状態が記録されています。このハッシュ値を使えば、`reset`する前の状態に`reset`し直すことができます。
```bash
git reset --hard <hash_c2>
```
これで、`C2`のコミットが再び歴史に現れ、作業ディレクトリの状態も元通りになりました。

---
## 32.3 「`detached HEAD`状態になった！」 -> ブランチを作って解決

第12章でも触れましたが、`detached HEAD`（分離HEAD）は、ブランチではなく特定のコミットを直接チェックアウトしたときに発生する状態です。`git checkout <commit-hash>`などを実行するとこの状態になります。

この状態で新しいコミットを作成すること自体は可能ですが、その後で別のブランチに切り替えてしまうと、新しいコミットを指すポインタがなくなり、いずれGitのガベージコレクションによって削除されてしまう可能性があります。

**解決策: 現在の場所に新しいブランチを作成する**

`detached HEAD`はエラーではありません。Gitが「あなたは今、ブランチという名前札のない、ただのコミットの上に直接いますよ」と教えてくれているだけです。

もしその場所で作業を続けたいのであれば、その場所に新しい名前札、つまり**ブランチ**を作ってあげれば解決します。
```bash
# 現在地から新しいブランチを作成して、そちらに切り替える
git switch -c new-feature-branch
```
たったこれだけで、`detached HEAD`状態は解消され、あなたの作業は`new-feature-branch`という名前で安全に保護されます。

---
**まとめ**

この章では、開発現場で頻繁に遭遇するトラブルとその解決法を学びました。
-   **`git reflog`は最強の安全網**: ブランチの削除や`reset --hard`といった破壊的な操作で失われたと思われたコミットも、`reflog`を辿ればほとんどの場合救出できる。
-   `reflog`はあくまで**ローカルの記録**: `push`していないコミットの回復には絶大な力を発揮しますが、リモートリポジトリには`reflog`は存在しません。
-   **`detached HEAD`は怖くない**: 単にブランチを作って名前をつけてあげるだけで、安全な状態に戻ることができる。

これらの知識があれば、Gitの操作で少しミスをしても、冷静に、そして確実に対処することができます。
