---
layout: post
title: "如何发布自己的 artifacts 到 maven 中央仓库"
date: 2020-10-28 09:16:37 +0800
comments: true
categories: [Archives, Web Development]
keywords: publish, deploy, artifact, maven
description: How to Publish Your Artifacts to Maven Central
---

当我们越来越熟悉 Java 开发，可能也想发布自己的 artifacts 到 maven 中央仓库，那么该如何做呢？  

>The easiest way to upload another project is to use the [Open Source Software Repository Hosting (OSSRH)](http://central.sonatype.org/pages/ossrh-guide.html), which is an approved repository provided by Sonatype for any OSS Project that want to get their artifacts into the Central Repository.

从 maven 的官方文档可知是使用 [Open Source Software Repository Hosting (OSSRH)](http://central.sonatype.org/pages/ossrh-guide.html), 于是我们可以参考她的指南。  

这份指南勾勒了发布工作的主要流程，分别是：  

1. Create a ticket with Sonatype
2. Review Requirements
3. Deployment
4. Releasing to Central

###创建 Sonatype 工单

Sonatype 使用 JIRA 管理请求，所以我们要创建 JIRA 帐号, 然后创建一个新工程工单。  

1. [Create your JIRA account](https://issues.sonatype.org/secure/Signup!default.jspa)  
2.	 [Create a New Project ticket](https://issues.sonatype.org/secure/CreateIssue.jspa?issuetype=21&pid=10134)  

###复核要求

为了确保中央仓库库中可用组件的最低质量水平，部署组件必须满足一些要求。这使组件的用户能够从中央仓库中提供的元数据中找到有关组件的所有相关细节。

这些要求是：  

1. Supply Javadoc and Sources
2. Sign Files with GPG/PGP
3. Sufficient Metadata
<!--more-->
####Supply Javadoc and Sources

我们可以集成 maven 插件来提供 Javadoc 和源文件。  

{% codeblock %}
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-source-plugin</artifactId>
	<version>2.2.1</version>
	<executions>
		<execution>
			<id>attach-sources</id>
			<goals>
				<goal>jar-no-fork</goal>
			</goals>
		</execution>
	</executions>
</plugin>
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-javadoc-plugin</artifactId>
	<version>2.9.1</version>
	<executions>
		<execution>
			<id>attach-javadocs</id>
			<goals>
				<goal>jar</goal>
			</goals>
		</execution>
	</executions>
</plugin>
{% endcodeblock %}

####Sign Files with GPG/PGP

我们需要用 GPG/PGP 对文件签名，以 macOS 为例：

{% codeblock %}
#1. Install GPG
brew install gpg

#2. Generate the key pair
gpg --full-gen-key

###3.Publish the GPG key pair and distribute your key to GPG servers

gpg --keyserver hkps.pool.sks-keyservers.net  --send-keys your_key_id

{% endcodeblock %}

生成 GPG 公私钥对并发布到 GPG 服务器后，我们还需要将 GPG 提供给 maven, 这是通过 maven settings。   

{% codeblock %}
# ~/.m2/settings.xml
<settings>
  <profiles>
    <profile>
      <id>ossrh</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <properties>
        <gpg.passphrase>the_pass_phrase</gpg.passphrase>
      </properties>
    </profile>
  </profiles>
</settings>
{% endcodeblock %}

macOS 上我还遇到了 gpg: signing failed: Inappropriate ioctl for device 这个错误，通过在 `~/.bash_profile` 中加入如下配置解决了:  

{% codeblock %}
GPG_TTY=$(tty)
export GPG_TTY
{% endcodeblock %}

####Sufficient Metadata

元数据信息要包换：  

* Correct Coordinates
* Project Name, Description and URL
* License Information
* Developer Information
* SCM Information

对于 Correct Coordinates， 我这里是使用 github 托管代码，于是我们可以按照 maven 官方文档的建议:  

>My project is hosted at a project hosting service like SourceForge or Github, what should I use as groupId?
>If your project name is foo at SourceForge, you can use net.sf.foo. If your username is bar on Github, you can use com.github.bar. You can also use another reversed domain name you control. The group ID does not have to reflect the project host.

###Deployment

至此，准备工作就完成了，接下来可以进入部署环节了。先还是要做些相关配置，主要是：  

* Distribution Management and Authentication
* Using a Profile

####Distribution Management and Authentication

为了配置 Maven，使其能够通过 Nexus Staging Maven 插件部署到 OSSRH 的 Nexus Repository Manager上，你必须进行如下配置：  

{% codeblock %}
<distributionManagement>
  <snapshotRepository>
    <id>ossrh</id>
    <url>https://oss.sonatype.org/content/repositories/snapshots</url>
  </snapshotRepository>
</distributionManagement>
<build>
  <plugins>
    <plugin>
      <groupId>org.sonatype.plugins</groupId>
      <artifactId>nexus-staging-maven-plugin</artifactId>
      <version>1.6.7</version>
      <extensions>true</extensions>
      <configuration>
        <serverId>ossrh</serverId>
        <nexusUrl>https://oss.sonatype.org/</nexusUrl>
        <autoReleaseAfterClose>true</autoReleaseAfterClose>
      </configuration>
    </plugin>
    ...
  </plugins>
</build>
{% endcodeblock %}

另外，如果你使用 Maven 部署插件，这是默认行为，你需要添加一个完整的distributionManagement部分。  

{% codeblock %}
<distributionManagement>
  <snapshotRepository>
    <id>ossrh</id>
    <url>https://oss.sonatype.org/content/repositories/snapshots</url>
  </snapshotRepository>
  <repository>
    <id>ossrh</id>
    <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
  </repository>
</distributionManagement>
{% endcodeblock %}

上述配置将从你的Maven settings.xml文件中获取用户账户的详细信息来部署到OSSRH。认证的最小设置是：  

{% codeblock %}
<settings>
  <servers>
    <server>
      <id>ossrh</id>
      <username>your-jira-id</username>
      <password><![CDATA[your-jira-pwd]]></password>
    </server>
  </servers>
</settings>
{% endcodeblock %}

####Using a Profile

由于生成javadoc和源代码jars以及使用GPG签署组件是一个相当耗时的过程，这些执行通常从正常的构建配置中分离出来，并转移到一个配置文件中。然后，当通过激活配置文件进行部署时，该配置文件又会被使用。

{% codeblock %}
<profiles>
  <profile> 
    <id>release</id>
    <build>
      ...
      javadoc, source and gpg plugin from above
      ...
    </build>
  </profile>
</profiles>
{% endcodeblock %}

这些配置做完之后，最终得到的 pom.xml 类似如下：  

{% codeblock %}
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.github.damiansheldon</groupId>
	<artifactId>treasure</artifactId>
	<version>0.0.2</version>
	<build>
		<sourceDirectory>src</sourceDirectory>
		<plugins>
			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.8.1</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-deploy-plugin</artifactId>
				<version>2.8.2</version>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-release-plugin</artifactId>
				<version>2.5.3</version>
				<configuration>
					<arguments>-Dgpg.passphrase=${gpg.passphrase}</arguments>
					<autoVersionSubmodules>true</autoVersionSubmodules>
					<useReleaseProfile>false</useReleaseProfile>
					<releaseProfiles>release</releaseProfiles>
					<goals>deploy</goals>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.sonatype.plugins</groupId>
				<artifactId>nexus-staging-maven-plugin</artifactId>
				<version>1.6.7</version>
				<extensions>true</extensions>
				<configuration>
					<serverId>ossrh</serverId>
					<nexusUrl>https://oss.sonatype.org/</nexusUrl>
					<autoReleaseAfterClose>true</autoReleaseAfterClose>
				</configuration>
			</plugin>
		</plugins>
	</build>
	<dependencies>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jdbc</artifactId>
			<version>5.2.8.RELEASE</version>
		</dependency>
	</dependencies>
	<distributionManagement>
		<repository>
			<id>ossrh</id>
			<name>OSSRH Staging Repository</name>
			<url>https://oss.sonatype.org/service/local/staging/deploy/maven2</url>
		</repository>
		<snapshotRepository>
			<id>ossrh</id>
			<name>OSSRH Snapshots</name>
			<url>https://oss.sonatype.org/content/repositories/snapshots</url>
		</snapshotRepository>
	</distributionManagement>
	<scm>
		<connection>scm:git:https://github.com/DamianSheldon/Treasure.git</connection>
		<developerConnection>scm:git:https://github.com/DamianSheldon/Treasure.git</developerConnection>
		<tag>HEAD</tag>
		<url>https://github.com/DamianSheldon/Treasure</url>
	</scm>
	<profiles>
		<profile>
			<id>release</id>
			<build>
				<plugins>
					<plugin>
						<groupId>org.apache.maven.plugins</groupId>
						<artifactId>maven-source-plugin</artifactId>
						<version>2.2.1</version>
						<executions>
							<execution>
								<id>attach-sources</id>
								<goals>
									<goal>jar-no-fork</goal>
								</goals>
							</execution>
						</executions>
					</plugin>
					<plugin>
						<groupId>org.apache.maven.plugins</groupId>
						<artifactId>maven-javadoc-plugin</artifactId>
						<version>2.9.1</version>
						<executions>
							<execution>
								<id>attach-javadocs</id>
								<goals>
									<goal>jar</goal>
								</goals>
							</execution>
						</executions>
					</plugin>
					<plugin>
						<groupId>org.apache.maven.plugins</groupId>
						<artifactId>maven-gpg-plugin</artifactId>
						<version>1.5</version>
						<executions>
							<execution>
								<id>sign-artifacts</id>
								<phase>verify</phase>
								<goals>
									<goal>sign</goal>
								</goals>
							</execution>
						</executions>
					</plugin>
				</plugins>
			</build>
		</profile>
	</profiles>

	<!-- More Project Information -->
	<name>Treasure</name>
	<description>The treasures I have accumulated in my daily java development. It can help other people face the same problem.</description>
	<url>https://github.com/DamianSheldon/Treasure</url>
	<licenses>
		<license>
			<name>MIT License</name>
			<url>http://www.opensource.org/licenses/mit-license.php</url>
			<distribution>repo</distribution>
		</license>
	</licenses>
	<developers>
		<developer>
			<id>meiliang</id>
			<name>Meiliang Dong</name>
			<email>dongmeilianghy@sina.com</email>
			<url>http://damiansheldon.github.io</url>
			<roles>
				<role>architect</role>
				<role>developer</role>
			</roles>
			<timezone>Asia/Shanghai</timezone>
		</developer>
	</developers>
</project>
{% endcodeblock %}

做好相关配置之后就可以真正部署了，主要有两种部署方式：  

* Nexus Staging Maven Plugin for Deployment and Release
* Performing a Release Deployment with the Maven Release Plugin

推荐的方式是使用 Nexus Staging Maven Plugin。  

####Nexus Staging Maven Plugin for Deployment and Release

Performing a Snapshot Deployment

当你的版本以`-SNAPSHOT`结尾时，会进行快照部署。当执行快照部署时，您不需要满足要求，只需在工程上运行 `mvn clean deploy`

SNAPSHOT版本不同步到中央版本库。如果您希望您的用户使用您的 SNAPSHOT 版本，他们需要将快照库添加到他们的 Nexus Repository Manager、settings.xml 或 pom.xml 中。成功部署的SNAPSHOT版本可以在`https://oss.sonatype.org/content/repositories/snapshots/`找到。

Performing a Release Deployment

为了执行发布部署，你必须在所有的POM文件中编辑你的版本，以使用发布版本。这意味着它们不能以`-SNAPSHOT`结尾，此外插件和依赖性声明也不能使用快照版本。这保证了你只能依赖其他发布的组件。理想情况下，它们都在中央仓库中可用。这确保了你的用户可以从中央仓库中检索你的组件以及你的过渡性依赖。

在多模块设置中，可以手动或借助Maven版本插件来更改项目的版本和父级引用。  

{% codeblock %}
mvn versions:set -DnewVersion=1.2.3
{% endcodeblock %}

一旦你更新了所有的版本，并确保你的构建没有部署就通过了，你就可以使用发布配置文件进行部署，并使用

{% codeblock %}
mvn clean deploy -P release
{% endcodeblock %}

{% codeblock %}
{% endcodeblock %}

####Performing a Release Deployment with the Maven Release Plugin

Maven发布插件可以用来自动完成对Maven POM文件的修改、健康检查、所需的SCM操作和实际部署执行。

Maven发布插件的配置应该包括禁用Maven super POM 中的发布配置文件，因为我们使用的是我们自己的配置文件，并在激活发布配置文件的同时指定部署目标。

{% codeblock %}
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-release-plugin</artifactId>
  <version>2.5.3</version>
  <configuration>
    <autoVersionSubmodules>true</autoVersionSubmodules>
    <useReleaseProfile>false</useReleaseProfile>
    <releaseProfiles>release</releaseProfiles>
    <goals>deploy</goals>
  </configuration>
</plugin>
{% endcodeblock %}

在SCM连接配置正确的情况下，您可以通过以下方式向OSSRH进行发布部署。

{% codeblock %}
mvn release:clean release:prepare
{% endcodeblock %}

回答版本和标签的提示，然后是

{% codeblock %}
mvn release:perform
{% endcodeblock %}

由于使用了Nexus Staging Maven Plugin，并将autoReleaseAfterClose设置为true，这个执行将一次性部署到OSSRH并发布到中央仓库。

###Releasing to Central

在前面的介绍中，我们提到使用 Nexus Staging Maven Plugin，并将autoReleaseAfterClose设置为true，部署到OSSRH后会发布到中央仓库。我们也可以手动执行 `mvn nexus-staging:release
` 来发布 staging repository。  

###Reference 

* [OSSRH Guide](https://central.sonatype.org/pages/ossrh-guide.html)  
* [Requirements](https://central.sonatype.org/pages/requirements.html)  
* [Deploying to OSSRH with Apache Maven](https://central.sonatype.org/pages/apache-maven.html#nexus-staging-maven-plugin-for-deployment-and-release)  
* [How to Publish Your Artifacts to Maven Central](https://dzone.com/articles/publish-your-artifacts-to-maven-central)  
* [“gpg: signing failed: Inappropriate ioctl for device” on MacOS with Maven](https://stackoverflow.com/questions/57591432/gpg-signing-failed-inappropriate-ioctl-for-device-on-macos-with-maven)  


