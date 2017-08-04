## 方案：


1. 代码中加开关控制

2. versionName/versionCode 加一些规则

3. ChannelID 控制

## 比较：


| 方案                      |                   优缺点                                      |
| :----------------------- | :--------------------------------------------------------:   |
| 代码开关                  |                    灵活                                        |
| VersionName/VersionCode  | <ul><li>优势：发包时能够直观看到beta/production；</li><li>弊端：beta用户能够感知到这个版本号</li></ul> |
  | ChannelID               |               缺点：影响数据统计/分析               |

  > 关于VersionName/VersionCode区分beta的实现方法：
  >
  > 1. 可以直接在VersionName中写上alpha/beta 字段区分
  > 2. 可以利用VersionCode处理（比如奇偶/是否被n整除）

## 结论：

  - 推荐方案1或2

