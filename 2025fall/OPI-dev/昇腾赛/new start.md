现在我的任务是user_intent_inference（功能三）

一。项目需求：

1.功能1主要实现了识别用户语音，写一份提示词，运用qwen写一份json格式的用户日志，然后用于功能2HA控制

2.每天功能3运行一次，输入为一天的日志json list，要求使用qwen，结合上下文理解，推理用户行为，修改用于功能1的提示词，完成迭代更新。(用户行为日志 SQLite 每日滚动归档)

3.缓存前几天的日志，日志的效力随时间的距离衰减，缓存的时候将之前的日志压缩

4.实际例子如下
# 模拟样本 1：语音识别模型输入与输出

## 输入（符合嵌入式精简需求）

json

```json
{
  "system_bg": "HomeAssistant已连地下室水位传感器、主水泵；用户偏好：水位超阈值时先语音提醒，再自动开水泵",
  "user_speech_text": "地下室水位好像超了，快处理",
  "output_requirement": {
    "speech_feedback_text": "8字内口语化反馈",
    "ha_flood_command": "格式为{\"command_type\":\"flood_control\",\"execute\":true/false,\"device\":\"水泵/传感器\",\"threshold_check\":true/false}"
  }
}
```

## 输出

json

```json
{
  "speech_feedback_text": "已开水泵提醒",
  "ha_flood_command": {
    "command_type": "flood_control",
    "execute": true,
    "device": "水泵",
    "threshold_check": true
  }
}
```

# 模拟样本 2：日志学习模型输入与输出

## 输入

json

```json
{
  "input_logs": [
    "{\"time\":\"2024-05-25 10:10\",\"user_speech\":\"水位超了\",\"ha_command\":{\"command_type\":\"flood_control\",\"execute\":true,\"device\":\"水泵\",\"threshold_check\":true},\"feedback\":\"已开水泵提醒\"}",
    "{\"time\":\"2024-05-25 15:30\",\"user_speech\":\"查下水位\",\"ha_command\":{\"command_type\":\"flood_control\",\"execute\":false,\"device\":\"传感器\",\"threshold_check\":true},\"feedback\":\"水位正常\"}"
  ],
  "task": "1. 学习日志规律：用户说“水位超了”触发水泵启动，说“查下水位”仅触发传感器检测；2. 生成50字内历史信息；3. 检查是否需更新样本1的system_bg",
  "output_requirement": {
    "history_info": "含时间、用户指令、对应设备操作",
    "updated_system_bg": "无冲突填“无需更新”，有冲突则补充"
  }
}
```

## 输出

json

```json
{
  "history_info": "2024-05-25用户2次指令：“水位超了”开水泵，“查下水位”用传感器检测，均完成操作",
  "updated_system_bg": "无需更新"
}
```

# 模拟样本 3：日志学习与提示词更新输入与输出

## 输入

json

```json
{
  "input_logs": [
    "{\"time\":\"2024-05-26 08:45\",\"user_speech\":\"水位降了，关水泵\",\"ha_command\":{\"command_type\":\"flood_control\",\"execute\":false,\"device\":\"水泵\",\"threshold_check\":true},\"feedback\":\"已关水泵\"}",
    "{\"time\":\"2024-05-26 19:20\",\"user_speech\":\"预警误报，取消检测\",\"ha_command\":{\"command_type\":\"flood_control\",\"execute\":false,\"device\":\"传感器\",\"threshold_check\":false},\"feedback\":\"已取消检测\"}"
  ],
  "task": "1. 学习新规律：用户“关水泵/取消检测”触发设备停止；2. 生成50字内历史信息；3. 更新样本1的system_bg补充新偏好",
  "output_requirement": {
    "history_info": "含时间、停止操作触发词、设备",
    "updated_system_bg": "在原bg后补充新偏好"
  }
}
```

## 输出

json

```json
{
  "history_info": "2024-05-26用户2次停止指令：“水位降了”关水泵，“预警误报”取消传感器检测，操作均生效",
  "updated_system_bg": "HomeAssistant已连地下室水位传感器、主水泵；用户偏好：水位超阈值时先语音提醒，再自动开水泵；新增：用户说“关水泵/取消检测”时，停止对应设备"
}
```

二。实现步骤：

1.先在本地windows部署qwen3 0.6b

2.参考模拟输入输出，确定日志格式，输入为一天的日志json list，要求使用qwen，结合上下文理解，推理用户行为，修改用于功能1的提示词，完成迭代更新。(用户行为日志 SQLite 每日滚动归档)，实现功能三核心功能

4.将功能三的核心功能代码封装至一个文件夹中

（以下测试模块封装到另一个文件夹中）
5.参考模拟输入输出，创建一个模拟测试提示词，通过qwen用于生成日志

6.写一个交互脚本，我文字模拟用户输入，生成日志，调整提示词，让我可以直观看到每天日志以及提示词的变化

7.写一个README，写清晰项目的架构，要求（1）有端到端的项目实例，让新手也能看懂（2）告诉我qwen部署到了哪个路径，如何进行部署的，哪些脚本相互依赖，相互联系


现在我只需要完成功能三，功能一仅限测试使用，请完善本地交互测试模块
1.实现我文字模拟用户语音输入
2.实现按天输入，之后使用qwen模型生成模拟日志
3.日志接入功能3处理，更新提示词
4.将每一天的日志与提示词作为输出展示给我

目标闭环

- 文字模拟语音输入 → 按天生成日志 → 接入功能三学习更新提示词 → 输出每一天日志与提示词
新增与调整

- 扩展客户端：在本地测试默认启用 force_rule ，不需要下载模型也能稳定生成结构化结果。
  - 配置项新增 force_rule: true ，必要时可改为 false 使用真实模型。
- 新增脚本： scripts/daily_interactive_test.py 支持按天批量输入或交互输入，自动：
  - 生成当日日志（写入 data/events.db ）
  - 缓存当天日志（ data/cache ）
  - 接入功能三 learn_and_update 更新 configs/prompt_system_bg.txt
  - 展示“当日日志”“更新后提示词”“简史”三项输出
使用方法

- 安装依赖：在项目根目录执行 pip install -r requirements.txt
- 按天批量输入（推荐）
  - 示例： python scripts/daily_interactive_test.py 2025-10-09 "水位超了" "查下水位" "水位降了，关水泵" "预警误报，取消检测"
  - 行为：
    - 为 2025-10-09 生成日志并写入数据库
    - 调用功能三学习，更新提示词
    - 输出当天日志、更新后的提示词、精简历史摘要
- 交互输入（当日）
  - 示例： python scripts/daily_interactive_test.py
  - 输入多条文本后回车空行结束，自动进入学习更新并输出结果
输出示例（已在本地验证）

- 当日日志输出
  - 包含时间、原始文本、HA命令、反馈；写入 data/events.db
- 提示词更新后
  - 在原有 system_bg 后自动追加新的偏好（例如“用户说‘关水泵/取消检测’时停止对应设备”）
- 简史
  - 约50字摘要，含时间/指令/设备操作，便于人工验证学习效果
注意事项

- 当前默认 force_rule: true 保证无网络也能演示功能三；若要使用真实 Qwen 模型，将其改为 false 并确保 model_id 可用。
- 日志缓存与时间衰减已生效；近期历史会参与学习，权重按 exp(-decay_lambda * days) 衰减。
- 功能一仅用于测试：测试模块只生成结构化日志，不直接接入 Home Assistant 执行设备控制。
下一步建议

- 如果你希望一次测试多天数据，可按天多次调用该脚本，例如连续运行三天的样本并观察提示词的迭代变化。
- 我可以补充一个“批量多天测试”脚本，读取 CSV/JSON 按天批量写入与学习，生成每日报告文件，是否需要我继续添加？


1.改成一次可测试多天测试
2.模型已经拉取成功，请改成真实的qwen接口

1.请根据HA可以控制的电器，生成对应的日志，不能我命令空调关闭，生成的日志依然是"ha_flood_command": {"command_type": "flood_control",
2.更新的提示词输出现在没有根据当前日志生成，请找到错误原因并修改
3.提示词更新主要是学习到用户习惯，基本的开关操作不需要持续更新
4.如果可以，请重新设计一份更适合嵌入式开发项目的提示词格式，要求qwen产出json格式的日志，日志格式与之前保持一致

1.让我可以输入模拟命令交互测试

空调开到26度
浴缸中放水，50度我比较喜欢
帮我把水温调低一点
done
开电视
帮我检查一下水龙头，为什么好像还在放水
我马上回家了，帮我再浴缸里放好水
done
开灯，调成黄光就好了
换成白光吧，我更喜欢亮一点的
关掉客厅的灯
谢谢
done