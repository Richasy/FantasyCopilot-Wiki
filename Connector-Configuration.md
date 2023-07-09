本文旨在说明 Custom Connector 的配置结构，你可以在 [Custom connector examples](https://github.com/Richasy/FantasyCopilot/tree/main/examples/custom-connector) 中找到 `Chat`, `Text Completion` 和 `Embedding` 的Python示例。

## 连接器属性

这里是一些连接器的定义，它们决定连接器如何被显示、分类以及启动。

|Property name|Required?|Description|Example value|
|-|-|-|-|
|id|Y|连接器标识符，它应该是唯一的|com.richasy.connector|
|name|Y|连接器名称，它会被显示在UI上|My Connector|
|base_url|Y|API服务地址，必须是http/https协议|http://localhost:8000|
|config_path|N|如果连接器需要用户手动配置，比如添加密钥等，可以在这里写上配置文件的相对路径|config.json|
|readme|N|如果有说明文件，请填写此字段，用户导入后，应用会自动打开说明文件|README.txt|
|execute_name|N|如果api服务入口是一个可执行文件，请填写此字段|api.exe|
|script_file|N|如果api服务需要通过运行脚本来启动，请填写此字段，应用会通过powershell运行此脚本|run.ps1|
|script_command|N|如果api服务需要通过命令来启动，请填写此字段，应用会通过powershell执行此命令|python api.py|
|features|Y|Feature列表，请参见 [Feature](#Feature)|[{...}, {...}]|

> **NOTE**  
> `execute_name`, `script_file` 和 `script_command` 是 API 服务的主要启动方式，一个连接器只能选择其中一种启动方式。  
> 如果你的 API 服务部署在云上，且数据结构符合 Fantasy Copilot 的预期，那么可以省略这几个字段，将你的部署网址填入 `base_url` 即可。

## Feature

Feature 定义了连接器支持的功能。

|Property name|Required?|Description|Example value|
|-|-|-|-|
|type|Y|功能类型，目前支持 `chat`, `text-completion` 和 `embedding`| chat|
|endpoints|Y|定义的 API 终结点列表，其具体数据结构参见 [Endpoint](#Endpoint)|[{...}, {...}]|

### 关于功能类型

- `chat`：该连接器支持对话。它可以被用在 `会话` 场景。
- `text-completion`：该连接器支持文本完成。它同样适用于 `chat` 的使用场景，但是如果你需要使用 `知识库` 或者 `语义技能` 等功能，你必须添加 `文本完成模型`。
- `embedding`: 该连接器支持文本向量化（嵌入）。如果你需要使用知识库，那么你必须要添加支持 embedding 的连接器。

## Endpoint

Endpoint指的是我们的 API 端点。

|Property name|Required?|Description|Example value|
|-|-|-|-|
|type|Y|定义终结点类型，关于支持的终结点类型，请查看 [Endpoint type](#Endpoint-type)|chat-rest|
|path|Y|终结点相对于 `base_url` 的路径|`/chat`|

最终，应用会根据当前设置，将 `base_url` 和 `path` 进行拼接，形成如 `http://localhost:8000/chat` 这样的完整 url。

### Endpoint type

|Type|所属功能|描述|
|-|-|-|
|`chat-rest`|`chat`|它表示一次聊天 POST 请求，服务会返回完整的文本响应|
|`chat-stream`|`chat`|服务会以SSE（Server-Send Events）的方式逐字生成并返回|
|`chat-stream-cancel`|`chat`|告知服务停止执行 `chat-stream`|
|`text-completion-rest`|`text-completion`|服务会在文本完成之后返回完整响应|
|`text-completion-stream`|`text-completion`|服务会以SSE（Server-Send Events）的方式逐字生成并返回|
|`text-completion-cancel`|`text-completion`|告知服务停止执行 `text-completion-stream`|
|`embedding-rest`|`embedding`|传入文本，并生成向量，然后返回结果|

## 完整配置

```json
{
    "id": "com.richasy.connector",
    "name": "My Connector",
    "base_url": "http://127.0.0.1:8000",
    "config_path": "config.json",
    "readme": "README.txt",
    "script_file": "If needed",
    "script_command": "If needed",
    "execute_name": "If needed",  
    "features":[
        {
            "type": "chat",
            "endpoints":[
                {
                    "type": "chat-rest",
                    "path": "/chat"
                },
                {
                    "type": "chat-stream",
                    "path": "/chat-stream"
                },
                {
                    "type": "chat-stream-cancel",
                    "path": "/chat-stop"
                }
            ]
        },
        {
            "type": "text-completion",
            "endpoints":[
                {
                    "type": "text-completion-rest",
                    "path": "/text"
                },
                {
                    "type": "text-completion-stream",
                    "path": "/text-stream"
                },
                {
                    "type": "text-completion-stream-cancel",
                    "path": "/text-stop"
                }
            ]
        },
        {
            "type": "embedding",
            "endpoints": [
                {
                    "type": "embedding-rest",
                    "path": "/embedding"
                }
            ]
        }
    ]
}
```