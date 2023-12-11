# Local LLM Server for Semantic Kernel .NET Developer

*如果你使用中文，请[访问这里](README.zh-cn.md)*

*Support Semantic-Kernel RC.3*

Or you are using Semantic Kernel's [Hugging Face http server](https://github.com/microsoft/semantic-kernel/tree/3451a4ebbc9db0d049f48804c12791c681a326cb/samples/apps/hugging-face-http-server) as your local LLM service  , but based on the inaccessibility of hugging face in mainland China and management reasons, I tried to reconstruct the project. At this stage, it is adapted for macOS and Linux environments.

At this stage, the implementation of ChatCompletion and Embedding has been completed.

**ChatCompletion** is adapted to  LLM  *baichu2*, *ChatGLM3*, *Microsoft-phi 1.5*

**Embeddings** adapted to LLM *jina-embeddings* (English Embedding model), *text2vec-large-chinese* (Chiense Embedding model)

**Samples**


0. download your LLM firstly and using pip to install python library


```bash

pip install -r requirement.txt

```

1. .env config your ChatCompletion and Embedding model location

```txt

CHAT_COMPLETION_URL = 'Your chat completion model location'
EMBEDDING_URL = 'Your embeddings model location'

```

2. Start your Local LLM Http Server

```bash

python local_llm_service.py

```

3. Add Microsoft.SemanticKernel, Microsoft.SemanticKernel.Connectors.AI.HuggingFace, Microsoft.SemanticKernel.Connectors.Memory.Qdrant(You can choose different vector database) packages 

4. Initialization endpoint for chatcompletion, embeddings, and qdrant 


```csharp

string chat_endpoint = "http://localhost:5002/v1/chat/completions";
string embeddings_endpoint = "http://localhost:5002/v1/embeddings";
string qdrant_endpoint = "http://localhost:6333";


```


5. Sample 1 - ChatCompletion


```csharp

using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Connectors.Memory.Qdrant;
using Microsoft.SemanticKernel.Plugins.Memory;
using Microsoft.SemanticKernel.Connectors.AI.HuggingFace.TextEmbedding;

#pragma warning disable SKEXP0020

Kernel kernel = new KernelBuilder()
            .AddHuggingFaceTextGeneration(
                model: "baichuan2",
                endpoint: chat_endpoint)
            .Build();

var questionAnswerFunction = kernel.CreateFunctionFromPrompt("问: {{$input}} 答:");

var result = await kernel.InvokeAsync(questionAnswerFunction, new("介绍一下自己"));

result.GetValue<string>()


```

6. Sample 2 - Embeddings


```csharp

#pragma warning disable SKEXP0052
#pragma warning disable CS1061
#pragma warning disable SKEXP0011
#pragma warning disable SKEXP0026

#pragma warning disable SKEXP0020

var qdrantMemoryBuilder = new MemoryBuilder();

var hfembeddings = new HuggingFaceTextEmbeddingGeneration("text2veccn", embeddings_endpoint);

qdrantMemoryBuilder.WithTextEmbeddingGeneration(hfembeddings);
qdrantMemoryBuilder.WithQdrantMemoryStore(qdrant_endpoint, 1024);

var builder = qdrantMemoryBuilder.Build();

string MemoryCollectionName = "text2vecdemo";


await builder.SaveInformationAsync(MemoryCollectionName, id: "id1", text: "我是卢建晖");
await builder.SaveInformationAsync(MemoryCollectionName, id: "id2", text: "卢建晖是微软云技术布道师");
await builder.SaveInformationAsync(MemoryCollectionName, id: "id3", text: "卢建晖从 2012 年到 2020 年是微软最有价值专家");
await builder.SaveInformationAsync(MemoryCollectionName, id: "id4", text: "卢建晖是人工智能讲师");

var searchResults =  builder.SearchAsync(MemoryCollectionName, "你认识卢建晖吗", limit: 3, minRelevanceScore: 0.6);

await foreach (var item in searchResults)
{
    Console.WriteLine(item.Metadata.Text + " : " + item.Relevance);
}


```

If you want to get English text embedding and chat completion , please click [here](./samples/dotnet_notebook_en.ipynb)

🍔🍔🍔🍔🍔🍔🍔🍔🍔 More functions will be added in the future 






