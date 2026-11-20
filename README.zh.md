# Razr Fold：主用户保持干净，Google 只放在专用用户中（中文版）

> **语言：** [English](README.md) · [Русский](README.ru.md) · **中文**（本文档）
>
> **许可协议：** [CC BY-ND 4.0](LICENSE) — 你可以免费复制、传播、复刻和打印本指南，但必须完整且未经修改，并保留作者和联系方式（GitHub 与 Telegram 链接）。不允许修改文本。

Nov 20, 2026 · @No

**本指南的原始出处：** [github.com/sikkatu/debloat-Motorola-Razr-Fold](https://github.com/sikkatu/debloat-Motorola-Razr-Fold)

**社交媒体上的原始出处（Telegram）：** 频道 [t.me/motorolarazrfold](https://t.me/motorolarazrfold)，聊天群 [t.me/motorolafold](https://t.me/motorolafold)

## 你会得到什么，代价是什么

主用户将变得干净：没有 Google，也没有 Moto 的垃圾应用。Google 只留在第二个用户“Google”中：你手动启动它，用一条命令关闭它。

**工作原理。** 命令 `pm uninstall -k --user 0 <package>` 不会擦除应用，而是把它从主用户中隐藏起来：文件仍留在系统分区，数据保留，你可以用 `cmd package install-existing` 把应用找回来。其他用户不受影响：在“Google”用户中，各项服务原样保留；Shelter 工作资料（如果你有的话）单独清理（步骤 11）。所以顺序是这样的：先创建 Google 用户并确认它能正常工作，然后才从主用户中移除 Google。

**哪些在更新后依然有效。** 隐藏状态保存在 `/data/system/users/0/package-restrictions.xml` 中。刷入内核（`fastboot flash boot`）不会动这个文件，普通的 A/B OTA 升级也不会。但在大版本更新之后，有些应用可能会回来，或者出现新的应用：步骤 10 会找出它们并再次移除。恢复出厂设置会让一切恢复原状，你需要把这些步骤重新做一遍。

**主用户中哪些功能将不再可用：**

- 通过 Google 推送的通知：没有自己长连接的应用，通知会延迟到达，甚至收不到；
- Play 商店及其更新：请从 F-Droid 和开发者网站安装应用；
- 基于 Wi-Fi 和基站的定位：只剩下 GPS，首次定位耗时更长；
- Google Pay 和 Wallet、“查找我的设备”、Android Auto、使用 Google 账号登录；
- 步骤 5 中的 Moto 功能：Moto AI 和 Qira AI 助手、Moto Unplugged、Family Space、Moto 的“游戏”“录音机”和“翻译”、Moto 的互动壁纸、Motorola Notifications 消息客户端，以及预装的 Clip Studio Paint；如果你还移除了第 4–5 项中的应用，则还包括 Moto 手势、Moto Secure、Smart Connect（连同它在 USB 模式菜单中的入口）和手写笔的 Quick Clip；
- Moto AI Services 和 Moto AI Core（步骤 5 第 6 项）：根据它们的说明，Moto 应用从中获取 AI 模型，用于“笔记”、“风格化”、优化充电和自适应亮度；具体哪些功能会失效，我没有验证；
- 普通相机的联网（步骤 12）：它仍能照常拍摄，但需要联网的相机功能将停止工作；你的相机有哪些这类功能，我没有核实；对你在步骤 12a 中禁止联网的任何包也是如此；
- 步骤 5a 的剩余项：主屏幕左侧的 Smart Feed、Moto 的“Create with AI”和 Journal、Android AICore（应用基于 Gemini Nano 的 AI 功能）、ARCore（应用中的增强现实）、“OK Google”语音训练、运营商占位包（`carriers.txt`，只有使用这些运营商的 SIM 卡时才需要）；这些里面哪些你会在手机上感觉到，我没有核实；
- 需要 Play Integrity 的银行类应用：在已解锁 bootloader 并获取 root 的手机上，其中许多本来就无法使用。

通话、短信、相机、Wi-Fi、蓝牙和 GPS 仍可正常使用。本指南会单独检查锁屏上的“紧急呼叫”按钮（步骤 6 和 8）。我们不碰 Motorola 的更新。

## 开始之前

请准备一台装有 adb 的电脑并做好备份：从手机上移除 Google 账号之后，所有只存放在该账号里的内容都会丢失。

**你需要准备**

- 一台运行 Linux、macOS 或 Windows 的电脑（在 Windows 上，请在 Git Bash 中执行本指南中的命令：在 WSL 中，adb 通常看不到手机）。
- 一根 USB 数据线，以及 Android Platform Tools 软件包中的 adb。fastboot 就在同一个软件包里，所以你很可能已经有了。
- 电量不低于 50%。
- 备好原厂 `boot.img` 和 `fastboot flash boot` 命令，以便在手机无法启动时回滚内核。
- 已通过 APatch 获得 root。它只在步骤 9、11、12 和 12a 中用到：一次性启用“Google”用户的自动停止，并在主屏幕上制作可选的按钮（步骤 9）；用于 Shelter 工作资料；以及用于禁止相机和其他包联网。其余操作都不需要 root。

**备份（请在第一步之前完成）**

- 联系人：联系人应用 → 设置 → 导出 → 得到一个 `.vcf` 文件，把它复制到电脑上。存放在 Google 账号中的联系人会随账号一起从手机上消失。之后再把 `.vcf` 导入本地的“手机”账号。
- 双重验证码：如果你使用 Google Authenticator，请把账号迁移到其他应用（在 Authenticator 中：菜单 →“转移账号”）。否则你可能会失去对某些服务的访问。
- 只存放在 Google 云端的照片、视频和文件：通过 Google Takeout 下载。
- 日历和任务，如果它们只存在于 Google：从网页版导出。
- 存放在 Google Drive 中的即时通讯备份（例如 WhatsApp）：保存一份本地副本。
- Google 密码管理器中的密码和通行密钥：导出密码。通行密钥无法导出，所以请确认重要服务有备用的登录方式。

先不要删除主用户中的 Google 账号：这一步在步骤 7 中进行。

## 步骤 1. 将手机连接到电脑

开启 USB 调试，用数据线连接手机，让 `adb devices` 显示该设备且状态为 `device`。

本指南中的所有命令都在电脑上的终端里输入。如果某条命令要在手机本身上输入，我会单独说明。

**如果输出和这里写的不一样。** 请停下，不要继续往下做。在这类地方，本指南会写“把输出发到文首的聊天群里”：指的就是文档开头的 [Telegram 聊天群](https://t.me/motorolafold)。请写明你停在了哪一步、第几项，并附上命令的完整输出。

**如何重启手机。** 当本指南在移除或恢复应用的命令之后说“重启手机”时，请这样做（在电脑上）：

```
sleep 20
adb shell svc power reboot
```

不要使用 `adb reboot`：Android 大约 10 秒后才会把“包已移除”的标记写入磁盘，如果重启太快，标记就会丢失，被移除的应用会回来（这在 Razr Fold 上就发生过；细节见步骤 4 第 4 项）。如果 `adb shell svc power reboot` 不起作用，请按电源键并选择“重启”，同样要在执行命令后至少 20 秒再操作。如果 `debloat.sh` 脚本在结尾打印出“adb reboot”，说明那是旧版本：请按步骤 4 第 1 项修改那一行。

**在手机上**

1. 打开设置 → 关于手机，连续点按“版本号”七次。屏幕上会出现一条消息，说你现在已是开发者。
2. 打开设置 → 系统 → 开发者选项，开启“USB 调试”。在某些固件中，这个板块叫“开发者设置”。
3. 用数据线把手机连接到电脑。手机屏幕上会出现提示“要允许 USB 调试吗？”。勾选“一律允许使用这台计算机进行调试”，然后点按“允许”。

**在电脑上**

打开终端，输入命令：

```
adb devices
```

正确的输出如下（你的序列号会不同）：

```
List of devices attached
ZY22XXXXXXXX    device
```

如果有问题：

- 显示的不是 `device` 而是 `unauthorized`：解锁手机，再次确认屏幕上的提示。
- 列表为空：换一根数据线或换一个 USB 端口。在手机的 USB 通知中选择“文件传输”。
- 在 Windows 上换任何数据线列表都为空：安装 Google 或 Motorola 的 USB 驱动。

如果终端提示找不到 `adb` 命令，请把 Platform Tools 文件夹加入 PATH。如果你把它解压到了主目录，输入 `export PATH="$PATH:$HOME/platform-tools"`。该命令只在当前这个终端窗口中有效。步骤 4 中的脚本调用 `adb` 时不带路径，所以不这样做它就无法运行。

当状态为 `device` 时，检查命令是否发给了正确的手机：

```
adb shell getprop ro.product.device
adb shell getprop ro.build.version.release
```

第一条命令会打印手机的代号（Razr Fold 上是 `blanc_g`，可能不带后缀）。第二条打印 `16`。如果输出完全不同，说明电脑连接的是别的设备：断开其他设备后重试。

## 步骤 1a. 删除 Shelter 工作资料

如果你安装了 Shelter，手机上就有它的工作资料（通常编号为 10）。如果你用不到，请在做其他步骤之前先把它删除：[据 Shelter 用户反馈](https://discuss.privacyguides.net/t/why-shelters-work-profile-comes-pre-loaded-with-some-google-services/14153)，这类资料一创建好，里面就会出现 Google 应用，而步骤 4–7 中的命令只会在主用户中隐藏它们。如果你需要这个资料，请跳过这一步：本指南仍然适用，但该资料中会保留 Google 和 Moto 的垃圾应用。如何把它们从那里清除，以及如何清理你新建的资料，见步骤 11。

**删除会抹掉资料内部的一切：** 应用、它们的数据和账号登录状态。此操作无法撤销。请先打开 Shelter，看看里面装了哪些应用，并用这些应用自带的功能保存你需要的内容。

**1. 删除工作资料（在手机上）**

设置 → 密码和账号 →“工作”标签页 →“删除工作资料”→ 确认。Moto 上的名称可能不同：请在账号部分查找“工作资料”。“先删资料，再删应用”的顺序取自 [Shelter 的说明](https://github.com/PeterCxy/Shelter?tab=readme-ov-file#uninstalling)。

**2. 如果无法在设置中删除该资料（在电脑上）**

先查看用户列表：

```
adb shell pm list users
```

确认编号为 10 的那一行确实是 Shelter 工作资料，而不是你需要的其他用户或资料，然后删除它。该资料的所有数据都会被删除（[命令出处](https://bayton.org/android/android-enterprise-faq/enrolment-failed-delete-wp/)）：

```
adb shell pm remove-user 10
```

**3. 检查资料已被删除（在电脑上）**

```
adb shell pm list users
```

编号为 10 的那一行必须消失。如果它还在，等一分钟后再执行一次该命令。

**4. 卸载 Shelter 应用（在手机上）**

长按 Shelter 图标并选择“卸载”。如果系统不允许卸载，请先在设置 → 安全 → 高级 → 设备管理应用中把它关闭（Moto 上该项的名称可能不同）。

之后进入步骤 2：最好在没有这个资料的情况下拍“之前”快照。

## 步骤 2. “之前”快照与角色检查

把所有包的列表保存到电脑上，并记下目前哪些应用负责通话、短信和键盘。之后你将依据这份快照回滚改动，并找出重新出现的垃圾应用。

**包快照（在电脑上）**

创建一个工作文件夹。把本指南中的所有文件都放进去。

```
mkdir -p ~/razr-debloat && cd ~/razr-debloat
```

生成三份列表。`|` 和 `>` 这两个字符由你的电脑处理，所以文件出现在电脑上，而不是手机上。

```
adb shell pm list packages -f -s -u --user 0 | sort > baseline_before.txt
adb shell pm list users > users_before.txt
adb shell pm list packages -3 --user 0 | sort > userapps_before.txt
```

`baseline_before.txt` 将包含所有系统包及其 APK 路径，包括已经被隐藏的包。这是最主要的文件。不要删除它，并在另一块磁盘上留一份副本。

**如果你得到的不是列表，而是错误“Shell does not have permission to access user 10”**

这表示命令到达了手机的另一个用户（对作者来说是 10 号用户，即 Shelter 工作资料），而 adb shell 不被允许进入它：那里禁止调试。这项检查存在于 Android 源代码中（[/e/OS 镜像中的副本](https://gitlab.e.foundation/e/os/android_frameworks_base/-/commit/8cd28b57ed732656d002d97879e15c5695b54fff)）。为什么不带 `--user` 的 `pm list packages` 会碰到另一个用户，我并不确定。显式加上 `--user 0` 就能解决：这样命令只查看主用户。本指南中所有这类命令里已经带上了它。如果错误仍然存在，请查明有哪些用户，以及现在屏幕上是哪一个（在电脑上）：

```
adb shell pm list users
adb shell am get-current-user
```

第二条命令会打印当前显示在屏幕上的用户编号。如果它不是 `0`，请切回主用户，等待五秒，然后重新生成快照：

```
adb shell am switch-user 0
```

这里不要动 Shelter 工作资料：如何处理它，见步骤 1a。

**当前负责通话、短信和键盘的是谁（在电脑上）**

```
adb shell cmd role get-role-holders --user 0 android.app.role.DIALER
adb shell cmd role get-role-holders --user 0 android.app.role.SMS
adb shell cmd role get-role-holders --user 0 android.app.role.HOME
adb shell cmd role get-role-holders --user 0 android.app.role.BROWSER
adb shell cmd role get-role-holders --user 0 android.app.role.ASSISTANT
adb shell settings get secure default_input_method
```

每条命令会打印一个包名：依次是拨号器、短信、桌面启动器、浏览器、助理和键盘。对于键盘，包名后面会跟一个斜杠和一个类名：不需要它。记下这些结果，步骤 6 会用到。如果某一行是空的，说明该角色没有被指定，或者命令没有生效。这时请在手机上打开设置 → 应用 → 默认应用，到那里查看。

**你的手机上有哪些垃圾应用（在电脑上）**

```
adb shell pm list packages --user 0 | grep -E 'motorola|lenovo|google|facebook|netflix|amazon|microsoft|spotify' | sort | tee present_now.txt
```

列表会很长。其中大部分 `com.google.android.*` 的行是 Android 系统模块，我们不碰它们（见“不能移除的内容”一节）。这份列表用来和步骤 4 和步骤 5 中的清单进行比对。

## 步骤 3. 创建“Google”用户并检查

创建一个单独的用户“Google”，把 Google 服务安装到其中，登录你的账号，并在主用户尚未改动的情况下确认 Play 商店能正常工作。

请在从主用户中移除 Google 服务之前完成这一步。这样，该用户是在一个完全正常运行的系统上创建的，你也能立刻看到 Google 在其中运行。

**关于用户编号。** “Google”用户的编号取决于手机上已有哪些用户：可能是 10、11，等等。为了不用手动替换编号，下面的命令使用 `$GU`。每个代码块的第一行会自己在列表中找到名为“Google”的用户并记住它的编号。请把代码块整段复制，包括这一行。

**1. 检查系统是否允许多个用户（在电脑上）**

```
adb shell pm get-max-users
adb shell pm list users
```

类似 `Maximum supported users: 4` 的回答表示一切正常。你需要的数字是 2 或更大。如果显示的是 `1`，请停下。目前什么都没有被移除，手机原封未动，但没有多用户模式，这个方案行不通。

第二条命令会打印已经存在的用户：形如 `UserInfo{0:Owner:c13} running` 的行。如果其中已经有一个名为 `Google` 的用户，请停下：你不需要再创建第二个。如果它是空的，是上一次尝试遗留下来的，请用“回滚”一节（最后一个代码块）中的命令把它删除，然后重新开始这一步。

**2. 创建用户（在电脑上）**

```
adb shell pm create-user Google
```

回答会是类似 `Success: created user id 11` 的一行。你的编号可能是任意值，例如 10 或 11。如果打印的不是 `Success` 而是错误，请停下：用户没有创建成功，手机原封未动。检查能否找到这个编号（在电脑上）：

```
GU=$(adb shell pm list users | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
echo Google 用户编号: $GU
```

应该打印出与 `Success` 那一行相同的编号。如果没有打印出任何编号，请停下，并把 `adb shell pm list users` 命令的输出发到文首的聊天群里。

**3. 启动该用户并把 Google 服务安装到其中（在电脑上）**

```
GU=$(adb shell pm list users | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
adb shell am start-user -w $GU
adb shell cmd package install-existing --user $GU com.google.android.gms
adb shell cmd package install-existing --user $GU com.android.vending
adb shell cmd package install-existing --user $GU com.google.android.gsf
```

最后三条命令的回答都是类似 `Package ... installed for user: 11` 的一行（你的编号会不同）。这就是成功，即使这个包原本已经安装过。检查结果：

```
GU=$(adb shell pm list users | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
adb shell pm list packages --user $GU | grep -E 'google.android.gms|android.vending|google.android.gsf'
```

应该有 `com.google.android.gms`、`com.android.vending` 和 `com.google.android.gsf` 这几行。如果还有别的，例如 `com.google.android.gms.location.history` 或 `com.google.android.gms.supervision`，这是正常的：搜索会匹配名称中带有 `google.android.gms` 的任何包，而且系统自己会把这两个包放进新用户。

如果某条命令打印了 `Shell does not have permission to access user` 以及“Google”用户的编号，说明这个用户里同样禁止调试，和 Shelter 工作资料一样。请尝试只对我们的用户解除这项禁止（在电脑上）：

```
GU=$(adb shell pm list users | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
adb shell pm set-user-restriction --user $GU no_debugging_features 0
```

然后重新执行带 `install-existing` 的代码块和检查。这是根据 Android 源代码得出的；我没有在你的手机上验证过。如果命令打印了同样的错误，请停下：这时只能通过 root 解除禁止，而本指南不涵盖这种情况。请把错误发到文首的聊天群里。

**4. 登录该用户和 Google 账号（在手机上）**

用下面的命令把屏幕切换到新用户：

```
GU=$(adb shell pm list users | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
adb shell am switch-user $GU
```

可能会打开首次启动向导。连接 Wi-Fi，登录你的 Google 账号，其余的都跳过。然后打开 Play 商店，安装任意一个小应用。如果装上了，就说明 Google 在这个用户中可以正常工作。

**5. 回到主用户并关闭“Google”用户（在电脑上）**

整段粘贴这个代码块，它会在切换后自己等待五秒：

```
GU=$(adb shell pm list users | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
adb shell am switch-user 0
sleep 5
adb shell am stop-user -f $GU
adb shell pm list users
```

最后一条命令的输出大致如下（花括号里的数字对你会不同）：

```
Users:
    UserInfo{0:Owner:c13} running
    UserInfo{11:Google:10}
```

有一点很重要：带 `Google` 的那一行没有 `running` 这个词。已关闭的用户就是这个样子。列表中的其他用户不要动。

在“Google”用户中，只安装没有 Google 就无法使用的东西：银行类应用、特殊用途的应用。其余一切都留在主用户中。步骤 4–7 清理的是主用户，“Google”用户保持不动。

## 不能移除的内容

只移除本指南中点名的包：任何其他包都可能恰好是缺了它手机就无法启动或无法打电话的那个。

如果某个包看起来像垃圾，但不在下面的清单里，就不要动它。先弄清楚这个包是什么，再决定。

| 是什么 | 示例名称 | 会出什么问题 |
| --- | --- | --- |
| Android 核心 | 所有以 `com.android.` 开头的包，以及 `android` 包本身。例外：`com.android.vending`、`com.android.chrome` 和两个 `com.android.hotwordenrollment.*` 包（“OK Google”语音训练）；它们在清单里点了名 | 启动、通话、蓝牙、NFC、设置、界面 |
| Google Mainline 模块 | 所有不在你的清单里的 `com.google.android.*`：`permissioncontroller`、`packageinstaller`、`documentsui`、`ext.services`、`networkstack`、`cellbroadcastreceiver`、`tethering`、`conscrypt` | 权限、安装应用、文件、网络、紧急警报 |
| WebView | `com.google.android.webview`、`com.android.webview`、`com.google.android.trichromelibrary` | 应用内的网页将无法打开 |
| Qualcomm | `com.qualcomm.*`、`com.qti.*`、`vendor.qti.*`、`org.codeaurora.*` | 基带、GPS、Wi-Fi、声音 |
| Overlay | 名称中含有 `overlay` 一词的任何包 | 主题、屏幕尺寸、运营商设置 |
| 更新与 Moto 核心 | `com.motorola.ccc.*`（包括 `com.motorola.ccc.ota`；唯一的例外是 `com.motorola.ccc.notification`，它可以移除）、`com.motorola.launcher3`、`com.motorola.camera3`、`com.motorola.camera5`、`com.motorola.launcher.secondarydisplay` | 更新、主屏幕、相机、外屏 |
| Razr Fold 的屏幕和铰链 | 名称中含有 `razr`、`fold`、`cover`、`hinge`、`flip`、`display`、`sensor` 这些词的包 | 折叠、内屏和外屏 |
| Moto 设置 | `com.motorola.android.providers.*`、`com.motorola.android.coresettingsext*`、`com.lc.motorola.coresettingsext` | “设置”菜单和设置存储 |

步骤 4 中的 `debloat.sh` 脚本自己会跳过一部分受保护的包。这是一道保险，不能代替这条规则。任何移除都可以按“回滚”一节所述撤销。对 `com.motorola.ccc.*` 有一个有意设置的例外：`com.motorola.ccc.notification` 不算受保护：它在步骤 5 的 Moto 清单里，会像普通包一样被移除。

## 步骤 4. 移除 Google 应用

创建 `debloat.sh` 脚本和 `google.txt` 清单，执行移除并重启手机：这样会隐藏那些不负责通话、短信和键盘的 Google 应用。

拨号器、短信、通讯录、键盘、时钟、日历、计算器、浏览器和相册不在这份清单里。它们会在步骤 6 中、在有了替代品之后再移除。

**1. 创建脚本（在电脑上）**

脚本会读取一份包清单，并为每个包打印结果。它会跳过手机上没有的包，也不会动部分受保护的包。请整段复制代码块，并粘贴到在工作文件夹 `~/razr-debloat` 中打开的终端里：

```
cat > debloat.sh <<'EOF'
#!/usr/bin/env bash
# 用法：
#   ./debloat.sh remove list.txt   隐藏清单中的包
#   ./debloat.sh restore list.txt  把清单中的包找回来
mode="$1"
list="$2"
if [ "$mode" != "remove" ] && [ "$mode" != "restore" ]; then
  echo "第一个参数：remove 或 restore"; exit 1
fi
if [ ! -f "$list" ]; then
  echo "找不到清单文件：$list"; exit 1
fi

installed=$(adb shell pm list packages --user 0 | tr -d '\r' | sed 's/^package://')
if [ -z "$installed" ]; then
  echo "看不到手机。请检查 adb devices 命令"; exit 1
fi

grep -Ev '^[[:space:]]*(#|$)' "$list" | tr -d '\r' | while read -r pkg _; do
  case "$pkg" in
    com.motorola.ccc.notification) ;;  # 例外：这个包可以移除（步骤 5）
    android|com.android.systemui|com.android.settings|com.android.phone|com.android.server.telecom|com.android.providers.*|com.qualcomm.*|com.motorola.ccc.*|*overlay*|com.google.android.webview|com.google.android.trichromelibrary)
      echo "$pkg: 受保护的包，跳过"; continue ;;
  esac
  if [ "$mode" = "remove" ]; then
    if printf '%s\n' "$installed" | grep -Fqx "$pkg"; then
      printf '%s: ' "$pkg"
      adb shell pm uninstall -k --user 0 "$pkg" </dev/null | tr -d '\r'
    else
      echo "$pkg: 手机上没有或已被移除，跳过"
    fi
  else
    printf '%s: ' "$pkg"
    adb shell cmd package install-existing --user 0 "$pkg" </dev/null | tr -d '\r'
  fi
done
echo "完成。请等待 20 秒后重启手机：adb shell svc power reboot"
EOF
chmod +x debloat.sh
```

**如果你已经用本指南的旧版本创建了 debloat.sh**，它在结尾会打印“完成。请重启手机：adb reboot”。不能这样重启，原因见下面第 4 项。只需修改这一行（在电脑上，在 `~/razr-debloat` 文件夹中）：

```
sed 's|^echo ".*adb reboot"$|echo "完成。请等待 20 秒后重启手机：adb shell svc power reboot"|' debloat.sh > debloat.sh.new && mv debloat.sh.new debloat.sh && chmod +x debloat.sh
grep -c 'adb reboot' debloat.sh
tail -n 1 debloat.sh
```

第一个检查应该打印 `0`，第二个应该打印出带有 `adb shell svc power reboot` 的一行。如果你已经有新版本的脚本，这条命令不会改变任何东西。

**2. 创建清单（在电脑上）**

这份清单是根据两篇公开文章（[droidwin 上的 Motorola 文章](https://droidwin.com/?p=19403)和 [inatagan/android-debloater](https://github.com/inatagan/android-debloater)）以及我自己的知识整理的。注释里带星号的行，是我从通用清单中知道的；它们不在那两篇文章里。我没有对照你的手机检查过这份清单，脚本会跳过多余的行。所有不带星号的行，除了 nbu.files、ar.lens 和 apps.adm 之外，都在 droidwin 的“可安全移除”清单里。它没有把前两个称为安全的，而 apps.adm（“查找我的设备”）是根据 Razr Fold 上的应用列表添加的：文章里没有它。如果有疑虑，请在执行前把这几行从清单中删除。

```
cat > google.txt <<'EOF'
# Google 应用。每行格式：包名  # 名称。星号：这个包是我从通用清单里知道的。
com.google.android.youtube  # YouTube
com.google.android.apps.youtube.music  # YouTube Music
com.google.android.apps.maps  # Google 地图
com.google.android.gm  # Gmail
com.google.android.apps.docs  # Google 云端硬盘
com.google.android.videos  # Google TV（前身是 Play 电影）
com.google.android.music  # Play 音乐，旧版应用
com.google.android.apps.tachyon  # Google Meet（前身是 Duo）
com.google.android.apps.subscriptions.red  # Google One
com.google.android.apps.walletnfcrel  # Google 钱包和 Pay
com.google.android.apps.googleassistant  # Google 助理
com.google.android.apps.nbu.files  # Files by Google
com.google.android.googlequicksearchbox  # Google 应用和主屏幕上的搜索栏
com.google.android.printservice.recommendation  # 打印推荐
com.google.android.syncadapters.calendar  # Google 日历同步
com.google.android.syncadapters.contacts  # Google 通讯录同步
com.google.ar.lens  # Google 智能镜头
com.google.android.apps.adm  # 查找我的设备（Find Hub）
com.google.android.apps.bard  # Gemini *
com.google.android.apps.magazines  # Google 新闻 *
com.google.android.apps.chromecast.app  # Google Home *
com.google.android.projection.gearhead  # Android Auto *
com.google.android.setupwizard  # Android Setup：“Finish setting up your device”通知 *
EOF
```

`com.google.android.setupwizard`（Android Setup）这一行是为了消除“Finish setting up your device”通知而加的。这个应用的通知开关在设置里是不可用的（灰色），所以我们把应用本身隐藏掉。根据通用的说明，它只在手机首次启动和恢复出厂设置之后才需要；恢复出厂设置之后数据会被清空，所有被隐藏的包会自己回来，所以初始设置不会丢失。这个包是我从通用清单中知道的，我没有在你的手机上验证过。

如果 `google.txt` 已经创建好了，就把这一行加进去，再执行一次移除（在电脑上）：

```
echo 'com.google.android.setupwizard  # Android Setup：“Finish setting up your device”通知 *' >> google.txt
./debloat.sh remove google.txt
sleep 20
adb shell svc power reboot
```

其他行会显示“手机上没有或已被移除，跳过”：这是正常的，它们已经被移除了。`com.google.android.setupwizard` 这一行应该显示 `Success`；如果显示 `Failure`，请把这一行完整地发到文首的聊天群里。重启之后，“Finish setting up your device”通知应该消失了。

如果重启后通知仍然存在，或者脚本写了“手机上没有或已被移除，跳过”，说明它是由另一个包显示的。查看有通知的包的列表（在电脑上）：

```
adb shell dumpsys notification | grep -o 'pkg=[^ ]*' | sort -u
```

在其中找一个名称里带有 `setup` 一词的包，并把输出发到文首的聊天群里。

**3. 执行移除（在电脑上）**

```
./debloat.sh remove google.txt
```

每一行都会得到下列回答之一。`Success` 表示该包已移除。`Failure [...]` 表示系统不允许移除：保持这个包不动。`Failure: package is non-disable` 表示 Motorola 保护了这个包不被隐藏；对 Gboard 的分析在步骤 6。“手机上没有或已被移除”这句话表示这一行被跳过了。

**4. 重启并检查**

```
sleep 20
adb shell svc power reboot
```

不要在脚本刚运行完就用 `adb reboot` 重启手机。Android 不是立刻、而是大约 10 秒后才把“包已移除”的标记写入磁盘（[这是系统本身的设计](https://gitlab.e.foundation/e/os/android_frameworks_base/-/commit/a83500e548584005e35a7836b7362333460c8c99)）。如果手机在此之前关机，标记就会丢失，被移除的应用会回来。所以这里和后面都是先 `sleep 20` 暂停，再用正常的重启 `svc power reboot`：正常关机时 Android 有时间保存一切。如果这条命令不起作用，请用电源键重启手机（“重启”选项）。在 Razr Fold 上发生的正是这种情况：在脚本刚运行完就用 `adb reboot` 之后，步骤 7 中的所有包都回来了，而在暂停并使用 `svc power reboot` 之后，它们保持被移除的状态。

解锁手机并等待一分钟。如果主屏幕、通知栏和设置都能打开，这一步就通过了。如果出了问题，用命令 `./debloat.sh restore google.txt` 恢复一切。

**可选**

- Digital Wellbeing（`com.google.android.apps.wellbeing`）：应用计时器和屏幕使用统计将消失。
- Google 文字转语音（`com.google.android.tts`）：导航应用和阅读器中的朗读功能将消失。请先安装另一个文字转语音引擎。

对每个包，命令都是一样的：`adb shell pm uninstall -k --user 0 package`。

**这一步之后什么会消失**

主屏幕上的 Google 搜索栏，以及清单中的 YouTube、地图、Gmail、云端硬盘等应用。在“Google”用户中，它们仍然保留，或者从 Play 商店安装。

## 步骤 5. 移除 Moto 垃圾应用和合作伙伴预装应用

创建包含 Motorola 和合作伙伴应用的 `moto.txt` 清单，运行同一个脚本并重启手机（第 1–3 项）。然后单独处理那些“移除是有代价的”应用（第 4–6 项）。如果你之前已经做完了第 1–3 项，而此后清单又增加了内容，请重新创建这个文件（第 1 项）并重复第 2 项：脚本会跳过已经移除的包。第一份清单中的其余包（Google 的残留、Moto 的小包、合作伙伴占位包、运营商包）在步骤 5a 中处理。

这份清单取自同样的公开文章，以及 [bayton 系统应用数据库](https://sysapps.bayton.org)。数据库能确认这些包存在以及它们的作用，但并不声明它们可以安全移除。droidwin 只把 genie、fmradio、fmplayer、moto 和四个 Facebook 包称为安全。其他各行只是出现在通用的已安装包列表里，或者根本没有出现：如果有疑虑，请在执行前把那一行从清单中删除。带星号的行来自我自己的知识。主屏幕、相机、更新和 Razr Fold 的屏幕不在清单里，绝对不能动它们。

有七行是根据 Razr Fold 上的应用列表添加的：`com.motorola.gamemode`、`com.motorola.audiorecorder`、`com.motorola.spaces`、`com.motorola.aitranslation`、`com.motorola.om`、`com.motorola.uxcore` 和 `com.lenovo.qira`。在 droidwin 的文章里没有人把它们称为可安全移除：`gamemode` 只出现在通用的已安装包列表里，其余的根本没有出现。每个包的作用都写在清单的注释里，依据的是公开的说明。我无法在手机本身上核实它们：如果某一行让你起疑，请在执行前把它删除。

`jp.co.celsys.clipstudiopaint.googleplay`（Clip Studio Paint）这一行是按作者的决定加上的：这个应用是付费的，已经当场被移除了。据我所知，这个包与手写笔没有关系，但第 5 项会在移除之后检查手写笔。

`com.motorola.ccc.notification`（Motorola Notifications）这一行是按作者的决定加上的：这个包之前就被移除过，这次也被移除了。此前 `debloat.sh` 把它当作受保护的包跳过，因为所有 `com.motorola.ccc.*` 包都与更新有关；现在脚本为它设了一个例外，其余的 `ccc.*` 脚本仍然不碰。如果你之前创建过 `debloat.sh`，请重新创建一次（步骤 4 第 1 项）：旧版本会打印“受保护的包，跳过”。有一个风险我无法核实：在通用数据库里，这个包拥有联系更新服务（`ccc.ota`）的权限，所以它们可能是相关的。因此在系统更新之前应当把它找回来，方法见步骤 10。

`com.motorola.livewallpaper3`（Moto Interactive Wallpaper，Moto 的互动壁纸）这一行同样是按作者的决定加上的。如果主屏幕或锁屏当前正在使用的恰好就是这款壁纸，请先进入设置 → 壁纸和个性化，选择一张普通图片（Moto 上该板块的名称可能不同）。据我所知，壁纸消失时系统会自己设置默认壁纸，但我没有在你的手机上验证过。

**1. 创建清单（在电脑上）**

```
cat > moto.txt <<'EOF'
# Moto 和合作伙伴。每行格式：包名  # 名称。星号：这个包是我从通用清单里知道的。
com.motorola.genie  # Moto 帮助
com.motorola.help  # Moto：反馈
com.motorola.demo  # 展示机演示模式
com.motorola.timeweatherwidget  # Moto 的“时间和天气”小部件
# com.motorola.gesture  # 未列入：从 APK 名称看，这是导航手势教程，移除的收益很小
com.motorola.android.fmradio  # FM 收音机
com.motorola.fmplayer  # FM 收音机
com.dti.motorola  # Moto App Manager：推荐应用的安装器
com.aura.oobe.motorola  # MotoApps：设置过程中的推荐应用
com.motorola.gamemode  # Moto Games：游戏模式
com.motorola.audiorecorder  # Moto 录音机
com.motorola.spaces  # Family Space：儿童空间
com.motorola.aitranslation  # Moto 的 Translate（据包名判断，AI 翻译）
com.motorola.om  # Moto Unplugged：限制应用和通知的模式
com.motorola.uxcore  # Moto AI
com.lenovo.qira  # Motorola Qira：联想的 AI 助手，部分在云端运行
jp.co.celsys.clipstudiopaint.googleplay  # Clip Studio Paint：付费绘画应用
com.motorola.ccc.notification  # Motorola Notifications：Motorola 的消息客户端
com.motorola.livewallpaper3  # Moto Interactive Wallpaper：互动壁纸
com.motorola.mototour  # Moto 入门介绍（教程） *
com.facebook.katana  # Facebook
com.facebook.appmanager  # Facebook App Manager
com.facebook.services  # Facebook Services
com.facebook.system  # Facebook System
com.netflix.mediaclient  # Netflix *
com.amazon.mShop.android.shopping  # Amazon *
com.spotify.music  # Spotify *
EOF
```

**2. 执行移除（在电脑上）**

```
./debloat.sh remove moto.txt
```

脚本的回答与步骤 4 中相同。Razr Fold 上可能缺少某些包：脚本会跳过它们。

**3. 重启并检查**

```
sleep 20
adb shell svc power reboot
```

`sleep 20` 的暂停和正常重启的原因与步骤 4 相同：否则 Android 可能来不及保存已移除的内容。

开机后，检查主屏幕、相机、通知栏、“返回”和“主页”手势、折叠手机以及外屏。如果出了问题，用命令 `./debloat.sh restore moto.txt` 恢复一切。

**4. 移除有代价的应用（在电脑上）**

我没有把这三个应用放进 `moto.txt`：每一个都有明显的功能，是否移除由你决定。请阅读下面的说明。如果你想保留其中某个，就在下面清单里它那一行的开头加上 `#`。

**Moto**（`com.motorola.moto`）。这不是初始设置向导。根据运营商的帮助页面，Moto 手势是通过这个应用打开的（Moto → 手势），Motorola 的个性化设置也在其中。`com.motorola.actions` 包（Moto Actions）是独立的，我不碰它。droidwin 文章下的一位读者在评论中写道，没有这个应用，自适应性能、优化充电和过充保护都会消失。我没有验证这一点。如果你使用 Moto 手势或这些功能，就保留它。

**Moto Secure**（`com.motorola.securityhub`）。Motorola 把它描述为 Android 安全与隐私设置的统一页面，外加它自己的功能：不安全 Wi-Fi 警告、Wi-Fi 拦截、用于 PIN 码的乱序键盘和受保护文件夹。标准的 Android 设置不会因此消失：它们在设置 → 安全和隐私中。在通用系统应用数据库里，它带有标注了联网、后台定位、相机和所有已安装应用列表的权限。我没有发现它发送任何东西的数据，但代码是闭源的，无法证明它不发送。如果你不用受保护文件夹，就移除它。如果你在用，请先把文件移出来。根据作者的检查，这个包有联网权限，作者把它移除了，所以这一行留在了下面的清单里。

**Dolby Atmos**（`com.motorola.dolby.dolbyui`）。据我所知，这是 Dolby 音效设置窗口。根据作者的检查（下面的命令），它没有直接联网的权限，所以保留了它，它不在移除清单里。也不要碰负责实际音效处理的 `com.dolby.daxservice` 包。如果你已经移除了 `com.motorola.dolby.dolbyui`，请用命令 `adb shell cmd package install-existing --user 0 com.motorola.dolby.dolbyui` 把它找回来，然后等待 20 秒，再用命令 `adb shell svc power reboot` 重启手机。

**Smart Connect**（`com.motorola.mobiledesktop`）。这是以前的 Ready For：把手机连接到电脑，以及在显示器或电视上的桌面模式。根据 Google Play 上的说明，它还管理 Moto Buds 耳机和 Moto Tag 追踪器。存在开放的替代品：scrcpy 可以在电脑上显示手机屏幕并让你控制它，KDE Connect 可以把手机和电脑连起来（通知、文件、剪贴板）。在通用数据库里，它有联网、读取和发送短信、联系人、录音和相机的权限。根据作者的检查，这个包有联网权限，作者把它移除了，所以这一行留在了下面的清单里。移除之后，“Smart Connect”这一项会从 USB 模式菜单中消失（用数据线把手机连接到电脑时会出现该菜单）：你的情况就是这样，其他选项都还在。这说明这一项是由该应用自己创建的。如果你不需要在显示器或电视上的桌面模式，就什么都不用恢复。

联网检查（在电脑上）。请把包名替换为你需要的那个：

```
adb shell dumpsys package com.motorola.dolby.dolbyui | grep -iE 'permission.INTERNET|sharedUser'
```

如果打印出带有 `android.permission.INTERNET` 的一行，说明这个包可能直接访问网络。如果什么都没有打印，就说明没有直接访问。如果只打印出一行 `sharedUser=`，说明这个包与其他系统包共享权限，这次检查什么也证明不了。这个检查同样看不出通过另一个有联网权限的应用传输数据的情况。

创建清单（在电脑上）：

```
cat > moto-extra.txt <<'EOF'
# 移除有代价的 Moto 应用。你想保留哪个，就在那一行开头加上 #。
com.motorola.moto  # Moto：手势和 Motorola 功能
com.motorola.securityhub  # Moto Secure
com.motorola.mobiledesktop  # Smart Connect（前身是 Ready For）
EOF
```

执行移除并重启手机（在电脑上）：

```
./debloat.sh remove moto-extra.txt
sleep 20
adb shell svc power reboot
```

如果某一行显示的不是 `Success` 而是 `Failure: package is non-disable`，说明该包受到系统保护。请跳过它，并把完整的输出发到文首的聊天群里：这类包需要用与步骤 6 中 Gboard 相同的办法处理。

重启之后，在手机上检查：

- 设置 → 安全和隐私可以打开，手机可以用指纹或面部解锁。
- 如果你以前用 Moto 手势，它们能正常工作。
- 折叠和外屏正常工作，与步骤 8 中一样。
- 任何地方都没有弹出“……已停止运行”的窗口。

如果出了问题，把这三个应用都找回来（在电脑上）：

```
./debloat.sh restore moto-extra.txt
sleep 20
adb shell svc power reboot
```

单个包可以用命令 `adb shell cmd package install-existing --user 0 PACKAGE_NAME` 找回来，然后同样是暂停加重启。

对于 Smart Connect（在电脑上）：`adb shell cmd package install-existing --user 0 com.motorola.mobiledesktop`。暂停并重启之后，它在 USB 模式菜单中的那一项会重新出现。

**5. 手写笔（在电脑上）**

问题在于，没有 `com.motorola.stylus`（在应用列表中它的标签是“笔记”）手写笔还能不能用。我没有确切的答案：我没有找到任何来源说明手写笔按键和连接的逻辑存放在哪个包里。已知的情况是：Moto Pen Ultra 通过蓝牙连接，在 Razr Fold 的两块屏幕上都能使用，并支持压感、倾斜和防误触。笔上的按键可以配置，Quick Clip 功能会把文字发送到 Motorola 的笔记应用。所以请最后再移除这个包，并立刻检查。Clip Studio 已经当场移除了：它已经在 moto.txt 清单里（第 1–3 项），所以我们在没有它的情况下检查压感。

根据作者的检查，`com.motorola.stylus` 有联网权限。冻结（`pm disable-user`）对手写笔很可能不会有什么改变：应用同样会停止工作，变化的只是把它找回来的命令（`pm enable`）。所以仍然采用移除，和其他包一样：更新之后对它的检查已经写在步骤 10 里。

移除这个包（在电脑上）：

```
adb shell pm uninstall -k --user 0 com.motorola.stylus
sleep 20
adb shell svc power reboot
```

重启之后，在手机上检查：

- 设置 → 已连接的设备：手写笔显示为已连接。
- 检查压感。如果你有绘画应用，请在改变压力的同时画一条线：线条应该变细变粗。如果没有这类应用，请开启设置 → 系统 → 开发者选项 →“指针位置”（Moto 上的名称可能不同），并在改变压力的同时用手写笔在屏幕上移动：顶部栏中的压力数值应该变化（通常标为 Prs）。然后再把这一项关掉。
- 按下手写笔的侧键：它应该和以前做同样的事。
- Quick Clip 和“笔记”应用很可能会随这个包一起消失。如果你需要它们，就把这个包找回来。

如果手写笔的表现比以前差，就把这个包找回来（在电脑上）：

```
adb shell cmd package install-existing --user 0 com.motorola.stylus
sleep 20
adb shell svc power reboot
```

如果你不得不把这个包找回来，它仍然有联网权限。你可以用步骤 12 中的方法禁止它联网：把 `com.motorola.stylus` 加入 `block-net.list`（先像步骤 12a 那样检查 uid 编号）。

如果 Clip Studio 是预装的那个，`restore moto.txt` 会把它连同清单里的其他包一起找回来；单独找回它可以用命令 `adb shell cmd package install-existing --user 0 jp.co.celsys.clipstudiopaint.googleplay`（然后同样是暂停和重启，如上）。如果这样无法找回，你就需要它的 APK。

**6. Moto AI 模型（在电脑上）**

这些是 Moto 应用从中获取 AI 模型的包：`com.motorola.aiservices`（“Moto AI Services”）和 `com.motorola.aicore`（“Moto AI Core”）。Moto AI 和 Qira 这两个应用本身已经在第 1–3 项中移除了，而模型会一直留在手机上，直到你把它们移除。这不是 Google AICore：那个的包名不同，是 `com.google.android.aicore`；它在步骤 5a 中单独移除。

已知的情况。根据 [APKMirror 上的说明](https://apkmirror.com/apk/motorola-mobility-llc/moto-ai-services/moto-ai-services-1-1-149-13-release/moto-ai-services-1-1-149-13-android-apk-download)，Moto AI Services 为 Moto 的“笔记”、“风格化”、优化充电、自适应亮度等功能提供模型。在[系统应用数据库](https://sysapps.bayton.org/packages/com.motorola.aiservices)里，它标注了定位权限和接收推送消息的权限。“Moto”应用和 Moto Actions 从它们的权限来看，会连接到这些服务。没有它们到底哪些会停止工作，我没有核实。如果你为了手写笔保留了“笔记”（第 5 项），其中的 AI 功能在没有这些包的情况下可能无法使用。

创建清单（在电脑上）：

```
cat > moto-ai.txt <<'EOF'
# Moto AI 模型。Moto AI 和 Qira 这两个应用本身已在第 1–3 项中移除。
com.motorola.aiservices  # Moto AI Services：Moto 应用的 AI 模型
com.motorola.aicore  # Moto AI Core：Moto 应用的 AI 模型
EOF
```

执行移除并重启手机（在电脑上）：

```
./debloat.sh remove moto-ai.txt
sleep 20
adb shell svc power reboot
```

重启之后，检查这些包是否已被移除（在电脑上）。这条命令应该什么都不打印：

```
adb shell pm list packages --user 0 | grep -E 'com.motorola.aiservices|com.motorola.aicore'
```

如果打印出了什么，请重复上一个代码块中的移除，并再次带暂停地重启手机。如果某一行显示的不是 `Success` 而是 `Failure: package is non-disable`，请像第 4 项那样，把输出发到文首的聊天群里。

在手机上检查：

- 设置 → 显示：自适应亮度（也可能叫“自动亮度”）正常。把手机从亮处移到阴影里：亮度应该会变化。
- 手机可以充电，设置 → 电池可以打开。
- 相机可以打开并拍照。
- 如果你以前用 Moto 手势，它们能正常工作，并且“Moto”应用（如果你在第 4 项中保留了它）可以打开。
- 任何地方都没有弹出“……已停止运行”的窗口。

如果出了问题，把这些包找回来（在电脑上）：

```
./debloat.sh restore moto-ai.txt
sleep 20
adb shell svc power reboot
```

**我有意没有列入的内容**

`com.motorola.ccc.*` 这些包（“Motorola Services”）不在清单里。它们与更新和设备管理有关，而且我没有找到任何公开资料表明它们可以毫无后果地移除。不要碰它们。唯一的例外 `com.motorola.ccc.notification` 已经在步骤 5 的 Moto 清单里。

带有 Moto AI 模型的包（`com.motorola.aiservices` 和 `com.motorola.aicore`）也不在 `moto.txt` 里：它们在第 6 项中单独移除。

`com.motorola.gesture` 包同样没有列入。公开资料里没有对它的描述，而从文件名（GestureNavigationTutorial.apk）看，它是导航手势教程。我没有核实它与 Moto 手势有没有联系，移除的收益很小，所以保留它。

## 步骤 5a. 第一份清单中的其余包

在步骤 2 中，你保存了所有包的清单（`baseline_before.txt`）：共 533 行。步骤 4–7 的清单已经包含了其中的 55 个包。其余的我逐个过了一遍。结果如下：

- 171 个 overlay（文件路径中包含 `/overlay/`）和 30 个 APEX 模块。我们不碰它们：见“不能移除的内容”一节。
- 277 个单独的包：用本步骤的清单移除其中 75 个，另有 9 行以禁用状态写入清单（行首是 `#`），其余 193 个保留。具体是哪些、为什么，写在本步骤末尾的表格里。

**如何阅读新清单。** 每行的格式是：`包名  # 名称：作用`。如果注释末尾没有标记，说明这个包的用途在公开资料中有描述。星号 `*` 表示这个包在公开资料中没有描述，其用途是根据清单、APK 名称或相邻的包推断出来的。行首带 `#` 的行是禁用的，脚本会跳过它；这样的行里出现问号 `?`，表示我不知道这个包是做什么的。如果你想移除某个被禁用的包，请删掉行首的 `#`，然后重复移除。

与步骤 4–7 一样，这些命令只在主用户（`--user 0`）中移除包。在“Google”用户中它们会保留。Shelter 工作资料在步骤 11 中清理：新清单也会加到那里。

**1. Google 的残留：13 个包（在电脑上）**

里面有：Google Fi 应用（`apps.tycho`）和自动连接运营商 Wi-Fi 的功能（`carrierwifi`）；首次设置的服务包（`partnersetup`、`onetimeinitializer`、`searchselector`）；发送错误报告的功能（`feedback`）；Android Device Policy（`clouddpc`）；Android AICore，即 Gemini Nano AI 模型服务（`aicore`；这不是步骤 5 第 6 项中的 Moto AICore）；ARCore，即增强现实（`ar.core`）；跨设备服务（`ambient.streaming`）；Quick Share 扩展（`mosey`）；两个“OK Google”语音训练包（`hotwordenrollment`）。

主用户中可能不再工作的功能：带增强现实的应用、向 Chromebook 投屏的应用、基于 Android AICore 的应用 AI 功能、Quick Share、“OK Google”语音训练。没有步骤 7 中的 Google 服务，这些功能在主用户中本来也大多无法使用。只有手机由组织管理（带有设备策略的工作账号）的人才需要 Android Device Policy：如果这就是你，请删掉它那一行。

清单末尾有两行是禁用的：`com.google.android.as`（Android System Intelligence：实时字幕、实时翻译、智能自动旋转）和 `com.google.android.as.oss`（Private Compute Services，为它提供的私密分析服务）。如果移除它们，这些功能会消失，而 Moto 里还有什么依赖它们，我不知道。要移除就只能两个一起移除。

创建清单（在电脑上）：

```
cat > google-more.txt <<'EOF'
# 第一份清单中 Google 的残留。每行格式：包名  # 名称。
# 没有星号：用途在公开资料中有描述。有星号：公开资料中没有描述，用途是根据清单或名称推断的。
# 以 # 开头的行是禁用的。如果你想移除这样的包，请删掉 # 再执行移除。
com.google.android.apps.tycho  # Google Fi：Google 的美国运营商的应用
com.google.android.partnersetup  # Google Partner Setup：Google 应用的合作伙伴设置 *
com.google.android.onetimeinitializer  # Google One Time Init：首次启动后的一次性设置 *
com.google.android.feedback  # Google Feedback：向 Google 发送错误报告 *
com.google.android.apps.work.clouddpc  # Android Device Policy：由组织管理手机 *
com.google.android.aicore  # Android AICore：Google 面向应用的 AI 模型服务（Gemini Nano）
com.google.ar.core  # Google Play Services for AR：应用中的增强现实 *
com.google.ambient.streaming  # Cross-device services：向 Chromebook 和其他设备投屏应用
com.google.android.mosey  # Quick Share（扩展）：在设备之间发送文件
com.google.android.apps.carrier.carrierwifi  # Google Wi-Fi Provisioner：自动连接运营商的 Wi-Fi（Google Fi）
com.google.android.apps.setupwizard.searchselector  # 首次设置时选择搜索引擎 *
com.android.hotwordenrollment.okgoogle  # “OK Google”语音训练 *
com.android.hotwordenrollment.xgoogle  # “OK Google”语音训练 *
# com.google.android.as  # Android System Intelligence：实时字幕、实时翻译、智能自动旋转。已禁用：这些功能会消失，我也不知道 Moto 里还有什么依赖它。要移除就与 as.oss 一起移除
# com.google.android.as.oss  # Private Compute Services：据 Google 的说明，是 Android System Intelligence 的私密分析和训练服务。出于同样的原因已禁用。要移除就与 as 一起移除
EOF
```

**2. Moto 和合作伙伴：23 个包（在电脑上）**

里面有：

- Play Auto Install（`android.autoinstalls.config.motorola.layout`、`playautoinstallext`）：Play 商店自行安装的应用列表。
- Qira Core（`lenovo.qiracore`）：Motorola Qira 的核心。Qira 应用本身已在步骤 5 中移除。
- “Create with AI”（用 AI 制作壁纸）和 Journal（Moto AI 日记）。
- 发给 Motorola 的错误报告：Bug2Go、Moto Feedback Assistant（`help.extlog`）和 Native DropBox Agent（它与 Dropbox 无关）。
- 两套互动壁纸（`blackened_blue`、`lily_white`）。`com.motorola.livewallpaper3` 本身已在步骤 5 中移除。
- Smart Feed：主屏幕左侧的信息流。
- 合作伙伴的占位包：Amazon App Manager、Express Weather、面向印度尼西亚的应用（Etalase、Kompas、Vision+）和面向意大利的应用（Rai News）、Signature Club、PayJoy 客户端，以及四个 ironSource Aura 包（设置过程中的应用推荐）。

**Smart Feed。** 2026 年 5 月，在启动 Amazon 时，这个信息流替换了联盟推广代码。Motorola 称这是[非有意为之](https://www.androidcentral.com/phones/motorola/motorola-affiliate-codes-your-amazon-orders-states-this-was-unintended)，并[修正了设置](https://www.notebookcheck.net/Motorola-comments-on-the-debacle-surrounding-hijacked-shopping-apps.1308600.0.html)。没有这个信息流时主屏幕的表现，我没有检查过：见第 6 项。如果主屏幕出现问题，请把 `com.motorola.smartfeed` 找回来。

**PayJoy。** 这是分期付款购买手机时用的锁机客户端。如果手机不是分期购买的，就移除它。如果是分期购买的，请删掉这一行：没有客户端时锁机会怎样，我没有检查过。

有六行是禁用的。Screenshot Editor（`screenshoteditor`：截图编辑、长截图、GIF）和“受保护文件夹”（`securevault`）因其功能而保留：如果移除第二个包，文件夹里的文件在你把它找回来之前将无法访问。带问号的四个包（`installer`、`appforecast`、`motosecurecore`、`securityhubext`）之所以禁用，是因为我没有查清它们的作用。关于 `motosecurecore`，只知道步骤 5 中移除的 Moto Secure 包引用了它。

创建清单（在电脑上）：

```
cat > moto-more.txt <<'EOF'
# 第一份清单中 Moto 和合作伙伴的残留。每行格式：包名  # 名称。
# 没有星号：用途在公开资料中有描述。有星号：公开资料中没有描述，用途是根据清单、APK 名称或相关的包推断的。
# 以 # 开头的行是禁用的：要么这个功能可能会用到，要么用途不明（带问号）。如果你想移除其中某个，请删掉 # 再执行移除。
android.autoinstalls.config.motorola.layout  # Play Auto Install：Play 商店自行安装的应用列表 *
com.motorola.playautoinstallext  # Moto：Play Auto Install 的扩展 *
com.lenovo.qiracore  # Qira Core：Motorola Qira 的核心，应用本身已被移除 *
com.motorola.createwithai  # Create with AI：用 AI 制作壁纸
com.motorola.journal  # Journal：Moto AI 日记（笔记、照片、音频、AI 摘要）
com.motorola.help.extlog  # Moto Feedback Assistant：“Moto：反馈”的扩展 *
com.motorola.livewallpaper3.prebuilt.blackened_blue  # Moto 互动壁纸：blackened_blue 套装 *
com.motorola.livewallpaper3.prebuilt.lily_white  # Moto 互动壁纸：lily_white 套装 *
com.motorola.smartfeed  # Smart Feed：主屏幕左侧的信息流；2026 年 5 月启动 Amazon 时替换了联盟推广代码
com.motorola.bug2go  # Bug2Go：为 Motorola 收集错误报告 *
com.motorola.android.nativedropboxagent  # Native DropBox Agent：Moto 的服务代理，与 Dropbox 无关 *
com.amazon.appmanager  # Amazon App Manager：Amazon 应用的安装器 *
com.handmark.expressweather  # Express Weather：合作伙伴的天气占位应用 *
id.mjs.etalaseapp  # Etalase Apps：面向印度尼西亚的合作伙伴应用 *
com.ertanto.kompas.official  # Kompas.com：面向印度尼西亚的新闻应用 *
it.rainet.android.rainews  # Rai News：新闻，面向意大利的合作伙伴应用 *
com.signatureclub.prod  # Signature Club：合作伙伴应用 *
com.zte.iptvclient.android.idmnc  # Vision+：电视，面向印度尼西亚的合作伙伴应用 *
com.payjoy.access  # PayJoy：分期付款购买手机的锁机客户端（如果手机是分期购买的，请删掉这一行） *
com.ironsource.appcloud.oobe.hutchison  # ironSource Aura（Hutchison）：设置过程中的应用推荐 *
com.ironsrc.aura.tmo  # ironSource Aura（T-Mobile）：设置过程中的应用推荐 *
com.orange.aura.oobe  # ironSource Aura（Orange）：设置过程中的应用推荐 *
com.aura.oobe.vodafone  # ironSource Aura（Vodafone）：设置过程中的应用推荐 *
# com.motorola.screenshoteditor  # Screenshot Editor：截图编辑、长截图、GIF。已禁用：这个功能可能会用到
# com.motorola.securevault  # Moto 的“受保护文件夹”（Secure folder）。已禁用：里面的文件在你把这个包找回来之前将无法访问
# com.motorola.installer  # ? Moto Installer：Moto 的应用安装服务；不排除它与更新有关
# com.motorola.appforecast  # ? Moto App Forecast：主屏幕上的应用建议，根据名称推断
# com.motorola.motosecurecore  # ? Moto Secure 的核心：已移除的 securityhub 引用了它，还有什么依赖它我不知道
# com.motorola.securityhubext  # ? Moto Secure 的扩展，位于系统分区，根据名称推断
EOF
```

**3. 运营商的占位包：39 个（可选，在电脑上）**

这些是 Verizon、T-Mobile 和 Metro、AT&T、Xfinity Mobile、Spectrum Mobile、Sprint、Telekom 和 Orange 的应用与扩展、运营商的云服务和预装联系人，以及两个面向美国网络的 Google 包（CBRS 专用网络监测，以及 Wi-Fi 通话时紧急呼叫使用的地址）。它们在公开资料中没有逐个的说明，用途从包名和 APK 文件名就能看出来。只有使用这些运营商的 SIM 卡时才需要它们。这份清单是可选的：跳过它，什么都不会改变。如果你使用 Wi-Fi 通话，请删掉 `com.google.android.wfcactivation` 这一行。有一行是禁用的：`com.motorola.brapps` 从名称看像是巴西运营商的应用，但我不确定。

创建清单（在电脑上）：

```
cat > carriers.txt <<'EOF'
# 美国和欧洲运营商的占位包。每行格式：包名  # 名称。只有使用这些运营商的 SIM 卡时才需要。
# 用途从包名和 APK 文件名就能看出来，这些包在公开资料中没有逐个的说明。
com.vzw.apnlib  # Verizon：APN 设置库
com.vzw.ecid  # Verizon：ECID 来电识别
com.vzw.hss.myverizon  # Verizon：My Verizon 应用
com.verizon.obdm  # Verizon：OBDM 服务（占位）
com.verizon.mips.services  # Verizon：MVS Lite 服务
com.vcast.mediamanager  # Verizon Cloud：备份和媒体
com.securityandprivacy.android.verizon.vms  # Verizon：Security & Privacy（VMS）
com.motorola.omadm.vzw  # Verizon：设备管理（OMA-DM）
com.motorola.vzw.cloudsetup  # Verizon：云服务设置
com.motorola.vzw.pco.extensions.pcoreceiver  # Verizon：PCO 接收器
com.motorola.vzw.phone.extensions  # Verizon：拨号器扩展
com.motorola.vzw.settings.extensions  # Verizon：设置扩展
com.motorola.vzw.provider  # Verizon：统一设置（provider）
com.motorola.vzw.voicemail.extensions  # Verizon：语音信箱扩展
com.motorola.android.buacloudcontactadapter  # 运营商：云端联系人（BUA）
com.tmobile.pr.adapt  # T-Mobile：Adapt 客户端
com.tmobile.echolocate  # T-Mobile EchoLocate：网络数据
com.tmobile.echolocate.system  # T-Mobile：流量统计（EchoLocate）
com.tmobile.rsuadapter.qualcomm  # T-Mobile：远程 SIM 解锁（RSU），适配器
com.tmobile.rsusrv  # T-Mobile：RSU，服务
com.tmobile.rsuapp  # T-Mobile：RSU，应用
com.tmobile.m1  # T-Mobile：主屏幕左侧的屏幕
com.motorola.tmo.settingsext  # T-Mobile：设置扩展
com.motorola.tmo.setupext  # T-Mobile：设置向导扩展
com.metro.minus1  # Metro by T-Mobile：主屏幕左侧的屏幕
com.metropcs.metrozone  # Metro by T-Mobile：MetroZone
com.motorola.att.phone.extensions  # AT&T：拨号器扩展
com.motorola.attvowifi  # AT&T：Wi-Fi 通话
com.xfinitymobile.cometcarrierservice  # Xfinity Mobile：运营商服务
com.motorola.comcastext  # Xfinity Mobile：扩展
com.motorola.comcast.settings.extensions  # Xfinity Mobile：设置扩展
com.spectrum.cm.headless  # Spectrum Mobile：运营商服务
com.motorola.spectrum.setup.extensions  # Spectrum Mobile：设置向导扩展
com.motorola.sprint.setupext  # Sprint：设置向导扩展
de.telekom.tsc  # Telekom：App Assistant，应用推荐
com.orange.update  # Orange：运营商更新
com.motorola.contacts.preloadcontacts  # 运营商预装的联系人
com.google.android.apps.cbrsnetworkmonitor  # CBRS：美国的专用网络监测
com.google.android.wfcactivation  # Operator setup：美国运营商 Wi-Fi 通话时紧急呼叫使用的地址
# com.motorola.brapps  # ? BRApps2：从名称看像是巴西运营商的应用，但我不确定
EOF
```

**4. 执行移除并重启手机（在电脑上）**

```
./debloat.sh remove google-more.txt
./debloat.sh remove moto-more.txt
./debloat.sh remove carriers.txt
sleep 20
adb shell svc power reboot
```

脚本的回答与步骤 4 中相同：`Success` 表示该包已被移除，“手机上没有或已被移除，跳过”表示你的手机上没有这个包。如果你没有运营商清单，脚本会提示没有找到该文件：这是正常的。如果某一行显示的不是 `Success` 而是 `Failure: package is non-disable`，请像步骤 5 第 4 项那样，把输出发到文首的聊天群里。`sleep 20` 的暂停和正常重启的原因与步骤 4 中相同。

**5. 检查这些包是否已被移除（在电脑上）**

这条命令应该什么都不打印：

```
grep -hEv '^[[:space:]]*(#|$)' google-more.txt moto-more.txt carriers.txt 2>/dev/null | tr -d '\r' | awk '{print "package:" $1}' > names-5a.txt
adb shell pm list packages --user 0 | tr -d '\r' | grep -Fxf names-5a.txt
```

第一行把三份清单中已启用的行里的包名收集到文件 `names-5a.txt` 中，第二行显示其中仍在手机上的包。如果打印出了什么，请重复第 4 项中的移除，并再次带暂停地重启手机。

**6. 在手机上检查**

- 能打电话和收发短信，移动网络可以上网。如果你使用 Wi-Fi 通话，也请检查一下。
- 主屏幕可以打开，在主屏幕上向侧面滑动不会卡死（Smart Feed 就在那里）。
- 设置可以打开，任何地方都没有弹出“……已停止运行”的窗口。

如果出了问题，把这些包找回来（在电脑上）：

```
./debloat.sh restore google-more.txt
./debloat.sh restore moto-more.txt
./debloat.sh restore carriers.txt
sleep 20
adb shell svc power reboot
```

单个包请用“回滚”一节中的命令找回来。这三份清单也包含在更新之后的检查中（步骤 10），这样已移除的包就不会悄悄回来；工作资料（步骤 11）中也用到它们。

**我保留了什么，为什么**

在我过了一遍的 277 个包中，193 个保留。overlay 和 APEX 模块是为了完整才加进表里的。

| 分组 | 数量 | 里面有什么，为什么保留 |
| --- | --- | --- |
| Overlay | 171 | 它们让系统适配屏幕、运营商和主题。不能移除。 |
| APEX 模块 | 30 | Android 中单独更新的部分（蓝牙、紧急警报等）。不能移除。 |
| Android 核心 | 67 | 所有名为 `android` 或 `com.android.*` 的包：联系人、SIM 卡菜单、打印、数据提供程序等。依据“不能移除的内容”一节中的规则。 |
| Qualcomm | 29 | `com.qualcomm.*`、`com.qti.*`、`vendor.qti.*`、`org.codeaurora.ims`：基带、GPS、Wi-Fi、声音。 |
| 系统用的 Google 模块 | 11 | `packageinstaller`、`documentsui`、`networkstack`、`captiveportallogin`、`ext.shared`、`modulemetadata`、`mainline.adservices`、`mainline.telemetry`、`configupdater`、`webview`、`carrier`。安装应用、文件、网络、WebView、设置更新。 |
| Moto 核心 | 14 | `launcher3`、`launcherconfig`、`camera5`、`camera.ai.colortone`、`faceunlock`、`batterycare`、`coresettingsext`、`providers.chromehomepage`、`providers.settings`、`systemserver`、`hce`、`thermalservice`、`ccc.devicemanagement`、`ccc.ota`。主屏幕、相机、人脸解锁、电池、设置、更新和设备管理。 |
| Moto 服务包 | 17 | `motocare`、`paks`、`paks.notification`、`motocit`、`hiddenmenuapp`、`cqawizard`、`cloudconfig`、`lifetimedata`、`bach.modemstats`、`wifi.motowifimetrics`、`dciservice`、`nfwlocationattribution`、`slpc_sys`、`revoker.services`、`enterprise.adapter.service`、`enterprise.service`、`imagertuning_w`。诊断、配置、基带、Wi-Fi、地理定位。根据公开数据（Bayton、其他 Moto 机型），`motocit` 和 `bach.modemstats` 声明了联网权限，`motocare`、`hiddenmenuapp` 和 `nfwlocationattribution` 没有，其余的则没有关于联网权限的数据。移除它们几乎没有收益，所以可以不移除这些包，只禁止它们联网（步骤 12a）。 |
| 折叠屏和桌面模式 | 5 | `freeform`、`systemui.desk`、`mobiledesktop.core`、`laptoppanel`、`motcameradesktop`。Razr Fold 需要它们。 |
| Moto 功能 | 23 | `actions`、`actioncore`、`touchshare`、`smartmode`、`deptheffect`、`easyprefix`、`dynamicvolume`、`dolbyvisionservice`、`dolby.dolbyui`、`daxservice`、`clockface`、`personalize`、`activestylus`、`mykey`、`smart5g`、`sstservice`、`aptxmode`、`motosignature`、`motosignature2`、`appdirectedsmsproxy`、`iotservice`、`account`、`gesture`。Moto 手势和动作、声音、手写笔、Moto Buds、5G。Dolby 是根据作者的检查保留的：检查显示它没有联网权限。 |
| 首次设置和数据迁移 | 6 | `setup`、`setupwizard.controller`、`setupwizard.devicesetup`、`setupwizard.phoneservice`、`migrateassist`、`motomigrate`。手机恢复出厂设置之后会用到它们。 |
| 电话通信 | 10 | `entitlement`、`omadm.service`、`carrierconfig`、`carriersettingsext`、`visualvoicemail`、`android.provisioning`、`euiccpartnerapp`、`callredirectionservice`、`msimsettings`、`rcsConfigService`。SIM 卡、运营商设置、eSIM、语音信箱、RCS。 |
| 我保留的 Google 包 | 11 | 见下文。 |

**我没有移除的 Google 包**

- `verifier`：Android Developer Verifier 服务，在安装时检查应用的开发者。在首批推出的国家，这项验证[自 2026 年 9 月 30 日起生效](https://androidauthority.com/android-sideloading-developer-verification-first-wave-rollout-3717921)；详情见 [Google 的说明](https://developer.android.com/developer-verification/guides)。如果这个包被隐藏会发生什么，我不知道，所以保留了它。如果重启或更新之后 APK 无法再安装，请先检查这个包是否还在。
- `apps.turbo`：Device Health Services，电池续航估算。没有它，设置 → 电池[可能会显示“Need to pack turbo apk”的消息](https://github.com/ishacker003/turboproxy)。
- `apps.safetyhub`：[Personal Safety](https://sysapps.bayton.org/packages/com.google.android.apps.safetyhub)，紧急功能，例如紧急 SOS 和向联系人发送位置（根据应用说明）。我没有动它。
- `apps.scone`：[Adaptive Connectivity Services](https://9to5google.com/2020/12/05/google-adaptive-connectivity-services/)。从它在 [bayton 数据库](https://sysapps.bayton.org/packages/com.google.android.apps.scone)中的权限来看，它与基带设置和指标有关。我没有动它。
- `euicc`（eSIM 支持）、`apps.restore`（首次设置时的数据迁移）、`apps.wallpaper`（壁纸选择器）、`marvin.talkback`、`accessibility.switchaccess` 和 `tts`（无障碍和语音合成）、`apps.wellbeing`（Digital Wellbeing）：这些是需要或有用的功能，移除的收益很小。

## 步骤 6. 替换拨号器、短信、键盘等

先安装替代应用并把它们设为默认，用命令检查，然后再移除 Google 的版本：否则你会没有通话、短信或键盘可用。

**1. 安装 F-Droid 和替代应用（在手机上）**

1. 在手机上打开浏览器，访问 f-droid.org，下载 F-Droid 商店的 APK。系统询问时，允许浏览器安装应用，并安装这个文件。
2. 打开 F-Droid，等几分钟，让清单更新完。
3. 从表格中找到并安装这些应用。请在主用户中从 F-Droid 安装 Termux：步骤 9 中需要用它一次性开启“Google”用户的自动停止，步骤 11 和 12 中也要用到。不装也行，但那样就得用电脑上的命令手动停止“Google”用户。

| 角色 | 我们要移除的（Google） | 用 F-Droid 中的什么替换 |
| --- | --- | --- |
| 拨号器 | `com.google.android.dialer` | Fossify Phone |
| 短信 | `com.google.android.apps.messaging` | Fossify Messages |
| 联系人 | `com.google.android.contacts` | Fossify Contacts |
| 键盘 | `com.google.android.inputmethod.latin` | Fossify Keyboard，或者支持滑行输入的 HeliBoard 和 FlorisBoard |
| 时钟和闹钟 | `com.google.android.deskclock` | Fossify Clock |
| 日历 | `com.google.android.calendar` | Fossify Calendar |
| 计算器 | `com.google.android.calculator` | Fossify Calculator |
| 照片和图库 | `com.google.android.apps.photos` | Fossify Gallery |
| 浏览器 | `com.android.chrome` | 随你喜欢：Firefox、Brave、Vivaldi |

表中 F-Droid 里的应用名称是我凭印象写的。如果有哪个找不到，就装任何类似的应用。如果在步骤 2 中某个角色显示的包与表中不同，就移除命令显示的那个包。由于 Google 信息被替换，RCS 聊天会消失：普通的短信和彩信会保留。

**2. 迁移联系人**

你在开始之前保存的 `.vcf` 文件在电脑上。请用电脑上的命令把它复制到手机上，并换成你自己的文件名：

```
adb push contacts.vcf /sdcard/Download/
```

然后在手机上打开 Fossify Contacts，菜单 → 导入，选择这个文件。检查联系人是否出现了。

**3. 把这些应用设为默认（在手机上）**

- 电话、短信和浏览器：设置 → 应用 → 默认应用。选择 Fossify Phone、Fossify Messages 和你的浏览器。
- 键盘：设置 → 系统 → 键盘 → 屏幕键盘。开启新键盘并把它设为主键盘。
- 闹钟：设置 → 应用 → Fossify Clock → 电池 → “不受限制”。否则系统可能会让闹钟不响。

Moto 上的菜单项名称可能略有不同。

**4. 检查角色是否已经转移（在电脑上）**

```
adb shell cmd role get-role-holders --user 0 android.app.role.DIALER
adb shell cmd role get-role-holders --user 0 android.app.role.SMS
adb shell cmd role get-role-holders --user 0 android.app.role.BROWSER
adb shell settings get secure default_input_method
```

每条命令都应该给出新应用的名称。Fossify 的包名以 `org.fossify.` 开头。如果你装的是别的键盘或浏览器，名称就会不同。关键是不能再出现 com.google.android.… 或 com.android.chrome 的包。如果还出现，请回到第 3 项。

**5. 移除除拨号器之外的 Google 版本（在电脑上）**

```
cat > google-roles.txt <<'EOF'
# 已经被替换的 Google 版本。每行格式：包名  # 名称
com.google.android.apps.messaging  # Google 信息
com.google.android.contacts  # Google 通讯录
com.google.android.inputmethod.latin  # Gboard
com.google.android.deskclock  # Google 时钟
com.google.android.calendar  # Google 日历
com.google.android.calculator  # Google 计算器
com.google.android.apps.photos  # Google 相册
com.android.chrome  # Chrome
EOF
./debloat.sh remove google-roles.txt
sleep 20
adb shell svc power reboot
```

这里的暂停和正常重启的原因与步骤 4 中相同。

**如果 Gboard 显示 `Failure: package is non-disable`**

包名是对的，问题出在 Motorola 的保护机制上。[根据 XDA 上的报告](https://xdaforums.com/t/uninstall-bloatware-with-adb.3948300/)，在 Moto 手机上，每个受保护的包都会在 `nondisable` 文件夹（通常是 `/product/etc/nondisable`）里记录为一个单独的文件。这个分区是只读挂载的，所以即使有 root 也无法删除这个文件。但可以用 root 模块把它对系统隐藏起来：文件还在原处，系统看不到它，`pm` 就允许移除这个包。[清理这个文件夹的模块](https://xdaforums.com/t/wireguard-kernel-module-for-xt2215-4-motorola-g-stylus-5g-2022.4535379/)已经在其他 Moto 手机上装过。APatch 要把模块文件挂载到系统里，还需要另一个模块，即元模块（metamodule）：我们在 5d 中安装它。在 Razr Fold 上这样做成功了：文件在 /product/etc/nondisable 里，模块连同元模块把它隐藏了，Gboard 也被移除了。你的手机上这个文件现在在哪里、叫什么，由 5b 中的命令显示。如果在任何一步，回答与所写的不一致，请停下，并把输出发到文首的聊天群里。

**5a. 检查屏幕锁（在手机上）**

打开设置 → 安全 → 屏幕锁定（这一项的名称取决于固件）。如果设置的是 PIN 码或图案，请直接进入 5b：它们在锁屏界面上不需要键盘。如果设置的是由字母组成的密码，请先把它改成 PIN 码。重启之后，在第一次解锁之前要输入密码，而此时键盘必须能在解锁前工作（在 Android 中这叫直接启动，Direct Boot）。Gboard 具备这一点。如果你移除了 Gboard，而新键盘不具备，你就无法输入密码，只有恢复出厂设置才能解决。在装有 GrapheneOS 的 Pixel 上[就有这样的案例](https://discuss.grapheneos.org/d/16744-no-keyboard-on-lockscreen-on-pixel-9-pro-with-fossify-keyboard/13)：Fossify Keyboard 被设为主键盘，而密码是由字母组成的。

如果你仍然想保留由字母组成的密码，请用下面这条命令检查所选的键盘（在电脑上）：

```
adb shell cmd package query-services -a android.view.InputMethod | grep -E 'packageName=|directBootAware='
```

在你键盘的包名旁边必须有 `directBootAware=true` 这一行。如果显示的是 `false`，或者根本没有这样的行，你就继续用 PIN 码。这条命令我没有在 Razr Fold 上检查过。

**5b. 在保护清单中找到 Gboard 的文件（在电脑上）**

这条命令会遍历多个分区，最长可能运行一分钟：

```
adb shell "find /system /system_ext /product /vendor /odm -path '*nondisable*' -type f 2>/dev/null" | grep -i inputmethod
```

预期的回答：一行路径，例如 `/product/etc/nondisable/nondisable-com.google.android.inputmethod.latin.xml`。你手机上的文件名可能不同，但很可能包含 `inputmethod.latin`。如果什么都没有打印，请停下：去掉 `| grep -i inputmethod` 再运行同一条命令，并把完整的输出发到文首的聊天群里。这说明保护机制的设置方式不同，下面的步骤就不适用了。

**5c. 制作模块（在电脑上）**

这个代码块会创建带有两个文件的 `gboard-module` 文件夹，并把它们打包成 `unprotect-gboard.zip`。这个模块什么都不删除：它在保护文件上盖一个空的“占位文件”，系统就看不到原文件了。请整块复制：

```
cd ~/razr-debloat
mkdir -p gboard-module
cd gboard-module
cat > module.prop <<'EOF'
id=unprotect_gboard
name=Unprotect Gboard (Motorola nondisable)
version=1.0
versionCode=1
author=local
description=Hides the Motorola nondisable marker of Gboard so that pm uninstall --user 0 works
EOF
cat > customize.sh <<'EOF'
# 对系统隐藏 Motorola 的保护文件（nondisable）。什么都不删除。
# PATTERN：文件名掩码。'*inputmethod.latin*' 表示 Gboard。
PATTERN='*inputmethod.latin*'

FOUND=0
for PART in product system_ext vendor odm system; do
  if [ "$PART" = system ]; then
    SRC=/system/etc/nondisable
    DST=$MODPATH/system/etc/nondisable
  else
    SRC=/$PART/etc/nondisable
    DST=$MODPATH/system/$PART/etc/nondisable
  fi
  [ -d "$SRC" ] || continue
  for F in "$SRC"/$PATTERN; do
    [ -e "$F" ] || continue
    mkdir -p "$DST"
    mknod "$DST/${F##*/}" c 0 0
    ui_print "- Hiding marker: $F"
    FOUND=1
  done
done

[ "$FOUND" = 1 ] || abort "! No marker file matches $PATTERN in /product /system_ext /vendor /odm /system"
set_perm_recursive "$MODPATH/system" 0 0 0755 0644
EOF
python3 -m zipfile -c ../unprotect-gboard.zip module.prop customize.sh
cd ..
adb push unprotect-gboard.zip /sdcard/Download/
```

如果找不到 `python3`，请把含有它的那一行替换为 `zip ../unprotect-gboard.zip module.prop customize.sh`。

**5d. 安装元模块（在电脑和手机上）**

APatch 自己不会把模块文件挂载到系统里：这需要另一个模块，即元模块。没有它，Unprotect Gboard 能无错误地安装，但什么作用也没有。标志是 APM 标签页上的红色提示：“module not mounted, because metamodule not installed”。安装元模块并重启手机之后，这条提示就会消失。如果 APM 里已经有某个元模块，就不需要再装第二个：请直接进入 5e。顺序很重要：先装元模块并重启，然后再装模块。在元模块之前安装的模块不会被挂载：在 Razr Fold 上就必须把它移除再重新安装。

请用 Magic Mount-rs。它在 [APatch 网站的元模块列表](https://apatch.dev/meta-module.html)中，说明里写明支持 APatch，它用 magic mount 方式挂载文件（bind-mount，和 Magisk 一样），不需要内核里有 overlayfs。手机上只能有一个元模块，请不要安装第二个。它每次开机都以 root 身份运行，所以只能从下面的链接下载。已在 Razr Fold 上检查过：它装上了，红色提示消失了，之后 Gboard 的保护文件被隐藏了。

下载它并放到手机上（在电脑上）：

```
cd ~/razr-debloat
curl -fL -o magic_mount_rs-arm64.zip https://github.com/Tools-cx-app/meta-magic_mount-rs/releases/download/v4.1.12/magic_mount_rs-4.1.12-959-arm64.zip
adb push magic_mount_rs-arm64.zip /sdcard/Download/
```

如果 `curl` 打印出错误（例如 404），请在浏览器中打开[发布页面](https://github.com/Tools-cx-app/meta-magic_mount-rs/releases/latest)，下载名称以 `-arm64.zip` 结尾的文件（在 Assets 区域里，而不是“Source code”），并把它放进手机的 Download 文件夹。

安装它（在手机上）：APatch → APM 标签页 → 安装按钮 → 在 Download 文件夹中选择这个 zip。如果日志中出现以 `!` 开头的一行，请停下，并把日志发到文首的聊天群里。安装完成后立刻重启手机（用日志里的按钮，或用电脑上的命令 `adb shell svc power reboot`）：装完元模块之后、安装其他模块之前，[APatch 要求重启](https://apatch.dev/meta-module.html)。

手机启动后，打开 APM：红色提示应该消失了。然后进入 5e。如果 Unprotect Gboard 在装元模块之前就已经装上了，请在 5e 中先把它移除，并重启手机。

**5e. 安装模块（在手机上）**

如果这个模块已经在 APM 列表里，而 Gboard 还没有被移除，请先移除模块（APM → 移除，即 Remove）并重启手机。在元模块之前安装的模块不会被挂载：在 Razr Fold 上只有重新安装才有用。然后打开 APatch 应用，进入 APM 标签页（APatch 模块；KPM 标签页不适用于这一步）。点击安装按钮，在 Download 文件夹（下载）中选择 `unprotect-gboard.zip`。安装日志中应该出现一行 `- Hiding marker:`，后面是 5b 中的路径。然后重启手机：用日志里的按钮，或用电脑上的命令 `adb shell svc power reboot`。

如果日志里显示的是 `! No marker file matches`，说明模块没有装上。这是设计如此：没有可隐藏的东西时它就不会安装。请把 5b 的输出发到文首的聊天群里。

**5f. 检查文件是否已被隐藏（在电脑上）**

重启之后，确认 `adb devices` 显示为 `device`，然后重复 5b 中的命令：

```
adb shell "find /system /system_ext /product /vendor /odm -path '*nondisable*' -type f 2>/dev/null" | grep -i inputmethod
```

现在输出应该是空的。如果 Gboard 那一行还在，说明模块没有生效：不要进入 5g。先看 APM：如果还有关于元模块的红色提示，请重复 5d。如果没有提示，那很可能是模块在元模块之前安装的（在 Razr Fold 上，这一行没消失正是因为这个）。请移除模块（APM → Remove），重启手机，重新安装模块（5e），再重启一次，然后重复这项检查。如果这一行还在，请把 APM 的截图和命令的输出发到文首的聊天群里。

**5g. 移除 Gboard（在电脑上）**

```
./debloat.sh remove google-roles.txt
sleep 20
adb shell svc power reboot
```

现在 `com.google.android.inputmethod.latin` 这一行应该显示 `Success`，而已经被移除的包会回答“手机上没有或已被移除”。重启之后，打开任意输入框：新键盘应该会出现。如果根本没有键盘，请用命令 `adb shell cmd package install-existing --user 0 com.google.android.inputmethod.latin` 把 Gboard 找回来，等待 20 秒，用命令 `adb shell svc power reboot` 重启手机，并把命令 `adb shell settings get secure default_input_method` 的输出发到文首的聊天群里。

**关于这个模块。** 只要 Gboard 处于被移除状态，就不要删除或关闭这个模块，也不要删除或关闭元模块：没有元模块，模块就不起作用；而当文件重新变得可见时，Motorola 会不会自己把受保护的包找回来，我没有检查过。如果固件更新之后 Gboard 又显示 `non-disable`，请打开 APatch → APM：关于元模块的红色提示不应该出现（如果出现了，请重复 5d），而且模块应该在列表里并且处于启用状态。如果一切都正常，回答却还是一样，说明新固件里文件的名称变了：请重复 5b–5f。如果是别的包显示 `non-disable`，请把输出发到文首的聊天群里：模块只需改一行就能修好。

**如果安装模块或元模块之后手机无法启动。** 这种情况不太可能：模块只隐藏一个文件，元模块只负责挂载模块文件。但如果真的发生了，请开启 APatch 的安全模式：其中所有模块都会被关闭，而且在 APM 中“Remove”按钮可用：把你最后安装的那个（元模块或模块）移除。[APatch 的说明](https://apatch.dev/rescue-bootloop.html)：按住电源键直到屏幕亮起，然后按下再松开，直到出现第一个画面。那个页面的标题里还提到了音量“下”键，但没有写明按几次，所以这个方法可能需要尝试几次。如果没有成功，请按末尾“如果手机无法启动”一节操作。

重启之后，检查键盘、短信、闹钟和相机（照片应该能在 Fossify Gallery 中打开）。完整的检查清单在步骤 8。

**6. 最后再移除拨号器，并做检查（在电脑上）**

锁屏上的“紧急呼叫”按钮可能依赖系统拨号器。所以请单独移除拨号器，并立刻检查。这一点已在 Razr Fold 上检查过：移除 Google 电话（默认拨号器是 Fossify Phone）之后，“紧急呼叫”按钮会打开拨号界面，普通的呼出和呼入电话也都正常。没有检查真实的紧急呼叫。如果在步骤 2 中拨号器是另一个包，请替换成那个包。

移除之前，请检查 Fossify Phone 已被设为默认拨号器：命令 `adb shell cmd role get-role-holders --user 0 android.app.role.DIALER` 应该打印 `org.fossify.phone`，而不是 `com.google.android.dialer`。如果打印的是 Google 的，就不能移除拨号器：请先做第 3 项。

```
adb shell pm uninstall -k --user 0 com.google.android.dialer
sleep 20
adb shell svc power reboot
```

开机之后，锁定手机，在锁屏上点“紧急呼叫”。拨号界面应该会打开。不需要真的拨出。然后拨打一个普通号码，并接听一个来电。如果拨号界面没有打开，就把拨号器找回来，并把它留在系统里：

```
adb shell cmd package install-existing --user 0 com.google.android.dialer
```

找回来的拨号器应该不会碍事：只要 Fossify Phone 是默认拨号器，普通通话就都经过它，而系统拨号器只在紧急呼叫时才需要。这是我所知道的情况，没有在你的手机上检查过。

## 步骤 7. 从主用户中移除 Google 服务

这是最后一个移除步骤：退出你的 Google 账号，重启手机检查，然后隐藏 Play 商店、Google Services Framework 和 Google Play 服务。

完成这一步之后，主用户中就没有 Google 了。所有离开 Google 就无法工作的东西，都放在步骤 3 中创建的“Google”用户里。关于哪些功能会停止工作，请见“你会得到什么，代价是什么”一节。

自带后台连接的应用，例如 Signal 和 Telegram，仍然会收到通知，但会更耗电。这是我所知道的情况，没有在你的手机上检查过。其他需要通过 Google 推送的应用，通知会延迟，或者根本不显示。请在“Google”用户中运行这类应用。

**1. 确认“Google”用户可以正常工作**

这个用户必须已在步骤 3 中创建并检查过：完成这一步之后，它是唯一有 Google 的地方。如果你还没有创建它，请回到那一步。如果已经创建，请再简单检查一遍（在电脑上）：

```
GU=$(adb shell pm list users | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
adb shell am switch-user $GU
```

在手机上打开 Play 商店：它应该能加载出来。然后切换回主用户，并把这个用户关掉（这个代码块会在命令之间自己等待五秒）：

```
GU=$(adb shell pm list users | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
adb shell am switch-user 0
sleep 5
adb shell am stop-user -f $GU
```

**2. 在主用户中退出 Google 账号（在手机上）**

设置 → 密码和账号 → Google → 你的账号 → “删除账号”。你已经在步骤 6 中把联系人迁移到了 Fossify Contacts，并把照片和文件保存在了备份里。连同账号一起，这部手机上只存放在账号里的一切内容都会被清除。

**3. 重启，并确认前面的步骤没有弄坏任何东西（在电脑上）**

```
sleep 20
adb shell svc power reboot
```

如果经过步骤 4–6 之后手机运行没有故障，就继续。如果不是，请先处理这些故障：在一部运行不正常的手机上，没有必要去移除 Google 服务。

**4. 创建清单并执行移除（在电脑上）**

行的顺序很重要：Google Play 服务最后移除。

如果某一行得到的回答是 `Failure: package is non-disable`，请停下，并把完整的输出发到文首的聊天群里。步骤 6 中的模块只针对 Gboard 设置，要用于这些包就得重做。在 Razr Fold 上它们没有保护：五个包都回答了 Success。

```
cat > gms.txt <<'EOF'
# Google 服务。每行格式：包名  # 名称
com.android.vending  # Google Play 商店
com.google.android.gsf  # Google Services Framework
com.google.android.gms.location.history  # Google 位置记录（如果存在）
com.google.android.gms.supervision  # Google 家长控制，Family Link（如果存在）
com.google.android.gms  # Google Play 服务
EOF
./debloat.sh remove gms.txt
sleep 20
adb shell svc power reboot
```

**5. 检查手机**

开机后等待两分钟。然后在电脑上检查被移除的东西有没有回来：

```
adb shell pm list packages --user 0 | grep -E 'package:(com.android.vending|com.google.android.gsf|com.google.android.gms)'
```

这条命令应该什么都不打印（`overlay.gmsconfig` 这类行在这里不会匹配：它们不是应用）。在 Razr Fold 上，经过暂停和正常重启之后，输出是空的。如果打印出了什么，说明被移除的东西又回来了：请重复第 4 项，不要跳过暂停，并把输出发到文首的聊天群里。如果一切都干净，就去走一遍步骤 8 的检查清单。如果不断弹出“应用已停止运行”的消息，请用命令 `./debloat.sh restore gms.txt` 把这些服务找回来，等待 20 秒，用命令 `adb shell svc power reboot` 重启手机，并记下消息里的应用名称。

## 步骤 8. 检查手机是否正常工作

重启之后请把这份检查清单过一遍：只要有一项不正常，就趁你还记得最后改了什么，去看“回滚”一节。

| 检查项 | 怎么做 | 应该出现什么 |
| --- | --- | --- |
| 开机 | `adb shell svc power reboot`，然后解锁手机 | 主屏幕可以打开，没有“应用已停止运行”的消息 |
| 设置 | 打开设置，进入应用、电池和显示 | 所有页面都能打开 |
| 通话 | 拨打一个普通号码，并接听一个来电 | 双方都能听见 |
| 紧急呼叫 | 锁定屏幕，点“紧急呼叫” | 拨号界面会打开。不需要真的拨出 |
| 短信 | 发送并接收一条短信 | 消息能发出，也能收到 |
| 移动数据 | 关闭 Wi-Fi，打开任意网站 | 页面能加载 |
| Wi-Fi 和蓝牙 | 连接一个网络和一副耳机 | 两者都正常 |
| NFC | 在下拉通知栏或设置中开启和关闭 NFC | 开关正常 |
| 解锁 | 锁定手机，再用指纹或人脸解锁 | 解锁正常 |
| 相机 | 拍一张照片和一段视频，从缩略图打开拍好的照片 | 照片在 Fossify Gallery 中打开 |
| 相机联网 | 如果你做了步骤 12：读取该步骤第 7 项中的日志 | 日志里有“com.motorola.camera5: 已发送禁止指令”（block-net.list 里的其他包也一样）和“完成”这样的行，并且没有错误文本（只有带 get-package-networking-enabled 的那一行允许出现错误） |
| 折叠和外屏 | 折叠再展开手机，在外屏上打开一个应用 | 屏幕切换没有故障 |
| 声音 | 播放音乐或视频，打开设置 → 声音 | 声音正常播放，“声音”页面能打开 |
| 手写笔 | 如果你移除了手写笔的包（步骤 5 第 5 项）：检查手写笔的连接，在改变压力的同时画一条线，按一下侧键 | 手写笔已连接，压感和侧键都正常 |
| 亮度和充电 | 如果你移除了 Moto AI 的包（步骤 5 第 6 项）：打开设置 → 显示，检查自适应亮度，连接充电器 | 亮度会随光线调节，充电正常 |
| 键盘 | 在任意输入框中输入文字 | 新键盘会出现 |
| 闹钟 | 设为两分钟之后，然后关闭屏幕 | 闹钟会响 |
| GPS | 到室外，打开不依赖 Google 的地图（例如 F-Droid 中的 OsmAnd 或 Organic Maps） | 能定位到位置。第一次搜索可能要几分钟：已经没有 Wi-Fi 定位了 |
| Google 用户 | 开启“Google”用户（步骤 9），打开 Play 商店 | Play 商店可以正常使用。然后再把这个用户关掉 |

如果某个应用崩溃了，请在电脑上看看到底是什么崩溃了：

```
adb logcat -b crash -d | tail -n 60
```

在输出中找到带有 `Process:` 的那一行：里面有正在崩溃的包的名称。如果这个包在你的某份清单里，请用“回滚”一节中的命令把它找回来。

## 步骤 9. 如何开启和关闭“Google”用户

通过切换用户来开启这个用户，并用一条命令让关闭变成自动的：这样只要回到主用户，“Google”用户就会自己停止。

这个用户在后台运行时，会消耗电量和流量。Android 默认不会自动停止后台用户，所以必须明确开启这项设置。

**1. 一次性操作：开启自动停止（在手机上，在 Termux 中）**

你需要有 root 权限的 Termux。如果你还没有 Termux，请在主用户中从 F-Droid 安装它。要给它 root 权限，请打开 APatch Manager 中的 SuperUser 标签页，找到 Termux 并为它开启 root；如果这个标签页的名称不同，就找带有 root 开关的应用列表。然后打开 Termux（在主用户中），输入 `su -c id`：它应该打印 `uid=0(root)`。接着输入下面的命令（这个代码块可以整块粘贴）：

```
GU=$(su -c 'pm list users' | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
su -c "pm set-user-restriction --user $GU no_run_in_background 1"
```

这条命令禁止“Google”用户在后台运行。根据 Android 源码，这样的用户会在你切换到另一个用户时被停止。这一点可以在 [AOSP 的提交（/e/OS 镜像中的副本）](https://gitlab.e.foundation/e/os/android_frameworks_base/-/commit/ffbe9543d74665595401be61f7a1ad71612c5ed0)中看到：作者们手动检查过该用户确实会停止。在作者的 Razr Fold 上，这条命令可以正常工作，成功时什么都不打印，而用户是否真的停止，请在第 3 项中检查。如果这条命令打印了错误，或者给 Termux 开启 root 没有成功，请跳过这一项，并像第 3 项所述那样手动停止这个用户。

为什么不在电脑上操作：在 Razr Fold 上，同样的命令从 adb shell 执行，得到的回答是 `SecurityException: You need MANAGE_USERS permission to: setUserRestriction`。这个固件上的 adb shell 没有 MANAGE\_USERS 权限，而限制用户需要它。root 能通过这项检查，所以请在 Termux 中通过 su 运行上面的命令，而不是在电脑上。

**2. 需要 Google 应用时，开启这个用户**

- 不用 root：设置 → 系统 → 多用户 → “Google”。如果你有用户图标，也可以在锁屏或下拉通知栏里用它来完成。如果设置里没有“多用户”这一项，请用下面电脑上的命令。
- 用电脑：使用列表下面的那个命令块。
- 用主屏幕上的按钮：见下面的“Termux 中的按钮”。

用电脑开启（在电脑上）：

```
GU=$(adb shell pm list users | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
adb shell am switch-user $GU
```

**3. 关闭这个用户，并检查它是否已停止**

回到主用户：设置 → 系统 → 多用户 → 第一个用户（机主）。等待十秒，然后在电脑上检查：

```
adb shell pm list users
```

带有 `Google` 的那一行不能有 `running` 这个词。如果有，说明自动停止没有生效。那么每次使用之后，请用电脑上的命令手动停止这个用户：

```
GU=$(adb shell pm list users | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
adb shell am stop-user -f $GU
```

如果你做了下面第 4 项中的按钮，可以用 `google-off` 按钮停止这个用户：它会自己找到用户编号。

**4. Termux 中的按钮（可选）**

如果你想不用电脑、点一下就开启这个用户，请创建两个脚本。你已经在第 1 项中为 Termux 开启了 root。

1. 从 F-Droid 安装 Termux:Widget 应用。它必须与 Termux 来自同一个来源。
2. 在 Termux 中输入这些命令（这个代码块可以整块粘贴）：

```
mkdir -p ~/.shortcuts
cat > ~/.shortcuts/google-on.sh <<'EOF'
#!/data/data/com.termux/files/usr/bin/sh
GU=$(su -c 'pm list users' | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
[ -n "$GU" ] && su -c "am switch-user $GU"
EOF
cat > ~/.shortcuts/google-off.sh <<'EOF'
#!/data/data/com.termux/files/usr/bin/sh
GU=$(su -c 'pm list users' | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
[ -n "$GU" ] && su -c "am stop-user -f $GU"
EOF
chmod +x ~/.shortcuts/google-on.sh ~/.shortcuts/google-off.sh
```

3. 长按主屏幕上的空白处 → 小部件 → Termux:Widget，并放置这个小部件。点 `google-on` 即可切换到“Google”用户。

`google-off` 脚本只能在主用户中使用。如果第 1 项中的自动停止已经生效，它几乎用不上。

**提示：给“Google”用户设置 PIN 码**

没有锁的话，任何解锁了手机的人都能打开这个用户，而你的 Google 账号就在里面。请切换到它并设置 PIN 码：设置 → 安全 → 屏幕锁定。

## 步骤 10. 刷入内核或更新系统之后

刷入带有补丁内核的 boot 镜像不会动 data 分区，而被隐藏应用的清单就存放在那里；系统更新之后，你需要检查有没有包回来了，并重新执行移除。

**为什么被隐藏的应用应该保持隐藏**

被隐藏包的清单在 `/data/system/users/0/package-restrictions.xml` 中。`fastboot flash boot` 命令只替换 boot 镜像。采用 A/B 方案的更新会把新系统写入相邻的槽位。data 分区保持不变。更新只会添加以前不存在的新包。这一点来自 Android 的构造方式。我没有在你的手机上检查过，所以第一次刷入之后，请做下面的检查。

**刷入内核之后的检查（在电脑上）**

对所有清单再执行一遍移除。脚本会跳过已隐藏的包，并移除回来的那些包：

```
./debloat.sh remove google.txt
./debloat.sh remove moto.txt
./debloat.sh remove moto-extra.txt
./debloat.sh remove moto-ai.txt
./debloat.sh remove google-roles.txt
./debloat.sh remove gms.txt
./debloat.sh remove google-more.txt
./debloat.sh remove moto-more.txt
./debloat.sh remove carriers.txt
```

如果每一行都显示“手机上没有或已被移除”，说明没有包回来。如果某处显示 `Success`，说明那个包回来了，现在又被移除了。如果你没有 `moto-extra.txt` 或 `moto-ai.txt` 这两个文件（你跳过了步骤 5 的第 4 项或第 6 项），脚本会提示没有找到该文件：这是正常的。步骤 5a 的 `google-more.txt`、`moto-more.txt` 和 `carriers.txt` 也是一样。拨号器和手写笔不在这些清单里，请单独检查（在电脑上）：

```
adb shell pm list packages --user 0 | grep -E 'com.google.android.dialer|com.motorola.stylus'
```

如果打印出了包名完全一致的一行，而且你之前移除过这个包（拨号器在步骤 6 中，其他的在步骤 5 中），请用命令 `adb shell pm uninstall -k --user 0 PACKAGE_NAME` 再次移除它，并换上那一行里的名称。如果你有意保留了某个包，它那一行就会被打印出来：这是正常的。不要动名称不同的行（例如以 `overlay` 结尾的行）。然后像步骤 4 那样，带暂停地重启手机：

```
sleep 20
adb shell svc power reboot
```

**Shelter 工作资料。** 如果你清理过它（步骤 11），在这次重启之后，请在其中重复步骤 11 的第 4–6 项：脚本会把在那里回来的包也一并移除。如果手机上已经没有 `profile-remove.sh` 文件，请从第 3 项开始。

**系统更新之后查找新包（在电脑上）**

系统更新之后（不只是刷内核），还要查找新包。把今天的清单与步骤 2 中“之前”的快照做对比：

```
adb shell pm list packages -f -s -u --user 0 | sort > baseline_after.txt
sed 's/.*=//' baseline_before.txt | sort > names_before.txt
sed 's/.*=//' baseline_after.txt | sort > names_after.txt
comm -13 names_before.txt names_after.txt
```

最后一条命令会打印出不在“之前”快照中的包。如果其中有 Google 或 Moto 的应用，就把它们加到合适的清单里并执行移除。在弄清楚是什么之前，不要动不在本指南清单里的包。可以这样往清单里添加一行：

```
echo 'com.example.newpackage  # 名称' >> moto.txt
```

**空中升级（OTA）系统更新之前**

目前我们先不做更新。等你要做的时候，请注意以下几点。

- 不要隐藏 `com.motorola.ccc.ota`：它是 Motorola 的更新客户端。如果你移除了 `com.motorola.ccc.notification`（它在步骤 5 的 Moto 清单里），请在更新之前用命令 `adb shell cmd package install-existing --user 0 com.motorola.ccc.notification` 把它找回来；更新之后，重新运行清单时（上面的检查）它会再次被移除。这是一种预防措施，我没有检查过。
- 如果你有一个已清除 Moto 和 Google 垃圾的 Shelter 工作资料（步骤 11），更新之后请在其中重复步骤 11 的第 4–6 项：脚本会再次移除回来的包。
- 在 boot 已打补丁的手机上，OTA 可能无法通过校验。对已 root 设备的通用做法是：在 OTA 之前用命令 `fastboot flash boot boot_stock.img` 换回原厂 boot，安装更新，首次开机之后，在 APatch 中重新给新的 boot 打补丁并刷入。我没有检查过这在 Razr Fold 上配合 APatch 的效果如何。
- 用新系统首次开机之后，请做上面的两项检查：重新运行清单，以及查找新包。如果你做了步骤 12，请重复它的第 7 项（改动内核之后还要重复第 3 项，即 Termux 的检查）：更新或改动内核之后，这个禁止联网的设置可能没有生效。如果你还禁止了其他包的联网（步骤 12a），请重复它的第 1–3 项，并把清单与上一次的对比：新出现的有联网权限的包，用同一张表来处理。如果你清理过 Shelter 工作资料（步骤 11），请重复它的第 4–6 项。然后过一遍步骤 8 的检查清单。

**Gboard 保护模块和元模块。** 它们在 `/data/adb/modules` 中，系统更新不会动 `/data`，但这个模块只有在 APatch 处于活动状态时才起作用，也就是 boot 重新打上补丁之后。如果重新运行清单时 Gboard 显示 `non-disable`，请打开 APatch → APM：关于元模块的红色提示不应该出现（如果出现了，请重复 5d），而且模块应该在列表里并且处于启用状态。如果一切都正常，回答却还是一样，请重复步骤 6 中的 5b–5f。

## 步骤 11. Shelter 工作资料：像主用户一样清除 Moto 和 Google 垃圾

步骤 4–7 中的命令只在主用户中移除应用：它们都带有 `--user 0`。Shelter 创建的工作资料是手机上一个独立的用户，有它自己的被隐藏应用清单，而 Moto 和 Google 的东西会在里面再次出现。这一步在那里移除的，是我们在主用户中移除过的同样的东西：Google 应用（`google.txt`）、Moto 垃圾（`moto.txt`、`moto-extra.txt`、`moto-ai.txt`）、`google-roles.txt` 中的 Google 版本、Google 拨号器、Moto 的“笔记”，以及 `gms.txt` 中的 Google 服务（Play 商店、Services Framework、Google Play 服务）和步骤 5a 的残留（`google-more.txt`、`moto-more.txt`、`carriers.txt`）。

安装 Shelter 并创建工作资料时，Google 很可能会在里面再次出现。工作资料是手机上的又一个用户，而 Shelter [只是请求系统添加它](https://discuss.privacyguides.net/t/why-shelters-work-profile-comes-pre-loaded-with-some-google-services/14153)：那里有一位用户写道，刚创建完资料，就发现里面有 3–4 个 Google 应用。这类资料里保留哪些系统应用，由固件决定：厂商有一个[列出工作资料必备应用的文件](https://bayton.org/notes/103/)，而在 [Bayton 的分析](https://bayton.org/android/what-are-vital-apps/)中，Google Play 服务和 Play 商店被列为工作资料的必备应用。Moto 固件会留下什么，我事先并不知道：只有在创建资料之后才能看到。

主用户不会因此改变：每个用户都有自己的被隐藏应用清单，而带 `--user 0` 的命令只动了主用户的那一份。这是 Android 的构造方式；我没有在你的手机上检查过。

请在完成步骤 1–10、主用户已经干净之后再做这一步，并且只在你有或将要有 Shelter 工作资料时才做。你需要有 root 权限的 Termux。如果还没有，请在主用户中从 F-Droid 安装 Termux，并按步骤 9 第 1 项中的操作为它开启 root。

**1. 创建工作资料（在手机上）**

安装 Shelter 并完成它的设置：它会自己创建工作资料。

**2. 查出工作资料的编号（在电脑上）**

```
adb shell pm list users
```

找到工作资料的新增那一行：名称类似“Work profile”（如果手机不是英文界面，则是它的译名，例如“工作资料”）。`{` 后面的数字就是它的编号，下面一律用 12。**这个编号必须正好是工作资料的编号。** 如果你填成了“Google”用户的编号，脚本就会把 Google 从那里移除。

**3. 用你的清单生成脚本（在电脑上）**

adb shell 无法进入工作资料：对于带 `--user 12` 的命令，它会回答 `Shell does not have permission to access user 12`（[示例](https://discuss.grapheneos.org/d/26196-installing-apps-via-adb-to-work-profile)）。在 Android 源码中，这项检查[只对 adb shell 起作用](https://gitlab.e.foundation/e/os/android_frameworks_base/-/commit/8cd28b57ed732656d002d97879e15c5695b54fff)，root 不受它影响。所以由 Termux 中的 root 来运行这些命令，而包名取自你自己的清单：脚本是用 `google.txt`、`moto.txt` 和其他文件生成的。受保护的包（与 `debloat.sh` 中是同一份名单）不会进入脚本。

```
N=12
{ echo com.google.android.dialer; echo com.motorola.stylus; grep -hEv '^[[:space:]]*(#|$)' google.txt moto.txt moto-extra.txt moto-ai.txt google-roles.txt gms.txt google-more.txt moto-more.txt carriers.txt 2>/dev/null; } | tr -d '\r' | awk -v n="$N" '$1 == "com.motorola.ccc.notification" || $1 !~ /^(android|com\.android\.(systemui|settings|phone|server\.telecom|providers\..*)|com\.qualcomm\..*|com\.motorola\.ccc\..*|.*overlay.*|com\.google\.android\.webview|com\.google\.android\.trichromelibrary)$/ { print "pm uninstall -k --user " n " " $1 " 2>&1 | grep -q Success && echo \"已移除: " $1 "\"" }' > profile-remove.sh
adb push profile-remove.sh /data/local/tmp/profile-remove.sh
```

请在第一行把 12 换成你自己的编号。如果你缺少其中的某些文件，这是正常的。Google 拨号器和 Moto 的“笔记”是单独加在清单开头的：在主用户中它们是用各自的命令移除的（步骤 6 和步骤 5）。`com.motorola.ccc.notification` 这个包在工作资料中像步骤 5 那样，作为普通包被移除；脚本仍然会跳过其他的 `com.motorola.ccc.*`。步骤 5a 的清单也是以同样的方式接入的。这条命令会悄悄跳过不在工作资料里的包。

**4. 运行脚本（在手机上，在 Termux 中）**

```
su -c 'sh /data/local/tmp/profile-remove.sh'
```

每移除一个包，都会打印一行“已移除: …”。脚本会悄悄跳过不在工作资料里的包，以及系统不允许它移除的包：还留着的那些包，会由第 5 项中的检查显示出来。如果你的手机上 root 在 adb 中可用（`adb shell su -c id` 打印 `uid=0(root)`），同样的命令也可以在电脑上运行：`adb shell "su -c 'sh /data/local/tmp/profile-remove.sh'"`。

**5. 检查（在手机上，在 Termux 中）**

```
comm -13 <(su -c 'pm list packages --user 0' | sort) <(su -c 'pm list packages --user 12' | sort) | grep -E 'google|motorola'
```

这条命令会显示出现在工作资料里、但主用户里已经没有的 Google 和 Moto 包。它应该什么都不打印。如果打印出了什么，请把输出发到文首的聊天群里。不要动工作资料里其他的 Google 系统包，例如 `com.google.android.permissioncontroller` 或 `com.google.android.webview`。然后带暂停地重启手机（第 6 项）。

**6. 带暂停地重启手机（在电脑上）**

```
sleep 20
adb shell svc power reboot
```

这里的暂停和步骤 4 中的原因相同。重启之后请重复第 5 项中的检查：它应该仍然什么都不打印。

**如果工作资料里出了问题。** 最简单的办法是删除这个工作资料（步骤 1a）再重新创建。单个包可以用下面的命令找回来（在手机上，在 Termux 中）：

```
su -c 'cmd package install-existing --user 12 PACKAGE_NAME'
```

**系统更新之后。** 如果工作资料的编号没有变，请再次执行第 4–6 项（脚本、检查、带暂停的重启）：脚本会再次移除回来的包。

我在测试清单上运行过脚本的生成和第 5 项中的检查。通过 root 在工作资料中运行脚本，以及工作资料里究竟会留下哪些 Moto 包，我没有在你的手机上检查过。

## 步骤 12. 禁止普通 Moto 相机联网

普通相机 `com.motorola.camera5` 不能移除：没有它手机就无法拍摄，而且它在“不能移除的内容”一节里。所以我们只切断它的联网。INTERNET 权限只说明相机能够访问网络，并不说明它真的在访问。据我所知，普通拍摄并不需要网络。

**工作原理。** Android 在系统层面有针对单个应用的联网禁令：它不使用 VPN，也不需要额外的应用。它由 `cmd connectivity` 命令控制。根据使用同一机制的 ShizuWall 应用的说明，这些命令在 Android 11 及更高版本上有效，而且规则会在每次重启时被重置。所以检查之后，我们会在手机上放一个小脚本：APatch 每次开机都会运行它，并重新开启这个禁令。脚本放在 `/data/adb/service.d` 文件夹里，系统更新或重新刷入 boot 镜像都不会把它抹掉。

请在完成步骤 1–10 之后再做这一步。你需要有 root 权限的 Termux，和步骤 11 中一样。这些命令我没有在你的 Razr Fold 上检查过，所以请先在 Termux 上检查这个禁令（第 3 项），然后再禁止相机联网。

**1. 检查手机是否支持这些命令（在电脑上）**

```
adb shell cmd connectivity help | grep -iE 'chain3|package-networking'
```

应该打印出含有 `set-chain3-enabled` 和 `set-package-networking-enabled` 这两个词的行（旁边可能还有带 `get-` 的行）。如果什么都没有打印，说明这个固件没有这样的命令：请停下，并把 `adb shell cmd connectivity help` 命令的输出发到文首的聊天群里，那里会给出替代方法。

**2. 查明相机是否与其他应用共用同一个编号（在电脑上）**

Android 是按应用编号（uid）而不是按包名来切断网络的。如果其他包与相机共用这个编号，它们也会失去联网。

```
U=$(adb shell pm list packages -U --user 0 | tr -d '\r' | sed -n 's/^package:com\.motorola\.camera5 uid:\([0-9]*\)$/\1/p'); echo "相机编号: $U"; adb shell pm list packages -U --user 0 | tr -d '\r' | grep " uid:$U\$"
```

第一行会显示一个 10000 或更大的数字，它下面应该恰好有一行 `package:com.motorola.camera5 uid:…`。如果这个编号是空的，说明手机上没有这个包：请把命令 `adb shell pm list packages --user 0 | grep camera` 的输出发到文首的聊天群里。如果编号下面有多行，或者编号小于 10000，请停下，并把输出发到文首的聊天群里。

**3. 在 Termux 上检查这个禁令（在手机和电脑上）**

先不要在相机上检查，而是在 Termux 上检查这个禁令：这能说明它在你的固件上是否有效。请先确认 Termux 能联网（在手机上，在 Termux 中）：

```
pkg install -y curl
curl -sS -m 5 -I https://example.com
```

应该打印出响应头，第一行以 `HTTP/` 开头。禁止 Termux 联网（在电脑上）：

```
adb shell cmd connectivity set-chain3-enabled true
adb shell cmd connectivity set-package-networking-enabled false com.termux
```

在 Termux 中重复 `curl -sS -m 5 -I https://example.com`（在手机上）：现在它应该以错误结束，例如 `Could not resolve host` 或 `timed out`。如果 curl 又打印出了响应头，说明这个禁令在你的固件上不起作用：请停下，并把输出发到文首的聊天群里。如果一切如上所述，就把网络还给 Termux（在电脑上）：

```
adb shell cmd connectivity set-package-networking-enabled true com.termux
```

**4. 禁止相机联网（在电脑上）**

```
adb shell cmd connectivity set-chain3-enabled true
adb shell cmd connectivity set-package-networking-enabled false com.motorola.camera5
```

这些命令什么都不打印。打开相机，拍一张照片和一段视频：拍摄应该和以前一样。如果相机打不开，或者整部手机都没有网络了，请用命令 `adb shell svc power reboot` 重启手机：规则会被重置。然后把发生的情况描述发到文首的聊天群里。如果第 1 项中帮助信息的输出里有 `get-package-networking-enabled` 这一行，就可以检查这个禁令：相机得到的回答应该与普通应用（例如 `com.android.settings`）得到的回答不同：

```
adb shell cmd connectivity get-package-networking-enabled com.motorola.camera5
adb shell cmd connectivity get-package-networking-enabled com.android.settings
```

**5. 创建清单和脚本（在电脑上）**

这份清单和其他清单的写法一样：先是包名，然后是 `#` 和名称。要再禁止一个应用联网，就把它那一行加到清单里（命令在步骤 12a 的第 6 项中），然后重复第 6 项和第 7 项。但请先看一遍步骤 12a：它会显示哪些包需要网络、哪些不需要，并检查这个包是否有它自己的编号（uid）。如果这个编号是系统的或是共用的，这个禁令也会把其他包的网络一起切断。

```
cat > block-net.list <<'EOF'
# 禁止联网的应用。每行格式：包名  # 名称
com.motorola.camera5  # 普通 Moto 相机
EOF
```

脚本：

```
cat > block-net.sh <<'EOF'
#!/system/bin/sh
# 禁止清单中的应用联网。APatch 每次开机都会运行这个脚本。
export PATH=/system/bin:/system/xbin:$PATH
LIST=/data/adb/block-net.list
LOG=/data/adb/block-net.log
exec > "$LOG" 2>&1
n=0
until [ "$(getprop sys.boot_completed)" = "1" ] || [ "$n" -ge 150 ]; do sleep 2; n=$((n+1)); done
sleep 15
echo "开始: $(date)"
cmd connectivity set-chain3-enabled true </dev/null
grep -Ev '^[[:space:]]*(#|$)' "$LIST" | tr -d '\r' | while read -r pkg _; do
  cmd connectivity set-package-networking-enabled false "$pkg" </dev/null
  echo "$pkg: 已发送禁止指令"
  cmd connectivity get-package-networking-enabled "$pkg" </dev/null
done
echo "完成"
EOF
```

脚本会等待开机完成，再等 15 秒，然后开启总开关，并对清单中的每个包禁止联网。结果写入 `/data/adb/block-net.log`。带有 `get-package-networking-enabled` 的那一行只是为了写日志：如果你的固件没有这个命令，日志里这一行会显示错误，但禁令仍然有效。

**6. 把文件放到手机上（在电脑和手机上）**

把文件复制到手机上（在电脑上）：

```
adb push block-net.sh /data/local/tmp/block-net.sh
adb push block-net.list /data/local/tmp/block-net.list
```

安装它们（在手机上，在 Termux 中）：

```
su -c 'mkdir -p /data/adb/service.d && cp /data/local/tmp/block-net.sh /data/adb/service.d/block-net.sh && chmod 755 /data/adb/service.d/block-net.sh && cp /data/local/tmp/block-net.list /data/adb/block-net.list && chmod 644 /data/adb/block-net.list && echo 完成'
```

应该打印出“完成”。如果没有，请把全部输出发到文首的聊天群里。

**7. 重启手机并检查日志（在电脑和手机上）**

重启手机（在电脑上）：

```
adb shell svc power reboot
```

解锁手机，等待一分钟，然后读取日志（在手机上，在 Termux 中）：

```
su -c 'cat /data/adb/block-net.log'
```

日志里应该有带日期的 `开始:` 这一行，然后是 `com.motorola.camera5: 已发送禁止指令`，接着是 `get-package-networking-enabled` 的回答，最后是 `完成`。如果文件不存在或是空的，说明 APatch 没有运行这个脚本：请用命令 `su -c 'ls -l /data/adb/service.d/block-net.sh'` 检查权限（这一行应该以 `-rwxr-xr-x` 开头），并把输出发到文首的聊天群里。如果日志里有错误，例如 `Unknown command`，请把完整的日志发到文首的聊天群里。然后过一遍步骤 8 的检查清单。

**8. 如何解除禁令（在电脑和手机上）**

立刻解除，不用重启（在电脑上）：

```
adb shell cmd connectivity set-package-networking-enabled true com.motorola.camera5
```

为了让这个禁令在下次开机时不再自动开启（在手机上，在 Termux 中）：

```
su -c 'rm /data/adb/service.d/block-net.sh'
```

**系统更新和内核改动。** 脚本和清单放在 `/data/adb` 中，所以只要 APatch 在手机上正常工作，系统更新、重新刷入 boot 镜像和更换内核都不会影响它们。在 root 还没有恢复期间（例如在更新之后、按步骤 10 重新刷入 boot 之前），相机的联网是没有被切断的。据我所知，这条规则存放在系统的 BPF 表中，而不是 netfilter 这类内核模块里，所以为 Droidspaces 定制的内核应该不会破坏它；这是我的推测，不是检查的结果。更新或改动内核之后请重复第 7 项，改动内核之后还要重复第 3 项中的 Termux 检查：日志只能说明命令已发送，不能说明禁令已生效。

**我没有在你的手机上检查的内容。** 这个固件上的 `cmd connectivity` 命令本身，APatch 下从 `service.d` 运行脚本的情况，以及禁令能否在更新后保留。所以第 3 项和第 7 项是必做的。这个禁令只适用于主用户：它不影响“Google”用户和 Shelter 工作资料中的相机（如果那里有的话）。

## 步骤 12a. 保留的包中哪些仍能联网

这一步回答本文的主要问题：数据是否会通过我们保留的包离开手机。一个包要发送任何东西，都需要 INTERNET 权限。如果没有这个权限，这个包自己就连不上网络（它只能把数据交给另一个有网络访问权限的应用）。如果有这个权限，也还不能证明它发送了什么：这只说明存在这种可能。没有网络监控工具，就看不到实际的连接，所以我们这样做：找出能联网的包，看看它们为什么需要网络，并为不需要网络的包关闭网络。包仍然留在原处，方法与步骤 12 相同，只用一条命令就能解除禁令。

**我在公开资料中找到的内容。** 我按名称搜索了其余系统服务包的说明。数据最多的是 [Bayton 系统应用数据库](https://sysapps.bayton.org/)：它收集了各种 Moto 机型的包名和权限。这是对样本的描述，不是定论，而且这些数据并非来自 Razr Fold（主要是 edge 50 fusion 和 moto g 系列），所以你手机上的权限可能不同。对于步骤 5a 中的 17 个 Moto 服务包，结果是这样的：两个声明了 INTERNET，三个没有，其余十二个没有关于联网的公开数据；在手机上自己检查的方法见下文。

- [`com.motorola.motocit`](https://sysapps.bayton.org/packages/com.motorola.motocit)（CQATest）：从名称和权限（如 `MOT_CAM_TEST` 和 `QCOM_DIAG`）来看，这是厂商的测试应用，不会显示给用户。它声明了 INTERNET、精确位置、相机、麦克风、身体传感器和手机状态。普通机主用不到它，所以请先为它关闭网络。
- [`com.motorola.bach.modemstats`](https://sysapps.bayton.org/packages/com.motorola.bach.modemstats)（Modem Service）：声明了 INTERNET、位置、读取系统日志和应用使用统计。
- [`com.motorola.ccc.devicemanagement`](https://sysapps.bayton.org/packages/com.motorola.ccc.devicemanagement)（Device Management）：INTERNET、读取日志和已安装应用列表，没有位置权限。它属于 `ccc.*` 系列，该系列与 Motorola 的更新相关，所以脚本不会碰它，你也不应该为它关闭网络。
- [`com.motorola.omadm.service`](https://sysapps.bayton.org/packages/com.motorola.omadm.service)（Carrier Provisioning Service）：INTERNET 以及使用受限运营商网络的权限，没有位置或日志权限。从名称和权限判断，运营商用它按 OMA DM 标准来配置手机。
- [`com.motorola.paks`](https://sysapps.bayton.org/packages/com.motorola.paks)（PAKS）：根据 [Lenovo 论坛](https://forums.lenovo.com/t5/moto-g-5G-2023/PAKS/m-p/5273709)上一位版主的回复，它是用于供应链保护和销售前准备的组件，与较底层的软件层相关。它的权限中有 INSTALL\_EXISTING\_PACKAGES，从技术上讲，这允许把隐藏的包重新找回来。我没有找到它会这样做的数据；如果被移除的包在重启或更新后确实回来了，步骤 10 的检查会发现它们。
- [`com.motorola.motocare`](https://sysapps.bayton.org/packages/com.motorola.motocare)（Device Care）、[`com.motorola.hiddenmenuapp`](https://sysapps.bayton.org/packages/com.motorola.hiddenmenuapp)（HiddenMenu）、[`com.motorola.iotservice`](https://sysapps.bayton.org/packages/com.motorola.iotservice)（Moto Iot Service）和 [`com.motorola.nfwlocationattribution`](https://sysapps.bayton.org/packages/com.motorola.nfwlocationattribution)（Carrier Location Access）没有声明联网权限，所以它们自己不会发送任何东西。它们声明的是别的内容：Device Care 声明了应用使用统计和所有应用的列表，HiddenMenu 声明了位置和通话记录，Iot Service 声明了与 Moto Buds 的连接，Carrier Location Access 声明了位置，包括后台位置。
- [`com.google.mainline.telemetry`](https://sysapps.bayton.org/packages/com.google.mainline.telemetry)（“Support components”）和 [`com.google.mainline.adservices`](https://sysapps.bayton.org/packages/com.google.mainline.adservices)（“Main components”）：Google Play 系统更新模块的服务版本标记；它们没有界面，也不是应用。Privacy Sandbox 广告本身（主题、基于应用的广告、广告衡量）位于 AdServices 模块中（[AOSP 中的说明](https://source.android.com/docs/core/ota/modular-system/adservices)）。Bayton 建议通过 Android 隐私设置来管理它，而不是移除这些包：在“设置”中找到关于广告和 Privacy Sandbox 的部分（在 Moto 上名称可能不同），在那里关闭这些开关。
- 对于 `lifetimedata`（“LifetimeData”）、`dciservice`（“Moto DCI Service”）、`paks.notification`（“PAKS Notification”）、`slpc_sys`（“Motorola Slpc System”）、`revoker.services`（“Revoker Services”）、`enterprise.service`（“Moto Thinkshield-MM”）、`enterprise.adapter.service`（“Enterprise Adapter Service”）和 `wifi.motowifimetrics`（没有名称），只找到了名称和权限数量（从 2 到 19 个），没有说明。这是[来自 edge 50 fusion 包列表](https://sysapps.bayton.org/devices/117)的数据。`cloudconfig`、`cqawizard` 和 `imagertuning_w` 不在其中，我按名称也没有找到它们。

**1. 建立有联网权限的包的清单（在电脑上）**

这条命令会逐个询问主用户的每个系统包是否声明了 INTERNET（跳过 overlay）。它大约运行一分钟，不会改动任何东西：

```
adb shell 'pm list packages -s -U --user 0 | sed "s/^package://" | while read -r p u; do case "$p" in *overlay*) continue;; esac; if dumpsys package "$p" </dev/null | grep -q "android.permission.INTERNET"; then echo "INTERNET $u $p"; else echo "no-net $u $p"; fi; done' | tr -d '\r' | sort -k3 > audit_net.txt
wc -l audit_net.txt
```

文件中会有 `INTERNET uid:10123 com.package`（已声明联网）和 `no-net uid:10124 com.package`（未声明）这样的行。如果文件是空的，说明 adb 看不到手机：请检查 `adb devices`。

**2. 只保留有联网权限的 Moto、Lenovo、Google 和 Qualcomm 的包（在电脑上）**

```
grep '^INTERNET' audit_net.txt | grep -E 'motorola|lenovo|google|qualcomm|qti|codeaurora' > audit_internet.txt
cat audit_internet.txt
```

这是能够访问网络的包的清单。`android` 和 `com.android.*` 这些包不在其中：它们的代码是开放的，而且几乎都是系统本身的一部分。

**3. 去掉不能切断网络的包（在电脑上）**

与步骤 12 第 2 项一样，Android 是按应用编号（uid）而不是按包名来切断网络的。不要碰编号小于 10000 的包（那是系统本身，其他服务会因此失去网络），也不要碰与其他包共用编号的包（共用编号的所有包会同时失去网络）。下面的命令只保留有自己独立编号且编号不小于 10000 的包：

```
awk '{print $2}' audit_net.txt | sort | uniq -d > audit_shared_uids.txt
grep -Ev 'uid:[0-9]{1,4} ' audit_internet.txt | grep -Fvwf audit_shared_uids.txt > audit_can_block.txt
cat audit_can_block.txt
```

`audit_can_block.txt` 里现在是技术上可以关闭网络的包。是否应该关闭，由下一项决定。如果你需要的包不在清单中，说明它没有联网权限，或者它的编号是系统编号或共用编号：那里没有可关闭的内容，或者无法关闭。

**4. 决定为谁关闭网络**

| 包 | 怎么做 | 原因 |
| --- | --- | --- |
| `com.motorola.ccc.*`（Motorola 更新和设备管理） | 不要关闭 | 它们是更新所必需的，而本指南的条件是不能破坏更新。 |
| 与 SIM、eSIM、Wi-Fi 通话、RCS 和运营商设置有关的一切：`entitlement`、`carrierconfig`、`carriersettingsext`、`euicc`、`omadm.service`、`com.google.android.carrier` | 如果你使用移动网络服务、eSIM 或 Wi-Fi 通话，就不要关闭 | 它们通过网络从运营商获取设置。 |
| 网络检查，安装和更新系统数据：`networkstack`、`captiveportallogin`、`packageinstaller`、`configupdater`、`verifier`、`webview` | 不要关闭 | 没有网络，你会失去连接检查、Wi-Fi 登录窗口、应用验证和安装，以及证书更新。 |
| GPS 和位置：`com.qualcomm.*`、`apps.safetyhub` | 不要关闭 | GPS 会下载辅助数据，紧急功能会发送位置。 |
| `paks`、`paks.notification` | 保持原样 | 厂商称 PAKS 是设备保护的一部分，而没有网络时会发生什么还不清楚。 |
| Moto 的诊断、统计、工厂测试、企业管理和云配置：`motocit`、`bach.modemstats`、`lifetimedata`、`dciservice`、`cloudconfig`、`enterprise.service`、`enterprise.adapter.service`、`wifi.motowifimetrics`；Google 的 `apps.turbo` | 关闭网络并测试手机 | 从名称和说明来看，打电话、发短信、相机、Wi-Fi 和更新都不需要它们。 |

如果一个包既不在表中，也不在 `audit_can_block.txt` 中，就不要碰它。任何包的说明都可以在 [Bayton 数据库](https://sysapps.bayton.org/)中按名称找到（页面为 `sysapps.bayton.org/packages/package_name`）。

**5. 试着关闭网络（在电脑上）**

先从一两个包开始。规则立即生效，并在重启时被重置，和步骤 12 中一样，所以不会有任何东西变成永久的：

```
adb shell cmd connectivity set-chain3-enabled true
adb shell cmd connectivity set-package-networking-enabled false com.motorola.motocit
```

在手机重启之前，照常使用它（至少一天），并检查通话、短信、移动网络、Wi-Fi 和相机。如果手机重启过，规则已被重置：请重复命令。如果出了问题，就把网络还回去（在电脑上）：

```
adb shell cmd connectivity set-package-networking-enabled true com.motorola.motocit
```

**6. 让禁令永久生效（在电脑上）**

如果一切正常，就把这个包加到清单里。下面是针对 `com.motorola.motocit` 的命令；对于其他包，请替换那一行：

```
cat >> block-net.list <<'EOF'
com.motorola.motocit  # CQATest：厂商的测试应用
EOF
cat block-net.list
```

输出里应该有相机那一行和新的一行。如果 `block-net.list` 文件已不在电脑上，请按步骤 12 第 5 项重新创建，并把所有行都加进去。然后重复步骤 12 的第 6 项和第 7 项：重启之后，日志里会为清单中的每个包显示一行“已发送禁止指令”。如果某个包导致了问题，就从清单里删掉它的那一行，重复步骤 12 的第 6 项，并用该步骤第 8 项中的命令把网络还回去。

**系统更新或内核改动之后。** 新版本的固件可能带来有联网权限的包。第一次做完第 1–3 项后，请保存清单（在电脑上）：

```
cp audit_internet.txt audit_internet_before.txt
```

更新之后，重复第 1–3 项并进行比较（在电脑上，在 Bash 中）：

```
diff <(cut -d' ' -f3 audit_internet_before.txt) <(cut -d' ' -f3 audit_internet.txt)
```

以 `>` 开头的行是更新之后新出现的：请按上面的表格来处理它们。关闭网络是通过 `/data/adb` 中的脚本实现的，而不是通过移除清单，所以它不会让任何被移除的内容回来。内核改动之后，请再检查一次禁令：步骤 12 的第 3 项和第 7 项。

**我没有检查的内容。** 我只是在 `pm` 和 `dumpsys` 的示例输出上运行了这一步的命令；它们没有在 Razr Fold 上运行过。Bayton 的数据来自其他机型。INTERNET 权限说明的是一种可能性，而不是发送的事实：只有网络监控工具或带连接日志的防火墙，才能证明某个包什么都没有发送，或者抓到它在发送，而这不属于本指南的内容。

## 回滚：出问题时怎么办

任何隐藏操作都可以用一条命令撤销，前提是手机能启动，并且电脑能看到它。本指南中所有移除和找回包的命令都带有 `--user 0` 或 `--user $GU`。在步骤 11 中，用工作资料的编号代替零，这同样没有问题。不要从任何这类命令中去掉这个参数：没有它，pm uninstall 会同时影响所有用户。

**把单个包找回来（在电脑上）**

```
adb shell cmd package install-existing --user 0 com.example.package
```

把 `com.example.package` 替换成你需要的包名。回答 `Package ... installed for user: 0` 说明这个包已经回来了。如果应用没有出现，请等待 20 秒，然后用命令 `adb shell svc power reboot` 重启手机。

**把整份清单找回来（在电脑上）**

```
./debloat.sh restore google.txt
./debloat.sh restore moto.txt
./debloat.sh restore moto-extra.txt
./debloat.sh restore moto-ai.txt
./debloat.sh restore google-roles.txt
./debloat.sh restore gms.txt
./debloat.sh restore google-more.txt
./debloat.sh restore moto-more.txt
./debloat.sh restore carriers.txt
```

只运行你需要的那份清单。把 Google 服务找回主用户之后，请等待 20 秒，用命令 `adb shell svc power reboot` 重启手机，并重新登录你的账号：设置 → 密码和账号。

不在清单中的包，要用“把单个包找回来”一节中的命令逐个找回：它们是拨号器（步骤 6）和步骤 5 中的 `com.motorola.stylus`。包 `com.motorola.ccc.notification` 现在在 `moto.txt` 中，所以 `./debloat.sh restore moto.txt` 也会把它找回来；恢复时，该脚本同样会跳过其他的 `ccc.*` 包。步骤 5a 的清单（`google-more.txt`、`moto-more.txt`、`carriers.txt`）用“把整份清单找回来”那个代码块中的相同命令找回。它们不影响以 `#` 开头的行：那些包本来就没有被移除。联网禁令（步骤 12 和 12a）要单独解除：命令在步骤 12 的最后一项中；对于其他包，请把 `com.motorola.camera5` 替换成它的名称。

**只把 Gboard 找回来（在电脑上）**

```
adb shell cmd package install-existing --user 0 com.google.android.inputmethod.latin
sleep 20
adb shell svc power reboot
```

如果你还想移除步骤 6 中的保护模块，请先用这条命令把 Gboard 找回来，再移除模块：APatch → APM → Remove，然后重启手机。我没有检查过，当一个已被移除的包的保护重新生效时，系统会怎么处理它，所以才按这个顺序。最好把元模块留在原处：没有它，其他模块的文件也不会再被挂载。

**查看现在隐藏了什么（在电脑上）**

```
adb shell pm list packages -s -u --user 0 | sort > all_system.txt
adb shell pm list packages -s --user 0 | sort > visible_system.txt
comm -23 all_system.txt visible_system.txt
```

最后一条命令会打印出主用户中被隐藏的系统包。

**如果手机无法启动**

这种情况应该很少见：清单中的包取自公开的“可以安全移除”清单。风险不是零，所以请按顺序操作。

1. 等待十分钟。移除之后的第一次启动可能会很久。
2. 如果问题是在刷入内核之后开始的，而不是在移除应用之后，就把原厂的 boot 刷回去。手机必须进入 fastboot 模式（先关机，再按住电源键和音量减键），命令在电脑上输入：`fastboot flash boot boot_stock.img`。
3. 如果移除包之后手机陷入重启循环，请在电脑上运行 `adb devices`。如果能看到手机，就用“把单个包找回来”一节中的命令，把你最后移除的那个包找回来。在重启循环中，系统可能没有响应；那就转到第 4 项。
4. 如果 adb 通过 APatch 有 root shell，请在电脑上用一条命令重置整个隐藏包列表，并重启手机：`adb shell su -c 'rm /data/system/users/0/package-restrictions.xml'`。没有这个文件，系统会认为所有系统包都已安装，所有被隐藏的应用都会回来。“默认打开方式”的选择也会一起被重置。这是基于我对 Android 如何读取这个文件的了解。我没有在你的手机上检查过。
5. 最后的办法：通过恢复模式重置数据。手机上的所有数据都会丢失，所以需要事先做好“开始之前”一节中的备份。重置之后所有内置应用都会回来，你可以从头再来。

**彻底删除“Google”用户（在电脑上）**

```
GU=$(adb shell pm list users | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
echo 正在删除用户编号: $GU
[ -n "$GU" ] && adb shell pm remove-user $GU
```

这是不可逆的：该用户的所有数据，包括账号登录信息，都会丢失。如果你还需要这个用户，请不要运行这条命令。只会删除名为“Google”的用户，其他用户不受影响。
