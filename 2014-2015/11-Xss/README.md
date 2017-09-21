## 0x01 JS能用来做什么

* 请求资源
* 提交表单
* 修改html
* 响应事件
---

## 0x02 一个含有js的html文件是怎样变成一个动态网页的

**浏览器解析方式** 

在HTML中有五类元素
* 语言的解析一般分为`词法分析（lexical analysis）`和`语法分析（Syntax analysis）`两个阶段，WebKit中的 html解析也不例外，本文主要讨论词法分析。
* 词法分析的任务是对输入字节流进行逐字扫描，根据构词规则识别单词和符号，分词。
* 在WebKit中，有两个类，同词法分析密切相关，它是`HTMLToken`和`HTMLTokenizer`类，可以简单将HTMLToken类理 解 为标记，HTMLTokenizer类理解为词法解析器。HTML词法解析的任务，就是将输入的字节流解析成一个个的`标记（HTMLToken）`，然后由 语法解析器进行下一步的分析。
* 在XML/HTML的文档解析中，token这个词经常用到，我将其理解为一个`有完整语义的单元（也就是分出来的 “词”）`，一个元素通常对应于3 个 token，一个是`元素的起始标签`，一个是`元素的结束标签`，一个是元素的`内容`，这点同DOM树是不一样的，在DOM树上，起始标签和结束标签对应于一个元素节点，而元素内容对应另一个节点。
* 除了`起始标签(StartTag)`、`结束标签(EndTag)`和`元素内容（Character）`，HTML标签还有`DOCTYPE（文档类 型）`,`Comment（注释）`，`Uninitialized（默认类型）`和`EndOfFile（文档结束）`等类型，参见HTMLToken.h中的 Type枚举

**在HTML中有五类元素：**  

1. **空元素(Void elements)**，如`<area>`,`<br>`,`<base>`等等
2. **原始文本元素(Raw text elements)**，有`<script>`和`<style>`
3.  **RCDATA元素(RCDATA elements)**，有`<textarea>`和`<title>`
4.  **外部元素(Foreign elements)**，例如`MathML命名空间`或者`SVG命名空间`的元素
5. **基本元素(Normal elements)**，即除了以上4种元素以外的元素

**五类元素的区别如下**

1. **空元素**，不能容纳任何内容（因为它们没有闭合标签，没有内容能够放在开始标签和闭合标签中间）。
2. **原始文本元素**，可以容纳文本。
3. **RCDATA元素**，可以容纳文本和字符引用。 （&#60）
4. **外部元素**，可以容纳文本、字符引用、CDATA段、其他元素和注释
5. **基本元素**，可以容纳文本、字符引用、其他元素和注释

>如果我们回头看HTML解析器的规则，其中有一种可以容纳字符引用的情况是“RCDATA状态中的字符引用”。 这意味着在`<textarea>`和`<title>`标签中的字符引用会被HTML解析器解码。这里要再提醒一次，在解析这些字符 引用的过程中不会进入“标签开始状态”。这样就可以解释问题5了。另外，对RCDATA有个特殊的情况。在浏 览器解析RCDATA元素的过程中，解析器会进入`RCDATA状态`。在这个状态中，如果遇到`<`字符，它会转 换到`RCDATA小于号状态`。如果`<`字符后没有紧跟着`/`和对应的标签名，解析器会转换回`RCDATA状 态`。这意味着在RCDATA元素标签的内容中（例如`<textarea>`或`<title>`的内容中），唯一能够被解析器认做是 标签的就是`</textarea>`或者`</title>`。当然，这要看开始标签是哪一个。因此，在`<textarea>`和`<title>”`的内容中不会创建标签，就不会有脚本能够执行。  

---
## 0x03 XSS是什么

恶意攻击者往Web页面里插入恶意html代码，当用户浏览该页之时，嵌入其中Web里面的html代码会被执行，从而达到恶意攻击用户的特殊目的.通过xss可以进行一系列的越权操作，或者窃取用户cookies从而登录用户账户。

**说白了就是 插js, html耍流氓**

---

## 0x04 XSS的分类

**反射型XSS：**

* 恶意代码通常存在于URL中 
* 需要用户去点击相应的链接才会触发，隐蔽性较差
* 而且，而且可能会被浏览器的XSS Filter干掉 
* 流程：输入--输出

**存储型XSS：**

* 恶意代码通常存在于数据库中
* 用户浏览被植入payload的“正常页面”时即可触发，隐蔽性较 强，成功率高，稳定性好。 
* 流程：输入--进入数据库--取出数据库--输出
---


## 0x05 XSS漏洞挖掘技巧

* 输入在标签间的情况：测试`<>`是否被过滤或转义，若无则直接`<img src=1 onerror=alert(1)>`
* 输入在script标签内：我们需要在保证内部JS语法正确的前提下，去插入我们的payload。 如果我们的输出在字符串内部，测试字符串能否被闭合。如果我 们无法闭合包裹字符串的引号，这个点就很难利用了。 可能的解决方案：可以控制两处输入且\可用、存在宽字节
* 输入在HTML属性内：首先查看属性是否有双引号包裹、没有则直接添加新的事件属性； 有双引号包裹则测试双引号是否可用，可用则闭合属性之后添加 新的事件属性； TIP：HTML的属性，如果被进行HTML实体编码(形如`&#039`;`&#x27`)，那么 HTML会对其进行自动解码，从而我们可以在属性里以HTML实体编 码的方式引入任意字符，从而方便我们在事件属性里以JS的方式构 造payload。
* 输出在JS中，空格被过滤：使用`/**/`代替空格。
* 输出在JS注释中： 设法插入换行符`%0A，`使其逃逸出来。
* 输入在JS字符串内： 可以利用JS的十六进制、八进制、unicode编码。
* 输入在src/href/action等属性内：可以利用`javascript:alert(1)`，以及 `data:text/html;base64; `加上base64编码后的HTML。
* onxxx事件的js脚本可以用html编码来绕过对某些关键字的过滤。
* Chrome下iframe自身的弹框姿势：  
```html
<iframe onload="alert(1)"></iframe>   
<iframe src="javascript:alert(2)"></iframe> 
<iframe src="data:text/html,<script>alert(3)</script>"></iframe>   
<iframe srcdoc="<script>alert(4)</script>"></iframe>
```
* 坑点之自带HtmlEncode(转义)功能的标签：   
```html
<textarea></textarea> 
<title></title> 
<iframe></iframe> 
<noscript></noscript> 
<noframes></noframes>
 <xmp></xmp>
 <plaintext></plaintext>
```
当我们的XSS payload位于这些标签中间时，并不会解析，除非我们把它们闭合掉。

---

## 0x06 XSS练习网站、资料

* [xss-quiz](http://xss-quiz.int21h.jp/)
* [alert(1) to win](https://alf.nu/alert1)
* [prompt(1) to win](http://prompt.ml/0)
* [html5 Security Cheatsheet](http://html5sec.org/)
* [XSS Cheatsheet](http://ssv.sebug.net/XSS_Cheat_Sheet)
* [OWASP XSS Filter Evasion Cheat Sheet](https://www.owasp.org/index.php/XSS_Filter_Evasion_Cheat_Sheet)

