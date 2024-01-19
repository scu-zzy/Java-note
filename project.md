# 鉴权
一、AGC：ak/sk鉴权
1、实现filter，实现ConsoleAuthServiceFilter，用@Component加载为Spring的Bean，注册到resources下的org.apache.servicecomb.common.rest.filter.HttpServerFilter中。
2、从请求头中的authorization中的accessId即为ak，sk从敏感配置项中获取。
3、将ak和sk以及请求中的各种参数如path、body等做签名，判断和请求头（authorization）中的签名是否相等。

二、WO：

三、Event/Store：

# 获取AGC的用户信息-使用TreadLocal
1、
