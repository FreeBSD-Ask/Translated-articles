# FreeBSD 企业工作组首次会议总结

- 原地址：[Recap of first meeting of the FreeBSD Enterprise Working Group](https://freebsdfoundation.org/blog/recap-of-first-meeting-of-the-freebsd-enterprise-working-group/)
- 发布日期：2023-8-24
- 译者：ykla & ChatGPT

这个工作组的成立始于长期使用 FreeBSD 的用户 Michael Osipov，他花时间向基金会提供了关于他在使用 FreeBSD 时遇到的阻碍的详细笔记。

接下来的步骤是评估兴趣——我在 [LinkedIn](https://www.linkedin.com/posts/gtewallace_enterprise-freebsd-opensource-activity-7084607866363359234-YMZx?utm_source=share&utm_medium=member_desktop) 和 Discord 上发布了招募兴趣的信息，反应非常好。有 40 人举手加入了工作组。以下是这 40 人按角色划分的情况。

![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/55c74769-4274-4d76-b539-c71108a6bf12)


## 着手进行业务
幸运的是，有 13 人，包括我在内，参加了 8 月 16 日举行的企业工作组的第一次会议。以下是会议回顾。

在整理日常事务后，包括提醒遵守 FreeBSD [行为准则](https://www.freebsd.org/internal/code-of-conduct/)，我们批准了工作组章程，并继续讨论了现有的差距。


![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/32792fb2-3260-46f3-85e9-04cf5c69878d)

![image](https://github.com/FreeBSD-Ask/Translated-articles/assets/10327999/af2ccd45-9c5f-4232-b350-fe54caf7552e)


在此过程中，我们就以下几个领域展开了愉快的讨论：



## 科学计算
讨论的第一个重点涉及相近但我认为稍有不同的科学计算用例。这是很好的时机，因为就在那周早些时候，我与长期使用 FreeBSD 的用户和社区成员进行通话，他正在努力改进 FreeBSD 在这个领域的应用。通话结束后，我将所有人联系在了一起，我的基金会同事 Joe Mingrone 还加入了在这个领域工作的一些 Port 维护者。如果你有兴趣在这个领域提供帮助，请让我知道，我会将你与合适的人联系起来。

## Samba
下一个讨论涉及加强对 Samba 的支持。Allan Jude 提到 Klara 正在开发一个特殊的 Samba 模块，以便在上游使用，从而使其能够利用新的块引用树 ZFS 功能。要在 Samba CI 系统中获得更通用的 FreeBSD 上游支持，我们需要在 Samba 中引入 FreeBSD 的工作人员。下一步是从社区中寻找愿意在 FreeBSD 支持上游方面工作的志愿者。如果你可以提供帮助，请与我联系。

## 云原生
一位与会者评论说，在所有议题中，在他们看来云原生是最重要的，可能比其他所有议题加在一起还要重要。Allan 向大家介绍了一个名为 XC 的新项目：<https://github.com/michael-yuji/xc> 其他人询问了这与 runj 有什么关系。开放容器倡议社区正在为 FreeBSD 提供支持，一些企业工作组的参与者表示有兴趣提供帮助。谢谢大家！

## 高端人工智能/高性能计算 Nvidia GPU 驱动
一名在数据中心服务器制造商工作的参与者询问了 FreeBSD 中用于人工智能和高性能计算应用（例如 Nvidia h100/a100）的 Nvidia GPU 驱动的情况。

## 添加到差距清单
人们提出了以下要添加到差距清单的项目：

- eBPF
- 零信任 FreeBSD 构建
- 可重复的 FreeBSD 构建
- FreeBSD 的软件供应链和组成清单（SBOM）

## 关于配置管理的问题
一名与会者问了 SALT、Puppet 和 Ansible 对 FreeBSD 的支持情况。

这三者在生产中都被使用，并且运行良好——Ansible 和 SALT 似乎更受欢迎。以下是一些资源：

- SALT
  - 在 FreeBSD 上安装：<https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/freebsd.html>
  - 使用 SALT：<https://papers.freebsd.org/2019/bsdcan/hendryxparker-masters_and_minions_or_the_dream_of_bsd_automation/>
- Ansible
  - 在 BSD 主机上使用 Ansible：<https://docs.ansible.com/ansible/latest/os_guide/intro_bsd.html>
  - 快速入门指南：<https://blog.andreev.it/2021/02/ansible-quick-start-guide-for-freebsd-centos-and-ubuntu/>
  - 使用 Ansible Playbooks 自动部署 FreeBSD 上的各种服务
- Puppet
  - <https://wiki.freebsd.org/Puppet/GettingStarted/>

你可以在此处查看会议的录像：[FreeBSD 基金会企业工作组会议＃1](https://youtu.be/0TQUlipFvXM)

如果你对此感兴趣，并且希望提供帮助，[请填写此 Google 表单](https://forms.gle/beKQFL9nBpG11CFB7)，帮助使 FreeBSD 成为一个更好的通用企业操作系统。
