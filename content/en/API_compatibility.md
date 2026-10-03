# Compatibility of Inference Tools and APIs with onPanda

## Supported LLM API protocols

Chat Completions is onPanda's primary API protocol, with [vLLM's Chat Completions API](https://docs.vllm.ai/en/latest/serving/online_serving/openai_compatible_server/#chat-api) serving as the reference. Features not defined by the OpenAI protocol, such as model continuation and `prompt_logprobs`, use the interfaces defined by vLLM.

In addition to Chat Completions, you can configure the API's `api_protocol` field to use Anthropic's [Messages API](https://platform.claude.com/docs/en/api/messages) (`"api_protocol": {"protocol": "anthropic", "endpoint": "messages"}`) or Google's [Gemini API](https://ai.google.dev/api/generate-content) (`"api_protocol": {"protocol": "gemini", "endpoint": "generateContent"}`). These other API protocols are converted to the Chat Completions API through [utils/apiProtocols](https://github.com/on-panda/on-panda/tree/main/src/utils/apiProtocols/). They support only basic agent loops and do not support advanced onPanda features such as model continuation or logprobs.

## Advanced request parameters and their roles

onPanda's advanced features depend on the following API request parameters: `continue_final_message`, `top_logprobs`, `prompt_logprobs`, and `tool_choice: "none"` (all part of [vLLM's Chat Completions API](https://docs.vllm.ai/en/latest/serving/online_serving/openai_compatible_server/#chat-api)).

- `continue_final_message` is the key parameter for model continuation.
- `top_logprobs` provides token probabilities and candidates.
- `prompt_logprobs` enables the `Update phrase probabilities and candidates` feature and lets you inspect the model's underlying chat template and hidden system prompt.
- `tool_choice: "none"` disables the tool_call parser, allowing the tool_calls field to support logprobs as well.
- Because API requests are sent directly from the browser, the API server needs to support [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing).
    - If an API does not support CORS, you can use the built-in API proxy at `/api-proxy/{base_url}` provided by `@on-panda/serve` to bypass CORS restrictions.
    - For example, set `"base_url": "/api-proxy/https://api.inference.wandb.ai/v1"` in the API configuration.

Click the test button for `API compatibility` in the control parameters to automatically check whether the current API and model support the features listed above. When running an `API compatibility` test, press F12 to open the browser's developer tools and view the test requests and logs in the Console.

Alternatively, use the [testApiOnPandaCompatibility.js](https://raw.githubusercontent.com/on-panda/on-panda/refs/heads/main/src/utils/testApiOnPandaCompatibility.js) script to run batch tests on all models listed under the API's `/models` endpoint. For example:

```bash
curl -OfsSL https://raw.githubusercontent.com/on-panda/on-panda/refs/heads/main/src/utils/testApiOnPandaCompatibility.js && \
node ./testApiOnPandaCompatibility.js --base_url https://api.inference.wandb.ai/v1 --api_key $WANDB_API_KEY
```

## Inference tool support

vLLM and SGLang support all the advanced features that onPanda depends on.

- Note: In vLLM, using `prompt_logprobs` with a long context can cause the server to crash due to insufficient GPU memory (CUDA OOM). The following configuration is recommended to avoid CUDA OOM errors caused by `prompt_logprobs`:

```yaml
enable_chunked_prefill: true
max_num_batched_tokens: 2048  # Lower this value if CUDA OOM errors persist
```

llama.cpp and Ollama support all features except `prompt_logprobs`. They support displaying token probabilities, selecting `top_logprobs` candidates, and model continuation.

## API provider support

Version: `2026-09-30`

The following ratings describe how well existing API providers work with onPanda. You can test any API yourself using the [testApiOnPandaCompatibility.js](https://raw.githubusercontent.com/on-panda/on-panda/refs/heads/main/src/utils/testApiOnPandaCompatibility.js) script.

### Five-star recommendations

These APIs support all of onPanda's advanced features and are excellent for inspecting and experimenting with models, including support for `prompt_logprobs` (the `Update phrase probabilities and candidates` feature).

The following models from the [W&B Inference API](https://forge.coreweave.com/wandb/inference):

- moonshotai/Kimi-K2.7-Code
- Qwen/Qwen3.8-27B
- Qwen/Qwen3.6-35B-A3B
- deepseek-ai/DeepSeek-V3.1
- meta-llama/Llama-3.3-70B-Instruct
- Qwen/Qwen3-30B-A3B-Instruct-2507

### Four-star recommendations

These APIs support displaying token probabilities, selecting `top_logprobs` candidates, and model continuation, but do not support `prompt_logprobs`.

Most mainstream models from the [together.ai serverless inference API](https://www.together.ai/serverless-inference), including:

- zai-org/GLM-5.3
- moonshotai/Kimi-K3
- Qwen/Qwen3.8-2.4T-A95B
- deepseek-ai/DeepSeek-V4.1-Flash

Most models from the [StepFun API](https://platform.stepfun.com/), including:

- The step-5 series
- step-3.7-flash

Tips:

- If you notice minor API bugs, such as leading spaces being dropped or missing `top_logprobs` in the `reasoning` or `tool_calls` fields, try setting `"stream": false` and testing again.
