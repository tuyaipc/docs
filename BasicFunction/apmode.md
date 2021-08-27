## AP模式（直连模式）

AP模式是指设备与APP间无需介由云端，通过设备释放AP热点，APP与设备直接建立连接实现通讯的使用方式。适用于设备无法获取外网连接的使用场景。

[TOC]



### 名词解释

**本地AP模式**

本地AP模式指，APP无需对设备进行配网，直接连接APP至设备即可正常使用。

**联网AP模式**

联网AP模式指，APP对设备进行配网后，设备在线时可将设备切换至AP模式。本地AP模式与联网AP模式最大的区别在于，设备相关信息在未配网前均保存于云端，未配网则无法正常使用。联网AP模式是在设备联网并获取了所需设备信息后基于获取到的信息进行通讯。



### 相关DP点

#### AP模式状态查询
| DPID   | DPCode      | 传输类型     | 数据类型 |
| ------ | --------------- | ------------ | -------- |
| 自定义 | ipc_ap_mode | 可上报可下发 | 布尔型   |

该DP点用于查询设备是否处于AP模式

APP下发

```
NULL
```

设备返回：

```json
{
 is_ap: 1, //1 当前处于AP模式  0 当前未处于AP模式
 ap_ssid: "xxxx"，//返回设备默认的热点ssid
 password:"admin" //返回热点默认密码（预留暂不使用）
}
```



#### AP模式切换开关

| DPID   | DPCode        | 传输类型     | 数据类型 |
| ------ | ----------------- | ------------ | -------- |
| 自定义 | ipc_ap_switch | 可上报可下发 | 布尔型   |

该DP点用于当设备处于联网状态下降设备切换为AP模式。

APP下发：

```json
{
  ap_enable : 1,    // 1 启动AP模式 0 关闭AP模式
  ap_ssid : IPC-1AB2,   // 用户设置的SSID，目前SSID默认使用设备预设值，固此处与预设SSID相同
  ap_pwd : xxx      // 用户自定义的Wifi密码，设置该密码为Wifi密码并启动热点
}
```

设备回复:

```json
{ 
  ap_enable : 1,   //成功返回1 失败返回0
  errcode:0        //错误码默认返回0即可
}
```



#### AP模式时间同步

| DPID   | DPCode        | 传输类型     | 数据类型 |
| ------ | ----------------- | ------------ | -------- |
| 自定义 | ipc_ap_sync_tz | 可上报可下发 | 布尔型   |

AP模式下，未配网及设备断电后设备因无法与云端进行时间同步，本地录像功能将无法使用。为解决该问题，APP与设备间建立时间同步机制，每次APP连接设备时APP将自身时间和时区同步给设备。

APP下发格式

```
1592461931 //UTC时间戳
+8:00      //时区
```

获取到时间戳和时区信息后，

SDK调用`OPERATE_RET tuya_ipc_set_service_time(IN TIME_T new_time)`设置时间

SDK调用`OPERATE_RET uni_set_time_zone(IN CONST CHAR_T *time_zone)`设置时区

AP模式时间同步(废弃)


| DPID   | DPCode        | 传输类型     | 数据类型 |
| ------ | ----------------- | ------------ | -------- |
| 自定义 | ipc_ap_sync_time | 可上报可下发 | 布尔型   |



### 对接指引

#### 1.DP点数据处理

#### 1.1 handle_DP_AP_MODE

此接口用于处理app发送的`ipc_ap_mode` dp点消息，主要作用是：查询设备是否工作在ap模式下。

客户收到231 dp点的查询请求后，需上报以下格式的dp消息：

`"{"is_ap":%d,"ap_ssid":"%s","password":"%s"}"`

其中is_ap为设备是否工作在ap模式，1为是，0为否

`ap_ssid`为设备工作在ap模式下的ssid，需提供给app，目前demo中ssid格式为**IPC-Mac地址后四位**，如**IPC_1AB2**。

Password可忽略，此字段app不做处理

#### 1.2 handle_DP_AP_SWITCH

此接口用于处理app发送的`ipc_ap_switch`dp点消息，主要工作是切换ap工作模式，并回复dp消息。

App发送的消息格式

`{ ap_enable : 1, ap_ssid : xxxx, ap_pwd : xxx }`

设备回复的消息格式

`{ ap_enable : 1,  errcode:0}`

客户收到232 dp点的消息后，需做以下工作：

1. 解析dp点消息的json串，获取ap模式的操作指令（开还是关）
2. 获取app发送的ssid和password字段，保存在本地，并调用tuya_ipc_save_ap_info接口，保存到涂鸦的数据库中。
3. 构造回复的dp消息：{ ap_enable : 1,  errcode:0}并回复。
4. 按照app指令，关闭或者开启 ap模式。关闭ap模式时，若要连接到原有的前端wifi，请使用tuya_ipc_reconnect_wifi接口，此接口会使用涂鸦数据库中的wifi信息，连接前端。

**注意**：

步骤**<3>**和步骤**<4>**需异步操作，因为步骤**<4>**会导致网络环境的变更，从而引起步骤**<3>**发送失败，此处建议参考demo，创建线程去切换AP模式。

#### 1.3 handle_DP_AP_TIME_SYNC

此接口用于处理app下发的校时命令。

命令格式为`string型`的**utc时间（秒）**。

请调用`OPERATE_RET tuya_ipc_set_service_time(IN TIME_T new_time)`接口来设置涂鸦SDK时间。

时区也需要处理，请通过extern的方式调用`OPERATE_RET uni_set_time_zone(IN CONST CHAR_T *time_zone)`来设置时区，

如下实例：

```c
extern OPERATE_RET uni_set_time_zone(IN CONST CHAR_T *time_zone);

uni_set_time_zone("+8:00")
```

### 2. Tuya SDK初始化方式

#### 2.1 开机自启动AP模式

切换AP模式后，若要保存此模式，且掉电不丢失的话，初始化Tuya SDK时需要根据配置中的ap_mode_onoff来操作。

1. 调用`tuya_ipc_set_ap_info`接口，初始化AP模式的`ssid`的前缀（注意，目前AP的默认格式为“前缀-mac地址后四位”，型如：custom_ap-ABCD,其中custom_ap为客户设置的前缀，ABCD为设备的mac地址后四位）和密码。此处需与app传下来的配置相同，否则SDK校验不通过，会导致恢复出厂设置。
2. 通过`OPERATE_RET tuya_ipc_set_local_info(CHAR_T *schema, CHAR_T *local_id)`接口来设置设备的DP点信息和AP模式下的产品ID。两个参数均需要从TUYA项目经理处获取。
3. `tuya_ipc_start_sdk`中第一个参数`wifi_mode`，若非AP模式，则为`WIFI_INIT_AUTO`，若为AP模式，则为`WIFI_INIT_AP`。

#### 2.2 本地存储及P2P初始化

现在的tuya demo 中，本地存储和P2P均在联网同步到时间后才进行初始化。但若为ap模式，此时是无法从互联网同步时间的，建议客户在其代码合适的位置进行初始化，不需要等待网络同步时间。

同样，若要保证本地存储时间戳及录像时间的正确，请客户确认其设计支持**rtc时钟**，并调用`tuya_ipc_set_service_time`接口设置Tuya SDK中的时间。

详细操作，可以参考**2.3节**中的简单描述。

#### 2.3 设备工作模式的获取

通过`INT_T tuya_ipc_check_cur_stat()`来获取当前涂鸦SDK的工作模式。

返回值如下：

1. 非AP模式
2. AP模式，且云端已激活过 
3. 免配网AP模式，且已经本地激活
4. 免配网AP模式，且本地未经过激活

根据以上四种状态，需进行不同的操作：

若为1，则按照正常的模式去初始化涂鸦模块

若为2，则需要启动P2P模块，等待校时后启动本地存储

若为3，则需要启动P2P模块，上报DP点状态，等待校时后启动本地存储（参考mqtt上线）

若为4，则需要轮询此状态，等待变为3后，执行上面3的步骤。



#### 联网AP模式

联网AP模式与本地AP模式在功能上相互独立，设备可仅对接联网AP模式而无需对接本地AP模式，联网AP模式要求设备必须联网绑定账号后才能切换至AP模式，与此同时，联网AP模式安全性更加完善。



### 3.设备交互

#### 3.1 启动本地AP模式

开发者请遵循统一的模式切换及灯光音效规范

1. 设备首次启动进入配网模式
2. 短按重置按钮切换至AP热点模式，切换时可配合语音提示音
3. 切换至AP模式后，指示灯效果为**红灯常亮**，设备释放AP热点。***由于未加密且处于无外网连接的AP普遍存在手机连接困难的情况，开发者应对热点设置初始密码。该密码请在说明书中说明，或以贴纸形式贴在机身上***。
4. AP模式下长按重置键重置设备，重置设备因恢复设备为出厂状态。

![流程图](https://bianyby-1257475931.cos.ap-shanghai.myqcloud.com/img/流程图.png)

### 4.APP交互

#### 4.1 未配网状态

设备切换至本地AP模式后，将手机连接至设备热点，开启APP，该设备将会出现在设备列表本地设备模块中。首次连接设备时将会要求对设备设置访问密码，设备处于本地AP模式时，并不存在账号绑定关系，可被任何手机连接。当有新的手机连接到设备时，将会被要求验证访问问题以确认使用者身份。

![自定义预设 4](https://bianyby-1257475931.cos.ap-shanghai.myqcloud.com/img/自定义预设 4.png)



#### 4.2 本地AP设备切换至在线设备

用户可通过设置菜单中的连接设备至网络功能，根据引导流程将设备配网成为线上设备。切换为线上设备后，本地AP模式将不再可用直至设备被重置。

![peiwang](https://bianyby-1257475931.cos.ap-shanghai.myqcloud.com/img/peiwang.png)



