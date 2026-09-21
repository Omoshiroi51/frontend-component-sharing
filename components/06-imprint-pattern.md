# 06 · 快速切换背景纹理时，避免旧图片覆盖新选择

> 状态：源码解读。来源版本见[来源记录](../docs/sources.md)。

## 适合分享什么

智慧卡片的背景纹理由成长状态、外观和配色共同决定。`WisdomImprintPattern` 值得分享的部分是图片异步加载：用户连续切换选项时，旧请求可能更晚结束，组件需要确保只显示当前选择的资源。

## 源码已有的行为

组件先根据业务状态生成候选资源路径，再用 `requestKey` 标识本轮外观请求。`attempt` 和 `loaded` 都附带这个键，只有键匹配的已加载资源才会用于当前背景。

下面是 `app/components/WisdomImprintPattern.tsx:45–63` 中的加载 effect 摘录：

```tsx
useEffect(() => {
  if (!candidate) return;
  let active = true;
  const image = new window.Image();
  image.onload = () => {
    if (active) setLoaded({ key: requestKey, src: candidate });
  };
  image.onerror = () => {
    if (!active) return;
    setLoaded((current) => current?.key === requestKey ? null : current);
    setAttempt({ key: requestKey, index: attemptIndex + 1 });
  };
  image.src = candidate;
  return () => {
    active = false;
    image.onload = null;
    image.onerror = null;
  };
}, [attemptIndex, candidate, requestKey]);
```

这里有两道检查：effect 清理阻止旧回调更新状态；渲染时的键匹配阻止上一轮已存入的结果用于新选择。失败后递增候选下标，依次尝试同一成长状态对应的图片方案。

最终只是一个 `aria-hidden="true"` 的背景层，使用 CSS 平铺、透明度和模糊控制纹理密度，并通过 `pointer-events: none` 保持正文可操作。

## 复用边界

依赖 `app/wisdom/growth-imprint.ts`、`app/wisdom/imprint-appearance.ts`、对应图片资源及 `.echo-imprint-pattern*` 样式。提取时可把业务计算留在调用方，仅接收有序候选地址和能唯一标识请求的键；这是提取建议，尚未实现。

**所有候选图片失败时，此组件不显示背景图。** 共享渲染计划最后包含文字后备项，但这个装饰层仅筛选图片候选，不额外渲染后备文字；卡片自己的正文仍由父组件负责。

清理回调不代表取消网络请求。请求键也必须覆盖所有影响资源路径的输入；如果未来增加尺寸、主题色或资源版本，应重新检查键是否足够完整。

## 验证线索

调用位置可在 `app/components/WisdomCard.tsx` 与 `app/components/ImprintDensitySettings.tsx` 找到。`tests/imprint-assets.test.mjs` 检查资源集合，`tests/growth-imprint.test.mjs` 检查后备计划，但它们不能替代本组件的异步竞态验证。

独立提取后应人为控制图片加载顺序，验证 A → B 切换后 A 迟到不会覆盖 B、候选失败会推进、全部失败不保留旧纹理，以及卸载后没有过期状态更新。本次未执行这些浏览器场景。

[返回目录](../README.md)
