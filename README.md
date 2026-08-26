# TWRP for Apollo (Mi 10T / Redmi K30S Ultra) - Android 16 FBE 解密版

## 概述

本项目使用 GitHub Actions 云端编译 TWRP Recovery，目标是适配 LineageOS 23.2（Android 16）的 FBE v2 文件级加密。

### 技术方案

| 组件 | 来源 | 说明 |
|------|------|------|
| TWRP 源码 | `TWRP-Test/platform_manifest_twrp_aosp` `twrp-16.0` 分支 | 支持 Android 16 |
| 设备树 | `PitchBlackRecoveryDevices/android_device_xiaomi_apollo` | apollo 设备树 |
| 高通解密 | `TeamWin/android_device_qcom_twrp-common` | Keymaster 4.x 解密服务 |
| FBE 补丁 | 参考 MissMyTime 的 SM8850 框架 | 适配 SM8250/Keymaster |

### 设备信息

| 项目 | 值 |
|------|------|
| 设备 | Xiaomi Mi 10T / Mi 10T Pro / Redmi K30S Ultra |
| 代号 | apollo |
| SoC | Qualcomm SM8250 Snapdragon 865 |
| 加密方式 | FBE v2 (文件级加密) |
| 密钥服务 | Keymaster 4.1 (非 KeyMint) |
| 分区方案 | Virtual A/B 动态分区 |
| 目标 ROM | LineageOS 23.2 (Android 16) |

## 使用方法

### 步骤 1：Fork 项目

1. 在 GitHub 上创建一个新仓库（或 Fork 本项目）
2. 将 `.github/workflows/build-twrp.yml` 上传到仓库

### 步骤 2：运行编译

1. 进入仓库的 **Actions** 页面
2. 选择 **Build TWRP for Apollo** workflow
3. 点击 **Run workflow**
4. 等待编译完成（约 2-4 小时）

### 步骤 3：下载 recovery.img

1. 编译完成后，在 Actions 运行页面找到 **Artifacts**
2. 下载 `twrp-apollo-android16-recovery`
3. 解压得到 `recovery.img`

### 步骤 4：刷入 Recovery

```bash
# 方法 1：fastboot 临时引导（推荐先试）
adb reboot bootloader
fastboot boot recovery.img

# 方法 2：永久刷入到 recovery 分区
adb reboot bootloader
fastboot flash recovery recovery.img
fastboot reboot recovery
```

## 注意事项

### 关于解密兼容性

SM8250 使用 **Keymaster 4.1**，而 MissMyTime 的 Android 16 解密补丁针对的是 SM8850 的 **KeyMint**。两者的密钥服务实现不同：

- **Keymaster 4.x**：通过 `qseecomd` + `keymaster-4-1-qti` 服务解密
- **KeyMint**：通过 `TW_CRYPTO_USE_VENDOR_KEYMINT` 直接调用 vendor KeyMint HAL

本配置使用 TeamWin 官方的 `qcom_twrp-common` 来提供 Keymaster 4.x 解密支持，这是更成熟、兼容性更好的方案。

### 编译可能失败的情况

1. **设备树不兼容**：apollo 设备树是 Android 10 的，可能需要手动适配 Android 16
2. **缺少 vendor 二进制**：解密需要 vendor 分区的 `qseecomd`、`keymaster` 等二进制文件
3. **SELinux 策略**：Android 16 的 SELinux 策略更严格，可能需要额外补丁

### 如果编译失败

1. 查看 Actions 日志中的错误信息
2. 常见问题：
   - `lunch: unknown device` → 设备树路径不对
   - `build/make/core/... error` → 源码不兼容，需要补丁
   - `ninja: error` → 缺少依赖，检查 `ALLOW_MISSING_DEPENDENCIES`

3. 如果反复失败，考虑：
   - 降级到 `twrp-14.1` 分支编译（更稳定）
   - 在 XDA 上寻求帮助
   - 使用 LineageOS 自带 Recovery（天然支持 Android 16 加密）

### 备选方案

如果 TWRP 编译不成功，以下方式不需要解密也能刷机：

| 方法 | 命令 | 说明 |
|------|------|------|
| ADB Sideload | `adb sideload xxx.zip` | 不需要访问 /data |
| ADB Push | `adb push xxx.zip /tmp/` | 推送到 TWRP 临时目录 |
| 取消锁屏密码 | 系统设置中取消 | 重启 Recovery 后不需要解密 |

## 文件结构

```
twrp-apollo-build/
├── .github/
│   └── workflows/
│       └── build-twrp.yml    # GitHub Actions 编译配置
└── README.md                 # 本文件
```

## 参考项目

- [MissMyTime/twrp_device_sm8850](https://github.com/MissMyTime/twrp_device_sm8850) - Android 16 FBE 解密框架
- [PitchBlackRecoveryDevices/android_device_xiaomi_apollo](https://github.com/PitchBlackRecoveryDevices/android_device_xiaomi_apollo) - apollo 设备树
- [TeamWin/android_device_qcom_twrp-common](https://github.com/TeamWin/android_device_qcom_twrp-common) - 高通解密通用组件
- [TWRP-Test/platform_manifest_twrp_aosp](https://github.com/TWRP-Test/platform_manifest_twrp_aosp) - TWRP 源码 (twrp-16.0 分支)
