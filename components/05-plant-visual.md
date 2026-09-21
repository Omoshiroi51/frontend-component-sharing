# 05 · 图片加载失败后，如何保留可辨认的后备图形

> 状态：源码解读。来源版本见[来源记录](../docs/sources.md)。

## 适合分享什么

`TopicPlantVisual` 在圆形主题球、主题卡片和详情中展示植物。它优先使用已有图片，图片明确失败或没有资源时才展示由 HTML / CSS 组成的植物后备，避免把网络加载中误当成资源不可用。

## 源码已有的行为

组件以资源 URL 关联加载和失败状态，并使用模块内的 `Set<string>` 记住当前模块生命周期内成功加载过的资源。

源码 `app/components/TopicPlantVisual.tsx:36–41` 的状态判定摘录：

```tsx
const runtimeAsset = runtimeAssetOverride ?? resolvePlantRuntimeAsset(species, shape.stage);
const [loadedSource, setLoadedSource] = useState<string | null>(null);
const [failedSource, setFailedSource] = useState<string | null>(null);
const assetState = !runtimeAsset || failedSource === runtimeAsset.src
  ? "fallback"
  : loadedSource === runtimeAsset.src || loadedPlantAssetSources.has(runtimeAsset.src) ? "loaded" : "loading";
```

`img` 的 `onLoad` / `onError` 更新对应 URL；容器附加 `asset-loading`、`asset-loaded` 或 `asset-fallback` 类名。程序化植物节点始终有结构，显示时机由关联 CSS 控制。

图片带有明确宽高，支持 eager / lazy 加载，禁止拖动并异步解码。圆形展示还会使用资源对应的内缩比例。提供 `label` 时由外层承接图片语义，内层图片和装饰节点隐藏；纯装饰场景则整体隐藏于辅助技术。

## 复用边界

可分享的通用部分是“按资源身份划分的加载状态 + 后备呈现”。原植物组件还依赖：

- `app/wisdom/plant-assets.ts` 的资源解析和圆形内缩比例。
- `app/wisdom/plant-presentation.ts` 的阶段与形态参数。
- `app/wisdom/plant-asset-contract.generated.ts` 的缩放和变换原点。
- `app/globals.css` 中 `.plant-*`、物种样式及真实图片资源。

因此不能把一个 TSX 文件当成完整可分享组件。提取通用版时可先把输入缩小到资源地址、尺寸、可访问名称和后备内容，植物素材仍单独处理。

模块级 Set 只记录已加载地址，不等于缓存图片字节或提供离线能力。它没有容量上限；来源素材集合有限，若改用于大量动态 URL，需要重新考虑是否保留这个集合。原组件也没有主动重试同一失败 URL 的接口。

## 验证线索

`tests/browser/issue-85-topic-card-experience.spec.ts` 有“正式植物资源失败后才显示程序化后备”用例，`tests/browser/topic-sphere-static.spec.ts` 涉及球群素材失败后备。本次未运行。提取后应分别模拟慢加载、失败、切换 URL、重复显示同一资源，并检查圆形展示有无裁切。

[返回目录](../README.md)
