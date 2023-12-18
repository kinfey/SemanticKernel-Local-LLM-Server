# Semantic Kernel .NET 开发者离线大模型本地服务器

*支持 Semantic-Kernel 1.0.1*


或者您正在使用 Semantic Kernel 的 [Hugging Face http 服务器](https://github.com/microsoft/semantic-kernel/tree/3451a4ebbc9db0d049f48804c12791c681a326cb/samples/apps/hugging-face-http-server) 作为本地 LLM 服务，但是基于 Hugging Face 在中国大陆的访问受限以及企业管理方面的原因，我尝试重构这个项目，直接去掉对 Hugging Face 的依赖，针对macOS和Linux环境进行了适配。

现阶段，ChatCompletion和Embedding的实现就已经完成了。

**ChatCompletion** 适用大模型包括 *baichu2*、*ChatGLM3*、*Microsoft-phi 1.5*

**Embeddings** 适应大模型包括 *jina-embeddings*（英语嵌入模型），*text2vec-large-chinese*（中文嵌入模型）,*bge-\*-v1.5*(中文/英语嵌入模型)



**使用方法**

0. 你需要先下载你的模型以及设置你的 Python 环境


```bash

pip install -r requirement.txt

```

1. 通过 .env 配置你的 ChatCompletion and Embedding 模型在本地的位置

```txt

CHAT_COMPLETEION_URL = 'Your chat completion model location'
EMBEDDING_URL = 'Your embeddings model location'

```

2. 启动本地 LLM 服务

```bash

python local_llm_service.py

```

3. 添加 Microsoft.SemanticKernel, Microsoft.SemanticKernel.Connectors.AI.HuggingFace, Microsoft.SemanticKernel.Connectors.Memory.Qdrant(你可以选择你喜欢的向量数据库) packages 

4. 初始化 chatcompletion, embeddings, and qdrant 的 Endpoint


```csharp

string chat_endpoint = "http://localhost:5002/v1/chat/completions";
string embeddings_endpoint = "http://localhost:5002/v1/embeddings";
string qdrant_endpoint = "http://localhost:6333";


```


5. 例子一 - ChatCompletion


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

6. 例子2 - Embeddings


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

如果你希望测试英语版本的离线 text embedding and chat completion ,  [点击这里](./samples/dotnet_notebook_en.ipynb)

🍔🍔🍔🍔🍔🍔🍔🍔🍔 期待更多功能






