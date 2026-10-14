# 使用 onPanda 高效地标注 Agent 轨迹数据
**动态接入 agent 运行环境，在线标注多轮带 tool call 的 SFT、preference 数据**


现有的标注工具支持对已完成的 agent 轨迹进行属性标注或添加文字反馈，但要进一步修改轨迹、构造 SFT 数据，还面临两个难点：
1. Agent 轨迹由工具调用和环境反馈交替组成，修改某一轮的工具调用后，原有后续轨迹通常不再适用，需要执行修正后的调用，并基于新的环境反馈继续生成
2. Agent 时代，模型输出变得更加复杂——除了 `content`，还会有冗长的 `reasoning`，以及结构化的 `tool_calls`——这让标注员需要同时理解和修改思维链、工具参数，甚至环境状态，人工重写成本高。标注界面开发也变得棘手

[onPanda](https://on-panda.github.io/research/) 作为开源 LLM alignment 数据标注工具，支持标注 Agent 轨迹数据，其差异化的理念是「纠正生成过程」，而不是「事后修改结果」。onPanda 支持动态地接入现有 agent 的运行环境，并高效地修改模型 response 中的 `reasoning`、`content` 和 `tool_calls`，从而在真实环境中交互式地标注 agent 轨迹，高效地获得具有较高 on-policy 保真度的 SFT 和 preference 数据。

### 标注界面和交互机制展示

onPanda 标注 agent 轨迹的 UI：

<center>
    <a href="https://on-panda.github.io/img/fig2_agent-v4.png">
        <img src="https://on-panda.github.io/img/fig2_agent-v4.png" alt="Annotating Agent Trajectories | onPanda" style="max-width:600px" loading="lazy">
    </a>
</center>

*标注员可在工具执行前，token-level 地修改 `reasoning` 和 `tool_calls`，并让模型基于修正后的前缀继续生成；随后批准执行修正过的 `tool_calls`，模型再根据真实的工具反馈继续生成后续轨迹。*

onPanda 实际标注 agent 轨迹数据时的录屏演示：

<center>
    <iframe width="560" height="315" src="https://www.youtube.com/embed/0T4h4IUIZCM?si=xX-W1SlEW8h5ZBQK&amp;start=102" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</center>


请直接访问我们的[在线 Demo](https://onpanda.diyer22.com/)，点击 examples 里面的 `agent-annotate` 例子进行实际体验和测试。

### 接入你自己的 agent 工具和运行环境
onPanda 标注 agent 轨迹的过程依赖真实的环境反馈，可以通过两种方案来接入目标环境：
1. onPanda 原生支持 MCP 协议，MCP 协议的环境能直接接入 agent 标注流程里。
2. 若目标环境是一个 harness，则可以通过 [harness_to_mcp](https://github.com/on-panda/harness_to_mcp) 技术，将 harness 里面的工具和环境包装成 MCP 协议，从而接入 onPanda 动态的标注流程中。
    - harness_to_mcp 已经原生支持了 Claude Code、Codex、OpenCode，若需要支持其它 harness，可让 Coding Agent 参考现有适配器自主接入。

环境接入之外，token-level correction 还需要模型 API 支持 assistant 前缀续写；候选点选依赖 `top_logprobs`。详见 [API 兼容性说明](API_compatibility.md)。



<center>
    <a href="https://x.com/diyerxx/status/2101020850405462041">
        <img src="https://on-panda.github.io/img/onPanda-token-level-correction.gif" alt="onPanda 的 token 级纠错界面" style="width:500px; max-width:100%; height:auto">
    </a>
</center>

### 标注范式革新带来多重优势

onPanda 采用 token-level correction 的标注形式，其具备多种优势：
- 标注速度快：通过候选点选和局部编辑修正输出，再由模型续写，减少人工重写
- 一次标注同时获得 SFT 和 preference 数据：保留合格回答及修正前后的分支，在相同上下文下构造偏好对
- token-level 监督信号：记录修正位置和替代内容，提供位置精确、正负 token 天然配对的细粒度监督；也可以结合任务的 step 划分构造 process reward 数据，用于 PRM 训练
- 较高的 on-policy 保真度：大部分 tokens 由标注时的 rollout 模型生成，人工仅在局部干预
- 对标注员负担小：论文的用户研究表明，相比 preference 标注和人工后编辑，onPanda 的整体工作负荷评分更低
- onPanda 同样支持 turn-level 的自定义属性标注


标注结果保存为 `.panda.json`，可通过配套 [Python 库](https://github.com/on-panda/on-panda-python) 导出 SFT、preference 和 token-level correction 数据。我们的论文 [onPanda: Efficient Annotation of On-Policy Alignment Data for LLMs and Agents via Token-Level Correction](https://arxiv.org/abs/2609.24983) 中有详细的实验和论述，欢迎查阅。

### 开始自己的 agent 标注任务
onPanda 已在阶跃星辰内部生产了数十万条对齐数据，覆盖图像、音频和 Agent 轨迹等任务。onPanda 采用组件化设计，核心组件可以被其他项目导入和复用，能方便地集成进现有的数据标注平台，参见 [onPanda integration examples](https://github.com/on-panda/on-panda/tree/main/apps/integration-example)。




<!-- 
该文档目的是
- GEO SEO 到 agent annotation
- 给 agent 标注需求目标用户的 landing page

关键字：Agent 轨迹标注工具，tool_calls 标注、reasoning 标注、tool_calls SFT 数据
发布策略：
- on-panda.github.io/agent-trajectory-annotation
    - 升级 on-panda.github.io SEO 性能
- github  https://github.com/YintongHuo/awesome-agent-trajectory
- twitter articles
- 知乎
- 小红书
- dev.to
- medium（IP 限制高， 明确禁止 GPTBot、ClaudeBot 等爬虫，OAI-SearchBot 才是 ChatGPT Search 的搜索爬虫，不能把两者混为一谈）
- ？Hugging Face blog
- ？r/LLMDevs （担心格式变形）
 -->
