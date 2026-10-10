# 使用 onPanda 高效地标注 Agent Trajectory 数据
**通过 MCP 动态接入环境，在线标注多轮带 tool call 的 SFT、preference 数据**


[onPanda](https://on-panda.github.io/research/) 作为 LLM alignment 数据标注工具，全面支持标注 Agent Trajectory 数据。标注员可在 onPanda 界面高效修改模型 reponse 中的 `reasoning`, `content`, `tool_calls` 。

onPanda 通过 MCP 协议动态的接入环境，从而在真实环境中交互式地标注 agent 轨迹，获得 SFT 和 preference 数据。


<a href="https://on-panda.github.io/img/fig2_agent-v4.png">
  <img src="https://on-panda.github.io/img/fig2_agent-v4.png" alt="Annotating Agent Trajectories | onPanda" style="max-width:600px" loading="lazy">
</a>

**Annotating agent trajectories with onPanda.** Reasoning and tool-call arguments remain editable at token level. Corrected tool calls can be executed in the connected environment, and the resulting trajectory continues from the corrected context.




![The token-level correction interface](https://on-panda.github.io/img/onPanda-token-level-correction.gif)

此外，onPanda 创新性地采用 token-level correction 的标柱形式具备多种优势：
- 标注速度
- 一次标注获得多钟数据
- token-level 监督信号
- onPolicy
- 对标注员负担小



