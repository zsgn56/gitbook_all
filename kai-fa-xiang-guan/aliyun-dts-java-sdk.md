1、创建同步

```
package dtstest;

import com.aliyuncs.DefaultAcsClient;
import com.aliyuncs.dts.model.v20160801.ConfigureSynchronizationJobRequest;
import com.aliyuncs.dts.model.v20160801.ConfigureSynchronizationJobResponse;
import com.aliyuncs.profile.DefaultProfile;
import com.aliyuncs.profile.IClientProfile;

public class Main {

    public static void main(String[] args)  {

        IClientProfile profile = DefaultProfile.getProfile(
                "cn-hangzhou",          // 您的可用区ID
                "xxxx",      // 您的Access Key ID
                "xxxxx"); // 您的Access Key Secret

        DefaultAcsClient client = new DefaultAcsClient(profile);


        ConfigureSynchronizationJobRequest request = new ConfigureSynchronizationJobRequest();

        request.setSynchronizationJobId("dtssl9gn5wpi5pi");
        request.setSynchronizationJobName("test72");
        request.setSourceEndpointInstanceId("rm-bp12k9ixgfcqyf24w");
        request.setDestinationEndpointInstanceId("rm-bp156b160i68e8920");
        request.setStructureInitialization(true);
        request.setDataInitialization(true);
        String SyncObject="[{\"DBName\":\"sfs_move\",\"NewDBName\":\"alldb\","
                + "\"TableIncludes\":[{\"TableName\":\"job_order\",\"NewTableName\":\"sfs_job_order\"}]},{\"DBName\":\"sfs_move\",\"NewDBName\":\"alldb\","
                + "\"TableIncludes\":[{\"TableName\":\"combo\",\"NewTableName\":\"sfs_combo\"}]}]";

        request.setSynchronizationObjects(SyncObject);

        ConfigureSynchronizationJobResponse response = new ConfigureSynchronizationJobResponse();
        try {
            response = client.getAcsResponse(request);
            System.out.println("Configure Sync Job Succeed!");
        } catch (Exception e) {
            // TODO: handle exception
            System.out.println("Configure Sync Job Failed");
            System.out.println(e.toString());
        }

    }
```

2、批量修改同步对象关系

```
package dtstest;

import com.aliyuncs.DefaultAcsClient;
import com.aliyuncs.dts.model.v20160801.ModifySynchronizationObjectRequest;
import com.aliyuncs.dts.model.v20160801.ModifySynchronizationObjectResponse;
import com.aliyuncs.profile.DefaultProfile;
import com.aliyuncs.profile.IClientProfile;

public class Main {

    public static void main(String[] args)  {

        IClientProfile profile = DefaultProfile.getProfile(
                "cn-hangzhou",          // 您的可用区ID
                "LTAIb3K02OdB8tbn",      // 您的Access Key ID
                "kHiXaYC6tRD6ZJxQQAcH4fUch2VKY4"); // 您的Access Key Secret

        DefaultAcsClient client = new DefaultAcsClient(profile);



        String SyncObject="[{\"DBName\":\"hko_pro\",\"NewDBName\":\"data_db\",\"TableIncludes\":[{\"TableName\":\"act_user_questionnaire_record\",\"NewTableName\":\"hko_pro_act_user_questionnaire_record\"}]},"
                + "{\"DBName\":\"hko_pro\",\"NewDBName\":\"data_db\",\"TableIncludes\":[{\"TableName\":\"advertise_click\",\"NewTableName\":\"hko_pro_advertise_click\"}]},"
                + "{\"DBName\":\"simba_sbs\",\"NewDBName\":\"data_db\",\"TableIncludes\":[{\"TableName\":\"hks_stock_sync_log\",\"NewTableName\":\"sbs_hks_stock_sync_log\"}]},"
                + "{\"DBName\":\"simba_sbs\",\"NewDBName\":\"data_db\",\"TableIncludes\":[{\"TableName\":\"message\",\"NewTableName\":\"sbs_message\"}]}]";
        ModifySynchronizationObjectRequest request = new ModifySynchronizationObjectRequest();
        request.setSynchronizationJobId("dtsuvjq3mpxppa1");
        request.setSynchronizationObjects(SyncObject);
        ModifySynchronizationObjectResponse response = new ModifySynchronizationObjectResponse();

        try {
            response = client.getAcsResponse(request);
            String TaskId=response.getTaskId();
            System.out.println("Modify Sync Job  Succeed! Modify task Id:"+TaskId);

        } catch (Exception e) {
            // TODO: handle exception
            System.out.println("Modify Sync Job  Failed!");
            System.out.println(e.toString());

        }
    }



}
```

3、停止同步

```
package dtstest;

import com.aliyuncs.DefaultAcsClient;
import com.aliyuncs.dts.model.v20160801.ConfigureSynchronizationJobRequest;
import com.aliyuncs.dts.model.v20160801.ConfigureSynchronizationJobResponse;
import com.aliyuncs.dts.model.v20160801.SuspendSynchronizationJobRequest;
import com.aliyuncs.dts.model.v20160801.SuspendSynchronizationJobResponse;
import com.aliyuncs.profile.DefaultProfile;
import com.aliyuncs.profile.IClientProfile;

public class Main {

    public static void main(String[] args)  {

        IClientProfile profile = DefaultProfile.getProfile(
                "cn-hangzhou",          // 您的可用区ID
                "xxx",      // 您的Access Key ID
                "xxxx"); // 您的Access Key Secret

        DefaultAcsClient client = new DefaultAcsClient(profile);


        SuspendSynchronizationJobRequest request = new SuspendSynchronizationJobRequest();
        request.setSynchronizationJobId("dts1jy5hc7n7p0e0");
        SuspendSynchronizationJobResponse response = new SuspendSynchronizationJobResponse();
        try {
            response = client.getAcsResponse(request);
            System.out.println("Suspend Sync Job "+"dts1jy5hc7n7p0e0"+ " Succeed!");
        } catch (Exception e) {
            // TODO: handle exception
            System.out.println("Suspend Sync Job "+"dts1jy5hc7n7p0e0"+" Failed");
            System.out.println(e.toString());
        }


    }



}
```



