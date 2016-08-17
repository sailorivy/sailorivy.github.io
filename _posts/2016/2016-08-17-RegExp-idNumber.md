---
title: 港澳台身份证的正则表达式
layout: post
categories: Memento
tags: 正则表达式
comments: yes
---

##港澳台身份证的正则表达式

###香港
说明：一个英文+6个数字+(一个校验码，0~9或A)

正则表达式：[A-Z]{1,2}[0-9]{6}\([0-9A]\)

参考：[Wikipedia](https://zh.wikipedia.org/wiki/%E9%A6%99%E6%B8%AF%E8%BA%AB%E4%BB%BD%E8%AD%89#.E8.BA.AB.E4.BB.BD.E8.AD.89.E8.99.9F.E7.A2.BC)

###澳门
说明：第一位1、5、7，后面6个数字，最后带括号的一位校验码0~9

正则表达式：[157][0-9]{6}\([0-9]\)

参考：[Wikipedia](https://zh.wikipedia.org/wiki/%E6%BE%B3%E9%96%80%E5%B1%85%E6%B0%91%E8%BA%AB%E4%BB%BD%E8%AD%89#.E8.BA.AB.E4.BB.BD.E8.AD.89.E8.99.9F.E7.A2.BC)

###台湾
说明：1个英文+9个数字

正则表达式：[A-Z][0-9]{9}

参考：[Wikipedia](https://zh.wikipedia.org/wiki/%E4%B8%AD%E8%8F%AF%E6%B0%91%E5%9C%8B%E5%9C%8B%E6%B0%91%E8%BA%AB%E5%88%86%E8%AD%89#.E7.B7.A8.E8.99.9F.E8.A6.8F.E5.89.87)

1. 导入

    keytool -importcert -noprompt -keystore "Java标准信任密钥库的位置" -storepass changeit -alias alias -file 第三方服务证书的存放位置
	
2. 导入后查看、确认

    keytool -list -keystore "Java标准信任密钥库的位置" -storepass changeit
    
    
<h2>Groovy片段示例</h2>

```groovy
    def matchHK = {
        def matcher = it =~ /^[A-Z]{1,2}[0-9]{6}\([0-9A]\)/
    }
    def matchMO = {
        def matcher = it =~ /^[157][0-9]{6}\([0-9]\)/
    }
    def matchTW = {
        def matcher = it =~ /^[A-Z][0-9]{9}/
    }

    def id = context.getIdNumber()
    if (!id) return false
    if (id?.trim()?.isEmpty()) return false
    return matchHK(id.toUpperCase()).matches() || matchMO(id.toUpperCase()).matches() || matchTW(id.toUpperCase()).matches()
```



