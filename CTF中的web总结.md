## CTF中的web总结
### 基础知识
Mysql,Oracle,Mssql,Access等SQL语法的细微区别
熟悉和掌握手工sql注入所使用的语句
学会PHP,Java,JavaScript,Python,Python,Lua等编程中的一种或多种
学会配置Tomcat,IIS,Apache,Nginx等web服务器，了解配置信息及一些漏洞。

### web基础简介
一、爆破，包括md5、爆破随机数、验证码识别等  
二、绕WAF，包括花式绕Mysql、绕文件读取关键词检测之类拦截等  
三、花式玩弄几个PHP特性，包括弱类型，strpos和===，反序列化+destruct、\0截断、iconv截断等  
四、密码题，包括hash长度扩展、异或、移位加密各种变形、32位随机数过小、CBC翻转攻击等  
五、各种找源码技巧，包括git、svn、xxx.php.swp、ww w.(zip|tar.gz|rar|7z)、xxx.php.bak、.git、.DS_Store  
六、文件上传，包括花式文件后缀 .php345 .inc .phtml .phpt .phps、各种文件内容检测<?php <? <% <\script language=php>、花式解析漏洞  
七、Mysql类型差异，包括和PHP弱类型类似的特性,0x、0b、0e之类，varchar和integer相互转换,非strict模式截断等  
八、open_basedir、disable_functions花式绕过技巧，包括dl、mail、imagick、bash漏洞、DirectoryIterator及各种二进制选手插足的方法  
九、条件竞争，包括竞争删除前生成shell、竞争数据库无锁多扣钱  
十、社工，包括花式查社工库、微博、QQ签名、whois  
十一、windows特性，包括文件名、IIS解析漏洞、NTFS文件系统通配符、::$DATA，冒号截断  
十二、SSRF，包括花式探测端口，302跳转、花式协议利用、gophar直接取shell等  
十三、XSS，各种浏览器auditor绕过、富文本过滤黑白名单绕过、flash xss、CSP绕过  
十四、XXE，各种XML存在地方（rss/word/流媒体）、各种XXE利用方法（SSRF、文件读取）  
十五、协议，花式IP伪造 X-Forwarded-For/X-Client-IP/X-Real-IP/CDN-Src-IP、花式改UA，花式藏FLAG、花式分析数据包  
### web总结
#### 源代码
* robot.txt
* vim_swap_file
* -.pyc
* .DS_Store
* .git
* .svn
* bak file
* comment


为什么找源码是CTF比赛必备技巧？
《出题人心理学》
	代码审计类题目
    获取源码的过程也是考点之一
    题目脑洞太大，不得不给源码
哪些极其可能存在源码泄露的题（套）目（路）
	一个登陆框
    页面只有一两行字
    页面中包含Powered by XXX

#### waf
* 整数溢出过waf
https://github.com/Bypass007/vuln/blob/master/OpenResty/OpenResty%20uri%E5%8F%82%E6%95%B0%E6%BA%A2%E5%87%BA%E6%BC%8F%E6%B4%9E.md
* 字符替换空型waf
绕过:ununionion、UniON
* 特殊字符（串）拦截型waf
绕过：&&、Mysql注释符:/**/,/\*!条件注释*,--,;%00

#### CTF Trick
PHP弱类型
序列化/反序列化
XSS+CSRF+SSRF+RCE+Upload
源码审计
##### PHP弱类型
第一层
var_dump("0e321" == "00e123");//bool(true)
只要md5是0e开头。后面纯数字的情况才会把字符串转成科学计数法，0的N次方=0
md5('240610708') //0e462097431906509019562988736854
md5('QNKCDZO') //0e830400451993494058024219903391
第二层
s1[]=1&s2[]=2
第三层
```
<!--
	if((string)$_POST['param1']!==(string)$_POST['param2'] && md5($_POST['param1'])===md5($_POST['param2'])){
			die("success!);
		}
-->
```
使用强字符串转换，听说是去年的BKPCTF改的一题（没做过）
传URLencode后的二进制,payload:
```
Param1=%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%00%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%55%5d%83%60%fb%5f%07%fe%a2
Param2=%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%02%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%d5%5d%83%60%fb%5f%07%fe%a2
```
##### 序列化/反序列化
serialize($data)/unserialize($data)
preg_match( "/^{$token}:[0-9]+:/s", $data );正则绕过+8=8
##### XSS
1. 反射型XSS
2. 存储型XSS
3. Dom-XSS
4. 跨域问题
5. 同源

##### CSRF
html css flash
#### SQLi

#### 文件包含和文件上传
伪协议
* php://filter/convert.base64-encode/resource=index.php
* php://input
* zip://archive.zip#dir/file.txt
* zlib://
* ftp://
* data:image/png;base64,PD9waH==

花式文件后缀
.php345 .inc .phtml .phpt .phps
各种文件内容检测<?php <? <% <\script language=php> <?=
花式解析漏洞.php.jpg
各种校验
.htaccess文件
#### 危险函数
##### PHP代码执行相关函数
eval & assert & preg_replace & call_user_func & create_function & array_map
###### eval()函数——把字符串作为PHP代码执行
###### assert()函数——检查一个断言是否为 FALSE。（把字符串 $assertion 作为PHP代码执行）
###### preg_replace()函数——/e 修正符使 preg_replace() 将 replacement 参数当作 PHP 代码
```
preg_replace("/test/e",$_GET["h"],"jutst test"); 
```
7.0.0不再支持 /e修饰符。 请用 preg_replace_callback() 代替。
5.5.0 /e 修饰符已经被弃用了。使用 preg_replace_callback() 代替。
###### call_user_func()函数
```
mixed call_user_func ( callable $callback [, mixed $parameter [, mixed $... ]] )
```
第一个参数 callback 是被调用的回调函数，其余参数是回调函数的参数。 传入call_user_func()的参数不能为引用传递。
```
<?php
	call_user_func($_GET['hello'],$_GET['world']);
?>
```
http:127.0.0.1:/index.php?hello=assert&world=phpinfo()
###### call_user_func_array()函数
```
mixed call_user_func_array ( callable $callback , array $param_arr )
```
把第一个参数作为回调函数（callback）调用，把参数数组作（param_arr）为回调函数的的参数传入。
```
<?php
	call_user_func_array($_GET['hello'],$_GET['world']);
?>
```
http:127.0.0.1:/index.php?hello=assert&world[]=phpinfo()
###### create_function()函数——创建一个匿名函数
创建一个匿名函数，并返回独一无二的函数名。
```
<?php
//index.php?id=2;}phpinfo();//
$id=$_GET['id'];
$str2='echo  '.$a.'test'.$id.";";
echo $str2;
echo "<br/>";
echo "==============================";
echo "<br/>";
$f1 = create_function('$a',$str2);
echo "<br/>";
echo "==============================";
?>
```
http://127.0.0.1/index.php?id=2;}phpinfo();//
###### array_map()
```
array array_map ( callable $callback , array $array1 [, array $... ] )
```
作用是为数组的每个元素应用回调函数 。其返回值为数组，是为 array1 每个元素应用 callback函数之后的数组。 callback 函数形参的数量和传给 array_map() 数组数量，两者必须一样。
```
<?php
	$array = array(0,1,2,3,4,5);
	array_map($_GET['chybeta'],$array);
?>
```
http://127.0.0.1/index.php?chybeta=phpinfo
##### 系统命令执行相关函数
system & passthru
###### system()
system("whoami");
###### passthru()
passthru("whoami");
###### exec()
echo exec("whoami");
###### pcntl_exec()
pcntl_exec ( "/bin/bash" , array("whoami"));
###### shell_exec()
echo shell_exec("whoami");
###### popen()
打开一个指向进程的管道，该进程由派生给定的 command 命令执行而产生。 后面的mode，当为 ‘r’，返回的文件指针等于命令的 STDOUT，当为 ‘w’，返回的文件指针等于命令的 STDIN。
$handle = popen("/bin/ls", "r");
###### proc_open()
cmd是要执行的命令，其余见[文档](http://php.net/manual/zh/function.proc-open.php)
###### \`(反单引号)
在php中称之为__执行运算符__，__PHP 将尝试将反引号中的内容作为 shell 命令来执行__，并将其输出信息返回（即，可以赋给一个变量而不是简单地丢弃到标准输出，使用反引号运算符“\`”的效果与函数 shell_exec() 相同。
```
<?php echo `whoami`;?>
```
###### ob_start()
此函数将打开输出缓冲。当输出缓冲激活后，脚本将不会输出内容（除http标头外），相反需要输出的内容被存储在内部缓冲区中。想要输出存储在内部缓冲区中的内容，可以使用 ob_end_flush() 函数。

可选参数 output_callback 函数可以被指定。 此函数把一个字符串当作参数并返回一个字符串。 当输出缓冲区被( ob_flush(), ob_clean() 或者相似的函数)冲刷（送出）或者被清洗的时候；或者在请求结束之际输出缓冲区内容被冲刷到浏览器的时候该函数将会被调用。 当调用 output_callback 时，它将收到输出缓冲区的内容作为参数 并预期返回一个新的输出缓冲区作为结果，这个新返回的输出缓冲区内容将被送到浏览器。

下面的代码，由于调用了ob_end_flush()，所以会调用ob_start(cmd)中的cmd，把我们输入的_GET[a]作为cmd的参数。
```
<?php
	$cmd = 'system';
	ob_start($cmd);
	echo "$_GET[a]";
	ob_end_flush();
?>
```
http://127.0.0.1/index.php?a=whoami
##### php mail()
[mail 文档](http://php.net/manual/zh/function.mail.php)
```
bool mail (
	string $to ,
	string $subject ,
	string $message [,
	string $additional_headers [,
	string $additional_parameters ]]
)
```
要使用mail()函数，需要配置对应的服务器等，在php.ini中有两个选项：

* 配置SMTP服务器的主机名和端口
* 配置PHP用作邮件传输代理（MTA）的文件路径

当PHP配置了第二个选项时，对该mail()函数的调用将导致执行配置对MTA程序。虽然PHP内部使用escapeshellcmd()用于程序调用，防止新的shell命令注入，但第5个参数$additional_parameters中mail()允许添加的新程序。因此，攻击者可以附加程序标志，在某些MTA中可以创建具有用户控制内容的文件。

实例：[多个PHP mail函数引发的命令执行漏洞分析](http://blog.nsfocus.net/tag/php-mail%E5%87%BD%E6%95%B0%E5%BC%95%E5%8F%91%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C-%E6%BC%8F%E6%B4%9E/)
参考：[为什么mail()函数在PHP中是危险的](http://www.91ri.org/17039.html)
##### LD_PRELOAD绕过
实例：[第十届全国大学生信息安全技能赛PHP execise](https://bbs.ichunqiu.com/forum.php?mod=viewthread&tid=25351&extra=page%3D1%26filter%3Dtypeid%26typeid%3D146)
参考: [利用环境变量LD_PRELOAD来绕过php disable_function执行系统命令](http://wooyun.jozxing.cc/static/drops/tips-16054.html)

### 参考链接
CTF比赛总是输？你还差点Tricks!https://www.leavesongs.com/SHARE/some-ctf-tricks-ppt.html
