# Looper 的退出与消息循环的边界

> 系列：Framework-Source-Note · handler
> 难度：⭐⭐⭐⭐ 深入
> 更新：2026-08-27
> 前置知识：《Handler 消息机制》《HandlerThread》

---

## 一、Looper 是个「死循环」

前面讲过，Looper.loop() 是一个 `for(;;)` 无限循环。那么问题来了：

- 主线程的 Looper 永远不会退出（退出 = 应用结束）
- 但子线程的 Looper，用完得能退出，否则线程永远不结束

**Looper 的退出机制**，就是解决「什么时候、怎么安全地停止这个死循环」。

## 二、quit vs quitSafely

Looper 提供两个退出方法：

| 方法 | 行为 | 适用 |
|------|------|------|
| `quit()` | 立即退出，清空队列剩余消息 | 不再需要任何处理 |
| `quitSafely()` | 处理完当前消息，再退出 | 希望剩余消息执行完 |

```java
// 立即退出
looper.quit();

// 安全退出（推荐）
looper.quitSafely();
```

## 三、退出的底层原理

两个方法最终都会调用 `MessageQueue.quit(boolean safe)`：

```java
void quit(boolean safe) {
    if (!mQuitAllowed) {
        throw new IllegalStateException("Main thread not allowed to quit.");
    }
    if (safe) {
        removeAllFutureMessagesLocked();  // 移除未来消息，保留当前
    } else {
        removeAllMessagesLocked();        // 移除所有消息
    }
    mQuitting = true;
}
```

**关键点**：
1. `mQuitAllowed`：主线程的 Looper 不允许退出（`quit` 会抛异常）。
2. 退出后 `next()` 返回 null，`loop()` 结束。

## 四、主线程为什么不能退出

主线程（ActivityThread）创建 Looper 时，明确禁止退出：

```java
Looper.prepareMainLooper();  // 主线程 Looper 的 mQuitAllowed = false
```

因为主线程的 Looper 一旦退出，整个 App 就无法再响应任何消息，等于「假死」。

## 五、退出后的状态

Looper 退出后：

1. `loop()` 结束，线程的 `run()` 方法返回。
2. 再往这个 Handler 发消息，会失败（`sendMessage` 返回 false）。
3. 不能再复用这个 Looper（一个线程只能 prepare 一次）。

```java
// Looper 退出后再发消息
boolean success = handler.sendMessage(msg);
// success = false，消息不会被执行
```

## 六、实际场景：HandlerThread 的退出

```java
HandlerThread thread = new HandlerThread("worker");
thread.start();

Handler handler = new Handler(thread.getLooper());

// ... 使用 handler 处理任务 ...

// 不用了，安全退出
handler.getLooper().quitSafely();

// 或者
thread.quitSafely();
```

**推荐**：用 `quitSafely()` 而不是 `quit()`，让已经排队的任务能执行完，避免丢任务。

## 七、一个容易踩的坑

在 `onDestroy` 里退出 Looper 后，如果还有异步任务没完成，可能出问题：

```java
@Override
protected void onDestroy() {
    super.onDestroy();
    thread.quitSafely();  // 退出后，pending 的任务可能不执行
    // 如果有回调要更新 UI，可能已经无效
}
```

**建议**：退出前确保没有关键任务依赖这个 Looper，或用 `quitSafely` 让已排队的执行完。

## 八、总结

| 要点 | 结论 |
|------|------|
| quit vs quitSafely | 立即退出 vs 处理完再退出 |
| 主线程 Looper | 不允许退出 |
| 退出后 | 不能发消息，不能复用 |
| 最佳实践 | 用 quitSafely，避免丢任务 |

---

**下一篇预告**：《主线程卡顿检测与 BlockCanary 原理》

> 配套仓库：[Framework-Source-Note](https://github.com/jason5200/Framework-Source-Note)
