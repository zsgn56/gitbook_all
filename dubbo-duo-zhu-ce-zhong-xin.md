### dubbo多注册中心配置

同时注册项目：

org.jacoco:jacoco-maven-plugin:0.7.9:prepare-agent -Dmaven.test.skip=true clean package install assembly:assembly -U -Ddisconf\_version=0.1.0  -Dconf\_server\_host=disconf.dev-hkp.com  -Dapp=sgbs -Denv=QA -Dignore=,

disconf中的dubbo.properties

```
dubbo.registry.address = zookeeper://zookeeper.dev-hkp.com:2181|zookeeper://zookeeper.staging.dev-hkp.com:2181
```



分开注册项目：

新增disconf配置

org.jacoco:jacoco-maven-plugin:0.7.9:prepare-agent -Dmaven.test.skip=true clean package install assembly:assembly -U -Ddisconf\_version=staging-0.1.0 -Dconf\_server\_host=disconf.dev-hkp.com  -Dapp=sfs -Denv=QA -Dignore=,

