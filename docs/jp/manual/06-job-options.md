% ジョブオプション
% Alex Honor; Greg Schueler
% November 20, 2010

様々なコマンドやスクリプトをジョブ化できます。
しかし全てのユースケースに対応するジョブを作ろうとすると、スクリプトの呼び出し方が少し違う程度のジョブを大量に作ることになってしまうでしょう。
それらの違いとは、往々にして環境やアプリケーションバージョンに関連します。
そのほかの部分については人が必要な情報を与えてジョブを実行しているにすぎません。

Any command or script can be wrapped as a Job. Creating a Job for
every use case will proliferate a large number of Jobs differing only
by how the Job calls the scripts. These
differences are often environment or application version
related. Other times only the person running the Job can provide the
needed information to run the Job correctly. 

スクリプトやコマンドをデータドリブンにしましょう。
そうすればより一般化でき、他のコンテキストでも再利用できます。
同じプロセスの変数をメンテナンスするより、ジョブが外部データからのオプションモデルで駆動するようにすることで、よりよい抽象化とカプセル化を期待できます。

Making your scripts and commands data driven, will also make them
more generic and therefore, reusable in different contexts. Rather than
maintain variations of the same basic process, letting Jobs be driven
by a model of options from externally provided data will lead to
better abstraction and encapsulation of your process.

Rundeck のジョブは、1 つ以上の名前付き *オプション* を定義させるために入力プロンプトをユーザへ出すよう設定できます。
名前付きパラメータと呼ばれる *オプション* モデルは、必須もしくはオプショナルにでき、ジョブが実行されるときにユーザに提示される選択肢の範囲が含まれています。

Rundeck Jobs can be configured to prompt a user for input by defining
one or more named *options*. An *option* models a named parameter that
can be required or optional and include a range of choices that will
be presented to the user when the Job is run.

ユーザは、値を入力するか選択肢メニューから選んでジョブへ受け渡します。
バリデーションパターンは、入力をオプションの要件通りにコンパイルすることを保証します。

Users supply options by typing in a value or selecting from a menu
of choices. A validation pattern ensures input complies to the
option requirement. Once chosen, the value chosen for the option is
accessible to the commands called by the Job.

オプション選択肢は固定か、動的ソースからモデリングされます。
固定選択肢は、ジョブ定義の中でコンマ区切りでモデリングされます。
オプション値を動的にしなければならない場合、ジョブがオプションデータを外部ソースから拾うために有効な URL を定義しなけれななりません。
ジョブが URL から外部ソースにアクセスできるようになると、Rundeck を他のツールと統合して、そのデータをジョブワークフローに組み込むことができるようになります。

Option choices can be modeled as a static set or from a dynamic
source. Static choices can be modeled as a comma separated list in the
job definition. When option values must be
dynamic, the Job can be defined to use a URL to retrieve option data
from an external source. Enabling Jobs to access external sources via
URL opens the door to integrating Rundeck with other tools and
incorporating their data into Job workflows. 

## ユーザへのプロンプト提示

ジョブオプションを定義することによる明確な影響は、ジョブ実行時のアピアランスです。
ユーザに、入力と選択が求められる "Choose Execution Options...（オプションを選択して下さい）" ページが提示されます。

The obvious effect from defining Job options is their appearance to
the user running the Job. Users will be presented a page called "Choose
Execution Options..." where input and menu choices must be configured.

コマンドラインユーザがシェルからジョブを実行する場合は、`run` シェルツールを通じて引数でオプションを明記します。

Command line users executing Jobs via the `run` shell
tool also will specify options as an argument string.

オプションをどのようにジョブへのユーザインタフェースの一部とするか考えることに時間をかける価値があります。
そうすることで、手続きの汎用化を次のレベルへとすすめる、いくつかのアイデアを得られます。

It is worth spending a moment to consider how options become
part of the user interface to Jobs and give some thought to this next
level of procedure formalization.

* ネーミングと説明に関する規約: ユーザがオプション名を呼んだだけで役割をイメージでき、説明を読めばその目的を判断できること
* 必須オプション: ユーザが入力しないと失敗するものについては必須オプションとすること
* 入力値の制約とバリデーション: オプションの値を絞り込む必要がある場合、その制御のためにセーフガードを作れることを頭に入れておきましょう

* Naming and description convention: Visualize how the user will read
  the option name and judge its purpose from the description you supply.
* Required options: Making an option required means the Job will fail
  if a user leaves it out.
* Input restrictions and validation: If you need to make the option
  value be somewhat open ended consider how you can create
  safeguards to control their choice.

## 入力タイプ

オプションの入力タイプは GUI での表示方式と同じで、それらがジョブ実行時に利用されます。

Option Input Types define how the option is presented in the GUI, and how it is used when the Job executes.

入力タイプ:

* "Plain" - 通常オプション。入力文字列が表示される
* "Secure" - セキュアオプション。入力は隠蔽され、DB にも保存されません。
* "Secure Remote Authentication" - リモート認証でのみ利用されるセキュアなオプションで、スクリプトやコマンド内では参照されません

Input types:

* "Plain" - a normal option which is shown in clear text
* "Secure" - a secure option which is obscured at user input time, and the value of which is not stored in the database
*  "Secure Remote Authentication" - a secure option which is used only for remote authentication and is not exposed in scripts or commands.

## セキュアオプション

平文やドロップダウンメニュー以外のパスワードプロンプト用に、オプションをセキュアとしてマークできます。
セキュアオプションの値は他のオプション値のように実行と一緒に保存されるということはありません。

Options can be marked as Secure, to show a password prompt in the GUI, instead of a normal text field or drop down menu.  Secure option values are not stored with the Execution as are the other option values.

セキュアオプションには二つのタイプがあります:

There are two types of Secure options:

* Secure - 入力値はスクリプトやコマンド内へ展開されます
* Secure Remote Authentication - 入力値はスクリプトやコマンド内に展開*されず*、ノードの認証と実行のためだけに Node Executor に利用されます

* Secure - these option values are exposed in scripts and commands.
* Secure Remote Authentication - these option values *are not* exposed in scripts and commands, and are only used by Node Executors to authenticate to Nodes and execute commands.

Secure オプションは複数の値を選択する入力はサポートしません。

Secure Options do not support Multi-valued input. 

また、Node Executors への入力にも使えません。
そのような用途には Secure Remote AUthentication オプションを利用して下さい。

Secure Options cannot be used as authentication input to Node Executors, you must use a Secure Remote Authentication option described below.

**重要**

"Secure" オプション値はジョブ実行時に Rundeck データべースへは保存されません。
しかし、その値はスクリプトやコマンド内へ展開されます。
このセキュリティ的な影響について承知しておいてください。
Secure オプションはスクリプトとコマンド内では他のオプション値と同じように利用できます。

"Secure" option values are not stored in the Rundeck database when the Job is executed, but the value that is entered 
is exposed to use in scripts and commands.  Make sure you acknowledge these security implications before using them. Secure options are available for use in scripts and command like any other option value: 

* 平文引数として利用する `${option.name}`
    * コマンドへの引数としてオプション値を使うとシステムプロセステーブル内で平文の値として展開されます
* リモート及びローカルスクリプト実行時の環境変数として使う `$RD_OPTION_NAME` 
    * ローカルとリモートスクリプトはこの値を環境変数として利用することが可能です
* リモートスクリプト内で展開される平文トークンとして使う `@option.name@` 
    * トークン展開を含むインラインスクリプトのワークフローステップはテンポラリファイルにいったん展開されます。そして展開されたファイル内には平文のオプション値があります。

* as plaintext arguments using `${option.name}`
    * Using the option value as an argument to a command could expose the plaintext value in the system process table
* as plaintext environment variables in remote and local script execution as `$RD_OPTION_NAME`
    * Local and possibly remote scripts may be passed this value into their environment
* as plaintext tokens expanded in remote scripts as `@option.name@`.
    * Inline Script workflow steps that contain a token expansion will be expanded into a temporary file, and the temp file will contain the plaintext option value.

注：ジョブリファレンスへの引数とする場合、他のセキュアオプションの値としてのみ通すことができます。[ジョブリファレンスでのセキュアオプションの利用](#ジョブリファレンスでのセキュアオプションの利用) を参照して下さい。

Note: that when passed as arguments to Job References, they can only be passed as the value of another Secure option.  See [Using Secure Options with Job References](#using-secure-options-with-job-references).

### セキュアリモート認証オプション

ノード実行するための組み込みの SSH プロパイダは SSH, Sudo 認証メカニズムのため、パスワードを利用します。
パスワードはジョブ内に定義されたセキュアリモート認証オプションで供給することができます。

The built-in [SSH Provider](plugins.html#ssh-provider) for node execution allows using passwords for SSH and/or Sudo authentication mechanisms, and the passwords are supplied by Secure Remote Authentication Options defined in a Job.

セキュアリモート認証オプションは平文・セキュアオプションと比較していくつかの制約を持ちます。

Secure Remote Authentication Options have some limitations compared to Plain and Secure options:

通常のスクリプトとコマンドオプション値の展開にユーザが入力した値を使うことはできません。
言い換えると、このオプションはリモート認証のためにしか利用できません。

* The values entered by the user are not available for normal script and command option value expansion. This means that they can only be used for the purposes of the Remote Authentication.

### ジョブリファレンスでのセキュアオプションの利用

[ジョブ参照ステップをワークフロー内で定義する](job-workflows.html#ジョブ参照ステップ)と、それに引き渡す引数を指定することができます。
セキュアオプションとセキュアリモート認証オプションの値を最上位層のジョブからジョブ参照へ引き渡すことができます。
しかし、*あるオプション値を他の違うタイプのオプションの値として引き渡すことは不可能です。*
そのため、親ジョブはオプションタイプが子ジョブと同じならジョブ参照へオプション値を引き渡すことができます。

When you [define a Job Reference step in a workflow](job-workflows.html#job-reference-step), you can specify the arguments that are passed to it. You can pass Secure Option values and Secure Remote Authentication Option values from a top-level job to a Job Reference, but option values *cannot be passed into another option of a different type*. So a parent job can only pass option values to the Job reference if the option type is the same between the jobs.

この制約はこれらのオプションのセキュリティデザインを保つためのものです:

This constraint is to maintain the security design of these options:

1. セキュアオプションは Rundeck 実行データベースに保存してはならない。そのため、平文オプション値を利用してはならない。
2. セキュアリモート認証オプションはスクリプトやコマンド内で利用してはならない。そのため、セキュアや平文オプション値を利用してはならない

1. Secure options should not to be stored in the Rundeck execution database, so must not be used as plain option values.
2. Secure Remote Authentication options should not be used in scripts/commands, so must not be used as Secure or Plain option values.

例を挙げます。Job A, Job B の2つのジョブがそれぞれ以下の定義で存在するとしましょう:

As an example, here is are two jobs, Job A and Job B, which define some options:

* Job A
    * オプション "plain1" - 平文
    * オプション "secure1" - セキュア
    * オプション "auth1" - セキュアリモート認証
* Job B
    * オプション "plain2" - 平文
    * オプション "secure2" - セキュア
    * オプション "auth2" - セキュアリモート認証

* Job A
    * Option "plain1" - Plain
    * Option "secure1" - Secure
    * Option "auth1" - Secure remote authentication
* Job B
    * Option "plain2" - Plain
    * Option "secure2" - Secure
    * Option "auth2" - Secure remote authentication

もし Job A が Job B へのジョブ参照を定義しているとすると、可能となるマッピングは以下の通りです:

If Job A defines a Job reference to call Job B, then the only valid mapping is shown below:

* plain1 -> plain2
* secure1 -> secure2
* auth1 -> auth2

よって、ジョブ参照のための引数はこのようになるはずです。

So the arguments for the Job Reference might look like this:

    -plain2 ${option.plain1} -secure2 ${option.secure1} -auth2 ${option.auth1}

注: もしルールをまもらずに引数を定義すると、セキュアとセキュアリモート認証オプションはジョブ参照が呼ばれたときにセットされません。平文オプションはコマンドもしくはスクリプトの引数として振る舞い、残りは解釈不能のプロパティリファレンスとなります。

Note: If you define arguments in the wrong manner, then the Secure and Secure Remote Authentication options will not be set when the Job reference is called.  Plain options will behave the way they do in Command or Script arguments, and be left as-is as uninterpreted property references.

## オプションエディタ

オプションは保存されたジョブに作成できます。ジョブ編集ページには存在するオプションのサマリを表示するエリアがあり、新規に追加するか、既存の物を編集するためのリンクがあります。

Options can be created for any stored Job. The Job edit page contains
an area displaying a summary to existing options and a link to add new
ones or edit existing ones.

![Add option link](../figures/fig0501.png)

オプションサマリはそれぞれのオプションと定義があればそのデフォルト値を表示します。

The option summary shows each option and its default value if it defines
them.

"edit" リンクを押すとオプションエディタが開かれます。

Clicking the  "edit" link opens the options editor. 

![Option editor](../figures/fig0503.png)

オプションエディタはそれぞれのオプションについて、拡大されたサマリを表示します。
それぞれのオプションは利用法のサマリ、説明、値リスト、制約と共にリストアップされます。
"Add an option" リンクを押すと、新しいパラメータを定義するためのフォームがオープンします。
"Close" リンクを押すとエディタはたたまれてサマリビューに戻ります。

The options editor displays an expanded summary for each defined
option. Each option is listed with its usage summary,
description, values list and any restrictions. Pressing the "Add an
option" link will open a form to define a new parameter. Pressing the
"Close" link will collapse the options editor and return back to the
summary view.

オプションエディタのいずれかの行にマウスオーバすると、ハイライトされたオプションの削除や編集用リンクが表示されます。
削除アイコンを押すと、本当にこのオプションをジョブから削除しても良いか確認するプロンプトが表示されます。
"edit" リンクをクリックするとそのオプションの定義を変更するためのフォームが開きます。

Moving the mouse over any row in the options editor reveals links to
delete or edit the highlighted option. Pressing the remove icon will
display a prompt confirming you want to delete that option from the Job.
Clicking the "edit" link opens a new form that lets you modify all
aspects of that option.

オプションはジョブ定義の一部として定義され、Rundeck サーバにロードされます。
ジョブ定義の具体的な中身について興味がある場合は [job-v20(5)(XML)](http://rundeck.org/docs/manpages/man5/job-v20.html), [job-yaml-v12(5)(YAML)](http://rundeck.org/docs/manpages/man5/job-yaml-v12.html), [rd-jobs(1)](http://rundeck.org/docs/manpages/man1/rd-jobs.html) マニュアルページを参考にして下さい。

Options can also be defined as part of a job definition and later
loaded to the Rundeck server. See [job-v20(5)](../manpages/man5/job-v20.html)(XML) and [job-yaml-v12(5)](../manpages/man5/job-yaml-v12.html)(YAML) and 
[rd-jobs(1)](../manpages/man1/rd-jobs.html) manual
pages if you prefer using an textual Job definition.

## オプションを定義する

新しいオプションは "Add an option" リンクをクリックすれば定義でき、定義されたものについては "edit" リンクを押せば変更できます。

New options can be defined by pressing the "Add an option" link while
existing ones can be changed by pressing their "edit" link.

![Option edit form](../figures/fig0502.png)

オプション定義フォームはいくつかのエリアに分けられています:

The option definition form is organized into several areas:

Identification

:    オプション名と説明を入れて下さい。名前は他のジョブが受け入れ可能な引数の一部となり、説明は実行中ジョブのヘルプテキストとなります。
     デフォルト値はオプションが表示されたときに GUI 内で既にセレクト状態になります。

:    Here you provide the option's name and description. The name
     becomes part of acceptable arguments to the Job while the
     description will be provided as help text to users running the Job.
     
     The Default Value will be pre-selected in the GUI when the option is presented.

Input Type

:   "Plain"（平文）・"Secure"（セキュア）・"Secure Remote Authentication"（セキュアリモート）認証から選んで下さい。"Plain" 以外を選ぶとマルチ選択オプションは利用不可になります。
:   Choose between "Plain", "Secure" and "Secure Remote Authentication". For input types other than "Plain", the multi-valued option will be disabled.

Allowed values

:    選択可能な入力値のモデルです。値のリストかオプションデータを提供するサーバへの URL を含めることができます。
     値にはコンマ区切りのリストを指定することができますが、[以下に記載するように](job-options.html#リモートオプション値) "remote URL" を使った外部ソースからのリクエストも可能です。

:    Allowed values provide a model of possible choices.
     This can contain a static list of values or a URL to a server
     providing option data. Values can be specified as a comma
     separated list as seen above but can also be requested from an
     external source using a "remote URL" [See below](job-options.html#remote-option-values).
     

Restrictions

:    入力を受け入れたり提示するための基準を定義します。オプションの選択肢は "Enforced drom values" 制約を利用してコントロールできます。
     もし "true" ならば、Rundeck はポップアップメニューのみを出します。"false" であれば、テキストフィールドが提示されます。
     正規表現を "Match Regular Expression" フィールドに入れるとジョブ実行時にそれが評価されます。

:    Defines criteria on which input to accept or present. Option
     choices can be controlled using the "Enforced from values"
     restriction. When set "true", Rundeck will only present a
     popup menu. If set "false", a text field will also be presented. 
     Enter a regular expression in the "Match Regular Expression"
     field the Job will evaluate when run.

Requirement

:    そのオプションが与えられている場合のみ実行を許可することを示します。"No" を選択すると、そのオプションは必須となりません。"Yes" を選ぶと要求されます。
:    Indicates if the Job can only run if a choice is provided for
     that Option. Choosing "No" states the option is not required
     Choose "Yes" to state the option is required.
     
     デフォルト値がオプションにセットされる場合、"Yes" が選択されると、コマンドラインか API からの実行時に引数が無ければデフォルト値が自動でオプションにセットされるようになります。
     If a Default Value is set for the option, then this value will automatically be set for the option if it is Required, even if not specified among the arguments when executing a job via the command-line or API.

Multi-valued

:    ユーザの入力が複数の値で構成できるかどうかを定義します。"No" を選ぶと、1つの値しか入力できません。"Yes" を選ぶと許可されている値の選択と、ユーザによる入力の組み合わせを利用できるようになります。ジョブ実行時に複数の入力値を区切るためにデリミタ文字列を利用します。
:    Defines if the user input can consist of multiple values. Choosing "No" states that only a single value can chosen as input. Choosing "Yes" states that the user may select multiple input values from the Allowed values and/or enter multiple values of their own.  The delimiter string will be used to separate the multiple values when the Job is run.

オプション定義ができたら、"Save" ボタンをおしてジョブ定義にそれを追加してください。
"Cancel" ボタンを押すと変更を捨ててフォームがクローズします。

Once satisfied with the option definition, press the "Save" button to
add it to the Job definition. Pressing the "Cancel" button will
dismiss the changes and close the form.


## リモートオプション値

オプション値のモデルは外部リソースから検索することが出来ます。
オプションで `valuesUrl` を定義すると、その URL から入力を許可するモデルが取得されます。

A model of option values can be retrieved from an external source.
When the `valuesUrl` is specified for an Option, then the model of
allowed values is retrieved from the specified URL. 

これは、Rundeck を他のシステムに依存するプロセスを調整するために利用したいときに便利です。たとえば:

This is useful in a couple of scenarios when Rundeck is used to 
coordinate process that depend on other systems:

* Hudson などの CI サーバやビルドで生成されるパッケージやバイナリのデプロイ
    * Hudson の最近のビルド成果リストをオプションデータとして読み込むことができます。その結果、ユーザはデプロイ対象の適切なパッケージ名をリストから選ぶことができます。
* CMDB に定義されている有効な環境セットを選ぶ
* ジョブへの入力値に、違うシステムによって生成された値セットのいくつかを選択しなければならない時

* Deploying packages or artifacts produced by a build or CI server, e.g. Hudson.
    * A list of recent Hudson build artifacts can be imported as Options data, so that a User can pick an appropriate package name to deploy from a list.
* Selecting from a set of available environments, defined in a CMDB
* Any situation in which input variables for your Jobs must be selected from some set of values produced by a different system.

参照：[Chapter 9 - オプションモデルプロバイダ](job-options.html#オプションモデルプロバイダ).

See [Chapter 9 - Option Model Provider](job-options.html#option-model-provider).

## Script usage

オプション値は引数としてスクリプトに通るか、スクリプト内の名前付きトークンを通じて参照されます。
全てのオプションは、`option.NAME` のように Options コンテキスト内に定義されます。

Option values can be passed to scripts as an argument or referenced
inside the script via a named token. Each option value is defined in the Options context variables as `option.NAME`.

[ジョブワークフロー - コンテキスト変数](job-workflows.html#context-variables) を参照して下さい。

See the [Job Workflows - Context Variables](job-workflows.html#context-variables) Section.

**例:**

**Example:**

"hello" という名前で、"message" というオプションをもつジョブがあるとします。

A Job named "hello" and has an option named "message".

"Hello" ジョブのオプションシグネチャは `-message <>` になります。

The "hello" Job option signature would be: `-message <>`.

![Option usage](../figures/fig0504.png)

引数は `${option.message}` として定義されてスクリプトに渡ります。

The arguments passed to the script are defined as `${option.message}`.

スクリプトの中身はこのようになります。

Here's the content of this simple script. 

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ {.bash}
    #!/bin/sh    
    echo envvar=$RD_OPTION_MESSAGE ;# read from environment
    echo args=$1                   ;# comes from argument vector
    echo message=@option.message@  ;# replacement token
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 

ユーザが "hello" ジョブを走らせると、"message" 値用のプロンプトが出ます。

When the user runs the "hello" job they will be prompted for the
"message" value.

![Option entered](../figures/fig0505.png)

"howdy" を入力したと仮定しましょう。ジョブのアウトプットは以下のようになるはずです。

Let's assume they entered the word "howdy" in response. 
The output of the Job will be:

    envar=howdy
    args=howdy    
    message=howdy    

これはオプションがセットされていないときの挙動を知るために大切です。
必須ではくデフォルト値もないオプションを定義しているとそのようなことが起こり得ます。

It's important to know what happens if the option isn't set. This can
happen if you define an option that is not required and do not give it
a default value. 

message オプション無しでジョブが実行された場合を考えてみて下さい。きっと以下のようなアウトプットになるでしょう。

Let's imagine the Job was run without a message option supplied, the
output would look like this:

    envar=
    args=
    message=@option.message@

これに対処するためのいくつかの tips があります。

Here are some tips to deal with this possibility:

環境変数:

Environment variable:

:    予防として、変数の存在をテストして（恐らく）デフォルト値をセットしているでしょう。
     変数の存在をチェックするために以下の構文を使っているかもしれません

         test -s  $RD_OPTION_NAME

:    As a precaution you might test existence for the variable and
     perhaps set a default value.
     To test its existence you might use: 

         test -s  $RD_OPTION_NAME

:    Bash の機能を使えば、テストしつつデフォルト値をセットすることが可能です。

         ${RD_OPTION_NAME:=mydefault} 

     You might also use a Bash feature that tests and defaults it to a
     value:

         ${RD_OPTION_NAME:=mydefault} 


置換トークン
Replacement token	 

:    オプションがセットされていないと、トークンはスクリプト内で放置されるでしょう。
     スクリプトの実装を以下のように変更することで、少しだけ堅牢にすることができます。

        message=@option.message@
        atsign="@"
        if [ "$message" == "${atsign}option.message${atsign}" ] ; then
           message=mydefault
        fi

:    If the option is unset the token will be left alone inside the
     script. You might write your script a bit more defensively and
     change the implementation like so:

        message=@option.message@
        atsign="@"
        if [ "$message" == "${atsign}option.message${atsign}" ] ; then
           message=mydefault
        fi

## Calling a Job with options

ジョブはコマンドラインの `run` シェルツールから、もしくは他のジョブのワークフローから実行することができます。

Jobs can be invoked from the command line using the `run`
shell tool or as a step in another Job's workflow.

オプションを指定するフォーマットは `-name value` です。

The format for specifying options is `-name value`.

`run` コマンドのあとに、ダブルハイフンでオプションとその値を指定します:

Using the `run` command pass them after the double hyphen:

    run -i jobId -- -paramA valA -paramB valB
    run -j group/name -p project -- -paramA valA -paramB valB

XML 定義内では、これらを `arg` 要素として挿入します:

Inside an XML definition, insert them as an `arg` element:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ {.xml}
<command>
    <jobref group="test" name="other tests">
        <arg line="-paramA valA -paramB valB"/>
    </jobref>
</command>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 

より詳細については [run(1)](../manpages/man1/run.html), [job-v20(5)](../manpages/man5/job-v20.html) のマニュアルページを参照して下さい。

Consult the [run(1)](../manpages/man1/run.html) and [job-v20(5)](../manpages/man5/job-v20.html) manual pages for additional
information.


## オプションモデルプロパイダ

オプションモデルプロバイダは、ジョブに定義されたオプションに対して、リモートサービスやデータベースから入力を受けることを可能にするメカニズムです。

The Option model provider is a mechanism to allow the Options defined for a Job to have some of the possible input values provided by a remote service or database.  

オプションモデルプロバイダは基本的にオプション毎に設定されます。

Option model providers are configured on a per-Option basis (where a Job may have zero or more Options).

### 必須条件 ###

1. オプションモデルデータは [JSON フォーマット](http://www.json.org)でなければなりません。
2. HTTP(S) アクセス可能であるか Runeck サーバのローカルディスクに配置する必要があります。
3. 二つの内のいずれかの JSON 構造でなければなりません。
    * 文字列配列
    * もしくは、Map の配列 (`name` and `value` 形式)

1. Options model data must be [JSON formatted](http://www.json.org).
2. It must be accessible via HTTP(S) or on the local disk for the Rundeck server.
3. It must be in one of two JSON structures, *either*:
    * An array of string values
    * OR, an array of Maps, each with two entries, `name` and `value`.

### 設定 ###

それぞれのジョブオプションのエントリは可能な値をリモート URL から取得するよう設定することができます。
もしジョブを [job.xml ファイルフォーマット](../manpages/man5/job-v20.html#option)で編集する場合、シンプルに `valuesUrl` 属性を `<option>` に加えて下さい。
もし Rundec web GUI から編集する場合、URL をオプションの Remote URL フィールドに入れて下さい。

Each Option entry for a Job can be configured to get the set of possible values from a remote URL.  If you are authoring the Jobs via [job.xml file format](../manpages/man5/job-v20.html#option), simply add a `valuesUrl` attribute for the `<option>`.  If you are modifying the Job in the Rundeck web GUI, you can entry a URL in the "Remote URL" field for the Option.

例：

    <option valuesUrl="http://site.example.com/values.json" ...

*注*：File URL スキーマも利用可能です (例 `file:/path/to/job/options/optA.json`).

*Note*: File URL scheme is also acceptable (e.g, `file:/path/to/job/options/optA.json`).

オプション値のデータは、必ず以下に示す JSON データフォーマットに沿ったものを返さなければなりません。

The value data must be returned in JSON data format described below.

### JSON フォーマット

戻すデータについては3つのスタイルがサポートされています。
シンプルリスト、シンプルオブジェクト、そして name/value リストです。
シンプルリストでは、ジョブ実行時にリスト値がポップアップリストで表示されます。
シンプルオブジェクトか name/value ペアが返されると、`name` がリストに表示され、`value` が実際の入力値として利用されます。

Three styles of return data are supported: simple list, simple object, and a name/value list. For a simple list, the list values will be displayed in a pop-up list when running the Job.  If a simple object or name/value pairs are returned, then the `name` will be displayed in the list, but the `value` will be used as the input.

*例*

*Examples*

シンプルリスト:

Simple List:

    ["x value for test","y value for test"]

値がセレクトメニューに投入されます。

This will populate the select menu with the given values.

シンプルオブジェクト:

Simple Object:

    { "Name": "value1", "Name2":"value2" }

names が表示され、value が使われるようにセレクトメニューへ投入されます。

This will populate the select menu to show the names and use the values.

Name Value List:
 
    [
      {name:"X Label", value:"x value"},
      {name:"Y Label", value:"y value"},
      {name:"A Label", value:"a value"}
    ] 

### カスケーディングリモートオプション

カスケーディングオプションは、ジョブ実行時にユーザが他のオプション値用に入力した値をリモートバリュー URL で利用することを可能にします。
ユーザが入力するか値を選択すると、リモートの JSON が現状のオプションに更新されます。

Cascading options allow an option's Remote values URL to embed the values
entered by the user for other options when executing a Job.  When the user
enters or selects a value for one of the other options, then the remote JSON is
refreshed for the current option.

これは、階層的もしくは独立したオプション値のセットを宣言するメカニズムを実現します。

This provides a mechanism for declaring hierarchical or dependent sets of option
values.

例. あなたが "repository" オプションを選択し、他のオプションでそのリポジトリ内の特定の "branch" を選択したいとします。
オプションプロパイダを選択された "repository" 値に正常に応答するように定義して、リモートオプション URL を "repository" オプション値への参照を含むように定義します。
Rundeck GUI は "branch" オプション値をロードする時に JSON 値を remote URL から再読込し、正しい "repository" 値を挿入します。
ユーザが選択中のリポジトリを変更すれば、branch 値も自動で更新されます。

E.g. if you wanted one option to choose a "repository", and  another option to
select a specific "branch" within that repository.  Define your option provider
to respond correctly based on the  selected "repository" value, and define your
Remote option URL to include a reference to the "repository" option value. The
Rundeck GUI will then reload the JSON values from the remote URL and insert the
correct value of the "repository" option when loading the "branch" option
values. If the user changes the selected repository, then the branch values will
be automatically refreshed.

あるオプションと別のオプションの依存関係を、リモートバリュー URL によるプロパティリファレンスを埋め込むことで宣言できます。
プロパティリファレンスは、`${option.[name].value}` の形式をとります。もし "http://server/options?option2=${option.option2.value}" のようなリモートバリューでオプションを宣言すれば、そのオプションは "option2" オプションの値に依存するようになります。

You can declare a
dependency of one option to another by embedding property references within the
remote values URL.  The property reference is of the form
`${option.[name].value}`.  If you declare an option with a remote values URL
like "http://server/options?option2=${option.option2.value}", then that option
will depend on the value of the "option2" option.

GUI では、オプションがロードされると、option2 がまず表示され、その後 opiton2 が選択されたら option1 が一度だけ読み込まれます。

In the GUI, when the options are loaded, option2 will be shown first, and the
remote values for option1 will only be loaded once a value has been selected for
option2, and the value will be placed in the URL when it is loaded.

オプションが他のオプションと依存関係をもち、URL をロードしたときに値がセットされていない場合は、埋め込まれた参照は ""（空文字列）として評価されます。

If an option has dependencies on other options that do not have a value set,
then the  embedded references will evaluate to "" (empty string) when loading
the URL.

オプションが他のオプションと依存関係を持ち、リモートバリューの [JSON data](#json-data) が空（空リスト or 空オブジェクト）なら、ユーザに必要な値を選択させるよう促す GUI が表示されます。
これは、オプションモデルプロパイダに、いくつかあるいは全ての依存関係を渡すようにできます。

If an option has dependencies on other options and the remote values [JSON 
data](#json-data) is empty (empty list or empty object), then the option shown in
the GUI will indicate that the user should select values for the necessary
options.  This allows Option model providers to indicate that some or all of the
dependent option values are necessary for the current option before showing the
input for the option.

1 つ以上のオプションが依存関係を持つ時は、ある変更によって、他のオプション値がリモート URL からリロードされる可能性があります。

It is possible to have dependencies on more than one option, and any change to
one of the dependencies will cause the option values to be reloaded from the
remote URL.

注: オプション値同士で循環参照を形成すると、自動リロードがオフになります。この場合、ユーザは手動でリロードボタンをクリックしてオプション値を更新する必要があります。

Note: It is also possible to declare a cycle of dependencies between option
values, which will cause the automatic reloading to be disabled.  In this case
the user must manually click the reload button to reload the option values if a
dependency has changed.

### リモート URL 内での変数展開

"valuesUrl" に定義された URL はリモートリクエスト時に特定のジョブコンテクスト項目が入る変数を埋め込めます。
これはジョブに対して URL をより汎用的に、コンテクスチュアルにします。

The URL declared for the "valuesUrl" can embed variables which will be
filled with certain job context items when making the remote request. This
helps make the URLs more generic and contextual to the Job.

展開パターンとして、ジョブコンテクストとオプションコンテクストがあります。

Two types of expansions are available, Job context, and Option
context.

URL 内にジョブ情報を含めるには、${job._property_} 形式で変数を指定します。

To include job information in the URL, specify a variable of the form
${job._property_}.

ジョブコンテクストで埋め込めるプロパティは以下の通りです:

Properties available for Job context:

* `name`: ジョブ名
* `group`: グループ名
* `description`: 説明
* `project`: プロジェクト名
* `argString`: ジョブのデフォルト引数の文字列

* `name`: Name of the Job
* `group`: Group of the Job
* `description`: Job description
* `project`: Project name
* `argString`: Default argument string for a job

URL 内にオプション情報を含めるには、${option._property_} 形式で変数を指定します。

To include Option information in the URL, specify a variable of the
form ${option._property_}:

オプションコンテキストで埋め込めるプロパティは以下の通りです:

Properties available for Option context:

* `name`: 現在のオプション名

* `name`: Name of the current option

[カスケーディングリモートオプション](#カスケーディングリモートオプション) 値の情報を URL 内に含めるには、${option._name_.value} で指定します:

To include [Cascading remote option](#cascading-remote-options) values information in the URL, specify a variable of the
form ${option._name_.value}:

* `option.[name].value`: 選択された値を他のオプション名で置き換える。もしオプションがセットされていなければ空文字列（""）で置き換える。

* `option.[name].value`: substitutes the selected value of another option by name. If the option is not set, then a blank string ("") will be substituted.

*例*

*Examples*

    http://server.com/test?name=${option.name}

オプション名は "name" クエリパラメータとして URL に渡る。

Passes the option name as the "name" query parameter to the URL.

    http://server.com/test?jobname=${job.name}&jobgroup=${job.group}

ジョブ名とグループがクエリパラメータとして渡る。

Passes the job name and group as query parameters.
    
    http://server.com/branches?repository=${option.repository.value}

選択された "repository" オプションの値か、セットされていなければ ""（空文字列）が渡ります。
このオプションは "repository" オプションに依存していて、"repository" 値が変化すれば、リモートオプション値もリロードされます。

Passes the value of the selected "repository" option, or "" (blank) if it is not set. This option becomes
a dependent of the "repository" option, and if the "repository" value changes, the remote option values
for this option will be reloaded.

### リモートリクエストの失敗

リモートオプション値の取得に失敗した場合、GUI フォームは警告メッセージを出します。

If the request for the remote option values fails, then the GUI form
will display a warning message:

![](../figures/fig0901.png)

この場合、オプションはテキストフィールドからセットされた値しか使えません。
    
In this case, the option will be allowed to use a textfield to set the value.

### 実装例 ###

2 セクションに渡って、オプションモデルプロパイダとして振る舞うシンプルな CGI スクリプトを用いた例を説明します。

The following two sections describe examples using simple CGI scripts
that act as option model providers.
 
#### Hudson アーティファクトオプションプロパイダ

end-to-end リリースプロセスでは、ビルド成果物の取得と、それを後で配布するための中央リポジトリへのアップが頻繁に求められます。
Hudson のような継続的インテグレーションサーバはシンプルなジョブ設定ステップでビルド成果物を特定できます。
Hudson API はビルドに成功した成果物の一覧をシンプルな HTTP GET リクエストで取得できるネットワークインタフェースを提供しています。

An end-to-end release process often requires obtaining build artifacts
and publishing them to a central repository for later distribution.
A continuous integration server like [Hudson] makes identifying the
build artifacts a simple Job configuration step. The [Hudson API]
provides a network interface to obtain the list of artifacts from
successful builds via a simple HTTP GET request.

Acme は RPM として成果物を構築し、それらを識別するためにビルドジョブを設定しています。
オペレーションチームは自動ビルドで生成された成果物のバージョンを選択できるジョブをつくりたいと思っています。

Acme builds its artifacts as RPMs and has configured their build job
to identify them. The operations team wants to create Jobs that would
allow them to choose a version of these artifacts generated by the
automated build.

[JSON] ドキュメントを生成して Hudson に情報を要求するシンプルな CGI スクリプトがあります。
クエリパラメータに Hudson サーバ、Hudson ジョブ、成果物のパスを利用できます。
ジョブライターは、成果物リストをオプションモデルとして受け取り、結果をジョブユーザにメニューとみせるためにパラメータ化された CGI スクリプトへの URL を指定できます。

A simple CGI script that requests the information from Hudson and then
generates a [JSON] document is sufficient to accomplish this. The CGI
script can use query parameters to specify the Hudson server, hudson job
and artifact path. Job writers can then specify the parameterized URL
to the CGI script to obtain the artifacts list as an options model
and present the results as a menu to Job users.

以下に示すコードは、成果物の情報を含む XML を取得するために [curl] コマンドを呼び出し、それをパースするために [xmlstarlet] を利用しています。

The code listing below shows the the CGI script essentially does a
call to the [curl] command to retrieve the XML document
containing the artifacts information and then parses it using
[xmlstarlet].
 
File listing: hudson-artifacts.cgi
 
    #!/bin/bash
    # Requires: curl, xmlstarlet
    # Returns a JSON list of key/val pairs
    #
    # Query Params and their defaults
    hudsonUrl=https://build.acme.com:4440/job
    hudsonJob=ApplicationBuild
    artifactPath=/artifact/bin/dist/RPMS/noarch/
    
    echo Content-type: application/json
    echo ""
    for VAR in `echo $QUERY_STRING | tr "&" "\t"`
    do
      NAME=$(echo $VAR | tr = " " | awk '{print $1}';);
      VALUE=$(echo $VAR | tr = " " | awk '{ print $2}' | tr + " ");
      declare $NAME="$VALUE";
    done

    curl -s -L -k $hudsonUrl/${hudsonJob}/api/xml?depth=1 | \
      xmlstarlet sel -t -o "{" \
        -t -m "//build[result/text()='SUCCESS']" --sort A:T:L number  \
        -m . -o "&quot;Release" -m changeSet/item -o ' ' -v revision -b \
        -m . -o ", Hudson Build " -v number -o "&quot;:" \
        -m 'artifact[position() = 1]' -o "&quot;" -v '../number' -o $artifactPath -o "{" -b \
        -m 'artifact[position() != last()]' -v 'fileName' -o "," -b \
        -m 'artifact[position() = last()]' -v 'fileName' -o "}&quot;," \
        -t -o "}"

このスクリプトを運用している web サーバの CGI 動作可能なディレクトリに配置したら、`curl` をつかって直接リクエストすることでテストできます。

After deploying this script to a CGI enabled directory on the
operations web server, it can be tested directly by requesting it using `curl`.

    curl -d "hudsonJob=anvils&artifactPath=/artifact/bin/dist/RPMS/noarch/" \
        --get http://opts.acme.com/cgi/hudson-artifacts.cgi

サーバレスポンスは以下の例のような JSON データである必要があります:

The server response should return JSON data resembling the example below:

    [ 
      {name:"anvils-1.1.rpm", value:"/artifact/bin/dist/RPMS/noarch/anvils-1.1.rpm"}, 
      {name:"anvils-1.2.rpm", value:"/artifact/bin/dist/RPMS/noarch/anvils-1.2.rpm"} 
    ]	

ここで、ジョブはオプションデータを以下のようにリクエストできます:

Now in place, jobs can request this option data like so:

     <option name="package" enforcedvalues="true" required="true"
        valuesUrl="http://ops.acme.com/cgi/hudson-artifacts.cgi?hudsonJob=anvils"/> 

Rundeck UI はパッケージ名をメニュー内に表示し、それが選択されると、ジョブは Hudson サーバ上のビルド成果物へのパスを保持することになります。

The Rundeck UI will display the package names in the menu and once
selected the Job will have the path to the build artifact on the
Hudson server.

[Hudson]: http://hudson-ci.org/
[Hudson API]: http://wiki.hudson-ci.org/display/HUDSON/Remote+access+API
[JSON]: http://www.json.org/

#### Yum リポジトリオプションモデルプロパイダ

[Yum] は [RPM] パッケージ管理を自動化してくれるグレートなツールです。Yum があれば、管理者はパッケージをリポジトリへ登録し、yum クライアントツールを依存関係に沿って自動でインストールするために利用できます。
Yum は [repoquery] と呼ばれる rpm クエリとよく似た問い合わせを Yum リポジトリにするための、便利なコマンド群を含んでいます。

[Yum] is a great tool for automating [RPM] package management. With Yum,
administrators can publish packages to the repository and then use the
yum client tool to automate the installation of packages along with
their declared dependencies. Yum includes a command
called [repoquery] useful for
querying Yum repositories similarly to rpm queries.

Acme はアプリケーションのリリースパッケージを配布するための自分自身の Yum リポジトリーをセットアップします。
Acme 管理者は、どのパッケージが与えられた能力を供給できるかを知るためのオプションモデルを提供したいです。

Acme set up their own Yum repository to distribute application release
packages. The Acme administrator wants to provide an option model to Jobs that
need to know what packages provide a given capability.

以下に示すコードは、repoquery コマンドをシンプルにラップして JSON データで結果を整形するものです。

The code listing below shows it is a simple wrapper around the
repoquery command that formats the results as JSON data.

File listing: yum-repoquery.cgi
    
    #!/bin/bash
    # Requires: repoquery
    # 
    # Query Params and their defaults
    repo=acme-staging
    label="Anvils Release"
    package=anvils
    max=30
    #
    echo Content-type: application/json
    echo ""
    for VAR in `echo $QUERY_STRING | tr "&" "\t"`
    do
      NAME=$(echo $VAR | tr = " " | awk '{print $1}';);
      VALUE=$(echo $VAR | tr = " " | awk '{ print $2}' | tr + " ");
      declare $NAME="$VALUE";
    done

    echo '{'
    repoquery --enablerepo=$repo --show-dupes \
      --qf='"${label} %{VERSION}-%{RELEASE}":"%{NAME}-%{VERSION}-%{RELEASE}",' \
      -q --whatprovides ${package} | sort -t - -k 4,4nr | head -n${max}
    echo '}'

このスクリプトを運用している web サーバの CGI 動作可能なディレクトリに配置したら、`curl` をつかって直接リクエストすることでテストできます。

After deploying this script to the CGI enabled directory on the
operations web server, it can be tested directly by requesting it using `curl`.

    curl -d "repo=acme&label=Anvils&package=anvils" \
        --get http://ops.acme.com/cgi/yum-repoquery.cgi
 
サーバレスポンスは以下の例のような JSON データである必要があります:

The server response should return JSON data resembling the example below:

    TODO: include JSON example
 
ここで、ジョブはオプションデータを以下のようにリクエストできます:

Now in place, jobs can request the option model data like so:

     <option name="package" enforcedvalues="true" required="true"
        valuesUrl="http://ops.acme.com/cgi/yum-repoquery.cgi?package=anvils"/> 

Rundeck UI はパッケージ名をメニュー内に表示し、それが選択されると、ジョブはマッチしたパッケージバージョンを保持することになります。

The Rundeck UI will display the package names in the menu and once
selected, the Job will have the matching package versions.
 
[Yum]: http://yum.baseurl.org/
[RPM]: http://www.rpm.org/
[repoquery]: http://linux.die.net/man/1/repoquery


## まとめ 

この章を読んだので、ジョブをオプションを含めて走らせる方法や、それを変更する方法について理解できたはずです。
オプションデータを生成してあなたのジョブで使うことに興味がでてきたら、[例から学ぶ Rundeck](#rundeck-by-example)の[オプションモデルプロパイダの例](rundeck-by-example.html#オプションモデルプロパイダの例)を参照してください。

After reading this chapter you should understand how to run Jobs with
options, as well as, add and edit them. If you are interested in
generating option data for one of your jobs, see the
[option model provider](rundeck-by-example.html#option-model-provider-examples) section in the
[Examples](#rundeck-by-example) chapter.
