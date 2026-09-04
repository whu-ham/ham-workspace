# Android：从 GitHub Release 检查版本更新

## 背景

Android 当前由 `VersionManager` 调用 `GetNewVersion` gRPC 接口获取版本信息，再由关于页和启动生命周期触发检查。现希望改为直接读取 GitHub Release，使版本发布与 GitHub Release 附件保持一致，并允许用户在设置中选择 GitHub 访问源。

## 目标

1. Android 版本检查不再依赖现有版本接口，改为读取指定 GitHub 仓库的 Release 数据。
2. `beta` 构建渠道读取 GitHub Pre-release；其他渠道读取正式 Release。
3. 设置页提供 GitHub 源选择，至少支持官方 GitHub 与 `gh-proxy.org` 这类代理源。
4. 保留现有版本检查入口、更新提示、强制更新判断和更新日志展示能力。

## 已确认的现状

- 版本检查入口：`android/business/my/.../setting/about/VersionManager.kt`。
- 更新弹窗：`android/business/my/.../ui/version/VersionUpdateSheet.kt`。
- 关于页手动检查：`android/business/my/.../setting/about/AboutView.kt`。
- 启动检查由 `CheckVersionActivityLifecycleHandler` 间接触发。
- `App.VERSION_TYPE` 来自 `versionType` 构建参数；现有发布工作流将其设置为 `beta` 或 `release`，应以此判断 Release 类型。
- 现有发布工作流会创建 GitHub Release/Pre-release，并上传 Github flavor APK；客户端读取的 owner/repo 应通过云控配置，默认值为 `whu-ham/whu-ham.github.io`，不能硬编码为其他仓库。
- 本地设置统一使用 `LocalStorage.Key` + `LSKVContext` 持久化。

## 实现范围

### 1. GitHub Release 数据源

- 新增 Android 侧 GitHub Release client/repository，使用现有网络栈或项目统一的 HTTP client。
- owner/repo 通过现有 `CCKVContext` 云控读取，新增专用 `CCKVKey`；云控缺失、为空或格式非法时回退到 `whu-ham/whu-ham.github.io`。格式限定为 `owner/repo`，并对 owner、repo 做非空及安全字符校验。
- GitHub 源按“源头 + GitHub 原始链接”规则构造，例如：`https://gh-proxy.org/` + `https://github.com/<owner>/<repo>/releases/latest`；实现时统一处理源头末尾斜杠，避免重复或缺失 `/`。
- 内置源至少包括：
  - GitHub 官方源：直接使用 `https://github.com/` 链接
  - `https://gh-proxy.org/`
  - `https://v4.gh-proxy.org/`
  - `https://v6.gh-proxy.org/`
  - `https://cdn.gh-proxy.org/`
  - `https://axisnow.gh-proxy.org/`
- 代理源的请求形式为 `{源头}{github链接}`，其中 `{github链接}` 保留完整的 `https://github.com/...`；示例：`https://gh-proxy.org/https://github.com/WJQSERVER-STUDIO/ghproxy/releases/download/4.3.4/ghproxy-linux-amd64.tar.gz`。
- Release API 请求和 APK asset 下载链接都必须遵守同一拼接规则；不得只替换域名后丢失原始 GitHub 路径。
- 将 Release JSON 映射为现有更新 UI 所需的领域模型，至少包含：tag/name、是否 prerelease、发布时间、正文、版本名称/版本号、APK 下载地址、是否强制更新所需的最低版本信息（如 GitHub Release 不提供，则定义替代规则）。
- 版本号格式严格限定为 `x.y.z`，其中 `x`、`y`、`z` 均为非负十进制整数；禁止四段版本、缺段版本、前缀字母或其他非标准格式进入版本比较。
- Release tag 应使用 `vX.Y.Z`；beta 可使用 `vX.Y.Z-betaN`，其中 `N` 为正整数且仅用于标识预发布序号，比较版本主体仍为 `X.Y.Z`。
- 无法从 tag 解析出严格 `x.y.z` 版本时返回可诊断错误，不得静默把未知版本当成可更新版本。
- 根据 `App.VERSION_TYPE == "beta"` 选择最新 Pre-release；其他值选择最新正式 Release。正式渠道不得误选 Pre-release。
- 每个公开 GitHub Release/Pre-release 必须上传一个固定名称的客户端安装包：`Ham-Github.apk`。该文件必须由 `Github` flavor 构建生成（当前 flavor 仅面向 `arm64-v8a`），并在 CI 创建 Release 前重命名为该固定名称；版本号不放入文件名，版本信息以 Release tag 为准。
- 客户端只选择 Release assets 中名称精确等于 `Ham-Github.apk` 的 asset，不按模糊关键词、上传顺序或任意第一个 APK 选择；找不到、重复或 MIME/扩展名不符时视为资产错误。
- `OSS`、`PlayStore` flavor APK 不参与 GitHub 源更新下载。找不到 `Ham-Github.apk` 时保留浏览器打开 Release 页面或给出明确错误。
- 将用户选择的源应用到 API 请求与 APK/Release 页面链接，并集中处理 URL 拼接、斜杠、编码和代理路径规则。
- 默认源应保持官方 GitHub 可用；代理源不可用时展示失败状态，不应阻塞应用启动。

### 2. 设置页

- 在 `SettingView` 增加“GitHub 源/更新源”入口，显示当前选择。
- 新增独立设置页或底部选择弹窗，至少提供：
  - GitHub 官方源
  - `gh-proxy.org` 代理源
- 同时提供 `v4.gh-proxy.org`、`v6.gh-proxy.org`、`cdn.gh-proxy.org`、`axisnow.gh-proxy.org`。
- 选择结果持久化到 `LocalStorage`，应用重启后保持。
- 预留扩展自定义源的模型/校验边界，但本次不要求开放任意 URL 输入，除非实现者确认安全校验方案。
- 切换源后，下次手动检查立即使用新源；必要时清理旧请求缓存。
- 补齐中/英/日文案，并保持现有 Compose 设置页风格。

### 3. 更新 UI 与错误处理

- 尽量复用 `VersionInfoSheet`、`VersionUpdateSheet` 和关于页现有状态流，避免把 GitHub API 细节泄漏到 UI。
- 更新按钮应打开所选源下的 APK asset URL；无法确定 asset 时打开 Release 页面。
- 网络错误、HTTP 非 2xx、JSON/版本解析失败、无匹配 Release/asset 都要进入可重试的失败状态，并提供用户可理解的提示。
- 启动后台检查失败只记录日志，不弹出误导性的“已是最新版本”。
- 保留当前版本变更日志展示逻辑；GitHub Release body 为空时提供合理空态。

### 4. 测试与文档

- 为源 URL 构造、beta/release 筛选、tag/版本解析、asset 选择和异常响应补充单元测试。
- 为设置持久化与切换行为补充测试；如现有模块缺少 UI 测试，至少覆盖 ViewModel/存储层。
- 更新 Android 文档中的版本检查流程、数据源配置和发布资产命名约定。
- 记录官方 GitHub 与 `gh-proxy.org` 的实际 API/下载 URL 规则，避免仅凭域名前缀拼接。

### 5. Android 发布工作流

- 修改 `ham-android/.github/workflows/release.yml` 的 `finalize` job：在下载 `release-apks` artifact 后，单独收集 `Github` flavor 的 release APK。
- 发布前必须校验恰好找到一个 Github flavor APK；找不到或找到多个时使工作流失败，禁止创建包含错误或不明确资产的公开 Release。
- 将该 APK 复制/重命名为 `Ham-Github.apk`（放入独立的发布目录，避免覆盖原始 artifact）。
- 创建公开仓库的 draft Release 时，只上传该发布目录中的 `Ham-Github.apk`；不得继续以 `apks/**` 通配上传 `OSS`、`PlayStore` 或其他 APK。
- 同一固定资产名规则同时适用于 `release` 和 `beta`：beta 创建为 Pre-release，release 创建为正式 Release。
- Release tag 继续与客户端解析规则一致：正式版 `vX.Y.Z`，beta `vX.Y.Z-betaN`；工作流在创建 Release 前验证其符合格式。
- 在 workflow 日志中输出最终公开 Release 仓库、tag、源 APK 路径和目标资产名，但不得输出任何 secret。

## 非目标

- 不修改后端 `ham-proto` 或 `ham-backend-go` 的版本接口实现。
- 不调整 Play Store、OSS 或内部 Release 的产物策略；本任务只调整 Android workflow 创建公开 GitHub Release 时的资产准备、校验和上传。
- 不在本任务中实现任意第三方镜像源的自动发现或健康探测系统。

## 验收标准

- `release` 构建检查时只命中最新正式 GitHub Release；`beta` 构建检查时只命中最新 Pre-release。
- 当前版本低于 Release 版本时，关于页和启动检查均能显示更新提示；当前版本不低于目标版本时显示最新状态/既有更新日志。
- 更新操作能从所选源打开正确 APK；无匹配 APK 时行为明确且可用。
- 正式 Release 与 Pre-release 均包含且只包含一个可供客户端更新的 `Ham-Github.apk`；客户端不会误下载 `OSS` 或 `PlayStore` APK。
- Android `release.yml` 在 Github flavor APK 缺失或不唯一时失败；成功时公开 Release 的 assets 中仅有 `Ham-Github.apk`。
- 在设置中切换官方源与 `gh-proxy.org` 后，重新检查请求和下载链接均使用新源，重启应用后选择仍保留。
- 断网、代理不可达、GitHub 返回异常、Release 数据不完整时不会导致崩溃或误报更新成功。
- 相关单元测试通过，Android 受影响模块可编译；验证结果需按 `ham-android` 子模块单独记录。

## 待实现前确认

1. 云控 owner/repo 的具体 key 命名及下发位置；默认值已确定为 `whu-ham/whu-ham.github.io`。客户端不直接读取 CI 的 `RELEASE_REPO` secret。
2. GitHub 源选择统一采用 `{源头}{github链接}` 拼接方式，还是某些代理需要专用 API 路径；需在实现中用实际请求验证上述五个代理源对 GitHub Release API 和 asset 下载的支持情况。
3. Release tag 的标准格式已限定为 `vX.Y.Z`，beta 为 `vX.Y.Z-betaN`；APK asset 名称确定为 `Ham-Github.apk`，需要在发布工作流中落实重命名。
4. GitHub Release 是否需要承载“最低支持版本”字段；若没有，需确定是否取消客户端现有强制更新判断，或约定 Release body/asset 元数据格式。
