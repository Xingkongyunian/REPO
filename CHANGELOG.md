# Changelog

## 2.0.0

- 适配 R.E.P.O. 0.5.1。
- **不再接管 `ChatManager.Update`**：0.5.1 聊天已原生支持 IME（`OnCompositionChange`）与 Ctrl+V 粘贴（`ChatPasteLogic`），旧版接管会把聊天状态机顶坏。
- 重写菜单输入框补丁：IME 组合显示、候选框跟随光标、提交帧交还原版追加 `Input.inputString`、Ctrl+V 粘贴、去掉强制大写。
- 接管期间主动刷新 `MenuManager.TextInputActive()`，避免长组合时输入框掉焦点。
- 无输入框激活时把 `imeCompositionMode` 恢复为 Auto，避免与聊天输入法冲突。
- 启动时打印字段探测与聊天原生能力，字段缺失自动降级。
