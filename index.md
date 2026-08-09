# <center>Wenkang's BLOG</center>

> 工作 Work，生活 Life，记录 Record
>
> 兔兔 🐰 冲冲冲
>
> The brain has cache expiration too.
> I am leaving this here for my future forgetful self — and for you, human reader, as well as my AI agents.

[TOC]

## <center>Services & Digital Products</center>

这个博客主要记录我的技术学习、工作经验和个人项目。随着内容越来越多，我把其中一部分可以被复用、交付或进一步咨询的内容，整理到了独立页面：

### 🛒 [Ji Wenkang 技术服务与数字资源](https://buy.jiwenkang.com/)

它不是一个传统意义上的“商品店”，更像是这个博客的延伸：

* **博客**：记录学习路径、技术文章、项目索引和个人经历；
* **Buy 页面**：整理可以预约、购买、交付或进一步展开的服务与资源；
* **Prompt Library / Vibe Coding**：沉淀可复用的 Prompt 包、项目手册和自动化工具；
* **Red Hat / KVM / Virtio-Win / OpenShift**：沉淀技术咨询、排障笔记和实验环境方案。

适合访问 Buy 页面的场景：

* 想系统学习 Linux / KVM / OpenShift / Red Hat 认证；
* 想请我帮忙看一个虚拟化、容器、网络或 Homelab 问题；
* 想获取 Prompt、脚本、部署手册或实验资源；
* 想搭建 NAS、软路由、AI API、内网穿透或个人数字基础设施；
* 想定制自动化脚本、数据监控工具或个人 AI Agent。

👉 [进入 Ji Wenkang 技术服务与数字资源页面](https://buy.jiwenkang.com/)

---

## <center>Tech Knowledges</center>

### General Linux

- [文件系统基础版](https://www.cnblogs.com/itxdm/p/filesystem_base_version.html)
- [Linux 的启动流程图](./redhat/typora/202208281647545.png)
- [Linux 下 grub 引导修复](./redhat/typora/202208221147727.png)
- [如何将 Linux 系统转移至 LVM 卷](https://linux.cn/article-7718-1.html)
- [Linux 意外操作后如何进行数据抢救](https://www.cnblogs.com/itxdm/p/linuxdate_recover.html)

### Linux Commond Line

- [Linux 下 find 命令](https://www.cnblogs.com/itxdm/p/5936907.html)
- [Linux 下 OpenSSH 高级运用两则](https://linux.cn/article-7475-1.html)

### C/Shell Programing

- [一文让你明白C语言中的指针 ](https://www.cnblogs.com/itxdm/p/c_pointer2.html)

- [C 语言中指针的指针究竟是个啥](https://www.cnblogs.com/itxdm/p/c_pointer_of_pointer.html)
- [C 语言 struct 结构体](https://www.cnblogs.com/itxdm/p/C_language_struct_structure.html)

- [C 语言之函数访问 ](https://www.cnblogs.com/itxdm/p/c_visiting_from_function.html)

- [Linux 下 Shell 脚本几种基本命令替换区别](https://www.cnblogs.com/itxdm/p/something_of_shellscirpt.html)

- [Linux 下 Shell 脚本的基础/高级编程](https://cdn.jiwenkang.com/BashShell/index.html)
### Standard Network

- [网络基础：IP地址子网划分](https://www.cnblogs.com/itxdm/p/6087727.html)
- [记一次在黑盒环境下使用网络设备寻找主机](https://www.cnblogs.com/itxdm/p/Remember_to_use_a_network_device_to_find_a_host_in_a_black_box_environment.html)
- [OpenShift Network Slides](./redhat/OpenShift introduction for personal learning.pdf)
- [OpenShift CNV Videos](https://www.bilibili.com/video/BV1cd4y1D7MW)
- [OpenWrt 双出口分流：在不动主链路的前提下增加第二个境外出口](./openwrt-dual-egress/OpenWrt 双出口分流：在不动主链路的前提下增加第二个境外出口.html)（[md 源](./openwrt-dual-egress/OpenWrt 双出口分流：在不动主链路的前提下增加第二个境外出口.md)）
- [家庭网络分层拓扑：从远端笔记本到双出口的完整链路](./home-network-topology/家庭网络分层拓扑：从远端笔记本到双出口的完整链路.html)（[md 源](./home-network-topology/家庭网络分层拓扑：从远端笔记本到双出口的完整链路.md)）

### Qemu-KVM Virtualization

- [虚拟化介绍](https://www.bilibili.com/video/BV12G411p7JW)
- [虚拟化文字稿](./redhat/QEMU.html)
- [OpenShift Introduce Videos](https://www.bilibili.com/video/BV1TV4y1u7hg/)
- [VM live migration deep dive in OCP-V](./redhat/VM live migration deep dive in OCP-V.pdf)
- [KVM 基座能力来支持OpenShift Virtualization Slide](./redhat/KVM 基座能力来支持OpenShift Virtualization.pdf)

#### Virtio-Win

- [Virtio-win 性能架构解析](./redhat/Virtio-win 性能架构解析.png)
- [Virtio-Win 驱动程序及相关虚拟化技术组件概览](./redhat/Virtio-Win 驱动程序及相关虚拟化技术组件概览.png)

#### NetKVM

- [软件之桥：Windows_如何在_Linux_云上运行](./redhat/软件之桥：Windows_如何在_Linux_云上运行.mp4)
- [NetKVM Feature Presentation](./redhat/NetKVM Presentation.pdf)
- [VirtIO-Win-NetKVM - UDP Segmentation Offload](./redhat/RHEL_Week.pdf)
- [Virtio_Windows_KVM_Enterprise_Drivers](./redhat/Virtio_Windows_KVM_Enterprise_Drivers.pdf)

#### FWcfg

- [Fwcfg & vioinput transfer](./redhat/fwcfg.pdf)

### AI part

- [腾讯研究院 AIGC 发展趋势报告2023.pdf](./AI/腾讯研究院 AIGC发展趋势报告2023.pdf)
- [Instruct Lab & Granite Model .pdf](./AI/Instruct Lab & Granite Model .pdf)
- [CS146S: The Modern Software Developer](https://themodernsoftware.dev/)
- [Deep Learning](https://ocw.mit.edu/courses/6-7960-deep-learning-fall-2024/pages/syllabus/)
- [codechangeai](./AI/codechangeai.md)

#### Prompt Library

把我本地已经在用、以及已经归档过的 prompt 资源拆开收纳。这里不混成一坨，而是按“单条 prompt / prompt 源码项目 / system prompt 归档”分开挂，同时在首页直接放一批可读入口。

> 部分已经沉淀成可复用的 Prompt 工作流、AI Agent 模板或 Vibe Coding 项目手册，会逐步整理到：[buy.jiwenkang.com](https://buy.jiwenkang.com/)

##### Local Workflow Prompts

* [ChatGPT System Prompt Snapshot](./AI/prompts/chatgpt_system_prompt_snapshot.md)
* [RHEL / avocado / tp-qemu Debug User Prompt](./AI/prompts/rhel_avocado_tp_qemu_debug_user_prompt.md)
* [Microsoft Docs MCP Prompt](./AI/prompts/microsoft_docs_mcp.md)
* [oh-my-opencode Setup Prompt](./AI/prompts/oh_my_opencode_setup.md)
* [去 AI 特征 Prompt](./AI/prompts/de_ai_style_rewrite.md)
* [WJI Local Rules Prompt - Sanitized](./AI/prompts/wji_local_rules_sanitized.md)

##### Prompt Source Projects

* [CC-Haha Prompt Source Index](./AI/prompts/cc_haha_prompt_sources.md)

##### System Prompt Archives

* [OpenAI / ChatGPT / Codex System Prompt Archive](./AI/prompts/openai_system_prompts_archive.md)
* OpenAI 直接阅读：[Codex](https://github.com/asgeirtj/system_prompts_leaks/blob/main/OpenAI/Codex.md), [ChatGPT GPT-5 Agent Mode](https://github.com/asgeirtj/system_prompts_leaks/blob/main/OpenAI/ChatGPT-GPT-5-Agent-mode-System-Prompt.md), [GPT-4.1](https://github.com/asgeirtj/system_prompts_leaks/blob/main/OpenAI/GPT-4.1.md), [GPT-4o](https://github.com/asgeirtj/system_prompts_leaks/blob/main/OpenAI/GPT-4o.md)
* [Anthropic / Claude System Prompt Archive](./AI/prompts/anthropic_system_prompts_archive.md)
* Anthropic 直接阅读：[Claude Code](https://github.com/asgeirtj/system_prompts_leaks/blob/main/Anthropic/claude-code.md), [Claude Sonnet 4](https://github.com/asgeirtj/system_prompts_leaks/blob/main/Anthropic/claude-sonnet-4.md), [Claude 3.7 Full Tools](https://github.com/asgeirtj/system_prompts_leaks/blob/main/Anthropic/claude-3.7-full-system-message-with-all-tools.md)
* [Google / Gemini System Prompt Archive](./AI/prompts/google_gemini_system_prompts_archive.md)
* Gemini 直接阅读：[Gemini 2.5 Pro Webapp](https://github.com/asgeirtj/system_prompts_leaks/blob/main/Google/gemini-2.5-pro-webapp.md), [Gemini 2.0 Flash Webapp](https://github.com/asgeirtj/system_prompts_leaks/blob/main/Google/gemini-2.0-flash-webapp.md), [Gemini Diffusion](https://github.com/asgeirtj/system_prompts_leaks/blob/main/Google/gemini-diffusion.md)
* [Other AI System Prompt Archive](./AI/prompts/other_system_prompts_archive.md)
* 其他直接阅读：[xAI Grok 4](https://github.com/asgeirtj/system_prompts_leaks/blob/main/xAI/grok-4.md), [Perplexity Voice Assistant](https://github.com/asgeirtj/system_prompts_leaks/blob/main/Perplexity/voice-assistant.md), [Warp 2.0 Agent](https://github.com/asgeirtj/system_prompts_leaks/blob/main/Misc/Warp-2.0-agent.md), [Proton Luma](https://github.com/asgeirtj/system_prompts_leaks/blob/main/Proton/Luma.md)

## <center>Vibe Coding</center>

把我自己已经跑通过、并且能继续交给 AI 复现的项目整理成一个手册库。这里不只放一个项目，而是持续收纳 Gemini、Cursor、语音输入、图像生成、中转服务之类的 Vibe Coding 项目，每个项目再进入自己的执行手册。

> 部分已经跑通的项目会进一步整理成可复用的交付版本、部署手册或源码资源，统一收录在：[buy.jiwenkang.com](https://buy.jiwenkang.com/)

* [前台文本发送助手](./AI/vibe_coding/frontstage_text_sender/index.md)
* [Xfyun Wayland 实时语音输入](./AI/vibe_coding/xfyun_wayland_asr/index.md)
* [Nanobanana AI 时尚工作室](./AI/vibe_coding/nanobanana_studio/index.md)
* [Cursor Agent Browser 登录方案](./AI/vibe_coding/cursor_agent_browser_login/index.md)
* [Sojiang Survey Assistant](./AI/vibe_coding/sojiang_survey_assistant/index.md)
* [Gemini API 中转站](./AI/vibe_coding/nano_banana_gemini_proxy/index.md)

## <center>Document</center>

### General Help

* [Manpages](https://man.cx/)
* [BashShell 基础](./BashShell/BashShell基础.html)

### Red Hat Links

* [所有红帽产品和文档](https://access.redhat.com/products/)

  * [上下游预览](https://redhatofficial.github.io/#!/main)
  * [操作系统下载](https://access.redhat.com/downloads/content/rhel)
  * [Red Hat 软件包](https://access.redhat.com/downloads/content/package-browser)
* [证书查询](https://www.credly.com/earner/earned)
* [付费订阅查询](https://access.redhat.com/management/subscriptions)




## <center>About me</center>
<script src="https://platform.linkedin.com/badges/js/profile.js" async defer type="text/javascript"></script>
<div class="badge-base LI-profile-badge" data-locale="en_US" data-size="medium" data-theme="light" data-type="VERTICAL" data-vanity="wenkang-ji-b2120324a" data-version="v1"><a class="badge-base__link LI-simple-link" href="https://cn.linkedin.com/in/wenkang-ji-b2120324a?trk=profile-badge">Wenkang Ji</a></div>


* 【从 2016 年 05 月 10 日 到 2022 年 05 月 14 日】拥有 [博客园](https://www.cnblogs.com/itxdm) 6 年撰稿经验，50w 阅读量技术博主。

* 九九后，在沪，前 [linux.cn](https://linux.cn) 开发组成员，前腾科网络技术、交大慧谷红帽课程讲师。详见 Bilibili：[文康的读书经历](https://www.bilibili.com/video/BV1iR4y1c7o4)

  * 【从 2018 年到 2023 年，红帽政策原则无法继续】一线城市培训机构 5 年讲师

    * 已培训超千名工程师，遍布各大高校、电信、金融、电商等相关行业。

* 【从 2019 年 5 月 29 日 到 2023 年 2 月 18 日】三年半兔兔铲屎官 🐰

  * Tips：兔子是非常聪明的宠物，饲养需要多喂水，大量供草，尽早绝育。

* 【从 2022 年 02 月 至 2026 年 07 月】前 Red Hat 虚拟化团队员工 [wji@redhat.com](mailto:wji@redhat.com)（IBM 为 Red Hat 母公司）

  * [质量工程师的旅程：从学习者到领导者](./redhat/质量工程师的旅程：从学习者到领导者.mp4)
  * 目前工作在以下开源项目中：

    * Our Goal: [Virtio-Win 性能架构解析](./redhat/Virtio-win 性能架构解析.png)
    * Virtio-Win 开源项目：[kvm-guest-drivers-windows](https://github.com/virtio-win/kvm-guest-drivers-windows)

      * [Download latest or stable binary drivers](https://docs.fedoraproject.org/en-US/quick-docs/creating-windows-virtual-machines-using-virtio-drivers/index.html)
    * Virtio-Win 安装包：[virtio-win-guest-tools-installer](https://github.com/virtio-win/virtio-win-guest-tools-installer)
    * 测试框架：[avocado-framework](https://github.com/avocado-framework/avocado-vt/)
    * 测试用例：[tp-qemu](https://github.com/autotest/tp-qemu/)

* 联系我：[wenkangji@gmail.com](mailto:wenkangji@gmail.com) / WeChat: ShanghaiedKang


## <center>Honours</center>

* Patches

  * [Jun 23, 2016] 人生发出的第一个 Patch：[Linux.CN 社区：文章翻译](https://github.com/LCTT/TranslateProject/pull/4084)
  * [Aug 28, 2022] 人生首个开源 Patch：[K8S 社区：OVN 文档修复](https://github.com/ovn-org/ovn-kubernetes/commit/8c149e5ecbf49d96f2dc95af4d5fdad3f74b18df)
  * [Apr 6, 2023] 人生首次多人协作 Patch：[OpenPLC 社区：OpenPLC 对于 RT 支持](https://github.com/thiagoralves/OpenPLC_v3/pull/201)

    * 关于 OpenPLC 介绍：[OpenPLC RealTime tests](https://www.bilibili.com/video/BV1eT411C7qA/)

* 本人具有 [RHCA](https://www.credly.com/badges/fd817161-668d-40ed-9dd3-3678cdb35a6c/public_url)、ISTQB、HCIP 等多项职业认证，红帽认证多数均满分通过。

---

## <center>Others</center>

* IT 梦想的起点：

  * [2015 年全国职业院校技能大赛网络搭建与应用竞赛 - 赛题分享](guosai/国赛-compressed.pdf)
* 带我 Linux 入门的师傅们：饶 & 贺老师

  * 上海科管 — 饶老师，带我进入 Linux 世界的引导者。
  * 红帽 GLS 团队 — 贺老师，领我进入 Red Hat 公司的指路人。
* 在红帽公司期间遇到的部分同事，在此感谢：

  * Product Owner: [Yan Vugenfirer](https://www.linkedin.com/in/yanvugenfirer/)
  * Manager: [Qianqian Zhu](https://www.linkedin.com/in/qianqian-zhu-1aa45a35b/)
  * Nini Gu, Menghuan Li, Xiaoling Gao, [Kostiantyn Kostiuk](linkedin.com/in/ACoAADMN34YBkATSEFBDVaBPFmWrmZWEjx1azh8), [Vadim Rozenfeld](https://www.linkedin.com/in/vadim-rozenfeld-0131683/), [Yuri Benditovich](https://www.linkedin.com/in/yuri-benditovich-35a27239/)
  * Qiao Zhao

## <center>Funny</center>

* [红帽创造营第一期 - 多才多艺的开源人](https://www.bilibili.com/video/BV15H4y1p7Lm)
* [清华武汉籍女生英文演讲：我三岁会唱国歌，一场疫情才理解了其中真正的含义](https://www.bilibili.com/video/BV17i4y187sN/)

## <center>Wake up</center>

<iframe width="1785" height="874" src="https://www.youtube.com/embed/a4xvn5SkhF0?list=PLFI1Cd4723_Q2aZchRncgYvOlxdZr5feI" title="1.4 逻辑学的特点" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## <center>Links</center>

### Friend Links

* [CCAV](https://ccav.me/) - 创业哥们
* [h0u5er](https://www.h0u5er.com/) - 黑客好友
* [fishstar](https://www.ssout.top/) - 大学室友
* [xiaoming](https://www.gaoxinming.com/) - 大学室友
* [Alberthua](https://github.com/Alberthua-Perl/tech-docs) - GLS 华老师

### Tool Links

* [手写风格画图](https://excalidraw.com)

---

Domain Names: [Ji Wenkang - 季文康](https://jiwenkang.com), [Wenkang Ji - 文康季](https://wenkangji.com), [ITxdm - IT 兄弟盟](https://itxdm.com)

本博客挂在 Github Pages 上进行[分布式版本控制](https://github.com/heywji/wji.github.io/)，配合 [Cloudflare](https://cloudflare.com/) 实现动态防护。**国内访问很慢，需[魔法](./v2ray/v2ray 搭建教程.html)**。
