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
多模块和继承的区别就在于"子module"中是否包含<parent>节点,
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