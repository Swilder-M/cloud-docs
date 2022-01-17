# 部署相关问题

## 是否可以延长免费试用部署时长？
可以。

如果您有特殊的使用需求，或者其他使用情况，可以提工单或发送邮件(cloud@emqx.io)与我们取得联系，我们会在 24 小时之内回复。


## 要如何才能连接到部署？
您可以通过客户端软件如 [MQTT X 客户端](https://mqttx.app) 进行连接。也可以通过 SDK 进行连接，可以查看 [连接到部署](../connect_to_deployments/overview.md)了解更多内容。

## 连接失败有哪几类原因？
1. 首先需要检查部署是否处于运行的状态，对于没有活跃连接的部署，系统会自动停止。
2. 查看连接地址和端口是否正确，如果是基础版的实例，请留意端口号不是1883和8883
3. 连接到部署需要设置认证鉴权，需要在 `认证鉴权` > `认证` 中设置用户名和密码，并且通过用户名和密码连接


## 会提供定期的升级吗？会对业务有影响吗？大概会在什么时间段升级？
会提供定期的升级，bugfix或者小版本升级可以做到热更新，对用户来说是无感的。如果是大版本升级或者重大更新，会有个设备的闪断，需要客户设备支持自动重连。升级前都会和用户确认或协商升级时间。

## 我的部署被误删除了，能恢复么？
目前删除部署之前需要确认部署名称正确之后才能删除，保证误删除的概率很小。不过需要注意部署删除了之后是无法恢复的。

## 如何能在部署里面看到客户端发的消息？
在部署控制台中是不能直接查看客户端发送的消息，需要做消息的持久化保存情使用规则引擎做转发，同时EMQ X Cloud是不会保存客户端消息。
