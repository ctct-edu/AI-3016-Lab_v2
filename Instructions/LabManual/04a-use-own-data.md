---
lab:
  title: ツールを使用する生成 AI アプリを作成する
  description: ツールを使用してモデルの機能を拡張する方法について学びます。
  level: 300
  duration: 30
  islab: true
---

# ツールを使用する生成 AI アプリを作成する

この演習では、Microsoft Foundry ポータルと Responses API を使用して AI チャット アプリケーションを構築します。 次に、*web_search* と *file_search* ツールを使用して、アプリケーションにナレッジを統合します。

この演習は約 **30** 分かかります。

> **注**: この演習で使用されるテクノロジの一部は、プレビューの段階または開発中の段階です。 予期しない動作、警告、またはエラーが発生する場合があります。

## プレイグラウンドでツールを試す

チャット アプリケーションを開発する前に、プレイグラウンドでモデルがどのように応答するかを調べてみましょう。 これは、典拠データが重要な理由を理解するのに役立ちます。

1. モデルのプレイグラウンドに移動します。左側のウィンドウで、**gpt-4.1** モデルが選択されていることを確認します。

1. **[手順（もしくは指示）]** フィールドに、次のプロンプトを入力します。

    ```
   あなたは、マーギーズ・トラベルが提供する旅行サービスに関する情報を案内する旅行アシスタントです。
   ```

1. チャット ペインに、「`来月、ニューヨークでおすすめの観光アクティビティは何がありますか？`」という質問を入力し、応答を確認します。

    応答はかなり一般的なはずです。モデルはトレーニング データに基づいた一般的なサポート情報を提供しますが、来月ニューヨークで何が起きるかについて現在の情報にはアクセスできません。

1. 左側のペインの手順の下にある **[ツール]** セクションで **[追加]** を選択し、**web検索** ツールを追加します。

1. チャット ウィンドウで、「`来月、ニューヨークでおすすめの観光アクティビティは何がありますか？`」という質問を入力し、応答を確認します。

    今度は、モデルが *web_search* ツールを使用して、ニューヨークでのアクティビティに関する現在の情報を検索します。

## ツールを使用するアプリを作成する

プレイグラウンドでツールを使用してモデルの機能を拡張する方法を確認したので、ツールを使用して Margie's Travel のお客様に旅行に関するアドバイスを提供するクライアント アプリケーションを構築しましょう。

# エンドポイントを取得する

クライアント アプリケーションからモデルに接続するには、エンドポイントが必要です。 この演習では、OpenAI SDK を使用してモデルとチャットします。また、Azure OpenAI エンドポイントと Entra ID 認証を使用して接続します。

> **注**: Entra ID 認証の代わりに、プロジェクトに API キーを使用できます。 可能な限り、Entra ID 認証を使用することをお勧めします。

1. メニュー バーで、**[ホーム]** ページを選択します。
1. そこに **Azure OpenAI エンドポイント**が表示されていることに注目します。

    > **ヒント**: この演習では、**Azure OpenAI エンドポイント**を使用します。プロジェクト エンドポイントは<u>使用しません</u>。

### GitHub からアプリケーション ファイルを取得する

チャット アプリケーションの開発に必要な初期アプリケーション ファイルが、GitHub リポジトリに提供されます。

1. Visual Studio Code を開きます。
1. コマンド パレット (*Ctrl + Shift + P*) を開き、`Git:clone` コマンドを使用して、`https://github.com/microsoftlearning/mslearn-ai-studio` リポジトリをローカル フォルダーにクローンします (どのフォルダーでもかまいません)。 次に、それを開きます。

    作成者を信頼することを確認するメッセージが表示される場合があります。

### アプリケーション構成を準備する

1. [エクスプローラー] ペインで、**/labfiles/tools/python/tools-app** にあるアプリケーション コード ファイルを含むフォルダーに移動します。 アプリケーション フローには以下が含まれます。

    - **brochures** (Margie's Travel のパンフレットを含むフォルダー)
    - **.env** (アプリケーション構成ファイル)
    - **requirements.txt** (インストールする必要がある Python パッケージの依存関係)
    - **tools-app.py** (アプリケーションのコード ファイル)
1. **[エクスプローラー]** ペインで、アプリケーション ファイルを含む **tools-app** フォルダーを右クリックし、**[統合ターミナルで開く]** (または **[ターミナル]** メニューでターミナルを開き、*/labfiles/tools/python/tools-app* フォルダーに移動します)。

    > **注**: Visual Studio Code でターミナルを開くと、Python 環境が自動的にアクティブ化されます。 お使いのシステムでスクリプトの実行を有効にする必要がある場合があります。

1. 作成した Python 環境がアクティブであることを示すプレフィックス **(.venv)** の付いた **/labfiles/tools/python/tools-ap** フォルダーでターミナルが開かれていることを確認します。
1. 次のコマンドを実行して、OpenAI SDK、Azure ID、およびその他の必要なパッケージをインストールします。

    ```
    pip install -r requirements.txt
    ```

1. **[エクスプローラー]** ペインの **/labfiles/tools/python/tools-app** フォルダーで、**.env** ファイルを選択して開きます。 次に、構成値を更新して、**Azure OpenAI エンドポイント**と **gpt-4.1** モデルのデプロイに割り当てられた名前を含めます。

    > **ヒント**: Foundry ポータルのプロジェクト ホーム ページから **Azure OpenAI エンドポイント** (プロジェクト エンドポイントではありません) をコピーし、`MODEL_DEPLOYMENT` 設定でデプロイに割り当てられている正確なデプロイの名前を入力します。

    変更した構成ファイルを保存します。

### ツールを使用してチャットを実装するコードを記述する

1. **[エクスプローラー]** ペインの **/labfiles/tools/python/tools-app** フォルダーで、**tools-app.py** ファイルを選択して開きます。

1. 既存のコードを確認します。 OpenAI SDK を使用してモデルにアクセスするコードを追加します。

    > **ヒント**: コード ファイルにコードを追加する際は、必ず正しいインデントを維持してください。

1. コード ファイルの上部にある既存の名前空間参照の下で、**Import namespaces (名前空間をインポートする)** というコメントを検索し、OpenAI SDK を使用するために必要な名前空間をインポートする次のコードを追加します。

    ```python
   # import namespaces
   from openai import OpenAI
   from azure.identity import DefaultAzureCredential, get_bearer_token_provider
   ```

1. **main** 関数では、構成ファイルからエンドポイントとキーを読み込むためのコードが既に提供されていることに注目してください。 次に、**Initialize the OpenAI client (OpenAI クライアントを初期化する)** というコメントを見つけ、次のコードを追加して、OpenAI API 用のクライアントを作成します。

    ```python
   # Initialize the OpenAI client
   token_provider = get_bearer_token_provider(
        DefaultAzureCredential(), "https://ai.azure.com/.default"
   )
    
   openai_client = OpenAI(
        base_url=azure_openai_endpoint,
        api_key=token_provider
   )
   ```

1. **main** 関数で、**Create vector store and upload files (ベクトル ストアを作成しファイルをアップロードする)** というコメントを検索して次のコードを追加します。

    ```python
   # Create vector store and upload files
   print("Creating vector store and uploading files...")
   vector_store = openai_client.vector_stores.create(
        name="travel-brochures"
   )
   file_streams = [open(f, "rb") for f in glob.glob("brochures/*.pdf")]
   if not file_streams:
        print("No PDF files found in the brochures folder!")
        return
   file_batch = openai_client.vector_stores.file_batches.upload_and_poll(
        vector_store_id=vector_store.id,
        files=file_streams
   )
   for f in file_streams:
        f.close()
   print(f"Vector store created with {file_batch.file_counts.completed} files.")
   ```

    このコードでは、モデルのベクトル ストアを作成し、パンフレットをアップロードします。 このベクトル ストアは、*file_search* ツールで使用します。

1. **main** 関数で、ユーザーがアプリを終了するまでユーザー プロンプトを要求するコードが提供されていることに注目してください。 このループ内で、**Get a response using tools (ツールを使って応答を取得する)** コメントを検索し、次のコードを追加します。

    ```python
   # Get a response using tools
   response = openai_client.responses.create(
        model=model_deployment,
        instructions="""
        You are a travel assistant that provides information on travel services available from Margie's Travel.
        Answer questions about services offered by Margie's Travel using the provided travel brochures.
        Search the web for general information about destinations or current travel advice.
        """,
        input=input_text,
        previous_response_id=last_response_id,
        tools=[
            {
                "type": "file_search",
                "vector_store_ids": [vector_store.id]
            },
            {
                "type": "web_search"
            }
        ]
   )
   print(response.output_text)
   last_response_id = response.id
   ```

    このコードはプロンプトを送信し、*file_search* ツールをベクトル ストア*web_search* ツールを、一般的な Web 検索を実行したりできることを指定します。

1. コード ファイルに加えた変更を保存します。 次に、ターミナル ウィンドウで、次のコマンドを使用して Azure にサインインします。

    ```powershell
    az login
    ```

    > **注**: ほとんどのシナリオでは、*az ログイン*を使用するだけで十分です。 ただし、複数のテナントにサブスクリプションがある場合は、*[--tenant]* パラメーターを使用してテナントを指定する必要があります。 詳細については、「[Azure CLI を使用して対話形式で Azure にサインインする](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)」を参照してください。

1. 次のコマンドを入力してアプリケーションを実行します。

    ```powershell
   python tools-app.py
   ```

    プログラムはターミナルで実行されている必要があります (そうでない場合は、エラーを解決してからもう一度お試しください)。

1. ダイアログが表示されたら、`来月、サンフランシスコではどんなイベントがあるのでしょうか？` を入力し、生成 AI モデルからの応答を確認します。

    応答には、*web_search* ツールを使用して取得した情報が含まれています。

1. このフォローアップの質問をしてみましょう。「`マーギーズ・トラベルでは、その地域でどのようなホテルを取り扱っていますか？`」。

    応答には、*file_search* ツールを使用して取得した情報が含まれています。

1. 終了したら、`quit` を入力してプログラムを終了します。
