pom.xml

```
         <dubbo.version>2.8.4</dubbo.version>

+        <sentinel.version>0.1.1</sentinel.version>

+        <dependency>
+            <groupId>com.alibaba.csp</groupId>
+            <artifactId>sentinel-core</artifactId>
+            <version>${sentinel.version}</version>
+        </dependency>
+        <dependency>
+            <groupId>com.alibaba.csp</groupId>
+            <artifactId>sentinel-annotation-aspectj</artifactId>
+            <version>${sentinel.version}</version>
+        </dependency>
+        <dependency>
+            <groupId>com.alibaba.csp</groupId>
+            <artifactId>sentinel-transport-simple-http</artifactId>
+            <version>${sentinel.version}</version>
+        </dependency>
+        <dependency>
+            <groupId>com.alibaba.csp</groupId>
+            <artifactId>sentinel-dubbo-adapter</artifactId>
+            <version>${sentinel.version}</version>
+        </dependency>
+        <dependency>
+            <groupId>com.alibaba.csp</groupId>
+            <artifactId>sentinel-web-servlet</artifactId>
+            <version>${sentinel.version}</version>
+        </dependency>
```

**AbstractWebConfig.java**

```
   servletContext.addFilter("SentinelCommonFilter", new com.alibaba.csp.sentinel.adapter.servlet.CommonFilter()).addMappingForUrlPatterns(null, false, "/*");
```

**config/SentinelAspectConfiguration.java  
**

```
+package com.hk.app.user.config;
+
+import com.alibaba.csp.sentinel.annotation.aspectj.SentinelResourceAspect;
+import org.springframework.context.annotation.Bean;
+import org.springframework.context.annotation.Configuration;
+
+/**
+ * <p>
+ * 文件名称： SentinelAspectConfiguration</p>
+ * <p>
+ * 文件描述：</p>
+ * <p>
+ * 版权所有： 版权所有(C)2017-2099 </p>
+ * <p>
+ * 公司： 好慷 </p>
+ * <p>
+ * 内容摘要： </p>
+ * <p>
+ * 其他说明： </p>
+ *
+ * @author zhisheng Tu
+ * @version 1.0
+ * @date 2018/9/17 0017 15:11
+ */
+@Configuration
+public class SentinelAspectConfiguration {
+    @Bean
+    public SentinelResourceAspect sentinelResourceAspect() {
+        return new SentinelResourceAspect();
+    }
+}
```



