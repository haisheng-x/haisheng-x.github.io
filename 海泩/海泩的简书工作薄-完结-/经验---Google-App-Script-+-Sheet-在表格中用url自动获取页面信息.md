上一篇 [Excel + Markdown 为简书文添加表格和批量链接](https://www.jianshu.com/p/91643b2d9f5b)发布后，很开心有几个人用起来了。昨日仙灵更是提出[改进版](https://www.jianshu.com/p/45ec0cd72b80)。

这一想法很赞，可以进一步提高效率，减少手动复制频率。不过我试了一下，用谷歌在线表格的时候不适用，因为复制到谷歌 sheet 不会自动生成超链接。

并且平日我的习惯是收录文章之后，直接在移动端复制文章链接到表格保存（如下图），后续在用电脑进行审文、校对、排期等操作。

![初始状态](https://upload-images.jianshu.io/upload_images/12120257-bdf3eced34d724db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但是仙灵给了我灵感（果然是·仙·灵·）。于是，产生一个想法，能不能在表格中写个函数**根据文章链接自动获取我需要的其他三个信息**呢？也就是作者名字、主页链接以及文章标题。

![想要的样子](https://upload-images.jianshu.io/upload_images/12120257-04bdbbea039d3e26.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

摸索一番之后，发现无法用表格函数直接实现，需要通过Google App Script写Javascript代码自定义函数才能达成目的。接下来展现操作过程。

---

##第一步、打开 Google App Script

打开谷歌 Sheet 之后，进入AppSheet。
> Tool > AppSheet > Create an app

![Tool > AppSheet > Create an app](https://upload-images.jianshu.io/upload_images/12120257-bbf335618232226d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---

##第二步、新建一个 Script

如下图所示，新建一个脚本。随后输入一个名字，复制下方代码就可以了。

![新建](https://upload-images.jianshu.io/upload_images/12120257-89892ba25fb286f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![命名](https://upload-images.jianshu.io/upload_images/12120257-35e646d85de44310.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---

##第三步、复制粘贴代码

将下方代码（三个函数）复制粘贴在上一步新建的脚本里面，覆盖原有的代码框架即可。

![默认新建脚本里的初始代码，可直接删除](https://upload-images.jianshu.io/upload_images/12120257-533a83eddf53d8e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面以从网页获取文章标题的代码为例稍加解释。

```
function title(input){ // 根据文章url获取文章标题
  var url = UrlFetchApp.fetch(input); //根据input（也就是单元格存储的链接）获取网页内容
  var html = url.getContentText("UTF-8"); //将网页内容存成字符放在html命名的一个地方

  var searchstring = '_1RuRku">'; //在网页内容中搜索引号内的字符串
  var index = html.search(searchstring); //字符串中第一个字符的索引（=门牌号）
  var pos = index + searchstring.length; //index加上字符串长度得到字符串后面第一个字母的索引
  var rate = html.substring(pos, pos+40); //从pos这个位置开始，取40个字符长的一部分
  var title = rate.split("<")[0]; //从取出来的rate里面截取<符号前面的内容就是文章的标题了
  return title; //把标题内容显示在单元格里面
}
```

上面这一部分代码其实就是在让电脑根据网页的链接，找到网页内容，从中找出来我需要的文章标题。需要注意的是其中用到的搜索参考节点```_1RuRku">```和```<```。我是怎么知道这两个东西中间就是文章标题的呢？这需要先打开一篇简书文章的网址，查看源代码，对照确认。

如下图所示，查看红剪头指向的方框内部会发现文章标题左右的内容，这就是我用来搜索的参考坐标。

![网页源代码对照](https://upload-images.jianshu.io/upload_images/12120257-9d9f51ca9d111907.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面给出获取作者名字及作者主页链接的代码。在返回作者主页链接时稍有不同，因为需要自己把网址补全。这里不展开来说。

```
function author(input){ // 根据文章url获取作者名字
  var url = UrlFetchApp.fetch(input);
  var html = url.getContentText("UTF-8");

  var searchstring = '_22gUMi">';
  var index = html.search(searchstring);
  var pos = index + searchstring.length;
  var rate = html.substring(pos, pos+40);
  var author = rate.split('<')[0];
  return author;
}

function author_url(input){ // 根据文章url获取作者个人主页链接
  var url = UrlFetchApp.fetch(input);
  var html = url.getContentText("UTF-8");

  var searchstring = 'qzhJKO" href="';
  var index = html.search(searchstring);
  var pos = index + searchstring.length;
  var rate = html.substring(pos, pos+40);
  var author_url = rate.split('"')[0];
  return "https://www.jianshu.com/"+author_url;
}
```

---

## 第四步、保存代码，返回原表格使用自定义函数

将上述三段代码保存之后，返回之前打开的Google Sheet。假如文章链接本来存放在D2单元格，在需要生成文章标题的位置输入```=title(D2)```，回车之后就会看到文章标题自己出来了。类似的，用```=author(D2)```，```=author_url(D2)```生成作者名字和作者主页链接。这是因为代码里面是这么给函数命名的。

![生成文章标题](https://upload-images.jianshu.io/upload_images/12120257-ea8a53c033476c8e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---

#完美收工

至此就实现了懒人高效生成表格内容，再衔接 [Excel + Markdown 为简书文添加表格和批量链接](https://www.jianshu.com/p/91643b2d9f5b)一起使用，又省了好多分钟呢[笑哭]。

>*你永远无法想象一个人为了偷懒几分钟花几十分钟去使劲扒拉的样子*

另外，可惜腾讯在线表格貌似不支持自定义函数功能。
