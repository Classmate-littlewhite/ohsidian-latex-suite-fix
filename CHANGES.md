# ohsidian-latex-suite 修改记录

基于 obsidian-latex-suite v1.11.5，适配 OHSidian (HarmonyOS)。

---

## 修改 1：修复首次加载崩溃

**文件**：`src/main.ts` 第 82 行

**问题**：插件首次加载时 `loadData()` 返回 `null`，valibot 的 `v.parse(v.record(v.string(), v.any()), null)` 抛出 `ValiError: Invalid type: Expected Object but received null`，导致插件加载失败。

```diff
- let data = v.parse(v.record(v.string(), v.any()), await this.loadData());
+ let data = v.parse(v.record(v.string(), v.any()), await this.loadData() || {});
```

---

## 修改 2：修复 HarmonyOS 自动触发（auto-snippet）

**文件**：`src/latex_suite.ts` 第 32-63 行

**问题**：HarmonyOS 的 IME 框架对所有按键发送 `keyCode=229` / `key="Process"`，导致原始的 `keyboardEventPlugin.onKeydown` 路径（key 为 Process 时直接 return）和 `onInput` 路径（`isComposing()` 检查 `event.keyCode === 229`）均被阻断。加上 `snippetIMEVersion` 设置在 OHSidian 中无法开启、`suppressSnippetTriggerOnIME` 卡在默认值 `true`，自动触发完全失效。

**修复方案**：在 `handleUpdate` 中，移除对 `snippetIMEVersion` 的依赖，直接从 CodeMirror 事务的 changeset 中提取插入字符，硬编码 `isIME=false`（因为在 update 阶段 IME 已提交完毕），从而绕过所有 DOM 级 IME 检测。

```diff
- if (userEvent.length === 0 || !settings.snippetIMEVersion) {
+ if (userEvent.length === 0 || !settings.snippetsEnabled) {
       return;
  }

- // HACK: reusing logic from handleKeydown with empty string
- const success = handleKeydown("", false, update.view.composing, update.view);
+ // Auto-trigger snippets from committed input events.
+ // When keyboardEventPlugin successfully handles an auto-snippet, it calls
+ // event.preventDefault(), so no input.type transaction is created — we
+ // won't double-fire. This path matters on platforms (HarmonyOS, some IME
+ // configurations) where the keydown event carries keyCode=229 or key="Process",
+ // causing keyboardEventPlugin to bail out and onInput's isComposing() check
+ // to block the trigger.
+ //
+ // Since we run after text is committed, there is no risk of interfering with
+ // IME composition — view.composing is already false. We therefore bypass the
+ // suppressSnippetTriggerOnIME setting here so that auto-trigger works even when
+ // settings cannot be persisted (e.g. on some embedded platforms).
+
+ // Only handle single-character insertions (not pastes)
+ let inserted = "";
+ update.changes.iterChanges((_fromA, _toA, _fromB, _toB, ins) => {
+     inserted += ins.toString();
+ });
+ if (inserted.length !== 1) return;
+
+ // key is "" because the character is already in the document at cursor pos.
+ // isIME is false: at update time the composition has already committed,
+ // so there's no risk of interfering with IME input.
+ const success = handleKeydown("", false, false, update.view);
  if (success) {
      console.debug("Handled input event as snippet trigger");
  }
```

### 为什么不会重复触发

- 当 `keyboardEventPlugin` 成功匹配自动片段时，会调用 `event.preventDefault()` 阻止字符插入，因此不会产生 `input.type` 事务
- `handleUpdate` 仅在存在 `input.type` 事务时进入此分支
- 两条路径互斥，不会重复处理

### 输入事件流对比

| 阶段 | 桌面端 | HarmonyOS |
|------|--------|-----------|
| keydown | key="s" → handleKeydown → 匹配成功 → preventDefault | key="Process" → 保存 lastKeyboardEvent → return |
| onInput | 不触发（已 preventDefault） | text="s" + lastKeyboardEvent → isComposing(event) → keyCode=229 → isIME=true → suppressSnippetTriggerOnIME 阻断 |
| handleUpdate | 无 input.type 事务 | **新路径**：input.type 存在 → 提取 inserted="s" → handleKeydown("", false, false, view) → 匹配成功 |

---

## 修改 3：manifest.json

```diff
- "id": "obsidian-latex-suite",
+ "id": "ohsidian-latex-suite",
- "version": "1.11.5",
+ "version": "0.0.1-1.11.5",
- "author": "artisticat",
+ "author": "artisticat, littlewhite0417",
```

## 修改 4：package.json

```diff
- "name": "obsidian-latex-suite",
+ "name": "ohsidian-latex-suite",
- "version": "1.11.5",
+ "version": "0.0.1-1.11.5",
```

---

## 构建

```bash
npm install
npm run build
# 产物: main.js (238.5KB), manifest.json, styles.css
```

## 发布文件

- `main.js`
- `manifest.json`
- `styles.css`
