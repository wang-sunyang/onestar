# 签名与覆盖升级

本文说明拾光 OneStar 的签名约定、CI 如何拿到证书，以及当覆盖安装失败时怎么定位。
配套工具：`tools/verify_apk_signature.py`；相关代码：`build.sh` 第 9 节、`.github/workflows/android.yml`。

## 1. 为什么签名必须固定

Android 在安装时对「升级」有硬性条件，三条同时满足才允许覆盖：

| 条件 | 不满足时的报错 |
| --- | --- |
| 包名相同 | `INSTALL_FAILED_INVALID_APK` / 被当成另一个应用并存 |
| **签名证书相同** | `INSTALL_FAILED_UPDATE_INCOMPATIBLE` |
| `versionCode` 不小于已装版本 | `INSTALL_FAILED_VERSION_DOWNGRADE` |

证书是应用的长期身份。一旦发布出去的包换了证书，老用户只能卸载重装，而本应用的数据存在应用私有目录里，卸载即删除——等于用户数据全丢。

## 2. 曾经的坑：CI 每次现场生成证书

早期的工作流里没有 keystore，`build.sh` 在缺省分支中执行：

```bash
keytool -genkeypair -keystore "$WORK/debug.keystore" \
  -alias onestar-debug -keyalg RSA -keysize 2048 -validity 10000 \
  -dname "CN=OneStar Debug, O=OneStar, C=CN"
```

`CN` 是固定的，但**密钥对每次都是新的**，所以每次构建产物的证书指纹都不同。表现到用户侧就是：上一版能装，下一版覆盖安装直接失败。

更隐蔽的一点是：`keytool` 生成的证书只看主题名完全一样，`apksigner verify` 也只会说"签名有效"，不会提示"证书变了"。必须比对指纹才能发现。

## 3. 现在的约定

### 3.1 固定证书

| 项 | 值 |
| --- | --- |
| 包名 | `com.wangsunyang.onestar` |
| 证书主题 | `C=CN, O=Wang Sunyang, OU=Independent Developer, CN=Wang Sunyang` |
| 指纹（SHA-256） | `1E:8F:70:DF:5C:14:38:B6:50:DF:AC:74:0F:BD:D3:CC:AD:89:7E:E1:0F:A7:38:9B:3F:4F:52:98:AA:AA:BB:DC` |
| keystore 别名 | `wangsunyang`（PKCS#12，`keytool -list` 显示的 Alias name 同值） |
| 有效期 | 2026-11-07 → 2056-10-30（10000 天） |
| 签名方案 | v1（JAR 签名）+ v2（APK Signature Scheme v2） |

### 3.1.1 证书切换记录

| 版本区间 | 证书 | 状态 |
| --- | --- | --- |
| ≤ 0.12.2（含 rc1） | CI 现场生成的一次性 debug 证书，主题 `CN=OneStar Debug`，指纹 `B0:F1:E2:A1:…:4C:D1` | 已废弃，私钥随 runner 销毁，不可追回 |
| ≥ 1.0.0 | 开发者发布证书，主题 `CN=Wang Sunyang`，指纹 `1E:8F:70:DF:…:BB:DC` | 现行 |

切换点只有一次：从 0.12.2 升到 1.0.0 必须卸载重装（数据丢失）。1.0.0 之后只要 `ONESTAR_KS_B64` 不换，所有版本都能覆盖升级。

这张证书在本地、CI、Release 三处必须是同一张。指纹写死在 `tools/verify_apk_signature.py` 的 `EXPECTED_SHA256` 常量里。

### 3.2 证书怎么进 CI

证书本体不进仓库（公开仓库等于把私钥公开），改走 Actions Secret：

```bash
# 1. 把 keystore 转成单行 base64
base64 -w0 onestar-debug.keystore > ks.b64        # macOS 用 base64 -i ... | tr -d '\n'

# 2. 写入仓库 Secret（需要仓库 admin 权限）
gh secret set ONESTAR_KS_B64   --repo <owner>/onestar < ks.b64
gh secret set ONESTAR_KS_PASS  --repo <owner>/onestar --body '<keystore storePassword>'
gh secret set ONESTAR_KS_ALIAS --repo <owner>/onestar --body 'wangsunyang'

# 3. 清掉本地的中间文件
rm -f ks.b64
```

工作流中的 `Restore signing keystore` 步骤会把 `ONESTAR_KS_B64` 解码成 `ci-keys/signing.keystore`，并导出 `ONESTAR_KS` / `ONESTAR_KS_PASS` / `ONESTAR_KS_ALIAS` 三个环境变量；`build.sh` 检测到 `ONESTAR_KS` 后就不再走"一次性 keystore"分支。

三个 Secret 必须同时配置：`ONESTAR_KS_B64`（keystore 本体）、`ONESTAR_KS_PASS`（storePassword）、`ONESTAR_KS_ALIAS`（别名，当前为 `wangsunyang`）。缺 `ONESTAR_KS_ALIAS` 时还原步骤会直接失败，避免又退回到写死的错误别名。

也可以在本地直接用 `ONESTAR_KS_B64` 构建，无需落盘 keystore：

```bash
ONESTAR_KS_B64="$(cat ks.b64)" \
ONESTAR_KS_PASS="$(cat ~/.onestar_ks_pass)" ONESTAR_KS_ALIAS=wangsunyang \
ANDROID_HOME=$HOME/Android/Sdk ./build.sh
```

### 3.3 构建末尾自动验签

- 使用固定证书（`ONESTAR_KS` 或 `ONESTAR_KS_B64`）时：`build.sh` 调用 `tools/verify_apk_signature.py`，指纹不符直接失败
- 使用一次性 debug keystore 时：只打印指纹，不阻断（本地试验场景）

CI 里另有一个独立步骤 `Verify signing certificate`，Secret 已配置时强制执行。

> 踩坑：这一步原先写的是 `if: ${{ env.ONESTAR_KS != '' }}`。`GITHUB_ENV` 写入的变量只进后续 step 的 shell 环境，**不进 `if` 的 `env` 上下文**，所以条件恒为假，验签从未真正执行过——日志里那一步是灰的，容易被误读为"通过"。现改为在 shell 内判断 `${ONESTAR_KS:-}`，与 `build.sh` 的 `SIGNING_FIXED` 判定保持一致。

```bash
# 人工比对
apksigner verify --print-certs build/onestar-0.12.2-signed.apk

# 或直接比对仓库约定值
python3 tools/verify_apk_signature.py build/onestar-0.12.2-signed.apk
```

## 4. 排错

| 现象 | 原因 | 处理 |
| --- | --- | --- |
| `INSTALL_FAILED_UPDATE_INCOMPATIBLE` | 新旧包证书不同 | 分别 `--print-certs` 比对；证书已不同则只能卸载重装，或用同一 keystore 重签 |
| `INSTALL_FAILED_VERSION_DOWNGRADE` | `versionCode` 未递增 | 改大 `AndroidManifest.xml` 的 `versionCode` 后重建 |
| `INSTALL_FAILED_OLDER_SDK` | 设备系统低于 `minSdkVersion 21` | 无解，最低支持 Android 5.0 |
| 验签脚本报"没有 v1 签名块" | 只做了 v2 签名 | v2 只覆盖 APK 内容，需要 v1 才能从 `META-INF/` 取证书；用 `apksigner` 默认参数会同时打 v1+v2 |
| 签名后安装报 `INSTALL_PARSE_FAILED_NO_CERTIFICATES` | APK 被重新打包破坏了签名块 | 任何对齐/重打包必须在签名**之前**做，本仓库的 `tools/zip16k.py` 就在签名前执行 |
| CI 日志出现"未配置 ONESTAR_KS_B64"警告 | Secret 缺失或名字写错 | 按 3.2 配置，注意大小写 |

## 5. 安全边界

这张是 **debug 证书**，主题名里也写明了。即使私钥只存在于 Secret 中，它也不具备任何防伪能力——任何拿到过该 keystore 的人都能签发"看起来是同一次升级"的包。

因此：

- 只适合自用、小范围体验、验证构建链
- **不能上架应用商店**，也不适合对外分发
- 正式分发必须换成自己的 release keystore（`ONESTAR_KS` / `ONESTAR_KS_PASS`），届时要接受一个现实：换证书意味着老用户卸载重装
- 一旦决定换成 release 证书，就要同步更新本文与 `EXPECTED_SHA256`，并把 `MAJOR` 版本号递增（签名变更属于破坏性变更，见 `docs/release-spec.md`）
