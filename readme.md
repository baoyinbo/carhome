##整理了一些这段时间做爬虫的思路：

##需求：
从首页（http://www.autohome.com.cn/car/?pvareaid=101452） 开始，一共1962款车系需要全部爬取，
然后需要爬取三级目录，例如 阿尔法罗密欧 - 阿尔法罗密欧 - MiTo这种格式，爬取到[MiTo](http://www.autohome.com.cn/715/#levelsource=000000000_0&pvareaid=101594)
的详情页，需要获取的数据包括新车指导价，二手车价格等等，然后进入[参数配置](http://car.autohome.com.cn/config/series/2097.html)
的页面，将所有车款的信息爬取完毕。在车系首页会有两种情况，参数配置可点击或者不可点击,同时需要对停售的链接进行单独处理，或者停售车型的数据。关于详细车款组的信息
一定要获取全部的数据，避免遗漏。所以一条字段大概有300条左右，全部的数据大概有2w多条，全部代码由我一人完成。

##涉及的知识点：

1.关于Ajax请求：

①首页我们一直下拉才能看到获取的数据，很明显是Ajax请求，只要看XHR里的请求对其获取即可。

②我们在审查元素里看到的，不一定是真实的。可能是通过其他方式加载的，所以用正常的方式获取不到很正常，
这时候我们就要去network里刷新，找出是哪个请求获取的数据，然后请求这个链接，将其获取的数据进行解析即可。

2.关于JSON解析的技巧：

在获取车款组信息的时候，可以看到数据都是以JSON的方式加载的，keyLink 中观察到所有链接这个参数都相同，
判断可知这代表全部的数据，我们以这个为标准，进行解析。
解析JSON可以使用fastJSON，一个技巧就是将JSON平铺成guava中的table，非常方便使用。

3.正则表达式的技巧：

这个一般来说需要多加使用才能熟练，比如在何处截断，还有<![CDATA]>中的数据必须用正则表达式进行处理
才可以获取，需要注意。

4.关于模拟浏览器的行为：

这是下下策，效率非常低，如非必要，最好不好随意使用，最好做法是直接去network中获取请求来源对其进行解析
这里推荐httpclient和[Retrofit](http://square.github.io/retrofit/)这两个库进行解析，非常方便。

5.关于post请求：

要注意提交的参数，这里推荐DHC 一个chrome插件，可以方便的发送post请求，添加提交的表单参数，进行测试。
或者使用curl命令行进行试验。


6.关于模拟登陆：

其实模拟登陆就是提交表单，只要抓包找出需要的参数即可，如果需要验证码的话可以请求验证码的图片，然后人工打码。
或者携带登录之后的cookie也可。

7.分析javascript：

做爬虫一定要会javascript，如果遇到参数里携带不好识别的，这时候要分析页面的javascript源码，比如很多链接里都会携带一个随机数，
就是分析js找到的。

8.关于效率：

我这里采用了并行流，最终爬取的效率是20分钟内爬取完毕。

9.关于URL去重：

按理说这个需求下的URL是不会重复的，所以没有做去重工作，如果需要去重的话可以用HashSet，数据量大采用bloomfilter或者redis里的set

10.关于错误重试机制：

可以将请求出错的URL放入队列末尾重试，也可以像我代码中一样放入文本中。有时候请求过多过于频繁容易被封掉，也容易出错，
所以我们可以将请求的URL存入文本，然后在后续步骤中读入，这样效率会更高一些，也避免对网站请求过大对服务器造成压力。

11.关于初衷：

本意是用springboot做成一个web项目，顺便学习下kafka，目前精力要转入搜索的部分，暂时搁置，有空继续更新。所有代码均为我一人完成。



