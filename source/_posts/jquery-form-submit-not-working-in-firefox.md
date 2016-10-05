title: jQuery使用form.submit在Firefox和IE下无效
date: 2015-10-27 22:01:38
category: 
	- JavaScript
	- 前端
tags:
	- js
	- jquery
	- formsubmit
---
jQuery中的form.submit在Firefox和IE下无效。

<!--more-->

今天碰到一个很诡异的问题，jQuery中使用自定义form 提交时，在Chrome下可以正常使用，FF和ie下却无法使用。

代码

~~~
function downBills() {	
	var d=$("#hidoribills").text();
	var fm=$("<form action='agt_fltbillfoc_down.do' method='post' target='_blank'></form>");
	var ipt=$("<textarea name='oristr'></textarea>");
	ipt.val(d||"...");
	fm.append(ipt);
	//fm.appendTo("body");
	fm.submit();	
}
~~~
原因很简单，代码中定义的 `fm` 是jQuery 对象，不是 DOM 元素，所以无法执行 submit方法。

解决方法也很简单，添加一句代码

~~~
fm.appendTo("body");
~~~


