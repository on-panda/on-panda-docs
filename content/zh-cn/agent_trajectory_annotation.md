# 高效的 Agent Trajectory Annotation：通过 MCP 动态接入环境，在线标注带 tool call 的 SFT、DPO 数据


![Annotating Agent Trajectories | onPanda](https://on-panda.github.io/img/fig2_agent-v4.png)  
**Annotating Agent Trajectories.** Reasoning and tool-call arguments remain editable at token level. Corrected tool calls can be executed in the connected environment, and the resulting trajectory continues from the corrected context.


onPanda 作为 LLM alignment 标注工具，支持 Agent Trajectory 标注，其能动态


![The token-level correction interface](https://on-panda.github.io/img/onPanda-token-level-correction.gif)

此外，onPanda 创新性地采用 token-level correction 的标柱形式具备多种优势：
- 标注速度
- 一次标注获得多钟数据
- token-level 监督信号
- onPolicy
- 对标注员负担小



