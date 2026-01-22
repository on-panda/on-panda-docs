# 使用 onPanda 做体感对战

对于每个实验需要保存为如下的 `*.arena.json` 格式。只需提供两个 `*.arena.json` 就能组建体感对战
## `arena.json` 规范：
- "data" 内部以 dict 的形式存储每条包含完整问答的 openAI messages
    - 两个 `*.arena.json` 组建对战时，只有 key 一致的 messages 才会对战
- `messages[-1]` 需要包含 `["arena"]["exp_name"]` 以表明当前这个 `*.arena.json` 的实验名
    - `messages[-1]` 可选添加 `display_info`，来展示在对战页面。

```JSON
{
  "data": {
    "prompt_key1": [
      {
        "role": "system",
        "content": "You are a helpful assistant."
      },
      {
        "role": "user",
        "content": "I want to learn about investing"
      },
      {
        "role": "user",
        "content": [
          {
            "type": "input_audio",
            "input_audio": {
              "data": "UklGRqQ....AAB3AQ",
              "format": "wav"
            }
          }
        ],
        "comment": "ok i'm more focused on long term gains."
      },
      {
        "role": "assistant",
        "content": "That's great! For long-term investments, you can consider diversifying your portfolio with a mix of stocks, bonds, and real estate. This helps balance risk and potential returns. Index funds or bond funds are also low-risk options. Remember to review and adjust your investment plan regularly to stay aligned with your goals.",
        "arena": {
          "exp_name": "model-name-sft-250611"
        },
        "description": "String to display on UI"
      }
    ]
  }
}
```
