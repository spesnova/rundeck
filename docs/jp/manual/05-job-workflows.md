% ジョブのワークフロー
% Alex Honor; Greg Schueler
% November 20, 2010

ジョブの最も基本的な特徴は、1 つまたは複数のコマンドをあるノードセットに対してまとめて実行できることにあります。コマンドシーケンス（連続したコマンド）は_ワークフロー_と呼ばれ、ワークフローの各ステップには、コマンド呼び出しが定義されています。

ジョブワークフローの各ステップは、ジョブの一覧画面またはジョブの編集フォームにてジョブの詳細として表示されます。

## ワークフローの定義

ワークフローはグラフィカルコンソールから、または XML or YAML ドキュメントをサーバーに読み込ませることで定義できます。

グラフィカルコンソールではステップの追加・編集・削除・順序の入れ替えが出来るようになっています。

テキスト形式でジョブを定義することを好むユーザは、2 つの定義方法を参照するといいでしょう：

*   XML:  [job-v20(5)](../manpages/man5/job-v20.html)
*   YAML: [job-yaml-v12(5)](../manpages/man5/job-yaml-v12.html)

グラフィカルコンソールの中でジョブを定義しておいて、そこから [rd-jobs(1)](../manpages/man1/rd-jobs.html) Shell ツールが利用する定義ファイルをエクスポートすることも可能です。

[ジョブ定義ファイルのエクスポート](#ジョブ定義ファイルのエクスポート)および、[ジョブ定義ファイルのインポート](#ジョブ定義ファイルのインポート)を参照してください。

## ワークフローコントロールの設定

ワークフローの実行は、「Keepgoing」（続行）と「Strategy」（計画）という 2 種類の重要な設定によってコントロールされています。

![Workflow controls](../figures/fig0401.png)

「Keepgoing」：これはあるステップにてエラーが起きた際どうするかをコントロールします：

*   「No」： 直ちに失敗とします。（デフォルト）
*   「Yes」：次のステップに進みます。

デフォルトでは直ちに失敗となりますが、場合によっては実行を続けるよう設定することもできます。

「Strategy」： 各ステップの実行と各ノードへのコマンドディスパッチの順序をコントロールします。

*   「Node-oriented」： 次のノードに進む前にすべてのワークフローを実行します（デフォルト）
*   「Step-oriented」： 次のノードの前にすべてのノード上の各ステップを実行します。

以下に示すでは、2 つのノードに対して 3 つのステップがどのように進められるかを対比させています。

「Node-oriented」フローはこうなります：

~~~~~~~~~~~~~~~~~~~~~
1.   NodeA    step#1
2.     "      step#2
3.     "      step#3
4.   NodeB    step#1
5.     "      step#2
6.     "      step#3
~~~~~~~~~~~~~~~~~~~~~

「Step-oriented」フローはこうなります：

~~~~~~~~~~~~~~~~~~~~~
1.   NodeA    step#1
2.   NodeB      "
3.   NodeA    step#2
4.   NodeB      "
5.   NodeA    step#2
6.   NodeB      "
~~~~~~~~~~~~~~~~~~~~~

「Node-oriented」フローが一般的であることも踏まえて、あなたが自動化しているプロセスでどちらのフローを使うのが適切か決めることになります。

## ワークフローの各ステップ

以下の章では、様々な種類のコマンドを呼び出す各ステップを 1 つのセットとして、どのようにワークフローを作るか説明します。

新しいジョブ定義を作る際、ワークフローのフォームはデフォルトの状態であり、ワークフローステップの定義はされていません。ワークフローのエディターで最初のステップの定義としてフォームにシェルコマンドを入力するようになっています。

![Add a step](../figures/fig0402.png)

新しいステップを追加するには、ワークフローのエディターフォームの中の「Add a step」リンクを押すだけです。これで、どのような種類のワークフローステップを追加したいか選択するようダイアログが出ます。フォームの中を埋めたら、「Save」を押してシーケンスの中に保存します。「Cancel」を押すとフォームが閉じられ、シーケンスの変更は保存されません。

![Workflow step types](../figures/fig0403.png)

新しいステップは、常にシーケンスの最後に加えられます。ステップの順番を管理するには[ステップの順序を変更する](job-workflows.html#ステップの順序を変更する)を見てください。

この先のいくつかの章ではコマンドステップの各種類の詳細について説明します。

### コマンドステップ

システムコマンドを呼び出すには、コマンドステップを使ってください。これはジョブを作る際のデフォルトのワークフロータイプです。リモートホストで実行したいコマンドを入力してください。

![Command step type](../figures/fig0404.png)

これは `dispatch` を使ってコマンドを呼びだすことに似ています：

    dispatch [filter-options] -- command

### スクリプトステップ

入力されたシェルスクリプトを実行します。テキストフィールドからスクリプトに引数を渡すこともできます。

![Script step type](../figures/fig0405.png)

これは `dispatch` を用いてコマンドを呼びだすことに似ています：

    dispatch [filter-options] --stdin -- args <<EOF 
    script content here 
    EOF

### スクリプトファイルステップ

フィルタリングされたノードセットに対してのみローカルスクリプトを実行します。テキストフィールドにスクリプトへの引数を指定できます。

![Script file step type](../figures/fig0406.png)

これは `dispatch` を使ってスクリプトファイルを呼びだすことに似ています：

    dispatch [filter-options] -s scriptfile -- args

### スクリプト URL ステップ

URL からスクリプトをダウンロードし、フィルタリングされたノードセットに対してそれを実行します。テキストフィールドにスクリプトへの引数を指定できます。

Downloads a script from a URL, and executes it to the filtered Node
set. Arguments can be passed to the script by specifying them in the
lower text field.

![Script URL step type](../figures/fig0406.png)

これは `dispatch` を使ってスクリプト URL を呼びだすことに似ています：

    dispatch [filter-options] -u URL -- args

URL には実行時に展開される[コンテキスト変数](#コンテキスト変数)を入れることができます。

### ジョブリファレンスステップ

もう一つ保存されたジョブを呼ぶには、「Job reference step」（ジョブリファレンスステップ）を作成してください。ジョブの名前とそのジョブグループを入力します。

![Job step type](../figures/fig0407.png)

ジョブリファレンスのフォームは保存されているジョブの中から選択しやす閲覧画面を提供します。「Choose A Job...」リンクを押し、希望するジョブを選んでください。


最後に、オプションが定義されたジョブならそれらをコマンドライン引数としてテキストフィールドに指定したり、現在のジョブへ入力オプションを渡すための変数として使うことができます。フォーマット：

    -optname <value> -optname <value> ...

オプションを指定するためのフォーマットはコマンドラインツールにてオプションを指定する場合と全く同じです、オプションに対して変数を指定することもできます。例えば：

    -opt1 something -opt2 ${option.opt2}

この例では、「opt1」オプションに対して、"something" をセットし、「opt2」オプションをトップレベルのジョブからジョブリファレンスに直接渡しています。

これは [run](../manpages/man1/run.html) を用いて他のジョブを呼ぶことに似ています：

    run [filter-options] -j group/jobname -- -opt1 something -opt2 somethingelse

ジョブがオプションを必要としていて、それが指定されていない場合「defalutValue」が定義されていればそれを使います。オプションが「defaultValue」を持っていない場合、必要なオプションが指定されていないということでジョブの実行は失敗します。

## ステップの順番を変える

ジョブのアローアイコンをドラッグ & ドロップして好きな場所に移動することで各ステップの順番を編集できます。青い水平線バーは、ドラッグしているジョブがどこに置かれるか分かりやすくしてくれます。

![Job step reorder](../figures/fig0408.png)

選択したジョブをドロップすると希望する位置に着き、ステップの順番が更新されます。

ステップの順番変更を元に戻したい場合は、各ステップの上にある「Undo」リンクを押してください。

「Redo」ボタンは最新の「Undo」によって元に戻したものを再度変更しなおします。

「Revert All Changes」ボタンを押すと、全ての変更を戻してオリジナルの状態に戻します。

## Save the changes

Once the Workflow steps have been defined and order, changes are
permanently saved after pressing the "Create" button if new or the
"Update" button if the Job is being modified.

## Context Variables

When a Job step is executed, it has a set of "context" variables that you can access in the Job step. There are several sets of context variables, including: the Job context `job`, the Node context `node`, and the Option context `option`.

Job context variables:

* `job.name`: Name of the Job
* `job.group`: Group of the Job
* `job.id`: ID of the Job
* `job.execid`: ID of the current Execution
* `job.username`: Username of the user executing the Job
* `job.project`: Project name

Node context variables:

* `node.name`: Name of the Node being executed on
* `node.hostname`: Hostname of the Node
* `node.username`: Usernae of the remote user
* `node.description`: Description of the node
* `node.tags`: Comma-separated list of tags
* `node.os-*`: OS properties of the Node: `name`,`version`,`arch`,`family`
* `node.*`: All Node attributes defined on the Node.

Option context variables are referred to as `option.NAME` (more about [Job Options](job-options.html) in the next chapter.)

### Context Variable Usage

Context variables can be used in a few ways in a Job step, with slightly different sytanxes:

* Commands, Script Arguments and Job Reference Arguments

    :     `${ctx.name}`

* Inline Script Content (*see note*)

    :     `@ctx.name@`

    **Note**: The "Inline Script Content" variable expansion is **not** available for "Script File" steps.  The Script File is not rewritten at all when used for execution.

* Environment Variables (*see note*)

    :     `$RD_CTX_NAME`

    The syntax for Environment variables is that all letters become uppercase, punctuation is replaced with underscore, and the name is prefixed with `RD_`.

    **Note**: See the chapter [Administration - SSH - Passing Environment Variables Through Remote Commands](../administration/ssh.html#passing-environment-variables-through-remote-command) for information about requirements of the SSH server.

## Summary

At this point you should understand what a Job workflow is, the kinds
of steps they can contain and how to define a workflow.

Next, we'll cover more about Rundeck's Job Option features.
