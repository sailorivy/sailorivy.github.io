---
title: 在Mac上使用tree命令
layout: post
categories: 工具
tags: Mac tree
comments: yes
---

在终端上显示一个目录的结构可以使用tree命令。在Ubuntu上安装tree非常方便：


    sudo apt-get install tree
然后就可以在终端里查看一个目录的结构： 

	.
	└── org
		└── dmg
			└── pmml
				├── pmml_4_1
				│   ├── bindings.xjb
				│   ├── bindings.xml
				│   └── pmml-4-1.xsd
				└── pmml_4_2
					├── bindings.xjb
					├── bindings.xml
					└── pmml-4-2.xsd
	
	5 directories, 6 files
					
那要在Mac下使用tree该怎么做呢？

最实用的方法就是编译、安装tree源码的
 
1. 下载源码：curl -O ftp://mama.indstate.edu/linux/tree/tree-1.7.0.tgz 
2. 解压源码：tar xzvf tree-1.7.0.tgz
3. 进入tree-1.7.0目录：cd tree-1.7.0
4. 修改Makefile文件：vi Makefile

tree源码的默认编译环境是Linux，在Mac里面要注释掉Linux的编译选项、启用Mac的编译选项

	# Linux defaults:
	#CFLAGS=-ggdb -Wall -DLINUX -D_LARGEFILE64_SOURCE -D_FILE_OFFSET_BITS=64
	#CFLAGS=-O4 -Wall  -DLINUX -D_LARGEFILE64_SOURCE -D_FILE_OFFSET_BITS=64
	#LDFLAGS=-s

	# Uncomment for OS X:
	CC=cc
	CFLAGS=-O2 -Wall -fomit-frame-pointer -no-cpp-precomp
	LDFLAGS=
	MANDIR=/usr/share/man/man1
	OBJS+=strverscmp.o

接下来make，然后[sudo] make install就可以了。


