---
title: 十条你得知道的 Wwise Launcher 用法（2019 版）
date: 2019-07-08 19:48:59
tags: wwise, gameaudio
---

注：本文基于 2018 年初在 Audiokinetic 公众号上发表的文章《十一条你可能不知道的 Wwise  Launcher 用法》，由作者针对软件更新做了改动，**不影响操作的截图沿用旧版本**。

本文基于以下环境

- Wwise Launcher: 2019.5.0.905
- Wwise: 2019.1.0.6947
- macOS Mojave 10.14.5

从 2016 年起，你在 Audiokinetic 官网点击下载 Wwise 的时候，实际上下载的是 Wwise Launcher（下文简称 Launcher），后来你就习惯用 Launcher 来下载更新 Wwise 了。你还学会了用它管理本机上的多个工程和 Wwise 版本，对吧？但对新手，Launcher 可能还有不明显的功能和注意事项对你会很有用，本文初版一年多后，我们再来梳理一下吧！



## 1. Wwise Launcher 有权威参考资料吗？

如今我们有了详细的[官方图文文档](https://www.audiokinetic.com/library/edge/?source=InstallGuide&id=the_wwise_launcher)，可从 Launcher 的` ？`页菜单项``Wwise Launcher Documentation`直达，本文中的许多内容已可在文档中找到。不过在本文写作时，文档只有英文版，零星信息尚未与最新软件版本同步，因此下文还会给出必要的向导。英文自信的同学扫读本文即可。



## 2. 程序员：哪里有 Wwise 集成／整合方面的例程？

我们可以通过 Launcher 来安装 Wwise 的[`IntegrationDemo`](https://www.audiokinetic.com/library/edge/?source=SDK&id=soundengine__integration__samplecode.html)。如果我们已经安装了 Wwise，但是没有找到这个 Demo，则需要打开 Launcher，找到`WWISE`页上我们的目标版本，然后修改安装。

{% asset_img modify-installs.jpg %}

勾选左边选项框中的`SDK (C++)`，和右边的目标平台。

{% asset_img install-sdk-demos.jpg %}

之后点击`Install...`开始安装。安装结束后切换到`SAMPLES`页，选中正确的 Wwise 版本（图中为`2017.1.4`），便可以看到`IntegrationDemo`了。

{% asset_img samples-demos.jpg %}

如果想运行 Demo，则直接点击`Run IntegrationDemo`；如果想打开对应的 Wwise 工程检查或修改声音设计内容，那么点击`Open in Wwise`。

如果想打开源码工程，则点击左边按钮的下拉菜单，之后打开 Demo 所在文件夹，

{% asset_img open-demo-code-projects.jpg %}

通过文件系统打开对应平台的子文件夹，找到并打开 IDE 工程文件。

{% asset_img open-demo-code-project-2s.jpg %}

在研究 IntegrationDemo 的时候，推荐对照[`Wwise SDK Help`](https://www.audiokinetic.com/library/edge/?source=SDK&id=index.html)文档来学习，特别是`声音引擎集成纵览`一节。

不论我们的项目用的是自研引擎还是商业引擎，`IntegrationDemo`都是我们学习 Wwise 整合代码最好的帮手。这是因为`IntegrationDemo`是 Wwise SDK 的一部分，Audiokinetic 会持续维护测试这组例程和对应的 Wwise 工程；每当有重要的新功能时，该 Demo 中一般会加入新功能演示。

更重要的是，假如我们的项目出现了奇怪的问题，开始怀疑是不是 Wwise 的 bug 时，可以首先尝试用 IntegrationDemo 来对照重现问题，Demo 提供的简化环境经过了反复测试，有助于隔离发现应用端的整合问题。Demo 中重现不了的问题往往会暴露出用户端的使用不当。



## 2. 我们公司大部分机器在内网，怎么安装 Wwise？

我们可以用某台外网机安装需要的 Wwise 组件，然后通过 Launcher 制作离线安装包给内网机使用：

{% asset_img offline-installers.jpg %}

添加插件为单独步骤，在制作离线包时注意勾选所有需要的插件。

做好的离线包除了组件文件夹`bundle`外，会自带一个配套的 Launcher 安装包：

{% asset_img offline-installer-2s.jpg %}

最后将离线安装包发送到各台内网机上，在内网机上分别装好包里自带的 Launcher，再打开 Launcher 定位到离线包来安装。

{% asset_img offline-installer-3s.jpg %}

注意：Windows 版的 Launcher 只能安装 Windows 版的设计工具，Mac 版类似。



## 3. 我下载安装了 Wwise，但是我的工程在生成 SoundBank 时报错，说最多只能使用 200 个声音文件，怎么办？

这说明我们的工程没有添加合适的 Wwise 授权码。我们需要打开 Launcher，**登录我们的 Audiokinetic 账号**，在`PROJECTS`页找到我们的 Wwise 工程，点击钥匙状按钮，接着我们有两个选择：

1）注册新项目，并等待 Audiokinetic 商务联系人批准项目注册并接收系统邮件取得授权码。

{% asset_img project-licenses.jpg %}

这时我们会来到官网项目注册页面，需要根据向导填好所有信息并提交申请。之后如果确信申请通过了但没有收到系统邮件，则最好检查一下垃圾邮箱。

{% asset_img project-license-2s.jpg %}

2）如果我们的项目确定已经注册了，只是没有导入授权码，则需要注意：每个 Wwise 项目在 Audiokinetic 官网上都有若干管理员（Wwise Project Leader)，他们一般是我们自己项目团队的成员。可以联系管理员把我们的 Audiokinetic 账号加入该项目，之后便可以在 Launcher 里工程的`Set Project License`菜单中找到对应的项目授权码并授权项目了，**做授权码导入操作时要确保关闭已经打开的工程**。

{% asset_img project-licenses-2.jpg %}



## 4. 如何安装 Unity 集成？

假设我们的电脑上已经装了 Unity，并且创建了 Unity 工程，而我们现在想给其中某个 Unity 工程安装 Wwise 集成。这时我们需要来到 Launcher 的`UNITY`页，点开顶部菜单的浏览按钮：

{% asset_img unity-browse-projects.jpg %}

选中目标 Unity 工程并确认后，在`UNITY`页的工程列表里就能看到这个工程了。接下来我们有两个选择：

1）直接从官网下载 Unity 集成并同时安装，

{% asset_img unity-integrates.jpg %}

2）先下载离线安装包，

{% asset_img unity-offline-1s.jpg %}

然后通过 Launcher 手动安装。

{% asset_img unity-offline-2s.jpg %}

注意，无论用哪种方法，安装期间必须**关闭所有打开的 Unity 编辑器**。如果需要定制 Unity 集成，可以在离线安装包中解压出源码包，再[重新编译 Unity 集成](https://www.audiokinetic.com/library/edge/?source=Unity&id=pg__building.html)。

Unreal Engine 的安装方法与此类似，在`UNREAL ENGINE`页中完成。



## 5. 新手怎么学习 Unity 和 Unreal Engine 集成？

学习 Unity 集成有两个选择：

1）通过 Launcher 安装运行 Unity 集成示例，再对照[官方文档](https://www.audiokinetic.com/library/edge/?source=Unity&id=main.html)学习。

{% asset_img unity-demos.jpg %}

2）通过[官方免费的认证课程系列](https://www.audiokinetic.com/learn/certifications/) 中的 Wwise-301 （以及 Wwise-251 中的一部分）学习，对 Wwise 新手而言这一般意味着你要先学习入门课程 Wwise-101，但可能很多过来人都会告诉你，这是值得的，别问我为什么知道 ^_*

学习 Unreal 集成， 我们也有两个选择：

1）通过 Launcher 的`UNREAL ENGINE`页安装运行 Unreal 集成示例，再对照[官方文档](https://www.audiokinetic.com/library/edge/?source=UE4&id=index.html)学习。

{% asset_img unreal-demos.jpg %}

2）通过 Launcher 安装运行 Wwise 的空间音频 Demo：Wwise Audio Lab

{% asset_img wal-1s.jpg %}

{% asset_img wal-2s.jpg %}



## 6. 我网速慢，看在线文档不方便，如何在本地查看 Wwise 的音频设计和程序整合文档？

我们可以通过 Launcher 安装离线文档，注意要选择想要的平台。

{% asset_img offline-doc-1s.jpg %}

之后就可以在`WWISE`页的对应 Wwise 版本下找到各个平台对应的多语言离线文档了。

{% asset_img offline-doc-2s.jpg %}

容易忽略的是：Unity 和 Unreal Engine 集成的离线包中自带了离线文档（`.chm` 和`.html`包），解包时不要错过哦～



## 7.我感觉碰到了一个 Wwise 的 Bug，该怎么上报？

我们可以去`?`页的`About`，点击`Report a Bug...`，接着根据向导提供必要信息确认上传即可，支持上传截图、Wwise 工程 zip 包和普通 zip 包。汇报成功后，你会看到类似下图的截图，别问我怎么知道的 :P

{% asset_img wwise-bug-report.png %}



## 8. Launcher 在安装中出现错误或者操作失败怎么办？

我们可以去`?`页的`About Wwise Launcher...`，

{% asset_img log-1s.jpg %}

找到 Launcher 的完整日志，

{% asset_img log-2s.jpg %}

然后想办法汇报给 Audiokinetic 的项目技术支持或[社区问答论坛](https://www.audiokinetic.com/qa/)，或者如果你确信是 Bug，参见上一条。

最常见的问题是网络问题，出这类问题的时候如果开了 VPN，则可以尝试关闭了 VPN 再使用 Launcher。

有一类特殊问题是通过 Launcher 安装和升级 Unity 集成的时候失败了，这时怎么办呢？

首先我们需要根据[Unity 集成版本说明](https://www.audiokinetic.com/library/edge/?source=Unity&id=pg__releasenotes.html)来确认要安装的 Wwise 版本对应支持的 Unity 版本。

如果版本都是对的，则需要打开 Unity 编辑器观察控制台里的错误信息。Wwise Unity 集成安装时需要运行 Unity 程序来做一些初始化或者升级工作，这个过程中 Unity 工程内部可能错误，但这些错误并不会在 Launcher 界面上显示出来，要在 Unity 编辑器中查看。



## 9. 向 Wwise 工作人员提技术问题时，怎样让沟通更准确高效？

我们在项目档期紧张时遇到 Wwise 相关的技术问题，常常撒腿就找官方技术支持。但是巧妇难为无米之炊，没有客观详实的诊断信息，支持人员也只能来回提问试探，沟通效率可能会不理想。为了高效沟通，我们可以善用 Wwise 的 Profiler（性能分析器）来记录问题过程，利用 Launcher 来制作诊断包，在提问时将诊断包发送给官方技术支持。

Windows 版本的 Launcher 整合了 Wwise 中的辅助工具`Wwise Project Zipper`的功能，支持 Windows 和 macOS。

{% asset_img zippers.jpg %}

可以将 Wwise 工程及性能分析器日志记录 （profiler session）定制内容后打包。

{% asset_img zipper-2s.jpg %}

之后便可以前往官网项目的技术支持频道将 zip 包作为附件发送给 Audiokinetic 的技术支持了。这样的做法通常可以为我们省去好几轮前期沟通。



##  10. 怎样了解 Audiokinetic 的新闻和最新技术？

我们打开 Launcher 的首页，便可以看到最新的 Audiokinetic 的英文版新闻和技术博客。

{% asset_img home-feedss.jpg %}

首页上还有社区问答论坛的最新提问，可以从中学习其他用户的经验。

如果要看中文版，现在可以直接点击 Launcher 首页右上角的语言列表切换。



## 后记

越来越多的集成开发环境包括 Unity 和 Unreal Engine 利用独立于编辑器的 Launcher（启动器）作为控制中心程序来管理工程和资源。这样做，一来可以方便用户管理多个引擎版本和工程，避免和操作系统文件管理器中铺天盖地的文件夹和文件类型缠斗，甚至发生零散文件操作引起的意外；二来可以整合引擎开发商提供的一系列服务。在 Wwise 工作流程中，Wwise Launcher 正在扮演类似的角色，并且还在成长中。