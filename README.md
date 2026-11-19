# nim-uts-sdk-demo

网易云信 IM `nim-uts-sdk`（UTS 插件）的 uni-app x 演示工程。一套代码可编译到 Android / iOS / 鸿蒙三端。

> 当前版本：1.0.1。三端统一使用 `initNIMSDKJson(options): Promise<...>` 初始化；登录支持 `login` 与
> `loginWithProviders`（iOS 的动态 token provider 受桥接限制，需传 `null` 使用静态 token）。

## 功能

- 初始化 `nim-uts-sdk`（`initNIMSDKJson`，Android / iOS / 鸿蒙统一 Promise 形态）
- 登录 / 动态 token 登录（`login` / `loginWithProviders`，iOS 走静态 token 路径）
- 登录 / 登出 / 登录状态监听
- 用户、好友、本地会话、消息、群 5 个常用模块接口测试
- 页面底部展示接口调用和查询返回结果
- 页面顶部监听状态区域展示监听回调内容

## 开发者首次配置

拉到代码后，需要自行配置以下内容才能跑通：

### 0. 安装 nim-uts-sdk 插件（必装）

本仓库已将 `uni_modules/nim-uts-sdk` 加入 `.gitignore`，克隆后工程里**没有这个插件**。

蒸汽模式适配依赖本地插件里的 `initNIMSDKJson` 入口（Android / iOS / 鸿蒙三端统一，返回 Promise），
以及消息 / 好友等公共方法内部的 `any -> 原生类型` 归一化逻辑（`createTextMessage`、`sendMessage`、
`getMessageList`、`addFriend` 等）。**市场版插件没有这些适配**，直接导入会在真机上出现跨桥类型错误。

本工程通过软链接或目录拷贝使用本地 SDK 源码（二选一）：

```sh
ln -s /path/to/nim-uts-sdk/uni_modules/nim-uts-sdk uni_modules/nim-uts-sdk
# cp -R /path/to/nim-uts-sdk/uni_modules/nim-uts-sdk uni_modules/nim-uts-sdk
```

如果换机器或重新克隆，请重新指向包含蒸汽模式适配的本地插件版本。

### 1. 云信账号信息（必填）

打开 `pages/index/index.uvue`，修改 `DEMO_LOGIN_CONFIG`。其他使用者只需要改这里的 `appKey / accountId / token`：

```uts
const DEMO_LOGIN_CONFIG : DemoLoginConfig = {
	appKey: 'your yunxin appKey',
	accountId: 'login account id',
	token: 'login token'
}
```

| 字段 | 说明 |
| --- | --- |
| `appKey` | 在[云信控制台](https://app.netease.im/)创建应用后获取 |
| `accountId` | 测试账号的 accid |
| `token` | 对应 accid 的 token |

首页已拆成“初始化”和“登录”两块：先点击“初始化”，成功后再点击“登录”。页面输入框仍可临时修改配置。

> iOS uni-app-x 当前同步 Storage 桥接存在返回 `undefined` 抛错问题，Demo 在 iOS 下会跳过本地缓存写入。Android / Harmony 仍会通过 `uni.setStorageSync` 保存页面输入值。

### 2. 模块测试说明

登录成功后，可以从首页进入以下模块：

| 模块 | 说明 |
| --- | --- |
| 用户 | 获取用户列表、云端获取用户、搜索用户、更新自己资料、用户监听 |
| 好友 | 获取好友列表、按账号获取好友、添加好友、删除好友、好友监听 |
| 本地会话 | 获取会话列表、创建会话、删除会话、置顶会话、本地会话监听 |
| 消息 | 查询消息、创建文本消息、发送消息、消息监听 |
| 群 | 创建群、获取已加入群列表、获取群资料、获取群成员列表、群监听 |

列表类接口拉取成功后，会自动把第一条数据回填到后续操作使用的输入框。例如本地会话页获取会话列表后，会把第一条 `conversationId` 填入“会话 ID”，后续可以直接点击置顶或删除。

### 3. 鸿蒙运行配置（鸿蒙端必填）

`manifest.json` 的 `app-harmony.distribute.signingConfigs` 字段值已留空，需要本机生成：

1. 在 HBuilderX 顶部菜单登录 DCloud 账号。
2. 打开 `manifest.json` → `app-harmony` 配置页，点击"重新生成"调试证书（或直接"运行 → 运行到鸿蒙"，HBuilderX 会自动生成并回填 `certpath / profile / storeFile / keyPassword / storePassword`）。
3. `compatibleSdkVersion` 保持 `"5.0.5(17)"`：云信鸿蒙 SDK `@nimsdk/nim` 要求 `minCompatibleSdkVersion = 17`，低于 17 会报 `00306004 Specification Limit Violation`。
4. `bundleName` 默认 `uni.app.UNI1997414`，如需改成自己的包名，同时要在 AGC 控制台匹配对应证书。

### 4. Android / iOS

- Android：`manifest.json` 的 `app-android` 段用默认配置即可，无需额外证书（debug keystore 由 HBuilderX 自动管理）。
- iOS：首次运行需在 HBuilderX 里配置开发者证书 / profile。

## 运行

在 HBuilderX 中打开本工程：

- 运行 → 运行到 Android / iOS / 鸿蒙

首次运行鸿蒙前，建议清掉旧产物避免 profile 合并冲突：

```sh
rm -rf unpackage/dist/dev/app-harmony
```

### 蒸汽模式（vapor）注意事项

本工程 `manifest.json` 已开启 `uni-app-x.vapor`，并指向本地插件源码（`uni_modules/nim-uts-sdk`）。
蒸汽模式下页面逻辑层是 JS，UTS 插件与页面之间要跨桥，因此页面侧需要遵循下面的约定；本工程已全部按此适配。

#### 1. 页面不能 import 插件导出的枚举对象

`export enum V2NIMXxx {...}` 这类对象在**生产打包 / 自定义基座**下不会进入 uts 代理导出表：

- 运行（dev）模式正常：插件走 uts2js 直接编译，枚举可用；
- 云打包 / 自定义基座会报 `"V2NIMLoginStatus" is not exported by ...uni_modules/nim-uts-sdk?uts-proxy`。

页面侧因此不再 import 任何枚举对象，枚举入参一律传**数字**（数值取自 `utssdk/types/Enum.uts`）：

```uts
const LOGIN_STATUS_LOGINED : number = 1
if (status == LOGIN_STATUS_LOGINED) { /* ... */ }
```

个别接口（如群、好友）的枚举参数已在插件内部做数字归一化，页面直接传数字 / 对象字面量即可，见第 5 节。

#### 2. 初始化：使用 `initNIMSDKJson`

**嵌套在对象字面量里的类实例过桥后会退化成 `UTSJSONObject`**，所以不能把 `SDKOptions` 实例塞进
`initNIMSDK({ options: ... })`：

```uts
// ❌ 运行时可抛：java.lang.ClassCastException: UTSJSONObject cannot be cast to SDKOptions
initNIMSDK({ options: new AndroidSDKOptions(), success: ..., fail: ... })
```

Android / iOS / 鸿蒙三端统一改用 `initNIMSDKJson(options): Promise<NIMResult<UTSJSONObject>>`：
配置走 JSON，结果走 Promise（成功 `code = 200`、`data = null`，失败也 resolve），页面不再传回调参数：

```uts
initNIMSDKJson({ appKey: appkey.value } as UTSJSONObject).then((res) => {
	if (res.code == 200) {
		// 初始化成功
	} else {
		// res.code / res.desc 为失败原因
	}
})
```

字段按平台对齐：Android 用 `AndroidSDKOptions` 字段、鸿蒙用 `HarmonySDKOptions` 字段、
iOS 用当前 `IOSSDKOptions` 已支持的字段（`appKey`、`apnsCername`、`pkCername`、`v2`、`useV1Login`、`enableV2*`）。
iOS 对 `serverConfig` / `mixPushConfig` / `authProvider` 等对象回调类字段以及 `avoidNosAccelerationBuckets`
会明确返回失败，不会静默丢弃。

#### 3. 登录：`login` + JSON option

SDK 已撤回 `loginJson`：Android 的 `login` 实现本身接收 `UTSJSONObject` 并解析数字枚举，
其它端继续接收 typed option。页面统一传对象字面量即可：

```uts
loginService.login(accountId, token, { syncLevel: 0 })
```

动态 token / 第三方回调鉴权用 `loginWithProviders`，provider 必须作为**顶层函数参数**传入：

| 平台 | provider 回调 | 说明 |
| --- | --- | --- |
| Android / 鸿蒙 | 支持 | 传 `tokenProvider` / `loginExtensionProvider`，原生在需要时同步取 token |
| iOS | 不支持 | iOS 桥接层没有回调返回值通道，传非 null provider 会明确返回 `code = 2003`；请传 `null` 走静态 token 登录 |

```uts
// iOS：provider 传 null（静态 token）
loginService.loginWithProviders(accountId, token, { authType: 1, syncLevel: 0 } as UTSJSONObject, null, null)

// Android / 鸿蒙：provider 作为顶层回调
loginService.loginWithProviders(accountId, token, { authType: 1, syncLevel: 0 } as UTSJSONObject, (id: string): string => token, null)
```

#### 4. 消息：公共方法内部做 JSON 归一化

当前 SDK 不再提供 `*Json` 系列，而是把 `sendMessage` / `replyMessage` / `getMessageList` 等公共方法的
入参改为 `any`，插件内部用 `messageFromBridge` / `sendParamsFromBridge` 等函数判断入参是
`UTSJSONObject` 还是原生对象，再逐字段还原：

```uts
const message = new V2NIMMessageCreate().createTextMessage(text)
messageService.sendMessage(message, conversationId, null, null)
messageService.getMessageList({ conversationId, limit: 20 })
```

页面不需要再手动转 `UTSJSONObject`，也不需要区分 Android / 其它端。

#### 5. 群 / 好友：公共方法接收 `any` 并做字段级归一化

群接口当前也已改成 `any`：`teamType`、群成员查询 option、`createTeam` 参数等都可以直接传数字 / 对象字面量，
插件内部会归一化成原生枚举与强类型参数：

```uts
const service = new V2NIMTeamService()
service.createTeam({ teamType: 1, name: 'demo', memberLimit: 100 }, null, null, null)
service.getTeamInfo(teamId, 1)
service.getTeamMemberList(teamId, 1, { limit: 20 })
```

好友接口同理：`addFriend` 的 `params` 已改为 `any`，插件内部会把数字 `addMode` 归一化成原生枚举：

```uts
friendService.addFriend(accountId, { addMode: 2, postscript: 'hi' })
```

#### 6. 其它

- 当前 SDK 的跨桥策略是：`initNIMSDKJson` 单独处理初始化，Message / Friend / Team 等公共方法接收 `any`
  并在插件内做字段级归一化；`loginJson`、`addFriendJson`、消息 `*Json`、群 builder 以及 Setting / Storage 等
  `*Json` 镜像都已移除或不再需要。
- 三端初始化已统一为 `initNIMSDKJson`（Promise），页面不再按平台拼 `SDKOptions`；
  当前平台差异只剩「iOS 不支持 provider 回调」这一处，已在第 3 节说明。
- 若真机遇到枚举入参抛 `ClassCastException`，说明对应接口还没有在插件内做字段级归一化，
  需要在插件侧补 `*FromBridge` / `*FromJson` 转换，而不是在页面侧硬转类型。
- 蒸汽模式下页面自身即可滚动，根节点 `scroll-view` 的 `scroll-y` 属性会告警，可按需改回 `view`。
- 本地运行到 Android App 基座时，HBuilderX 会重新编译页面与 UTS 插件；发行 / 独立安装包需要重新出包，
  仅更新 wgt 不会更新插件原生代码。

## 目录结构

```
.
├── pages/index/index.uvue        # 首页：默认配置 + 初始化 + 登录 + 模块入口
├── pages/common/demo-utils.uts   # Demo 公共工具：输入框缓存封装
├── pages/user/user.uvue          # 用户模块常用接口
├── pages/friend/friend.uvue      # 好友模块常用接口
├── pages/local-conversation/     # 本地会话模块常用接口
├── pages/message/message.uvue    # 消息模块常用接口
├── pages/team/team.uvue          # 群模块常用接口
├── uni_modules/nim-uts-sdk/      # 云信 IM UTS 插件（已被 .gitignore 忽略，见上方"安装 nim-uts-sdk 插件"）
│   ├── utssdk/
│   │   ├── app-android/          # Android 桥接
│   │   ├── app-ios/              # iOS 桥接
│   │   ├── app-harmony/          # 鸿蒙桥接
│   │   ├── index.d.ts            # 类型声明
│   │   └── interface.uts         # 跨端接口
│   └── readme.md                 # 插件使用说明（初始化/服务清单/命名差异）
└── manifest.json                 # 工程配置（签名信息需本机生成）
```

## 进一步文档

- 插件初始化与服务清单：见 `uni_modules/nim-uts-sdk/readme.md`
- 云信 IM 客户端 API：https://doc.yunxin.163.com/messaging2/client-apis?platform=client
