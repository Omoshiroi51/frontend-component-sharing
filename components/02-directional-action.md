# 02 · 用原生按钮封装图标、加载和禁用状态

> 状态：源码解读。来源版本见[来源记录](../docs/sources.md)。

## 适合分享什么

记录流程中的上一页、下一页、保存、清空，以及详情中的编辑、删除，使用同一个 `DirectionalAction`。可以分享一个很小的组件怎样统一交互，而不接管业务回调或异步任务。

## 源码已有的行为

接口继承原生 `ButtonHTMLAttributes<HTMLButtonElement>`，只收回组件需要统一管理的 `type`、`children` 和 `aria-label`。`label` 必填，`variant` 选择图标和配色，`loading` 由父组件控制。

源码 `app/components/DirectionalAction.tsx:61–72` 摘录：

```tsx
return <button
  {...props}
  type="button"
  className={classes}
  aria-label={label}
  aria-busy={loading || undefined}
  disabled={disabled || loading}
>
  <span className="directional-action-icon" aria-hidden="true">
    <ActionIcon variant={variant} />
  </span>
</button>;
```

固定 `type="button"` 避免在表单里意外提交。图标对辅助技术隐藏，操作名来自 `label`。加载时按钮禁用，并用 CSS 伪元素显示转圈；原样式延迟 150ms 才开始加载动画。

接入示意：

```tsx
<DirectionalAction
  variant="commit"
  label="保存记录"
  loading={saving}
  disabled={!canSave}
  onClick={saveRecord}
/>
```

这里的 `saving`、`canSave`、`saveRecord` 均由调用方提供。

## 复用边界

依赖 `app/components/SeedIcon.tsx` 和 `app/globals.css` 的 `.directional-action*`、颜色变量、焦点和减少动态规则。原按钮宽度和最小高度均为 48px。产品专用的种子图标可在提取时替换。

`loading` 只是视觉与禁用状态，不会自动追踪 Promise，也不负责处理错误。涉及写入的业务仍需要自己的提交状态与重复调用保护，不能把“按钮变灰”当成完整的幂等机制。

## 验证线索

`tests/browser/capture-growth-navigation.spec.ts` 包含记录流程方向按钮的集成检查。独立提取时应检查在表单内不会提交、禁用和加载时不可激活、每种图标都有准确操作名，以及减少动态时仍能辨认忙碌状态。本次只读查阅，未运行测试。

[返回目录](../README.md)
