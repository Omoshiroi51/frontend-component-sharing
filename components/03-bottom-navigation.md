# 03 · 用一个滑块表现底部导航的选中状态

> 状态：源码解读。来源版本见[来源记录](../docs/sources.md)。

## 适合分享什么

智慧回响的“记录 / 回响 / 我的”底部导航，在三个固定位置之间移动同一个背景指示器。适合分享 CSS 位移动画、手机底部安全区，以及装饰动画如何与导航语义配合。

## 源码已有的行为

`app/components/BottomNavigation.tsx` 接受 `activeTab` 和 `onNavigate`。外层 `nav` 标注主导航，当前按钮使用 `aria-current="page"`，组件只通知目标，不负责路由、返回栈或离开保护。

`app/globals.css:341–342` 的位置规则摘录：

```css
.bottom-nav[data-active-tab="echo"] .bottom-nav-indicator { transform: translate3d(calc(100% + 6px),0,0); }
.bottom-nav[data-active-tab="garden"] .bottom-nav-indicator { transform: translate3d(calc(200% + 12px),0,0); }
```

两条规则分别移动一格、两格，并补偿网格间距。背景指示器的过渡使用 `560ms cubic-bezier(.2,.86,.28,1.08) 60ms`，是 CSS 缓动曲线，不是运行时弹簧模拟。

导航本体用 `env(safe-area-inset-bottom)` 避开底部安全区；单独的 `.bottom-nav-layer` 使用渐变 mask 和 backdrop blur，使内容在接近底栏时逐渐虚化，并通过 `pointer-events: none` 保持装饰层不拦截操作。

## 复用边界

需要一起提取 `.bottom-nav*`、图标样式、相关关键帧、减少动态规则和 `SeedIcon`。原 CSS 固定为三列，指示器宽度也绑定三项布局；改变导航数量需要同步调整。

`garden` 是遗留内部键，当前显示文案已经是“我的”，分享时不要误称仍有“花园”一级页面。完整页面的底部留白由页面布局负责，复制导航不能自动防止正文被遮挡。

`EchoCircleIcon` 内的 SVG 渐变、滤镜和遮罩使用固定 ID。若提取后在同页渲染多个带流体效果的实例，需要为每个实例生成唯一 ID，并同步更新 `url(#...)` 引用。

## 验证线索

`tests/browser/bottom-navigation.spec.ts` 已编写选中态、指示器移动、渐变蒙版和减少动态的用例。本次未运行。提取时重点复核手机安全区、正文末尾可达性、键盘焦点和减少动态时流体装饰静止。

[返回目录](../README.md)
