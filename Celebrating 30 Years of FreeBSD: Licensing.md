# 庆祝 FreeBSD 成立 30 周年：许可证

- 译者：ykla

**今天的主题：许可证**

在决定新项目要使用什么许可证时，有许多许可证可供选择。今天，我们将专注于 Berkeley （伯克利）软件发行（BSD）许可证的 2 条款变体，也被称为 FreeBSD 许可证。对于许多人来说，商业友好的 FreeBSD 许可证提供了他们所寻求的灵活性。为什么呢？首先让我们看一下许可证本身。

**FreeBSD 许可证**
```
Copyright <YEAR> <COPYRIGHT HOLDER>

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.

2. Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS “AS IS” AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.”
```

参考译文：

```
FreeBSD 许可证

版权所有（年份）（版权所有者）

在包括但不限于修改的源代码和二进制形式的重新分发和使用是允许的，前提是满足以下条件：

1. 源代码的再分发必须保留上述版权声明、条件列表和以下免责声明。

2. 二进制形式的再分发必须在文档和/或其他提供的材料中复制上述版权声明、条件列表和以下免责声明。

本软件按原样提供，版权所有者和贡献者对于任何直接、间接、偶然、特殊、示例或后果性损害（包括但不限于替代商品或服务的获取、使用数据或利润损失、业务中断）不负任何责任，无论是在合同、严格责任或侵权行为（包括疏忽或其他）的任何理论下，即使事先已被告知可能发生此类损害。
```


换句话说，FreeBSD 许可证不要求派生源代码使用与原始代码相同的许可证条款。这意味着 FreeBSD 可以作为其他项目和专有软件的组件使用，而无需共享派生代码。相比之下，Linux 是根据 GNU 通用公共许可证（GPL）许可的。GPL 也被称为“共享协议”，是相互的，意味着 GPL 项目的派生代码也必须以相同的许可证条款提供。

FreeBSD 及其组件可以用于构建专有软件，没有任何限制。从法律角度来看，这也对开发者具有吸引力。你可以随心所欲地使用（他的）代码。

此外，FreeBSD 许可证已被自由软件基金会验证为与 GPL 兼容，也经过了开放源代码促进会【注1】的审查，被确认为开源许可证。这意味着 FreeBSD 仍然是作为开发产品的参考平台的绝佳选择。FreeBSD 的代码具有最广泛的适用性。你可以从 FreeBSD 入手，然后在任何地方发布。

像 Nextflix 和他们开发的 Open Connect Appliances【Open Connect Appliance 软件内置了一个 FreeBSD 操作系统和 NGINX 服务器，许可证是 BSD】这样的公司，最初就是因为宽松的许可证才吸引了 Open Connect 工程师使用 FreeBSD【注2】。

其他公司，例如 Beckhoff 之所以选择 FreeBSD 和宽松的许可证，部分原因是因为 GPL【注3】不能满足要求。

简而言之，BSD 许可证，尤其是 FreeBSD 许可证，在构建产品和服务时提供了灵活性和安心感。

1.-BSD 许可证维基百科页面-https://en.wikipedia.org/wiki/BSD_licenses

2.-FreeBSD 案例研究：Netflix-https://freebsdfoundation.org/blog/freebsd-case-study-netflix/

3.-2020 年 FreeBSD 供应商峰会：Beckhoff-https://youtu.be/8LUdZseNrpE

---

原文地址：https://freebsdfoundation.org/blog/celebrating-30-years-of-freebsd-licensing/
