---
title: Maven基础应用
date: 2019-01-14 21:26:16
tags: 工具
---

### 常用标签

#### mirror标签配置私服

~~~xml
<mirrors>
    <mirror>   
        <id>ylcloud-repo</id>   
        <mirrorOf>ylcloud-repo</mirrorOf>   
        <url>http://ylcloud-repo:5001/repository/maven-public/</url>   
    </mirror> 
</mirrors>

~~~

#### proxy标签配置代理

~~~xml
<proxies>
  <proxy>
      <id>xxx</id>
      <active>true</active>
      <protocol>http</protocol>
      <username>用户名</username>
      <password>密码</password>
      <host>代理服务器地址</host>
      <port>代理服务器的端口</port>
      <nonProxyHosts>不使用代理的主机</nonProxyHosts>
  </proxy>
</proxies>

~~~



#### pluginGroups插件

~~~xml
<pluginGroups>
    <!-- pluginGroup
     | Specifies a further group identifier to use for plugin lookup.
    <pluginGroup>com.your.plugins</pluginGroup>
    -->
</pluginGroups>
~~~



#### servers配置私服用户名密码等

~~~xml
<servers>
    <server>
      <id>id</id>
      <username>用户名</username>
      <password>密码</password>
    </server>
  </servers>
~~~



#### profiles，activeProfile配置不同环境

~~~xml
<profiles>
    <profile>  
        <id>ylcloud</id>  
        <repositories>  
            <repository>  
                <id>ylcloud-repo</id>  
                <url>http://ylcloud-repo</url>  
                <releases><enabled>true</enabled></releases>  
                <snapshots><enabled>true</enabled></snapshots>  
            </repository>  
        </repositories>  
        <pluginRepositories>  
            <pluginRepository>  
                <id>ylcloud-repo</id>  
                <url>http://ylcloud-repo</url>  
                <releases><enabled>true</enabled></releases>  
                <snapshots><enabled>true</enabled></snapshots>  
            </pluginRepository>  
        </pluginRepositories>  
    </profile>  
    <profile>  
        <id>backup</id>  
        <repositories>  
            <repository>  
                <id>backup</id>  
                <url>http://repo1.maven.org/maven2/</url>   
                <releases><enabled>true</enabled></releases>  
                <snapshots><enabled>true</enabled></snapshots>  
            </repository>  
        </repositories>  
        <pluginRepositories>  
            <pluginRepository>  
                <id>backup</id>  
                <url>http://repo1.maven.org/maven2/</url>   
                <releases><enabled>true</enabled></releases>  
                <snapshots><enabled>true</enabled></snapshots>  
            </pluginRepository>  
        </pluginRepositories>  
    </profile>  
</profiles>

<activeProfiles>  
      <activeProfile>ylcloud</activeProfile>  
  </activeProfiles>

~~~





#### packaging

> jar：打成jar包，springboot项目一般多用jar包启动
>
> war：部署到tomcat下
>
> pom：聚合工程
>
> maven-plugin



#### dependencyManagement

>  只能出现在父pom工程，不是给子类直接用的，需要使用时还是需要自己引用，类似抽象类，可以对版本进行控制

#### scope

>compile 编译（默认）
>
>test 测试
>
>provided 编译，例如：Servlet，打包时不会打进去
>
>runtime 运行时有效，编译时可以不用，jdbc驱动实现
>
>system 本地一些jar加上systemPath



#### 依赖仲裁

> 最短路径，pom书写顺序，exclusions排除包



####  生命周期

> clean：pre-clean,clean,post-clean
>
> default：compile,package,install,deploy....
>
> site：pre-site,site,post-site,site-deploy

​	

#### 常用命令

> clean:删除target
>
> compile:编译
>
> test:运行项目中所有的测试用例
>
> install:	将模块安装到本地仓库
>
> package:打包
>
> deploy:把本地jar发布到远程仓库中

​	

#### 常用插件

> mojo，findbugs，versions，source



mvn dependency:tree > d.txt

mvn clean package -U 强制从版本库拉一次

maven-model-builder-3.3.9.jar/org/apache/maven/model中有超级pom