# WMS 窗口管理解析

> 系列：Framework-Source-Note · ams-wms
> 难度：⭐⭐⭐⭐ 深入
> 前置知识：《AMS 启动流程解析》、Surface 基础

---

## 一、WMS 是什么

**WMS（WindowManagerService）** 是 Android 负责**窗口管理**的核心服务：

- 管理所有窗口的**层级（Z 轴顺序）**
- 管理窗口的**显示区域、大小、位置**
- 分发**输入事件**（触摸、按键）到正确的窗口
- 处理窗口的**添加、移除、动画**

一句话：**AMS 管「有哪些 Activity」，WMS 管「这些 Activity 的窗口怎么显示、谁在上谁在下」。**

## 二、Window 与 View 的关系

先厘清概念：

```
Window（窗口）          ← 抽象概念，一个界面区域
   │
   └── DecorView（根 View）
          │
          └── View 树（具体内容）
```

- 每个 Activity 有一个 **Window**（由 `PhoneWindow` 实现）。
- Window 里挂着一个 **DecorView**，它是 View 树的根。
- WMS 管理的是 Window 这一层，不关心 View 内部怎么画。

## 三、窗口的层级（Z-Order）

屏幕上多个窗口叠在一起，WMS 要决定**谁盖住谁**。这就是 Z-Order。

常见的窗口层级（从下到上）：

```
底部
 ├── 壁纸窗口
 ├── 应用窗口（普通 Activity）
 ├── 子窗口（PopupWindow、Dialog）
 ├── 系统窗口（Toast、状态栏）
 ├── 输入法窗口（IME）
 └── 顶部（如权限弹窗）
```

WMS 用一个 **WindowList**（按 Z 序排序的列表）维护所有窗口，每次变化都重排。

## 四、一次窗口添加的流程

以 Activity 显示为例：

```
1. Activity 创建，调用 WindowManager.addView()
        │
        ▼
2. ViewRootImpl.setView()（建立与 WMS 的连接）
        │  Binder 跨进程
        ▼
3. WMS.addWindow()（system_server 侧）
        │
        ├── 校验窗口参数
        ├── 计算窗口层级（Z-Order）
        ├── 分配 Surface
        │
        ▼
4. SurfaceFlinger 合成显示
```

**关键理解**：`addView` 最终会走到 WMS，WMS 给这个窗口分配一个 **Surface**，View 树画到 Surface 上，再由 SurfaceFlinger 合成到屏幕。

## 五、ViewRootImpl：连接 View 与 WMS 的桥梁

`ViewRootImpl` 是每个 Window 的核心对象，它：

- 持有 `WMS` 的 Binder 代理（`IWindowSession`）
- 负责 View 的**测量、布局、绘制**（performTraversals）
- 通过 `Choreographer` 驱动渲染（配合上一篇的同步屏障）

```
View 树
   │
   ▼
ViewRootImpl（performTraversals）
   │  measure → layout → draw
   ▼
Surface（绘制结果）
   │
   ▼
SurfaceFlinger → 屏幕
```

## 六、输入事件分发

WMS 的另一大职责：把触摸事件送到「最上层、能接收」的窗口。

```
用户触摸屏幕
   │
   ▼
InputManagerService 收到硬件事件
   │
   ▼
WMS 根据 Z-Order 找到目标窗口
   │
   ▼
通过 InputChannel 发送到目标窗口
   │
   ▼
ViewRootImpl 分发到具体 View
```

**核心理解**：触摸事件是「从外到内」的——先确定窗口（WMS 按 Z 序），再在窗口内确定 View（View 树按逆序遍历）。

## 七、WMS 与 SurfaceFlinger 的分工

| 服务 | 职责 |
|------|------|
| WMS | 管窗口的「逻辑」：层级、大小、位置、输入 |
| SurfaceFlinger | 管像素的「合成」：把各 Surface 合成到屏幕 |

WMS 决定「谁在哪、谁在上」，SurfaceFlinger 负责「把它们画出来」。两者通过 Surface 建立联系。

## 八、总结

| 要点 | 结论 |
|------|------|
| WMS 是什么 | 窗口管理服务，管层级/大小/位置/输入 |
| Window vs View | Window 是界面区域，View 是内容 |
| Z-Order | 窗口堆叠顺序，决定谁盖谁 |
| ViewRootImpl | 连接 View 与 WMS 的桥梁 |
| 输入分发 | 按 Z-Order 找窗口，再按 View 树找 View |
| 分工 | WMS 管逻辑，SurfaceFlinger 管合成 |

---

**下一篇预告**：《Choreographer 与渲染机制》

> 配套仓库：[Framework-Source-Note](https://github.com/jason5200/Framework-Source-Note)
