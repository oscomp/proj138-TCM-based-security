# proj138-TCM-based-security
UOS下基于TCM的系统安全设计


## 项目描述

现行通用个人计算机终端基于开放架构，绝大多数安全机制都以软件形式实现并建立在操作系统之上。而软件形式的安全机制主要是基于密码学算法和敏感信息存储来构建，就现存的个人计算机安全环境，密码学算法大都是使用国际通用算法（主要来自以美国为首的西方国家），敏感信息或多或少以某种形式暴露在内存中或者硬盘上，都存在一定的安全风险。目前，一种基于可信计算（TCM）的个人计算机体系框架成为了最佳的解决方案，它提供了独立国密算法密码学算力和敏感信息存储方式，从根上彻底解决了个人计算机安全问题。

平台配置寄存器（PCR）是TCM的基本特征之一。它的主要用途是提供一种（国密）密码学记录（度量）软件状态的方法，包括平台上运行的软件和该软件使用的配置数据。PCR更新计算被称为扩展，是一种单向哈希计算。这样的计算结果无法被破解。然后人们可以读取这些PCR的值来报告它们的状态。PCR也可以通过（国密）签名来返回一份安全报告。

《基于TCM的系统安全设计》是一个基于硬件、密码学（主要是国密）、操作系统设计的一个综合性题目，可以分解为下列三个独立的小题目：

题目1：完成基于可信计算TCM模块的TSS软件栈。
题目2：设计并完成基于TCM TSS软件栈的操作系统静态度量。
题目3：设计并完成基于国密算法的操作系统度量报告以及验证机制。


## 所属赛道

2022全国大学生操作系统比赛的“OS功能设计”赛道


## 参赛要求

- 以小组为单位参赛，最多三人一个小组，且小组成员是来自同一所高校的本科生（2022年春季学期或之后本科毕业的大一~大四的学生）
- 如学生参加了多个项目，参赛学生选择一个自己参加的项目参与评奖
- 请遵循“2022全国大学生操作系统比赛”的章程和技术方案要求

## 项目导师


## 难度

难

## 特征

- 熟悉可信计算模块TCM，以及相关标准；
- 熟悉国密算法，特别是签名算法；
- 熟悉静态度量相关知识；
- 熟悉操作系统架构，如：uos国产操作系统；

## 参考项目

- TPM TSS2软件栈

## 预期目标

注意：下面的内容是建议内容，不要求必须全部完成。选择本项目的同学也可与导师联系，提出自己的新想法，如导师认可，可加入预期目标

### 题目一

- 实现TCM模块需要的相关指令（TCM TSS软件栈）；
- 支持多进程多线程访问（代理模式）；

例子：

```c
#define TCM_ORD_Startup 0x00008099

uint32_t TCM_Startup()
{
	uint32_t ret;
	uint32_t ordinal_no = TCM_ORD_Startup;
	STACK_TCM_BUFFER(tcmdata)
	
	ret = TSS_buildbuff("00 c1 T L 00 01",&tcmdata,
	                             ordinal_no);
	if ((ret & ERR_MASK)) {
		return ret;
	}
	
	ret = TCM_Transmit(&tcmdata,"Startup");
	
	STORE32(tcmdata.buffer, 6, ret);
	
	return ret;
}

```

参考资料

- [1] 中华人民共和国密码行业标准. GM/T 0011 2012 可信计算可信密码支撑平台功能与接口规范 [S]. 北京：中国标准出版社, 2012.
- [2] 中华人民共和国密码行业标准. GM/T 0012 2012 可信计算 可信密码模块接口规范 [S]. 北京：中国标准出版社, 2012.
- [3] 中华人民共和国密码行业标准. GM/T 0013 2012 可信计算可信密码模块符合性检测规范 [S].北京：中国标准出版社, 2012.

### 题目二

IMA是内核的静态度量子系统，可对文件的度量值进行搜集、度量、存储和评估，需要和TCM结合使用。EVM是用来对文件元数据、扩展属性进行保护的模块，与IMA结合，防止文件被离线修改。静态度量方案基于IMA/EVM实现，主要的设计在于对IMA/EVM的使用。此方案主要通过分层模块化设计，总体分为四层，分别为内核层，系统层，服务层，应用层；

- 应用层：控制中心界面设置，以及静态度量策略文件管理
- 系统层：度量管理程序，主要用于逻辑控制，以及对外提供DBus接口；包安装程序，主要提供软件包安装过程填充扩展属性。
- 服务层：包的构建与签名，在软件构建的过程中，对包进行签名，并将文件以及对应的签名信息保存，以供后续包安装设置扩展属性做准备。
- 内核层：主要分为两部分，内核自带的ima/evm模块，在此基础上做修改，满足对文件的控制以及错误信息处理机制；uos_ima模块，uos度量安全模块，主要提供对扩展属性设置保护的功能，禁止未授权的程序对扩展属性修改。

名词解释：

- LSM，即Linux Security Module。在Linux内核中，LSM提供了一个可扩展的强制访问控制机制，各强制访问控制模块可以通过此机制实现自己的策略，对内核的各个操作进行限定，从而可以为系统提供不同的强制访问控制策略
- IMA，即Integrity Measurement Architecture，完整性度量。
- EVM，即Extended Verification Module，扩展属性验证模块。
- 内核模块：操作系统内核提供的扩展功能，以通过外部模块动态扩展操作系统内核的功能，内核模块的开发一般需要使用对应的内核头文件，以及gcc的C语言编译器。

参考资料：

- GB/T 20272-2006：《信息安全技术操作系统安全技术要求》 中华人民共和国标准化委员会 2006年5月
- GB/T 20271-2006：《信息安全技术信息系统通用安全技术要求》 中华人民共和国标准化委员会 2006年5月
- 王永吉，吴敬征等 隐蔽信道研究 软件学报 2010年9月
- 沈晴霓，卿斯汉 操作系统安全设计 机械工业出版社 2013年9月
- 李洋 防线:企业Linux安全运维理念和实战 清华大学出版社 2013年8月
- 李志 Linux内核安全模块深入剖析 机械工业出版社 2016年12月
- 软件工程 伊恩·萨默维尔 机械工业出版社 2018年2月
- IMA WIKI https://sourceforge.net/p/linux-ima/wiki/Home/
- SDB:Ima evm https://en.opensuse.org/SDB:Ima_evm
- EVM WIKI https://wiki.gentoo.org/wiki/Extended_Verification_Module

### 题目三

- 实现基于国密算法（SM2签名算法）的可信度量报告，并论证安全性；
- 可信度量报告支持第三方软件验证（如：UOS国产操作系统OpenSSL）;
- 支持操作系统实时度量，实时创建可信度量报告，性能优异（如：核心系统度量时间小于5分钟）；

参考资料：

- [1] 中华人民共和国密码行业标准. GM/T 0003 2012 SM2 椭圆曲线公钥密码算法第 2 部分：数字签名算法 [S]. 北京：中国标准出版社, 2012.
- [2] 中华人民共和国密码行业标准. GM/T 0003 2012 SM2 椭圆曲线公钥密码算法第 5 部分：参数定义 [S]. 北京：中国标准出版社, 2012.
- [3] 中华人民共和国密码行业标准. GM/T 0009 2012 SM2 密码算法使用规范 [S]. 北京：中国标准出版社, 2012.

