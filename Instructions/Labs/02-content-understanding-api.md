---
lab:
  title: コンテンツ解釈クライアント アプリケーションを開発する
  description: Azure AI コンテンツ解釈 REST API を使用して、アナライザーのクライアント アプリを開発します。
---

# コンテンツ解釈クライアント アプリケーションを開発する

この演習では、Azure AI コンテンツ解釈を使用して、名刺から情報を抽出するアナライザーを作成します。 次に、アナライザーを使用してスキャンした名刺から連絡先の詳細を抽出するクライアント アプリケーションを開発します。

この演習は約 **30** 分かかります。

## Azure AI Foundry ハブとプロジェクトを作成する

この演習で使用する Azure AI Foundry の機能には、Azure AI Foundry *ハブ* リソースに基づくプロジェクトが必要です。

1. Web ブラウザーで [Azure AI Foundry ポータル](https://ai.azure.com) (`https://ai.azure.com`) を開き、Azure 資格情報を使用してサインインします。 初めてサインインするときに開いたヒントまたはクイック スタート ウィンドウを閉じます。また、必要に応じて左上にある **Azure AI Foundry** ロゴを使用してホーム ページに移動します。それは次の画像のようになります (**[ヘルプ]** ウィンドウが開いている場合は閉じます)。

    ![Azure AI Foundry ポータルのスクリーンショット。](./media/ai-foundry-home.png)

1. ブラウザーで `https://ai.azure.com/managementCenter/allResources` に移動し、**[新規作成]** を選択します。 次に、新しい **AI ハブ リソース**を作成するオプションを選択します。
1. **[プロジェクトの作成]** ウィザードで、プロジェクトの有効な名前を入力し、新しいハブを作成するオプションを選択します。 次に、**[ハブ名の変更]** リンクを使用して新しいハブの有効な名前を指定し、**[詳細オプション]** を展開し、プロジェクトに次の設定を指定します。
    - **[サブスクリプション]**:"*ご自身の Azure サブスクリプション*"
    - **リソース グループ**: *リソース グループを作成または選択します*
    - **ハブ名**: ハブの有効な名前
    - **[場所]**: 次のいずれかの場所を選択します。\*
        - オーストラリア東部
        - スウェーデン中部
        - 米国西部

    > \*執筆時点では、Azure AI コンテンツ解釈はこれらのリージョンでのみ使用できます。

    > **ヒント**: **[作成]** ボタンが無効な場合は、ハブの名前を一意の英数字にしてください。

1. プロジェクトが作成されるまで待ってから、プロジェクトの概要ページに移動します。

## REST API を使用してコンテンツ解釈アナライザーを作成する

REST API を使用して、名刺の画像から情報を抽出できるアナライザーを作成します。

1. 新しいブラウザー タブを開きます (既存のタブで Azure AI Foundry ポータルを開いたままにします)。 新しいブラウザー タブで [Azure portal](https://portal.azure.com) (`https://portal.azure.com`) を開き、メッセージに応じて Azure 資格情報を使用してサインインします。

    ウェルカム通知を閉じて、Azure portal のホーム ページを表示します。

1. ページ上部の検索バーの右側にある **[\>_]** ボタンを使用して、Azure portal に新しい Cloud Shell を作成し、サブスクリプションにストレージがない ***PowerShell*** 環境を選択します。

    Azure portal の下部にあるペインに Cloud Shell のコマンド ライン インターフェイスが表示されます。 作業しやすくするために、このウィンドウのサイズを変更したり最大化したりすることができます。

    > **ヒント**: ペインのサイズを変更して、主に Cloud Shell で作業している際に Azure portal ページにキーとエンドポイントが表示されるようにします。これらをコードにコピーする必要があります。

1. Cloud Shell ツール バーの **[設定]** メニューで、**[クラシック バージョンに移動]** を選択します (これはコード エディターを使用するのに必要です)。

    **<font color="red">続行する前に、クラシック バージョンの Cloud Shell に切り替えたことを確認します。</font>**

1. Cloud Shell 画面で、次のコマンドを入力して、この演習のコード ファイルを含む GitHub リポジトリを複製します (コマンドを入力するか、クリップボードにコピーしてから、コマンド ラインで右クリックし、プレーンテキストとして貼り付けます)。

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **ヒント**: Cloudshell にコマンドを入力すると、出力が大量のスクリーン バッファーを占有する可能性があります。 `cls` コマンドを入力して、各タスクに集中しやすくすることで、スクリーンをクリアできます。

1. リポジトリが複製されたら、アプリのコード ファイルを含んだフォルダーに移動します。

    ```
   cd mslearn-ai-info/Labfiles/content-app
   ls -a -l
    ```

    フォルダーには、スキャンした名刺画像 2 枚と、アプリの構築に必要な Python コード ファイルが格納されています。

1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して、使用するライブラリをインストールします。

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt
    ```

1. 次のコマンドを入力して、提供されている構成ファイルを編集します。

    ```
   code .env
    ```

    このファイルをコード エディターで開きます。

1. コード ファイルのプレースホルダー **YOUR_ENDPOINT** と **YOUR_KEY** を、Azure AI サービス エンドポイントとそのキーのいずれか (Azure portal からコピーしたもの) に置き換えます。また、必ず **ANALYZER_NAME** を `business-card-analyzer` に設定します。
1. プレースホルダーを置き換えたら、コード エディター内で **Ctrl + S** コマンドを使用して変更を保存してから、**Ctrl + Q** コマンドを使用して、Cloud Shell コマンド ラインを開いたままコード エディターを閉じます。

    > **ヒント**: Cloud Shell ペインを最大化できます。

1. Cloud Shell のコマンド ラインに次のコマンドを入力して、提供された **biz-card.json** JSON ファイルを表示します。

    ```
   cat biz-card.json
    ```

    Cloud Shell ペインをスクロールして、名刺のアナライザー スキーマが定義されているファイルの JSON を確認します。

1. アナライザーの JSON ファイルを確認したら、次のコマンドを入力して、提供された **create-analyzer.py** Python コード ファイルを編集します。

    ```
   code create-analyzer.py
    ```

    Python コード ファイルがコード エディターで開かれます。

1. コードを確認します。以下の処理が行われます。
    - **biz-card.json** ファイルからアナライザー スキーマを読み込みます。
    - 環境構成ファイルからエンドポイント、キー、アナライザー名を取得します。
    - まだ実装されていない **create_analyzer** という関数を呼び出します

1. **create_analyzer** 関数で、コメント "**Create a Content Understanding analyzer**" を見つけて、次のコードを追加します (正しいインデントを維持するように注意してください)。

    ```python
   # Create a Content Understanding analyzer
   print (f"Creating {analyzer}")

   # Set the API version
   CU_VERSION = "2025-05-01-preview"

   # initiate the analyzer creation operation
   headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"}

   url = f"{endpoint}/contentunderstanding/analyzers/{analyzer}?api-version={CU_VERSION}"

   # Delete the analyzer if it already exists
   response = requests.delete(url, headers=headers)
   print(response.status_code)
   time.sleep(1)

   # Now create it
   response = requests.put(url, headers=headers, data=(schema))
   print(response.status_code)

   # Get the response and extract the callback URL
   callback_url = response.headers["Operation-Location"]

   # Check the status of the operation
   time.sleep(1)
   result_response = requests.get(callback_url, headers=headers)

   # Keep polling until the operation is no longer running
   status = result_response.json().get("status")
   while status == "Running":
        time.sleep(1)
        result_response = requests.get(callback_url, headers=headers)
        status = result_response.json().get("status")

   result = result_response.json().get("status")
   print(result)
   if result == "Succeeded":
        print(f"Analyzer '{analyzer}' created successfully.")
   else:
        print("Analyzer creation failed.")
        print(result_response.json())
    ```

1. 追加したコードを確認します。実行内容は次のとおりです。
    - REST 要求に適切なヘッダーを作成します
    - アナライザーが既に存在する場合は、HTTP *DELETE* 要求を送信してアナライザーを削除します。
    - アナライザーの作成を開始する HTTP *PUT* 要求を送信します。
    - 応答を確認して、*Operation-Location* コールバック URL を取得します。
    - 実行が終了するまで、コールバック URL に HTTP *GET* 要求を繰り返し送信して、操作の状態を確認します。
    - 操作の成功 (または失敗) をユーザーに確認します。

    > **注**:このコードには、サービスの要求レート制限を超えないように、意図的な時間遅延が組み込まれています。

1. **Ctrl + S** キー コマンドを使用してコードの変更を保存します。ただし、コード内のエラーを修正する必要がある場合に備えて、コード エディター ペインは開いたままにしておきます。 コマンド ライン ペインがよく見えるようにペインのサイズを変更します。
1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して Python コードを実行します。

    ```
   python create-analyzer.py
    ```

1. プログラムの出力を確認します。アナライザーが作成されたことが示されているはずです。

## REST API を使用してコンテンツを分析する

アナライザーを作成したので、コンテンツ解釈 REST API を使用してクライアント アプリケーションからアナライザーを実行できます。

1. Cloud Shell コマンド ラインで次のコマンドを入力して、提供された **read-card.py** Python コード ファイルを編集します。

    ```
   code read-card.py
    ```

    Python コード ファイルはコード エディターで開かれます。

1. コードを確認します。以下の処理が行われます。
    - 分析する画像ファイルを特定します。既定値は **biz-card-1.png** です。
    - プロジェクトから Azure AI サービス リソースのエンドポイントとキーを取得します (認証には現在の Cloud Shell セッションの Azure 資格情報を使用します)。
    - まだ実装されていない **analyze_card** という関数を呼び出します

1. **analyze_card** 関数で、コメント "**Use Content Understanding to analyze the image**" を見つけて、次のコードを追加します (正しいインデントを維持するように注意してください)。

    ```python
   # Use Content Understanding to analyze the image
   print (f"Analyzing {image_file}")

   # Set the API version
   CU_VERSION = "2025-05-01-preview"

   # Read the image data
   with open(image_file, "rb") as file:
        image_data = file.read()
    
   ## Use a POST request to submit the image data to the analyzer
   print("Submitting request...")
   headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/octet-stream"}
   url = f'{endpoint}/contentunderstanding/analyzers/{analyzer}:analyze?api-version={CU_VERSION}'
   response = requests.post(url, headers=headers, data=image_data)

   # Get the response and extract the ID assigned to the analysis operation
   print(response.status_code)
   response_json = response.json()
   id_value = response_json.get("id")

   # Use a GET request to check the status of the analysis operation
   print ('Getting results...')
   time.sleep(1)
   result_url = f'{endpoint}/contentunderstanding/analyzerResults/{id_value}?api-version={CU_VERSION}'
   result_response = requests.get(result_url, headers=headers)
   print(result_response.status_code)

   # Keep polling until the analysis is complete
   status = result_response.json().get("status")
   while status == "Running":
        time.sleep(1)
        result_response = requests.get(result_url, headers=headers)
        status = result_response.json().get("status")

   # Process the analysis results
   if status == "Succeeded":
        print("Analysis succeeded:\n")
        result_json = result_response.json()
        output_file = "results.json"
        with open(output_file, "w") as json_file:
            json.dump(result_json, json_file, indent=4)
            print(f"Response saved in {output_file}\n")

        # Iterate through the fields and extract the names and type-specific values
        contents = result_json["result"]["contents"]
        for content in contents:
            if "fields" in content:
                fields = content["fields"]
                for field_name, field_data in fields.items():
                    if field_data['type'] == "string":
                        print(f"{field_name}: {field_data['valueString']}")
                    elif field_data['type'] == "number":
                        print(f"{field_name}: {field_data['valueNumber']}")
                    elif field_data['type'] == "integer":
                        print(f"{field_name}: {field_data['valueInteger']}")
                    elif field_data['type'] == "date":
                        print(f"{field_name}: {field_data['valueDate']}")
                    elif field_data['type'] == "time":
                        print(f"{field_name}: {field_data['valueTime']}")
                    elif field_data['type'] == "array":
                        print(f"{field_name}: {field_data['valueArray']}")
    ```

1. 追加したコードを確認します。実行内容は次のとおりです。
    - 画像ファイルの内容を読み取ります
    - 使用するコンテンツ解釈 REST API のバージョンを設定します
    - コンテンツ解釈エンドポイントに HTTP *POST* 要求を送信し、画像を分析するように指示します。
    - 操作からの応答を確認して、分析操作の ID を取得します。
    - 実行が停止するまで、コンテンツ解釈エンドポイントに HTTP *GET* 要求を繰り返し送信して、操作の状態を確認します。
    - 操作が成功した場合は、JSON 応答を保存し、JSON を解析して、各種類固有のフィールドで取得された値を表示します。

    > **注**:シンプルな名刺スキーマでは、すべてのフィールドは文字列です。 ここのコードは、より複雑なスキーマからさまざまな値を抽出できるように、各フィールドの種類を確認する必要があることを示しています。

1. **Ctrl + S** キー コマンドを使用してコードの変更を保存します。ただし、コード内のエラーを修正する必要がある場合に備えて、コード エディター ペインは開いたままにしておきます。 コマンド ライン ペインがよく見えるようにペインのサイズを変更します。
1. Cloud Shell コマンド ライン ペインで、次のコマンドを入力して Python コードを実行します。

    ```
   python read-card.py biz-card-1.png
    ```

1. プログラムからの出力を確認します。次の名刺のフィールドの値が表示されるはずです。

    ![Adventure Works Cycles の従業員、Roberto Tamburello の名刺。](./media/biz-card-1.png)

1. 次のコマンドを使用して、プログラムを別の名刺で実行します。

    ```
   python read-card.py biz-card-2.png
    ```

1. 結果を確認します。この名刺の値が反映されるはずです。

    ![Contoso の従業員 Mary Duartes の名刺。](./media/biz-card-2.png)

1. Cloud Shell のコマンド ライン ペインで次のコマンドを使用して、返された JSON 応答全体を表示します。

    ```
   cat results.json
    ```

    スクロールして JSON を表示します。

## クリーンアップ

コンテンツ解釈サービスの操作が終わったら、不要な Azure コストが発生しないように、この演習で作成したリソースを削除する必要があります。

1. Azure portal で、この演習で作成したリソースを削除します。
