# CarWith 安装与回退

这套方案在已 Root 的 **POCO F8 Pro 国际版（`annibale`）** 上运行。原版 CarWith 提供车联应用与系统服务；EEA Fix v2 负责国际版接口适配；APGC 0.1.2 是独立的候选补丁，用于让 CarWith 收到 Wi-Fi 底层实际返回的 AP/GC 模式切换结果。可以先装基础车联与 v2，确认正常后再启用候选。

**已验证：** 小米侧成功连接作为接收端的 OPPO Ace2／MiAuto，完成 Wi-Fi P2P 组网、认证并进入投屏，底层也确认关闭了临时模式。**2023 款小鹏 G6 的 CarLink 尚未复测。** 另一台手机上运行的接收端不能代表真车的组网、认证或投屏行为。

## 1. 安装前检查

| 项目 | APGC 0.1.2 实测环境 |
| --- | --- |
| 手机与系统 | POCO F8 Pro（`annibale`），`OS3.0.302.0.WPKMIXM`，Android 16 / API 36 |
| 小米车联 | `com.miui.carlink`，`4.0.3-20260513`，`versionCode 104000003` |
| Root 与挂载 | [SukiSU Ultra](https://github.com/SukiSU-Ultra/SukiSU-Ultra) `4.1.2-2-gf39c001e`、已启用 Zygisk、[Hybrid Mount](https://github.com/Hybrid-Mount/meta-hybrid_mount) `6.2.6-rc.1` |
| LSPosed | [LSPosed 官方入口](https://lsposed.org/)；实测版本 `2.2.1 (7912)` / API 102 |

手机应已经配置好上述 Root 环境，并能正常重启、打开 LSPosed 管理器。安装前记下已启用的 Root 和 LSPosed 模块及其作用域，备份你已有的挂载配置。本文从已 Root 的设备开始，不包含解锁、刷机或框架升级步骤。已有正常工作的车联基础配置可以跳过对应重装步骤。

APGC 还会核对下列两份系统库的完整 SHA-256。其余机型、固件、车联版本以及更新后的系统文件均需单独验证。遇到不匹配或无法读取，先停在基础车联测试；不要改机型属性、绕过校验或覆盖系统库。

| 系统库 | SHA-256 |
| --- | --- |
| `/apex/com.android.wifi/javalib/service-wifi.jar` | `b3bcfe451e72621ff551928ac18192c4054dcba2dea2988b33afa0384383d524` |
| `/system_ext/framework/miui-wifi-service.jar` | `ddd6d1e1b652c1fa8e5db9893e30b03d43d294bce72e2648b917dafcb0fd4f75` |

## 2. 下载与校验

完整包解压后，`packages/` 里有两个实测 APK。原版车联 ZIP 从上游获取：在解压目录中运行 `python3 scripts/fetch_upstream.py`，脚本会下载并校验固定版本，把 ZIP 放入 `packages/`。也可以在浏览器打开[上游 26.11.05 固定发布页](https://github.com/DuolaD/CarWith4XiaomiEEA/releases/tag/26.11.05)，在 Assets 选择 `CarWith4XiaomiEEA-26.11.05.zip`，保存到 `packages/` 后核对下表 SHA-256。外层整包与原版 Root 模块 ZIP 用途不同：把原版 ZIP 原样交给 Root 管理器；两个 APK 交给 Android 安装器。

| 文件 | SHA-256 |
| --- | --- |
| `CarWith4XiaomiEEA-26.11.05.zip` | `03010d2aa1bf66c0c2dad88ad0b768a1e856f880b265a94ed06f2ca1ccfb8c30` |
| `CarWith-EEA-Fix-2.0-force-success.apk` | `d76e40c0ef087239ebc642ad1b0dffb7b22fcd54c883988cf944ba841370c74e` |
| `CarWith-APGC-0.1.2-candidate.apk` | `b1534e43b1167240ddb88761511927628a279e9fb6d3f494e7b13309d3414f52` |

在解压目录下可用 `shasum -a 256 packages/*`（macOS）或 `sha256sum packages/*`（Linux）对照。Windows PowerShell 可对单个文件运行 `Get-FileHash -Algorithm SHA256 <文件路径>`。校验不一致时停止安装。

## 3. 安装基础车联

1. 把原版 ZIP 复制到手机，打开 SukiSU 管理器 → 模块 → 从本地安装，选择 `CarWith4XiaomiEEA-26.11.05.zip`。确认模块 ID 为 `carwith_eea_port`，安装结束后重启并解锁手机。
2. 在手机设置搜索 **CarWith** 并打开车联页。电脑已授权 ADB 时，可运行下列检查。正常情况下开机完成值为 `1`，服务显示 `MiuiCarService: found`，应用版本与上表一致。

   ```sh
   adb shell getprop sys.boot_completed
   adb shell service check MiuiCarService
   adb shell dumpsys package com.miui.carlink | grep -E 'version(Code|Name)'
   ```

   如果设置里找不到入口，但服务和版本正确，可在解锁后打开设置页：

   ```sh
   adb shell am start -n com.miui.carlink/com.carwith.launcher.settings.phone.UCarScreenSettingsActivity
   ```

3. **仅当 `MiuiCarService` 未找到，且 Hybrid Mount 状态显示这个模块的 OverlayFS 挂载失败**，才调整挂载规则。先备份 `/data/adb/hybrid-mount/config.toml`，再用有 Root 权限的编辑器只合并以下规则；已存在同名表时只修改该表内的 `default_mode`，不要重复声明表或覆盖其他模块配置：

   ```toml
   [rules.carwith_eea_port]
   default_mode = "magic"
   ```

   重启后确认模块的两个系统文件已挂载，`MiuiCarService` 返回 `found`，再继续。应用已经出现于设置里时，系统服务仍可能缺失：上游模块也会把车联 APK 安装到 `/data/app`，所以要分别检查界面和服务。若服务仍找不到，先排查基础挂载。

## 4. 安装与启用两个 LSPosed APK

可以先在手机文件管理器点开 APK，按 Android 的授权提示安装。电脑通过 [Android Platform-Tools](https://developer.android.com/tools/releases/platform-tools) 安装时，建议使用完整安装，避免本次实测中分块安装后 LSPosed 无法读取 APK 路径的问题。在解压目录运行：

```sh
adb install --no-incremental -r packages/CarWith-EEA-Fix-2.0-force-success.apk
adb install --no-incremental -r packages/CarWith-APGC-0.1.2-candidate.apk
```

如果同时连接多台设备，先运行 `adb devices -l`，再在 `adb` 后加 `-s <目标设备序列号>`，确认目标是 POCO。若手机拒绝 USB 安装，使用本机安装器按提示操作。`-r` 允许更新已装的同签名应用；遇到签名不一致，先核对已装 APK 和来源，避免为覆盖安装而删掉已有应用或数据。

打开 LSPosed 管理器 → 模块，按下表选择作用域。列表中名称可能因语言和版本而不同，以括号里的标识为准。

| 模块 | 勾选项 |
| --- | --- |
| CarWith EEA Fix 2.0（`com.xue.carwitheea`） | 仅勾选 **小米车联／CarWith**（`com.miui.carlink`） |
| CarWith APGC 候选补丁（`com.xue.carwithapgc`） | 仅勾选 **系统框架**（通常显示 `android`，部分界面标为 `system`）和 **小米车联／CarWith**（`com.miui.carlink`） |

建议先启用 v2、重启并核对 CarWith 界面与 Wi-Fi 正常，然后在符合第 1 节条件的设备上启用 APGC 并再重启。APGC 只覆盖模式切换路径；v2 继续处理其他接口。APGC 需要系统框架作用域，重启才能让系统进程重新加载作用域。

## 5. 验证连接

1. 重启解锁后，检查手机 Wi-Fi 可正常使用、`MiuiCarService` 存在、CarWith 设置页可打开。LSPosed 日志可查找 `global_wifi_profile_verified`、`service_hook_installed`、`hal_hook_installed` 和 `carwith_replacement_installed`。`optimization_reader_absent_in_profile` 在已验证的固件组合上允许出现。
2. 若有第二台 Android 设备，可在接收端安装 [MiAuto 0.1.3](https://github.com/PiggyBackBayonet/MiAuto/releases/tag/v0.1.3)，打开接收端并用小米连接。验收分开看：底层是否记录 `enable_acknowledged_lease_60s`／`disable_acknowledged`、是否完成 P2P 组网、是否通过车联认证（`ConnectedState`）以及接收端是否进入投屏。模式命令成功只代表底层确认；仍要观察投屏。测完退出接收服务，避免它参与真车测试。
3. 到 G6 旁停车，打开车机 **CarLink**，用小米连接一次，记录是否认证并成功投屏。若界面仍显示“连接失败”，根据该次日志区分模式命令、P2P 组网、认证或投屏阶段；若要比较 APGC 开关差异，保持同一车机入口和同一连接方式，关闭 APGC 后重启再测。分享日志时只保留所需事件，删去连接口令、认证材料、MAC、设备标识和 SSID。

| 现象 | 处理 |
| --- | --- |
| `wifi_profile_mismatch_no_commands` 或 `wifi_profile_read_failed_no_commands` | 系统库校验未通过，候选不会发送模式命令。关闭候选，核对系统版本和库文件。 |
| `class_observation_expired_incomplete` 或 `loader_target_missing_*` | 必需的系统加载条件未满足，先核对框架和作用域，不能据此认定底层驱动拒绝。 |
| `disable_not_acknowledged_state_retained` 或连接后 Wi-Fi 明显异常 | 停止继续尝试，关闭 APGC 并重启，重新检查 Wi-Fi。 |

## 6. 回退

**只撤回候选：** LSPosed → 模块 → 关闭 **CarWith APGC 候选补丁**，重启。保留 v2、原版 CarWith 与 Hybrid Mount；重启后系统进程中的候选挂接和临时状态会清除。回退后检查 Wi-Fi、CarWith 页面和已有连接。需要时可再卸载候选 APK，无须清除车联数据。

**撤回整套适配：** 先在 LSPosed 中停用 APGC 和 EEA Fix v2，再到 SukiSU 管理器移除 `carwith_eea_port`，重启。原版卸载脚本会在下次开机时尝试清理它装入 `/data/app` 的车联副本，之后应核对实际状态；已有车联系统应用的设备，先核对包的来源，避免误删。若曾添加 `magic` 规则，只撤回该条改动；确认备份以来没有其他配置变化时，也可用原备份恢复。

**进不了桌面：** 若电脑仍有已授权的 ADB 和 Root，且确认 LSPosed 根模块 ID 是 `zygisk_lsposed`，可临时停用该根模块后重启。此操作会暂停**所有** LSPosed 模块。恢复桌面后先卸载候选 APK，再按需恢复框架与原有模块。请在确认已选中故障手机后执行；只在停用命令成功时重启。

```sh
adb shell "su -c 'test -f /data/adb/modules/zygisk_lsposed/module.prop && touch /data/adb/modules/zygisk_lsposed/disable'" && adb reboot
```

如果 ADB 也不可用，所列 SukiSU 版本的备用入口是：重新开机，从第一屏开始尽早快速**按下并松开音量下键至少三次**，使模块装载前记录到三次独立按下。源码显示安全模式会停用全部 Root 模块；恢复桌面后只重新启用明确需要的模块。这是该版本[按键判定](https://github.com/SukiSU-Ultra/SukiSU-Ultra/blob/f39c001efdcc0a7c84d1212f32ded55b1a207f35/kernel/ksud.c#L429-L473)与[禁用模块逻辑](https://github.com/SukiSU-Ultra/SukiSU-Ultra/blob/f39c001efdcc0a7c84d1212f32ded55b1a207f35/userspace/ksud/src/init_event.rs#L33-L56)所示的入口，本机未实测，其他内核或故障状态下也不保证生效。
