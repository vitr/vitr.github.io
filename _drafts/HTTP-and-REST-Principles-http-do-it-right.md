---
layout: post
title: "HTTP Potocol - Do It Right"
date: {}
author: Vitaliy Ryepnoy
summary: HTTP and REST tips.
categories: 
  - http
  - rest
thumbnail: heart
tags: 
  - welcome
  - to
  - carte
  - noire
published: true
---








Here are some good quotes.

quote: "If you're going to do something, then do it right." 

and another one:

If you don’t have time to do it right, when will you have time to do it over?

~ John Wooden

<sub><sup>from the comments (here [http://empoweredquotes.com/2008/06/30/if-you-don%E2%80%99t-have-time-to-do-it-right-when-will-you-have-time-to-do-it-over/ ](http://empoweredquotes.com/2008/06/30/if-you-don%E2%80%99t-have-time-to-do-it-right-when-will-you-have-time-to-do-it-over/))</sup></sub>
   
It is amazing how well this applies to **software engineering**, home improvement and auto repairs!


##HTTP & REST Principles

Organising endpoints
URLs identify resources, resources are some entities. A file is a resource, a user is a resource. Calling a method isn't a resource.
If you want to wash your car don't do it like this
GET /?method=wash&target=mycar - WRONG
Better create a washer resource
POST /washer/?target=mycar
Why POST? Because, first of all, GET is safe method and designed just to represent resources, not to change them. Secondly, GET response could be cached, so, the really action hasn't been performed.
This is explained in Using http verbs section

Scheme   Host      Path        Query        Fragment
  ↓        ↓         ↓            ↓             ↓
http://cars.com/make/bmw/?deliver_to=Albion#photo

If you have a car catalogue by makes, logically you can structure it by path, as a car can belong to only one make. However, you can deliver a car to multiple places, so, the "deliver to X" filter is better to organise by query string.

Using http verbs
HTTP call means applying a verb to URL. The result of this should be what a verb is saying, e.g. GET returns a representation of a resource, DELETE deletes it, etc.
Methods GET, HEAD, OPTIONS are safe, assuming, they don’t change the state of resources. Many applications, like link pre-fetchers in browsers or messengers feel free to visit such URLs without asking an end-user.
Methods GET and HEAD are cacheable by default. OPTIONS, POST, PUT, PATCH, DELETE are not. So, if you perform
POST /washer/?target=mycar
You will be (almost) sure that the request will be performed, but if you perform a GET method, some intermediate proxy can decide to return a cached result and the car won't be washed in reality.
Methods GET, PUT, DELETE are symmetrical. PUT creates or updates a resource, GET returns the resource representation, DELETE deletes it.
Method HEAD is a synonym of GET, but returns only headers (resource's metadata), without a body.
POST is used when if you don't have an URL of the resource. Example post a new message on forum
POST threads/dealer/messages
But if the client is allowed to generate messages ids, then the PUT method should be called
PUT threads/dealer/messages/100500
If you accidentally (or because of network issues) call POST twice, it  will create the same message with different ids. PUT may be called multiple times with no consequences.  In other words, PUT method is idempotent.  Using idempotent PUT may lead to another issues, like conflict resolution, but developers can resolve these issues in the code and the end result will be safer and more reliable.
PUT could be used for both creation and updating. However, for updating you should provide the resource full data. If you need a partial update, the PATCH method should be used instead. PATCH is non-cacheable, non-safe and non-idempotent.
http://programmers.stackexchange.com/questions/260818/why-patch-method-is-not-idempotent
It's hard to find network intermediaries without PUT/PATCH/DELETE support nowadays (2015). Nevertheless,  if you face such, use GET/POST (for reading/writing operations) and pass the real method in X-HTTP-Method header.
http://www.hanselman.com/blog/HTTPPUTOrDELETENotAllowedUseXHTTPMethodOverrideForYourRESTServiceWithASPNETWebAPI.aspx


Response codes
The response codes give hints to a client what to do next.
3xx - extra action should be performed
4xx - client error, client request is incorrect. It's recommended to include a particular error explanation to the response
5xx - server error, client request is correct
Successful response codes
GET - 200
PUT - 201 if resource created or 200  if resource updated
POST - 200 or 201 (with URL of created resource)

Common misunderstandings
401 Unauthorized must be accompanied with WWW- Authenticate header, therefore, must be used only with HTTP authentication, in other cases use 403 Forbidden.
3xx status codes are not only redirects, they indicate that a client must perform a additional action, e.g. 304 Not Modified - get the actual resource version from cache.
404 one of not many you can call again. It means that the resource isn't present at the moment, but could be in the future. 404 is a status of uncertainness, when we don't want to reveal the actual error. If we want to give a hint we should use
410 Gone - resource deleted  or 400 - general status

Capability URL
Capability URLs grant access to a resource to anyone who has the URL. There are particular application design patterns for which this is useful as they remove the necessity for users to log in to a site and are easily delegated to others. But their use can open up some security issues. URLs are not generally required to be kept secret, and there are various routes through which capability URLs can leak into unintended hands
This is a special subclass of URLs. They contain both a resource and its action. Example - password recovery, direct links to 'secret' pdf reports. Some security measures for using such URLs
	- Strong random generator (UUID 4)
	- HTTPS
	- Pages available via capability URL (the 'secret' pages) should be excluded from search engines indexing
The damage control
	- Revoking capability URLs, e.g. after sharing a document make possible to hide it again
	- Expiration time must be shorter for more important links
The secret pages should be protected at maximum:
	- No third-party content (scripts, images, etc.)
	- No links to third-party sites (hide referral if the links are needed)
	- Change the URL via History API right after visiting a page in a browser (no one can see the actual url)
	- Any input should be confirmed by user action via CSRF token signed form (this disables browser's auto fill feature)


Conclusion, final thoughts about RFC's
Everything above exists in RFC's and other standards as recommendations. You may or may not follow the recommendations, but just keep in mind that many other network agents live between your service and your clients. Most of them follow RFC in their HTTP protocol implementation. So, if you break the recommendations in your applications, you'll be responsible for consequences, but the most important point here is that even if you communicate in isolated environment, how are you going to explain all non-standard behaviour to your colleagues? In fact, you'll be creating your own HTTP dialect, which at least should be documented properly.

http://www.rfc-editor.org/rfc/rfc2616.txt
http://tools.ietf.org/html/rfc5789
http://www.w3.org/TR/webarch
http://w3ctag.github.io/capability-urls

##15 trivial facts about correct usage of the HTTP protocol


### #1 http resource is a noun not a verb

URL identifies a resource, a data entity. A file is a resource, a handle is resource, a method call isn't a resource.
If you want to clean your car, don't do it like this

    GET /?method=clean&target=myCar  (hit the button???)

better make a resource called **cleaner**, it will make more sense

    POST /cleaner/?target=myCar

Wait, POST, why not GET? Please, read further


### #2 organise hierarchical data in path and filters in query
The URL includes scheme, host, path, query and fragment.

The path is used for hierarchically organised resources, the query is used for non-hierarchically organised resourses and operation parameters. The fragment identifies a subresource, which has no direct URL.

Scheme      Host                 Path               Query      Fragment
  ↓           ↓                    ↓                  ↓            ↓
http://pretty-kittens.com/breeds/maine-coon/?deliver_to=London#photo

If you have on your site "pretty-kittens" a breed catalogue, it's logic to organize the catalogue in path parts, as every kitten belongs to one breed.
In cotrary, you can deliver a kitten to different ciies, therefore the "deliver to city XYZ" filter shoould be created in the query.

### #3 apply http verb to the url
An HTTP call is about applying a method (a verb) to URL. The result of such applying - surprise, surprise - what is the verb says, i.e. GET returns a resource view, DELETE deletes a resourse, etc. 

### #4 methods GET, HEAD, OPTION don't change the state of a resource
Methods GET, HEAD, OPTIONS are safe. Calling these methods does not change a resource state. Therefore, many network agents, e.g. url prefetcher in a browser or instant messanger, assume visiting such resources without explicit user requests. At that, they do not violate any standards.

Методы GET, HEAD, OPTIONS — безопасные. Предполагается, что вызов этих методов состояния ресурса не изменяет. Поэтому многие сетевые агенты — такие, например, как префетчер ссылок в браузере или мессенджере — считают себя вправе по таким ссылкам ходить без явного волеизъявления пользователя. ИЧСХ, никаких стандартов не нарушают.

### #5 methods GET ? HEAD could be cached, OPTIONS, POST, PUT, PATCH, DELETE couldn't
Methods GET and HEAD are cacheable by default, but OPTIONS,POST,PUT, PATCH, DELETE are not. Therefore if you hit the button with POST, you're (almost) sure that the request will be executed. If you hit with GET, some intermediate proxe can SUDDENLY return cached response and, in fact, the click hasn't happend.
шарахнули по Луне - click a button, button clicker, hit a button.

По умолчанию методы GET и HEAD кэшируются, OPTIONS, POST, PUT, PATCH, DELETE — нет. Поэтому если вы шарахнули по Луне методом POST, вы можете быть (почти) уверены, что этот запрос выполнится. Если вы шарахаете методом GET, какой-нибудь промежуточный прокси может ВНЕЗАПНО отдать вам ответ из кэша, и шарах в реальности не произойдёт.
  
### ????Operation GET, PUT, DELETE are symmetrical????  
### #6 Операции GET, PUT, DELETE симметричны. PUT кладёт нечто по URL-у (создавая новый ресурс или перезаписывая старый), GET по этому URL-у возвращает представление того, что положил PUT, DELETE удаляет ресурс.
       Метод HEAD синонимичен по семантике методу GET, но не возвращает тело ответа, а только его заголовки (метаинформацию о ресурсе).
       
### POST is used if you don't have a specific URL for applying the acation. For example, if a user writes a new post on the forum thread, he can calculate the id himself and do

#7 POST используется в том случае, если у вас нет URL, к которому вы хотите применить операцию. Например, если пользователь пишет новое сообщение в тредик на форуме, он может сам вычислить его id и сделать:
       
       PUT /threads/php-rulezz/messages/100500
       
       
       Если клиенту генерировать id не разрешено, ему придётся делать POST на ресурс уровнем выше по иерархии:
       
       POST /threads/php-rulezz/messages
       
       
       И этот ресурс сам создаст новое сообщение.
       Обратите внимание, если вы по ошибке или вследствие сетевых проблем повторите POST запрос — создастся второе сообщение в треде, идентичное первому. PUT вы можете делать хоть 100500 раз, результат не изменится. Это свойство называется идемпотентностью.
               Разумеется, использование идемпотентного PUT порождает свои проблемы — в частности, как разрешать конфликты. Придётся больше программировать, зато результат будет более надёжным и безопасным.

 
 
### #8 PUT может использоваться как для создания новых ресурсов, так и для обновления старых. Однако в случае использования PUT для перезаписи предполагается, что в теле запроса передаётся закодированный ресурс целиком. Если же вы хотите модифицировать ресурс, т.е. изменить его внутреннее представление без полной перезаписи, то для этого был придуман метод PATCH. Этот метод некэшируемый, небезопасный и неидемпотентный.


### #9  Коды ответа нужны в первую очередь для того, чтобы клиент мог понять, что ему делать дальше. 3хх говорит, что для успешного выполнения запроса нужно выполнить дополнительное действие. 4хх говорит, что клиент, составляя запрос, сделал что-то неправильно и, обычно, о том, что умолять бесполезно — повторное выполнение запроса всё равно выкинет ошибку. В 4хх крайне рекомендуется включать информацию о том, что конкретно клиент сделал не так. 5хх говорит о том, что клиент всё сделал правильно — проблема на стороне сервера.
       
       Обычно при успешном выполнении операции сервер отвечает на GET — 200, на PUT — 201 Created (если ресурс создан) или 200 (ресурс обновлён), на DELETE — 204 (операция успешна, возвращать нечего), на POST — 200 или 201 (во втором случае в заголовке, обычно Location, указывается URL созданного ресурса).

        
### #10 Работая с HTTP-статусами, не наступите на популярные грабли:
        
            статус 401 Unauthorized обязан сопровождаться заголовком WWW-Authenticate и, таким образом, применим только тогда, когда клиент аутентифицируется посредством HTTP-аутентификации; во всех остальных случаях следует использовать 403 Forbidden;
            статусы 3xx — это не только редиректы; они показывают, что клиент должен выполнить дополнительное действие, иначе запрос не может считаться успешным; например, по статусу 304 Not Modified клиент должен взять актуальную версию ресурса из кэша;
            статус 404, как ни странно, один из немногих 4xx статусов, которые клиент имеет право повторять — он означает, что ресурса сейчас нет, но вполне возможно, что он появится; вообще 404 — статус неопределённости, который используется, если сервер не хочет раскрывать механику ошибки; для того, чтобы индицировать клиенту, что без дополнительных действий с его стороны ресурс не появится, следует использовать 410 Gone (ресурс был удалён) либо общий статус 400.

        
### #11 Существует особый подкласс URL-ов, которые кодируют в себе и ресурс, и действие над ним. В англоязычной литературе их принято называть Capability URLs. Классический пример такого URL — ссылки на восстановление паролей, а также всевозможные «секретные» прямые ссылки на всяческие ресурсы.


### #12 Поскольку основная опасность при работе с Capability URL — возможность их утечки, следует максимально закрыть возможности случайно такой URL найти или перехватить:
        
            для генерации секретных частей URL должен использоваться сильный генератор случайных строк (например, UUID 4), исключающий возможности найти Capability URL перебором; разумеется, URL не должен генерироваться детерминированным способом типа md5(username) и такие URL нельзя пропускать через сокращатели ссылок;
            Capability URLs должны работать только по HTTPS;
            страницы, доступные через Capability URL, должны быть закрыты wildcard-ом от индексации роботами.

        
### #13 Должны быть предусмотрены меры минимизации возможного ущерба:
        пользователь, создавший Capability URL (например, расшаривший документ), должен иметь возможность сделать обратную операцию, т.е. отозвать URL;
        Capability URLs должны протухать со временем; чем опаснее предоставляемый доступ, тем короче должен быть срок жизни URL.
                
### #14  Наконец, сами «секретные» страницы должны быть защищены от сливания данных сторонним агентам:
        
            на них не должно быть никаких third-party скриптов и картинок, желательно — на уровне CSP;
            на них не должно быть ссылок на third-party сайты; если они необходимы, то нужно скрывать referrer, например, через rel=«noreferrer»;
            вообще желательно через Referrer Policy настроить скрытие referrer-а;
            желательно сразу после захода пользователя через History API менять URL в адресной строке браузера, чтобы его нельзя было подсмотреть через плечо;
            если ссылка предполагает какое-то действие (например, смену пароля), то на секретной странице должна быть форма (кнопка, скрипт), которую требуется отослать, чтобы действие осуществить, причём эта форма должно быть подписана CSRF-токеном (иначе префетчер браузера / почтового клиента / мессенджера сможет восстановить пароль за юзера).

                
### #15 Conclusion 

Everything above exists in RFC's and other standards as recommendations. It doesn't force anyone to follow the recommendations. Everyone is free to build their API's even upon only GET requests, as long as their services work.


Всё описанное выше существует в стандартах исключительно в форме рекомендации, и принудить кого-либо к строгому исполнению этих рекомендаций нельзя. 

Я уже не первый раз рассказываю про всю эту тривию, и часто слышу в ответ «да плевать я на всё это хотел, придумали какой-то ненужной ерунды; как у меня работали все сервисы только на GET, так и дальше будут, мучайтесь со своими PUT-ами и DELETE-ми сами».
        
        Разумеется, вы вольны писать свой сервис сами. Но имейте, пожалуйста, в виду, что между вашим сервером и вашим клиентом, даже если они стоят физически рядышком в одном ДЦ, есть огромное множество других сетевых агентов — браузеров, прокси, роутеров, имплементаций HTTP-протокола в разных языках программирования и разных ОС, DPI-оборудование провайдеров и так далее. Все эти агенты плюс-минус имплементируют протокол HTTP с оглядкой на RFC.
        
        Если вдруг клиентский браузер запрефетчит GET-ссылку и шарахнет по Луне — это будет ваша вина, бесполезно писать производителю. Если у вас деньги переводятся GET-запросом, а имплементация HTTP протокола в вашем языке программирования, не дождавшись ответа от соседнего роутера, решит повторить запрос и проведёт транзакцию дважды — это будет опять ваша вина.
        
        Но даже не это главное. Допустим, ваши HTTP-пакеты гуляют в строго контролируемой среде. Как вы собираетесь объяснять другим разработчикам, какие рекомендации вы нарушили и почему? Как ваш коллега должен понять, что вот этот GET-запрос повторять нельзя, а статус 400 вовсе не означает клиентскую ошибку? Отступая от рекомендаций, вы, фактически, каждый раз создаёте какой-то свой диалект HTTP с собственной семантикой. Не забудьте его хотя бы задокументировать ;)
        
        Список литературы:
        
            www.rfc-editor.org/rfc/rfc2616.txt
            tools.ietf.org/html/rfc5789
            www.w3.org/TR/webarch
            w3ctag.github.io/capability-urls
        
        
        (В разработке последнего документа ваш покорный слуга принимал определённое участие.)             
original 
 [http://habrahabr.ru/company/yandex/blog/265569/]
 
 
comments 
В 2015 году промежуточных узлов, не умеющих PUT/PATCH/DELETE, кажется, не осталось. Я, по крайней мере, не сталкивался.

В любом случае, есть стандартный механизм обхода таких ограничений: использовать GET/POST (первый в случае не изменяющих состояние ресурса операций, второй — для изменяющих), а нужный реально метод передавать заголовком X-HTTP-Method. 

 
 
 Organising endpoints
 
 
 
 Using http verbs
 
 
 
 Response codes
 
 
 
 Capability URL
 
 
 Conclusion, final thoughts about RFC
