<div align="center">

# Framework-Source-Note 📚

**Android Framework 源码精读笔记 —— Binder / Handler / AMS / WMS**

[![Stars](https://img.shields.io/github/stars/jason5200/Framework-Source-Note?style=social)](https://github.com/jason5200/Framework-Source-Note)
[![Forks](https://img.shields.io/github/forks/jason5200/Framework-Source-Note?style=social)](https://github.com/jason5200/Framework-Source-Note)
[![License](https://img.shields.io/github/license/jason5200/Framework-Source-Note)](https://github.com/jason5200/Framework-Source-Note)
[![Visitors](https://komarev.com/ghpvc/?username=jason5200&repo=Framework-Source-Note&color=blueviolet)](https://github.com/jason5200/Framework-Source-Note)

</div>

---

## 📌 为什么有这个仓库

车载 Android、Framework 开发绕不开源码，但 AOSP 源码浩瀚，直接硬啃效率极低。本仓库的目标是：**用「一条主线 + 分层拆解 + 图解」的方式，把 Android Framework 的核心机制讲透**。

写作原则：**结论先行，源码佐证，图优先于文字。**

## 🧭 内容主线

```
Binder（进程通信的基石）
   │
   ▼
Handler（消息机制）
   │
   ▼
AMS / WMS（系统服务核心）
```

## 🗂️ 目录结构

```
Framework-Source-Note/
├── README.md           # 本文件
├── binder/             # Binder 机制：概述、驱动、Proxy/Stub、一次通信全流程
├── handler/            # Handler 消息机制：MessageQueue、Looper、同步屏障、IdleHandler
├── ams-wms/            # AMS/WMS 核心服务、Choreographer 渲染
├── view/               # View 体系：事件分发、绘制流程
├── classloader/        # 类加载机制、双亲委派、热修复
└── assets/             # 图片、时序图
```

## 📚 系列文章

| 序号 | 目录 | 文章 | 状态 |
|------|------|------|------|
| 01 | binder | 《Binder 机制总览：为什么 Android 用它做 IPC》 | ✅ 已发布 |
| 02 | binder | 《一次 Binder 通信的完整流程》 | ✅ 已发布 |
| 03 | binder | 《Binder 驱动层深入》 | ✅ 已发布 |
| 04 | binder | 《Binder 连接池与多线程并发》 | ✅ 已发布 |
| 05 | binder | 《AIDL 深入：in/out/inout 与 Parcelable》 | ✅ 已发布 |
| 06 | binder | 《Binder 的 oneway 异步调用》 | ✅ 已发布 |
| 07 | handler | 《Handler 消息机制：从 Looper 到 MessageQueue》 | ✅ 已发布 |
| 08 | handler | 《消息队列与 IdleHandler》 | ✅ 已发布 |
| 09 | handler | 《同步屏障与异步消息》 | ✅ 已发布 |
| 10 | handler | 《HandlerThread 与 IntentService》 | ✅ 已发布 |
| 11 | handler | 《Looper 的退出与消息循环的边界》 | ✅ 已发布 |
| 12 | handler | 《主线程卡顿检测与 BlockCanary 原理》 | ✅ 已发布 |
| 13 | ams-wms | 《AMS 启动流程解析》 | ✅ 已发布 |
| 14 | ams-wms | 《WMS 窗口管理解析》 | ✅ 已发布 |
| 15 | ams-wms | 《Choreographer 与渲染机制》 | ✅ 已发布 |
| 16 | view | 《View 事件分发机制》 | ✅ 已发布 |
| 17 | view | 《View 绘制流程：measure / layout / draw》 | ✅ 已发布 |
| 18 | classloader | 《类加载机制：ClassLoader 与双亲委派》 | ✅ 已发布 |
| 19 | component | 《Activity 启动模式与任务栈》 | ✅ 已发布 |
| 20 | component | 《Activity 生命周期与异常恢复》 | ✅ 已发布 |
| 21 | component | 《Service 的启动与绑定机制》 | ✅ 已发布 |
| 22 | component | 《BroadcastReceiver 的动态注册与分发》 | ✅ 已发布 |
| 23 | component | 《ContentProvider 的原理与使用》 | ✅ 已发布 |
| 24 | component | 《Window 与 WindowManager 体系》 | ✅ 已发布 |

## 🚀 阅读建议

```bash
git clone https://github.com/jason5200/Framework-Source-Note.git
cd Framework-Source-Note
# 建议顺序：binder → handler → ams-wms
```

## 🤝 参与共建

欢迎 PR 纠错与补充，特别是源码版本标注（AOSP 版本不同，行号可能有差异）。

## 📄 License

[Apache-2.0](LICENSE) © [jason5200](https://github.com/jason5200)
