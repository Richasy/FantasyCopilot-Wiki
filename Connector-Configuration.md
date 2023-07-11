This article aims to explain the configuration structure of Custom Connector, and you can find Python examples of `Chat`, `Text Completion` and `Embedding` in [Custom connector examples](https://github.com/Richasy/FantasyCopilot/tree/main/examples/custom-connector).

## Data Structures of Messages and Responses

For how to use these data structures, please refer to [[Custom Connector Overview]]

```python
from pydantic import BaseModel

class Message(BaseModel):
    role: str
    content: str

class RequestSettings(BaseModel):
    temperature: float
    maxResponseTokens: int
    topP: float
    frequencyPenalty: float
    presencePenalty: float

class CopilotChatRequest(BaseModel):
    message: str
    history: List[Message]
    settings: RequestSettings

class CopilotTextRequest(BaseModel):
    message: str
    settings: RequestSettings

class CopilotMessageResponse(BaseModel):
    content: str
    isError: bool
```

## Connector Properties

Here are some definitions of connectors that determine how they are displayed, categorized, and launched.

|Property name|Required?|Description|Example value|
|-|-|-|-|
|id|Y|Identifier of the connector, it should be unique|com.richasy.connector|
|name|Y|Name of the connector, it will be displayed on the UI|My Connector|
|base_url|Y|API service address, must be http/https protocol|http://localhost:8000|
|config_path|N|If the connector requires manual configuration by the user, such as adding keys, etc., the relative path of the configuration file can be written here|config.json|
|readme|N|If there is a description file, please fill in this field. After the user imports, the application will automatically open the description file|README.txt|
|execute_name|N|If the API service entry is an executable file, please fill in this field|api.exe|
|script_file|N|If the API service needs to be started by running a script, please fill in this field. The application will run this script through powershell|run.ps1|
|script_command|N|If the API service needs to be started by a command, please fill in this field. The application will execute this command through powershell|python api.py|
|features|Y|List of Features, please refer to [Feature](#Feature)|[{...}, {...}]|

> **NOTE**  
> `execute_name`, `script_file` and `script_command` are the main ways to start the API service, and a connector can only choose one of them.  
> If your API service is deployed in the cloud and the data structure meets the expectations of Fantasy Copilot, you can omit these fields and fill in your deployment URL in `base_url`.

## Feature

Feature defines the features supported by the connector.

|Property name|Required?|Description|Example value|
|-|-|-|-|
|type|Y|Feature type, currently supports `chat`, `text-completion` and `embedding`| chat|
|endpoints|Y|Defined API endpoints list, see [Endpoint](#Endpoint) for specific data structure|[{...}, {...}]|

### About Feature Type

- `chat`: The connector supports conversation. It can be used in `session` scenarios.
- `text-completion`: The connector supports text completion. It is also applicable to the use scenarios of `chat`, but if you need to use `knowledge base` or `semantic skills`, you must add `text completion model`.
- `embedding`: The connector supports text vectorization (embedding). If you need to use knowledge base, you must add a connector that supports embedding.

## Endpoint

Endpoint refers to our API endpoints.

|Property name|Required?|Description|Example value|
|-|-|-|-|
|type|Y|Defines the endpoint type, see [Endpoint type](#Endpoint-type) for supported endpoint types|chat-rest|
|path|Y|Endpoint path relative to `base_url`|`/chat`|

Finally, the application will splice `base_url` and `path` according to the current settings to form a complete url such as `http://localhost:8000/chat`.

### Endpoint type

|Type|Belonging feature|Description|
|-|-|-|
|`chat-rest`|`chat`|It represents a chat POST request, and the service will return a complete text response|
|`chat-stream`|`chat`|The service will generate and return one character at a time in the form of SSE (Server-Send Events)|
|`chat-stream-cancel`|`chat`|Inform the service to stop executing `chat-stream`|
|`text-completion-rest`|`text-completion`|The service will return a complete response after text completion|
|`text-completion-stream`|`text-completion`|The service will generate and return one character at a time in the form of SSE (Server-Send Events)|
|`text-completion-cancel`|`text-completion`|Inform the service to stop executing `text-completion-stream`|
|`embedding-rest`|`embedding`|Input text and generate vectors, then return the result|

## Full Configuration

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