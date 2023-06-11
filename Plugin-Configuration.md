When creating a plugin, there is only one `config.json` file, which contains information about the plugin, the invocation methods of all commands, and an i18n module for localization.

In this article, we will introduce the main parts of the configuration file in three modules.

## Package Properties

We need some properties to define the plugin you create, which determines how your package will be presented to users. Here is the property table:

| Property Name | Required? | Description | Example Value |
| --- | --- | --- | --- |
| schema_version | Y | Indicates the version of the parser currently referenced by the configuration, which can only be set to `1` | 1 |
| package_name | Y | Plugin name (supports localization) | title or {{$title}} |
| package_desc | Y | Plugin description (supports localization) | description or {{$description}} |
| package_id | Y | The ID of the plugin, which should be a unique value | com.richasy.test |
| version | Y | Plugin version | 1.0.0 |
| author | N | Plugin author name | someone |
| author_site | N | Personal website of the plugin author | https://example.com |
| repository | N | Open source address or plugin website of the plugin | https://example.com |
| logo_url | N | The URL of the plugin logo. If not filled in, the default icon will be displayed in the application | https://logoimage.xx |

> It should be noted that the application prefers open source plugins. If no open source address is provided, the application will remind users that the plugin is closed source and should be used with caution.

## Commands

All commands need to be written under the `commands` attribute, and the specific structure can be found in the [Configuration Overview](#configuration-overview).

### Basic Properties of Commands

| Property Name | Required? | Description | Example Value |
| --- | --- | --- | --- |
| command_name | Y | Command display name (supports localization) | name or {{$name}} |
| command_desc | Y | Command description, preferably within 20 words, clear and concise, which helps to plan tasks reasonably (supports localization) | description or {{$description}} |
| command_id | Y | Command ID, which requires a GUID and must be unique | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx |
| execute_name | Y | Executable file name (or path) | plugin.exe |
| category | N | Category, please refer to [Command Categories](#command-categories) for specific command categories | text |
| only_final_output | N | The application will read the text you print on the console as the output of the command. If you only need the application to read the text of the last output, please set it to `true` | false |
| config_set | N | Configuration items displayed on the UI when the user configures the command | Please refer to [Command Configuration](#command-configuration) |
| parameters | N | List of parameters required to execute the command | Please refer to [Command Parameters](#command-parameters) |
| output | Y | The output method of the command | Please refer to [Command Output](#command-output) |

### Command Categories

|Value|Description|
|-|-|
|text|Commands related to text operations, such as converting case, translating, etc.|
|file|Commands related to file operations, such as reading/writing files.|
|web|Commands related to web operations, such as opening web pages/reading web content.|
|image|Commands related to image operations, such as generating images/cropping.|
|video|Commands related to video operations, such as generating videos.|
|voice|Commands related to audio operations, such as speech recognition/text-to-speech.|
|native|Commands related to native operations, such as displaying dialogs/notifications.|
|accessibility|Commands related to accessibility, such as screen reading.|
|tool|Commands with broad meanings, usually referring to composite commands.|
|knowledge|Commands related to knowledge bases.|
|other|Uncategorized commands.|

### Command Configuration

If your command supports multiple invocation methods and requires users to select one when calling the command, or if it requires some manual input, you can add command configurations.

| Property Name | Required? | Description | Example Value |
| - | - | - | - |
| type | Y | The type of configuration, currently supports two types, `input` and `option` | `input` |
| variable_name | Y | The value of the configuration will be stored in the workflow context, and you need to provide a variable name to associate this value | xxx |
| default_value | N | The default value that is displayed on the UI when the user configures the command | xxx |
| title | N | The title of the configuration item (supports localization) | xxx |
| options | N | Available when `type` is `option`, you can provide a set of alternative values for users to choose from | Please refer to [Command Configuration Options](#command-configuration-options) |

#### Command Configuration Options

| Property Name | Required? | Description | Example Value |
| - | - | - | - |
| id | Y | Identifier for the option, which will be stored as the corresponding value in the workflow context | xx |
| display_name | Y | Text displayed for the option in the UI, supports localization | name or {{$name}} |

### Command Parameters

| Property Name | Required? | Description | Example Value |
| --- | --- | --- | --- |
| id | Y | The variable name in the workflow context | xx |
| name | Y | The parameter name in the command line | xx |
| description | N | The description of the parameter, which will inform the user of its purpose (supports localization) | xx |
| required | N | If the parameter is required, set it to `true`. If the parameter is missing, the workflow will be interrupted and an error will be reported. | `false` |

You may be confused about why we need both `id` and `name`.

In fact, when designing plugin commands, the parameters we provide are usually generic words, such as `-type`, `-version`, and so on.

However, as the workflow context is a key-value pair, if multiple commands attempt to write to the same key, we cannot guarantee that the value will be reliable.

Therefore, when writing to the context, we usually give a specific key, such as `SPECIFIC_PREFIX_TYPE`, and then convert it to the command line parameter we preset when calling the command.

### Command Output

| Property Name | Required? | Description | Example Value |
| - | - | - | - |
| type | Y | The type of output, currently supports `plain` and `json` | xxxx |
| output_key | N | Valid when the output type is `json`. It represents which key of the JSON is passed as the final output to the next command, currently only supports direct sub-levels | childKey |
| context_items | N | Valid when the output type is `json`. It represents which values are stored in the current workflow context, currently only supports direct sub-levels | Please refer to [Context Item Storage](#context-item-storage) |

#### Context Item Storage

| Property Name | Required? | Description | Example |
| --- | --- | --- | --- |
| key | Y | Corresponding key in the JSON, only supports direct children | childKey |
| variable_name | Y | The key name that needs to be stored in the context | name |
| override | N | Whether to overwrite when there is a key-value pair with the same name in the context | false |

## Localization

To ensure that your plugin performs well in various languages, I recommend that developers provide localization resources.

All localized text resources need to be written under the `resources` property.

```json
{
    ...
    "resources": {
        "en": {
            "key": "Value"
        },
        "zh": {
            "key": "值"
        }
    }
}
```

Language names should comply with the RFC-4646 standard. When you define a resource for a language, please remember to define a resource with the same name in other languages you support.

You can use **{{$key}}** as a placeholder in supported attributes, replacing `key` with the name of the resource you defined. The application will determine how to display the command based on the current UI language.

If the language currently used by the user is not in your list of supported languages, the application will display the first supported language.

## Configuration Overview

```json
{
  "schema_version": 1,
  "package_name": "{{$packageName}}",
  "package_desc": "Just description",
  "package_id": "com.richasy.test",
  "version": "1.0.0",
  "author": "",
  "author_site": "",
  "repository": "https://github.com",
  "logo_url": "",
  "commands": [
    {
      "command_name": "test",
      "command_desc": "just a test method.",
      "command_id": "guid",
      "category": "text",
      "only_final_output": true,
      "config_set": [
        {
          "type": "input",
          "variable_name": "",
          "default_value": "",
          "title": ""
        },
        {
          "type": "option",
          "variable_name": "",
          "default_value": "",
          "title": "",
          "options": [
            {
              "id": "",
              "display_name": ""
            }
          ]
        }
      ],
      "parameters": [
        {
          "id": "Input",
          "name": "Input",
          "description": "",
          "required": true
        }
      ],
      "execute_name": "test.exe",
      "output": {
        "type": "json/plain",
        "output_key": "",
        "context_items": [
          {
            "key": "",
            "variable_name": "",
            "override": true
          }
        ]
      }
    }
  ],
  "resources": {
    "en": {
      "packageName": "Test"
    },
    "zh": {
      "packageName": "测试"
    }
  }
}
```
