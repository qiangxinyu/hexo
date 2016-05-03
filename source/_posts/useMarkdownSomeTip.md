title: 使用 markdown 的小tip
---

# 非常重要的一点  

<p > <font color = red size = 5> 文件夹 和 文件 的名字千万不要有汉字，24小时的血与泪告诉你们，千万不要有汉字，我就是因为有个空文件夹，名字叫 新建文件夹 因为是空的 我也没管它，结果 hexo 部署不上去了啊 ,在网上怎么都找不到解决的办法。 </font> </p>


### 字体 

设置了字体的颜色，必须在这段 添加上 p 标签， 否则 markdown 无法生成 p 标签，会影响到下面的布局。


```html
<p>
	<font color=#a2c700> JPEG </font>  
	是目前最常见的图片格式，它诞生于 1992 年，是一个很古老的格式。它只支持有损压缩，
	其压缩算法可以精确控制压缩比，以图像质量换得存储空间。由于它太过常见，
	以至于许多移动设备的 CPU 都支持针对它的硬编码与硬解码。
</p>
```


### 使用图片

要让图片可以点击，需要添加 html 的 a 标签

```html
<a href = "/images/handleImageSomTip/image_baseline.gif">
	<img src = "/images/handleImageSomTip/image_baseline.gif"
	 width = 160 height = 160 alt = "image_baseline.gif" />
</a>
```

### 生成目录
在内容前面加上这段代码


```javascript
<link rel="stylesheet" href="http://yandex.st/highlightjs/6.2/styles/googlecode.min.css">
<script src="http://code.jquery.com/jquery-1.7.2.min.js"></script>
<script src="http://yandex.st/highlightjs/6.2/highlight.min.js"></script>
<script>hljs.initHighlightingOnLoad();</script>
<script type="text/javascript">
 $(document).ready(function(){
      $("h2,h3,h4,h5,h6").each(function(i,item){
        var tag = $(item).get(0).localName;
        $(item).attr("id","wow"+i);
        $("#category").append('<a class="new'+tag+'" href="#wow'+i+'">'+$(this).text()+'</a></br>');
        $(".newh2").css("margin-left",0);
        $(".newh3").css("margin-left",20);
        $(".newh4").css("margin-left",40);
        $(".newh5").css("margin-left",60);
        $(".newh6").css("margin-left",80);
      });
 });
</script>
<div id="category"></div>
```

### 添加评论区

我使用了 `Disqus` ,在国内速度会慢一点，但是为了逼格，我还是用了它，国内可以用 [多说][多说]。

废话不多说，首先得去 [Disqus]Disqus] 注册一个账号,有一个邮箱就行，收到邮件后验证一下 

<a href = "/images/userMarkdownSomeTip/signUpDisqus.png"><img src = "/images/userMarkdownSomeTip/signUpDisqus.png" width = 394 height = 537 alt = "signUpDisqus.png"/></a>

登陆进来以后再右上角找到设置,点击 `Add Disqus To Site` 

<a href = "/images/userMarkdownSomeTip/addDisqusToSite.png"><img src = "/images/userMarkdownSomeTip/addDisqusToSite.png" width = 305 height = 315 alt = "addDisqusToSite.png"/></a>

进入后点击 `Start Using Engage` ,然后设置 `Site` 

<a href = "/images/userMarkdownSomeTip/setUpDisqusNewSite.png"><img src = "/images/userMarkdownSomeTip/setUpDisqusNewSite.png" width = 804 height = 524 alt = "setUpDisqusNewSite.png"/></a>

`siteName` 就填你的 `github.io` 地址，下面的URL会自动生成，选择一个类别，点击 Next，然后会有两个按钮 `My site is part of a larger organization (我的网站是一个更大的组织的一部分)` 和 `My site is just a personal site (我的网站只是个人网站)`, 然后填写2个问题，就进入了管理页面，

<a href = "/images/userMarkdownSomeTip/siteManager.png"><img src = "/images/userMarkdownSomeTip/siteManager.png" width = 1090 height = 250 alt = "siteManager.png"/></a>

点击 `General` 

<a href = "/images/userMarkdownSomeTip/settingGeneral.png"><img src = "/images/userMarkdownSomeTip/settingGeneral.png" width = 815 height = 834 alt = "settingGeneral.png"/></a>

`Shortname` 一会要用到，`Website Name` 会在评论区的顶部显示， `Website URL` 你博客的网址，必须填对。其他的设置不填也可以。

我使用的是 `hexo` 搭建的博客，进入到博客根目录，找到 `_config.yml` 文件，在里面添加

	# Disque
	disqus_shortname: 刚才的那个 Shortname
	
如果有这个字段，直接修改值就可以了。 

然后就可以提交到 github 刷新页面，评论框就出来啦。	

[多说]: http://duoshuo.com/
[Disqus]: https://disqus.com/