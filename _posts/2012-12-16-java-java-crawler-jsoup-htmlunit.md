---
layout: post
title:  "工具 - 使用jsoup和htmlunit抓取和解析网页"
desc: "工具 - 使用jsoup和htmlunit抓取和解析网页"
keywords: "工具 - 使用jsoup和htmlunit抓取和解析网页,jsoup,htmlunit,kinglyjn"
date: 2012-12-16
categories: [Java]
tags: [java,jsoup,htmlunit]
icon: fa-coffee
---

### Jsoup的不足之处：

说到网页信息抓取，相信 [Jsoup](http://www.open-open.com/jsoup) 基本是首选的工具，完全的类JQuery操作，让人感觉很舒服。但java爬虫的一般思路是使用 [HtmlUnit](http://htmlunit.sourceforge.net/) 等第三方工具抓取原网页，再使用jsoup等解析工具解析源网页。原因在于jsoup在获取原网页上存在一些不足，测试如下：<br>

1、首先新建一个页面

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>  
    <head>  
        <title>main.html</title>  
        <meta http-equiv="keywords" content="keyword1,keyword2,keyword3">  
        <meta http-equiv="description" content="this is my page">  
        <meta http-equiv="content-type" content="text/html; charset=UTF-8">  
    <style type="text/css">  
        a {  
            line-height: 30px;  
            margin: 20px;  
        }  
    </style>  
    <!--<link rel="stylesheet" type="text/css" href="./styles.css">-->  
    <script type="text/javascript">  

	var datas = [ {  
	    href : "http://news.qq.com/a/20140416/017800.htm",  
	    title : "高校一保安长相酷似作家莫言"  
	}, {  
	    href : "http://news.qq.com/a/20140416/015167.htm",  
	    title : "男子单臂托举悬空女半小时"  
	}, {  
	    href : "http://news.qq.com/a/20140416/013808.htm",  
	    title : "女子上门讨房租遭强奸拍裸照"  
	}, {  
	    href : "http://news.qq.com/a/20140416/016805.htm",  
	    title : "澳洲骆驼爱喝冰镇啤酒解暑"  
	} ];  
	  
	window.onload = function() {  
	    var infos = document.getElementById("infos");  
	    for( var i = 0 ; i < datas.length ; i++) {  
            var a = document.createElement("a");  
            a.href = datas[i].href ;  
            a.innerText = datas[i].title;  
            infos.appendChild(a);     
            infos.appendChild(document.createElement("br"))  
	    }  
	}  
	</script>  
    </head>  
  
    <body>  
        Hello Main HttpUnit!  
        <br>  
  
        <div id="infos"  
            style="width: 60%; border: 1px solid green; border-radius: 10px; margin: 0 auto;">  
        </div>  
  
    </body>  
</html> 
```
如果你看到这样的页面，你会觉得拿Jsoup来抓取，简直就是呵呵，小菜一叠，于是我们写了这样的代码：

```java
@Test  
public void testUserJsoup() {  
	try {  
	    Document doc = Jsoup.connect(  
	            "http://localhost:8080/strurts2fileupload/main.html")  
	            .timeout(5000).get();  
	    Elements links = doc.body().getElementsByTag("a");  
	    for (Element link : links) {  
	        System.out.println(link.text() + " " + link.attr("href"));  
	    }  
	} catch (IOException e) {  
	    e.printStackTrace();  
	} 
}
```

你会觉得就这几行代码，轻轻松松搞定，快快乐乐下班。于是运行发现，其实什么的抓取不到。于是我们再回到页面，打开页面源代码，也就是上面的HTML代码，你恍然大悟，我靠，body里面根本没有数据，难怪抓不到。这就是Jsoup的不足，如果Jsoup去抓取的页面的数据，全都是页面加载完成后，ajax获取形成的，是抓取不到的。<br>

推荐另一个开源项目 - HttpUnit，看名字是用于测试的，但是用来抓取数据也不错。HttpUnit其实就相当于一个没有UI的浏览器，它可以让页面上的js执行完成后，再抓取信息。测试代码如下：<br>

```java
import java.io.Serializable;
import java.util.Set;
import com.gargoylesoftware.htmlunit.WebResponse;
import com.gargoylesoftware.htmlunit.util.Cookie;
public class SerializedAllIn implements Serializable {
	private static final long serialVersionUID = 1L;
	private WebResponse webResponse;
	private Set<Cookie> cookies;
	private String imageUrl;
	
	public WebResponse getWebResponse() {
		return webResponse;
	}
	public void setWebResponse(WebResponse webResponse) {
		this.webResponse = webResponse;
	}
	public Set<Cookie> getCookies() {
		return cookies;
	}
	public void setCookies(Set<Cookie> cookies) {
		this.cookies = cookies;
	}
	public String getImageUrl() {
		return imageUrl;
	}
	public void setImageUrl(String imageUrl) {
		this.imageUrl = imageUrl;
	}
}

public class WebCrawler {
	private static final Logger log = LoggerFactory.getLogger(WebCrawler.class);
	private WebCrawler instance = new WebCrawler();
	private static String alertMsg = "";
	public static final int DEFAULT_PAGE_TIME_OUT = 20000; //ms
	public static final int DEFAULT_JS_TIME_OUT = 5000;
	
	public static WebCrawler getInstance() {
		return instance;
	}

	public synchronized static String getAlertMsg() {
		String msg = alertMsg;
		alertMsg = "";
		return msg;
		
	}
	public WebClient getWebClient() {
		if (webClient == null) { 
			webClient = new WebClient(BrowserVersion.CHROME);
			webClient.setRefreshHandler(new ThreadedRefreshHandler());
			webClient.getOptions().setCssEnabled(false); //禁用css支持
			webClient.getOptions().setJavaScriptEnabled(true); //启用JS解释器，默认为true
			webClient.getOptions().setThrowExceptionOnScriptError(false); //js运行错误时，是否抛出异常 
			webClient.getOptions().setThrowExceptionOnFailingStatusCode(false);
			webClient.getOptions().setRedirectEnabled(true);
			//15->60  //设置连接超时时间 ，这里是10S。如果为0，则无限期等待
			webClient.getOptions().setTimeout(DEFAULT_PAGE_TIME_OUT);   
			//5s      //设置JS后台等待执行时间
			webClient.waitForBackgroundJavaScript(DEFAULT_JS_TIME_OUT); 
			webClient.setAjaxController(new NicelyResynchronizingAjaxController()); 
			webClient.getOptions().setUseInsecureSSL(true); //
			//webClient.getJavaScriptEngine()
			//.getContextFactory().enterContext().setOptimizationLevel(9);
			webClient.setAlertHandler(new AlertHandler() {
				@Override
				public void handleAlert(Page page, String message) {
					alertMsg = message;
				}
			});
			webClient.setJavaScriptErrorListener(new JavaScriptErrorListener() {
				@Override
				public void scriptException(HtmlPage htmlPage,
						ScriptException scriptException) {
					log.info(htmlPage.getUrl().toString()
					    +"---scriptException----"
					    +scriptException.getMessage());
				}

				@Override
				public void timeoutError(HtmlPage htmlPage, long allowedTime,
						long executionTime) { 
					log.info(htmlPage.getUrl().toString()
					    +"---timeoutError----"+allowedTime
					    +"-----executionTime----"+executionTime);
				}

				@Override
				public void malformedScriptURL(HtmlPage htmlPage, String url,
						MalformedURLException malformedURLException) {
					log.info(htmlPage.getUrl().toString()
					    +"---malformedScriptURL----"
					    +url+"-----executionTime----"
					    +malformedURLException.getMessage());
				}

				@Override
				public void loadScriptError(HtmlPage htmlPage, 
					    URL scriptUrl, Exception exception) {
					log.info(htmlPage.getUrl().toString()
					    +"---loadScriptError----"+scriptUrl.toString()
					    +"-----executionTime----"+exception.getMessage());
				}
			}); 
		}
		return webClient;
	}

	//清除webClient之前的cookie
	@SuppressWarnings({ "deprecation" } )
	public static void clearCookies(WebClient webClient, URL url) throws MalformedURLException {
		CookieManager manager = webClient.getCookieManager();
		Set<Cookie> cookies = manager.getCookies(url);
		Set<String> cookieNames = new HashSet<String>();
		for (Cookie cookie : cookies) {
			cookieNames.add(cookie.getName());
		}
		for (String cookieName : cookieNames) {
			Cookie cookie = manager.getCookie(cookieName);
			manager.removeCookie(cookie);
		}
	}
	//给webClient增加cookie
	public static void addCookies(WebClient webClient, Set<Cookie> cookies) {
		for (Cookie cookie : cookies) {
			webClient.getCookieManager().addCookie(cookie);
		}
	}

	/*
	* 测试方法
	*/
	public static void main(Srting[] args) throws Exception {
		WebClient webClient = WebCrawler.getInstance().getWebClient();

		/**
		*  ...
		*  序列化操作
		* 
		* 【注一】序列化和反序列化通常用在一次爬取过程中含有图片验证码的情况，当图片验证码很复杂，ocr技术不能有效识别
		*  图片时,这时候不得不将一次爬取过程人为的划分成两次，第一次访问的目的在于获取图片验证码（将保存的图片验证码路径、
		*  页面cookie和页面信息等保存到序列化文件中），第二次访问时人工输入验证码，并携带验证码参数再次请求抓取数据。
		* 
		* 【注二】如果当图片验证码很复杂很简单，ocr技术（如第三方超级鹰）能够有效识别该验证码图片，则可以将两次访问合并为
		*  一个接口，方便开发和使用。
		*/
		String imageName = UUID.randomUUID() + ".jpg";
		File codeImageFile = new File(StormTopologyConfig.getNfs_filepath() + "/" + imageName); //

		HtmlImage image = searchPage.getFirstByXPath(webParam.getCodeImageId());
		//获取图片的方式还可以 InputStream is = page.getWebResponse().getContentAsStream();
		if (image!=null && StringUtils.isEmpty(image.getAttribute("src")) {
			image.click(); //点击换一张图片
			image.saveAs(codeImageFile);
			//保存图片验证码地址，供验证码手动输入完成后，下次访问使用
			serializedAllIn.setImageUrl("http://" 
			    + StormTopologyConfig.getNfs_nginx_server() 
			    + "/" + imageName);
		}
		//
		Set<Cookie> cookies = webClient.getCookieManager().getCookies(new URL(searchPageUrl));
		serializedAllIn.setCookies(cookies);
		//
		WebResponse webResponse = searchPage.getWebResponse();
		serializedAllIn.setWebResponse(webResponse);
		//
		String serializedAllInFileName = UUID.randomUUID().toString() 
		    + StatusCodeDef.SERIALIZED_FILE_SUFFIX;
		File serializedAllInFile = new File(StormTopologyConfig.getNfs_filepath() 
		    + "/" + serializedAllInFileName);
		ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(serializedAllInFile));
		oos.writeObject(serializedAllIn);
		oos.close();
		//...


		/**
		* 反序列化操作，webParam是传入对象，携带再次请求的一些信息
		*/
		serializedAllInFileName = webParam.getSerializedFileName();
		serializedAllInFile = new File(StormTopologyConfig.getNfs_filepath() 
		    + "/" + serializedAllInFileName);
		ObjectInputStream ois = new ObjectInputStream(new FileInputStream(serializedAllInFile));
		SerializedAllIn serializedAllIn = (SerializedAllIn) ois.readObject();
		ois.close();


		/**
		* 使用反序列化信息恢复原始页面(searchPage)，并回填验证码和搜索关键字，准备发送请求
		*/
		Set<Cookie> cookies = serializedAllIn.getCookies();
		clearCookies(webClient, new URL(searchPageUrl));
		addCookies(webClient, cookies);

		WebResponse webResponse = serializedAllIn.getWebResponse();
		PageCreator pageCreator = new DefaultPageCreator();
		searchPage = (HtmlPage) pageCreator.createPage(webResponse, 
			webClient.openWindow(new URL(searchPageUrl), "windowName"));

		Map<String, String> params = webParam.getParams();
		String keywordXpath = params.get("keywordXpath");
		String keyword = params.get("keyword").trim(); 
		String imagecodeXpath = params.get("imagecodeXpath");
		String imagecode = params.get("imagecode"); 
		String loginBtnXpath = params.get("loginBtnXpath");
		
		//点击提交按钮发送请求
		/*
		//回填关键字
		HtmlInput inputKeyword = 
		//  (HtmlInput)searchPage.getFirstByXPath(keywordXpath); //querySelector("#keyword");
		inputKeyword.reset();
		inputKeyword.setAttribute("value", keyword);
		//回填图片验证码
		HtmlInput inputImagecode = (HtmlInput)searchPage.getFirstByXPath(imagecodeXpath);
		inputImagecode.reset();
		inputImagecode.setValueAttribute(imagecode);
		//获取目标列表页
		HtmlElement loginButton = (HtmlElement) searchPage.getFirstByXPath(loginBtnXpath);
		HtmlPage firstInfoPage = webClient.getPage(webRequest);
		*/

		
		//直接发送请求
		WebRequest webRequest = new WebRequest(new URL(targetUrl), HttpMethod.GET);
		//设置请求头
		webRequest.setCharset("UTF-8");
		webRequest.setAdditionalHeader("Upgrade-Insecure-Requests", "1");
		webRequest.setAdditionalHeader("Origin", "http://gsxt.zjaic.gov.cn");
		webRequest.setAdditionalHeader("Referer", 
		    "http://gsxt.zjaic.gov.cn/search/doEnGeneralQueryPage.do");
		//设置请求参数
		List<NameValuePair> nameValuePairs = new ArrayList<NameValuePair>();
		nameValuePairs.add(new NameValuePair("keyword", keyword);
		nameValuePairs.add(new NameValuePair("imagecode", imagecode);
		webRequest.setRequestParameters(nameValuePairs);
		//
		HtmlPage firstInfoPage = webClient.getPage(webRequest);
		//
		webClient.removeRequestHeader("Upgrade-Insecure-Requests");
		webClient.removeRequestHeader("Origin");
		webClient.removeRequestHeader("Referer");

		//拿到列表页之后，就可以取到列表页中的文本内容了...
		//进一步的解析工作可以由jsoup来承担，代码略...
	}
}
```

### Jsoup fromUrl、fromString、fromFile

```java
/**
* test fromUrl
*/
public static void main(String[] args) throws IOException {
    Document doc = Jsoup.connect("http://www.baidu.com").get();
    String title = doc.title();
    System.out.println(title);
    FileUtils.writeStringToFile(new File("../Desktop/doc01.html"), doc.toString(), "UTF-8");

    Document doc2 = Jsoup.connect("http://www.baidu.com")
    	.data("query", "java").userAgent("Mozilla")
    	.cookie("auth", "token")
    	.timeout(3000)
    	.get();
    String title2 = doc2.title();
    System.out.println(title2);
}

/**
* test fromString
*/
public static void main(String[] args) {
    //解析html
    String html = "<html>
    	<head><title>First parse</title></head>
    	<body><p>Parsed HTML into a doc.</p></body>
    	</html>";
    Document doc = Jsoup.parse(html);
    System.out.println(doc);
    
    //解析body元素以内的dom树
    String html2 = "<div><p>Lorem ipsum.</p>";
    Document doc2 = Jsoup.parseBodyFragment(html2);
    System.out.println(doc2);
}

/**
* test fromFile
*/
public static void main(String[] args) throws IOException {
    File htmlFile = new File("C:/Users/Administrator/Desktop/doc01.html");
    Document doc = Jsoup.parse(htmlFile, "UTF-8");
    System.out.println(doc);
}
```
<br>


### Jsoup navigate API

```java
public static void main(String[] args) throws IOException {
    Document doc = Jsoup.connect("http://jsoup.org/cookbook/extracting-data/dom-navigation").get();
    Element ele_main = doc.getElementById("main"); //
    System.out.println(ele_main);
    
    Elements ele_imgs = doc.getElementsByTag("li"); //
    for (Element ele_img : ele_imgs) {
        System.out.println(ele_img);
    }
    
    Elements ele_n1_cookbooks = doc.getElementsByClass("n1-cookbook");
    for (Element ele_n1_cookbook : ele_n1_cookbooks) {
        System.out.println(ele_n1_cookbook);
    }
    
    Elements ele_hashrefs = doc.getElementsByAttribute("href");
    for (Element ele_hashref : ele_hashrefs) {
        System.out.println(ele_hashref);
    }
    
    Elements eles_uls = doc.getElementsByTag("ul");
    Elements eles_lis = eles_uls.get(0).getElementsByTag("li");
    Element ele_li_first = eles_lis.get(0);
    ele_li_first = ele_li_first.firstElementSibling(); //
    System.out.println(ele_li_first);
    Element ele_li_last = ele_li_first.lastElementSibling(); //
    System.out.println(ele_li_last);
    Element ele_next = ele_li_first.nextElementSibling(); //
    System.out.println(ele_next);
    Element ele_pre = ele_li_last.previousElementSibling(); //
    System.out.println(ele_pre);
    Elements ele_siblings = ele_li_first.siblingElements(); //
    System.out.println(ele_siblings);
    
    
    Element ele_wrap = doc.getElementsByClass("wrap").get(0);
    Element ele_wrap_parent = ele_wrap.parent(); //
    System.out.println(ele_wrap_parent.tagName());
    
    Elements ele_wrap_children = ele_wrap.children();
    for (Element ele_wrap_child : ele_wrap_children) {
        System.out.println(ele_wrap_child.tagName());
    }
    

    Elements eles_uls = doc.getElementsByTag("ul");
    Elements eles_lis = eles_uls.get(0).getElementsByTag("li");
    Element ele_li_first = eles_lis.get(0);
    String ele_li_first_outerHtml = ele_li_first.outerHtml(); //
    System.out.println(ele_li_first_outerHtml); //<li class="n1-home"><h4><a href="/">jsoup</a></h4></li>
    System.out.println(ele_li_first); //<li class="n1-home"><h4><a href="/">jsoup</a></h4></li>
    
    
    Elements ele_scripts = doc.getElementsByTag("script");
    for (Element ele_script : ele_scripts) {
        System.out.println(ele_script);
    }
    
    Elements eles_uls = doc.getElementsByTag("ul");
    Elements eles_lis = eles_uls.get(0).getElementsByTag("li");
    Element ele_li_first = eles_lis.get(0); //<li class="n1-home"><h4><a href="/">jsoup</a></h4></li>
    Element ele_append = ele_li_first.append("<a href='hehe'><img src='hehe'></a>"); 
    //<li class="n1-home"><h4><a href="/">jsoup</a></h4><a href="hehe"><img src="hehe"></a></li>
    System.out.println(ele_append);*/
    
    Elements eles_uls = doc.getElementsByTag("ul");
    Elements eles_lis = eles_uls.get(0).getElementsByTag("li");
    Element ele_li_first = eles_lis.get(0); //<li class="n1-home"><h4><a href="/">jsoup</a></h4></li>
    Element ele_append = ele_li_first.appendText("呵呵呵呵"); 
    //<li class="n1-home"><h4><a href="/">jsoup</a></h4>呵呵呵呵</li>
    System.out.println(ele_append);
    
    Elements eles_uls = doc.getElementsByTag("ul");
    Elements eles_lis = eles_uls.get(0).getElementsByTag("li");
    Element ele_li_first = eles_lis.get(0); //<li class="n1-home"><h4><a href="/">jsoup</a></h4></li>
    ele_li_first = ele_li_first.html("<a href='xxx'>哈哈哈哈</a>");
    System.out.println(ele_li_first); //<li class="n1-home"><a href="xxx">哈哈哈哈</a></li>      
}
```
<br>

### Jsoup [selector](http://jsoup.org/cookbook/extracting-data/selector-syntax)

jsoup elements support a CSS (or jquery) like selector syntax to find matching elements, that allows very powerful and robust queries. <br>

The select method is available in a Document, Element, or in Elements. It is contextual, so you can filter by selecting from a specific element, or by chaining select calls. <br>

Select returns a list of Elements (as Elements), which provides a range of methods to extract and manipulate the results. <br>


```default
selector overview:
----------------------
tagname        find elements by tag, e.g. a
ns|tag         find elements by tag in a namespace, e.g. fb|name finds <fb:name> elements
#id            find elements by ID, e.g. #logo
.class         find elements by class name, e.g. .masthead
[attribute]    elements with attribute, e.g. [href]
[^attr]        elements with an attribute name prefix
[attr=value]   elements with attribute value, e.g. [width=500] 
[attr^=value]  elements with attributes that start with,
[attr$=value]  elements with attributes that end with
attr*=value]   elements with attributes that contain the value. e.g. [href*=/path/]
[attr~=regex]  elements with attribute values that match the regular expression
*              all elements, e.g. *


selector combinations:
----------------------
el#id                elements with ID, e.g. div#logo
el.class             elements with class, e.g. div.masthead
el[attr]             elements with attribute, e.g. a[href]
any combination      e.g. a[href].highlight
parent > child       child elements that descend directly from parent
and body > *         finds the direct children of the body tag
siblingA + siblingB  finds sibling B element immediately preceded by sibling A, e.g. div.head + div
siblingA ~ siblingX  finds sibling X element preceded by sibling A, e.g. h1 ~ p
el,el,el             group multiple selectors, find unique elements that match any of the selectors


pseudo selectors:
----------------------
:lt(n)              find elements whose sibling index is less than n; e.g. td:lt(3)
:gt(n)              find elements whose sibling index is greater than n; e.g. div p:gt(2)
:eq(n)              find elements whose sibling index is equal to n; e.g. form input:eq(1)
:has(seletor)       find elements that contain elements matching the selector; e.g. div:has(p)
:not(selector)      find elements that do not match the selector; e.g. div:not(.logo)
:contains(text)     find elements that contain the given text. The search is case-insensitive
:containsOwn(text)  find elements that directly contain the given text
:matches(regex)     find elements whose text matches the specified regular expression
:matchesOwn(regex)  find elements whose own text matches the specified regular expression

Note that the above indexed pseudo-selectors are 0-based, that is, the first element is at index 0, the second at 1, etc. More selector API is 'https://jsoup.org/apidocs/org/jsoup/select/Selector.html'.
```

```java
public static void main(String[] args) throws IOException {
    Document doc = Jsoup.connect("http://jsoup.org/cookbook/extracting-data/dom-navigation").get();
    Elements eles = doc.select("div.wrap>div.header ul li:eq(0)"); 
    //<li class="n1-home"><h4><a href="/">jsoup</a></h4></li>
    System.out.println(eles);
    
    Element ele_body = doc.getElementsByTag("body").get(0);
    eles = ele_body.select("div.wrap > div.header ul li:eq(0) + li"); //下一个兄弟节点
    System.out.println(eles);
    eles = ele_body.select("div.wrap > div.header ul li:eq(1) ~ li"); //后面的所有兄弟节点
    System.out.println(eles);
    System.out.println();
    
    eles = ele_body.select("div.wrap > div.header ul li:eq(0), div.wrap > div.header ul li:eq(0) + li"); 
    //find unique elements that match any of the selectors
    System.out.println(eles); //
}
```
<br>

### Jsoup防止XSS攻击

在做网站的时候，经常会提供用户评论的功能。有些不怀好意的用户，会搞一些脚本到评论内容中，而这些脚本可能会破坏整个页面的行为，更严重的是获取一些机要信息，此时需要清理该HTML，以避免跨站脚本[cross-site scripting](http://en.wikipedia.org/wiki/Cross-site_scripting)攻击（XSS）。通常采取的做法是对前端输入的内容进行过滤。<br>

可以选用的工具有：
* Jsoup 是一款Java的HTML 解析器，可直接解析某个URL地址、HTML文本内容。它提供了一套非常省力的API，可通过DOM，CSS以及类似于JQuery的操作方法来取出和操作数据。
* HTML Parser 是一个对HTML进行分析的快速实时的解析器，最新的发行版本是1.6，另外2.0的开发版本已经两年没有进展了。（采用Java实现）
* XSS HTMLFilter 一个采用Java实现的开源类库。用于分析用户提交的输入，消除潜在跨站点脚本攻击(XSS)，恶意的HTML，或简单的HTML格式错误。
* HTML Purifier 是基于php 5所编写的HTML过滤器，帮助用户保障HTML的合法性，支持自定义过滤规则，还可以把不标准的HTML转换为标准的HTML。它可以使你确认HTML是否包含跨站脚本攻击企图或其它的恶意攻击，有效防范XSS攻击。

Jsuop使用示例代码：

```java
String unsafe = "<p><a href='http://xxx.com/' onclick='stealCookies()'>Link</a></p>";
String safe = Jsoup.clean(unsafe, Whitelist.basic());


//OSChina网站使用的Jsoup, 对输入内容进行过滤的代码：
private final static Whitelist user_content_filter = Whitelist.relaxed();  
static {  
    user_content_filter.addTags("embed","object","param","span","div");  
    user_content_filter.addAttributes(":all", "style", "class", "id", "name");  
    user_content_filter.addAttributes("object", "width", "height","classid","codebase");      
    user_content_filter.addAttributes("param", "name", "value");  
    user_content_filter.addAttributes("embed", "src","quality",
        "width","height","allowFullScreen",
        "allowScriptAccess","flashvars",
        "name","type","pluginspage");  
}  
  
/** 
 * 对用户输入内容进行过滤 
 * @param html 
 * @return 
 */  
public static String filterUserInputContent(String html) {  
    if(StringUtils.isBlank(html)) return "";  
    return Jsoup.clean(html, user_content_filter);  
    //return filterScriptAndStyle(html);  
}  
```

说明: <br>
XSS又叫CSS (Cross Site Script) ，跨站脚本攻击。它指的是恶意攻击者往Web页面里插入恶意html代码，当用户浏览该页之时，嵌入其中Web里面的html代码会被执行，从而达到恶意攻击用户的特殊目的。XSS属于被动式的攻击，因为其被动且不好利用，所以许多人常忽略其危害性。所以我们经常只让用户输入纯文本的内容，但这样用户体验就比较差了。<br>
一个更好的解决方法就是使用一个富文本编辑器WYSIWYG如CKEditor 和 TinyMCE。这些可以输出HTML并能够让用户可视化编辑。虽然他们可以在客户端进行校验，但是这样还不够安全，需要在服务器端进行校验并清除有害的HTML代码，这样才能确保输入到你网站的HTML是安全的。否则，攻击者能够绕过客户端的Javascript验证，并注入不安全的HMTL直接进入您的网站。<br>
jsoup的whitelist清理器能够在服务器端对用户输入的HTML进行过滤，只输出一些安全的标签和属性。<br>
jsoup提供了一系列的Whitelist基本配置，能够满足大多数要求；但如有必要，也可以进行修改，不过要小心。<br>
这个cleaner非常好用不仅可以避免XSS攻击，还可以限制用户可以输入的标签范围。<br>










