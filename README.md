# 知苗易约抢购小程序

目前开发完成，暂未测试，因而暂不给（以后可能也不给）使用说明。本项目是作者本人写的第一个Go程序。只作学习交流使用。之初遇到不少的问题，多亏一些大佬点拨，不胜感激。

【2021-03-20】**v1.0.0**  验证码exe识别模块改为直接调用py脚本，以支持非windows用户使用

**建了个群，感兴趣的可以加一下：788512807**

* 问题进度
  * **【已解决】2021/2/22 -运行时出现403错误，经调试后发现小程序做出了相当大的更新导致的。原来请求的url都添加了一些动态的变化参数如key302和expire302**
    * 【问题详细说明】当程序请求原来的URL时，会返回403错误，但实际是由于302重定向引起的。即当第一次请求URL时，实际请求的资源是不存在的，小程序后端会重定向到一个新的URL，并赋予key302和expire302两个参数，而在zmyy程序中是无法被重定向的。
    * 【解决方案】若想拿到重定向的url，可以通过请求原url，并在响应报文resp的header中得到Location参数，即为重定向的url
    * 有空会更新代码
  * **【已解决】获取验证码时，并未得到验证码图片，而是一大串文字**
    * 该大串响应报文是图片的Base64编码，可转换成图片
  * **【已解决】滑块验证码的x轴坐标可通过opencv来解决。附一个大佬的Python代码，亲测可用https://github.com/crazyxw/SlideCrack**
    * 在golang调用Python程序时，若该*.py文件中没有导入第三方依赖，可顺利被go调用。当导入三方依赖时，会报错，这种情况目前本人无法解决。
    * 由于本人是Windows系统，故将python代码打包成了*.exe可执行程序进行调用，测试后可行
  * **【已解决】滑块验证码获取需要一个zftsl的参数，猜测与时间戳和sessionid有关，目前无法得到该值，故无法拿到滑块验证码文本。**
    * 逆向小程序后，拿到相关的js文件，找到了zftsl参数的来源。
    * 在使用golang调用js时，出现了与调用Python一样的情况，即不能有三方依赖。作者通过将几个js文件函数拼接到了一个文件中-app.js。go调用该js后可得到zftsl参数。测试通过。
  * **【已解决】在拿到滑块验证码的Base64编码时，发现该编码太长导致[]byte接收不完全，因而无法复原验证码图片。**
    * 通过将resp.body写入到本地文件中，再用json解析本地文件得到两个图片的base64编码
    * 需要注意的是，在多协程环境下，会有大量的文件读写操作，可能会造成文件的覆盖。未来需要对文件名（如 file+协程ID）进行区分，在验证码识别完成后可不对文件进行删除操作，直接覆盖对应文件名
  * **【已解决】在代码运行过程中出现503异常，Degug时程序却运行正常，猜测是请求过快导致被服务禁止**
    * 通过在关键请求前后增加sleep时间，降低访问频率
    * 增加LimitRate限流，在实际测试中发现，当请求速率设置有1次/s时，有小概率被限流的风险。当频率为1次/2s时，不会被限流。因此暂时设置速率为1次/2s
  * **【已解决】增加多线程以提高运行效率**
    * 增加多线程访问时，出现验证码识别失败的问题。猜测使用同一session并发请求获取验证码图片时，会导致小程序服务端前一次请求的验证码结果失效，即多线程并发请求和验证验证码会出现混乱状况。
    * 为解决上述问题，作者采用加锁的方式。从验证码获取到识别完成这一流程整合成一个步骤对其加锁，当流程中有任一操作失败时就释放锁，把请求资源让给其他线程执行。测试可行。
* 开发进度
  * 开完发成。
* 项目结构说明：
  * 用户信息在config/conf.yaml中手动配置
  * 项目build后会生成可执行文件zmyy_seckill.exe，使用时勿修改文件名。(因路径使用了正则匹配。)
  * 本人已尽可能减少使用难度，但conf.yaml中的Cookie因技术原因无法解决，故需要使用者本人获取
    * 推荐在电脑上打开小程序，并使用fiddler抓包即可

