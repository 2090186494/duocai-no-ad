# duocai-no-ad
多彩校园（去广告）v4.3.119 [学习测试使用]
[README-去广告说明.md](https://github.com/user-attachments/files/34265066/README-.md)
# 多彩校园 (com.pxkjformal.parallelcampus) v4.3.119 去广告说明

产物：`D:\edgeDN\parallelcampus_noAd_v1.apk`
- 包名：`com.pxkjformal.parallelcampus`，版本 4.3.119 (versionCode 96)，minSdk 24 / targetSdk 30
- 处理方式：只重打包 **dex**，`res/`、`assets/`、`lib/`、`AndroidManifest.xml`、`resources.arsc` 全部沿用原包字节
- 已 zipalign，已签名（v2 + v3，debug key）
- SHA256：`652A39382D0CBB7329EC63C0E8294E06F137DF7773FB3D0596147E93C19DFBAB`

## 打了哪些补丁

| # | 位置 | 处理 | 作用 |
|---|------|------|------|
| 1 | `com/yfanads/android/core/i;->startLoad` | 直接 return | 亿帆聚合 SDK 所有广告类型的请求入口，一刀切 |
| 2 | `com/yfanads/android/core/i;->loadOnly` | 直接 return | 插屏/开屏的“只加载”入口 |
| 3 | `com/yfanads/android/core/i;->showAds(Activity)` | 直接 return | 激励/插屏/原生展示 |
| 4 | `com/yfanads/android/core/i;->showAds(Activity,ViewGroup)` | 直接 return | banner 展示 |
| 5 | `com/yfanads/android/core/i;->findReadyAd` | 返回 false | 就绪广告判定置空 |
| 6 | `Lo9/b;->d(Activity,Integer,Z)` | 直接 return | **插屏广告启动器**：扫码返回、回首页、点底部栏、热启动时弹的那个全屏广告 |
| 7 | `ad/b;->d(Context,String,c)` | 直接 return | 自营广告位请求 `advert-service-api.dcrym.com/advert/app/Advert4AppList` |
| 8 | `ad/b;->c(Context)` | 直接 return | 广告口令偷偷写剪贴板 |
| 9 | `ad/AdView;->z(...)` | 直接 return | 浮窗(PopupWindow)广告弹层 |
| 10 | `ad/AdView;->m(F,Activity)` | 直接 return | 浮窗广告收尾/跳转 |
| 11 | `HomeAdDialog$a;->a(FragmentManager,ExternalBlockItemVo)` | 直接 return | 首页弹窗广告 |
| 12 | `SplashAdActivity;->u0(Bundle)` | 直接 `finish()` | 广告落地页（WebView 广告详情）进不去 |

原包里的证据：`home/activity/SplashActivity` 里 "广点通开屏广告点击"、`q9/b;->c` 日志 "获取插屏了,type："、`o9/b;->d` 内 `new YFAdInterstitialAds(...); loadOnly(...)`、`assets/` 下 `ksad_idc.json`/`bd*`/`opos_module_*`（快手、百度、优量汇资源），说明是「自营广告位 + 亿帆聚合(穿山甲/优量汇/快手/百度)」双通道。

## 安装

1. 先卸载手机上原版（签名不同，不能覆盖安装）：
   `adb uninstall com.pxkjformal.parallelcampus`（或在设置里卸载）
2. 安装：`adb install -r parallelcampus_noAd_v1.apk`，或直接拷进手机点击安装
3. 首次进入照旧登录（卸载会清掉登录态，账号密码还是要用的）

## 还剩下的

- **H5 页面内部的网页广告**：首页/饮水页是远程 H5（`dcxy-home-app.dcrym.com` 等），页面里如果插了 HTML 广告位，属服务端下发，改 APK 动不到。
- **积分任务页的激励视频**（`integraltask` 里直接用穿山甲/优量汇，看广告换积分）：保留，属于你主动点的功能。要一起干掉再说一声。
- 亿帆 SDK 的初始化（`BaseApplication` 里 `YFAdsManager.init`）没删，只掐了广告请求，避免 SDK 半初始化导致崩溃。

## 回滚

原包 `D:\edgeDN\base.apk` 还在，直接装回去即可（同样要先卸载改过的包）。

## 下个版本怎么复用

`apkwork\patch_ads.py` 是纯文本 smali 批量替换脚本，改 `PATCHES` 里的签名即可套到新版本；
编译只走 `apktool b`（只为生成 7 个 dex），再用 `apkwork\repack.py` 的思路把新 dex 塞回官方 APK，最后 `uber-apk-signer` 签名。
这样绕开 apktool 重编资源（本包有一批 0 字节的 `.9.png`，aapt2 会直接报错）。
