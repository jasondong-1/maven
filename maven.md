<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [maven 将模块和依赖分别打到不同目录](#maven-%E5%B0%86%E6%A8%A1%E5%9D%97%E5%92%8C%E4%BE%9D%E8%B5%96%E5%88%86%E5%88%AB%E6%89%93%E5%88%B0%E4%B8%8D%E5%90%8C%E7%9B%AE%E5%BD%95)
- [maven dependency 顺序的影响](#maven-dependency-%E9%A1%BA%E5%BA%8F%E7%9A%84%E5%BD%B1%E5%93%8D)
- [向maven中添加本地依赖  支持mvn clean package](#%E5%90%91maven%E4%B8%AD%E6%B7%BB%E5%8A%A0%E6%9C%AC%E5%9C%B0%E4%BE%9D%E8%B5%96--%E6%94%AF%E6%8C%81mvn-clean-package)
- [maven help 插件](#maven-help-%E6%8F%92%E4%BB%B6)
- [mvn 运行程序](#mvn-%E8%BF%90%E8%A1%8C%E7%A8%8B%E5%BA%8F)
- [浏览项目的依赖](#%E6%B5%8F%E8%A7%88%E9%A1%B9%E7%9B%AE%E7%9A%84%E4%BE%9D%E8%B5%96)
- [mvn 忽略测试失败](#mvn-%E5%BF%BD%E7%95%A5%E6%B5%8B%E8%AF%95%E5%A4%B1%E8%B4%A5)
- [跳过单元测试](#%E8%B7%B3%E8%BF%87%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95)
- [mvn 离线多线程](#mvn-%E7%A6%BB%E7%BA%BF%E5%A4%9A%E7%BA%BF%E7%A8%8B)
- [mvn scope](#mvn-scope)
- [分析依赖](#%E5%88%86%E6%9E%90%E4%BE%9D%E8%B5%96)
- [pom可以引用的变量](#pom%E5%8F%AF%E4%BB%A5%E5%BC%95%E7%94%A8%E7%9A%84%E5%8F%98%E9%87%8F)
- [依赖版本界限](#%E4%BE%9D%E8%B5%96%E7%89%88%E6%9C%AC%E7%95%8C%E9%99%90)
- [将一组公用的依赖放到一个pom中](#%E5%B0%86%E4%B8%80%E7%BB%84%E5%85%AC%E7%94%A8%E7%9A%84%E4%BE%9D%E8%B5%96%E6%94%BE%E5%88%B0%E4%B8%80%E4%B8%AApom%E4%B8%AD)
- [多模块vs继承](#%E5%A4%9A%E6%A8%A1%E5%9D%97vs%E7%BB%A7%E6%89%BF)
- [maven resource](#maven-resource)
- [maven profiles](#maven-profiles)
- [maven profiles 的激活](#maven-profiles-%E7%9A%84%E6%BF%80%E6%B4%BB)
- [settings profile](#settings-profile)
- [assembly ,允许用户个性化设置的插件](#assembly-%E5%85%81%E8%AE%B8%E7%94%A8%E6%88%B7%E4%B8%AA%E6%80%A7%E5%8C%96%E8%AE%BE%E7%BD%AE%E7%9A%84%E6%8F%92%E4%BB%B6)
- [自己搭建maven仓库,nexus](#%E8%87%AA%E5%B7%B1%E6%90%AD%E5%BB%BAmaven%E4%BB%93%E5%BA%93nexus)
- [高可移植性之多模块](#%E9%AB%98%E5%8F%AF%E7%A7%BB%E6%A4%8D%E6%80%A7%E4%B9%8B%E5%A4%9A%E6%A8%A1%E5%9D%97)
- [本地依赖的改进](#%E6%9C%AC%E5%9C%B0%E4%BE%9D%E8%B5%96%E7%9A%84%E6%94%B9%E8%BF%9B)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

### maven 将模块和依赖分别打到不同目录  
```xml
<plugin>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.0.2</version>
    <configuration>
        <!-- 指定打包的jar包输出路径-->
        <outputDirectory>
            ${user.dir}/../lib2
            <!--${project.build.directory}/lib2-->
        </outputDirectory>
        <!--不打入jar包的文件类型或者路径-->
        <!--<excludes>
          <exclude>**/*.properties</exclude>
          <exclude>**/*.xml</exclude>
          <exclude>**/*.yml</exclude>
          <exclude>static/**</exclude>
          <exclude>templates/**</exclude>
        </excludes>-->
    </configuration>
</plugin>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>3.1.1</version>
    <executions>
        <execution>
            <id>copy-dependencies</id>
            <phase>package</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
            <configuration>
                <!--<outputDirectory>${project.build.directory}/aa</outputDirectory>-->
                <outputDirectory>${user.dir}/dependency</outputDirectory>
                <overWriteReleases>false</overWriteReleases>
                <overWriteSnapshots>false</overWriteSnapshots>
                <overWriteIfNewer>true</overWriteIfNewer>
                <excludeArtifactIds>junit,netty-all</excludeArtifactIds>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### maven dependency 顺序的影响  
一个spring-boot 项目使用到了 slf4j-log4j，如下的以来顺序征程运行    
```xml
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.2</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>ch.qos.logback</groupId>
                    <artifactId>logback-classic</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
``` 
但是调换顺序就不行，spring-boot依赖到了slf4j-log4j中的一个类    
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>ch.qos.logback</groupId>
                    <artifactId>logback-classic</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.2</version>
        </dependency>
```

### 向maven中添加本地依赖  支持mvn clean package
```xml
        <dependency>
            <groupId>org.apache.kudu</groupId>
            <artifactId>kudu-spark2.3_2.11</artifactId>
            <version>1.5.0-cdh5.13.0</version>
            <scope>system</scope>
            <systemPath>${project.basedir}/libs/kudu-spark2.3_2.11-1.5.0-cdh5.13.0.jar</systemPath>
        </dependency>

```

### maven help 插件
1.查看活跃的profile
> mvn help:active-profiles
  
2.显示项目实际的pom
>mvn help:effective-pom  

3.显示项目实际的settings setting.xml 中的settings
>mvn help:effective-settings

4.描述插件的属性
>1.使用groupid,atifactid
mvn help:describe -DgroupId=xxx -DartifactId=xxx -Dversioni=
2.使用插件名称
mvn help:describe -Dplugin=jar

### mvn 运行程序
>mvn exec:java -Dexec.mainClass=com.jason.core.HookTest

### 浏览项目的依赖  
>mvn dependency:resolve  列出所有的依赖
mvn dependency:tree   以树状形式列出所有的依赖

### mvn 忽略测试失败
1.在plugin中配置
```xml
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <configuration>
                    <testFailureIgnore>true</testFailureIgnore>
                </configuration>
            </plugin>
```
2.在命令行配置
>mvn -Dmaaven.test.failure.ignore=true

### 跳过单元测试
1.命令行
>mvn -Dmaven.test.skip=true

2.plugin 配置
```xml
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <configuration>
                    <skip>true</skip>
                </configuration>
            </plugin>
``` 

### mvn 离线多线程
>mvn -o -T 2  
-0 启用offline模式  
-T 设置线程数  

### mvn scope 
```
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>3.9</version>
            <scope>compile</scope>
        </dependency>
scope 可以取以下值        
1.compile 
默认scope,如果不加scope元素,则scope为compile,在编译及运行阶段会把依赖加到classpath
2.provided
表示在运行环境已经提供该依赖,在编译阶段会被加入到classpath,但是不会被打入保重,运行时不会被加入classspath
3.test
只在测试的编译和运行时需要
4.system
参考maven添加本地依赖一节
```

### 分析依赖
>mvn dependency:analyze  
```
[WARNING] Used undeclared dependencies found:
[WARNING]    log4j:log4j:jar:1.2.17:compile
[WARNING]    org.apache.poi:poi-ooxml-schemas:jar:3.9:compile
[WARNING]    org.apache.xmlbeans:xmlbeans:jar:2.3.0:compile
[WARNING] Unused declared dependencies found:
[WARNING]    org.slf4j:slf4j-log4j12:jar:1.7.2:compile
[WARNING]    org.apache.poi:poi:jar:3.9:compile
[WARNING]    junit:junit:jar:4.11:test
[WARNING]    ch.qos.logback:logback-classic:jar:1.2.3:compile
[WARNING]    ch.qos.logback:logback-core:jar:1.2.3:compile

```
该命令可以列出了亮相内容  
1.在代码中直接引用了,但是没有直接在pom声明的依赖  
>为什么没声明也没报错,该依赖可能被递归依赖了,建议直接在pom中声明该依赖,
免得删除某个依赖(其递归依赖也会被删除)代码报错  
  
2.未使用,但是声明了的依赖,(不太准确,经测试有的依赖的确使用到了,也被列了出来)


### 解决依赖冲突
当依赖a的递归依赖包含b1,依赖c的递归依赖也包含b2,即依赖了两个版本的b,为了解决冲突,
可以采取如下方法,把递归依赖排除掉
```xml
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.2</version>
            <exclusions>
                <exclusion>
                    <groupId>log4j</groupId>
                    <artifactId>log4j</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```



### pom可以引用的变量
```
1.从shell系统变量获取
<finalName>${env.JAVA_HOME}</finalName>
2.从project中获取
<finalName>${project.version}</finalName>
3.从settings中获取
<finalName>${settings.localRepository}</finalName>
4.从java System.getProperti中获取
<finalName>${user.name}</finalName>
5.从pom <properties>中获取

```

### 依赖版本界限
会下载界限内的所有依赖
```
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>[3.2,3.8]</version>
        </dependency>
类似数学中的上下限,意义也是一样的 [x,y],(x,y)
```

### 将一组公用的依赖放到一个pom中
比如spark项目中依赖基本是固定的,这是后我们写一个如下的pom  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.jason</groupId>
    <artifactId>spark-dependency</artifactId>
    <version>1.0-SNAPSHOT</version>
    <!--打包类型是pom-->
    <packaging>pom</packaging>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.7</maven.compiler.source>
        <maven.compiler.target>1.7</maven.compiler.target>
        <spark.version>2.3.0</spark.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.11</artifactId>
            <version>${spark.version}</version>
            <!--<scope>provided</scope> -->   <!-- 正式发布，需要放开本行 -->
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.11</artifactId>
            <version>${spark.version}</version>
            <!--<scope>provided</scope>-->     <!-- 正式发布，需要放开本行 -->
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.2.11</version>
            <!--<scope>provided</scope>-->
        </dependency>
    </dependencies>


</project>
```


然后在其他项目中引用它
```xml
        <dependency>
            <groupId>com.jason</groupId>
            <artifactId>spark-dependency</artifactId>
            <version>1.0-SNAPSHOT</version>
            <!--依赖类型是pom-->
            <type>pom</type>
        </dependency>
```

###  多模块vs继承
多模块和继承的区别就在于"子module"中是否包含\<parent\>节点,
不包含则为多模块,包含则为继承,多模块不会继承"父"pom中的属性,依赖等,
多模块只是为了方便管理

###  maven resource

```xml
<build>
        <finalName>${user.name}</finalName>
        <resources>
            <resource>
            <!--进行资源过滤-->
                <filtering>true</filtering>
                <!--存放资源的目录-->
                <directory>src/main/resources</directory>
                <!--打包活编译时候需要包含的文件或者目录,mzven会先解析include,再解析exclude-->
                <includes>
                    <include>**/*</include>
                </includes>
                <!--打包活编译时候需要排除的文件或者目录-->
                <excludes>
                    <exclude>log4j.properties</exclude>
                </excludes>
                <!--编译时候存放资源文件的目录-->
                <targetPath>${project.basedir}/target/classes</targetPath>
            </resource>
        </resources>
 </build>
```
重点介绍下filtering,资源过滤功能,默认情况下,mmaven再copy资源时候是原封不动的copy的,  
但是考虑有以下资源文件  
aa.properties,内容如下:    
```
username=${username}
```
那么如果设置filtering为true,则maven会寻找 usernamae属性来替换${username}  

### maven profiles  
使用maven profiles可以实现项目高可移植性  
可以为不同的环境配置不同的profile,在编译,打包项目时指定使用哪个profile  
profile之一事项:  
```
1.一般出现在pom中最后一个元素
2.profile必须有一个id元素
3.profile可以包含pom中project下的任一元素
```
profile的一个简单例子,利用resources元素在不同的环境下为mysql提供不同的配置信息   
假设在src/main/resources/aa.properties中配置了如下内容  
```
jdbc.url=${jdbc.url}
jdbc.passwd=${jdbc.passwd}
jdbc.username=${jdbc.username}
```
pom中配置了如下profile  
```
    <profiles>
        <profile>
            <id>dev</id>
            <properties>
                <jdbc.url>jdbc:mysql://localhost:3306/shgb_fz</jdbc.url>
                <jdbc.passwd>123</jdbc.passwd>
                <jdbc.username>test</jdbc.username>
            </properties>
            <build>
                <resources>
                    <resource>
                        <directory>src/main/resources</directory>
                        <filtering>true</filtering>
                    </resource>
                </resources>
            </build>
        </profile>
        <profile>
            <id>pro</id>
            <properties>
                <jdbc.url>jdbc:mysql://192.168.0.105:3306/shgb_fz</jdbc.url>
                <jdbc.passwd>456</jdbc.passwd>
                <jdbc.username>data</jdbc.username>
            </properties>
            <build>
                <resources>
                    <resource>
                        <directory>src/main/resources</directory>
                        <filtering>true</filtering>
                    </resource>
                </resources>
            </build>
        </profile>
    </profiles>
```

在打包命令行中添加-P参数,即可使用指定的profile内容  
```
mvn -Pdev clean package  #使用dev profile
mvn -Ppro clean package  #使用pro profile
```

### maven profiles 的激活
上面的介绍中我们采用mvn -P 来激活profile,还可以配置activaion元素来指定如何激活profile  
1.当运行环境中某个property的value为指定值的时候  
```
        <profile>
            <id>dev</id>
            <activation>
                <property>
                    <name>jason.env</name>
                    <value>dev</value>
                </property>
            </activation>
            ...
        </profiles>
激活方式:
mvn -Djason.env=dev clean package
```  

2.默认激活  
```
        <profile>
            <id>dev</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profiles>    
```
3.jdk 不举例   
4.os 不举例  

###  settings profile  
之前配置的profiles都是在pom中配置的,只能当前项目使用,如果配置在settings.xml中,  
则全部项目都能使用,有些公司的生产数据库存储了隐私数据,只有开发组长活运营人员  
才能知道密码,这是运营人员可以将生产上数据库密码配置在settings.xml的profiles中  
pom.xml
```
    <profiles>
        <profile>
            <id>dev</id>
            <properties>
                <jdbc.url>jdbc:mysql://localhost:3306/shgb_fz</jdbc.url>
                <jdbc.passwd>123</jdbc.passwd>
                <jdbc.username>test</jdbc.username>
            </properties>
            <build>
                <resources>
                    <resource>
                        <directory>src/main/resources</directory>
                        <filtering>true</filtering>
                    </resource>
                </resources>
            </build>
        </profile>
        <profile>
            <id>pro</id>
            <properties>
                <jdbc.url>jdbc:mysql://192.168.0.105:3306/shgb_fz</jdbc.url>
                <jdbc.passwd>456</jdbc.passwd>
            </properties>
            <build>
                <resources>
                    <resource>
                        <directory>src/main/resources</directory>
                        <filtering>true</filtering>
                    </resource>
                </resources>
            </build>
        </profile>
    </profiles>
```
settings.xml 
```
<profiels>
   <profile>
            <id>pro</id>
            <properties>
                <jdbc.username>data</jdbc.username>
            </properties>
        </profile>
</profiles>
```
这样开发时不知道生产密码的,但是运营人员电脑配置了settings.xml ,mvn -Ppro clean package 时,  
可以将密码写入properties文件  


###  assembly ,允许用户个性化设置的插件  
1.先介绍assembly plugin预定义的一些功能，先上个例子  
```
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>2.2-beta-5</version>
                <executions>
                    <execution>
                        <id>jason</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                        <configuration>
                            <descriptorRefs>
                                <descriptorRef>jar-with-dependencies</descriptorRef>
                            </descriptorRefs>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
```
我们将此插件绑定在了 package阶段，使用的是预定义descriptorRef是：<descriptorRef>jar-with-dependencies</descriptorRef>  
该descriptorRef的作用是将项目的依赖解压，一并打到jar包内  
另外两个常用的descriptorRef是：  
>1.project:打包源代码，将当前项目中除了target,.idea,.svn等文件外的其他项目文件打到包里，打包好的代码可以直接在  
ide里打开
>2.src:打包源代码，功能与project 类似，但是只打包maven定义的src标准目录，不打包用于自定义在项目中的其他文件  

2.自定义descriptor  
pom部分    
```
<plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>2.2-beta-5</version>
                <executions>
                    <execution>
                        <id>filter</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <filters>
                        <!--定义了变量，回从这个文件找value去替换变量-->
                        <filter>src/assembly/filter.properties</filter>
                    </filters>
                    <descriptors>
                        <descriptor>src/assembly/filter.xml</descriptor>
                    </descriptors>
                </configuration>
            </plugin>
```
filter.xml   
```
<assembly>
    <id>filter</id>
    <formats>
        <format>jar</format>
    </formats>
    <includeBaseDirectory>false</includeBaseDirectory>
    <fileSets>
        <fileSet>
            <directory>${basedir}</directory>
            <includes>
                <include>*.txt</include>
            </includes>
            <excludes>
                <exclude>README.txt</exclude>
            </excludes>
        </fileSet>
    </fileSets>
    <files>
        <file>
            <source>README.txt</source>
            <outputDirectory></outputDirectory>
            <!--代表该文件需要使用过滤功能，文件里面有变量-->
            <filtered>true</filtered>
        </file>
    </files>
</assembly>
```

3.自定义descriptor的其他功能
filter.xml
```
<assembly>
    <id>filter</id>
    <formats>
        <format>jar</format>
    </formats>
    <includeBaseDirectory>false</includeBaseDirectory>
    <!--将源码以及依赖的jar生成mavenrepository的样式，可以用来部署使用-->
    <repositories>
        <repository>
            <includeMetadata>false</includeMetadata>
            <outputDirectory>maven2</outputDirectory>
        </repository>
    </repositories>
    <fileSets>
        <fileSet>
            <directory>${basedir}</directory>
            <includes>
                <include>*.txt</include>
            </includes>
            <excludes>
                <exclude>README.txt</exclude>
            </excludes>
        </fileSet>
    </fileSets>
    <files>
        <file>
            <source>README.txt</source>
            <outputDirectory></outputDirectory>
            <!--代表该文件需要使用过滤功能，文件里面有变量-->
            <filtered>true</filtered>
        </file>
    </files>
    <dependencySets>
        <dependencySet>
            <outputDirectory>lib</outputDirectory>
            <unpack>false</unpack>
            <includes>
                <include>*:*</include>
            </includes>
            <excludes>
                <exclude>org.seleniumhq.selenium:*</exclude>
                <exclude>log4j:log4j</exclude>
            </excludes>
        </dependencySet>
    </dependencySets>

</assembly>
```


### 自己搭建maven仓库,nexus

### 高可移植性之多模块  
在profiles一节我们已经谈过高可移植性，最近遇到一个需求，一个多模块项目，父子关系最深有三层，  
grandpa->parent->child,不同的平台所需要的模块不同，比如a平台只需要模块1、2、3，b平台需要模块  
2,、3、4，所以打包时候需要打入不同的配置和jar包，最终的解决方案是再添加一个assembly模块，利用  
assembly plugin + profiles 来解决
```
<...>
    <groupId>com.ideal.sailer</groupId>
    <artifactId>assembly</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
     <...>    
    《profiles>...</profiles>
<...>
```

###  本地依赖的改进  
之前我们在maven中依赖本地依赖时介绍了如下方法：  
```xml
<dependency>
            <groupId>org.apache.kudu</groupId>
            <artifactId>kudu-spark2.3_2.11</artifactId>
            <version>1.5.0-cdh5.13.0</version>
            <scope>system</scope>
            <systemPath>${project.basedir}/libs/kudu-spark2.3_2.11-1.5.0-cdh5.13.0.jar</systemPath>
</dependency>
```
如果我们的项目只有一个模块用这种方法无可厚非，但是如果是多模块，如果在父pom定义了  
```xml
    
    <dependencyManagement>
        <dependencies>
           <dependency>
                       <groupId>org.apache.kudu</groupId>
                       <artifactId>kudu-spark2.3_2.11</artifactId>
                       <version>1.5.0-cdh5.13.0</version>
                       <scope>system</scope>
                       <systemPath>${project.basedir}/libs/kudu-spark2.3_2.11-1.5.0-cdh5.13.0.jar</systemPath>
           </dependency>
        </dependencies>
    </dependencyManagement>
```
在子模块中引用时，必须重新指定systemPath，还会有如下警告：  
```
xxx:jar should not point at files within the project directory,...
```

新的解决方案如下,按照正常方式指定依赖，在clean阶段将本地jar install到本地,  
这样子模块就可以正常依赖：    
```xml
...
<dependency>
                       <groupId>org.apache.kudu</groupId>
                       <artifactId>kudu-spark2.3_2.11</artifactId>
                       <version>1.5.0-cdh5.13.0</version>
</dependency>
...
<plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-install-plugin</artifactId>
                <version>2.5.2</version>
                <executions>
                    <execution>
                        <id>install-kudu-spark</id>
                        <phase>clean</phase>
                        <configuration>
                            <file>${project.basedir}/../../libs/kudu-spark2.3_2.11-1.5.0-cdh5.13.0.jar</file>
                            <repositoryLayout>default</repositoryLayout>
                            <groupId>org.apache.kudu</groupId>
                            <artifactId>kudu-spark2.3_2.11</artifactId>
                            <version>${kudu-spark.version}</version>
                            <packaging>jar</packaging>
                            <generatePom>true</generatePom>
                        </configuration>
                        <goals>
                            <goal>install-file</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
...
```  

