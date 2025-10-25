# 第14章: 実践演習（第1部 + 第2部）

お疲れ様でした。第1部と第2部を通して、私たちはGitの心臓部である`.git`ディレクトリの探検を終え、日常的に使う基本コマンドが内部で何をしているのかを解き明かしました。

この章では、それらの知識を総動員し、 একটি ছোট (ちいさな) 機能開発のシナリオを通して、学んだことを実践に繋げます。ここでの目的は、コマンドを打つ指を動かすだけでなく、そのコマンドが`.git`内部のオブジェクトや参照、インデックスにどのような影響を与えるかを常に意識しながら進めることです。

---
## 14.1 シナリオ: シンプルな電卓プログラムの開発

今回のシナリオは、「基本的な四則演算ができる電卓プログラムを開発する」です。
まずはプロジェクトのセットアップから始めましょう。

```bash
# 演習用ディレクトリを作成して移動
mkdir calculator-project && cd calculator-project
git init

# プロジェクトの顔となるREADMEを作成
echo "# Simple Calculator" > README.md
git add README.md
git commit -m "docs: Add README.md"
```
**内部の動きの確認**:
- `git init`: `.git`ディレクトリが作成されました。
- `git add`: `README.md`の`blob`オブジェクトが作成され、その情報が`index`に記録されました。
- `git commit`: `index`から`tree`オブジェクトが、そしてメタデータと共に`commit`オブジェクトが作成され、`master`ブランチがその新しいコミットを指しました。

---
## 14.2 最初の機能: 足し算機能の実装

最初の機能として、足し算機能を実装します。

```bash
# main.pyを作成し、足し算関数を記述
echo "def add(a, b):" > main.py
echo "    return a + b" >> main.py
```

`git status`を叩いてみましょう。"Untracked files" に`main.py`が表示されるはずです。これは、`main.py`がワーキングディレクトリに存在するものの、インデックスにはまだ記録されていないためです。

```bash
# 変更をステージング
git add main.py
```
**内部の動きの確認**:
- `git add`が`main.py`の内容から`blob`オブジェクトを作成しました。
- `index`に`main.py`のエントリが追加され、新しい`blob`のハッシュが記録されました。
- `git ls-files --stage`でインデックスの内容を確認してみましょう。

では、最初の機能をコミットします。
```bash
git commit -m "feat: Add 'add' function"
```
これで、足し算機能を含む新しいスナップショットが歴史に刻まれました。`git log --oneline`で確認してみてください。

---
## 14.3 機能追加とリファクタリングを分割する

次に、引き算機能を追加します。しかし、作業の途中で、既存の`add`関数にドキュメントコメント（docstring）を追加したくなったとします。

```bash
# main.pyを編集
# 1. 引き算関数を追加
echo "" >> main.py
echo "def subtract(a, b):" >> main.py
echo "    return a - b" >> main.py
# 2. 既存のadd関数にdocstringを追加
# (ここではsedコマンドで擬似的に行います)
sed -i '2i\    """This function adds two numbers"""' main.py

# ファイル全体の内容を確認
cat main.py
```
`git diff`を実行すると、2種類の変更（引き算関数の追加と、足し算関数のdocstring追加）が混ざっているのが分かります。これらを一つのコミットに入れるのは、歴史を分かりにくくします。

ここで`git add -p`の出番です。
```bash
git add -p main.py
```
対話モードが始まったら、Gitが提示する変更の塊（hunk）をよく見てください。
1.  まず、docstringの追加が表示されるはずです。これはリファクタリングなので、まだステージングしません。`n` (no) を入力します。
2.  次に、引き算関数の追加が表示されます。これは新しい機能なのでステージングします。`y` (yes) を入力します。

`git status`を見ると、`main.py`が"Changes to be committed"と"Changes not staged for commit"の両方に表示されているはずです。これは、`index`とワーキングディレクトリで`main.py`の状態が異なるためです。

**内部の動きの確認**:
- `index`に記録されている`main.py`は、「引き算関数だけが追加された」状態のスナップショット（`blob`）を指しています。
- ワーキングディレクトリの`main.py`は、さらに「docstringが追加された」状態です。

この状態で、まずは機能追加のコミットを行います。
```bash
git commit -m "feat: Add 'subtract' function"
```

コミットが完了したら、残っているリファクタリングの変更をステージングしてコミットします。
```bash
git add main.py
git commit -m "refactor: Add docstring to 'add' function"
```

`git log --oneline`を見ると、2つの関心事が、2つの綺麗なコミットに分割されているのが分かります。ステージングエリアを意図的に使うことで、これが可能になるのです。

---
## 14.4 ブランチを切って新機能開発

最後に、掛け算機能を新しいブランチで開発します。
```bash
git switch -c multiply-feature
```
**内部の動きの確認**:
- `.git/refs/heads/multiply-feature`というファイルが作成されました。
- `.git/HEAD`の中身が`ref: refs/heads/multiply-feature`に書き換わりました。

この新しいブランチで、掛け算機能を追加してコミットしましょう。
```bash
echo "" >> main.py
echo "def multiply(a, b):" >> main.py
echo "    return a * b" >> main.py

git add .
git commit -m "feat: Add 'multiply' function"
```

`git log --oneline --graph --all`を実行して、歴史が分岐した様子をその目で確かめてみてください。`HEAD`は`multiply-feature`を指し、`master`は元の場所に取り残されているのが見えるはずです。

---
**第2部のまとめ**

この演習を通して、私たちは第1部と第2部で学んだ知識を実践的なワークフローに落とし込みました。
- `add`がインデックスを操作するコマンドであることを利用して、`git add -p`でコミットを綺麗に分割しました。
- `switch -c`が`HEAD`参照と`refs`ファイルを操作するコマンドであることを理解しながら、安全に新しい作業空間（ブランチ）を作成しました。

`.git`内部の動きというメンタルモデルを持つことで、全ての基本操作が「なぜそうなるのか」というレベルで理解できるようになりました。この土台があれば、第3部以降で学ぶマージやリベースといった、より複雑な操作にも自信を持って挑むことができます。

最後に演習用ディレクトリを削除しておきましょう。
```bash
cd ..
rm -rf calculator-project
```
