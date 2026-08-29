"Windows 拉取 iPhone 应用日志"步骤如下。

## 前置条件（一次性）

1. 电脑装 iTunes/Apple Mobile Device Support 驱动（提供 usbmux 通道）
2. 装 Python 3.8+ 和 pymobiledevice3：

```powershell
pip install pymobiledevice3
```

3. iPhone 数据线连接电脑，解锁并点"信任此电脑"（首次）

## 核心流程（每次拉取）

**第 1 步：确认设备可见**

```powershell
pymobiledevice3 usbmux list
```

输出含 `DeviceClass: iPhone` 即正常；空数组说明未解锁/未信任/驱动问题。

**第 2 步：找到目标 app 的 bundle id**

```powershell
pymobiledevice3 apps list
```

在输出里找你要排查的 app，记下 `CFBundleIdentifier`（形如 `com.xxx.app`）。

**第 3 步：列出 app 沙盒里的日志文件**

```powershell
$env:PYTHONIOENCODING='utf-8'
"ls /Documents" | pymobiledevice3 apps afc <bundle_id> --documents
```

常见日志位置：`/Documents`、`/Documents/Logs`、`/Library/Logs`、`/Documents/<项目自定义日志目录>`。app 自己写的日志（文本文件、log、sqlite 等）都在这里。

**第 4 步：拉取文件到本地**

```powershell
pymobiledevice3 apps pull <bundle_id> "/Documents/Logs/xxx.log" "D:\logs\xxx.log" --documents
```

## 可选：崩溃日志和系统日志

**app 崩溃报告**（`.ips` 文件，设备诊断目录，无需沙盒）：

```powershell
pymobiledevice3 crash ls
pymobiledevice3 crash pull /App-xxx-2026-08-28.ips D:\logs\crash.ips
```

**系统层日志**（只能看到系统进程，看不到 app 内部日志）：

```powershell
pymobiledevice3 syslog collect D:\logs
```

## 三条关键限制

1. **app 必须把日志写进了文件**才能拉——很多 app 只打 NSLog/os_log 不落盘，这类 app 用沙盒拉不到业务日志，唯一出路是开开发者模式（`pymobiledevice3 developer`，需手机重启）或 Mac/Xcode
2. **只能访问目标 app 自己的沙盒**，拿不到别的 app 或系统日志内容
3. iOS 没有 Android logcat 的等价物：syslog 流（`syslog live`）对 app 进程内部日志默认不可见，这是系统隐私机制，不是命令问题

这套流程对任何 iOS app 通用，核心就四步：usbmux 检测 → apps list 找包名 → afc 列目录 → pull 拉文件。