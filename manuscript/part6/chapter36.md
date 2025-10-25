# 第 36 章: Git Hooks で品質を自動化する

ソフトウェア開発において、コードの品質を一定に保つことは非常に重要です。コーディング規約の遵守、静的解析ツールの実行、単体テストの実施など、コミットや push の前に行うべきチェックは数多くあります。しかし、これらのチェックを毎回手動で行うのは手間がかかり、忘れてしまうこともあります。

Git Hooks は、このような定型的なチェックを**特定の Git イベント (コミットや push など) の発生時に自動的に実行する**仕組みです。これにより、チーム全体の品質基準を効率的に、そして確実に維持することができます。

---
## 36.1 Git Hooks の仕組み

Git Hooks は、`.git/hooks` ディレクトリに置かれた実行可能なスクリプトです。Git は特定のイベントが発生すると、このディレクトリに対応する名前のスクリプトがないか探し、もし存在すればそれを実行します。

フックスクリプトは、サンプルとして `.sample` という拡張子付きで `.git/hooks` ディレクトリに最初から用意されています。

```bash
ls -F .git/hooks/
# applypatch-msg.sample*  pre-rebase.sample*
# commit-msg.sample*      pre-receive.sample*
# post-update.sample*     prepare-commit-msg.sample*
# pre-applypatch.sample*  push-to-checkout.sample*
# pre-commit.sample*      update.sample*
# pre-push.sample*
```

これらの `.sample` ファイルをリネームして拡張子を取り除き、実行権限を与えるだけで、そのフックが有効になります。スクリプトはシェルスクリプト、Python, Ruby, Perl など、実行可能であれば何でもかまいません。

---
## 36.2 クライアントサイドフックとサーバーサイドフック

Git Hooks は、実行される場所によって大きく 2 種類に分けられます。

-   **クライアントサイドフック**: 開発者個人のローカルリポジトリで実行されます。
    -   例: `pre-commit`, `prepare-commit-msg`, `commit-msg`, `post-commit`, `pre-push`
    -   用途: コミットメッセージのフォーマットチェック、コードの静的解析 (リンティング)、単体テストの実行など。
    -   **注意**: これらのフックは `.git` ディレクトリ以下で管理されるため、**リポジトリと一緒にクローンされません**。チームで共有するには、別の仕組みが必要です (後述)。

-   **サーバーサイドフック**: Git サーバー (GitHub, GitLab など) 上で実行されます。
    -   例: `pre-receive`, `update`, `post-receive`
    -   用途: push されたコードが特定の基準を満たしているかの強制、CI/CD パイプラインのトリガー、関係者への通知など。
    -   サーバーの管理者が設定するため、チーム全員に強制力を持ちます。

この章では、個人開発やチーム開発で特によく使われるクライアントサイドフックに焦点を当てます。

---
## 36.3 実践: `pre-commit` フックでコミット品質を保証する

`pre-commit` フックは、`git commit` を実行した際、コミットメッセージを入力する**前**に実行されます。このスクリプトが 0 以外のステータスコードで終了すると、Git はコミットプロセスを中断します。これにより、品質基準を満たさないコードがコミットされるのを防ぐことができます。

**シナリオ**: コミットにデバッグ用の `console.log` が含まれていないか自動でチェックする。

1.  **フックスクリプトの作成**:

    `.git/hooks/pre-commit` という名前で新しいファイルを作成します。(もし `pre-commit.sample` があれば、それをコピーまたはリネームします)

    ```bash
    #!/bin/sh

    echo "Running pre-commit hook..."

    # JavaScript ファイル (*.js) を対象に "console.log" が含まれていないかチェック
    # --quiet オプションで、マッチしなかった場合は何も出力しない
    # --invert-match (--invert-match) で、マッチしなかった場合に 0 を返す (成功)
    # 1行でもマッチしたら grep は 0 を返し、`!` で反転して 1 (失敗) になる
    if ! git diff --cached --name-only --diff-filter=ACM | grep '\.js$' | xargs grep -n 'console\.log' --quiet; then
        echo "✅ No console.log found."
        exit 0
    else
        echo "🚨 Error: Found 'console.log' in staged files."
        echo "Please remove them before committing."
        exit 1 # コミットを中断
    fi
    ```

2.  **実行権限の付与**:

    スクリプトが実行できるように、実行権限を与えます。

    ```bash
    chmod +x .git/hooks/pre-commit
    ```

3.  **動作確認**:

    `console.log` を含む JavaScript ファイルを作成してコミットしてみます。

    ```bash
    echo "console.log('debug info');" > main.js
    git add main.js
    git commit -m "feat: Add main script"

    # --- フックの出力 ---
    # Running pre-commit hook...
    # main.js:1:console.log('debug info');
    # 🚨 Error: Found 'console.log' in staged files.
    # Please remove them before committing.
    ```

    エラーメッセージが表示され、コミットが中断されました。次に、`console.log` を削除して再度コミットします。

    ```bash
    # console.log を削除
    sed -i "/console.log/d" main.js
    git add main.js
    git commit -m "feat: Add main script"

    # --- フックの出力 ---
    # Running pre-commit hook...
    # ✅ No console.log found.
    # [main <hash>] feat: Add main script
    # ...
    ```

    今度はチェックを通過し、正常にコミットが完了しました。

---
## 36.4 チームでのフック共有: Husky

前述の通り、`.git/hooks` はリポジトリで共有されません。チームで同じフックを使いたい場合、`pre-commit` や `husky` のようなツールを導入するのが一般的です。

**Husky** は、npm パッケージとしてインストールでき、`package.json` に設定を記述するだけで、チームメンバーが `npm install` を実行した際に自動的に Git Hooks をセットアップしてくれます。これにより、フックの導入と管理が非常に簡単になります。

---
**まとめ**

- Git Hooks は、特定の Git イベントをトリガーにスクリプトを自動実行する仕組みである。
- `.git/hooks` ディレクトリに実行可能なスクリプトを置くことで有効になる。
- `pre-commit` フックは、コミット前にコードの品質チェックを行うのに非常に便利である。
- スクリプトが 0 以外のステータスで終了すると、Git のプロセス (コミットや push) は中断される。
- チームでフックを共有するには、Husky などの専用ツールを利用するのが一般的である。
