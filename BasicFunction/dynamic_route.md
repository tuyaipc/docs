## 消息动态跳转(v3.18.0+)
受成本限制，平台无法提供视频消息的推送服务。用户在查看告警消息时仅能查看单张图片，体验较差。为优化消息体验，通过有机结合云存储和本地存储的视频录制功能，提升消息定位的体验。
###功能原理
当用户点击告警消息时，通过动态跳转链接，自动跳转至云存储或本地存储模块，并自动定位至事件发生时间点。
 **详细逻辑**
 >1.事件发生0-120秒，由于云存储或本地存储产生视频片段需要时间，且在事件发生当时，查看实时视频具有更大的价值，因此在此时间范围内点击推送消息会自动跳转至实时视频页面。
 >2.时间发生120秒后，则自动跳转至存储页面，此时，APP将自动判断云存储的开通状况，开通云存储时，则跳转至云存储回放页面，未开通云存储时，则跳转至本地回放。

**操作方式**
>1.点击手机系统推送消息
>2.点击APP消息中心对应消息**点击查看**
>3.点击IPC消息中心对应消息**点击查看**

###配置方式
**APP版本要求** 
>涂鸦智能及智能生活v3.18.0及以上版本及基于ODM v3.18.0以上版本

**面板配置要求**
>00000002vx
>0000000432
>需满足设备使用上述面板。

**Iot平台消息配置**
>Iot平台>>产品详情>>告警配置>>推送消息跳转设置
>此处跳转参数配置为：

```cameraPanel?type=ipcroute&time=${eventTime}```

