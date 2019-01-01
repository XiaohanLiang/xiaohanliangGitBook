---
description: 挖掘一些信息出来
---

# dig

> dig命令主要用于从 DNS域名服务器 查询主机的地址信息
>
> ### 查询单个域名的DNS信息

```bash
$ dig baidu.com
```

![dig&#x67E5;&#x8BE2;&#x7684;&#x7ED3;&#x679C;&#x53EF;&#x4EE5;&#x5206;&#x6210;1~5&#x4E00;&#x5171;&#x4E94;&#x4E2A;&#x90E8;&#x5206;&#xFF0C;&#x6BCF;&#x4E2A;&#x90E8;&#x5206;&#x7684;&#x89E3;&#x91CA;&#x5982;&#x4E0B;](../.gitbook/assets/dig-1.jpg)

1. dig命令的版本以及输入的参数
2. status是比较重要的信息，其中 NOERROR代表本次查询成功
3. QUESTION-SECTION 代表你本次想要查询什么域名
4. ANSWER-SECTION 代表本次查询的结果
5. 第五部分显示了本次查询的一些统计信息，包含用了多长时间，查询的是那个 DNS服务器 等

