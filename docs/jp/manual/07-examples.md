% 例から学ぶ Rundeck
% Alex Honor; Greg Schueler
% November 20, 2010

この章では、Rundeck を用いた様々なソリューションを表す実践例を説明します。出てくる例は、これまでの章で紹介した概念や機能を実践する際の助けとなることに焦点を当てています。抽象的な例を挙げるのではなく、オンラインアプリケーションサービスを経営する架空の組織 Acme Avils の状況に合わせた例を挙げていきます。

## Acme Anvils 

Acme Anvils はオンラインで新品・中古の金敷き台(anvils)を売っている架空のスタートアップです。会社の中には金敷き台を売るアプリケーションの開発とサポートに関わる 2 つのチームが存在します。まだ新しい会社なので、本番環境へのアクセスはしっかり管理されていません。どちらのチームにもソースコードの変更によってミスや障害を起こす可能性があります。シニアマネージャー達がチームに新しい機能のリリースを出来る限り頻繁にさせようと夢中になっているからです。残念ながら、これは別の問題を引き起こしていました：Acme Anvil のウェブサイトは、メモリを大量に喰い、偶発的にリスタートが起こっていたのです。

リスタート処理には 2 つの方法があります: "kill" と "normal" です。"kill" でのリスタートには、アプリケーションが一切レスポンスしなくても良い時間が必要になります。"normal" でのリスタートには、十分な量のメモリを必要とします。

緊急事態または急いでいる状況下で、開発者がリスタートの実行を行うのか、システムの管理者がリスタートの実行を行うのかは全く別物です。なぜならソフトウェアを書いている開発者は、アプリケーションの観点からリスタートの必要性について理解しており、一方それらの必要性を知らされてないシステム管理社はシステムの観点からリスタートについて理解しています。これがやり方の違いを生む、顧客に対して影響を及ぼす主原因でした。

あるシステム管理者はアプリケーションをリスタートさせるよう深夜に電話がかかってくることにうんざりしていました。また開発と運用の知識の溝にイライラしていました。彼はもっと良いアプローチでやっていくために先導を取っていくことを決めました。

## Rundeck のセットアップ

管理者は本番環境のサーバーにアクセスするためのマシンを用意してそこに Rundeck をインストールすることにしました。

アプリケーションサポートに関することを管理するためにプロジェクト名 "anvils" を作りました。

管理者はシェルツール [rd-project](../manpages/man1/rd-project.html) を使ってプロジェクトを作成します。これは Rundeck GUI ([プロジェクトのセットアップ](rundeck-basics.html#rundeck+のセットアップ)を参照)からでも可能です。Rundeck サーバーにログインし、以下のコマンドを実行します:

    rd-project -p anvils -a create

このコマンドにより "anvils" プロジェクトが Rundeck 上に作られ、そこには Rundeck サーバーに当たるノード情報のみが含まれています。本番環境に置かれたノード情報の追加についてはこれから説明します。([リソースモデル](getting-started.html#リソースモデル) を参照)

本番環境には anv1 から anv5 までの 5 つのノードが存在します。アプリケーションは 3 階層あります。ウェブ、アプリケーション、データベースのコンポーネントが 5 つのノードにまたがってインストールされています。

さらに管理者はアプリケーションのコンポーネントをコントロールする SSH コマンドを実行するのに、それぞれ違ったログインユーザーを使うようルールを敷くことに決めます。ウェブコンポーネントには "www" というユーザー、アプリケーションとデータベースコンポーネントには "anvils" というユーザーを用います。

このルールをきちんと管理するために、管理者は [resource-v13(5) XML](../manpages/man5/resource-v13.html) または [resource-v13(5) YAML](../manpages/man5/resource-yaml-v13.html) を使ったプロジェクトリソースモデルを準備します。下記の定義ファイルには、5 つのノードの定義がリストアップされています -- anv1, anv2, anv3, anv4, anv5

File listing: resources.xml

    <?xml version="1.0" encoding="UTF-8"?>

    <project>
      <node name="anv1" type="Node"
         description="an anvils web server" 
         hostname="anv1.acme.com"  username="www" tags="web"/>
      <node name="anv2" type="Node" 
         description="an anvils web server" 
         hostname="anv2.acme.com"  username="www" tags="web"/>
      <node name="anv3" type="Node" 
         description="an avnils app server" 
         hostname="anv3.acme.com"  username="anvils" tags="app"/>
      <node name="anv4" type="Node" 
         description="an anvils app server" 
         hostname="anv4.acme.com"  username="anvils" tags="app"/>
      <node name="anv5" type="Node" 
         description="the anvils database server" 
         hostname="anv5.acme.com"  username="anvils" tags="db"/> 
    </project>

XML コンテンツを見てみると、全体に渡って XML タグでホスト情報が表されています。各ノードの論理名は、``name`` 属性に定義します。(例 name="anv1") ``hostname`` 属性で定義されたアドレス(例 hostname="anv1.acme.com") は ``username`` 属性にて指定されたログインユーザー(例 username="www")が SSH コマンドを実行する際に利用されます。`tags` 属性で定義された値は、ノードの役割/機能を表しています。(例 tags="web" vs tags="app")

管理者は彼が決めたパスにファイルを保存します。Rundeck にそのファイルを認識させるために、プロジェクトの設定ファイルを編集する必要があります、`$RDECK_BASE/projects/anvils/etc/project.properties` の `project.resources.file` を編集します:

    project.resources.file = /etc/rundeck/projects/anvils/resources.xml
   
リソースファイルを置きプロジェクト設定を更新すれば、管理者としてのリソースモデルの準備とディスパッチコマンドを実行する準備が整ったことになります。

フィルタを開け、``.*`` を名前フィールドに入力し、"Filter" を押して、anvils プロジェクトのすべてのノードをリストアップします。6 つのノードが出ているはずです。

![Anvils resources](../figures/fig0601.png)

## タグ付けとコマンドディスパッチング

アプリケーションの役割を表すタグを使うと、ターゲットとなるホスト名を一切ハードコーディングせずにコマンド実行が可能になります。[dipatch](../manpages/man1/dispatch.html) コマンドにて、指定したタグによりフィルタリングされたノードセットをリストアップできます:

web ノードをリストアップするタグのキーワードを使います:

    dispatch -p anvils -I tags=web
    anv1 anv2
    
app ノードのリストアップ:

    dispatch -p anvils -I tags=app
    anv3 anv4

db ノードのリストアップ:

    dispatch -p anvils -I tags=db
    anv5

"+" (AND) 演算子を使って web と app ノードをリストアップします:

    dispatch -p anvils -I tags=web+app
    anv1 anv2 anv3 anv4

web と app ノードを除きます

    dispatch -p anvils -X tags=web+app
    anv5

ノード名にワイルドカードを使って、全てのノードをリストアップします:

    dispatch -p anvils -I '.*' 
    anv1 anv2 anv3 anv4 anv5 

以下の図はグラフィカルコンソールでのフィルタリングの使用例です。

![Anvils filtered list](../figures/fig0602.png)

タグによるフィルタリング機能によってホスト郡が抽象化され、管理者はゆるやかな分類を使ったスクリプトの処理について考えられます。新しいノードの追加も可能ですし、既に役割を与えられているホスト郡からその役割を外すことも可能です、そして、それぞれフィルタリング条件に紐づけられるため、各役割における処理に影響はありません。

この簡単な分類の仕組みは、開発者と管理者に Anvils アプリケーションのノードについて話す際の共通用語となります。

## ジョブ

ジョブは繰り返し実行する処理のライブラリをつくる便利な方法です。その性質上、保存されたジョブは処理をカプセル化する働きをします。ジョブは小さな、または大きなシェルスクリプトを呼ぶような 1 つのワークフローから、マルチステップのワークフローまで扱う事が出来ます。各ジョブを再利用可能な処理のブロックとして捉えると、より複雑な自動化を構築することができます。

起動とシャットダウンの処理の管理用として既に 2 つのスクリプトセットが存在していました。管理者は、どちらのスクリプトの方が優れているかといったことを強要することよりも、ジョブワークフローによってどのようにスクリプトがカプセル化されるかを簡単に示すために骨組みを作ることに専念しました。このシンプルなフレームワークについてのデモを行った後、管理者はジョブの定義に両方のスクリプト合わせた一番いいものを入れるにはどうしたらいいか議論することができます。

管理者は、骨組みとして実行しようとしていることと、与えられた引数をただ echo するだけのシンプルなプレースホルダースクリプトを作りました - start.sh と stop.sh - リスタートの処理を 2 つのステップで表現しています。

スクリプト:

File listing: start.sh

    #!/bin/sh
    # Usage: $0 
    echo Web started.

File listing: stop.sh

    #!/bin/sh
    # Usage: $0 [normal|kill]
    echo Web stopped with method: $1.

"method" オプションにて normal または kill のどちらの方法か指定できるようになっているため、ユーザーはスクリプトに引数を与える必要があります。

リスタート処理はジョブワークフローとして定義されるため、リスタートの処理にそのまま対応するスクリプトは存在しません。

### ジョブの構造

リスタートスクリプトのアイデアを念頭に置き、次のステップでは、リスタートの処理を含んだジョブの定義をします。最終的なゴールは、リスタートの処理をひとつ提供し、再利用できるように各ステップ毎にジョブを分けた方がいいでしょう。

管理者は以下のようなアプローチをイメージしています:

*   スタート: ウェブサービスをスタートさせるために start.sh スクリプトを実行する
*   ストップ: ウェブサービスをストップさせるために stop.sh スクリプトを実行する
*   リスタート: ストップジョブ、スタートジョブの順番で実行する

リスタートの処理が優先的であるため、大文字にして差別化しています。

後々の管理を考えて将来出てくるジョブと既存のジョブが結合できるようジョブを全ての個々のステップ毎に定義しようとすると大変複雑になってきますが、これは後からでもできることです。どのように処理を個々のジョブに分解するかは、保守性と再利用性のバランスです。

### Job grouping

Though not a requirement, it is helpful to use job groups and 
have a convention for naming them. 
A good convention assists others with a navigation scheme that
helps them remember and find the desired procedure.

The administrator chooses to create a top level group named
"/anvils/web/" where the web restart related jobs will be organized.

    anvils/
    `-- web/
        |-- Restart
        |-- start
        `-- stop

After choosing the "anvils" project users will see this
grouping of jobs.

![Anvils job group](../figures/fig0604.png)

## Job option

To support specifying the restart method to the scripts,
the the three jobs will declare an option named "method".
Without such a parameter, the administrator would be forced to
duplicate restart Jobs for both the kill and normal stop methods.

Another benefit from defining the job option is the ability to display a
menu of choices to the user running the job.  Once chosen, the value
selected by the menu is then passed to the script.

### Allowed values

An option can be defined to only allow values from a specified
list. This places safe guards on how a Job can be run by limiting
choices to those the scripts can safely handle.

The administrator takes advantage of this by limiting the "method" option
values to just  "normal" or "kill" choices.

The screenshot below contains the Option edit form for the "method" option.
The form includes elements to define description and default
value, as well as, Allowed Values and Restrictions.

![Option editor for method](../figures/fig0605.png)

Allowed values can be specified as a comma separated list as seen above but
can also be requested from an external source using a "remote URL".

Option choices can be controlled using the "Enforced from values"
restriction. When set "true", the Rundeck UI will only present a
popup menu. If set "false", a text field will also be presented. Use
the "Match Regular Expression" form to validate the input option.

Here's a screenshot of how Rundeck will display the menu choices:

![Option menu for method](../figures/fig0606.png)


### Script access to option data

Option values can be passed to scripts as an argument or referenced
inside the script using a named token. For example, the value for the
"method" option selection can be accessed in one of several ways:

Value referenced as an environment variable:

* Bash: $CT\_OPTION\_METHOD

Value passed in the argument vector to the executed script or command
via the ``scriptargs`` tag:

* Commandline Arguments: ${option.method}

Value represented as a named token inside the script and replaced
before execution:  

* Script Content: @option.method@

      
## Job workflow composition

With an understanding of the scripts and the option needed to
control the restart operation, the final step is to compose the Job
definitions. 

While each job can be defined graphically in Rundeck, each can
succinctly be defined using an XML file conforming to the
[job-v20(5)](../manpages/man5/job-v20.html) document format. This 
document contains a set of tags corresponding to the choices seen in
the Rundeck GUI form.

Below are the XML definitions for the jobs. One or more jobs can be
defined inside a single XML file but your convention will dictate how to
organize the definitions. The files can be named any way desired and
do not have to correspond to the Job name or its group.

File listing: stop.xml

    <joblist>	
        <job> 
           <name>stop</name>  
           <description>the web stop procedure</description>  
           <loglevel>INFO</loglevel>  
           <group>anvils/web</group>  
           <context> 
               <project>anvils</project>  
                 <options> 
                   <option name="method" enforcedvalues="true"
                           required="true" 
                       values="normal,kill"/> 
                   </options> 
           </context>  
           <sequence threadcount="1" keepgoing="false" strategy="node-first"> 
             <command> 
               <script><![CDATA[#!/bin/sh
    echo Web stopped with method: $1.]]></script>  
                <scriptargs>${option.method}</scriptargs> 
             </command> 
           </sequence>  
           <nodefilters excludeprecedence="true"> 
             <include> 
              <tags>web</tags> 
              </include> 
           </nodefilters>  
           <dispatch> 
             <threadcount>1</threadcount>  
             <keepgoing>false</keepgoing> 
           </dispatch> 
         </job>
    </joblist>


Defines Job, /anvils/web/stop, and executes the shell script to
Nodes tagged "web". Using the ``scriptargs`` tag, the shell
script is passed a single argument, ``${option.method}``,
containing the value chosen in the Job run form.

File listing: start.xml

    <joblist>	
       <job> 
         <name>start</name>  
         <description>the web start procedure</description>  
         <loglevel>INFO</loglevel>  
         <group>anvils/web</group>  
        <context> 
          <project>anvils</project>  
        </context>  
        <sequence threadcount="1" keepgoing="false" strategy="node-first"> 
         <command> 
          <script><![CDATA[#!/bin/sh
     echo Web started.]]></script>
         </command> 
      </sequence>  
        <nodefilters excludeprecedence="true"> 
          <include> 
            <tags>web</tags> 
          </include> 
       </nodefilters>  
       <dispatch> 
         <threadcount>1</threadcount>  
         <keepgoing>false</keepgoing> 
       </dispatch> 
      </job>
    </joblist>

Defines Job, /anvils/web/start, that also executes a shell script to
Nodes tagged "web".


File listing: restart.xml

    <joblist>	
       <job> 
         <name>Restart</name>  
         <description>restart the web server</description>  
         <loglevel>INFO</loglevel>  
         <group>anvils/web</group>  
         <context> 
           <project>anvils</project>  
             <options> 
               <option name="method" enforcedvalues="true" required="false" 
	          values="normal,kill" /> 
            </options> 
         </context>  
         <sequence threadcount="1" keepgoing="false" strategy="node-first"> 
          <command> 
            <jobref name="stop" group="anvils/web">
              <arg line="-method ${option.method}"/> 
            </jobref> 
          </command>  
          <command> 
            <jobref name="start" group="anvils/web">
            </jobref> 
          </command> 
        </sequence>
       </job>	
    </joblist>


Defines Job, /anvils/web/Restart, that executes a sequence of Job calls,
using the ``jobref`` tag.

Note that we don't define a `<nodefilters>` or `<dispatch>` section, because we
only want this sequence to execute **once**, on the server node.  The Job
references will each be called once, and the "start" and "stop" Jobs will
each be dispatched to the nodes they define.

Saving the XML definitions files located on the Rundeck server,
one can load them using the [rd-jobs](../manpages/man1/rd-jobs.html) command.

Run the ``rd-jobs load`` command for each job definition file:

    rd-jobs load -f start.xml
    rd-jobs load -f stop.xml
    rd-jobs load -f restart.xml

The ``rd-jobs list`` command queries Rundeck and prints out the list of
defined jobs:

    rd-jobs list -p anvils
    Found 3 jobs:
	- Restart - 'the web restart procedure'
	- start - 'the web start procedure'
	- stop - 'the web stop procedure'

Of course, the jobs can be viewed inside the Rundeck graphical console by going to
the Jobs page. Hovering over the "Restart" job name reveals job detail.

![Anvils restart jobs](../figures/fig0607.png)

You will see the composition of the "Restart" job as a workflow
calling the jobs: stop and start. The "Restart" job passes the
``-method`` option value to the lower level stop Job.

## Running the job

The Jobs can be run from the Rundeck graphical console by going to the
"Jobs" page. From there, navigate to the "Anvils/web" job group to
display the three stored Jobs.

Clicking the "Run" button for the Restart job, will display the
options selection page. The menu for the "method" option displays the
two choices: "normal" and "kill". No other choices can be made, nor a
textfield for free form entry, because the "method" option was defined
with the restriction "enforced from allowed values".

![Restart run page](../figures/fig0608.png)

The jobs can also be started from the command line using the
[run](../manpages/man1/run.html) shell tool. The job group and name are specified
using the "-j" parameter. Any options the Job supports are supplied
after the "--" (double dash) parameter. (The "-p" parameter specifies the project,
but it can be left out if there is only one project available.)

Run Restart specifying the method, "normal": 

    run -j "anvils/web/Restart" -p anvils -- -method normal

Run Restart specifying the method, "kill":

    run -j "anvils/web/Restart" -p anvils -- -method kill


## Job access control

Access to running or modifying Jobs is managed in an access control
policy defined using the aclpolicy document format ([aclpolicy-v10(5)](../manpages/man5/aclpolicy-v10.html)). 
This file contains a number of policy elements that describe what user
group is allowed to perform which actions. The
[Authorization](../administration/authorization.html) Chapter of the Administration Guide
covers this in detail.

The administrator wants to use the aclpolicy to define two levels of
access. The first level, has limited privilege and allows for just
running jobs. The second level, is administrative and can modify job
definitions.

Policies can be organized into more than one file to help organize
access by group or pattern of use. The normal Rundeck install will
define two user groups: "admin" and "user" and have a generated a policy
for the "admin" group. 

The Acme administrator decides to create a policy that allows users in
the "user" group to run commands just in the "anvils" and
"anvils/web" Job groups. We can employ the "user" login and group as
it was also included in the normal install.

To create the aclpolicy file for the "user" group:

    cp $RDECK_BASE/etc/admin.aclpolicy $RDECK_BASE/etc/user.aclpolicy

Modify the `<command>` and `<group>` elements as shown in the example
below. Notice that just workflow\_read,workflow\_run actions are
allowed.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~{.xml}
$ cat $RDECK_BASE/etc/user.aclpolicy
<policies>
  <policy description="User group access policy.">
    <context project="*">
      <command group="anvils" job="*" actions="workflow_read,workflow_run"/>
      <command group="anvils/web" job="*" actions="workflow_read,workflow_run"/>
    </context>
    <by>
      <group name="user"/>
    </by>
  </policy>
</policies>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Restart Rundeck to load the new policy file (see [Administration - Startup and Shutdown](../administration/startup-and-shutdown.html)).

    rundeckd restart

Once the Rundeck webapp has started, login as the "user" user (the
password is probably "user"). Just the Jobs in the "anvils" group are
displayed in the Jobs page. The "user" user does is not allowed to access
jobs outside of "/anvils group.

Notice the absence of the "New Job" button that would be displayed if
logged in as "admin". Job creation is an action not granted to
"user". Notice also, that the button bar for the listed Jobs does
not include icons for editing or deleting the Job. Only workflow\_read
and workflow\_actions were allowed in the `user.aclpolicy` file.


## Resource model source examples

See [Administration - Node Resource Sources - Resource Model Source](../administration/node-resource-sources.html#resource-model-source).


## Resource editor examples

See [Administration - Node Resource Sources - Resource Editor](../administration/node-resource-sources.html#resource-editor).

## Option model provider examples

See [Job Options - Option Model Provider](job-options.html#option-model-provider).
