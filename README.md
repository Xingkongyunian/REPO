# REPO Chinese Input Plus（原「聊天框可输入中文」的 0.5.1 重写版）

BepInEx 5 插件，解决 **0.5.1** 下菜单输入框（房间名、玩家名、存档名、密码）无法输入中文、不能粘贴、强制大写的问题。

---

## 0.5.1 变了什么，原版为什么不能用

| 位置 | 0.5.1 现状 | 原版 ChineseInput 1.0.0 |
| --- | --- | --- |
| 聊天框 | 已换成 **Input System 原生 IME**：`ChatManager.OnCompositionChange`、`compositionString`、`OnTextInput`，还有原生 `ChatPasteLogic`（Ctrl+V） | 仍用旧版 `Input.inputString` 重写 `ChatManager.Update`，且 Prefix 返回 false **直接跳过游戏状态机** —— 在 0.5.1 上会把聊天顶坏（打不开/发不出） |
| 菜单输入框 | 仍用旧版 `Input.inputString`，**不读 IME 组合串**，且 `upperOnly` 强制大写 | 接管了 `Update`，思路正确，但把聊天的补丁一起带上了 |
| 输入框激活计时 | `MenuManager.TextInputActive()` 每帧刷新 0.1s 计时器 | 跳过原版 Update 时没有刷新 → 长按输入法组合时输入框可能被判为“无输入” |

---

## 本版做了什么

只处理**菜单输入框**，聊天完全交给游戏本体：

1. **输入法组合显示**：组合期间把 `已有文本 + Input.compositionString` 显示到 `textCurrent`，候选框跟随光标（`Input.compositionCursorPos`），并跳过原版 Update，避免拼音字母上屏；
2. **提交处理**：组合结束的那一帧还原基础文本，交回原版把 `Input.inputString` 里已提交的中文追加进去；
3. **Ctrl+V 粘贴**：自动去掉控制字符与换行，并遵守每个输入框自己的 `maxLength`；
4. **去掉强制大写**：`upperOnly = false`（Start 与每帧双重保险）；
5. **不再抢聊天**：`ChatManager` 一行不碰；
6. **输入框计时刷新**：接管期间主动调用 `MenuManager.TextInputActive()`，长组合不会掉焦点；
7. **输入法按需开启**：没有菜单输入框激活时自动 `imeCompositionMode = Auto`，避免和聊天输入法冲突。

启动日志会打印字段探测与聊天原生能力：

```
[中文输入] 字段探测：textCurrent=OK, upperOnly=OK, maxLength=OK, textMain=OK, InputTextSet=OK, textInputActive=OK
[中文输入] 聊天框原生支持：IME=True, 粘贴=True（本插件不接管聊天）。
```

字段缺失时对应功能自动跳过，不会刷异常。

---

## 安装

`BepInEx/plugins/RepoChineseInputPlus/RepoChineseInputPlus.dll`

依赖：BepInEx 5.4.21+；游戏 0.5.1。

## 配置

`BepInEx/config/xingKong.RepoChineseInputPlus.cfg`：

| 配置 | 默认 | 说明 |
| --- | --- | --- |
| `1. 通用 / 启用` | 开 | 总开关（只影响菜单输入框） |
| `2. 输入 / 启用输入法组合` | 开 | 拼音/日文/韩文输入法组合与提交；关掉后只能粘贴 |
| `2. 输入 / Ctrl+V 粘贴` | 开 | 菜单输入框粘贴（自动清理换行/控制符） |
| `2. 输入 / 去掉强制大写` | 开 | 保留大小写，也是输入汉字的前提 |

---

## 验证方式

1. 进游戏 → 大厅 → 修改房间名/玩家名；
2. 切到中文输入法，输入拼音，应看到候选框跟随文本，组合期间不出现散落的拼音字母；
3. 选词后应正确落到输入框；
4. `Ctrl+V` 粘贴一段中文，应能完整粘贴且不超过长度上限。

如果选词后没有文字落下（少数输入法只在 composition 里给结果），把日志发我，我改成“提交帧取最后一帧 composition”的策略。

---

## 从源码构建

```powershell
dotnet build RepoChineseInputPlus.csproj -c Release -p:RepoDir="D:\steam\steamapps\common\REPO"
```
