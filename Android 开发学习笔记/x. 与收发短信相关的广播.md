>  https://developer.android.google.cn/reference/android/provider/Telephony.Sms.Intents.html#SMS_RECEIVED_ACTION 

### SMS_DELIVER_ACTION

广播动作：一条新的文本 SMS 消息到达设置。该 intent 只会发生给默认的 sms app。该 app 负责编写消息并通知用户。该 intent 附带如下额外数据：

- “pdus” - bytes[] 组成的 Object[] 包含了组成消息的 PDU。
- “format” - 描述 PDU 格式的字符串。字符串为 “3gpp” 或 “3gpp2”
- “subscription” -  一个可选的长值，描述接收该消息的订阅号。
- “slot” - 一个可选的 int 值，表示接收该消息的 SIM 卡所在的卡槽。
- “phone” - 一个可选的 int，与订阅关联的电话 id。
- “errorCode” -  一个与接收消息相关的可选 int 错误码。

可以使用 `getMessagesFromIntent(android.content.Intent)` 提取这些额外数据。

如果某个广播接收器在处理 intent 时遇到错误，应该适当地设置结果代码。

接收器需要 `Manifset.permission.RECEIVE_SMS` 权限。

### SMS_RECEIVED_ACTION

广播动作:一条新的文本 SMS 消息到达设备。Intent 将作为统治发送给所有已注册的接收者。这些 APP 不需要编辑消息或通知用户。Intent 将携带以下额外值：

- “pdus” - bytes[] 对象组成的 Object[] 对象，包含了组成消息的 PDU。

这些额外值可以使用 `getMessageFromIntent(android.content.Intent)` 方法提取。

如果某个广播接收器在处理 intent 时遇到错误，应该适当地设置结果代码。

接收器需要 `Manifset.permission.RECEIVE_SMS` 权限。

常量值: "android.provider.Telephony.SMS_RECEIVED" 

接收消息代码

```java
public void onReceive(Context context, Intent intent) {
        this.createNotificationChannel(context);
        SmsMessage[] smsMessages = Telephony.Sms.Intents.getMessagesFromIntent(intent);

        for (SmsMessage smsMessage: smsMessages ) {
            String from = smsMessage.getOriginatingAddress();
            String body = smsMessage.getMessageBody();
            long receive_at = smsMessage.getTimestampMillis();
        }
    }
```



### WAP_PUSH_DELIVER_ACTION

广播动作：一条新的 WAP PUSH 消息到达设备。对应的 intent 只会发送给默认的 sms app。该 app 负责编写消息并通知用户。对应的 intent 将附带一下额外数据：

- “transactionId” - (Integer) WAP 事务 ID



### WAP_PUSH_RECEIVED_ACTION

广播动作：一条新的 WAP PUSH 消息到达设备。Intent 将作为通知发送给所有注册过的 APP。这些 APP 不需要编写短信，或者发送通知给用户。Intent 携带如下额外数据：

- “transactionId” -（Integer） WAP 事务 ID 
- “pduType” -（Integer）WAP PDU 类型
- “header” -（Byte[]）信息头
- ”data“ - (byte[]) 信息数据
- “contentTypeParameters” - (HashMap<String, String>) 与内容类型相关的任何参数（从 WSP Content-Type 头中解码出）

如果广播接收器在处理 Intent 时遇到一个错误，应该适当地设置结束码。

 contentTypeParameters  由内容参数键值对组成的 map，键值为参数名。

键值为 'unassigned/0x...' 形式的键表示未分配参数。其中 "..." 为未分配参数的十六进制表示。如果一个参数没有值，它在 map 中的值为 null。

本条广播的常量值为："android.provider.Telephony.WAP_PUSH_RECEIVED" 

## 两种接收短信的intent的区别

```xml
<action android:name="android.provider.Telephony.SMS_RECEIVED" />
```

