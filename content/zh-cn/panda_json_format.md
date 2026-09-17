# `.panda.json` 数据格式规范

`.panda.json` 是 onPanda 保存和交换标注数据的 JSON 文件。一个文件可以保存同一任务的多份对话，以及对它们的评价。每份对话称为一个 **dialog**。

## 从一个最小示例开始

下面是一份可直接导入 onPanda 的待标注数据，保存为 UTF-8 编码的 `example.panda.json` 即可：

```json
{
  "version": "2.0",
  "dialogs": {
    "1": {
      "messages": [
        { "role": "user", "content": "2 + 3 等于多少？" },
        { "role": "assistant", "content": "5", "finish_reason": "stop" }
      ],
      "annotate": { "is_good": null }
    }
  }
}
```

先掌握这几个字段：

- **`dialogs`**：对话集合，是一个对象。编号使用 `"1"`、`"2"` 这样的正整数字符串，按数值排序，从 1 开始，允许中间有空缺。
- **`messages`**：该 dialog 的完整对话，按时间顺序排列，至少包含一条消息。多轮对话直接往数组中追加；不同 dialog 各自保存完整上下文。
- **`annotate.is_good`**：对这份对话的评价，取 `true`、`false` 或 `null`。
- **`version`**：文件格式版本，当前新数据建议填写 `"2.0"`。

手动准备数据时，必需的是 `dialogs` 和每个 dialog 的 `messages`。标注工具会补齐其他默认字段，并在保存时记录更新时间。数据也可以只有问题、以 `user` 消息结尾，供模型继续生成。

## 标注结果

### `annotate.is_good`：对话是否满意

| 值 | 含义 |
| --- | --- |
| `true` | 明确标为满意，可作为正样本 |
| `false` | 明确标为不满意 |
| `null` | 尚未明确选择，使用默认判断 |

默认规则是：**`dialogs` 中编号最大的 dialog 默认满意，其余默认不满意**。这条规则只作用于 `null`，不会覆盖显式的 `true` 或 `false`；可以同时将多份对话标为 `true`。如果希望评价固定下来，就显式填写布尔值。

## `messages` 和 Chat Completions

`messages` 沿用 [OpenAI Chat Completions 的消息格式](https://platform.openai.com/docs/api-reference/chat/create)。`role`、`content`、工具调用、多模态内容等字段都按该格式填写；需要哪些角色和字段，直接以 API 文档为准。

onPanda 对思考过程有一个统一约定：**持久化时使用 `reasoning`，不要使用 `reasoning_content`**。模型协议中的 `<think>` 等特殊标记由 response template 处理，不要把它们混入 `content`。旧数据中的 `reasoning_content` 会在导入时归一到 `reasoning`。

需要额外留意的是 `finish_reason`。它记录 assistant 回复的实际结束原因，常见值包括：

- `stop`：正常结束；
- `tool_calls`：输出工具调用；
- `length`：达到长度限制；
- `reasoning_end`：思考部分结束，后面还可能继续生成正文；
- 尚未结束时可以为空或省略。

## `tool_configs`：配置可执行工具

`tool_configs` 是 onPanda 的私有配置，放在 dialog 内，用来说明工具从哪里加载、如何调用。它不是 Chat Completions 的消息字段；对话实际发送给模型的标准工具定义会放在同级的 `tools` 中。

最常见的是 MCP：

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

- `type` 填 `"mcp"`。
- `server_url` 填 MCP Server 的 Streamable HTTP 地址。
- `server_label` 是可选的人类可读名称，用于区分多个 MCP Server。
- `require_approval` 控制工具调用前是否需要人工批准；常用值是 `"always"` 和 `"never"`，省略时默认为 `"never"`。

加载 MCP 配置时，onPanda 会连接 `server_url`，读取 Server 的 instructions 和 tools，把它们转换成 Chat Completions 的工具定义，并把 instructions 加入 system prompt。工具调用再通过同一个 MCP 连接转发给 Server。`dialog.tools` 可以保存已经解析出的工具定义；只有 `tool_configs` 时，onPanda 也会在加载或发起请求前自动解析。

onPanda 内置的浏览器 Agent 示例使用 `local-fetch://browser-agent-mcp`；这是应用内部注册的本地地址，不是通用的远程 URL。其他本地 MCP 服务应填写它实际提供的 HTTP 地址，例如 `http://127.0.0.1:9300/mcp`。在页面的 **Examples** 中点击 `🤖 browser-agent` 或 `codex/cc`，可以直接感受 MCP 配置和工具调用的效果。

`tool_configs` 也可以直接配置一个静态 function 工具：

```json
{
  "tool_configs": [
    {
      "type": "function",
      "function": {
        "name": "get_weather",
        "description": "查询天气",
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

## `annotate.customs`：自定义标注项

在开始前，先点击 onPanda web app 顶部的 `annotate` example，然后查看并测试标注面板来感受自定义标注项。 `annotate.customs` 是 dialog 内的自定义标注项数组。它把“标注面板长什么样”和“标注结果是什么”一起保存，适合为不同数据集增加任务专属字段。每一项至少有 `name` 和 `type`，取值字段由类型决定：

| `type` | 取值字段 | 用途 |
| --- | --- | --- |
| `checkbox` | `checkbox` | 是/否或三态选择；初始值可以是 `null` |
| `single_choice` | `single_choice` | 单选，选项写成 `{ "k": "选项名", "v": null }` |
| `multiple_choice` | `multiple_choice` | 多选，选项结构同上，`v` 分别保存是否选中 |
| `text` | `text` | 单行或多行文本 |
| `markdown` | `markdown` | 只读说明或标注提示 |

下面的例子同时展示了几种常用控件：

```json
{
  "annotate": {
    "is_good": null,
    "customs": [
      {
        "name": "标注状态",
        "type": "single_choice",
        "single_choice": [
          { "k": "完成", "v": null },
          { "k": "跳过", "v": null },
          { "k": "待确认", "v": null }
        ],
        "tips": "确认这条数据是否可以提交"
      },
      {
        "name": "问题类型",
        "type": "multiple_choice",
        "multiple_choice": [
          { "k": "事实错误", "v": null },
          { "k": "格式问题", "v": null },
          { "k": "工具使用错误", "v": null }
        ],
        "required": true
      },
      {
        "name": "备注",
        "type": "text",
        "text": "",
        "tips": "记录需要复查的地方"
      }
    ]
  }
}
```

`tips` 会显示为提示，`required: true` 表示该项需要填写，`disabled: true` 表示只展示信息而不能编辑。选项的 `v` 可以保留 `null`，这样能区分“还没有选择”和明确的 `true`/`false`。打开 onPanda 页面，在 **Examples** 中点击 `annotate`，再打开标注面板，可以直接看到这类控件的交互效果。

## 其他按需保存的字段

| 位置 | 字段 | 用途 |
| --- | --- | --- |
| 文件顶层 | `uuid` | 字符串，整份数据的标识；未提供时由 onPanda 生成 |
| 文件顶层 | `update_time` | 最近一次保存的 Unix 毫秒时间戳；尚未保存时可为 `null` |
| 文件顶层 | `title`、`description`、`comment` | 标题、给标注者看的说明、可编辑备注 |
| 文件顶层 | `deleted_dialogs` | 已删除对话的对象集合，结构同 `dialogs`，用于回收站 |
| 文件顶层 | `hash_map` | 对象，集中保存被引用的重复内容，减少文件体积 |
| 文件顶层 | `cache_tree` | 可选的 token、概率和候选信息缓存 |
| dialog 内 | `tools` | 已解析的 Chat Completions 工具定义；通常由 `tool_configs` 生成 |

`hash_map` 中的值可以是字符串、数组或对象。对话里的长文本、工具定义等可能被替换为形如 `<|hash|>…` 的引用：去掉前缀后，剩余的 Base64 摘要就是 `hash_map` 的键。读取时先还原这些引用，再处理消息；手写数据可以全部内联并省略 `hash_map`。

`cache_tree` 解压后按 dialog 编号组织，其中 `cache_tree[编号].tokens` 保存概率 token。当前 onPanda 导出为 `{"compressed_type": "lz-string-base64", "compressed": "…"}`，读取时用 LZString 解压再解析 JSON。**缓存可以整体省略或删除，完整对话仍保存在 `messages` 中。** 下游训练的 token 切分和纠正位置应按目标 tokenizer 从消息重新计算；三类 token 的职责见[核心设计](core_design.md)。
