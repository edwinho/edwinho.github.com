---
author: edwin
layout: post
slug: block-ciphers-notes
title: "Block Ciphers 笔记"
categories:
- codes
tags:
- block_cipher
- cryptography
- rijndael
---

**Principles**

[Kerckhoffs' principle](http://en.wikipedia.org/wiki/Kerckhoffs%27s_principle):
加密算法是公开的，除了密钥。

[分组密码](http://en.wikipedia.org/wiki/Block_cipher)的经典算法有[Rijndael](http://en.wikipedia.org/wiki/Rijndael)和[Feistel](http://en.wikipedia.org/wiki/Feistel_cipher)。

<!-- more -->

**Goals of Cryptography**

Confidentiality(保密性)
Integrity(完整性)
Authentication(可认证性)

Block Ciphers 属于[对称密码](http://en.wikipedia.org/wiki/Symmetric-key_algorithm)，对称密码还有Stream Ciphers 和 Message Authentication
Codes (MACs).密码学就是用一个比较短的密钥加密一个很长的信息，绝对安全是不可能的，到现时为止，在复杂性理论安全算法方面，还没有找到令人满意的结果。


**理论攻击和真实攻击**
Example:一个256位密钥的加密算法
真实攻击：穷举全部2256个可能的密钥
理论攻击：找到一个方法可以将理论复杂度降到或等于一个比2256更低的值

**攻击类型**
唯密文攻击
已经明文攻击
选择明文攻击
相关密钥攻击

**[香农](http://en.wikipedia.org/wiki/Claude_Shannon)定理**
Confusion(混淆性)
Diffusion(扩散性)

**设计原则：**
Substitution(代换)
Permutation(置换)

现代密码学都是根据香农定理设计的，再经过多轮迭代。从而形成SP-Networks，Feistel算法就是一个经典的例子。



**Modes of operation:**
Electronic Code Book
Cipher Block Chaining:像链一样，前面的密文块会影响后面的密文块。
Counter Mode:流密码常用的模式

参考书籍：

《[The Block Cipher Companion](http://www.amazon.cn/s/ref=nb_sb_noss?__mk_zh_CN=%E4%BA%9A%E9%A9%AC%E9%80%8A%E7%BD%91%E7%AB%99&url=search-alias%3Daps&field-keywords=block+cipher+companion)》

[singlepic id=4 w=332 h=500 float=]
