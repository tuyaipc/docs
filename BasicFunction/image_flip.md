## 画面翻转

画面翻转是摄像机类设备常用的功能，用于矫正因摄像机摆放方式造成的显示错误，常见的应用场景如摄像机倒置上墙安装会使画面倒置，使用画面翻转功能则可对画面进行矫正。APP提供两种画面翻转DP点。

###画面翻转（旧）
| DPID | DPCode    | 传输类型   | 数据类型 |
|------|-----------|--------|------|
| 103  | basic_flip | 可上报可下发 | 布尔型  |
功能说明：
该DP点实现较为简单，适合仅需对画面进行上下翻转的设备。

###画面翻转（新）
最低支持版本：3.16.0

| DPID | DPCode    | 传输类型   | 数据类型 |
|------|-----------|--------|------|
| 102  | ipc_flip | 可上报可下发 | 枚举型  |

枚举值：

| 枚举值 | 行为 |
| --- | --- |
| flip_none | 关闭 |
|flip_horizontal_mirror|水平镜像|
| flip_vertical_mirror |垂直镜像|
| flip_rotate_180 |180°旋转|

功能说明：
该功能未画面翻转的功能优化版本，配置该DP点无需配置`baisc_flip`旧版DP点。
