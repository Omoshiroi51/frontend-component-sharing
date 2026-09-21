# 07 · 把蜂窝布局、镜头和鱼眼效果分开计算

> 状态：源码解读。来源版本见[来源记录](../docs/sources.md)。

## 适合分享什么

主题球群可以平移、缩放，并在视口中心形成鱼眼放大效果。这一实现最值得分享的是坐标分层：主题排在哪里、镜头看哪里、屏幕怎样变形，各自有明确计算。

## 源码已有的结构

| 层次 | 入口 | 负责什么 |
| --- | --- | --- |
| 世界布局 | `buildHoneycomb` | 创建稳定蜂窝槽位，再按 y、x 阅读顺序分配 ID |
| 镜头 | `Camera` | 保存平移 x、y 和 zoom |
| 屏幕投影 | `projectNode` | 计算中心鱼眼变形、球半径、聚焦程度和透明度 |
| 逆变换 | `unwarpPoint`、`worldAtPoint`、`cameraAtAnchor` | 从屏幕位置反推世界锚点，使缩放围绕可见目标进行 |
| 边界与惯性 | `cameraBounds`、`rubberBand`、`advanceCamera` | 限制范围、越界阻力、速度衰减和回收 |
| React 交互 | `TopicSphereCluster` | 绑定 Pointer Events、键盘、排序草稿和业务回调 |

前五层都在 `app/components/topic-honeycomb.ts`，不导入 React、DOM 或智慧业务模型。镜头推进函数会修改传入的 camera 与 velocity，所以该模块是独立计算模块，并非每个函数都无副作用。

### 为什么布局排序与形状要分开

蜂窝先从中心向外生成位置集合，再按基础 y、x 坐标给主题分配槽位。七个节点仍保持 `2 / 3 / 2` 外形；阅读顺序改变的是 ID 与位置的对应关系，不是重新按视口宽度排成矩形网格。

### 为什么缩放要先反解鱼眼

屏幕上的点已经经过鱼眼变形。直接用线性镜头公式反推，会让指针下的目标在缩放时漂移。`unwarpPoint` 用 24 次二分反解径向变形，随后才转换世界坐标。

源码 `app/components/topic-honeycomb.ts:130–138` 摘录：

```ts
export function worldAtPoint(point: Point, camera: Camera, viewport: Viewport): Point {
  const base = unwarpPoint(point, viewport);
  return { x: (base.x - camera.x) / camera.zoom, y: (base.y - camera.y) / camera.zoom };
}

export function cameraAtAnchor(world: Point, screen: Point, zoom: number, viewport: Viewport): Camera {
  const base = unwarpPoint(screen, viewport);
  return { x: base.x - world.x * zoom, y: base.y - world.y * zoom, zoom };
}
```

### 当前手势合同

普通拖动平移，双指或滚轮缩放。允许编辑时，约 400ms 长按进入持续布局编辑；拖动松手交换两个节点，可以继续调整，单击球或空白处统一保存退出。Escape / 取消放弃整轮草稿，保存失败保留草稿。同一成员集合内排序后保留镜头。

这些行为以当前 `TopicSphereCluster.tsx` 为准；不能把早期文档的“每次松手立即保存”当成当前实现。

## 复用边界

建议第一篇只提取几何模块，使用简单圆形演示平移和缩放。完整组件还依赖主题排序、植物呈现、保存接口、焦点定位和大量 `.topic-sphere-*` CSS；鼠标、触摸和键盘都需要整体验证。

镜头属于会话状态，没有写入业务数据。几何函数也不替调用者校验唯一 ID、正数视口和缩放范围；原组件在调用侧处理测量与限幅，独立使用时需要保留这些约束。

`advanceCamera` 把单次经过时间限制为 48ms，再以最多 8ms 子步推进。分享时可解释这如何控制长帧影响，不应承诺任意规模都流畅或所有设备有相同性能。

## 验证线索

原项目有 `tests/browser/topic-sphere-desktop.spec.ts`、`tests/browser/topic-sphere-mobile.spec.ts` 和 `tests/browser/topic-sphere-static.spec.ts`。本次未运行。

独立算法重点检查：节点数量和位置唯一、阅读顺序、锚点缩放误差、空集合、单节点、边界及惯性停止。完整组件还要检查多指介入取消当前拖动、排序失败、减少动态、键盘定位和静态后备。

[返回目录](../README.md)
