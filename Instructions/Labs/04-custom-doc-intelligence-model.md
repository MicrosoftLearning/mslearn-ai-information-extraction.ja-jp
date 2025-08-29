---
lab:
  title: カスタム Azure AI Document Intelligence モデルを使用してフォームを分析する
  description: ドキュメントから特定のデータを抽出するカスタム Document Intelligence モデルを作成します。
---

# カスタム Azure AI Document Intelligence モデルを使用してフォームを分析する

ある会社が現在、手動で注文シートを購入し、データベースにデータを入力することを従業員に求めているとします。 会社は AI サービスを活用してデータ入力プロセスを改善したいと考えています。 あなたは、フォームを読み取り、データベースを自動的に更新するために使用できる構造化データを生成する機械学習モデルを構築することを決定しました。

**Azure AI Document Intelligence** は、ユーザーが自動データ処理ソフトウェアを構築できるようにする Azure AI サービスです。 このソフトウェアは、光学式文字認識 (OCR) を使用して、フォーム ドキュメントからテキスト、キーと値のペア、テーブルを抽出できます。 Azure AI Document Intelligence には、請求書、領収書、名刺を認識するための事前構築されたモデルがあります。 このサービスは、カスタム モデルをトレーニングする機能も提供します。 この演習では、カスタム モデルの構築に焦点を当てます。

この演習は Python に基づいていますが、次のような複数の言語固有の SDK を使用して同様のアプリケーションを開発できます。

- [Python 用 Azure AI Document Intelligence クライアント ライブラリ](https://pypi.org/project/azure-ai-formrecognizer/)
- [Microsoft .NET 用 Azure AI Document Intelligence クライアント ライブラリ](https://www.nuget.org/packages/Azure.AI.FormRecognizer)
- [JavaScript 用 Azure AI Document Intelligence クライアント ライブラリ](https://www.npmjs.com/package/@azure/ai-form-recognizer)

この演習は約 **30** 分かかります。

## Azure AI Document Intelligence リソースを作成する

Azure AI Document Intelligence サービスを使用するには、Azure サブスクリプションに Azure AI Document Intelligence または Azure AI Services リソースが必要です。 Azure portal を使用してリソースを作成します。

1. ブラウザー タブで Azure portal (`https://portal.azure.com`) を開き、自分の Azure サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
1. Azure portal のホーム ページで、上部の検索ボックスに「**Document Intelligence**」と入力し、**Enter** キーを押します。
1. **[Document Intelligence]** ページで、**[作成]** を選択します。
1. **[Document Intelligence の作成]** ページで、次の設定で新しいリソースを作成します。
    - **サブスクリプション**:Azure サブスクリプション。
    - **リソース グループ**: リソース グループを作成または選択します
    - **[リージョン]**: 利用可能な任意のリージョン
    - **[名前]**: Document Intelligence リソースの有効な名前
    - **価格レベル**: Free F0 ("Free レベルが使用できない場合" は Standard S0 を選択します)。**
1. デプロイが完了したら、**[リソースに移動]** を選択してリソースの **[概要]** ページを開きます。

## Cloud Shell でアプリを開発する準備をする

Cloud Shell を使用してテキスト翻訳アプリを開発します。 アプリのコード ファイルは、GitHub リポジトリで提供されています。

1. Azure portal で、ページ上部の検索バーの右側にある **[\>_]** ボタンを使用して、Azure portal に新しい Cloud Shell を作成し、***PowerShell*** 環境を選択します。 Azure portal の下部にあるペインに、Cloud Shell のコマンド ライン インターフェイスが表示されます。

    > **注**: *Bash* 環境を使用するクラウド シェルを以前に作成した場合は、それを ***PowerShell*** に切り替えます。

1. コマンド ライン コンソールと Azure portal の両方が表示されるように、Cloud Shell ペインのサイズを変更します。 2 つのペインを切り替えるときは、分割バーを使用して切り替える必要があります。

1. Cloud Shell ツール バーの **[設定]** メニューで、**[クラシック バージョンに移動]** を選択します (これはコード エディターを使用するのに必要です)。

    **<font color="red">続行する前に、クラシック バージョンの Cloud Shell に切り替えたことを確認します。</font>**

1. PowerShell ペインで、次のコマンドを入力して、この演習用の GitHub リポジトリを複製します。

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **ヒント**: Cloudshell にコマンドを貼り付けると、出力が大量のスクリーン バッファーを占有する可能性があります。 `cls` コマンドを入力して、各タスクに集中しやすくすることで、スクリーンをクリアできます。

1. リポジトリが複製されたら、アプリケーション コード ファイルを含むフォルダーに移動します。  

    ```
   cd mslearn-ai-info/Labfiles/custom-doc-intelligence
    ```

## トレーニング用のドキュメントを収集する

次のようなサンプル フォームを使用して、モデルのトレーニングとテストを行います。 

![このプロジェクトで使用される請求書の画像。](./media/Form_1.jpg)

1. コマンド ラインで `ls ./sample-forms` を実行して、**sample-forms** フォルダー内のコンテンツを一覧表示します。 フォルダー内に **.json** と **.jpg** で終わるファイルがあることに注意してください。

    **.jpg** ファイルを使用し、モデルをトレーニングします。  

    **.json** ファイルが生成され、ラベル情報が含まれています。 ファイルは、フォームと共に BLOB ストレージ コンテナーにアップロードされます。

1. **Azure portal** で、リソースの **[概要]** ページに移動します (まだ移動していない場合)。 *[Essentials]* セクションの **[リソース グループ]**、**[サブスクリプション ID]**、**[場所]** をメモします。 これらの値は後の手順で必要になります。
1. コマンド `code setup.sh` を実行して、コード エディターで **setup.sh** を開きます。 このスクリプトを使用して、必要な他の Azure リソースを作成するために必要な Azure コマンド ライン インターフェイス (CLI) コマンドを実行します。

1. **setup.sh** スクリプトのコマンドを確認します。 プログラムは次のことを行います。
    - Azure リソース グループにストレージ アカウントを作成する
    - ローカルの *sampleforms* フォルダーからストレージ アカウントの *sampleforms* というコンテナーにファイルをアップロードする
    - Shared Access Signature URI を印刷する

1. **subscription_id**、**resource_group**、**location** 変数の宣言を、Document Intelligence リソースをデプロイしたサブスクリプション、リソース グループ、およびロケーション名に適切な値で変更します。

    > **重要**: **location** 文字列には、必ず実際の場所のコード バージョンを使用してください。 たとえば、場所が "米国東部" の場合、スクリプト内の文字列は `eastus` にする必要があります。 そのバージョンは、Azure portal のリソース グループの **[Essentials]** タブの右側にある **[JSON ビュー]** ボタンで確認できます。

    **expiry_date** 変数が過去の場合は、将来の日付に更新します。 この変数は、共有アクセス署名 (SAS) URI を生成するときに使用されます。 実際には、SAS に適切な有効期限を設定する必要があります。 SAS について詳しくは、[こちら](https://docs.microsoft.com/azure/storage/common/storage-sas-overview#how-a-shared-access-signature-works)をご覧ください。  

1. プレースホルダーを置き換えたら、コード エディター内で、**Ctrl + S** コマンドを使用するか、**右クリックして保存**で変更を保存してから、**Ctrl + Q** コマンドを使用するか、**右クリックして終了**で、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

1. スクリプトを実行可能にし、行するために、次のコマンドを入力します。

    ```PowerShell
   chmod +x ./setup.sh
   ./setup.sh
    ```

1. スクリプトが完了したら、表示された出力を確認します。

1. Azure portal で、リソース グループを更新し、作成したばかりの Azure ストレージ アカウントが含まれていることを確認します。 ストレージ アカウントを開き、左側のウィンドウで **[ストレージ ブラウザー]** を選択します。 次に、ストレージ ブラウザーで、**[BLOB コンテナー]** を展開し、**sampleforms** コンテナーを選択して、ファイルがローカルの **custom-doc-intelligence/sample-forms** フォルダーからアップロードされていることを確認します。

## Document Intelligence Studio を使用してモデルをトレーニングする

次に、ストレージ アカウントにアップロードされたファイルを使用してモデルをトレーニングします。

1. 新しいブラウザー タブを開き、`https://documentintelligence.ai.azure.com/studio` の Document Intelligence Studio に移動します。 
1. **[カスタム モデル]** セクションまで下にスクロールし、**[カスタム抽出モデル]** タイルを選択します。
1. プロンプトが表示されたら、Azure 資格情報を使用してサインインします。
1. 使用する Azure AI Document Intelligence リソースを確認するメッセージが表示されたら、Azure AI Document Intelligence リソースの作成時に使用したサブスクリプションとリソース名を選択します。
1. **[マイ プロジェクト]** で、次の構成で新しいプロジェクトを作成します。

    - **プロジェクトの詳細の入力**:
        - **[プロジェクト名]**:プロジェクトの有効な名前
    - **サービス リソースの構成**:
        - **サブスクリプション**:お使いの Azure サブスクリプション
        - **[リソース グループ]**: Document Intelligence リソースをデプロイしたリソース グループ
        - **[Document Intelligence リソース]**: 実際の Document Intelligence リソース ([既定として設定] オプションを選択し、既定の API バージョンを使用します)**
    - **トレーニング データ ソースの接続**:
        - **サブスクリプション**:お使いの Azure サブスクリプション
        - **[リソース グループ]**: Document Intelligence リソースをデプロイしたリソース グループ
        - **[ストレージ アカウント]**: セットアップ スクリプトによって作成されたストレージ アカウント ([既定として設定] オプションを選択し、`sampleforms` BLOB コンテナーを選択し、フォルダー パスを空白のままにします)**

1. プロジェクトが作成されたら、ページの右上にある **[トレーニング]** を選択してモデルをトレーニングします。 次の構成を使用します。
    - **モデル ID**:モデルの有効な名前 (次の手順でモデル ID 名が必要になります)**
    - **テンプレートの構築**: テンプレート。
1. **[モデルに移動]** を選択します。
1. トレーニングには時間がかかる場合があります。 状態が**成功**になるまで待ちます。

## カスタム Document Intelligence モデルをテストする

1. Azure Portal と Cloud Shell が表示されているブラウザー タブに戻ります。 コマンド ラインで次のコマンドを実行して、アプリケーション コード ファイルが格納されているフォルダーに変更します。

    ```
    cd Python
    ```

1. 次のコマンドを実行して、Document Intelligence パッケージをインストールします。

    ```
    python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-formrecognizer==3.3.3
    ```

1. 次のコマンドを入力して、提供されている構成ファイルを編集します。

    ```
   code .env
    ```

1. Azure portal が表示されているペインの Document Intelligence リソースの **[概要]** ページで、**[キーを管理するにはここをクリック]** を選択して、リソースのエンドポイントとキーを表示します。 次に、次の値で構成ファイルを編集します。
    - 実際の Document Intelligence エンドポイント
    - 実際の Document Intelligence キー
    - モデルのトレーニング時に指定したモデル ID

1. プレースホルダーを置き換えたら、コード エディター内で **Ctrl + S** コマンドを使用して変更を保存してから、**Ctrl + Q** コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

1. クライアント アプリケーションのコード ファイル (C# の場合は `code Program.cs`、Python の場合は `code test-model.py`) を開き、含まれているコードを確認します。特に、URL の画像が Web 上のこの GitHub リポジトリのファイルを参照していることを確認します。 変更を加えずにファイルを閉じます。

1. コマンド ラインで次のコマンドを入力してプログラムを実行します。

    ```
   python test-model.py
    ```

1. 出力を表示し、`Merchant` や `CompanyPhoneNumber` などのフィールド名がモデルの出力に含まれる様子を観察します。

## クリーンアップ

Azure リソースでの作業が完了したら、追加料金が発生しないように、[Azure portal](https://portal.azure.com/?azure-portal=true) 内のリソースを必ず削除してください。

## 詳細情報

Document Intelligence サービスの詳細については、[Document Intelligence のドキュメント](https://learn.microsoft.com/azure/ai-services/document-intelligence/?azure-portal=true)を参照してください。
