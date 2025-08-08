# This is a note to memorize the problems I met/solved/wait to solve
***Problems are common for all beginners, if what I once got across had an opportunity to be a guide for someone else on the same path, I would appreciate my minor contibution.***

如果有佬看到这个页面，看到我标记为 ***wait to solve*** 的问题，求你来告诉我解决方法🙏🙏

### 1. How to establish this websites (a notebook actually)
首先，迈出这个mkdocs的第一步得要感谢图灵2302的jmgg😋😋，可以去撅他网站然后催更 (dremig.top) 。在知道有这么个东西之后本人也是一步一步找着去搞，最后找到了一个比较好的[教程](https://blog.csdn.net/m0_62342492/article/details/140589266),别去问jmgg，他只会告诉你  
![](./problem/jmthreat2.png)  

**as well as**  

![](./problem/jmthreat1.png)

😭😭😭😭😭😭😭😭😭😭  
<del>plmm/pljj应该可以放心问</del>
不过很神奇的是，每次我去吵完他，然后按照之前的步骤重做一遍，问题往往解决了（笑）

### 2. Use Markdown successfully! But some discrepancy between local and remote display
编译器问题，例如在vscode里面写的删除线可以用两左右两个波浪线的形式~~like this~~,但是我push 到网页上就显示不出来  
**解决办法：**查询发现是由于这样的语法不规范，<del>扯淡</del>，所以要使用两边《del》 《/del》的形式

### 3. I wish to write my website on my windowPC(linux inside in fact),and upload documents from there ,but failed ***(wait to solve)***

### 4. LaTex failed to display normally !

具体问题描述：Latex可以让Markdown在描述数学公式符号变得美观，我在写Data Structure and algorithms 的时候就用了，但是通过mkdocs gh-pages，弄到网页上就无法显示😠。

查询后发现是由于GitHub pages 在识别数学语言出问题（），<del>蛮常见的，所以不怪我</del>寻求到了如下解决办法：[在mkdocs上使用mathjax](https://squidfunk.github.io/mkdocs-material/reference/math/?h=mathjax#mathjax-mkdocsyml),把一段神奇小代码放到mkdocs.yml里面就好啦
   