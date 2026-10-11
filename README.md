# XREAL One 原生 OSD 扩展研究：一次未能安全部署的尝试

> **独立研究笔记｜2026-10-11｜研究已结束，未在眼镜上安装任何修改版固件。**  
> 本文为个人开发实践复盘，不代表 XREAL 官方意见，也不提供修改固件、绕过签名或刷写设备的教程。

## 想解决什么问题？

我们希望在 XREAL One **眼镜本体**的原生 OSD（屏上菜单）中增加一个自定义一级入口，用眼镜自身按键进入，而不是把手机、Beam Pro 或 PC 上运行的应用画面投射进眼镜。

这是两个不同的开发目标：

- **宿主侧 AR 应用**：使用 XREAL 公共 SDK 在兼容手机或其他宿主设备上运行应用，眼镜负责显示和感知交互。
- **眼镜原生 OSD 扩展**：修改眼镜内部固件所运行的菜单，涉及设备内部的组件加载、升级、安全与恢复机制。

这一差异容易在项目前期被忽略。一个看起来与眼镜菜单相同的 Unity 界面，并不等于已经增加了内置菜单。

## 我们验证了什么？

研究使用官方渠道取得的固件和公开文档进行**离线结构分析**，并对眼镜进行有限的非侵入式连接识别。这里仅公开研究结论，不公开厂商二进制、反编译代码、可写设备命令或可刷写的修改包。

| 研究主题 | 观察与经验 | 不能由此推断 |
| --- | --- | --- |
| 原生菜单结构 | 在本机研究副本中辨识了菜单组件、页面组织和动态库加载边界，制作过仅供离线研究的自定义菜单原型 | 原型通过模拟测试 ≠ 真实眼镜已显示菜单 |
| 固件构成 | 可区分内部应用组件与底层 MCU 更新组件；不同更新对象有不同处理流程 | 某一固件组件出现签名字段 ≠ 所有组件采用完全相同的验签规则 |
| 文件完整性 | 官方应用包中含逐文件 SHA-256 清单；本地原版文件重新计算可与清单一致 | 能计算普通哈希 ≠ 获得厂商签名或设备安装许可 |
| 压缩数据分析 | 官方 MCU 包包含分段压缩数据。起初一部分解码失败，后来发现是第三方解码器对长字面量的边界限制；调整独立研究副本后，全部 **66/66** 分段按声明长度解码，原先成功的 **43** 段新旧哈希一致 | 数据成功解码 ≠ 已取得系统维护权限或可靠刷机能力 |
| OTA 接收组件 | 能观察到一些更新模块及源码路径线索，但未能从已核实的数据中取得所需的完整设备端接收实现 | 相似名称的 DSP OTA 代码 ≠ 负责原生菜单安装的设备端库 |
| 实机连接 | 实际识别了 USB 复合设备和有限的网络服务；官方 SDK 中某些文件导出能力具有设备模式适用条件 | USB 连通 ≠ 存在可用的通用 SSH/ADB 上传入口；诊断导出 ≠ 可任意安装动态库 |
| 安全恢复 | 官方存在升级/恢复页面，但没有完成“修改失败后仍可恢复”的整机验证 | 官方恢复页面存在 ≠ 自制固件失败后保证能救回 |

## 最有价值的三条教训

**一、先验证部署权限，再投入大量底层改造。**  
早期的离线 UI 模拟、指令级测试和动态库装载测试具有研究价值，但它们不能解决“目标设备是否允许把修改程序装进去”这一根本问题。对于封闭硬件，应当先问清楚原生 SDK、OEM 合作、签名授权和维护模式，再决定是否值得投入固件逆向。

**二、把“文件格式正确”和“设备信任该文件”严格分开。**  
包内校验清单可能只是非加密的完整性校验。即使修改后的文件在本地具有自洽哈希，也不等于设备会接受，更不等于签名验证、安全启动和运行时兼容性已经通过。不同芯片/子系统还可能具有各自独立的更新流程，不能简单套用一份结论。

**三、恢复能力应该是部署的硬性前提。**  
可靠的恢复不仅是看见官网有“Recovery”按钮，还应包括经过证实的设备备份范围、适用设备模式、失败场景、可恢复的固件状态以及版本匹配。对正常使用的设备，在这些条件不明确时，不应以试刷来探索更新机制。

## 本次尝试的结果

**在本研究使用的正常 XREAL One、公开可用 SDK 与现有维护权限下，未找到受支持、可验证且具备可靠恢复方案的原生 OSD 自定义菜单部署路径，因此主动停止安装尝试。**

具体状态如下：

- 研究完成了离线原型、公开 SDK 文档核对、原版固件数据分析和有限设备连接识别。
- **没有**把自定义菜单安装到真实眼镜。
- **没有**向设备刷入、发送或写入任何自制固件。
- 研究中保留的某些离线测试结果，不应称作“实机成功”。

这不是证明 XREAL One 在任何条件下都无法修改，也不代表获得 OEM 维护能力的工程团队做不到；它只说明**当前公开途径和可验证恢复条件不满足实际部署**。

## 如果重做一次

建议按下面的顺序推进：首先确认产品的原生扩展权限与支持边界；然后获得明确的开发镜像或官方 SDK、可信的备份与恢复流程；接着在版本一致的隔离环境里验证 UI 和生命周期；最后才考虑通过厂商允许的方式安装与实机验收。若原生 OSD 不开放，应尽早把“宿主侧 AR 界面”作为**不同的产品方案**来评估，而不是把它称作原生菜单。

## 信息公开范围与风险说明

本文刻意**不包含**：任何 XREAL 固件文件、修改后的 ELF/动态库、固件补丁、反编译结果、签名材料、刷写数据格式/命令、真实设备序列号、个人目录结构、网络日志、账户凭据或未获授权的接口调用工具。

厂商知识产权、当地适用法律、服务协议和设备保修政策可能限制固件修改及分发。本文不提供法律结论，也不鼓励在未确认恢复能力的情况下进行设备刷写。

本项目与 XREAL 无隶属、授权或合作关系。商标与产品名称归各自权利人所有。

## 官方参考链接

- [XREAL One 系列原生 OSD 介绍](https://tutorials.xreal.com/docs/glasses/one-series/osd/osd-intro/)
- [XREAL SDK 起步指南](https://docs.xreal.com/Getting%20Started%20with%20XREAL%20SDK)
- [XREAL 官方 OTA](https://www.xreal.com/ota/)
- [XREAL Developer](https://developer.xreal.com/)
- [XREAL 美国商店保修说明](https://us.shop.xreal.com/pages/warranty-policy)

---

## English summary

**XREAL One native OSD extension — lessons from an unsuccessful deployment attempt (October 2026).**

The goal was to add a new menu item to the **glasses' built-in OSD**, accessible using the glasses' own buttons, rather than render an application on a connected phone or host. We studied publicly available SDK documentation, analyzed locally held official firmware offline, and conducted limited non-invasive device connectivity checks.

The critical lesson: **a working offline prototype and valid internal file checksums do not imply permission to install modified firmware.** The publicly documented host-side AR SDK does not establish a third-party installation route into the glasses' native OSD. We did not establish a supported native deployment mechanism, complete firmware acceptance checks, or a reliably verified recovery procedure.

**Outcome: no custom OSD menu was installed; no modified firmware was transferred or flashed.** We stopped rather than risk damaging a working device. This does not imply that vendor-authorized engineers could never implement native extensions.

**No proprietary firmware, binaries, reverse-engineered code, device identifiers, credentials, or flashing instructions are distributed in this repository.** Independent research; not affiliated with or endorsed by XREAL.
