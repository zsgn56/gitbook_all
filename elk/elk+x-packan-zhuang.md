### 破解x-pack

wget [https://artifacts.elastic.co/downloads/packs/x-pack/x-pack-6.0.0.zip](https://artifacts.elastic.co/downloads/packs/x-pack/x-pack-6.0.0.zip)

mkdir -p /root/zzb/x-pack-new /root/zzb/newlicense /root/zzb/x-pack-jar && cd /root/zzb/ x-pack-new

unzip x-pack-6.0.0.zip

cp /root/zzb/x-pack-new/elasticsearch/x-pack-6.0.0.jar /root/zzb/x-pack-jar && cd /root/zzb/x-pack-jar

jar -xvf x-pack-6.0.0.jar &&sz org/elasticsearch/license/LicenseVerifier.class

windows下载Luyten 反编译软件打开将两个static中内容改为返回true，命名到LicenseVerifier.java上传到newlicense

```
import java.nio.*;
import java.util.*;
import java.security.*;
import org.elasticsearch.common.xcontent.*;
import org.apache.lucene.util.*;
import org.elasticsearch.common.io.*;
import java.io.*;

public class LicenseVerifier
{
    public static boolean verifyLicense(final License license, final byte[] encryptedPublicKeyData) {
    return true;
    }

    public static boolean verifyLicense(final License license) {
    return true;
    }
}
```

在newlicense目录中生成LicenseVerifier.class

/usr/local/jdk1.8.0\_131/bin/javac -cp "/usr/local/elasticsearch/lib/elasticsearch-6.0.0.jar:/usr/local/elasticsearch/lib/lucene-core-7.0.1.jar:/root/zzb/x-pack-new/elasticsearch/x-pack-6.0.0.jar" LicenseVerifier.java

cp LicenseVerifier.java /root/zzb/x-pack-jar/org/elasticsearch/license/LicenseVerifier.class

cd /root/zzb/x-pack-jar/ && jar -cvf x-pack-6.0.0.jar ./\*

cp x-pack-6.0.0.jar /root/zzb/x-pack-new/elasticsearch

cd /root/zzb/x-pack-new && zip x-pack-6.0.0.zip -r \*

### 安装x-pack到ELK

cd /usr/local/elasticsearch

./bin/elasticsearch-plugin install file:/root/zzb/x-pack-new/x-pack-6.0.0.zip

cd /usr/local/kibana

./bin/kibana-plugin  install file:/root/zzb/x-pack-new/x-pack-6.0.0.zip

cd /usr/local/logstash

./bin/logstash-plugin install file:/root/zzb/x-pack-new/x-pack-6.0.0.zip

### 启动elasticsearch

su elasticsearch

cd /usr/local/elasticsearch

./bin/elasticsearch &

/usr/local/elasticsearch/bin/x-pack/setup-passwords auto

```
Changed password for user kibana
PASSWORD kibana = $oFY58I
Changed password for user logstash_system
PASSWORD logstash_system = yGJSbi
Changed password for user elastic
PASSWORD elastic = ObJhh4$Y
```

### 启动kibana

vi /usr/local/kibana/config/kibana.yml

```
#xpack.security.enabled: false
elasticsearch.username: "kibana"
elasticsearch.password: "$oFY58I"
```

登录访问账号密码elastic ObJhh4$Y

### 启动logstash

vi /usr/local/logstash/logstash.conf

```
output {
  # 输出到elasticsearch
  elasticsearch {
     hosts => "127.0.0.1:9200"
      index => "logstash-%{[fields][project]}-%{+YYYY.MM.dd}"
            user => "elastic"
            password => "ObJhh4$Y"

   }
}
```

vi /usr/local/logstash/config/logstash.yml

```
xpack.monitoring.elasticsearch.username: logstash_system
xpack.monitoring.elasticsearch.password: yGJS
```

### 配置新的license文件

去官网申请免费证书 [https://license.elastic.co/registration](https://license.elastic.co/registration)

修改type和日期，保存文件为：`license.json`

```
{"license":{
    "uid":"aaa",
    "type":"platinum",
    "issue_date_in_millis":1515024000000,
    "expiry_date_in_millis":1596646399999,
    "max_nodes":100,
    "issued_to":"aaa",
    "issuer":"Web Form",
    "signature":"111",
    "start_date_in_millis":1515024000000
    }
}
```

[https://www.elastic.co/guide/en/x-pack/6.0/installing-license.html](https://www.elastic.co/guide/en/x-pack/6.0/installing-license.html) 按照提示更新证书

导入之前elasticsearch.yml中要禁用security：`xpack.security.enabled: false`

curl -XPUT -u elastic:ObJhh4$YR7zigxCjfF0M  '[http://localhost:9200/\_xpack/license](http://localhost:9200/_xpack/license)' -H "Content-Type: application/json" -d @license.json

### 使用x-pack

访问x-pack，可以监控读写速度、节点、索引

![](/assets/x-pack.png)watch

支持对number类型字段进行统计告警

**不支持非number类型，以及查询后的结果告警**

![](/assets/watch.png)

### 使用elastalert

cd /usr/local/

git clone [https://github.com/Yelp/elastalert.git](https://github.com/Yelp/elastalert.git)

cd elastalert

yum -y install python-pip python-devel

pip install -r requirements.txt

pip install --upgrade pip 根据实际情况执行

python setup.py install

es6.0下，可能需要先创建4个索引

curl -XPUT [http://localhost:9200/elastalert\_status\_status](http://localhost:9200/elastalert_status_status)

curl -XPUT [http://localhost:9200/elastalert\_status\_error](http://localhost:9200/elastalert_status_error)

curl -XPUT [http://localhost:9200/elastalert\_status\_silence](http://localhost:9200/elastalert_status_silence)

curl -XPUT [http://localhost:9200/elastalert\_status\_past](http://localhost:9200/elastalert_status_past)

elastalert-create-index

在/etc/elastalert/下创建config.yaml、smtp\_auth\_file.yaml、rules（支持多个规则文件）

```
rules_folder: /etc/elastalert/rules

run_every:
  minutes: 1

buffer_time:
  minutes: 15

es_host: 192.168.30.4
es_port: 9200
es_username: 
es_password:

writeback_index: elastalert_status

alert_time_limit:
  days: 2
```

```
user: ""
password: ""
```

规则文件/etc/elastalert/rules/elastalert\_rule\_ums.yaml

```
es_host: localhost
es_port: 9200
es_username: elastic
es_password: ObJhh4$YR7
name: test-error
type: frequency
index: logstash-service*
num_events: 5
timeframe:
  minutes: 1
filter:
- query:
    query_string:
      query: "logmessage: ERROR"

include: ["@timestamp", "loglevel", "logmessage","source"]

alert:
- "email"

smtp_host: smtp.mxhichina.com
from_addr: 
email_reply_to: 
smtp_auth_file: /etc/elastalert/smtp_auth_file.yaml

email:
- "111111@qq.com"
```

python -m elastalert.elastalert --config /etc/elastalert/config.yaml --rule /etc/elastalert/rules/elastalert\_rule.yaml --verbose

启动

