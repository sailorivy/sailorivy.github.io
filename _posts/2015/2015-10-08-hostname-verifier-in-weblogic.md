---
title: WebLogic的SSL主机名验证
layout: post
categories: WebLogic
tags: WebLogic SSL 主机名验证
comments: yes
---

<h2>WebLogic中SSL主机名验证</h2>
WebLogic提供了一个高级功能——SSL主机名验证，WebLogic上运行的应用作为客户端向第三方服务发起HTTPS请求时，WebLogic会验证所请求的第三方服务URL里的主机名是否是WebLogic可信任的（也就是验证第三方服务URL里的主机名是否和可信任证书里配置的主机名相匹配），如果不匹配，WebLogic就会断开应用和第三方服务之间的SSL连接。

这个功能默认是开启的，只需要将第三方服务的证书导入WebLogic可信任证书里就可以了。WebLogic的可信任证书缺省是演示信任密钥库和Java标准的信任密钥库，最方便的就是导入Java标准的信任秘钥库（可以在管理控制台的“配置-秘钥库-信任-Java标准信任密钥库”查看Java标准信任密钥库的具体位置），命令示例如下：

1. 导入

    keytool -importcert -noprompt -keystore "Java标准信任密钥库的位置" -storepass changeit -alias alias -file 第三方服务证书的存放位置
	
2. 导入后查看、确认

    keytool -list -keystore "Java标准信任密钥库的位置" -storepass changeit
    
    
<h2>主机名验证器</h2>
<h3>缺省主机名验证器</h3>
缺省情况下，WebLogic使用最严格的主机名验证器（12c的管理控制台上是“BEA主机名验证器”），它会精确匹配主机名，只要不一样就会报错并断开SSL连接，当然localhost、127.0.0.1和本机IP不会验证。

下面是验证失败时报的错：

	<2015-9-18 下午02时02分48秒 CST> <Warning> <Security> <BEA-090504> <Certificate chain received from api.xxxx.cn - 60.12.226.18 failed hostname verification check. Certificate contained portal.xxxx.cn but check expected api.xxxx.cn>
	<2015-9-18 下午02时02分48秒 CST> <Error> <HTTP> <BEA-101019> <[ServletContext@1257418214[app:apitest module:apitest.war path:null spec-version:3.0]] Servlet failed with an IOException.
		javax.net.ssl.SSLKeyException: Hostname verification failed: HostnameVerifier=weblogic.security.utils.SSLWLSHostnameVerifier, hostname= api.xxxx.cn.

	
可以看出WebLogic缺省的主机名验证器是weblogic.security.utils.SSLWLSHostnameVerifier。第三方服务URL里的主机名和可信任证书里的主机名都必须是api.xxxx.cn。

<h3>通配符主机名验证器</h3>
有些第三方服务提供的证书并不会精确指定主机名，提供服务比较多的时候，通常会用通配符，比如*.xxxx.cn。

将第三方服务提供的证书导入WebLogic的信任秘钥库之后，如果仍然使用WebLogic缺省的主机名验证器就会失败：

    <2015-9-18 下午02时02分48秒 CST> <Warning> <Security> <BEA-090504> <Certificate chain received from api.xxxx.cn - 60.12.226.18 failed hostname verification check. Certificate contained `*.xxxx.cn` but check expected `api.xxxx.cn`>
    <2015-9-18 下午02时02分48秒 CST> <Error> <HTTP> <BEA-101019> <[ServletContext@1257418214[app:apitest module:apitest.war path:null spec-version:3.0]] Servlet failed with an IOException.
        javax.net.ssl.SSLKeyException: Hostname verification failed: HostnameVerifier=weblogic.security.utils.SSLWLSHostnameVerifier, hostname=api.xxxx.cn.

除了缺省的主机名验证器，WebLogic还提供了一个通配符主机名验证器，它支持满足下面要求的主机名：

1. 至少包含两个点`.`
2. 以星号`*`开头，但不包含其他星号

支持通配符的主机名验证器是weblogic.security.utils.SSLWLSWildcardHostnameVerifier。如果使用缺省主机名验证器时报如下的错误，那就可以将管理控制台的“配置-SSL-高级-主机名验证”改成“定制主机名验证器”，同时将“定制主机名验证器”设置为weblogic.security.utils.SSLWLSWildcardHostnameVerifier

将主机名验证器换成SSLWLSWildcardHostnameVerifier之后，前面的错误就没有了。

<h3>定制主机名验证器</h3>
如果缺省的和通配符的主机名验证器都不能满足使用要求，可以自定义主机名验证器。WebLogic提供了weblogic.security.SSL.HostnameVerifier接口，建立SSL连接的时候会回调HostnameVerifier接口的实现来验证主机名，进而确定是否允许建立连接。

要做的事情很简单，就是实现weblogic.security.SSL.HostnameVerifier接口。注意：定制主机名验证器要提供public的无参构造函数。

相当于关闭主机名验证功能的定制验证器示例如下：

    public class NulledHostnameVerifier implements
                     weblogic.security.SSL.HostnameVerifier {
      public boolean verify(String urlHostname, javax.net.ssl.SSLSession session) {
        return true;
      }
    }

然后，将实现类加到WebLogic的ClassPath中，将管理控制台的“配置-SSL-高级-主机名验证”改成“定制主机名验证器”，同时将“定制主机名验证器”设置为自定义的主机名验证器全限定名称就可以了。

<h3>关闭主机名验证器</h3>
出现主机名验证失败的错误后，最粗放的解决办法就是关闭WebLogic的主机名验证功能（将管理控制台的“配置-SSL-高级-主机名验证”改成“无”）。

但这种方式可能会有安全隐患，所以并不推荐。如果缺省主机名验证器太严格、不能满足实际使用要求，还是根据实际情况选择WebLogic提供的通配符主机名验证器或自己定制的主机名验证器。

