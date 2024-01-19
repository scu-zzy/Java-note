# 鉴权
一、AGC：ak/sk鉴权

1、实现filter，实现ConsoleAuthServiceFilter，用@Component加载为Spring的Bean，注册到resources下的org.apache.servicecomb.common.rest.filter.HttpServerFilter中。

2、从请求头中的authorization中的accessId即为ak，sk即密钥从敏感配置项中获取。

3、将ak和sk以及请求中的各种参数如path、body等做签名，判断和请求头（authorization）中的签名是否相等。

4、将用户信息uid、projectId、teamId等存入上下文中（使用threadLocal储存），程序后续从上下文获取。

二、WO：

1、同样实现filter。

2、从请求中获取cookie，从cookie中获取woAuth的值，这个值解码获取woToken。

3、通过woToken可以获取用户的相关信息，判断用户是否有权限登陆系统、获取资源等，没有则报错。

4、从http请求中获取csrfToken进行校验，该值由服务端生成，用woToken及通过woToken获取的useId生成字符串并设置过期时间后用密钥进行加密，返回给前端页面。前端页面将该值通过http头传下来。校验时，将csrfToken用密钥进行解析，与woken和useId对比，校验是否相等及是否过期。

5、将用户的woToken等存入上下文中（使用threadLocal储存），程序后续从上下文获取。通过woToken能获取到工号等信息。

三、API:

1、用户使用appId和自己的密钥从Oauth服务器获取accessToken，放入http头中请求api。

2、实现filter，获取accessToken从Oauth服务器中获取accessToken的详细信息（这里为了性能会做一个本地缓存），如过期时间、appId等，校验accessToken是否无效、是否过期、appId和传入的是否相同。
