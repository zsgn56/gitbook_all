### 父项目zzb.dubbo

参考https://blog.csdn.net/qq\_38240021/article/details/80557113

pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>zzb.dubbo</groupId>
  <artifactId>zzb.dubbo</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>pom</packaging>
  <modules>
    <module>zzb.dubbo.comon</module>
    <module>zzb.dubbo.provider</module>
    <module>zzb.dubbo.consumer</module>
  </modules>
</project>
```

### module 接口zzb.dubbo.comon

pom.xml

```
<?xml version="1.0"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>zzb.dubbo</groupId>
    <artifactId>zzb.dubbo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
  </parent>
  <groupId>zzb.dubbo.comon</groupId>
  <artifactId>zzb.dubbo.comon</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <name>zzb.dubbo.comon</name>
  <url>http://maven.apache.org</url>
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

代码src\main\java\org\zzb\dubbo\comon\SayHello.java

```
package org.zzb.dubbo.comon;

public interface SayHello {

    String sayHello(String name);

}
```

### module 服务提供者zzb.dubbo.provider

pom.xml

```
<?xml version="1.0"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>zzb.dubbo</groupId>
    <artifactId>zzb.dubbo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
  </parent>
  <groupId>zzb.dubbo.provider</groupId>
  <artifactId>zzb.dubbo.provider</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <name>zzb.dubbo.provider</name>
  <url>http://maven.apache.org</url>
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>
  <dependencies>
         <dependency>
            <groupId>zzb.dubbo.comon</groupId>
            <artifactId>zzb.dubbo.comon</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>4.3.10.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.5.3</version>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework</groupId>
                    <artifactId>spring</artifactId>
                </exclusion>
                <exclusion>
                    <artifactId>netty</artifactId>
                    <groupId>org.jboss.netty</groupId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.10</version>
        </dependency>
                <dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.10</version>
        </dependency>
     <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

applicationContext.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
<dubbo:application name="hello-world-app"></dubbo:application>
    <!--<dubbo:registry address="multicast://224.5.6.7:2181"/>-->
    <dubbo:registry address="zookeeper://192.168.100.23:2181"/>
    <dubbo:protocol name="dubbo" port="20880"/>
    <dubbo:service interface="org.zzb.dubbo.comon.SayHello" ref="sayHelloImpl"/>
    <bean id="sayHelloImpl" class="org.zzb.dubbo.provider.SayHelloImpl"/>
</beans>
```

src\main\java\org\zzb\dubbo\provider\SayHelloImpl.java

```
package org.zzb.dubbo.provider;

import org.zzb.dubbo.comon.SayHello;

public class SayHelloImpl implements SayHello {
    public String sayHello(String name) {
        return "Hello "+name;
    }
}
```

src\main\java\org\zzb\dubbo\provider\Main.java

```
package org.zzb.dubbo.provider;

import java.io.IOException;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Main {
    public static void main(String[] args) throws IOException {
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        ctx.start();
        System.in.read();
    }

}
```



