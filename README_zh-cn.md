# TrollStore
TrollStore是一款永久监禁的应用程序，可以永久安装你在其中打开的任何IPA。 
TrollStore 能让你无需越狱（虽然这个词也有点年代感）就能安装上你在 App Store 装不上的 iOS 应用。目前支持到的 iOS 版本为 14.0 - 16.6.1，以及 17.0。

它之所以有效，是因为存在AMFI/CoreTrust漏洞，iOS无法正确验证存在多个签名者的二进制文件的代码签名。

支持的版本：14.0-16.6.1、17.0

## Installing TrollStore

有关TrollStore的安装，请参阅上的指南 [ios.cfw.guide](https://ios.cfw.guide/installing-trollstore)

16.7.x和17.0.1+将永远不受支持（除非苹果第三次搞砸CoreTrust…）。

## Updating TrollStore

当新的TrollStore更新可用时，TrollStore设置的顶部将显示一个安装按钮。点击按钮后，TrollStore将自动下载更新、安装并重新发布。

或者（如果出现任何问题），您可以在Release下下载TrollStore.tar文件，并在TrollStore中打开它，TrollStore将安装更新并重新发布。

## Uninstalling an app

从TrollStore安装的应用程序只能从TrollStore本身卸载，点击应用程序或在“应用程序”选项卡中将其向左滑动以将其删除。

## Persistence Helper

TrollStore中使用的CoreTrust漏洞只足以安装“系统”应用程序，这是因为FrontBoard每次在用户应用程序启动之前都会进行额外的安全检查（它调用libmis）。不幸的是，无法安装通过图标缓存重新加载而保留的新“系统”应用程序。因此，当iOS重新加载图标缓存时，所有安装了TrollStore的应用程序，包括TrollStore本身，都将恢复到“用户”状态，不再启动。

解决此问题的唯一方法是将持久性帮助程序安装到系统应用程序中，然后可以使用此帮助程序将TrollStore及其安装的应用程序重新注册为“系统”，以便它们可以再次启动，TrollStore设置中提供了此选项。

在越狱的iOS 14上，当使用TrollHelper进行安装时，它位于/Applications中，并将通过图标缓存重新加载作为“系统”应用程序进行持久化，因此TrollHelpers在iOS 14上用作持久化助手。

## URL Scheme

从1.3版本起，TrollStore取代了系统URL方案“apple-magnifier”（这样做是为了让“越狱”检测无法像TrollStore有唯一URL方案那样检测到TrollStore）。此URL方案可用于直接从浏览器安装应用程序，格式如下：

`apple-magnifier://install?url=<URL_to_IPA>`

在没有安装TrollStore（1.3+）的设备上，这只会打开放大镜应用程序。

## Features

The binaries inside an IPA can have arbitrary entitlements, fakesign them with ldid and the entitlements you want (`ldid -S<path/to/entitlements.plist> <path/to/binary>`) and TrollStore will preserve the entitlements when resigning them with the fake root certificate on installation. This gives you a lot of possibilities, some of which are explained below.

### Banned entitlements

iOS 15 on A12+ has banned the following three entitlements related to running unsigned code, these are impossible to get without a PPL bypass, apps signed with them will crash on launch.

`com.apple.private.cs.debugger`

`dynamic-codesigning`

`com.apple.private.skip-library-validation`

### Unsandboxing

Your app can run unsandboxed using one of the following entitlements:

```xml
<key>com.apple.private.security.container-required</key>
<false/>
```

```xml
<key>com.apple.private.security.no-container</key>
<true/>
```

```xml
<key>com.apple.private.security.no-sandbox</key>
<true/>
```

The third one is recommended if you still want a sandbox container for your application.

You might also need the platform-application entitlement in order for these to work properly:

```xml
<key>platform-application</key>
<true/>
```

请注意，平台应用程序权限会导致副作用，例如沙盒的某些部分变得更紧，因此您可能需要额外的私人权限来规避这一点。（例如，之后您需要为要访问的每个IOKit用户客户端类提供一个异常权限）。

为了让具有“com.apple.private.security.no sandbox”和“平台应用程序”的应用程序能够访问其自己的数据容器，您可能需要额外的权限：
```xml
<key>com.apple.private.security.storage.AppDataContainers</key>
<true/>
```

### Root Helpers

When your app is not sandboxed, you can spawn other binaries using posix_spawn, you can also spawn binaries as root with the following entitlement:

```xml
<key>com.apple.private.persona-mgmt</key>
<true/>
```

You can also add your own binaries into your app bundle.

Afterwards you can use the [spawnRoot function in TSUtil.m](./Shared/TSUtil.m#L79) to spawn the binary as root.

### Things that are not possible using TrollStore

- Getting proper platformization (`TF_PLATFORM` / `CS_PLATFORMIZED`)
- Spawning a launch daemon (Would need `CS_PLATFORMIZED`)
- Injecting a tweak into a system process (Would need `TF_PLATFORM`, a userland PAC bypass and a PMAP trust level bypass)

## Credits and Further Reading

[@alfiecg_dev](https://twitter.com/alfiecg_dev/) - Found the CoreTrust bug that allows TrollStore to work through patchdiffing and worked on automating the bypass.

Google Threat Analysis Group - Found the CoreTrust bug as part of an in-the-wild spyware chain and reported it to Apple.

[@LinusHenze](https://twitter.com/LinusHenze) - Found the installd bypass used to install TrollStore on iOS 14-15.6.1 via TrollHelperOTA, as well as the original CoreTrust bug used in TrollStore 1.0.

[Fugu15 Presentation](https://youtu.be/rPTifU1lG7Q)

[Write-Up on the first CoreTrust bug with more information](https://worthdoingbadly.com/coretrust/).
