# 01 · 一个能用键盘操作的胶囊分段切换器

> 状态：源码解读。来源版本见[来源记录](../docs/sources.md)。

## 适合分享什么

智慧回响把“快速 / 详细”记录和“随机 / 分类”回响共用为一个 `ModeSwitch`。适合讲解：怎样把业务状态留给父组件，同时把选中外观、键盘操作和焦点移动封装在一个小组件里。

## 源码已有的行为

- `value`、`options`、`onChange` 构成受控接口，组件不保存另一份选中值。
- 容器是 `radiogroup`，按钮是 `radio`，通过 `aria-checked` 表示选择。只有选中项进入 Tab 顺序。
- 方向键循环选择；Home / End 跳到首尾。选择后在下一帧把焦点移到对应按钮。
- 一个独立的背景滑块通过 CSS 变量移动，文字和按钮本身保持原位。
- 点击已选项只调用可选的 `onActiveClick`；来源页面用这个入口承接当前模式的附加操作。

源码 `app/components/ModeSwitch.tsx:32–37` 的键盘移动辅助函数摘录：

```tsx
const moveTo = (index: number) => {
  const next = options[(index + options.length) % options.length];
  if (!next) return;
  onChange(next.value);
  window.requestAnimationFrame(() => buttonRefs.current[(index + options.length) % options.length]?.focus());
};
```

接入示意，需先提取组件及关联样式到当前应用：

```tsx
const [mode, setMode] = useState<"quick" | "detailed">("quick");

<ModeSwitch
  value={mode}
  options={[
    { value: "quick", label: "快速" },
    { value: "detailed", label: "详细" },
  ]}
  ariaLabel="记录模式"
  onChange={setMode}
/>
```

## 复用边界

依赖 React、TypeScript 和 `app/globals.css` 中 `.mode-switch*`、全局焦点及减少动态规则，无需复制智慧业务模型。

**现有 CSS 只支持两段**：它使用 `repeat(2, ...)` 和约 50% 的滑块宽度。虽然 TypeScript 接口接受数组，直接传三项不会得到正确布局。首个提取版保持两项即可；要扩展再同步计算列数和滑块尺寸。

还应要求选项非空、值唯一，且 `value` 属于选项。原组件找不到选中值时会把滑块放到第一项，但按钮没有相应的选中态和 Tab 入口；它没有替调用方校验这些约束。该交互也假定父组件及时接受键盘切换后的值。

## 验证线索

`tests/browser/issue-69-capture-workspace.spec.ts` 中“快速记录默认只给一句话输入……”用例包含方向键切换检查。独立提取时还应检查首尾循环、Home / End、禁用、焦点环，以及减少动态时的即时切换。本次未运行这些用例。

[返回目录](../README.md)
