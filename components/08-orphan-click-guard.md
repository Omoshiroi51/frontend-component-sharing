# 08 · 拦住打开详情时的残留点击，同时允许立即进行下一次操作

> 状态：源码解读；这是交互辅助函数，不是视觉组件。来源版本见[来源记录](../docs/sources.md)。

## 适合分享什么

触摸打开详情后，原坐标可能又收到一次 click。若新页面恰好在该位置放了另一个入口，就可能连续进入下一层。直接把新页面禁用一段时间，会同时吞掉用户主动进行的第二次点击。

`suppress-orphan-click.ts` 记录新页面打开时间，并判断是否见过属于新页面的一次 pointerdown。可以用它分享事件来源判断，尤其是事件发生时间与处理时间的区别。

## 源码已有的行为

| 事件情况 | 处理 |
| --- | --- |
| 打开时间尚未写入，且没有新 pointerdown 的指针 click | 拦截 |
| 打开后 700ms 内，没有新 pointerdown 的指针 click | 拦截 |
| pointerdown 的事件时间早于页面打开时间 | 仍视作旧手势，不解除防护 |
| 已收到发生于新页面打开后的 pointerdown | 立即放行，包括打开后的第一帧 |
| `click.detail === 0` | 放行，保留键盘等合成激活路径 |
| 已记录打开时间且超过 700ms | 即使没见到 pointerdown 也停止拦截 |

状态只含 `openedAt` 和 `sawUserPointerDown`。`TopicDetail` 在 `useLayoutEffect` 里写入 `performance.now()`，随后在捕获阶段连接辅助函数。

源码 `app/components/TopicDetail.tsx:242–253` 摘录：

```tsx
onPointerDownCapture={(event) => {
  orphanClickGuardRef.current = noteOrphanClickPointerDown(
    orphanClickGuardRef.current,
    event.timeStamp,
  );
}}
onClickCapture={(event) => {
  // 触摸松手打开详情后，设备工具栏会按原坐标补发一次 click。
  if (!shouldSuppressOrphanClick(orphanClickGuardRef.current, event.detail, performance.now())) return;
  event.preventDefault();
  event.stopPropagation();
}}
```

**这里传的是 `event.timeStamp`。** 如果把 pointerdown 的处理时刻重新记成 `performance.now()`，迟到的旧事件可能被误判为新手势，防护就失去意义。

## 复用边界

纯函数文件不依赖 React；React 侧只需要 ref、打开时刻和捕获事件接线。不同事件系统若使用不同时间基准，需要先归一化，不能将 Unix 时间戳直接与这里的高精度相对时间比较。

每次打开新操作面都要重新建立状态。来源组件按挂载初始化；如果提取后复用同一实例反复打开，必须在自己的打开生命周期中重置。

这只处理“新页面打开后、尚未发生新手势”的有限窗口。它不防双击提交，也不拦截所有合成事件；`detail === 0` 并不保证一定来自键盘。已有新手势后会持续放行，符合这里的页面转换场景。

## 验证线索

`tests/suppress-orphan-click.test.mjs` 对初始化窗口、普通残留 click、零 detail、立即开始的新手势、迟到的旧 pointerdown、700ms 边界均有断言。

`tests/browser/detail-exploration-path.spec.ts` 另有残留点击不误入、新一次完整点击立即可用的浏览器场景。本次只读查阅这些用例；提取时应同时保留纯函数边界检查和实际触摸接线验证。

[返回目录](../README.md)
