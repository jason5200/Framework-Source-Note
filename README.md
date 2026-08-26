<div align="center">

# Framework-Source-Note

**Android Framework 源码笔记 —— Binder / Handler / AMS / WMS / View**

[![Stars](https://img.shields.io/github/stars/jason5200/Framework-Source-Note?style=social)](https://github.com/jason5200/Framework-Source-Note)
[![License](https://img.shields.io/github/license/jason5200/Framework-Source-Note)](https://github.com/jason5200/Framework-Source-Note)
[![AOSP](https://img.shields.io/badge/AOSP-android--14.0.0__r67-green)](AOSP_VERSION.md)

</div>

---

## 为什么有这个仓库

AOSP 太大，直接硬啃效率低。这里按一条主线拆：**Binder → Handler → AMS/WMS → View**。源码默认对照 [android-14.0.0_r67](AOSP_VERSION.md)。

写作方式：结论先行，再跟源码路径；行号以该标签为准。

## 怎么读

```
Binder（IPC）
   │
   ▼
Handler / Looper / MessageQueue
   │
   ▼
AMS / WMS / Choreographer
   │
   ▼
View 测量、布局、绘制、事件
```

建议先读 `binder/binder-overview.md` 和 `handler/handler-message-mechanism.md`，再进 AMS。

## 目录结构

```
Framework-Source-Note/
├── README.md
├── AOSP_VERSION.md
├── binder/          # Binder 驱动、一次拷贝、AIDL、线程池
├── handler/         # Looper、MessageQueue、同步屏障
├── ams-wms/         # AMS / WMS / Choreographer / ANR
├── view/            # 事件分发与绘制
├── component/       # 四大组件与 Window
└── classloader/     # ClassLoader 与热修复
```

## 系列文章（48 篇）

### Binder

| 文章 | 文件 |
|------|------|
| Binder 机制总览 | [binder-overview.md](binder/binder-overview.md) |
| 一次 Binder 通信全流程 | [binder-full-flow.md](binder/binder-full-flow.md) |
| Binder 驱动层 | [binder-driver.md](binder/binder-driver.md) |
| mmap 一次拷贝 | [binder-mmap-deep.md](binder/binder-mmap-deep.md) |
| binder_transaction | [binder-transaction-source.md](binder/binder-transaction-source.md) |
| binder_proc / binder_thread | [binder-struct-source.md](binder/binder-struct-source.md) |
| 连接池与并发 | [binder-pool.md](binder/binder-pool.md) |
| Binder 线程池 | [binder-threadpool-source.md](binder/binder-threadpool-source.md) |
| AIDL：in/out/inout | [aidl-deep.md](binder/aidl-deep.md) |
| AIDL 生成代码 | [aidl-generated.md](binder/aidl-generated.md) |
| oneway 异步调用 | [oneway.md](binder/oneway.md) |

### Handler

| 文章 | 文件 |
|------|------|
| Handler 消息机制 | [handler-message-mechanism.md](handler/handler-message-mechanism.md) |
| native 层唤醒 | [handler-native-wakeup.md](handler/handler-native-wakeup.md) |
| Looper C++ | [looper-cpp.md](handler/looper-cpp.md) |
| Looper epoll | [looper-epoll-source.md](handler/looper-epoll-source.md) |
| MessageQueue native next | [messagequeue-source.md](handler/messagequeue-source.md) |
| IdleHandler | [idlehandler.md](handler/idlehandler.md) |
| 同步屏障 | [sync-barrier.md](handler/sync-barrier.md) |
| HandlerThread | [handlerthread.md](handler/handlerthread.md) |
| Looper 退出 | [looper-exit.md](handler/looper-exit.md) |
| BlockCanary 原理 | [blockcanary.md](handler/blockcanary.md) |

### AMS / WMS

| 文章 | 文件 |
|------|------|
| AMS 启动流程 | [ams-startup.md](ams-wms/ams-startup.md) |
| AMS 进程管理源码 | [ams-process-source.md](ams-wms/ams-process-source.md) |
| WMS 窗口管理 | [wms-window.md](ams-wms/wms-window.md) |
| Choreographer | [choreographer.md](ams-wms/choreographer.md) |
| 内存泄漏与 LeakCanary | [memory-leak.md](ams-wms/memory-leak.md) |
| ANR 原理 | [anr.md](ams-wms/anr.md) |

### 组件与 View

| 文章 | 文件 |
|------|------|
| Activity 启动模式 | [launch-mode.md](component/launch-mode.md) |
| 生命周期与异常恢复 | [lifecycle-restore.md](component/lifecycle-restore.md) |
| Service 启动与绑定 | [service-bind.md](component/service-bind.md) |
| Service 源码 | [service-source.md](component/service-source.md) |
| BroadcastReceiver | [broadcast.md](component/broadcast.md) |
| ContentProvider | [contentprovider.md](component/contentprovider.md) |
| ContentProvider 跨进程 | [contentprovider-source.md](component/contentprovider-source.md) |
| Window / WindowManager | [window.md](component/window.md) |
| 事件分发 | [touch-event.md](view/touch-event.md) |
| 事件分发源码 | [touch-source.md](view/touch-source.md) |
| measure / layout / draw | [draw-process.md](view/draw-process.md) |
| measure 源码 | [measure-source.md](view/measure-source.md) |
| layout / draw 源码 | [layout-draw-source.md](view/layout-draw-source.md) |
| ViewRootImpl | [viewrootimpl-source.md](view/viewrootimpl-source.md) |
| 硬件加速 | [hw-render-source.md](view/hw-render-source.md) |
| invalidate / requestLayout | [invalidate-requestlayout.md](view/invalidate-requestlayout.md) |
| 滑动冲突 | [scroll-conflict.md](view/scroll-conflict.md) |
| RecyclerView 缓存 | [recyclerview.md](view/recyclerview.md) |
| RecyclerView 源码 | [recyclerview-source.md](view/recyclerview-source.md) |
| ClassLoader | [classloader.md](classloader/classloader.md) |
| 热修复：Tinker / Sophix | [hotfix.md](classloader/hotfix.md) |

## 参与共建

欢迎核对 [AOSP_VERSION.md](AOSP_VERSION.md) 中的标签并修正路径/行号。流程见 [CONTRIBUTING.md](CONTRIBUTING.md)。

## License

[Apache-2.0](LICENSE) © [jason5200](https://github.com/jason5200)
