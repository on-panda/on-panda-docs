# 使用 onPanda 高效地标注 Agent 轨迹数据
**动态接入 agent 运行环境，在线标注多轮带 tool call 的 SFT、preference 数据**


现有的 agent 标注工具大都只支持对已完成的固定轨迹标注属性或者文字反馈，无法方便地修改轨迹获得 SFT 数据，这是因为两个难点：
1. Agent 轨迹由工具调用和环境反馈交替组成，修改某一轮的工具调用，会导致原有后续轨迹通常不再适用，需要重新执行并生成后续轨迹
2. Agent 时代，模型输出变得更加复杂——除了 `content`，还会有冗长的 `reasoning`，以及结构化的 `tool_calls`——这让标注员需要同时理解和修改思维链、工具参数，甚至环境状态，人工重写成本高。标注界面开发也变得棘手

[onPanda](https://on-panda.github.io/research/) 作为 LLM alignment 数据标注工具，全面支持标注 Agent 轨迹数据，其差异化的理念是「纠正生成过程」，而不是「事后修改结果」。onPanda 支持动态地接入现有 agent 的运行环境，并高效地修改模型 reponse 中的 `reasoning`, `content` 和 `tool_calls`，从而在真实环境中交互式地标注 agent 轨迹，高效地获得 on-policy 的 SFT 和 preference 数据。

onPanda 标注 agent 轨迹的标注界面示意图：

<center>
    <a href="https://on-panda.github.io/img/fig2_agent-v4.png">
        <img src="https://on-panda.github.io/img/fig2_agent-v4.png" alt="Annotating Agent Trajectories | onPanda" style="max-width:600px" loading="lazy">
    </a>
</center>

*onPanda 支持 token-level 地修改 `reasoning` 和 `tool_calls`，并让模型基于修正后的前缀继续生成，随后在真实的 agent 环境中执行修正过的 `tool_calls`。*

onPanda 实际标注 agent 轨迹数据时的录屏演示：

<center>
    <iframe width="560" height="315" src="https://www.youtube.com/embed/0T4h4IUIZCM?si=xX-W1SlEW8h5ZBQK&amp;start=102" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</center>


onPanda 标注 agent 轨迹的过程依赖真实的环境反馈，可以通过两种方案来接入目标环境：
1. onPanda  直接使用 MCP 协议接入 onPanda
 onPanda 是一个


通过 MCP。或者 [harness_to_mcp](https://github.com/on-panda/harness_to_mcp) 接入 harness 或者 agent 运行环境

<center>
    <a href="https://x.com/diyerxx/status/2101020850405462041">
        <img src="https://on-panda.github.io/img/onPanda-token-level-correction.gif" alt="onPanda 的 token 级纠错界面" style="width:500px; max-width:100%; height:auto">
    </a>
</center>


onPanda 创新性地采用 token-level correction 的标柱形式，其具备多种优势：
- 标注速度快
- 一次标注同时获得 SFT 和 perfrence 数据
- token-level 监督信号，可直接转换为 step-level data or process reward data (PRM)
- onPolicy
- 对标注员负担小
- 自定义标注支持 turn-level 的属性标注

作为开源软件，onPanda 已经在 StepFun 内部生产了数十万条 SFT 和 preference 数据，包括 agent trajectories。且 onPanda 的代码是组件化和前后端分离的设计，核心组件可以被其他项目导入和复用，能方便地集成进现有的数据标注平台，参见 [onPanda integration examples](https://github.com/on-panda/on-panda/tree/main/apps/integration-example)。


更多细节请查看我们研究项目主页： https://on-panda.github.io/research/

并查阅我们的论文： [onPanda: Efficient Annotation of On-Policy Alignment Data for LLMs and Agents via Token-Level Correction](https://arxiv.org/abs/2609.24983)

Why onPanda



<!-- 
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
