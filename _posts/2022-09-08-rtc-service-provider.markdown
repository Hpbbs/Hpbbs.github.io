---
layout: post
title: "RTC 服务提供商了解"
published: true
categories: [tech]
tags: [saas, rtc]
---

# RTC 服务提供商了解

**对比** 表格：见 1-1

**当下RTC市场阶段**：处于野蛮生长后期，标准制定，从产品能力变为平台性的网络基建，大厂商基于资源占有市场大头，小厂商或难以存在

**限制**：移动Web端( Android, iOS浏览器 )：无法支持投屏

**缺陷**：无国外RTC产品



|                          服务提供商                          |      管理/监控面板      | RTC质量承诺（与会可能是开了CDN，互动应该指IM，不等同于实时视频接入） | 差异化竞争（从宣传上看）   | 插件功能支持 | 项目成熟度 |    开发成本    | 商业成本 | 产品体验入口 |
| :----------------------------------------------------------: | :---------------------: | :----------------------------------------------------------- | -------------------------- | :----------: | :--------: | :------------: | :------------: | :------------: |
| **火山Enging RTC**<br />https://www.volcengine.com/product/veRTC<br />客户生态主要为字节系 |            ✅            | 用户端到端延迟< 250ms，99.98%进房成功率，1000 + 同房连麦，首帧300ms | 音视频编解码积累           |     支持     |     中     |      Demo      | 14元/千分  |https://demo.volcvideo.com/rtc/solution/meeting/|
|              **声网**<br />https://www.agora.io/cn/<br />客户生态广              |            ✅            | 最多17位视屏(一个频道)，千人同时音频<br />会议场景：全球端到端延时 <400ms，全年可用性高达 99.9%，此处额外补充1 | 各场景深化，支持私有化部署 |     支持     |     高     | LowCode + Demo |14元/千分|https://videocall.agora.io/|
|    **腾讯云TRTC**<br />https://cloud.tencent.com/product/trtc<br />客户生态较广    |            ✅            | 单个房间最多支持300人同时在线，最多支持50人同时开启摄像头，300人视频会议，端到端延迟<300ms，<br />抗丢包率超过80%、抗网络抖动超过1000ms，70%丢包率可正常视频 | 未看出                     |     支持     |     高     | LowCode + Demo |14元/千分|https://web.sdk.qcloud.com/component/tuiroom/index.html#/home|
| **华为云 RTC**<br />https://www.huaweicloud.com/product/cloudrtc.html<br />客户生态中等 | 提供OpenAPI而非UI的方式 | 端到端平均时延 <200ms，万人与会，千人互动<br />低卡顿，80%丢包下音频通话流畅，50%丢包下视频通话流畅 | 未看出                     |     支持     |     中     | Demo |14元/千分|暂无|
| **即构 RTC**<br />https://www.zego.im/blog/635a5c17ad5a3451b5da9dfa<br />客户生态中等 |            ✅            | 未收集到                                                     | 虚拟形象，元宇宙           |     支持     |     中     |      Demo      |12元/千分|暂无|
| **阿里 RTC**<br />https://www.aliyun.com/product/apsaravideo/rtc<br />客户生态弱 |            ✅            | 端到端延时**平均**可到250ms                                   | 虚拟形象，元宇宙           |     支持     |     中     |      Demo      |12元/千分|https://alivc-demo-cms.alicdn.com/versionProduct/other/htmlSource/beaconTower/index.html?spm=a2c4g.11186623.0.0.3347fc8cknBVEp#/|

表格 1-1

补充1: 从文档看，视频会议 场景的文档似乎没有更新，支持的平台没有Web，但是 通用RTC支持web，而在社区咨询后，问题回答不明确

### 问题

- 如果使用 混沌说等使用供应商，是否不用走 采购 流程了？

- 由于供应商很多，千万不要扎头进去就直接尝试Demo，首先粗放的选出 6款以内，然后再细致观察这6款以内
