# 组件来源

- **CarWith4XiaomiEEA 26.11.05**：[上游发布](https://github.com/DuolaD/CarWith4XiaomiEEA/releases/tag/26.11.05)，发布文件 `CarWith4XiaomiEEA-26.11.05.zip`，SHA-256 `03010d2aa1bf66c0c2dad88ad0b768a1e856f880b265a94ed06f2ca1ccfb8c30`。上游项目有 [MIT 许可证](https://github.com/DuolaD/CarWith4XiaomiEEA/blob/main/LICENSE)，发布 ZIP 另含厂商 APK/JAR。安装包通过下载脚本获取原样上游资产，本仓库不镜像这些厂商文件。
- **MiAuto 0.1.3**：可选的安卓接收端，[上游发布](https://github.com/PiggyBackBayonet/MiAuto/releases/tag/v0.1.3)，[AGPL-3.0](https://github.com/PiggyBackBayonet/MiAuto/blob/main/LICENSE)。`MiAuto-0.1.3.apk` 的 SHA-256 为 `b5b9acb0457772a8369be8a906b21784b323bb2ceccf3e7873f0f99ba641c358`。需要时使用 `python3 scripts/fetch_upstream.py --miauto`。
- **本项目的两个补丁**：`packages/` 中的 APK 是原样保存的实测安装文件，见 [SHA256SUMS](../packages/SHA256SUMS)。EEA Fix 2.0 提供既有的系统接口适配；现有资料只保留此 APK，没有对应原始源码。AP/GC 0.1.2 的源码及测试在 [module/](../module/)，开发构建使用独立签名。
- **历史版本**：Global v0.2.0 使用的是较旧的 CarWith 3.7.1 / Global .301；EEA v0.1.0 是 EEA `WPKEUXM` 专用候选。历史包不用于本教程。
