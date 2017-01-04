---
layout: post
title:  "网页中屏蔽快捷键"
desc: "网页中屏蔽快捷键"
keywords: "网页中屏蔽快捷键,kinglyjn"
date: 2012-11-16
categories: [HTMLX]
tags: [html]
icon: fa-html5
---


```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>屏蔽页面的快捷键测试</title>
</head>
<body>
    <div>
        哈哈哈，这个页面的很多快捷键（ctrl+s，F12...）都不能用（页面嵌入了iframe框架屏蔽失效），慢慢地手动点击吧！
    </div>
    
    <script src="jquery-2.1.4.min.js"></script>
    <script>
        (function(){
            var doc=document,
                ua = navigator.userAgent.toLowerCase(),
                check = function(r){return r.test(ua);},
                isOpera = check(/opera/),
                isChrome = check(/chrome/),
                isWebKit = check(/webkit/),
                isSafari = !isChrome && check(/safari/),
                isSafari2 = isSafari && check(/applewebkit\/4/),
                isSafari3 = isSafari && check(/version\/3/),
                isSafari4 = isSafari && check(/version\/4/),
                isIE = !isOpera && check(/msie/),
                isIE7 = isIE && check(/msie 7/),
                isIE8 = isIE && check(/msie 8/),
                isIE9 = isIE && check(/msie 9/),
                isIE10 = isIE && check(/msie 10/),
                isIE11 = isIE && check(/msie 11/),
                isIE6 = isIE && !isIE7 && !isIE8 && !isIE9 && !isIE10 && !isIE11,
                isGecko = !isWebKit && check(/gecko/),
                isGecko2 = isGecko && check(/rv:1\.8/),
                isGecko3 = isGecko && check(/rv:1\.9/);
            function preventKey(e) {
                var ev = e || window.event,//获取event对象  
                obj = ev.target || ev.srcElement,//获取事件源  
                t = obj.type || obj.getAttribute('type'),readonly = obj.readOnly||obj.getAttribute('readonly'), code = ev.keyCode||ev.which||ev.charCode,charcode = String.fromCharCode(code).toLowerCase();//获取事件源控件类型,控件只读属性，键盘值
                if ( (code == 8 && t != "password" && t != "text" && t != "textarea")||(readonly&&(t == "password" || t == "text" || t == "textarea"))) {//除不只读输入框外禁止后退键
                    return false;
                }
                if (((isOpera || isGecko)?ev.which==0:true)&&(code == 116 || code == 122|| code == 123 || (ev.shiftKey && code == 121))||(((isOpera || isGecko)?ev.which!=0:true)&&ev.ctrlKey && (charcode == 'a' || charcode == 's'))) {//屏蔽 F5,F11,F12,shift+F10,ctrl+a,ctrl+s
                    if (isIE)
                        ev.keyCode = 0;
                    ev.returnValue = false;
                    return false;
                }
            };
            if (isOpera || isGecko)
                doc.onkeypress = preventKey;
            else if (isIE || isChrome || isSafari)
                doc.onkeydown = preventKey;
            if(isIE)document.onselectstart=function(){return false;};
            doc.oncontextmenu = function(){
                if(window.event){
                    window.event.cancelBubble = true;
                    window.event.returnValue=false;
                }
                return false;
            };
            try{window.history.forward(1);}catch(e){}
        })();
    </script>
</body>
</html>
```

<br>
测试效果：<br>

<div>
    哈哈哈，这个页面的很多快捷键（ctrl+s，F12...）都不能用（页面嵌入了iframe框架屏蔽失效），慢慢地手动点击吧！
</div><br>

<script>
    (function(){
        var doc=document,
            ua = navigator.userAgent.toLowerCase(),
            check = function(r){return r.test(ua);},
            isOpera = check(/opera/),
            isChrome = check(/chrome/),
            isWebKit = check(/webkit/),
            isSafari = !isChrome && check(/safari/),
            isSafari2 = isSafari && check(/applewebkit\/4/),
            isSafari3 = isSafari && check(/version\/3/),
            isSafari4 = isSafari && check(/version\/4/),
            isIE = !isOpera && check(/msie/),
            isIE7 = isIE && check(/msie 7/),
            isIE8 = isIE && check(/msie 8/),
            isIE9 = isIE && check(/msie 9/),
            isIE10 = isIE && check(/msie 10/),
            isIE11 = isIE && check(/msie 11/),
            isIE6 = isIE && !isIE7 && !isIE8 && !isIE9 && !isIE10 && !isIE11,
            isGecko = !isWebKit && check(/gecko/),
            isGecko2 = isGecko && check(/rv:1\.8/),
            isGecko3 = isGecko && check(/rv:1\.9/);
        function preventKey(e) {
            var ev = e || window.event,//获取event对象  
            obj = ev.target || ev.srcElement,//获取事件源  
            t = obj.type || obj.getAttribute('type'),readonly = obj.readOnly||obj.getAttribute('readonly'), code = ev.keyCode||ev.which||ev.charCode,charcode = String.fromCharCode(code).toLowerCase();//获取事件源控件类型,控件只读属性，键盘值
            if ( (code == 8 && t != "password" && t != "text" && t != "textarea")||(readonly&&(t == "password" || t == "text" || t == "textarea"))) {//除不只读输入框外禁止后退键
                return false;
            }
            if (((isOpera || isGecko)?ev.which==0:true)&&(code == 116 || code == 122|| code == 123 || (ev.shiftKey && code == 121))||(((isOpera || isGecko)?ev.which!=0:true)&&ev.ctrlKey && (charcode == 'a' || charcode == 's'))) {//屏蔽 F5,F11,F12,shift+F10,ctrl+a,ctrl+s
                if (isIE)
                    ev.keyCode = 0;
                ev.returnValue = false;
                return false;
            }
        };
        if (isOpera || isGecko)
            doc.onkeypress = preventKey;
        else if (isIE || isChrome || isSafari)
            doc.onkeydown = preventKey;
        if(isIE)document.onselectstart=function(){return false;};
        doc.oncontextmenu = function(){
            if(window.event){
                window.event.cancelBubble = true;
                window.event.returnValue=false;
            }
            return false;
        };
        try{window.history.forward(1);}catch(e){}
    })();
</script>