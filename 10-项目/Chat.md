对标电报、微信的轻量私密即时通讯软件（BS，CS，移动端）

私聊、群聊。私密聊天、文件多端快传及多端管理（多端文件记录统一）

## DDD 架构设计
<font style="color:rgb(0, 0, 0);">一个微服务就是一个领域</font>

<font style="color:rgb(0, 0, 0);">领域domain：infrastructure entity service repository</font>

<font style="color:rgb(0, 0, 0);"></font>

<font style="color:rgb(0, 0, 0);">基础设置层 infrastructure：数据库、缓存</font>

<font style="color:rgb(0, 0, 0);">领域层 domain：</font>

+ <font style="color:rgb(0, 0, 0);">朋友领域：用户 朋友</font>
+ <font style="color:rgb(0, 0, 0);">聊天领域：文字 语音 文件</font>

应用服务层 service：调用基础设置和领域，处理用户界面发送的请求

用户接口层 api：

用户界面、web 服务

## 功能
1. 联系人：ID、头像、昵称
    1. 备注表：主 ID、副 ID、主对副的备注
    2. 联系人的增删
    3. 个人信息的改
2. 消息：聊天框 ID、接发消息（文字、语音、文件）
3. 通话：语音通话、视频通话

