[English](README.md) | **简体中文**

# SemanticKernel.DashScope

让 DashScope 与 Semantic Kernel 和 Kernel Memory 协同工作。

## 开始使用 Semantic Kernel

将 NuGet 包添加到项目中。

```shell
dotnet add package Cnblogs.SemanticKernel.Connectors.DashScope
```

```cs
using Microsoft.SemanticKernel;

var builder = Kernel.CreateBuilder();
builder.Services.AddDashScopeChatCompletion("your-api-key", "qwen-max");
var kernel = builder.Build();

var prompt = "<message role=\"user\">Tell me about the Cnblogs</message>";
var response = await kernel.InvokePromptAsync(prompt);
Console.WriteLine(response);
```

## 在 ASP.NET Core 中使用 Kernel Memory

安装 NuGet 包 `Cnblogs.KernelMemory.AI.DashScope`

安装 NuGet 包 `Microsoft.KernelMemory.Core`

安装 NuGet 包 `Microsoft.KernelMemory.SemanticKernelPlugin`

`appsettings.json`

```json
{
  "dashScope": {
    "apiKey": "your-key",
    "chatCompletionModelId": "qwen-max",
    "textEmbeddingModelId": "text-embedding-v2"
  }
}
```

`Program.cs`

```csharp
// Kernel Memory 配置
var memory = new KernelMemoryBuilder(builder.Services).WithDashScope(builder.Configuration).Build();
builder.Services.AddSingleton(memory);

// Semantic Kernel 配置
builder.Services.AddDashScopeChatCompletion(builder.Configuration);
builder.Services.AddSingleton(
    sp =>
    {
        var plugins = new KernelPluginCollection();
        plugins.AddFromObject(
            new MemoryPlugin(sp.GetRequiredService<IKernelMemory>(), waitForIngestionToComplete: true),
            "memory");
        return new Kernel(sp, plugins);
    });
```

服务

```csharp
public class YourService(Kernel kernel, IKernelMemory memory)
{
    public async Task<string> GetCompletionAsync(string prompt)
    {
        var chatResult = await kernel.InvokePromptAsync(prompt);
        return chatResult.ToString();
    }

    public async Task ImportDocumentAsync(string filePath, string documentId)
    {
        await memory.ImportDocumentAsync(filePath, documentId);
    }

    public async Task<string> AskMemoryAsync(string question)
    {
        // 使用 memory.ask 查询 Kernel Memory
        var skPrompt = """
                       Question to Kernel Memory: {{$input}}

                       Kernel Memory Answer: {{memory.ask $input}}

                       If the answer is empty say 'I don't know' otherwise reply with a preview of the answer, truncated to 15 words.
                       """;

        // 可将创建的函数放入单例服务以便复用
        var myFunction = kernel.CreateFunctionFromPrompt(skPrompt);
        var result = await myFunction.InvokeAsync(question);
        return result.ToString();
    }
}
```
