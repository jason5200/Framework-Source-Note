# 消息队列与 IdleHandler：主线程空闲时的秘密武器

> 系列：Framework-Source-Note · handler
> 难度：⭐⭐⭐ 进阶
> 更新：2026-08-26
> 前置知识：《Handler 消息机制》

---

## 一、MessageQueue 再认识

上一篇《Handler 消息机制》讲过，MessageQueue 是一个按时间排序的单链表。这里深入两个被忽略的细节：

1. **它不只是「队列」**：`next()` 取消息时，如果队首消息还没到时间，会阻塞等待。
2. **它有「空闲」概念**：当队列里没有需要立即处理的消息时，主线程进入空闲状态。

而 **IdleHandler** 正是利用「空闲」这个时机来执行任务的机制。

## 二、IdleHandler 是什么

IdleHandler 是一个接口，在 MessageQueue 空闲时被回调：

```java
public interface IdleHandler {
    boolean queueIdle();  // 返回 true 表示保留，下次空闲继续调用；false 表示移除
}
```

**核心理解**：IdleHandler 里的任务，会在**主线程没有消息需要处理时**执行。它不是立即执行，而是「等主线程闲下来再干」。

## 三、next() 里的空闲检测

IdleHandler 的触发时机藏在 `MessageQueue.next()` 里：

```java
Message next() {
    int pendingIdleHandlerCount = -1;
    for (;;) {
        // ... 处理消息、处理屏障 ...

        // 没有要处理的消息了，触发 IdleHandler
        if (pendingIdleHandlerCount < 0 && (mMessages == null || now < mMessages.when)) {
            pendingIdleHandlerCount = mIdleHandlers.size();
        }
        if (pendingIdleHandlerCount <= 0) {
            continue;
        }

        // 取出并执行 IdleHandler
        for (int i = 0; i < pendingIdleHandlerCount; i++) {
            final IdleHandler idler = mIdleHandlers.get(i);
            boolean keep = idler.queueIdle();  // 执行
            if (!keep) {
                mIdleHandlers.remove(idler);   // 返回 false 则移除
            }
        }
        pendingIdleHandlerCount = 0;
    }
}
```

**关键点**：
- 只有当 `mMessages == null`（队列空）或 `now < mMessages.when`（下条消息还没到时间）时，才触发 IdleHandler。
- 返回 `true` 的 IdleHandler 会**保留**，下次空闲继续调用；返回 `false` 的会被移除。

## 四、IdleHandler 的典型应用场景

### 1. 延迟初始化（性能优化利器）

把不紧急的初始化放到主线程空闲时，避免阻塞启动：

```java
Looper.getMainLooper().queue.addIdleHandler(new MessageQueue.IdleHandler() {
    @Override
    public boolean queueIdle() {
        // 主线程空闲了，做点不紧急的事
        initNonCriticalSdk();
        return false;  // 只执行一次
    }
});
```

这在**应用启动优化**里非常常见：核心初始化先做，非核心的等空闲再做。

### 2. 首帧之后再执行

有些任务不急于在首帧前完成，可以用 IdleHandler 推迟到首帧渲染后：

```java
Looper.myQueue().addIdleHandler(() -> {
    preloadNextPageData();  // 首帧后预加载下一页
    return false;
});
```

### 3. Activity 启动优化

在 Activity 的 onCreate/onResume 之后，用 IdleHandler 做非关键任务：

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    // 关键 UI 初始化 ...

    // 非关键的，等主线程空闲再做
    Looper.myQueue().addIdleHandler(() -> {
        loadAdBanner();       // 加载广告
        reportAnalytics();    // 上报统计
        return false;
    });
}
```

## 五、IdleHandler vs postDelayed

| 方式 | 时机 | 适用场景 |
|------|------|----------|
| `IdleHandler` | 主线程空闲时 | 不紧急、可推迟的任务 |
| `postDelayed` | 固定延迟后 | 需要精确时机的任务 |

**关键区别**：IdleHandler 的时机是「不确定的」——它取决于主线程什么时候空闲。如果主线程一直忙，IdleHandler 可能**很久都不会执行**。

所以**不要用 IdleHandler 做有时间要求的任务**，它只适合「越早越好，但晚了也行」的事。

## 六、使用注意事项

1. **不要做耗时操作**：IdleHandler 在主线程执行，耗时操作会再次阻塞主线程，本末倒置。
2. **注意返回值的含义**：返回 true 会反复执行，可能造成死循环（每次都做任务，主线程永远"忙"）。
3. **时机不确定**：主线程一直忙时，IdleHandler 不执行。

## 七、一个完整示例

```java
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // 首帧后执行非关键初始化
        Looper.myQueue().addIdleHandler(new MessageQueue.IdleHandler() {
            @Override
            public boolean queueIdle() {
                // 这里在主线程空闲时执行
                initNonCritical();
                return false;  // 执行一次就移除
            }
        });
    }
}
```

## 八、总结

| 要点 | 结论 |
|------|------|
| IdleHandler 是什么 | 主线程空闲时执行的回调 |
| 触发时机 | MessageQueue 无立即消息时 |
| 返回值 | true 保留，false 移除 |
| 典型应用 | 延迟初始化、首帧后任务 |
| 注意 | 不做耗时操作、时机不确定 |

---

**下一篇预告**：《类加载机制：ClassLoader 与双亲委派》

> 配套仓库：[Framework-Source-Note](https://github.com/jason5200/Framework-Source-Note)
