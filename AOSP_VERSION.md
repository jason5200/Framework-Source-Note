# 源码对照版本

本仓库叙述默认对照：

| 项 | 值 |
|----|----|
| AOSP 标签 | `android-14.0.0_r67` |
| 对应 | Android 14 |
| 代码浏览 | https://cs.android.com/android/platform/superproject/+/android-14.0.0_r67: |

常用路径（均相对 AOSP 根目录）：

| 主题 | 路径 |
|------|------|
| Binder 驱动 | `kernel/common/drivers/android/binder.c`（或设备内核树中的等价路径） |
| Binder native | `frameworks/native/libs/binder/` |
| AIDL / Java Binder | `frameworks/base/core/java/android/os/` |
| Handler / Looper / MessageQueue | `frameworks/base/core/java/android/os/` 与 `frameworks/base/core/jni/` |
| AMS | `frameworks/base/services/core/java/com/android/server/am/` |
| WMS | `frameworks/base/services/core/java/com/android/server/wm/` |
| ViewRootImpl | `frameworks/base/core/java/android/view/ViewRootImpl.java` |

行号随分支会变。提 PR 纠错时请注明你核对用的标签。
