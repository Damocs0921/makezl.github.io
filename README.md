# makezl.github.io
磊子的iOS技术博客

使用JSPatch一些需要注意的地方
-------------

### 前言
每个App版本迭代，都需要一定时间周期开发下一个版本，有些公司二周或者一个月一个版本等，然而有人的地方就有江湖，有码农的地方就会存在Bug，但我们客户端不能做到服务器那样能及时更新代码，有时候遇到一个没测出来并且比较严重的Bug的时候，将会特别麻烦，虽然大淘宝有紧急发版的卖，但是动辄就好几千，感觉没必要，这个时候就要用到一些动态修复的引擎了! 

#### [JSPatch](https://github.com/bang590/JSPatch) 动态加载JS脚本来改变iOS应用程序, 苹果官方比较推荐的一种Patch
#### [WaxPatch](https://github.com/mmin18/WaxPatch) 动态加载lua脚本来改变你的iOS应用程序的行为.

这里我只讲解JSPatch最近遇到的一些语法问题，简介跟使用还请看[JSPatch基础用法](https://github.com/bang590/JSPatch/wiki/%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95)

>>> JSPatch提供的OC转JS网址[这里](http://bang590.github.io/JSPatchConvertor/)

JSPatch一般用法就是，通过转换打好补丁js，第一步通过`[JSPatch startEngine]`开启引擎后, 加载[JSPatch evaluateScript:js]js代码.

那么稍微打补丁不注意会造成崩溃, 以下说说会崩溃的点，以及怎样使用.

1.第一点: 数组取值
-----------
```
OC代码: SYAPIPostReplyEntity *replyEntity = self.post.info.diarys[indexPath.row];
// 通过网站转换过来的, 会造成崩溃的，是需要分割下的
JSPatch: var replyEntity = self.post().info().replies[indexPath().row](); 
// 下面才是正确的
var indexPathRow = indexPath.row();
var replies =  self.post().info().replies();
var replyEntity = replies.objectAtIndex(indexPathRow);

```

2.第二点: 字典取值 
-----------
```
OC代码: NSString *contentHtml = contentDict[@"html"];
// 通过网站转换过来的, 会造成崩溃的，是需要分割下的
JSPatch: var contentHtml = contentDict["html"]
// 下面才是正确的
var contentHtml = contentDict.valueForKey("html");

```

3.第三点: 判断自定义的类型
-----------
```
OC代码: if ([cell isKindOfClass:SYPostTopicCell.class]){}
// 通过网站转换过来的>> 以下会造成崩溃的，因为你需要引入require('SYPostTopicCell,...')
JSPatch: if (cell.isKindOfClass(SYPostTopicCell.class())) {}
// 下面才是正确的
require('SYPostTopicCell')
if (cell.isKindOfClass(SYPostTopicCell.class())) {}
```

4.第四点: 自定义枚举跟系统枚举
-----------
```
OC代码: 
假设已经定义好了这个枚举 typedef NS_ENUM(NSInteger, SYPostTopicStatus) {
SYPostTopicStatusHtml = 0,
SYPostTopicStatusProduct,
SYPostTopicStatusDiary,
};

JSPatch代码: if (self.status == SYPostTopicStatusHtml) {}

// 通过网站转换过来的>> 以下会造成崩溃的，因为JSPatch解析不了枚举
if (self.status == SYPostTopicStatusHtml) {}
// 下面才是正确的
if (self.status == 0) {}
```

5.第五点: 关于方法带下划线的，但是JSPatch自己解析方法名有_下划线可以不用管
-----------
```
OC代码: [self.imageView sd_setImageWithURL: placeholderImage:];

// 通过网站转换过来的>> 以下会造成崩溃的，因为方法带了_下划线的方法名, 开头要有双下划线
self.imageView.sd_setImageWithURL_placeholderImage("url",image);
// 下面才是正确的
self.imageView.sd__setImageWithURL_placeholderImage("url",image);
```

```
6.第六点: 如果有一点static变量是在打Patch的方法进行初始化的需要注意
OC:
if (!replyHeights) {
replyHeights = [NSMutableDictionary dictionary];
}

JSPatch: 也需要进行初始化，不然外部使用这个变量的时候，是nil;
var replyHeights;
if (!replyHeights) {
replyHeights = NSMutableDictionary.dictionary();
}
```

7.第七点: 如果你重写的方法，是之前类不存在的, 是打不上的, 打比方tableView 计算高度的方法你没写, 然后想打JSPatch，返回一定的高度，是打不上的，只能覆盖原有存在方法

8. 获取一个View的frame.size的时候尽量分割多个变量来取，不要一次性全部取完. like>> self.view().frame().size().height ...

9. 使用dispatch_after换成 dispatch_after(0.5, function() {}
......

当然还有一些现在也还没有很全的总结出来，也希望能多交流提供，在打补丁的路上互勉



总结
-----------

打JSPatch多，总归是我们程序员本身写代码质量的问题, 不管是从什么角度来说, 都是细节没注意到位, 我就是个写代码很快缺细节总是不到位的一个人, 缺点很明显, 我要努力去要求自己像处女座一样.

1. 一个方法的代码量尽量不超过100行, 尽可能的越清晰越短, 打Patch也不至于那么麻烦
2. 写代码仔细, 细心, 可以跟同事CodeReview
3. 1,2 做好了, 3点也就没了

因为很多方法都是纯手打, 英文错了望指点, 写的不到位存在问题的, 也希望各位多多指点一下
同时非常感谢[bang590](https://github.com/bang590/JSPatch)的衷心付出!

@weibo:http://weibo.com/makezl
@Github:https://github.com/makezl

我是一名96年程序员, 叫MakeZL简称ML, 多多关照喔~ 

@soyoung团队 同时感谢包容我经常出错的团体 Thanks!

