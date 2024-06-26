# 项目简介
> 供多人使用的 chatGpt 项目, server, client 跨平台

# 我从这个项目学到了什么？
> 为了熟悉 server 的模块设计，写出一个 完整的 server, 说到底，是为了锻炼我的架构能力
+ orm 模式-sqlite数据库
+ server 的分层 router, serive, model, tools 模块
+ pytest

# 功能列表
+ 预设 prompt 功能: 将自动发送提示词给 gpt，并将标签页设置为相应 prompt 的名

# 优化项
+ v1.0 请求-响应模式
+ v2.0 请求者-调用者模式

# 一定要做
+ TODO: 每次登录，抛弃上下文
    每次登录，都是一个新的对话框，将存储在服务器端的 上下文 清空
    首次登录，将用户名与密码，加入 set
    每次 chat，会将上下文加入容易，但是上下文最多为15条

# 模块
+ Router模块(不用自己写)
+ Controller模块(视图模块)
+ Session 会话管理(每个 cookie 保存多久, 心跳包设计30s保活，超过30s认为已经失效)
+ Sql 调用模块(抽象 table 为 table 对象，通用增删改查)
+ 接口模块(定义 websocket event)
+
# pip list
pip install flask
pip install flask-socketio
pip install python-socketio
pip install openai
pip install toml
 业务逻辑层(距离业务逻辑处理) 

# 消息格式 json
login 接口请求示例：
```json
{
    "username": "testuser",
    "password": "testpassword"
}
```

login 接口响应示例：
```json
{
    "status": 0,
    "message": "Login successful.",
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```
logout 接口请求示例：
```json
{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

logout 接口响应示例：
```json
{
    "status": 0,
    "message": "Logout successful."
}
```

chat 接口请求示例：
{
    "input" : "你好，GPT!"
    "chat_session_id" : "xxxxxxxxxxxxxxxxxxxxx"
}

| 参数名 | 类型 | 必填 | 描述 |
| --- | --- | --- | --- |
| input | string | 是 | 用户输入的内容，可以是文本、图片等。 |
| temperature | float | 否 | 生成文本的随机度，数值越大生成结果越随机，范围在0到1之间。默认为1.0。 |
| max_length | int | 否 | 生成文本的最大长度，超过这个长度将自动截断。默认为1024。 |
| top_p | float | 否 | 按照概率采样生成结果，数值越大生成结果越符合概率分布，范围在0到1之间。默认为1.0。 |
| presence_penalty | float | 否 | 基于模型的先前输出进行惩罚，以避免模型生成相似的内容。数值越大，模型越倾向于生成不同的内容。默认为0.0。 |
| frequency_penalty | float | 否 | 基于之前的词汇使用频率进行惩罚，以避免模型生成过于常见的内容。数值越大，模型越倾向于生成罕见的单词。默认为0.0。 |
| stop_sequence | string | 否 | 当模型生成指定的停止序列时，停止生成结果。默认为空。 |
| model | string | 否 | 使用的模型名称。默认为"chat"，表示使用通用聊天模型。 |

chat 接口响应示例：
{
    "id" : "xxxx"
    "output" : "Here, 这里!"
    "chat_session_id" : "xxxxxxxxxxxxxxxxxxxxx"
    "created_at" : "20230425"
}

| 参数名 | 类型 | 描述 |
| --- | --- | --- |
| id | string | 生成的结果的唯一标识符。 |
| output | string | 生成的结果内容。 |
| created_at | string | 生成结果的时间戳，格式为ISO 8601。 |


# 逻辑积累
+ token 的生成，应该用什么方法确保其不唯一?
答:
+ chagpt 只负责将 输入，转化为 输出. 上下文必须自行存储在 server. 该如何确保用户的上下文不会超出限制，从而导致收费过高
    + 设置会话删除按钮
    + 上下文上限设置为 100 条,  超出100条，则重置对话，并自动发送预设的 prompt.
    + 上下文上限设置为 100 条,  超出100条，则截断最开始的对话，保留最后的100条。
      需要注意的是，如果用户输入了 prompt，则 prompt 要永远位于上下文对话的第一条. (此条不予成立，会影响对话的质量)
+ 限定，5s 内只能问一次，
+ 用户只能在一端登录，第二个端登录会挤掉第一个端。
+ 每个用户最多开三个 tab, 可以调用 fetch_chat_history 来获取历史对话.

# 随感 Question
+ server 对于 用户是无感的，只对 token 是有感的
+ 再写一个 server 之前，需要先设计好哪些东西？
 + 模块设计
 + 接口设计
 + 消息结构设计
 + 表对象，可以写成员函数来抽象
 + json web token 黑名单：确保xxx后，能被删除
 + 多端登录、重复登录验证问题
 + 有人说，pytetst 过于繁琐，不如在文件里 if __main__ 来测试。
    之前我也这么认为，直到使用了 pytest, 使我对于 Python 的包管理有了新的学习见解，以及学习了配置项目根目录的新方式

+ controller， service，model 应该输出哪些关键日志？确保后续分析，哪些日志不用输出
 答：
 在系统设计中，日志的输出应该既能满足排错需要，又能保证日志的清晰度。以下是可能需要输出的关键日志内容：
 + Controller：HTTP 请求、请求参数、返回结果、异常信息。
 + Service：请求处理、业务逻辑、异常信息。
 + Model：数据库操作、SQL 语句、异常信息。

+ 登录请求中解析出用户名与密码，应该放在 controller 层 与 service 层 的哪一层？
答: server要承受多种网络协议的访问(http, websocekt 等), controller 负责 网络逻辑 与 业务逻辑解耦。业务逻辑交给  service 处理.
    故, 解析用户名与密码的操作应该放在 controller 层，因为这属于数据传输和解析的部分，
    重申一下控制器层的职责，
    控制器通常需要处理以下网络连接的逻辑：
    1. 解析请求参数：从请求中解析出参数并进行校验，确保参数合法有效。
    2. 调用服务层：根据请求参数调用服务层提供的接口，获取处理结果。
    3. 处理异常：处理服务层返回的异常，例如参数错误、权限不足等。
    4. 返回响应：将处理结果封装成响应数据，并返回给客户端。
    

+ orm 和传统的 使用 sql语句, 哪个更好些
这取决于项目类型，ORM 跟原生的 TSQL 可以合并使用， 对于增删改 ORM不要太舒服，但是对于复杂的查询 原生的不要太舒服， 技术是服务于业务的，不是一门技术就能满足全部需求。学会取长护短就可以了。还多人说到切换数据库，不是外包项目几乎很少会切换数据库，就算切换，也有很长一段时间 缓冲，所以这方面并不是那么担心。最后用不用 完全取决于项目类型，当然可以用轻量的ORM。比如我上面说的增删改可以，查询你可以走TSQL语句发布于 2020-10-10 16:45

ORM适合业务复杂，用户访问量少的系统，比如企业内部的ERP系统，可能客户端的一个请求触发数据库中几十个表之间的查询修改，不适合互联网系统。

python 虚拟环境管理用的 pipenv
https://zhuanlan.zhihu.com/p/60647332


+ TODO: 编写压力测试工具
+ sql 封装，只抽取部分列