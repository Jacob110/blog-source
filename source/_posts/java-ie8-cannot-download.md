title: Java Web 应用在IE8下无法下载文件的问题
date: 2015-10-13 21:26:07
category: Java Web
tags: 
	- java
	- web开发
	- ie8
---

在Web应用中经常需要下载文件功能，Excel/TXT/Zip 等等，一般都直接用 `HttpServletResponse` 输出了，一直也没什么问题。最近突然有用户反馈在 IE8下无法下载，提示：Internet Explorer无法打开该Internet站点.请求的站点不可用,或找不到.请以后再试.
<!--more-->
提示错误如下图，看着都心烦啊，
![](http://7xngxf.com1.z0.glb.clouddn.com/down-001.png)

开始找问题，原始的代码如下

~~~
public static void OutputZip(HttpServletResponse response,String localZipFile) throws IOException {
    File f = new File(localZipFile);
    if (!f.exists()) {
        return;
    }
    String downloadFileName = f.getName();
    f = null;
    response.setContentType("application/x-zip-compressed");
    response.setHeader("Location", downloadFileName);
    response.setHeader("Cache-Control", "max-age=-1");
    response.setHeader("Content-Disposition", "attachment; filename=" + downloadFileName);
    OutputStream outputStream = response.getOutputStream();
    InputStream inputStream = new FileInputStream(localZipFile);
    response.setContentLength(inputStream.available());
    byte[] buffer = new byte[1024];
    int i = -1;
    while ((i = inputStream.read(buffer)) != -1) {
        outputStream.write(buffer, 0, i);
    }
    outputStream.flush();
    outputStream.close();
    inputStream.close();
    outputStream = null;
}
~~~

调了半天，发现本地开发环境和预发布环境都没有问题，当然其他浏览器Chrome和FF下也没有问题。一发布到生产环境就出错了，捉急啊。
Google了半天，试了半天也不行，最后猜测是 `response` 输出时设置的问题，就是下面这几行代码：

~~~
response.setContentType("application/x-zip-compressed");        response.setHeader("Location", downloadFileName);
response.setHeader("Cache-Control", "max-age=-1");
response.setHeader("Content-Disposition", "attachment; filename=" + downloadFileName);
~~~

于是乎把 `contentType` 的设置去掉了，也就是这行

~~~
response.setContentType("application/x-zip-compressed"); 
~~~

结果居然对了,正常显示是这样的

![](http://7xngxf.com1.z0.glb.clouddn.com/down-002.png)

真是坑爹啊，调了一下午啊啊,居然就注释掉一行代码就好了，擦泪。

问题虽然解决，但是具体的原因不知道是，为啥这么改也不知道，求大神赐教!

相关类似的其他问题
[IE cannot download files over SSL served by WebSphere](http://stackoverflow.com/questions/6573524/ie-cannot-download-files-over-ssl-served-by-websphere)

[IE : Unable to download * from *. Unable to open this Internet site. The requested site is either unavailable or cannot be found](http://stackoverflow.com/questions/3415370/ie-unable-to-download-from-unable-to-open-this-internet-site-the-request?rq=1)

[Internet Explorer无法打开该Internet站点.请求的站点不可用,或找不到.请以后再试.](http://jerval.iteye.com/blog/2037062)

