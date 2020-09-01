```
[ERROR] /data/jenkins/workspace/service-compass/compass-service/src/main/java/com/hk/compass/service/executive/AccessTokenServiceImpl.java:[10,30] error: cannot find symbol
[ERROR]  package com.hk.ets.common.utils
```

原因：查看代码，maven仓库中的ets-common-0.1-SNAPSHOT.jar  ，在com.hk.ets.common.utils下没有相关的class

```
remove type arguments
```

原因：import不相关的包导致冲突，重新确认import关系

