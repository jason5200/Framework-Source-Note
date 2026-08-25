# 一次 Binder 通信的完整流程

> 系列：Framework-Source-Note · binder
> 难度：⭐⭐⭐⭐ 深入
> 更新：2026-08-17
> 前置知识：《Binder 机制总览》

---

## 一、本文目标

上一篇讲了 Binder 的「是什么」和「为什么」，这一篇深入「怎么发生」。我们会沿着**一次跨进程方法调用**，走完从 Java 层到内核驱动再回来的完整链路。

用一句话概括全程：

> **Client 的 Proxy 把参数打包，通过 Binder 驱动跨进程传给 Server 的 Stub，Stub 拆包后执行真正的方法，再把结果原路返回。**

## 二、从一次调用说起

假设 App 要启动一个 Activity，最终会调用到 AMS（`system_server` 进程）的 `startActivity`。在 Java 层，这段调用看起来是这样：

```java
// Client 进程（App）
ActivityTaskManager.getService().startActivity(...);
```

`getService()` 返回的其实是一个 **Proxy 对象**，它实现了 `IActivityTaskManager` 接口。我们调用的 `startActivity` 是这个 Proxy 的方法。

## 三、Proxy 层：打包参数（Parcel）

Proxy 的 `startActivity` 实现（简化）大致如下：

```java
@Override
public int startActivity(...) {
    // 1. 创建两个 Parcel：data 存参数，reply 存返回值
    Parcel data = Parcel.obtain();
    Parcel reply = Parcel.obtain();
    int result;

    try {
        // 2. 写入 Binder 接口描述符
        data.writeInterfaceToken(DESCRIPTOR);
        // 3. 把参数依次写入 Parcel
        data.writeInt(callingPid);
        data.writeInt(callingUid);
        intent.writeToParcel(data, 0);
        // ...

        // 4. 发起跨进程调用（关键一步！）
        mRemote.transact(TRANSACTION_startActivity, data, reply, 0);

        // 5. 从 reply 读取返回值
        reply.readException();
        result = reply.readInt();
    } finally {
        data.recycle();
        reply.recycle();
    }
    return result;
}
```

**核心理解**：Proxy 只做两件事——**把参数序列化到 Parcel**，然后调用 `transact()`。它自己并不执行业务逻辑。

## 四、transact 到内核：binder 驱动

`mRemote.transact(...)` 是个 native 调用，最终进入 **binder 驱动**（`/dev/binder`）。

驱动的关键动作：

1. **找到目标进程**：根据 Binder 引用（句柄）定位 Server 进程。
2. **拷贝数据**：把 `data` 从 Client 进程的用户空间，拷贝到内核的缓冲区（**只有这一次拷贝**）。
3. **唤醒 Server**：把「有请求来了」的通知放入 Server 的待处理队列。
4. **Client 阻塞等待**：`transact` 是同步调用，Client 会挂起等 Server 返回。

```
Client 进程        内核(binder 驱动)        Server 进程
    │                    │                    │
    │── transact ──────→ │                    │
    │   (数据拷贝到内核)    │                    │
    │                    │── 唤醒 ──────────→  │
    │     阻塞等待        │                    │
```

## 五、Stub 层：拆包并执行

Server 进程有一个 Binder 线程池，会不断从驱动读取请求。收到请求后，Stub 的 `onTransact` 被调用：

```java
@Override
public boolean onTransact(int code, Parcel data, Parcel reply, int flags) {
    switch (code) {
        case TRANSACTION_startActivity: {
            // 1. 校验接口描述符
            data.enforceInterface(DESCRIPTOR);
            // 2. 从 Parcel 读取参数（顺序必须和写入时一致！）
            int callingPid = data.readInt();
            int callingUid = data.readInt();
            Intent intent = Intent.CREATOR.createFromParcel(data);
            // 3. 调用真正的业务逻辑
            int result = startActivity(callingPid, callingUid, intent);
            // 4. 把返回值写入 reply
            reply.writeNoException();
            reply.writeInt(result);
            return true;
        }
    }
    return super.onTransact(code, data, reply, flags);
}
```

**核心理解**：Stub 是 Server 端的入口，做的是 Proxy 的逆操作——**拆包、执行、回填结果**。

## 六、返回路径

Server 执行完，结果通过 `reply` 写回：

1. Stub 把 `reply` 交给驱动。
2. 驱动把 `reply` 拷贝回 Client 进程。
3. Client 的 `transact` 被唤醒，从 `reply` 读出结果。
4. 整个同步调用结束，Client 继续往下执行。

## 七、一次调用的完整时序图

```
Client(Proxy)          binder 驱动          Server(Stub)
    │                      │                    │
    │ 1. 序列化参数到 Parcel │                    │
    │ 2. transact() ──────→ │                    │
    │                      │ 3. 拷贝数据到内核    │
    │                      │ 4. 唤醒 Server ───→ │
    │ 5. 阻塞等待           │                    │ 6. onTransact()
    │                      │                    │ 7. 拆包、执行业务
    │                      │                    │ 8. 结果写入 reply
    │                      │ ← 9. 拷贝 reply ──── │
    │ 10. 醒来、读结果      │                    │
    │ 11. 返回给调用方      │                    │
```

## 八、几个关键细节

### 1. 为什么是「一次拷贝」

传统 IPC（如 Socket、管道）通常需要**两次拷贝**：发送方→内核→接收方。Binder 通过**内存映射（mmap）**，让内核缓冲区和接收方用户空间共享同一块物理内存，省掉了「内核→接收方」那次拷贝。

### 2. 为什么 transact 是同步的

`transact` 默认是同步阻塞调用，Client 会等 Server 返回。这是为了**语义简单**——跨进程调用像本地调用一样「调用即等待结果」。异步场景可以用 `oneway`（`FLAG_ONEWAY`）标记。

### 3. 参数顺序必须一致

Proxy 写入 Parcel 的顺序，和 Stub 读取的顺序必须**严格一致**。顺序错了会导致数据错乱甚至崩溃。这就是为什么 AIDL 会自动生成代码——它保证了读写顺序的一致性。

## 九、总结

| 阶段 | 谁 | 做了什么 |
|------|-----|----------|
| 打包 | Proxy | 参数序列化到 Parcel |
| 传输 | binder 驱动 | 一次拷贝 + 唤醒 Server |
| 拆包 | Stub | 读参数、执行业务 |
| 返回 | 原路 | 结果写回 reply，唤醒 Client |

**一句话记忆**：**Proxy 打包 → 驱动搬运 → Stub 拆包执行 → 结果原路返回。**

---

**下一篇预告**：《Handler 消息机制：从 Looper 到 MessageQueue》

> 配套仓库：[Framework-Source-Note](https://github.com/jason5200/Framework-Source-Note)
