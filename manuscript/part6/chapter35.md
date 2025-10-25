# 第35章: サブモジュールとサブツリー

プロジェクトが大規模になると、一つのリポジトリが他のリポジトリに依存する、という状況が頻繁に発生します。例えば、複数のプロジェクトで共有されるUIコンポーネントライブラリや、特定の機能を提供するマイクロサービスなどを、メインのリポジトリから利用したい場合です。

このような「リポジトリ内リポジトリ」の依存関係を管理するために、Gitは主に2つの仕組みを提供しています。「サブモジュール」と「サブツリー」です。これらは似た問題を解決しますが、そのアプローチとトレードオフは大きく異なります。

---
## 35.1 Git Submodule: 参照としての依存関係

サブモジュールは、親リポジトリ内に、**別の独立したGitリポジトリへのリンク（参照）** を埋め込む機能です。親リポジトリは、サブモジュールの中身のファイルを直接管理しません。代わりに、「このパスには、あのリポジトリの、このコミットを配置せよ」という情報だけを記録します。

**サブモジュールの追加**

`my-project`というプロジェクトが、外部の`shared-library`に依存しているとします。
```bash
# まずは親プロジェクトと、依存されるライブラリを準備
git init shared-library && cd shared-library
echo "Utility function" > lib.js && git add . && git commit -m "Initial library"
cd ..

git init my-project && cd my-project
echo "# My Project" > README.md && git add . && git commit -m "Initial project"

# サブモジュールとして shared-library を追加
git submodule add ../shared-library libs
```
このコマンドは、以下のことを行います。
1.  `libs`ディレクトリに`shared-library`リポジトリをクローンする。
2.  `.gitmodules`という設定ファイルを作成（または更新）し、サブモジュールの情報を記録する。
3.  親リポジトリに、`libs`ディレクトリへの参照をステージングする。

`git status`を見ると、`.gitmodules`と`libs`がステージングされていることがわかります。これらをコミットすることで、依存関係が記録されます。
```bash
git commit -m "feat: Add shared-library as a submodule"
```

**サブモジュールを含むリポジトリのクローン**

サブモジュールを含むリポジトリをクローンする場合、通常の方法では親リポジトリしかクローンされず、サブモジュールのディレクトリは空になります。
```bash
# 通常のクローンではサブモジュールの中身は空
git clone ../my-project project-clone-normal
ls project-clone-normal/libs # -> 空っぽ

# --recurse-submodules を使うと、サブモジュールも自動的に初期化・取得される
git clone --recurse-submodules ../my-project project-clone-recursive
ls project-clone-recursive/libs # -> lib.js が存在する
```
もし`--recurse-submodules`を忘れた場合は、クローン後に`git submodule update --init`を実行することで手動で取得できます。

-   **メリット**:
    -   親リポジトリと子リポジトリの歴史が完全に分離される。
    -   親リポジトリは子リポジトリの特定のコミットを指すだけなので、依存バージョンを厳密に管理できる。
-   **デメリット**:
    -   クローン時や更新時に、サブモジュールを意識した特別なコマンド (`--recurse-submodules`, `submodule update`) が必要。
    -   ワークフローが複雑になりがちで、初心者が混乱しやすい。

---
## 35.2 Git Subtree: コピーとしての依存関係

サブツリーは、サブモジュールよりもずっとシンプルなアプローチを取ります。**別のリポジトリのファイルと歴史を、自分のリポジトリのサブディレクトリに完全にコピーしてしまう**のです。一度取り込んでしまえば、それらは親リポジトリ内のごく普通のファイルとディレクトリとして扱われます。

**サブツリーの追加**
```bash
# my-project2 を準備
git init my-project2 && cd my-project2
git commit --allow-empty -m "Initial"

# git subtree add コマンドで shared-library を追加
# --prefixでディレクトリを指定, --squashで履歴を1コミットにまとめる
git subtree add --prefix=libs ../shared-library main --squash
```
このコマンドは、`shared-library`の`main`ブランチの全ファイルを`libs`ディレクトリにコピーし、その履歴を（`--squash`によって）1つのマージコミットとして親リポジトリに追加します。

`.gitmodules`のような特別なファイルは作られません。`libs`ディレクトリは、Gitから見れば他のディレクトリと何ら変わりありません。

**サブツリーを含むリポジトリのクローン**

サブツリーは単なるコピーなので、クローンは通常通りでOKです。
```bash
git clone ../my-project2 project2-clone
ls project2-clone/libs # -> lib.js が存在する
```
特別なフラグは不要で、クローンした人は依存関係を意識する必要すらありません。

**サブツリーの更新**

`shared-library`が更新された場合、`subtree pull`で変更を取り込めます。
```bash
git subtree pull --prefix=libs ../shared-library main --squash
```

-   **メリット**:
    -   利用者が依存関係を意識する必要がなく、`git clone`するだけで全てが揃う。
    -   特別なコマンドが不要で、ワークフローが非常にシンプル。
-   **デメリット**:
    -   子リポジトリの歴史を親リポジトリに取り込むため、親リポジトリの歴史が複雑になる可能性がある。
    -   子リポジトリへの変更の貢献（コントリビュート）がサブモジュールに比べて少し煩雑。

---
**まとめ**

| 特徴 | Git Submodule | Git Subtree |
| :--- | :--- | :--- |
| **依存関係の管理** | リンク (参照) | コピー (マージ) |
| **クローン方法** | `clone --recurse-submodules` | `clone` (通常通り) |
| **利用者の手間** | 追加のコマンドが必要 | 不要 |
| **歴史の分離** | 完全に分離 | 親リポジトリに統合 |
| **推奨ケース** | 依存バージョンを厳密に管理したい場合。頻繁に更新・貢献する場合。 | 依存関係をシンプルに保ちたい場合。利用者に手間をかけさせたくない場合。 |

どちらの技術も一長一短です。プロジェクトの性質、チームのスキルレベル、依存関係の更新頻度などを考慮して、最適な方法を選択することが重要です。一般的には、迷ったらよりシンプルな**サブツリー**から試してみるのが良いでしょう。
