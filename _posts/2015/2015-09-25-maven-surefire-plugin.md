---
title: Maven Surefire Plugin的基本使用
layout: post
categories: 工具
tags: Maven Surefire
comments: yes
---

<h2>Maven Surefire Plugin</h2>
Surefire是个测试框架。Maven提供了[Surefire插件](https://maven.apache.org/surefire/maven-surefire-plugin)，可以在应用构建生命周期的test阶段执行单元测试。

Surefire插件缺省会在`${basedir}/target/surefire-reports`目录下生成`txt`格式和`xml`格式的结果报告。

<h2>基本用法</h2>
要使用Surefire插件，只需要在`pom.xml`中定义如下内容就可以了：

	<project>
		[...]
		<build>
			<pluginManagement>
				<plugins>
					<plugin>
						<groupId>org.apache.maven.plugins</groupId>
						<artifactId>maven-surefire-plugin</artifactId>
						<version>2.18.1</version>
					</plugin>
				</plugins>
			</pluginManagement>
		</build>
		[...]
	</project>

然后运行下面的命令调用Surefire插件，就会运行应用中定义的单元测试了：

	mvn test

<h2>测试Provider</h2>
如果项目不作要求，大家编写单元测试的时候通常会选择自己常用的单元测试框架，常见的有：

* TestNG
* JUnit 3
* JUnit 4

那使用缺省的Surefire插件配置，能把上面几种单元测试都运行起来么？答案是否定的。

<h3>使用TestNG</h3>

首先在`pom.xml`中添加对TestNG的依赖：

	<dependencies>
	  [...]
	    <dependency>
	      <groupId>org.testng</groupId>
	      <artifactId>testng</artifactId>
	      <version>6.8.8</version>
	      <scope>test</scope>
	    </dependency>
	  [...]
	</dependencies>

TestNG缺省只运行`src/test/java`下以`Test`结尾的单元测试。如果类名有其他命名方式，需要用`includes`参数指定。


<h3>使用JUnit</h3>

和TestNG一样，首先要在`pom.xml`中添加对JUnit的依赖：

	<dependencies>
	  [...]
	    <dependency>
	      <groupId>junit</groupId>
	      <artifactId>junit</artifactId>
	      <version>4.11</version>
	      <scope>test</scope>
	    </dependency>
	  [...]
	</dependencies>

使用JUnit的时候，最好显式指定使用的Provider。下面示例是使用junit4：

	<plugins>
	[...]
	  <plugin>
	    <groupId>org.apache.maven.plugins</groupId>
	    <artifactId>maven-surefire-plugin</artifactId>
	    <version>2.18.1</version>
	    <dependencies>
	      <dependency>
	        <groupId>org.apache.maven.surefire</groupId>
	        <artifactId>surefire-junit4</artifactId>
	        <version>2.18.1</version>
	      </dependency>
	    </dependencies>
	  </plugin>
	[...]
	</plugins>

不指定的话，Surefire会按照下面的算法选择JUnit Provider版本：

1. 如果JUnit的版本大于等于4.7，并且配置了parallel属性，就使用junit47 provider
2. 如果JUnit的版本大于等于4.0，使用junit4 provider
3. 上述条件都不满足的话，使用junit3.8.1 provider

上面是Surefire文档里讲的，我自己测试的时候虽然指定了JUnit 4.11，但运行的时候仍然不会运行JUnit 4的测试用例，使用的还是junit3.8.1 provider。后来还是显式指定了一下JUnit Provider，所有的JUnit单元测试才运行起来。

<h2>测试用例的包含和排除</h2>

<h3>包含</h3>
Surefire插件的文档说会自动运行和下面模式匹配的测试类：

* `Test*.java`
* `*Test.java`
* `*TestCase.java`

没有一一测试，至少TestNG的测试类如果不是以Test结尾，Surefire插件是不会运行的。

如果测试类在指定Provider之后仍然不运行，可以使用includes包含进来。include既支持精确匹配，也支持正则表达式：

	<project>
	  [...]
	  <build>
	    <plugins>
	      <plugin>
	        <groupId>org.apache.maven.plugins</groupId>
	        <artifactId>maven-surefire-plugin</artifactId>
	        <version>2.18.1</version>
	        <configuration>
	          <includes>
	            <include>Sample.java</include>
	            <!-- %regex[正则表达式] -->
               <include>%regex[.*[Cat|Dog].*Test.*]</include>
	          </includes>
	        </configuration>
	      </plugin>
	    </plugins>
	  </build>
	  [...]
	</project>

<h3>排除</h3>

不运行的测试用例，可以使用excludes来排除：

	<project>
	  [...]
	  <build>
	    <plugins>
	      <plugin>
	        <groupId>org.apache.maven.plugins</groupId>
	        <artifactId>maven-surefire-plugin</artifactId>
	        <version>2.18.1</version>
	        <configuration>
	          <excludes>
	            <exclude>**/TestCircle.java</exclude>
	            <exclude>**/TestSquare.java</exclude>
	          </excludes>
	        </configuration>
	      </plugin>
	    </plugins>
	  </build>
	  [...]
	</project>

<h2>配置classpath</h2>

Surefire插件编译、运行测试用例时，classpath依次是：

1. test-classes目录
2. classes目录
3. 项目依赖
4. 补充的classpath

在不同环境下，classpath的设置可能不一样，编译、运行测试用例就会有不同的结果。比如项目A的测试用例需要使用项目B的类和资源，在IDE里项目A可以直接依赖项目B，运行项目A的测试用例是没有问题的，但在命令行下使用Surefire插件运行项目A的测试用例时，如果pom.xml里没有依赖项目B，编译、运行就会失败。这种情况下，可以配置classpath来确保测试用例的顺利执行：

	<project>
	  [...]
	  <build>
	    <plugins>
	      <plugin>
	        <groupId>org.apache.maven.plugins</groupId>
	        <artifactId>maven-surefire-plugin</artifactId>
	        <version>2.18.1</version>
	        <configuration>
	          <additionalClasspathElements>
	            <additionalClasspathElement>path/to/additional/resources</additionalClasspathElement>
	            <additionalClasspathElement>path/to/additional/jar</additionalClasspathElement>
	          </additionalClasspathElements>
	        </configuration>
	      </plugin>
	    </plugins>
	  </build>
	  [...]
	</project>