# 更新日志（dsh-mumu-mcp）

> 版本号只跟主仓库联动；这里按日期记改动。

## 2026-10-24

**触摸注入改走 sendevent 原始协议（修游戏内点不动/拖不动）**

- 症状：Android 15 上 `input tap` / `input swipe`（含 `input motionevent`）在开心消消乐等游戏里完全没反应——合成事件带 `TOOL_TYPE_FINGER`，游戏当拖动不当触摸。
- 修法：`vm/adb.py` 的 tap / swipe / long_press 改走 **sendevent 原始事件流**（MT 协议 B：`ABS_MT_SLOT` / `ABS_MT_TRACKING_ID` / `ABS_MT_POSITION_X/Y` + `BTN_TOUCH` + `SYN_REPORT`），写进自动探测的真触屏节点（`getevent -pl` 找 `INPUT_PROP_DIRECT` 设备，实测 MuMu Android 15 = `/dev/input/event4`）。
- swipe 按 50ms/步补中间 `MOVE` 事件（3~12 步自适应），长按=同点 swipe。
- 兜底链：sendevent 失败 → `input motionevent DOWN/MOVE/UP` → `input swipe`，任何系统不劣于原行为。
- 探测坑记录：设备头行 `add device N: /dev/input/eventX` 离 `INPUT_PROP_DIRECT` 标记隔着整张事件表，`grep -B8` 抠不到，要用 `-B30` + 正则抠路径。
- 游戏实测：登录页按钮、协议弹窗、账号确认全部正常响应。
- 同步：`README` 里 `qq_say_path` 文档与示例改对齐 `/dsh-qq-bridge/say`（QQ 桥 0.4.5 起的路由前缀）。

**qq_say_path 默认值对齐**

- 默认 `qq_say_path` / `qq_turn_path` / `qq_agent_state_path` 统一为 `/dsh-qq-bridge/*`（旧 `/dsh-qq-bridge-mumu/*` 是 0.4.4 前的路由前缀，沿用会导致 mumu_phone_say 404「发不出去」）。老配置文件记得手改。

## 2026-10-24（补充）

**游戏弹窗点不动的排查结论**

- 现象：开心消消乐「重复创建账号」弹窗的按钮全部点不动（sendevent/input tap/motionevent 全试）。
- 根因：**游戏引擎输入线程卡死**（进程活着但不消费任何注入事件）——是游戏卡死的症状，不是注入方案失效。强杀重启游戏（`am force-stop` + monkey LAUNCHER）后注入恢复正常。
- 附带教训：游戏引擎按钮的**命中区与视觉位置可能差 10~20px**（「同意」按钮热区比视觉位置低 ~7px），遇到「点了没反应」先用坐标网格扫描找热区，别急着判定注入失效。
- 完整登录链路已验证：登录页→账号列表→协议「同意」→自动登录→进主界面 ✅
