# `.panda.json` Data Format Specification

`.panda.json` is the JSON file onPanda uses to save and exchange annotation data. A file can store multiple conversations for the same task, together with evaluations of those conversations. Each conversation is called a **dialog**.

## Start with a minimal example

The following is annotation data that can be imported directly into onPanda. Save it as a UTF-8 encoded `example.panda.json` file:

```json
{
  "version": "2.0",
  "dialogs": {
    "1": {
      "messages": [
        { "role": "user", "content": "What is 2 + 3?" },
        { "role": "assistant", "content": "5", "finish_reason": "stop" }
      ],
      "annotate": { "is_good": null }
    }
  }
}
```

For a wide variety of practical examples, see the [`panda_json` directory in the on-panda-example-data repository](https://github.com/on-panda/on-panda-example-data/tree/main/panda_json), which contains many `.panda.json` data examples.

Start by learning these fields:

- **`dialogs`**: The collection of conversations, represented as an object. Keys are positive integer strings such as `"1"` and `"2"`, sorted numerically from 1; gaps are allowed.
- **`messages`**: The complete conversation for the dialog, ordered by time and containing at least one message. Append additional turns directly to the array; each dialog stores its own complete context.
- **`annotate.is_good`**: The evaluation of this conversation. Its value is `true`, `false`, or `null`.
- **`version`**: The file format version. New data should currently use `"2.0"`.

When preparing data manually, only `dialogs` and each dialog's `messages` are required. The annotation tool fills in other default fields and records the update time when saving. Data can also contain only a question and end with a `user` message, so that the model can continue generating.

## Annotation results

### `annotate.is_good`: whether the conversation is satisfactory

| Value | Meaning |
| --- | --- |
| `true` | Explicitly marked as satisfactory; it can be used as a positive example |
| `false` | Explicitly marked as unsatisfactory |
| `null` | No explicit choice yet; use the default judgment |

The default rule is: **the dialog with the largest number in `dialogs` is considered satisfactory, and all others are considered unsatisfactory**. This rule applies only to `null`; it does not override an explicit `true` or `false`. Multiple dialogs can be marked `true` at the same time. To make an evaluation fixed, write the Boolean value explicitly.

## `messages` and Chat Completions

`messages` follows the [OpenAI Chat Completions message format](https://platform.openai.com/docs/api-reference/chat/create). Fill in `role`, `content`, tool calls, multimodal content, and other fields according to that format; use the API documentation as the source of truth for which roles and fields are supported.

onPanda uses one convention for reasoning: **persist it as `reasoning`, never as `reasoning_content`**. Special markers such as `<think>` in a model protocol are handled by the response template; do not put them into `content`. When importing older data, `reasoning_content` is normalized to `reasoning`.

Pay particular attention to `finish_reason`. It records the actual reason an assistant reply ended. Common values include:

- `stop`: Normal completion;
- `tool_calls`: The output contains tool calls;
- `length`: A length limit was reached;
- `reasoning_end`: The reasoning section ended, and the main response may continue;
- While generation is still in progress, the field may be empty or omitted.

## `tool_configs`: configuring executable tools

`tool_configs` is a private onPanda configuration stored inside a dialog. It describes where tools are loaded from and how they are called. It is not a Chat Completions message field; the standard tool definitions actually sent to the model are stored in the sibling `tools` field.

The most common configuration is MCP:

```json
{
  "tool_configs": [
    {
      "type": "mcp",
      "server_url": "http://127.0.0.1:9330/mcp",
      "server_label": "local-harness",
      "require_approval": "always"
    }
  ]
}
```

- Set `type` to `"mcp"`.
- Set `server_url` to the MCP Server's Streamable HTTP endpoint.
- `server_label` is an optional human-readable name that distinguishes multiple MCP Servers.
- `require_approval` controls whether a person must approve a tool call before it runs. Common values are `"always"` and `"never"`; when omitted, it defaults to `"never"`.

When loading an MCP configuration, onPanda connects to `server_url`, reads the Server's instructions and tools, converts them into Chat Completions tool definitions, and adds the instructions to the system prompt. Tool calls are forwarded to the Server over the same MCP connection. `dialog.tools` can store the parsed tool definitions; when only `tool_configs` is present, onPanda parses them automatically while loading the dialog or before making a request.

The built-in browser Agent example uses `local-fetch://browser-agent-mcp`. This is a local address registered inside the application, not a general-purpose remote URL. Other local MCP services should use the HTTP address they actually expose, such as `http://127.0.0.1:9300/mcp`. On the page, click `🤖 browser-agent` or `codex/cc` under **Examples** to see MCP configuration and tool calls in action.

`tool_configs` can also define a static function tool directly:

```json
{
  "tool_configs": [
    {
      "type": "function",
      "function": {
        "name": "get_weather",
        "description": "Get the weather",
        "parameters": {
          "type": "object",
          "properties": { "city": { "type": "string" } },
          "required": ["city"]
        }
      }
    }
  ]
}
```

## `annotate.customs`: custom annotation items

Before you begin, click the `annotate` example at the top of the onPanda web app, then inspect and try the annotation panel to see how custom annotation items work. `annotate.customs` is an array of custom annotation items inside a dialog. It stores both what the annotation panel looks like and the resulting annotation data, making it suitable for task-specific fields in different datasets. Each item must have at least `name` and `type`; the field that stores its value depends on the type:

| `type` | Value field | Use |
| --- | --- | --- |
| `checkbox` | `checkbox` | Yes/no or three-state selection; the initial value can be `null` |
| `single_choice` | `single_choice` | Single selection; options use `{ "k": "option name", "v": null }` |
| `multiple_choice` | `multiple_choice` | Multiple selection; options use the same structure, with `v` storing whether each option is selected |
| `text` | `text` | Single-line or multiline text |
| `markdown` | `markdown` | Read-only instructions or annotation hints |

The following example shows several commonly used controls:

```json
{
  "annotate": {
    "is_good": null,
    "customs": [
      {
        "name": "Annotation status",
        "type": "single_choice",
        "single_choice": [
          { "k": "Complete", "v": null },
          { "k": "Skip", "v": null },
          { "k": "Needs review", "v": null }
        ],
        "tips": "Confirm whether this data can be submitted"
      },
      {
        "name": "Issue type",
        "type": "multiple_choice",
        "multiple_choice": [
          { "k": "Factual error", "v": null },
          { "k": "Formatting issue", "v": null },
          { "k": "Tool-use error", "v": null }
        ],
        "required": true
      },
      {
        "name": "Notes",
        "type": "text",
        "text": "",
        "tips": "Record anything that needs to be reviewed"
      }
    ]
  }
}
```

`tips` is shown as a hint, `required: true` means the item must be filled in, and `disabled: true` makes it display-only. An option's `v` can remain `null`, which distinguishes “not selected yet” from an explicit `true` or `false`. Open the onPanda page, click `annotate` under **Examples**, and open the annotation panel to try these controls.

## Other optional fields

| Location | Field | Use |
| --- | --- | --- |
| Top level of the file | `uuid` | A string identifying the entire data file; onPanda generates one when it is not provided |
| Top level of the file | `update_time` | The Unix timestamp in milliseconds of the most recent save; it can be `null` before the first save |
| Top level of the file | `title`, `description`, `comment` | The title, instructions for annotators, and an editable note |
| Top level of the file | `deleted_dialogs` | An object containing deleted dialogs, with the same structure as `dialogs`, used for the recycle bin |
| Top level of the file | `hash_map` | An object that stores repeated referenced content in one place to reduce file size |
| Top level of the file | `cache_tree` | An optional cache of tokens, probabilities, and candidate information |
| Inside a dialog | `tools` | Parsed Chat Completions tool definitions, usually generated from `tool_configs` |

Values in `hash_map` can be strings, arrays, or objects. Long text, tool definitions, and other content in a dialog may be replaced by references such as `<|hash|>…`: remove the prefix, and the remaining Base64 digest is the key in `hash_map`. Resolve these references before processing messages when reading the file. Handwritten data can keep everything inline and omit `hash_map`.

After decompression, `cache_tree` is organized by dialog number, and `cache_tree[number].tokens` stores probability tokens. onPanda currently exports it as `{"compressed_type": "lz-string-base64", "compressed": "…"}`; use LZString to decompress it and then parse the JSON. **The cache can be omitted or deleted as a whole; the complete conversation is still stored in `messages`.** Downstream training should recalculate tokenization and correction positions from the messages with the target tokenizer; see [Core Design](core_design.md) for the responsibilities of the three token classes.
