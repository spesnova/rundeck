% プラグイン
% Alex Honor; Greg Schueler
% November 20, 2010

Rundeck のプラグインは、Rundeck core で利用されるいくつかのサービスのための新しいプロバイダです。
Plugins for Rundeck contain new Providers for some of the Services used by
the Rundeck core.

Rundeck にはビルトインのプロバイダがありますが、自分で開発したり、サードパーティ製のプラグインも使ってみましょう。

プリインストールされている script-plugin と stub-plugin については [プリインストールプラグイン](plugins.html#プリインストールプラグイン) を参照してください。

## プラグインのインストール

プラグインのインストールはシンプルです:

`plugin.jar` や `some-plugin.zip` のようなプラグインファイルを Rundeck サーバの libext ディレクトリに配置して下さい:

    cp some-plugin.zip $RDECK_BASE/libext

これでプラグインが有効化されました。内部で定義されている provider をノードやプロジェクトで利用できるようになっています。

Rundeck サーバをリスタートする必要はありません。

## プラグインのアンインストールとアップデート

`$RDECK_BASE/libext` から目的のファイルを削除すればアンインストールできます。

アップデートの時には古いファイルを新しいファイルで上書きしてください。

## プラグインについて

プラグインは 1 つ以上のサービスプロバイダを実装したファイル群です。プラグインファイルそれぞれに、異なるタイプのサービス用プロバイダを複数含めることができますが、何らかの形で関連しているプロバイダのみをまとめるのが一般的です。

Rundeck にはいくつかの "built-in" プロバイダと、"included" プラグインファイルが含まれています。

このドキュメントでは、"プラグイン" と "プロバイダ" をほぼ同義に扱います。実際のプロバイダ実装ファイルを参照するときには "プラグインファイル" と言います。

![Rundeck Providers and Plugin Files](../figures/fig1102.png)

## プラグインの種類

Rundeck は様々なサービスで動作させるためにいくつかの異なるプラグインタイプをサポートしています。

### Node-Executor プラグイン

ノード上でのコマンド実行や、ノードへのファイルコピーの方法について定義するプラグインです。

より詳細な情報については:

*   設定: [Node Executor サービス](plugins.html#node+executor+サービス)
*   ライフサイクル: [Node Executor サービスプロバイダが実行されたとき](plugins.html#node+executor+サービスプロバイダが実行されたとき)
*   組み込みプラグイン: [Node Executor サービス](plugins.html#node+executor+サービス)
*   付属プラグイン: [プリインストールプラグイン](plugins.html#プリインストールプラグイン)

### リソースモデルソース プラグイン

リソースモデル情報を指定した種類のソース（たとえば URL、ファイル、ディレクトリ内のファイル群）から取得するメカニズムを定義するプラグインです。

より詳細な情報については:

*   設定: [リソースモデルソース](plugins.html#リソースモデルソース)
*   ビルトインプロバイダ: [リソースモデルサービス](plugins.html#リソースモデルサービス)

### リソースフォーマット プラグイン

このタイプのプラグインには異なるドキュメントフォーマットのリソースモデル情報に対するパーサとジェネレータを定義します。さらにこれらは Rundeck システムにおける他の部分と同じくリソースモデルソースプラグインから利用されます。

より詳細な情報については:

*   設定: [リソースフォーマットジェネレータとパーサ](plugins.html#リソースフォーマットジェネレータとパーサ)
*   組み込みプラグイン: [リソースフォーマットサービス](plugins.html#リソースフォーマットサービス)

## 各サービスと各プロバイダについて

Rundeck のコアは、複数ノードに対して様々なワークフロー・ジョブ・コマンドを実行する機能を持ったいくつかの異なる "サービス" を利用します。


各サービスは各 "プロバイダ" を利用しています。各プロバイダはそれぞれを識別するためのユニークな "プロバイダ名" を持っており、ほとんどのサービスはどれを使うか指定しなくてもいいようフォルトのプロバイダを持っています。

![Rundeck Services and Providers](../figures/fig1101.png)

サービスはどのように・どこで使われるかによって異なるサービスに分類されます。

*サービスカテゴリ*:

1.  **Node Executor サービス** - これらのサービスの各プロバイダはある一つのノード定義の条件下で処理を行います。そしてノードの範囲も設定可能です:

    1.  Node Executor 実行 - これらのプロバイダでは、（ローカルまたはリモートの）あるノードに対して実行するコマンドを定義します。
    2.  ファイルコピー - これらのプロバイダではあるノードに対してファイルをどのようにコピーするか定義します。

2.  **プロジェクトサービス** 

    1.  リソースモデルソース - (別名 "リソースプロバイダ" ）これらはあるプロジェクト用にノードリソースの取得方法を定義します。

3.  **グローバルサービス**（フレームワークレベル）

    1.  リソースフォーマットパーサ - これらにはドキュメントフォーマットパーサを定義します。
    2.  リソースフォーマットジェネレータ - これらにはドキュメントフォーマットジェネレータを定義します。

これらのプラグインのプロバイダがどのように動作するかの詳細については以下で説明しています。

なお、Rundeck のプラグインは 1 つ以上のプロバイダを持つことができます。

## プロバイダを使う

### Node Executor サービス

Node Executor と File Copier という 2 つの*ノードサービス*は共に似たような設定がされています。これは特定のノードに対して行われるよう設定されているか、またはあるプロジェクトまたはシステム用のデフォルトプロバイダとして設定されています。

複数プロバイダが定義される場合には以下のようにほとんどの定義で各プロバイダに優先順位を付けます:

1.  ノードの指定
2.  プロジェクトスコープ
3.  フレームワークスコープ

#### ノードの指定

あるノードに対してプロバイダを有効にするには、そのノードの定義にある属性を追加します。

To enable a provider for a node, add an attribute to the node definition.

*Node Executor プロバイダ属性*:

`node-executor`

:    ローカルではないノードに対してプロバイダ名を指定します。

`local-node-executor`

:    ローカル（Rundeck サーバー）ノードに対してプロバイダ名を指定します。

ファイルコピープロバイダ属性:

`file-copier`

:    ローカルではないノードに対してプロバイダ名を指定します。

`local-file-copier`

:    ローカル（Rundeck サーバー）ノードに対してプロバイダ名を指定します。

サンプルのノードに対して、`stub` という Node Executor プロバイダと File Copier プロバイダを YAML フォーマットで指定した例になります:

    remotehost:
        hostname: remotehost
        node-executor: stub
        file-copier: stub

#### プロジェクトまたはフレームワークのスコープ

*Node Executor*

プロジェクトまたはフレームワーク（または両方）のスコープにてノード郡を利用するためにデフォルトで接続されるプロバイダを定義できます。そのために、`project.properties` または `framework.properties` ファイルに、次のいずれかのプロパティを設定します。

`service.NodeExecutor.default.provider`

:   リモートノードに対してデフォルトの Node Executor プロバイダを指定します。

`service.NodeExecutor.default.local.provider`

:   ローカルノードに対してデフォルトの Node Executor プロバイダを指定します。

*File Copier*

`service.FileCopier.default.provider`

:   リモートノードに対してデフォルトの File Copier プロバイダを指定します。

`service.FileCopier.default.local.provider`

:   ローカルノードに対してデフォルトの File Copier プロバイダを指定します。

サンプルの `project.properties` に `stub` というデフォルトのローカルプロバイダを設定する例になります：

    service.NodeExecutor.default.local.provider=stub
    service.FileCopier.default.local.provider=stub

### リソースモデルソース

*リソースモデルソース*プロバイダはあるひとつのプロジェクト内の `project.properties` ファイルに対して設定することができます。

そのプロジェクトに対して複数のリソースモデルソースの定義もできます。また必要に応じて指定した指定するプロバイダを混ぜたり組み合わせたり出来ます。

あるプロジェクトで複数のソースプロバイダを定義しているときは、全てのソースのマージがノードセットの結果になり、それらは定義にされた順に並べられます。これは 2 つまたは複数のソースが同じ名前でノードを定義しているとき、一番下にある定義がリストに用いられるということを意味します。

プロバイダが読み込まれノードがマージされたときの順番は:

1.  `project.resources.file`: デフォルト設定のファイルモデルソース
2.  `project.resources.url` : デフォルト設定の URL モデルソース（オプション）
3.  All `resources.source.N`: 1 から始まる順番の設定

#### リソースモデルソースの設定

以下の方法で各プロジェクトの `project.properties` ファイルにてリソースモデルソースの設定ができます:

1.  `project.resources.file` を定義する - このファイルパスはあるファイルソースパスとして用いられ、*autogenration* と *includeServerNode* はともに true になっています。
2.  `project.resources.url` を定義する - この URL は URL ソース url として用いられ、キャッシュは ON になっています。

このようにより多くのソースのリストから定義することも可能です:

インデックスは `1` から始まり、ナンバー `N` のソースについて以下のようなプロパティを定義します:

    resources.source.N.type=<provider-name>
    resources.source.N.config.<property>=<value>
    resources.source.N.config.<property2>=<value2>
    ...

`<provider-name>` に有効なリソースモデルプロバイダ名を指定します。各リソースモデルプロバイダごとに、ソースについてのプロパティ設定が行えます。

デフォルトのファイルプロバイダとその他 2 つのプロバイダの `project.properties` 設定の例:

    project.resources.file=/home/rundeck/projects/example/etc/resources.xml
    
    resources.source.1.type=url
    resources.source.1.url=http://server/nodes.yaml
    
    resources.source.2.type=directory
    resources.source.2.directory=/home/rundeck/projects/example/resources

### リソースフォーマットジェネレータとパーサ

リソースフォーマットジェネレータとパーサはリソースノードの定義セットにパースされるまたは、そこからジェネレートされるファイルフォーマットについてのサポートを定義します。

これらはリソースモデルソースといった Rundeck の他のパーツから使われます。

これらのプロバイダを使うのに一切設定は必要ありません。しかし各ジェネレータとパーサといった特定のプロバイダに関してはプロバイダを利用可能な状態にするためにプロバイダ名を知らせる必要があります。パーサまたはジェネレータを使いたいときその特定のプロバイダ名は "フォーマット名"として使われます。

例えば、あるファイルリソースモデルソース（[ファイルリソースモデルソース設定](plugins.html#file-resource-model-source-configuration)を参照してください）に利用される特定のリソースフォーマットパーサを利用可能な状態にするために、そのプロバイダ名を指定します:

    resources.source.1.format=myformat

これで "myformat" プロバイダを使うという指定になります。

別のケースでは、実際のプロバイダ名は知られていないかもしれません（例えば、リモート URL からコンテンツを読み込む場合）。各ジェネレータとパーサはある MIME タイプとサポートされているファイルエクステンションのリストを定義していなければいけません。これらはどのパーサ/ジェネレータを使うか決める際に利用されます。

## Node Executor サービスプロバイダが呼ばれたとき

Rundeck がノード上でコマンドアイテムを実行します。そのコマンドはジョブ内のあるワークフローの一部かもしれません、また複数ノードに対して何度も実行されるコマンドかもしれません。

今のところワークフローにて指定できるコマンドアイテムは 3 "種類" あります:

1. "exec" コマンド - シンプルなシステムコマンド
2. "script" コマンド - 組み込みスクリプトファイルや Rundeck サーバーに置いてあるスクリプトファイルが指定されたノードに配布され入力された引数セットを用いて実行されます。
3. "jobref" コマンド - 入力された引数セットを用いて実行される他のジョブを名前から参照する

Rundeck はこれらのコマンドタイプを実行するプロセスの一部として Node Executor サービスと File Copier サービスを使います。

"exec" コマンドを実行する一連の処理は以下のようになります:

1.  指定されたノードとコンテキストに合わせて Node Executor プロバイダを読み込みます
2.  `NodeExecutor#executeCommand` メソッドを呼びます

"script" コマンドを実行する一連の処理は以下のようになります:

1.  指定されたノードとコンテキストに合わせて File Copier プロバイダを読み込みます
2.  `FileCopier#copy*` メソッドを呼び出します
3.  `NodeExecutor` メソッドを呼び出します
4.  （ファイルをコピーする際の "chmod +x" のような）途中に行うコマンドを実行する場合もあります
5.  コピーするファイルのファイルパスと、スクリプトコマンドにて実行するあらゆる引数を渡して `NodeExecutor#executeCommand` メソッドを実行します

## ビルトインプロバイダ

Rundeck がデフォルトサービスを提供するためのわずかなビルトインプロバイダを使います。

### Node Executor サービス

Node Executor プロバイダ:

`local`

:   ローカルでのコマンド実行。

`jsch-ssh`

:   SSH 経由のコマンドのリモート実行。ノードのアトリビュート "hostname" と "username" を必要とします。

File Copier プロバイダ:

`local`

:   あるスクリプト用に一時的なローカルファイルを作ります

`jsch-scp`

:   SCP 経由のリモートへのコマンドのコピー。ノードのアトリビュート "hostname" と "username" を必要とします。

#### SSH プロバイダ

SSH Node Executor と File Copier は Rundeck にデフォルトで付属しています。

それらを使うための典型的なノード設定はシンプルです。

* ノードの `hostname` アトリビュートをセットします。デフォルトポート 22 番以外を使っている場合は "hostname:port" というフォーマットも可能です。
* リモートノードへ接続する際に用いる username をノードの `username` アトリビュートにセットします。
* Rundeck サーバーからノード群に対して公開鍵/秘密鍵認証のセットアップを行います

これでノード上でリモートコマンドを実行したり、スクリプトを実行できるようになります。

設定のオプションについては以下を参照してください。

**Sudo パスワード認証**

SSH プロバイダは、補助的に Sudo パスワード認証もサポートしています。これは "sudo" コマンドがパスワード認証を必要としたときに、あるユーザーがパスワードをターミナルのパスワードプロンプトに入力するといった認証をシミュレートするものです。

##### SCP File Copier の設定

一般的な SSH 設定の記述に加えて、いくつか SCP 用に追加設定があります。
In addition to the general SSH configuration mentioned for in this section, some additional configuration can be done for SCP. 

あるリモートノード上であるスクリプトが実行されたとき、まず SCP 経由でスクリプトファイルをコピーし、それから実行します。SSH コネクションプロパティに加えて SCP 用に以下のようなノードアトリビュートを設定できます。

*   `file-copy-destination-dir`: 実行前にそのスクリプトファイルをコピーする先となるリモートノード上のディレクトリ。デフォルトの値は Windowns ノードだと `C:/WINDOWS/TEMP` 、それ以外は `/tmp` となります。
*   `osFamily`: windows ノード用には "windows" と指定します。

##### SSH 認証タイプの設定

SSH 認証は パスワードまたは公開鍵/秘密鍵認証の 2 つの方法で行われます。

デフォルトでは、公開鍵/秘密鍵認証方式が使われますが、フレームワークスコープ、プロジェクト、ノードによって変更できます。

どちらのメカニズムが使われるかは `ssh-authentication` プロパティによって決められます。このプロパティはどちらか 2 つの値を持ちます:

*   `password`
*   `privateKey` （デフォルト）

ある特定のノードに接続した時、正しい認証メカニズムが選ばれるようこのシーケンスが使われます。

1. **ノードレベル**: ノード上の `ssh-authentication` アトリビュートにターゲットノードにだけを適用します。
2. **プロジェクトレベル**: `project.properties` 内の `project.ssh-authentication` プロパティ。 `project.properties`  デフォルトでプロジェクト配下の全ノードに適用します。
3. **Rundeck レベル**: `framework.properties` 内の `framework.ssh-authentication` プロパティ。デフォルトで全てのプロジェクトのノードに適用します。

これらの値が一切セットされてない場合はデフォルトで公開鍵/秘密鍵認証方式が使われます。

##### SSH username 
の設定

SSH 経由で接続するための username は、ノードアトリビュート `username` から取得します:

*   `username="user1"`

動的にこの値を変化させたい場合はプロパティ参照を組み込むこともできます。例えば、現在の Rundeck 利用者またはジョブのオプション値として username を取得して参照するといったことです:

*   `${job.username}` - Rundeck でジョブを実行している username を使います
*   `${option.someUsername}` - "someUsername" というジョブオプションの値を使います

もしノードアトリビュート `username` がセットされていない場合はプロジェクトまたはフレームワーク設定で使われてる静的な値を使います。あるノードへの接続時の username は下記の順番で値を探しにいきます:

1.  **Node level**: node attribute `username` 。実行時の値やオプションを参照出来る場合は利用します
2.  **Project level**: プロジェクト内の `project.properties` ファイル内の `project.ssh.user` プロパティ
3.  **Rundeck level**: `framework.properties` ファイル内の `framework.ssh.user` プロパティ

##### SSH 秘密鍵の設定

デフォルトの認証メカニズムは公開鍵/秘密鍵認証です。

ビルトイン SSH コネクターにより秘密鍵を数種類の方法でできます。ノード毎、プロジェクト毎、Rundeck インスタンスごとにその方法を変えられます。

リモートノードに接続する時に、Rundeck は秘密鍵ファイルの場所を示したあるプロパティ/attribute を探します。その際、以下の順番で最初にヒットするまで探します:

1. **Node レベル**: ノード上の `ssh-keypath` attribute 。ターゲットノードにだけ適用します。
2. **Project レベル**: `project.properties` 内の `project.ssh-keypath` プロパティ。 デフォルトでプロジェクト配下の全ノードに適用します。
3. **Rundeck レベル**: `framework.properties` 内の `framework.ssh-keypath` プロパティ. デフォルトで全プロジェクトに適用します。
4. **Rundeck レベル**:  `framework.properties` 内の `framework.ssh.keypath` プロパティ。デフォルトで全プロジェクトに適用します。（1.3 より前の Rundeck にも互換性があります）（デフォルトの値は、`~/.ssh/id_rsa`）

暗号化したパスフレーズを使う秘密鍵の場合は、ノードで実行する際にパスフレーズを入力させるためにプロンプトをすする "Secure Option" が利用できます。下記の項目を参照してください。

##### SSH 秘密鍵パスフレーズの設定

パスフレーズを使った秘密鍵認証は以下のように動作します:

*   鍵のパスフレーズを入力させるプロンプトを出すようジョブにセキュアオプションについての定義がされていなければなりません。
*   ターゲットノードは秘密鍵認証を使うよう設定されてなければなりません。
*   ユーザがジョブを実行するとき、パスフレーズ入力用のプロンプトが表示されます。セキュアオプションの値はデータベースには保存されず、その実行にだけ利用されます。

さらに秘密鍵パスフレーズ認証にはいくつかの必要事項と制限があります:

1.  秘密鍵認証にてパスフレーズを要求するノードには予め定義されたジョブを通してのみ実行が可能です。アドホックコマンドでは実行できません。（今のところ）
2.  そのようなノード上で実行される各ジョブには実行前にパスフレーズを入力してもらうようセキュアオプションの定義がされていなければなりません。
3.  秘密鍵パスフレーズ認証を使う全てのノードでは入力するパスフレーズと定義されたセキュアオプション定義とがマッチする必要があります。または同じパスフレーズを共有して使う場合は同じ Option 名（またはデフォルトのまま）を使う必要があります。（例えば、同じ秘密鍵を共有して使う場合など）

パスフレーズは GUI ・CUI どちらからも入力できます。CUI または API 経由でジョブを実行したい場合はジョブの引数としてパスフレーズを入力します。

SSH 秘密鍵認証を利用可能にするために、まず `ssh-authentication` の値が [SSH 認証タイプの設定](plugins.html#ssh-認証タイプの設定)にて説明されているとおりに設定されてるか確かめてください。次に、[SSH 秘密鍵の設定](plugins.html#ssh-秘密鍵の設定)にて説明されているとおりに秘密鍵ファイルのパスが設定されているか確かめてください。

今度はジョブの設定を行います。オプションの定義 `secureInput` に `true` をセットします。このオプション名（ここでいう `secureInput`）は自由に決めて構いませんが、ノードの設定でデフォルト値として使われている `sshKeyPassphrase` を使うのが一番簡単です。

もしオプション名が `sshKeyPassphrase` で無い場合は、以下の attribute がセットされているか確かめてください:

*   `ssh-key-passphrase-option` = "`option.NAME`" `NAME` 部分にジョブのセキュアオプションの名前が入ります。

ノードとジョブオプションの設定の例です:

    <node name="egon" description="egon" osFamily="unix"
        username="rundeck"
        hostname="egon"
        ssh-keypath="/path/to/privatekey_rsa"
        ssh-authentication="privateKey"
        ssh-password-option="option.sshKeyPassphrase" />

ジョブ:

    <joblist>
        <job>
            ...
            <context>
              <project>project</project>
              <options>
                <option required='true' name='sshKeyPassphrase' secure='true'
                  description="Passphrase for SSH Private Key"/>
              </options>
            </context>
            ...
        </job>
    </joblist>

##### SSH パスワード認証の設定

パスワード認証は以下のように動作します:

*   ジョブにはユーザにパスワードを要求するセキュアオプションが定義されていなければいけません。
*   ターゲットノードはパスワード認証するよう設定されていなければいけません。
*   ユーザがジョブを実行する際、パスワード入力用にプロンプトを表示します。パスワード用セキュアオプションの値はデータベースには保存されず、その実行のみに使われます。

1.  パスワード認証を行うノードには予め定義されたジョブを通してのみ実行が可能です。アドホックコマンドでは実行できます。（今のところ）
2.  そのようなノード上で実行される各ジョブには実行前にパスワードを入力してもらうようセキュアオプションの定義がされていなければなりません。
3.  パスワード認証を使う全てのノードでは入力するパスワードと定義されたセキュアオプション定義とがマッチする必要があります。または同じパスワードを共有して使う場合は同じ Option 名（またはデフォルトのまま）を使う必要があります。（例えば、同じ秘密鍵を共有して使う場合など）

パスワードは GUI・CUI どちらからも入力できます。CUI または API 経由でジョブを実行したい場合はジョブの引数としてパスワードを入力します。

パスワード認証を利用可能にするために、まず `ssh-authentication` の値を [SSH 認証タイプの設定](plugins.html#ssh-認証タイプの設定)にて説明されているとおりに設定されているか確かめてください。

次にジョブの設定をします。オプションの定義 `secureInput` に `true` をセットします。このオプション名（ここでいう `secureInput`）は自由に決めて構いませんが、ノードの設定でデフォルト値として使われている

SSH 秘密鍵認証を利用可能にするために、まず `ssh-authentication` の値が [SSH 認証タイプの設定](plugins.html#ssh-認証タイプの設定)にて説明されているとおりに設定されてるか確かめてください。次に、[SSH 秘密鍵の設定](plugins.html#ssh-秘密鍵の設定)にて説明されているとおりに秘密鍵ファイルのパスが設定されているか確かめてください。

今度はジョブの設定を行います。オプションの定義 `secureInput` に `true` をセットします。このオプション名（ここでいう `secureInput`）は自由に決めて構いませんが、ノードの設定でデフォルト値として使われている `sshKeyPassphrase` を使うのが一番簡単です。

もしオプション名が `sshKeyPassword` で無い場合は、以下の attribute が各ノードにセットされているか確かめてください:

*   `ssh-password-option` = "`option.NAME`" : `NAME` 部分にジョブのセキュアオプションの名前が入ります。

ノードとジョブオプションの設定の例です:

    <node name="egon" description="egon" osFamily="unix"
        username="rundeck"
        hostname="egon"
        ssh-authentication="password"
        ssh-password-option="option.sshPassword1" />

ジョブ:

    <joblist>
        <job>
            ...
            <context>
              <project>project</project>
              <options>
                <option required='true' name='sshPassword1' secure='true' />
              </options>
            </context>
            ...
        </job>
    </joblist>

##### 2 つ目の Sudo パスワード認証の設定

SSH プロバイダは 2 つ目の Sudo パスワード認証を設定できるメカニズムをサポートしています。あなたの環境におけるセキュリティ要件で一般的な "rundeck" アカウントではなくそれぞれのユーザアカウントを使って SSH 接続させ、"sudo" レベルのコマンドを実行時にパスワードを要求するような場合に便利です。

このメカニズムは以下のように動作します:

*   ジョブの実行時に、ユーザに Sudo パスワードを入力するようプロンプトが出されます。
*   SSH 経由でリモートノードに接続後、例えば "sudo -u otheruser /sbin/some-command" のようなコマンドが "sudo" 認証を要求します。
*   リモートノードがユーザの Sudo パスワードの入力を促します。
*   SSH プロバイダがリモートノードにパスワードを入力します。
*   ユーザがエンターを押すと sudo コマンドが実行されます。

SSH パスワード認証に同じように、Sudo パスワード認証にも必要事項があります:

*   ジョブにはユーザにパスワードを入力させるようプロンプトを出すセキュアオプションの定義がされていなければなりません。
*   ターゲットノードは Sudo 認証を行うよう設定されていなければなりません。
*   ユーザがジョブを実行するとき、パスワード入力用のプロンプトが表示されます。セキュアオプションの値はデータベースには保存されず、その実行にだけ利用されます。

さらに Sudo パスワード認証にはいくつかの必要事項と制限があります:

1.  秘密鍵認証にてパスワードを要求するノードには予め定義されたジョブを通してのみ実行が可能です。アドホックコマンドでは実行できません。（今のところ）
2.  そのようなノード上で実行される各ジョブには実行前にパスワードを入力してもらうようセキュアオプションの定義がされていなければなりません。
3.  秘密鍵パスワード認証を使う全てのノードでは入力するパスワードと定義されたセキュアオプション定義とがマッチする必要があります。または同じパスワードを共有して使う場合は同じ Option 名（またはデフォルトのまま）を使う必要があります。（例えば、同じ秘密鍵を共有して使う場合など）

パスワードは GUI ・CUI どちらからも入力できます。CUI または API 経由でジョブを実行したい場合はジョブの引数としてパスワードを入力します。

SSH パスワード認証を利用可能にするために、まず `ssh-authentication` の値が [SSH 認証タイプの設定](plugins.html#ssh-認証タイプの設定)にて説明されているとおりに設定されてるか確かめてください。

今度はジョブの設定を行います。オプションの定義 `secureInput` に `true` をセットします。このオプション名（ここでいう `secureInput`）は自由に決めて構いませんが、ノードの設定でデフォルト値として使われている `sshKeyPassword` を使うのが一番簡単です。

ノードやプロジェクト、Rundeck システム単位にこれらのプロパティの設定を行う事で Sudo パスワード認証が動作するようになります。project.properties の中の `project.NAME` や framework.properties の中の `framework.NAME` というアトリビュート名をノードにセットします。
 
*   `sudo-command-enabled` - "true" をセットし、Sudo パスワード認証を利用可能にします。
*   `sudo-command-pattern` - Sudo 認証が必要なコマンドを見つけるための正規表現をセットします。デフォルトパターンは `^sudo$` です。
*   `sudo-password-option` - どのセキュアオプション値（パスワードとして利用される）かを定義するオプションリファレンス ("option.NAME") をセットします
*   `sudo-prompt-pattern` - Sudo 認証の際のプロンプトを見つけるための正規表現をセットします。デフォルトパターンは `^\[sudo\] password for .+: .*` です。
*   `sudo-failure-pattern` - 認証失敗時の出力を見つけるための正規表現をセットします。デフォルトパターンは `^.*try again.*` です。
*   `sudo-prompt-max-lines` - 期待されるパスワードプロンプトは最大何行まで読む込むかをセットします。（デフォルトは `12` です）
*   `sudo-prompt-max-timeout` - 期待されるパスワードプロンプトが表示されるまで何ミリ秒待つかをセットします。（デフォルト `5000` です）
*   `sudo-response-max-lines` - 認証失敗時の出力を探す際何行読み込むかをセットします。（デフォルト `2` です）
* `sudo-response-max-timeout` - maximum milliseconds to wait for response when detecting the failure response. (default `5000`)
*   `sudo-response-max-timeout` - 認証失敗時の出力が表示されるまで何ミリ秒待つかをセットします。（デフォルト `false` です）
* `sudo-success-on-prompt-threshold` - true/false. If true, succeed (without writing password), if the input max lines are reached without detecting password prompt. (default: `true`).
*   `sudo-success-on-prompt-threshold` - true/false. true にすると、もしパスワードプロンプトを見つけられずに最大読み込み行数に達した場合に（パスワードを入力せずに）認証成功です。（デフォルトは `true` です）????
*   `sudo-fail-on-prompt-timeout` - true/false. true にすると、パスワードプロンプトが表示される前にタイムアウトした場合はジョブの実行を失敗とします。（デフォルトは `true` です）
* `sudo-fail-on-response-timeout` - true/false. If true, fail on timeout looking for failure message. (default: `false`)
*   `sudo-fail-on-response-timeout` - true/false. true にすると、認証失敗の出力を探し見つからないままタイムアウトした場合ジョブの実行は失敗となります。（デフォルトは `false` です）???

ノート: unix で "sudo" が必要なコマンドのデフォルト値は既に設定されていますが、もしインタラクションをカスタマイズする必要がある場合は上書きできます。

今度はジョブの設定を行います。オプションの定義 `secureInput` に `true` をセットします。このオプション名（ここでいう `secureInput`）は自由に決めて構いませんが、ノードの設定でデフォルト値として使われている `sudoPassword` を使うのが一番簡単です。

もしオプション名が `sudoPassword` で無い場合は、以下の attribute がセットされているか確かめてください:

*   `sudo-password-option` = "`option.NAME`" : `NAME` 部分にジョブのセキュアオプションの名前が入ります。

ノードとジョブオプションの設定の例です:

    <node name="egon" description="egon" osFamily="unix"
        username="rundeck"
        hostname="egon"
        sudo-command-enabled="true"
        sudo-password-option="option.sudoPassword2" />

ジョブ:

    <joblist>
        <job>
             <sequence keepgoing='false' strategy='node-first'>
              <command>
                <exec>sudo apachectl restart</exec>
              </command>
            </sequence>

            <context>
              <project>project</project>
              <options>
                <option required='true' name='sudoPassword2' secure='true' description="Sudo authentication password"/>
              </options>
            </context>
            ...
        </job>
    </joblist>

##### Sudo パスワード認証を複数設定する

ノードに対してさらに高度な sudo パスワード認証設定ができます。例えば "sudo -u user1 sudo -u user2 command" といったような "sudo" コマンドを連鎖的に実行する必要性があり、両方の "sudo" に対してパスワードを入力する必要性がある場合です。あなたの node/project/framework に合わせた 2 つ目のプロパティセットを設定できます。

プロパティの設定方法は、1 つ目の sudo パスワード認証と同じです。 [2 つ目 sudo パスワード認証の設定](#configuring-secondary-sudo-password-authentication)にて説明されています。しかし以下のように、"sudo-" の代わりに "sudo2-" というプリフィックスを使います:

    sudo2-command-enabled="true"
    sudo2-command-pattern="^sudo .+? sudo .*$"

こうすることにより実行しようとしているコマンドが設定されたパターンにマッチするときは 2 つ目の sudo パスワードプロンプトを表示するよう仕向けます。

"sudo2-password-option" の値が設定されていないときは、デフォルトの値 `option.sudo2Password` が利用されます。

**"sudo2-command-pattern" について一つ注意点があります:**

sudo 認証メカニズムはどちらの設定が呼ばれたのかを確認するために 2 つの正規表現を使います。

1 つ目の sudo 認証かを判別するために、**実行されているコマンドの最初の部分**と "sudo-command-pattern" の値とマッチングします。このパターンのデフォルト値は `^sudo$` です。

そのため "sudo -u user1 some command" のようなコマンドは正しくマッチします。（例えば、"su" をサポートするよう）正規表現を編集できます。しかしマッチング対象とされるのは常にコマンドの最初の部分であることに注意してください。

"sudo2-command-enabled" が "true" に設定されているとき "sudo2-command-pattern" の値もマッチングします。正しくマッチした場合はもう 1 つの sudo 認証が利用されます。しかし、どちらの sudo 認証を使うべきか決められるよう、**コマンド文字列全体** を正規表現とのマッチング対象とします。デフォルトの正規表現は `^sudo .+? sudo .*$` です。必要に応じて正規表現はカスタマイズできます。

### リソースモデルソース

Rundeck は以下に示すビルトインプロバイダをコアに持っています:

`file`

:    サポートされているリソースフォーマットのファイルを使うプロバイダ。

`url`

:    サポートされているリソースフォーマットのリソースを URL から GET して取得するプロバイダ。

`directory`

:    あるディレクトリ内のサポートされているファイルを全て読み込むプロバイダ。ファイルプロバイダのエクステンションであり、内部では `file` プロバイダを使っています。

`script`

:    あるスクリプトによりパースされた出力をサポートされているフォーマットとして使うプロバイダ。

これらのプロバイダの設定を行うには, [リソースモデルソースの設定](plugins.html#リソースモデルの設定)を参照してください。そして以下の項目で出てくるプロパティを使ってください。

#### ファイルリソースモデルソースの設定

`file` リソースモデルソースプロバイダはサポートされている[リソースモデルドキュメントフォーマット](rundeck-basics.html#resource-model-document-formats)のファイルを読み込みます。

名前                          値                              説明 
-----                         ------                          ------
`file`                        ファイルパス                    ディスク上のファイルパス
`format`                      フォーマット名                  明示的にフォーマットを宣言することができます。利用できるフォーマットは `file` エクステンションのものに限ります。
`requireFileExists`           true/false                      値が true かつファイルが存在しないとき、ノードの読み込みは失敗となります。（デフォルト: false ）
`includeServerNode`           true/false                      値が true ならば Rundeck サーバーのノードを自動的に含めます（デフォルト: false ）
`generateFileAutomatically`   true/false                      If true, create the file automatically if it is missing. (Default: false)
----------------------------

Table: `file` リソースモデルソースプロバイダのプロパティの設定

`format` の値は[リソースモデルドキュメントフォーマット](rundeck-basics.html#リソースモデルドキュメントフォーマット)にてサポートされているものでなければなりません。ビルトインフォーマットは `resourcexml` または `resourceyaml` の 2 つです。
The built-in formats are: `resourcexml` or `resourceyaml`, but any format provided by a [Resource Format Plugin](plugins.html#resource-format-plugins) can be specified as well.

*Example:*

    resources.source.1.type=file
    resources.source.1.file=/home/rundeck/projects/example/etc/resources2.xml
    resources.source.1.format=resourcexml
    resources.source.1.requireFileExists=true
    resources.source.1.includeServerNode=true
    resources.source.1.generateFileAutomatically=true

#### URL Resource Model Source Configuration

The `url` Resource Model Source provider performs a HTTP GET request to retrieve the Nodes definition.

Configuration properties:

Name      Value       Notes
-----     ------      ------
`url`     URL         A valid URL, either `http:`, `https:` or `file:` protocol.
`cache`   true/false  If true, use ETag/Last-Modified information from the server to only download new content if it has changed. If false, always download the content. (Default: true)
`timeout` seconds     Number of seconds before request fails due to timeout. `0` means no timeout. (Default: 30) 
----------------------------

Table: Configuration properties for `url` Resource Model Source provider

The [Resource Model Document Format](rundeck-basics.html#resource-model-document-formats) that is used is determined by the MIME type
sent by the remote server. The built-in formats accept "\*/xml" and "\*/yaml" and "*/x-yaml". See [Resource Format Plugin](plugins.html#resource-format-plugins).

*Example:*

    resources.source.1.type=url
    resources.source.1.url=file:/home/rundeck/projects/example/etc/resources2.xml
    resources.source.1.cache=true
    resources.source.1.timeout=0

#### Directory Resource Model Source Configuration

The `directory` Resource Model Source provider lists all files in a directory, and loads each one that has a supported file extension
as File Resource Model Source with all default configuration options.

Name                          Value                           Notes
-----                         ------                          ------
`directory`                   directory path                  All files in the directory that have a supported file extension will be loaded
----------------------------

Table: Configuration properties for `directory` Resource Model Source provider

*Example:*

    resources.source.2.type=directory
    resources.source.2.directory=/home/rundeck/projects/example/resources
    
#### Script Resource Model Source Configuration

The `script` Resource Model Source provider executes a script file and reads
the output of the script as one of the supported [Resource Model Document Formats](rundeck-basics.html#resource-model-document-formats).

Name             Value                           Notes
-----            ------                          ------
`file`           Script file path                If required by the `interpreter`, the file should be executable
`interpreter`    Command or interpreter to use   e.g. "bash -c"
`args`           Additional arguments to pass    The arguments will be added after the script file name to the executed commandline
`format`         Format name                     Must be used to declare the format explicitly.
----------------------------

Table: Configuration properties for `script` Resource Model Source provider

The script will be executed in this way:

    [interpreter] file [args]

All output on STDOUT will be passed to a Resource Format Parser to parse.  The
format specified must be available.

*Example:*

    resources.source.2.type=script
    resources.source.2.file=/home/rundeck/projects/example/etc/generate.sh
    resources.source.2.interpreter=bash -c
    resources.source.2.args=-project example
    resources.source.2.format=resourceyaml

### Resource Format services

Resource Format services (Generators and Parsers) typically come in matched 
pairs, with both a parser and generator for the same format name.

Rundeck includes these built-in providers in the core installation:

`resourcexml`

:    Supports the Resource XML document format: [resource-v13(5) XML](../manpages/man5/resource-v13.html).

    Supported MIME types:

    * Generator: "text/xml"
    * Parser: "*/xml"

    Supported File extensions:

    * ".xml"

`resourceyaml`

:    Supports the Resource YAML document format: [resource-v13(5) YAML](../manpages/man5/resource-yaml-v13.html).

    Supported MIME types:

    * Generator: "text/yaml", "text/x-yaml", "application/yaml", "application/x-yaml"
    * Parser: "\*/yaml", "\*/x-yaml"

    Supported File extensions:

    * ".yml", ".yaml"

## Pre-installed plugins

Rundeck comes with two pre-installed plugins that may be useful, and also serve
as examples of plugin development and usage.

### script-plugin

The `script-plugin` includes these providers:

* `script-exec` for the NodeExecutor service
* `script-copy` for the FileCopier service

(Refer to [Using Providers](plugins.html#using-providers) to enable them.)

This plugin provides the ability to specify an external script or command
to perform a remote or local execution of a Rundeck command, and remote or local file copies.

It can be a replacement for the built-in SSH-based remote execution and SCP-based file-copy mechanism to
allow you to user whatever external mechanism you wish.

Note: this plugin offers similar functionality to the 
[Script Plugin Development](../developer/plugin-development.html#script-plugin-development) 
 model.  You may want to use this plugin to test your scripts, and
then later package them into a standalone plugin using that model.  

#### Configuring script-exec

To configure the plugin you must specify a commandline string to execute.  Optionally
you may specify a directory to be used as the working directory when executing
the commandline string, and a shell to invoke the command.

You can configure these across all projects (framework-wide), a single project 
(project-wide), or specifically for each node, with the most specific configuration
value taking precedence.

#### Configuring the command for script-exec

For Framework and Project-wide, configure a property in either the framework.properties or 
project.properties files:

`plugin.script-exec.default.command`

:   Specifies the default system command to run

For node-specific add an attribute named `script-exec` to the node.

`script-exec`

:   Specifies the system command to run

See [Defining the script-exec command](plugins.html#defining-the-script-exec-command) for
what to specify for this property.

#### Configuring the working directory

For Framework and Project-wide, configure a property in either the framework.properties or 
project.properties files:

`plugin.script-exec.default.dir`

:   Specifies the default working directory for the execution

For node-specific add an attribute named `script-exec-dir` to the node.

`script-exec-dir`

:   Specifies the default working directory for the execution (optional)

#### Configuring the shell

For Framework and Project-wide, configure a property in either the framework.properties or 
project.properties files:

`plugin.script-exec.default.shell`

:   Specifies the shell to use to interpret the command, e.g. "bash -c" or "cmd.exe /c"

For node-specific add an attribute named `script-exec-shell` to the node.

`script-exec-shell`

:   Specifies the shell to use to interpret the command, e.g. "bash -c" or "cmd.exe /c" (optional)

#### Defining the script-exec command

The value of this property or attribute should be the complete commandline 
string to execute in an external system process.

You can use *Data context properties* as you can in normal Rundeck command 
execution, such as `${node.name}` or `${job.name}`. 

In addition, the plugin provides these new data context properties:

`exec.command`

:   The command that the workflow/user has specified to run on the node

`exec.dir`

:   The working directory path if it is configured for the node or in a properties file

Example:

If you wanted to run some external remote connection command ("/bin/execremote") in lieu of the 
built-in ssh command, you could specify these attributes for node:

    mynode:
        node-executor: script-exec
        script-exec: /bin/execremote -host ${node.hostname} -user ${node.username} -- ${exec.command}

If the command you want to run requires special handling (such as quoting or other interpretation) you may want to have a shell execute it. In which case you could specify the shell to use:

    mynode:
        node-executor: script-exec
        script-exec-shell: bash -c
        script-exec: ssh -o "some quoted option" ${node.username}@${node.hostname} ${exec.command}

At run time, the properties specified would be expanded to the values for the
specific node and command string to execute.

OR, you could specify a default to apply to all nodes within the project.properties 
file located at `$RDECK_BASE/projects/NAME/etc/project.properties`.

    script-exec.default.command= /bin/execremote -host ${node.hostname} \
        -user ${node.username} -- ${exec.command}

Similarly for the `$RDECK_BASE/etc/framework.properties` file to apply to all
projects.

#### Requirements for the script-exec command

The command run by by the script plugin is expected to behave in the following manner:

* Exit with a system exit code of "0" in case of success.
* Any other exit code indicates failure

Note: all output from STDOUT and STDERR will be captured as part of the Rundeck job execution.

#### Configuring script-copy

To configure script-copy you must specify a commandline string to execute.  Optionally
you may specify a directory to be used as the working directory when executing
the commandline string, and a shell to use to interpret the command.  

You must also specify the filepath on the target node where the copied file will be placed, which can be done in two different ways.

You can configure these across all projects (framework-wide), a single project
(project-wide), or specifically for each node, with the most specific configuration
value taking precedence.

#### Configuring the command for script-copy

For Framework and Project-wide, configure these properties in either the framework.properties or
project.properties files:

`plugin.script-copy.default.command`

:   Specifies the default system command to run

For node-specific add these attributes to the node.

`script-copy`

:   Specifies the system command to run

See [Defining the script-copy command](plugins.html#defining-the-script-copy-command) for
what to specify for this property.

#### Configuring the working directory

For Framework and Project-wide, configure a property in either the framework.properties or
project.properties files:

`plugin.script-copy.default.dir`

:   Specifies the default working directory for the execution


For node-specific add an attribute named `script-copy-dir` to the node.

`script-copy-dir`

:   Specifies the default working directory for the execution (optional)

#### Configuring the shell

For Framework and Project-wide, configure a property in either the framework.properties or
project.properties files:

`plugin.script-copy.default.shell`

:   Specifies the shell to run the command (optional)

For node-specific add an attribute named `script-copy-shell` to the node.

`script-copy-shell`

:   Specifies the shell to run the command (optional)

#### リモートファイルパスの設定

フレームワークとプロジェクト内設定のためには、 framework.properties や project.properties ファイルを設定します。

`plugin.script-copy.default.remote-filepath`

:   コピーされるファイルのフルパスを定義します。

ノード依存の場合は、 `script-copy-remote-filepath` 属性をノードに追加します。
For node-specific add an attribute named `script-copy-remote-filepath` to the node.

`script-copy-remote-filepath`

:   コピーされるファイルのフルパスを定義します。

プロパティの詳細は [script-copy ファイルパスの定義](plugins.html#script-copy ファイルパスの定義) を参照してください。

#### script-copy コマンドの定義

このプロパティもしくは属性の値は、外部システムのプロセスで実行するコマンドライン文字列でなければなりません。

通常の Rundeck のコマンド実行で利用できる `${node.name}` や `${job.name}` などのような *データコンテキストプロパティ* を利用できます。

加えて、このプラグインは新たなデータコンテキストプロパティを提供します:

`file-copy.file`

:   リモートノードにコピーするローカルファイルのパス

`file-copy.filename`

:   パス情報を含まないファイル名

例:

標準の SCP コマンドの代わりに外部リモート接続コマンド ("/bin/copyremote") を利用したいとき、ノードごとに以下の属性を指定できます。

    mynode:
        file-copier: script-copy
        script-copy: /bin/copyremote -host ${node.hostname} -user ${node.username} -- ${file-copy.file} ${node.destdir}

実行時、このプロパティは指定したノード用の値に展開され、実行用のコマンド文字列になります。

もしくは、全てのノードに適用されるデフォルト値を `$RDECK_BASE/projects/NAME/etc/project.properties` にある project.properties ファイル内に定義することおｍできます。

    script-copy.default.command= /bin/copyremote -host ${node.hostname} -user ${node.username} -- ${file-copy.file} ${node.destdir}

同様に全てのプロジェクトに適用するために `$RDECK_BASE/etc/framework.properties` ファイルを利用できます。
projects.

#### script-copy ファイルパスの定義

このプロパティもしくは属性の値は、対象のノード上にあるファイルのフルパスでなければなりません。
スクリプトがコピーしたファイルを後から実行できるので、それを FileCopier サービスに伝えるためのものです。

これは *2つ* の方法で実現できます。ここに記載するように設定プロパティとするか、以下に記載する [script-copy コマンドの要件](plugins.html#script-copy コマンドの要件) のようにスクリプトからの出力を通じるかです。

通常の Rundeck のコマンド実行で利用できる `${node.name}` や `${job.name}` などのような *データコンテキストプロパティ* を利用できます。

くわえて、プラグインは新たなデータコンテキストプロパティを提供します:

`file-copy.file`

:   リモートノードにコピーされる必要のあるローカルファイルパス

`file-copy.filename`

:   パス情報を含まないファイル名

例:

上記から "/bin/copyremote" を例にします。リモートノードのどこにファイルをコピーするかきめるため、`script-copy-remote-filepath` を設定する必要があります。この例では `${file-copy.file}` を `${node.destdir}` にコピーします。ノード上のこの属性にディレクトリパスを設定したと仮定します。

`script-copy-remote-filepath` にファイルがコピーされた後のリモートノード上の場所を設定する必要があります。
ファイル名は `${file-copy.filename}` が利用可能なので、`${node.destdir}/${file-copy.filename}` とセットします:

    mynode:
        file-copier: script-copy
        script-copy: /bin/copyremote -host ${node.hostname} -user ${node.username} -- ${file-copy.file} ${node.destdir}
        script-copy-remote-filepath: ${node.destdir}/${file-copy.filename}

実行時、このプロパティは指定したノード用の値に展開され、実行用のコマンド文字列になります。

もしくは、全てのノードに適用されるデフォルト値を `$RDECK_BASE/projects/NAME/etc/project.properties` にある project.properties ファイル内に定義することおｍできます。

    script-copy.default.remote-filepath= ${node.destdir}/${file-copy.filename}

同様に全てのプロジェクトに適用するために `$RDECK_BASE/etc/framework.properties` ファイルを利用できます。

#### script-copy コマンドの要件

script-copy によって実行されるコマンドは以下のマナーに沿うことを求められます。

* 問題なく成功したときは終了コード "0" を返すこと
* 失敗したときはそれ以外の終了コードを返すこと
* **同義**
    * ターゲットノード上にコピーされたファイルのパス出力を STDOUT の1行目に出すこと
    もしくは
    * 上記を "remote-filepath" として定義すること

#### スクリプト例

利用パターンの参考になるスクリプトの例をいくつか載せます。

**script-exec**:

ノード定義:

    mynode:
        node-executor: script-exec

プロジェクト設定ファイル `project.properties`:

    plugin.script-exec.default.command: /tmp/myexec.sh ${node.hostname} ${node.username} -- ${exec.command}

`/tmp/myexec.sh` の内容:

    #!/bin/bash

    # args are [hostname] [username] -- [command to exec...]

    host=$1
    shift
    user=$1
    shift
    command="$*"

    REMOTECMD=ssh

    exec $REMOTECMD $user@$host $command

**script-copy**:

ノード定義:

    mynode:
        file-copier: script-copy
        destdir: /some/node/dir

`framework.properties` 内のシステムワイド設定:

    plugin.script-copy.default.command: /tmp/mycopy.sh ${node.hostname} ${node.username} ${node.destdir} ${file-copy.file}

`/tmp/mycopy.sh` の内容:

    #!/bin/bash

    # args are [hostname] [username] [destdir] [filepath]

    host=$1
    shift
    user=$1
    shift
    dir=$1
    shift
    file=$1

    name=`basename $file`

    # copy to node
    CPCMD=scp

    exec $CPCMD $file $user@$host:$dir/$name > /dev/null || exit $?

    echo "$dir/$name"

**ssh リプレースシステム**:

この例ではシステムの "ssh" と "scp" コマンドをノード実行とファイルのコピーに使っています。
また、外部スクリプトファイルは使用しません:

Node-only configuration:

    mynode:
        hostname: mynode
        username: user1
        node-executor: script-exec
        script-exec: ssh -o "StrictHostKeyChecking no" ${node.username}@${node.hostname} ${exec.command}
        script-exec-shell: bash -c
        file-copier: script-copy
        destdir: /tmp
        script-copy-shell: bash -c
        script-copy: scp ${file-copy.file} ${node.username}@${node.hostname}:${node.destdir}
        script-copy-remote-filepath: ${node.destdir}/${file-copy.filename}

デフォルト値は全て以下のような project.properties ファイル内のものがセットされます:

    # set default node executor
    service.NodeExecutor.default.provider=script-exec

    # set script-exec defaults
    plugin.script-exec.default.command=ssh -o "StrictHostKeyChecking no" ${node.username}@${node.hostname} ${exec.command}
    plugin.script-exec.default.shell=bash -c

    #set default file copier
    service.FileCopier.default.provider=script-copy

    #set script-copy defaults
    plugin.script-copy.default.command=scp ${file-copy.file} ${node.username}@${node.hostname}:${node.destdir}
    plugin.script-copy.default.shell: bash -c
    plugin.script-copy.default.remote-filepath: ${node.destdir}/${file-copy.filename}

このケースではノード定義は単純です:

    mynode:
        hostname: mynode
        username: user1
        destdir: /tmp

### スタブプラグイン

`スタブプラグイン` は以下のプロパイダを含んでいます:

* NodeExecutor サービスの`スタブ`
* FileCopier サービスの`スタブ`

(これらを有効にするには [プロパイダの利用](plugins.html#プロパイダの利用) を参考にして下さい)

このプラグインはリモートファイルのコピーやコマンド実行を実際に行うわけではありません。
コマンドが実行されたかのように単純なエコーを返したり、ファイルがコピーされたように振る舞います。

これは新しいノードやジョブ、ワークフローシーケンスを実際の実行環境に影響を与えずにテストしたい時に利用することを想定しています。

また、以下のノード属性を設定して障害シナリオをテストすることもできます。

`stub-exec-success`="true/false"

:   false にセットした場合、スタブコマンドの実行は失敗としてシミュレートされます

`stub-result-code`

:   実行結果の戻り値をシミュレートします

たとえば、単に `project.properties`　ノード executor プロバイダを `stub` に設定するだけでプロジェクトのワークフローやジョブを一通りテストしたり無効化することができます。

## プラグイン開発

プラグインはスクリプトを使ってカンタンに開発できます。Java も利用可能です。

詳細については [デベロッパーガイド - プラグイン開発](../developer/plugin-development.html) を参照して下さい。
