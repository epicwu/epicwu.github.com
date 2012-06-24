---
layout: post
title: 解决jslib-builder中波浪号文件名排序问题
category: problems
tags: 波浪号, 排序, PHP
---

在[jslib](http://github.com/elfjs/jslib)项目中我用到了包含波浪号的文件名，意图是区分命名空间中的实体对象和仅作为辅助执行过程的代码片段，所有以`~`开头的文件或文件夹都认为不是一个实体命名空间。在Windows资源管理器中以波浪线开头的文件排序是在字母之前的，但Linux或者程序下的默认排序是按字符ASCII码排的，而波浪线`~`的编码是`126(0x7e)`导致波浪线开头的文件都排在最后，而在[jslib-builder](http://github.com/elfjs/jslib-builder)打包器中有小部分虚体过程是无法设置依赖，但又需要优先于其他文件合并，这样波浪号开头的文件排序靠后就带来了问题，我必须在获取文件列表时将波浪号开头文件的顺序重排到字母之前。

解决的思路是通过自定义方法排序，在比较方法中将文件名中出现的`~`替换成一个比字母数字编码靠前的符号来计算，这样在获取文件列表时就可以得到波浪号在前的最终顺序了。

实施过程还算顺利，顺便记下几点PHP的Tips：

* PHP里类似JS可以通过自定义比较函数对数组排序的函数是`usort()`（如果要在参数中直接传匿名闭包函数需要PHP5.3以上支持）；
* 获取字符串中某个索引的字符可以简单的用大括号写下标获取`$str{N}`；
* 计算字符编码和通过编码转化为字符的两个对称函数`ord($char)`和`chr($ascii)`；
* PHP两个字符串比较可以直接用比较符号：`<`，`<=`，`==`，`>=`，`>`；

另外还顺便了解到波浪号的英文：`tilde`。

剩下上代码吧，其实很简单：

	/**
	 * 调整波浪号的字符顺序，以排序到字母之前
	 */
	function fix_tilde_code ($str) {
		return str_replace('~', '/', $str);
	}

	/**
	 * 加入波浪号修正顺序的字符串排序
	 */
	function fix_tilde_order (&$list) {
		usort($list, function ($a, $b) {
			return fix_tilde_code($a) <= fix_tilde_code($b) ? -1 : 1;
		});
	}

-EOF-
