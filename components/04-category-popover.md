# 04 · 在标题与底部导航之间放置分类浮层

> 状态：源码解读。来源版本见[来源记录](../docs/sources.md)。

## 适合分享什么

在回响页选择分类时，用户仍需要操作顶部模式切换和底部导航。`CategorySelectionPopover` 把浮层限定在两者之间，展示“先选分类方式，再选具体值”的流程，适合讲非模态浮层的布局与状态边界。

## 源码已有的行为

组件通过 `anchorRef` 测量标题底边，通过 `.bottom-nav` 测量导航顶边，把可用矩形写成 CSS 变量。`ResizeObserver`、窗口 resize 和 scroll 共同触发测量，并用 `requestAnimationFrame` 合并请求；首次完成定位前浮层隐藏。

核心计算来自 `app/components/CategorySelectionPopover.tsx:102–105`：

```tsx
const header = anchorRef.current?.getBoundingClientRect();
const nav = document.querySelector<HTMLElement>(".bottom-nav")?.getBoundingClientRect();
const top = Math.max(0, Math.ceil(header?.bottom ?? 0));
const bottom = Math.max(0, Math.ceil(window.innerHeight - (nav?.top ?? window.innerHeight)));
```

`draftKind` / `draftValue` 是浮层临时状态；已生效的 `selection` 由父组件提供。选择分类方式不会提交，选择具体值才调用 `onSelect`；应用结果和关闭浮层由父组件负责。

它明确使用 `role="dialog"`、`aria-modal="false"`，打开时聚焦分类方式按钮，不建立全屏焦点陷阱，也不设置页面滚动锁。只有已有选择、非必选且不忙时才允许通过关闭按钮、Escape 或背景取消。

## 复用边界

原实现依赖主题模型、成长分档、思行倾向和分类会话类型，不能直接作为通用筛选器发布。第一步可以把选项数据和显示文案交给调用方；布局方面把硬编码的 `.bottom-nav` 查询改为明确的底部锚点。

需要保留 `.category-picker*` 样式及其层级关系。浮层范围外的导航可用，是布局和事件边界共同作用的结果，单独设置 `aria-modal="false"` 不会自动实现这一点。

提取时还要补充关闭后焦点恢复策略；原组件没有通用的“返回触发按钮”实现。它用 `window.innerHeight` 计算高度，移动端软键盘、视觉视口变化及嵌套滚动容器需要单独验证。标题说明使用固定 DOM ID，多实例时也需处理冲突。

## 验证线索

源码可核对首次定位、监听器清理和 `canCancel` 条件。独立示例至少覆盖标题换行、底栏尺寸变化、无选项、已有选择取消、必选状态下离开页面，以及提交回调仅由选值触发。本篇未对独立浮层作浏览器验收。

[返回目录](../README.md)
