---
lab:
  title: 事前構築済み Azure AI Document Intelligence モデルを使用してフォームを分析する
  description: 事前構築済みの Azure AI Document Intelligence モデルを使用して、ドキュメントのテキスト フィールドを処理します。
---

# 事前構築済み Azure AI Document Intelligence モデルを使用してフォームを分析する

この演習では、ドキュメント分析に必要なすべてのリソースを備えた Azure AI Foundry プロジェクトを設定します。 Azure AI Foundry ポータルと Python SDK の両方を使用して、分析のためにそのリソースにフォームを送信します。

この演習は Python に基づいていますが、次のような複数の言語固有の SDK を使用して同様のアプリケーションを開発できます。

- [Python 用 Azure AI Document Intelligence クライアント ライブラリ](https://pypi.org/project/azure-ai-formrecognizer/)
- [Microsoft .NET 用 Azure AI Document Intelligence クライアント ライブラリ](https://www.nuget.org/packages/Azure.AI.FormRecognizer)
- [JavaScript 用 Azure AI Document Intelligence クライアント ライブラリ](https://www.npmjs.com/package/@azure/ai-form-recognizer)

この演習は約 **30** 分かかります。

## Azure AI Foundry プロジェクトを作成する

まず、Azure AI Foundry プロジェクトを作成します。

1. Web ブラウザーで [Azure AI Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインするときに開いたヒントまたはクイック スタート ウィンドウを閉じます。また、必要に応じて左上にある **Azure AI Foundry** ロゴを使用してホーム ページに移動します。それは次の画像のようになります (**[ヘルプ]** ウィンドウが開いている場合は閉じます)。

    ![Azure AI Foundry ポータルのスクリーンショット。](./media/ai-foundry-home.png)

1. ブラウザーで `https://ai.azure.com/managementCenter/allResources` に移動し、**[新規作成]** を選択します。 次に、新しい **AI ハブ リソース**を作成するオプションを選択します。
1. **[プロジェクトの作成]** ウィザードで、プロジェクトの有効な名前を入力し、新しいハブを作成するオプションを選択します。 次に、**[ハブ名の変更]** リンクを使用して新しいハブの有効な名前を指定し、**[詳細オプション]** を展開し、プロジェクトに次の設定を指定します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを作成または選択します*
    - **[リージョン]**:  使用できるリージョン**

    > **メモ**: ポリシーを使用して使用可能なリソース名を制限する Azure サブスクリプションで作業している場合は、**[新しいプロジェクトの作成]** ダイアログ ボックスの下部にあるリンクを使用して、Azure portal を使用してハブを作成する必要があります。

    > **ヒント**: **[作成]** ボタンが無効な場合は、ハブの名前を一意の英数字にしてください。

1. プロジェクトが作成されるまで待ちます。
1. プロジェクトが作成されたら、表示されているヒントをすべて閉じて、Azure AI Foundry ポータルのプロジェクト ページを確認します。これは次の画像のようになっているはずです。

    ![Azure AI Foundry ポータルの Azure AI プロジェクトの詳細のスクリーンショット。](./media/ai-foundry-project.png)

## 読み取りモデルを使用する

最初に、**Azure AI Foundry** ポータルと読み取りモデルを使って、複数の言語が使われているドキュメントを分析してみましょう。

1. 左側のナビゲーション パネルで **[AI サービス]** を選択します。
1. **[Azure AI サービス]** ページの **[Vision + ドキュメント]** タイルを選択します。
1. **[Vision + ドキュメント]** ページで、**[ドキュメント]** タブが選択されていることを確認し、**[OCR/読み取り]** タイルを選択します。

    **[読み取り]** ページでは、プロジェクトで作成された Azure AI サービス リソースが既に接続されているはずです。

1. 左側のドキュメントの一覧で、**read-german.pdf** を選択します。

    ![Azure AI Document Intelligence Studio の [読み取り] ページを示すスクリーンショット。](./media/read-german-sample.png#lightbox)

1. 上部のツール バーで **[分析オプション]** を選択し、**[分析オプション]** ペインの **[言語]** チェックボックス (**[省略可能な検出]** の下) をオンにして、**[保存]** を選択します。 
1. 左上の **[分析の実行]** を選びます。
1. 分析が完了すると、画像から抽出されたテキストが右側の **[コンテンツ]** タブに表示されます。このテキストを確認し、元の画像のテキストと比較して正確さを調べます。
1. **[結果]** タブを選びます。このタブには、抽出された JSON コードが表示されます。 

## Cloud Shell でアプリを開発する準備をする

次に、Azure Document Intelligence Service SDK を使用するアプリを調べてみましょう。 Cloud Shell を使用してアプリを開発します。 アプリのコード ファイルは、GitHub リポジトリで提供されています。

この請求書をコードで分析します。

![請求書ドキュメントのサンプルを示すスクリーンショット。](./media/sample-invoice.png)

1. Azure AI Foundry ポータルで、プロジェクトの **[概要]** ページを表示します。
1. **[エンドポイントとキー]** 領域で **[Azure AI サービス]** タブを選択し、**[API キー]** と **[Azure AI サービス エンドポイント]** をメモします。 これらの資格情報を使用して、クライアント アプリケーションで Azure AI サービスに接続します。
1. 新しいブラウザー タブを開きます (既存のタブで Azure AI Foundry ポータルを開いたままにします)。 新しいブラウザー タブで [Azure portal](https://portal.azure.com) (`https://portal.azure.com`) を開き、メッセージに応じて Azure 資格情報を使用してサインインします。
1. ページ上部の検索バーの右側にある **[\>_]** ボタンを使用して、Azure portal に新しい Cloud Shell を作成します。***PowerShell*** 環境を選択します。 Azure portal の下部にあるペインに、Cloud Shell のコマンド ライン インターフェイスが表示されます。

    > **注**: *Bash* 環境を使用するクラウド シェルを以前に作成した場合は、それを ***PowerShell*** に切り替えます。

1. Cloud Shell ツール バーの **[設定]** メニューで、**[クラシック バージョンに移動]** を選択します (これはコード エディターを使用するのに必要です)。

    **<font color="red">続行する前に、クラシック バージョンの Cloud Shell に切り替えたことを確認します。</font>**

1. PowerShell ペインで、次のコマンドを入力して、この演習用の GitHub リポジトリを複製します。

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **ヒント**: Cloudshell にコマンドを貼り付けると、出力が大量のスクリーン バッファーを占有する可能性があります。 `cls` コマンドを入力して、各タスクに集中しやすくすることで、スクリーンをクリアできます。

    ***選択したプログラミング言語の手順に従います。***

1. リポジトリがクローンされたら、コード ファイルが格納されているフォルダーに移動します。

    ```
   cd mslearn-ai-info/Labfiles/prebuilt-doc-intelligence/Python
    ```

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して、これから使用するライブラリをインストールします。

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-formrecognizer==3.3.3
    ```

1. 次のコマンドを入力して、提供されている構成ファイルを編集します。

    ```
   code .env
    ```

    このファイルをコード エディターで開きます。

1. コード ファイルのプレースホルダー **YOUR_ENDPOINT** と **YOUR_KEY** を、Azure AI サービス エンドポイントとその API キー (Azure AI Foundry ポータルからコピーしたもの) に置き換えます。
1. プレースホルダーを置き換えたら、コード エディター内で **Ctrl + S** コマンドを使用して変更を保存してから、**Ctrl + Q** コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

## Azure Document Intelligence サービスを使用するコードを追加する

これで、SDK を使用して PDF ファイルを評価する準備が整いました。

1. 次のコマンドを入力して、提供されたアプリ ファイルを編集します。

    ```
   code document-analysis.py
    ```

    このファイルをコード エディターで開きます。

1. コード ファイルで、コメント "**Import the required libraries**" を見つけて、次のコードを追加します。

    ```python
   # Add references
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.formrecognizer import DocumentAnalysisClient
    ```

1. コメント "**Create the client**" を見つけて、次のコードを追加します (正しいインデント レベルを維持するように注意してください)。

    ```python
   # Create the client
   document_analysis_client = DocumentAnalysisClient(
        endpoint=endpoint, credential=AzureKeyCredential(key)
   )
    ```

1. コメント "**Analyze the invoice**" を見つけて、次のコードを追加します。

    ```python
   # Analyse the invoice
   poller = document_analysis_client.begin_analyze_document_from_url(
        fileModelId, fileUri, locale=fileLocale
   )
    ```

1. コメント "**Display invoice information to the user**" を見つけて、次のコードを追加します。

    ```python
   # Display invoice information to the user
   receipts = poller.result()
    
   for idx, receipt in enumerate(receipts.documents):
    
        vendor_name = receipt.fields.get("VendorName")
        if vendor_name:
            print(f"\nVendor Name: {vendor_name.value}, with confidence {vendor_name.confidence}.")

        customer_name = receipt.fields.get("CustomerName")
        if customer_name:
            print(f"Customer Name: '{customer_name.value}, with confidence {customer_name.confidence}.")


        invoice_total = receipt.fields.get("InvoiceTotal")
        if invoice_total:
            print(f"Invoice Total: '{invoice_total.value.symbol}{invoice_total.value.amount}, with confidence {invoice_total.confidence}.")
    ```

1. コード エディターで、**Ctrl + S** キー コマンドを使用するか、**右クリック > [保存]** を選択して変更を保存します。 コード内のエラーを修正する必要がある場合に備えて、コード エディターを開いたままにしておきます。ただし、コマンド ライン ペインがよく見えるようにペインのサイズを変更します。

1. コマンド ライン ペインで、次のコマンドを入力してアプリケーションを実行します。

    ```
    python document-analysis.py
    ```

プログラムにより、ベンダー名、顧客名、請求書合計と信頼度レベルが表示されます。 表示された値と、このセクションの最初で開いたサンプルの請求書を比べます。

## クリーンアップ

Azure リソースでの作業が完了したら、追加料金が発生しないように、[Azure portal](https://portal.azure.com) (`https://portal.azure.com`) 内のリソースを必ず削除してください。
