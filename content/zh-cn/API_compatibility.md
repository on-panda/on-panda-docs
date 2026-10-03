# 各个推理工具和 API 对 onPanda 的兼容性


## 支持的 LLM API 协议
Chat Completions 协议是 onPanda 核心支持的 API 协议，尤其以 [vLLM 的 Chat Completions API](https://docs.vllm.ai/en/latest/serving/online_serving/openai_compatible_server/#chat-api) 为准。比如模型续写、prompt_logprobs 等 OpenAI 协议未定义的功能，都是用的 vLLM 定义的接口。

除了 Chat Completions，还可以通过配置 API 的 `api_protocol` 字段，支持 Anthropic 的 [messages API](https://platform.claude.com/docs/en/api/messages)（`"api_protocol": {"protocol": "anthropic", "endpoint": "messages"}`）和 Google 的 [Gemini API](https://ai.google.dev/api/generate-content)（`"api_protocol": {"protocol": "gemini", "endpoint": "generateContent"}`）。这些非核心 API 协议都会通过 [utils/apiProtocols](https://github.com/on-panda/on-panda/tree/main/src/utils/apiProtocols/) 转换为 Chat Completions API。使用这些非核心的 API 协议只能做基础的 agent loop，不支持模型续写、logprobs 等 onPanda 进阶能力。

## 进阶请求参数及其对应作用

onPanda 的进阶功能依赖的 API 请求参数有：`continue_final_message`、`top_logprobs`、`prompt_logprobs` 和 `tool_choice: "none"`（都属于 [vLLM 的 Chat Completions API](https://docs.vllm.ai/en/latest/serving/online_serving/openai_compatible_server/#chat-api)）。

- `continue_final_message` 是支持模型续写功能的关键字段。
- `top_logprobs` 则提供了 token 的概率和候选功能。
- `prompt_logprobs` 则允许 `更新词组概率和候选` 功能，并能一探模型背后的 chat template 和隐藏的 system prompt。
- `tool_choice: "none"` 则会关闭 tool_call parser，让 tool_calls 字段也能支持 logprobs。
- 因为 API 请求是从浏览器直接发起，需要 API 的 server 支持 [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)。
    - 若 API 不支持 CORS，可以用 `@on-panda/serve` 自带的 API 代理 `/api-proxy/{base_url}` 来绕过 CORS 限制。
    - 比如，填写 API 的时候改为 `"base_url": "/api-proxy/https://api.inference.wandb.ai/v1"`。


可点击控制参数中 `API compatibility` 的 test 按钮，自动检查当前 API 及模型与上述 onPanda 所需特性的兼容性。做 `API compatibility` 测试时，推荐用 F12 打开页面 Inspection，在 Console 中查看测试请求及日志。

或者使用脚本 [testApiOnPandaCompatibility.js](https://raw.githubusercontent.com/on-panda/on-panda/refs/heads/main/src/utils/testApiOnPandaCompatibility.js) 对 API `/models` 下的所有模型执行批量测试，例如：
```bash
curl -OfsSL https://raw.githubusercontent.com/on-panda/on-panda/refs/heads/main/src/utils/testApiOnPandaCompatibility.js && \
node ./testApiOnPandaCompatibility.js --base_url https://api.inference.wandb.ai/v1 --api_key $WANDB_API_KEY
```

## 推理工具支持情况

vLLM 和 SGLang 支持所有 onPanda 依赖的进阶功能。
- 注意：vLLM 在 context 较长的时候，使用 `prompt_logprobs` 功能容易导致 server 显存超限崩溃（CUDA OOM）。推荐使用如下配置来避免 `prompt_logprobs` 导致的 CUDA OOM：
```yaml
enable_chunked_prefill: true
max_num_batched_tokens: 2048  # 仍然遇到 CUDA OOM 则往小调
```

llama.cpp、Ollama 支持除了 `prompt_logprobs` 外的所有功能。即能正常渲染 token 概率，选择 top_logprobs 候选和支持模型续写操作。

## API 供应商支持情况
版本 `2026-09-30`

现有的 API 服务商对 onPanda 的兼容性评分，用户可自行使用脚本 [testApiOnPandaCompatibility.js](https://raw.githubusercontent.com/on-panda/on-panda/refs/heads/main/src/utils/testApiOnPandaCompatibility.js) 测试任何 API。

### 五星推荐
支持 onPanda 的所有进阶功能，检查、把玩模型的绝佳 API，支持 `prompt_logprobs`（`更新词组概率和候选`）功能。

[W&B inference API](https://forge.coreweave.com/wandb/inference) 的以下模型：
- moonshotai/Kimi-K2.7-Code
- Qwen/Qwen3.8-27B
- Qwen/Qwen3.6-35B-A3B
- deepseek-ai/DeepSeek-V3.1
- meta-llama/Llama-3.3-70B-Instruct
- Qwen/Qwen3-30B-A3B-Instruct-2507


### 四星推荐
能正常渲染 token 概率，选择 top_logprobs 候选和支持模型续写操作，但不支持 `prompt_logprobs`。

[together.ai serverless inference API](https://www.together.ai/serverless-inference) 的几乎大多数主流模型，包括：
- zai-org/GLM-5.3
- moonshotai/Kimi-K3
- Qwen/Qwen3.8-2.4T-A95B
- deepseek-ai/DeepSeek-V4.1-Flash

[StepFun API](https://platform.stepfun.com/) 的大多数模型，包括：
- step-5 系列
- step-3.7-flash

Tips：
- 如果发现 API 有琐碎的 BUG，比如前置空格被吞掉，reasoning or tool_calls 字段没有 top_logprobs 等现象，可以考虑设置 `"stream": false` 模式再来测试一下。


