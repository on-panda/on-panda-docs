# 使用 onPanda 高效地标注 Agent Trajectory 数据
**动态接入 agent 运行环境，在线标注多轮带 tool call 的 SFT、preference 数据**


[onPanda](https://on-panda.github.io/research/) 作为 LLM alignment 数据标注工具，全面支持标注 Agent Trajectory 数据。标注员可以用 onPanda 高效地修改模型 reponse 中的 `reasoning`, `content` 和 `tool_calls` 。

onPanda 可以动态的接入 agent 的 harness，从而在真实环境中交互式地标注 agent 轨迹，获得 on-policy 的 SFT 和 preference 数据。

<center>
    <a href="https://on-panda.github.io/img/fig2_agent-v4.png">
        <img src="https://on-panda.github.io/img/fig2_agent-v4.png" alt="Annotating Agent Trajectories | onPanda" style="max-width:600px" loading="lazy">
    </a>
</center>

**Annotating agent trajectories with onPanda.** Reasoning and tool-call arguments remain editable at token level. Corrected tool calls can be executed in the connected environment, and the resulting trajectory continues from the corrected context.




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

作为开源软件，onPanda 已经在 StepFun 内部生产了数十万条 SFT 和 preference 数据，包括 agent trajectories。且 onPanda 的代码是组件化和前后端分离的设计，核心组件可以被其他项目导入和复用，能方便地集成进现有的数据标注平台，参见 [onPanda integration examples](https://github.com/on-panda/on-panda/tree/main/apps/integration-example)。


更多细节请查看我们研究项目主页： https://on-panda.github.io/research/

并查阅我们的论文： [onPanda: Efficient Annotation of On-Policy Alignment Data for LLMs and Agents via Token-Level Correction](https://arxiv.org/abs/2609.24983)

Why onPanda
