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
