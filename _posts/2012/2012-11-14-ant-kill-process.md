---
title: 用Ant强杀Java进程
layout: post
categories: 工具
tags: Ant
comments: yes
---
  
强杀进程的方式有很多。如果应用程序需要在多个平台下运行，我们可以为不同的平台编写不同的脚本，然后在整体构建/运行的脚本里去分别调用，但这并不太利于维护。我们可以借助Ant、采用统一的方式去完成这件事情。

以强杀Java进程为例，主要思路：

 1. 使用Java自带的jps命令得到所有Java进程的信息
 2. 获得符合要求的Java进程的PID
 3. 调用强杀进程的命令杀死进程

Ant脚本片段：

    <property environment="env"/>
    <target name="kill-process">  
        <!-- 执行Java自带的jps命令，将所有Java进程的信息（包括PID）写入pid.out文件 -->  
        <exec executable="${env.JAVA_HOME}/bin/jps" output="pid.out">  
            <!-- 用jps命令的参数v，可以获取Java进程的变量信息。  
                 如果多个Java进程需要通过变量信息区分，这个参数很有用-->  
            <arg value="-v"/>  
        </exec>  
      
        <!-- 加载pid.out文件，用filterchain定义条件、得到符合条件的PID -->  
        <loadfile srcfile="pid.out" property="pid">  
            <filterchain>  
                <linecontains>  
                    <contains value="Bootstrap"/>  
                </linecontains>  
                <tokenfilter>  
                    <replaceregex pattern="^(\d+) Bootstrap (.*)" replace="\1"/>  
                    <trim/>  
                    <ignoreblank/>  
                </tokenfilter>  
                <striplinebreaks/>  
            </filterchain>  
        </loadfile>  
      
        <condition property="haveValue">  
            <isset property="pid"/>  
        </condition>  
      
        <antcall target="pidFound"/>  
        <antcall target="pidNotFound"/>  
      
        <delete file="pid.out"/>  
    </target>
    <target name="pidFound" if="haveValue">  
        <echo>Killing process with PID: ${pid}</echo>  
        <!-- 执行强杀进程的命令，这里以Windows自带的tskill为例 -->  
        <exec executable="tskill">  
            <arg value="${pid}"/>  
        </exec>
    </target>
    <target name="pidNotFound" unless="haveValue">  
        <echo>There is no matched process.</echo>
    </target> 


