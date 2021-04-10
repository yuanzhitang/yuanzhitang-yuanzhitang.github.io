---
layout: post
title:  "SSL 证书基本概念"
date:   2017-10-31 09:42:58
categories: Security
tags: SSL
author: Michael
mathjax: true
---

* content
{:toc}

PKC: Public-Key certificate 公钥证书或者简称Certificate

CA: Certification Authority认证机构，对公钥施加数字签名

CRL: Certificate Revocation List证书作废清单

PKI: Public Key Infrastructure公钥基础设置，是为了能够更有效的运用公钥而制定的一系列规范和规格的总称。

PKCS: PKI的一种，由RSK制定Public-Key Cryptography Standards.

Root CA: 根CA，最高的级别的机构对自己的公钥进行数字签名的行为成为自签名 (Self-signature)



数字证书格式（cer和pfx）的区别 

1. pfx 证书文件是带有私钥，由Public Key Cryptography Standards #12，PKCS#12标准定义，包含了公钥和私钥的二进制格式的证书形式

2. 二进制编码的证书 
证书中没有私钥，DER 编码二进制格式的证书文件，以cer作为证书文件后缀名。 

3. Base64编码的证书 
证书中没有私钥，BASE64 编码格式的证书文件，也是以cer作为证书文件后缀名。

由定义可以看出，只有pfx格式的数字证书是包含有私钥的，cer格式的数字证书里面只有公钥没有私钥。

在pfx证书的导入过程中有一项是“标志此密钥是可导出的。这将您在稍候备份或传输密钥”。一般是不选中的，如果选中，别人就有机会备份你的密钥了。如果是不选中，其实密钥也导入了，只是不能再次被导出。这就保证了密钥的安全。

如果导入过程中没有选中这一项，做证书备份时“导出私钥”这一项是灰色的，不能选。只能导出cer格式的公钥。如果导入时选中该项，则在导出时“导出私钥”这一项就是可选的。

如果要导出私钥（pfx),是需要输入密码的，这个密码就是对私钥再次加密，这样就保证了私钥的安全，别人即使拿到了你的证书备份（pfx),不知道加密私钥的密码，也是无法导入证书的。相反，如果只是导入导出cer格式的证书，是不会提示你输入密码的。因为公钥一般来说是对外公开的，不用加密