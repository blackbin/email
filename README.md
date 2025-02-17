# Email for Android
[![](https://img.shields.io/badge/platform-Android-green.svg)](https://developer.android.google.cn/)
[![](https://jitpack.io/v/mailhu/email.svg)](https://jitpack.io/#mailhu/email)

Email for Android是基于JavaMail封装的电子邮件框架，简化了开发者在Android客户端中编写发送电子邮件的的代码，同时还支持读取邮箱中的邮件。把它集成到你的Android项目中，你只需简单配置邮件服务器的参数，调用一些简易的API，即可完成你所需的功能，所见即所得。

本文档是3.x版本的文档，如果你想阅读2.x版本的文档请点击 [这里](https://github.com/mailhu/email/blob/master/old_doc.md)

* 相关阅读：
  + [《一个邮件框架的重构记录》](https://www.jianshu.com/p/e43456c752c9)
  + [《中国第一封电子邮件》](https://baike.baidu.com/item/%E4%B8%AD%E5%9B%BD%E7%AC%AC%E4%B8%80%E5%B0%81%E7%94%B5%E5%AD%90%E9%82%AE%E4%BB%B6)
  + [《SMTP百度百科》](https://baike.baidu.com/item/SMTP)
  + [《IMAP百度百科》](https://baike.baidu.com/item/imap)
  + [《POP3百度百科》](https://baike.baidu.com/item/POP3)

# 安装引入
步骤一、将JitPack存储库添加到根目录的build.gradle中：
```gradle
allprojects {
    repositories {
        ...
        maven { url 'https://jitpack.io' }
    }
}
```
步骤二、在项目的app模块下的build.gradle里加：
```gradle
dependencies {
    implementation 'com.github.mailhu:email:3.3.2'
}
```
注：因为该库内部使用了Java 8新特性，如果你的项目依赖该库在构建时失败，出现如下错误：
```
Invoke-customs are only supported starting with Android O (--min-api 26)
```
你可以在项目的app模块下的build.gradle里加添如下代码：
```gradle
android {
    ...
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}
```

# 使用文档
###  ● 获取联网权限
在Android项目中的AndroidManifest.xml文件中添加联网权限。
```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

###  ● 配置邮件服务器的参数
[如何开启邮箱SMTP，IMAP，POP3服务和获取授权码？](https://github.com/mailhu/email#%E5%BC%80%E5%90%AF%E6%9C%8D%E5%8A%A1%E4%B8%8E%E8%8E%B7%E5%8F%96%E6%8E%88%E6%9D%83%E7%A0%81)

快速配置，目前只支持QQ邮箱、Foxmail、腾讯企业邮（EXMAIL）、Outlook、163邮箱、126邮箱。
```java
Email.Config config = new Email.Config()
        .setMailType(Email.MailType.QQ)     //选择邮箱类型
        .setAccount("from@qq.com")          //发件人的邮箱
        .setPassword("password");           //发件人邮箱的密码或者授权码
```
自定义配置，自行填写你使用的邮件的服务器host和port
```java
Email.Config config = new Email.Config()
        .setSMTP("smtp.qq.com", 465, true)  //设置SMTP发件服务器主机地址、端口和是否开启ssl
        .setIMAP("imap.qq.com", 993, true)  //设置IMAP收件服务器主机地址、端口和是否开启ssl
        .setPOP3("pop.qq.com", 995, true)   //设置POP3收件服务器主机地址、端口和是否开启ssl
        .setAccount("from@qq.com")          //发件人的邮箱
        .setPassword("password");           //发件人邮箱的密码或者授权码
```
如果你需要频繁使用到Email.Config对象，还可以使用全局配置API，只需配置一次，全局多次使用。
```java
//快速配置
Email.getGlobalConfig()
        .setMailType(Email.MailType.QQ)     //选择邮箱类型
        .setAccount("from@qq.com")          //发件人的邮箱
        .setPassword("password");           //发件人邮箱的密码或者授权码


//自定义配置
Email.getGlobalConfig()
        .setSMTP("smtp.qq.com", 465, true)  //设置SMTP发件服务器主机地址、端口和是否开启ssl
        .setIMAP("imap.qq.com", 993, true)  //设置IMAP收件服务器主机地址、端口和是否开启ssl
        .setPOP3("pop.qq.com", 995, true)   //设置POP3收件服务器主机地址、端口和是否开启ssl
        .setAccount("from@qq.com")          //发件人的邮箱
        .setPassword("password");           //发件人邮箱的密码或者授权码     
```

**注：下面示例代码中的全部回调接口已是从子线程切换回UI线程，可以在里面直接更新UI。**

###  ● 发送邮件
发送邮件setCc( )和setBcc( )方法非必选，可省略；setText( )和setContent( )必需二选一。
如果你已经设置了全局配置，getSendService( )方法无需再传入参数config，getReceiveService( )方法和getExamineService( )方法同理。
```java
Email.getSendService(config)            //已设置全局配置后，无需再传入参数config
        .setTo("to@qq.com")             //收件人的邮箱地址
        .setCc("cc@qq.com")             //抄送人的邮箱地址（非必选）
        .setBcc("bcc@qq.com")           //密送人的邮箱地址（非必选）
        .setNickname("小学生")          //设置发信人的昵称
        .setSubject("这是一封测试邮件")  //邮件主题
        .setText("Hello World !")       //邮件正文，若是发送HTML类型的正文用setContent()
        .send(new Email.GetSendCallback() {
            @Override
            public void onSuccess() {
                Log.i(TAG, "发送成功！");
            }

            @Override
            public void onFailure(String msg) {
                Log.i(TAG, "错误信息：" + msg);
            }
        });
```

###  ● 读取邮箱中的邮件
使用IMAP协议或POP3协议读取邮箱中的邮件，以下使用IMAP协议为例
```java
Email.getReceiveService(config)
        .getIMAPService()           //如果你想使用POP3协议，这里改为getPOP3Service()
        .receive(new Email.GetReceiveCallback() {
            @Override
            public void receiving(Message message, int index, int total) {
                //每读取一封邮件立即回调该方法，返回该封邮件的数据
                Log.i(TAG, "标题：" + message.getSubject() + " 时间：" + message.getSentDate().getText());
            }

            @Override
            public void onFinish(List<Message> messageList) {
                //读取完邮箱的全部邮件会回调这个方法
                Log.i(TAG, "全部邮件数量：" + messageList.size());
            }

            @Override
            public void onFailure(String msg) {
                Log.i(TAG, "错误信息：" + msg);
            }
        });
```

使用IMAP协议快速读取邮箱中的邮件，**但不解析邮件中的内容**，速度提升50%（不支持POP3协议）
```java
Email.getReceiveService(config)
        .getIMAPService()
        .fastReceive(new Email.GetReceiveCallback() {
            @Override
            public void receiving(Message message, int index, int total) {
                Log.i(TAG, "标题：" + message.getSubject() + " 时间：" + message.getSentDate().getText());
            }

            @Override
            public void onFinish(List<Message> messageList) {

            }

            @Override
            public void onFailure(String msg) {

            }
        });
```

使用IMAP协议极速读取邮箱中的邮件，**但不解析邮件中的内容**，速度最快，更加智能和人性化（不支持POP3协议）；例如：第一次登录邮箱时，框架会拉取全部邮件消息，耗时会相对长一点，拉取到邮件消息并缓存到SQLite数据库中，下一次拉取消息，只需传入本地已存在的uid数组，框架会自动差异拉取邮件消息。**（推荐使用syncMessage方法来同步邮件）**
```java
//本地已缓存有的邮件uid时
long[] originalUidList = new long[]{15, 40, 35};
//本地没有邮件uid时
//long[] originalUidList = new long[]{};

//极速同步
Email.getReceiveService()
        .getIMAPService()
        .syncMessage(originalUidList, new Email.GetSyncMessageCallback() {
            @Override
            public void onSuccess(List<Message> messageList, long[] deleteUidList) {
                //messageList是本地还不存在的邮件消息，需要把这些消息缓存到本地。
                //deleteUidList是需要删除本地对应uid的消息。
            }

            @Override
            public void onFailure(String msg) {

            }
        });
```


###  ● 获取邮箱中全部邮件的UID
UID是邮箱创建的邮件序号，每个用户邮箱账号的序列号都是独一无二的。使用UID来同步邮件速度很快，每次同步最新的UID下来，再与之前缓存的UID进行比较即可分析哪些邮件是新的，哪些邮件是已被删除的。
```java
Email.getReceiveService(config)
        .getIMAPService()
        .getUIDList(new Email.GetUIDListCallback() {
            @Override
            public void onSuccess(long[] uidList) {
                for (long i : uidList) {
                    Log.i(TAG, "UID：" + i);
                }
            }

            @Override
            public void onFailure(String msg) {
                Log.i(TAG, "错误信息：" + msg);
            }
        });
```

###  ● 通过UID获取一封邮件数据
下面代码中的getMessage()方法的第一个参数是指该封邮件的UID
```java
Email.getReceiveService(config)
        .getIMAPService()
        .getMessage(870, new Email.GetMessageCallback() {
            @Override
            public void onSuccess(Message message) {
                Log.i(TAG, "主题：" + message.getSubject());
                Log.i(TAG, "发件人：" + message.getFrom().getNickname());
                Log.i(TAG, "收件人：" + message.getTo().getNickname());
                Log.i(TAG, "日期：" + message.getSentDate().getText());
                Log.i(TAG, "内容：" + message.getContent());
            }

            @Override
            public void onFailure(String msg) {
                Log.i(TAG, "错误信息：" + msg);
            }
        });
```

###  ● 通过一组UID获取多封邮件数据
```java
long[] uidList= new long[]{15, 40, 869, 870};
Email.getReceiveService(config)
        .getIMAPService()
        .getMessageList(uidList, new Email.GetMessageListCallback() {
            @Override
            public void onSuccess(List<Message> messageList) {
                //messageList是对应该组UID的邮件数据
            }

            @Override
            public void onFailure(String msg) {
                Log.i(TAG, "错误信息：" + msg);
            }
        });
```

###  ● 读取邮箱中全部邮件的数量
使用IMAP协议或POP3协议读取邮箱中的内容，以下使用IMAP协议为例
```java
Email.getReceiveService(config)
        .getIMAPService()           //如果你想使用POP3协议，这里改为getPOP3Service()
        .getMessageCount(new Email.GetCountCallback() {
            @Override
            public void onSuccess(int total) {
                Log.i(TAG, "邮箱中邮件总数：" + total);
            }

            @Override
            public void onFailure(String msg) {
                Log.i(TAG, "错误信息：" + msg);
            }
        });
```

###  ● 读取邮箱中未读邮件的数量
```java
Email.getReceiveService(config)
        .getIMAPService()
        .getUnreadMessageCount(new Email.GetCountCallback() {
            @Override
            public void onSuccess(int total) {
                Log.i(TAG, "未读邮件数：" + total);
            }

            @Override
            public void onFailure(String msg) {
                Log.i(TAG, "错误信息：" + msg);
            }
        });
```

###  ● 检查邮配置和账号密码是否正确
```java
Email.getExamineService(config)
        .connect(new Email.GetConnectCallback() {
            @Override
            public void onSuccess() {
                Log.i(TAG, "连接成功！");
            }

            @Override
            public void onFailure(String msg) {
                Log.i(TAG, "错误信息：" + msg);
            }
        });
```

###  ● Message对象的成员方法说明
```java
//获取邮件的uid，仅使用IMAP协议时生效
long uid = message.getUid();
//获取邮件的主题
String subject = message.getSubject();
//获取该封邮件的内容
String content = message.getContent();
//获取发件人（对方）的邮箱地址
String fromAddress = message.getFrom().getAddress();
//获取发件人（对方）的昵称
String fromNickname = message.getFrom().getNickname();
//获取收件人（自己）的邮箱地址
String toAddress = message.getTo().getAddress();
//获取发件人（自己）的昵称
String toNickname = message.getTo().getNickname();
//获取邮件的发送时间，格式为格式为yyyy年M月d日 hh:mm
String date = message.getSentDate().getText();
//获取邮件的发送时间，单位毫秒
long millisecond = message.getSentDate().getMillisecond();
//判断该封邮件是否已读
boolean isSeen = message.isSeen();
```


# 开启服务与获取授权码
若使用QQ邮箱（其他邮箱参考QQ邮箱），开启服务的步骤：登录QQ邮箱，进入【设置】-【帐户】，把下列服务开启，然后获取授权码。如下图：

<img src="https://github.com/mailhu/email/blob/master/image/image_1.PNG"  height="200" width="600">
<img src="https://github.com/mailhu/email/blob/master/image/image_2.PNG"  height="250" width="600">

# 混淆
```
-dontwarn org.apache.**
-dontwarn com.sun.**
-dontwarn javax.activation.**
-keep class org.apache.** { *;}
-keep class com.sun.** { *;}
-keep class javax.activation.** { *;}
-keep class com.smailnet.email.** { *;}
```

# 更新日志
* Email for Android 3.3.2
  + 快速配置增加对腾讯企业邮、outlook邮箱的支持
  + 修复使用局部配置时读取邮件出现崩溃的现象
  + 修复Email框架销毁内部对象出现空指针的问题
  + 优化内部代码
  
* Email for Android 3.3.1
  + 修复使用局部配置时发送邮件出现崩溃的现象
  + 优化内部代码

* Email for Android 3.3.0
  + 增加是否设置SSL的功能
  + 增加126邮箱的快速配置
  + 修复163，126邮箱读取邮件被阻止的问题
  + 修复163邮箱快速配置端口错误的问题
  + 重构和优化内部代码


* Email for Android 3.2.2
  + 增加消息极速同步API
  + 修复一些已知bug

* Email for Android 3.2.1
  + 修复获取收件人昵称错误的问题
  + 对解析的邮件内容进行“提纯”
  + 增加框架初始化的方法

* Email for Android 3.2.0
  + 增加邮件服务器参数的全局配置API
  + 增加快速读取邮件的API
  + Message类增加获取收件人、发件人昵称和判断邮件是否已读功能

* Email for Android 3.1.2
  + 修复getMessage方法回调两次的问题

* Email for Android 3.1.1
  + 规范回调接口的命名和简化接收邮件回调接口的逻辑

* Email for Android 3.1.0
  + 修改JavaMail的版本，用Android版本的JavaMail替换原来Java标准版的JavaMail
  + 彻底修复读取邮件时Object对象类型转Multipartd类型时出现java.lang.ClassCastException的错误
  + Email for Android支持的Android API级至少为19

* Email for Android 3.0.0
  + 对该邮件框架内部的全部代码进行重构。
  + 重新设计该框架的API和回调接口，使其更简单易用
  + 增加获取全部邮件数量的API和未读邮件数量的API
  + 增加通过邮箱UID来同步邮件的API
  + 增加通过一个UID或一组UID来获取邮件的数据
  + 废弃之前版本的子线程切换回UI线程的API设计，3.0.0版本后切回UI线程无需传入Activity参数
  + 3.0.0版本的API在子线程中执行完任务后自动切回UI线程，不再提供是否切换的选择
  + 3.0.0版本不向下兼容，后续更新的大版本都会在该版本基础上进行优化和完善

* Email for Android 2.4.1
  + 修复解析邮件内容出现的bug

* Email for Android 2.4.0
  + 增加检测垃圾邮件发送者的接口

* Email for Android 2.3.2
  + 再次修复ContentUtil类异常

* Email for Android 2.3.1
  + 修复ContentUtil类异常

* Email for Android 2.3
  + 增加设置发信人昵称的接口
  + 增加设置抄送人，密送人的接口
  + 增加设置文本型邮件正文的setText( )接口
  + 设置收件人的接口使用setTo( )代替setReceiver( )，2.3版本后建议使用setTo( )

* Email for Android 2.2
  + 对发送邮件和接收邮件等接口增加一些新特性
  + 优化部分代码

* Email for Android 2.1
  + 增加使用IMAP协议接收邮件的接口
  + 增加检查Host和Port的工具类

* Email for Android 2.0.1
  + 修改个别类和方法的命名

* Email for Android 2.0
  + 增加使用POP3协议接收邮件接口
  + 增加检查邮件服务器是否可连接的接口
  + 重构EmailConfig类
  + 重构使用SMTP协议发送邮件的类
  + 2.0版本是全新版本（不向下兼容）

* Email for Android 1.1
  + 优化和重构代码
  + 增删和修改个别接口

* Email for Android 1.0
  + 只需简单配置，即可发送一封电子邮件  

  
# 联系我
**E-mail：**<a target="_blank" href="http://mail.qq.com/cgi-bin/qm_share?t=qm_mailme&email=Zx0AEgYJDxInAQgfCgYOC0kECAo" style="text-decoration:none;"><img src="http://rescdn.qqmail.com/zh_CN/htmledition/images/function/qm_open/ico_mailme_01.png"/></a>

**微信扫一扫：**

<img src="https://github.com/mailhu/email/blob/master/image/WeChat.png"  height="100" width="100">


# License
```
Copyright 2018 张观湖

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```