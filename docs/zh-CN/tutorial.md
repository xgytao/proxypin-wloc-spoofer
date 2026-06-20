# ProxyPin WLOC 响应重写脚本图文教程

> 本教程仅用于授权设备测试、安全研究和 QA 场景复现。不要在未授权设备、账号或网络上使用。

本文演示如何在 iPhone 上使用 ProxyPin 本地脚本，拦截 Apple WLOC 网络定位响应，并把响应中的经纬度替换为你设置的测试坐标。

## 准备工作

你需要准备：

- 一台 iPhone。
- ProxyPin。
- 本项目脚本：`proxypin_wloc_compat_v2.js`。
- 一个可以正常联网的 Wi-Fi。
- 一个用于验证定位变化的 App 或系统定位入口。

## 1. 安装 ProxyPin

在 App Store 中搜索并安装 ProxyPin。

![安装 ProxyPin](../assets/zh-CN/tutorial/01-ca-entry.jpg)

## 2. 下载 ProxyPin CA 证书

打开 ProxyPin，进入 HTTPS 代理相关页面，下载 ProxyPin CA 证书。

![下载 CA 证书](../assets/zh-CN/tutorial/02-download-ca.jpg)

下载后，iOS 会提示已经下载描述文件。

![描述文件已下载](../assets/zh-CN/tutorial/03-profile-downloaded.jpg)

## 3. 安装描述文件

打开 iPhone 设置，进入“已下载描述文件”，点击 ProxyPin CA。

![进入设置安装描述文件](../assets/zh-CN/tutorial/04-open-settings.jpg)

安装时允许 Safari 打开设置并继续安装。

![允许安装描述文件](../assets/zh-CN/tutorial/05-install-profile.jpg)

输入锁屏密码后，完成 ProxyPin CA 描述文件安装。

![描述文件安装完成](../assets/zh-CN/tutorial/06-profile-installed.jpg)

## 4. 信任 ProxyPin CA

进入：

```text
设置 > 通用 > 关于本机 > 证书信任设置
```

开启 ProxyPin CA 的完全信任。

![信任 ProxyPin CA](../assets/zh-CN/tutorial/07-trust-ca.jpg)

这一步非常重要。如果没有信任 CA，ProxyPin 无法解密 HTTPS，脚本也无法修改 WLOC 响应。

## 5. 配置代理过滤

打开 ProxyPin 的“代理过滤”设置。

![代理过滤入口](../assets/zh-CN/tutorial/08-proxy-filter.jpg)

关闭“代理域名黑名单”。

![关闭代理域名黑名单](../assets/zh-CN/tutorial/09-disable-domain-blacklist.jpg)

开启“代理域名白名单”，并添加 Apple WLOC 域名：

```text
gs-loc-cn.apple.com
```

如果你需要兼容国际 WLOC 域名，也可以添加：

```text
gs-loc.apple.com
```

![开启代理域名白名单](../assets/zh-CN/tutorial/10-enable-domain-whitelist.jpg)

## 6. 添加本地脚本

进入 ProxyPin 的“脚本”设置。

![进入脚本设置](../assets/zh-CN/tutorial/11-script-settings.jpg)

点击添加脚本。

![添加脚本](../assets/zh-CN/tutorial/12-add-script.jpg)

把本项目的 `proxypin_wloc_compat_v2.js` 完整复制到脚本编辑器中。

匹配 URL 填写：

```text
gs-loc-cn.apple.com/clls/wloc
gs-loc.apple.com/clls/wloc
```

![编辑脚本](../assets/zh-CN/tutorial/13-edit-script.jpg)

建议先在外部文本编辑器里改好经纬度，再复制到 ProxyPin。ProxyPin 的脚本编辑器在长脚本下可能比较卡。

## 7. 修改目标坐标

在脚本中找到：

```js
var TARGET_LONGITUDE = 113.94114;
var TARGET_LATITUDE = 22.544577;
var TARGET_ACCURACY = 25;
```

改成你要测试的位置。

如果不知道经纬度，可以使用高德坐标拾取器：

![高德坐标拾取器](../assets/zh-CN/tutorial/18-amap-coordinate-picker.jpg)

注意顺序：

```text
经度 longitude 在前
纬度 latitude 在后
```

## 8. 启用脚本

保存脚本后，回到脚本列表，开启该脚本。

![开启脚本](../assets/zh-CN/tutorial/14-enable-script.jpg)

## 9. 开启抓包

回到 ProxyPin 主界面，开启抓包。界面上出现红色方块按钮时，表示抓包已经开启。

![开启抓包](../assets/zh-CN/tutorial/15-start-capture.jpg)

## 10. 触发定位

打开系统定位相关 App，或关闭再开启一次定位权限，触发 iPhone 重新请求网络定位。

![触发定位](../assets/zh-CN/tutorial/16-toggle-location.jpg)

如果脚本生效，定位会变成你设置的目标坐标附近。

![定位修改完成](../assets/zh-CN/tutorial/17-location-updated.jpg)

## 11. 如何确认脚本成功

在 ProxyPin 抓包记录中找到：

```text
https://gs-loc-cn.apple.com/clls/wloc
```

成功时，响应头里应该能看到类似内容：

```text
X-WLOC-ProxyPin: v5.3.0
X-WLOC-Origin-Status: 200
X-WLOC-Patched-Locations: 大于 0
```

如果 `X-WLOC-Patched-Locations` 大于 0，说明脚本已经成功修改 WLOC 响应中的定位结果。

## 常见问题

### 看不到 WLOC 请求

可能原因：

- 没有开启抓包。
- iPhone 没有走 ProxyPin 代理。
- ProxyPin CA 没有安装或没有信任。
- 代理过滤规则没有包含 `gs-loc-cn.apple.com`。
- 当前环境 GPS 信号很强，iOS 没有触发 WLOC 网络定位。

### 返回 400

如果响应头里看到：

```text
X-WLOC-Origin-Status: 400
```

说明 Apple 服务端拒绝了原始请求，脚本还没机会修改有效响应。

请检查：

- 脚本是否完整复制。
- `onRequest` 中是否保留了 `request.body = request.rawBody`。
- ProxyPin 是否错误修改了请求体。

### 定位没有变化

可能原因：

- 脚本没有启用。
- URL 匹配规则写错。
- 坐标顺序写反了。
- `X-WLOC-Patched-Locations` 为 0。
- iOS 优先使用真实 GPS，而不是 WLOC 网络定位。

### 室外测试不稳定

室外 GPS 信号强时，iOS 可能优先使用真实 GPS。建议在室内、弱 GPS 或 Wi-Fi 网络定位更明显的环境中测试。

## 测试结束后如何清理

测试完成后建议：

1. 关闭 ProxyPin 抓包。
2. 关闭 Wi-Fi HTTP 代理。
3. 禁用或删除 ProxyPin 本地脚本。
4. 不再使用时删除 ProxyPin CA 描述文件。

再次提醒：本项目只适用于授权测试，不要用于未授权设备、账号或网络。
