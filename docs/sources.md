# 首批内容的来源记录

## 代码基线

| 项目 | 值 |
| --- | --- |
| 来源项目 | Wisdom Echo（智慧回响） |
| 本地位置 | 与本仓库同级的 `wisdom-echo` 目录 |
| 整理日期 | 2026-09-21 |
| 来源分支 | `v0.5.1` |
| 来源提交 | `deb88bc2d8507f38b9f91d9b68e3662b3c0be0b3` |
| 提交说明 | 重构：集中草稿箱会话并保护保存期间的新修改 |
| 整理开始时的工作区 | 无已跟踪修改或未忽略的未跟踪文件 |

来源项目的 `package.json` 声明了 React / React DOM `19.2.6`、TypeScript `5.9.3`、vinext `0.0.50`、Next `16.2.6` 和 Tailwind CSS `4.2.1`。这些是来源版本，不是本仓库的安装要求；首批笔记不需要安装依赖。实际开发命令使用 vinext，不能只根据 Next 依赖名称判断启动方式。

## 如何定位原始代码

所有笔记中的 `app/...`、`tests/...` 路径都相对来源项目。行号对应上述提交，未来源码变化后应优先使用固定提交查阅。

从本仓库根目录运行下面的只读命令，可查看某个文件在整理时的版本：

```sh
GIT_OPTIONAL_LOCKS=0 git -C ../wisdom-echo show deb88bc2d8507f38b9f91d9b68e3662b3c0be0b3:app/components/ModeSwitch.tsx
```

| 笔记 | 主要源码入口 |
| --- | --- |
| [胶囊分段切换器](../components/01-mode-switch.md) | `app/components/ModeSwitch.tsx` |
| [图标按钮](../components/02-directional-action.md) | `app/components/DirectionalAction.tsx`、`app/components/SeedIcon.tsx` |
| [底部导航](../components/03-bottom-navigation.md) | `app/components/BottomNavigation.tsx` |
| [分类浮层](../components/04-category-popover.md) | `app/components/CategorySelectionPopover.tsx` |
| [植物图片后备](../components/05-plant-visual.md) | `app/components/TopicPlantVisual.tsx`、`app/wisdom/plant-assets.ts` |
| [背景纹理](../components/06-imprint-pattern.md) | `app/components/WisdomImprintPattern.tsx`、`app/wisdom/imprint-appearance.ts` |
| [蜂窝鱼眼](../components/07-honeycomb-camera.md) | `app/components/topic-honeycomb.ts`、`app/components/TopicSphereCluster.tsx` |
| [残留点击防护](../components/08-orphan-click-guard.md) | `app/components/suppress-orphan-click.ts`、`app/components/TopicDetail.tsx` |

这些组件的样式主要位于 `app/globals.css`，包括颜色变量、焦点样式、响应式规则和减少动态规则。只复制 TSX 文件通常无法得到原来的显示效果。

## 本次整理的边界

本次只读检查来源代码、调用位置、样式和部分测试内容，并在当前仓库整理说明与少量源码摘录。各篇的“验证线索”用于说明提取后该验证什么；引用原项目测试不代表本次运行过测试，也不代表组件已经通过独立提取验收。

验证范围是本仓库的文档链接、源码定位、摘录一致性和 Git 状态；另外核对来源工作区及文件内容指纹。原项目的构建、测试和素材脚本未在本次整理中运行。

本次核对通过：35 个内部链接、30 个来源文件路径、8 段源码摘录及行号均有效。来源项目 575 个 Git 已跟踪及未忽略文件的内容与权限指纹前后一致，来源 HEAD、索引指纹及工作区状态也保持一致。

当前内容没有设置新的开源许可证，也没有对原项目代码或图片作新的授权声明。

[返回目录](../README.md)
