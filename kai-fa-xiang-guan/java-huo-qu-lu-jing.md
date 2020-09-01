1、获取当前jar路径

```
        String path = System.getProperty("java.class.path");
        int firstIndex = path.lastIndexOf(System.getProperty("path.separator")) + 1;
         int lastIndex = path.lastIndexOf(File.separator) + 1;
         path = path.substring(firstIndex, lastIndex);
        System.out.println(path);
```

2、https://blog.csdn.net/T1DMzks/article/details/75099029

```
package pathtest;


import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;

public class TestPath {

    public static void main(String[] args) {
        TestPath testPath = new TestPath();
        testPath.testPath();

    }


    public  void testPath(){
        try {

            String userPath = System.getProperty("user.dir");
            String path = TestPath.class.getProtectionDomain().getCodeSource().getLocation().getPath();
            String pathGetClass =this.getClass().getProtectionDomain().getCodeSource().getLocation().getPath();
            String classLoaderPath = getClass().getClassLoader().toString();    

            System.out.println("userPath: "+userPath);
            System.out.println("path: "+path);
            System.out.println("pathGetClass: "+pathGetClass);
            System.out.println("classLoaderPath: "+classLoaderPath);

            //看看getClassLoader是读取jar包里面还是读取jar包外面
            InputStream inputStream = getClass().getClassLoader().getResourceAsStream("text.txt");
            BufferedReader br = new BufferedReader(new InputStreamReader(inputStream));         
            String line="";
            while((line=br.readLine())!=null){
                System.out.println("getClassLoader: "+line);
            }

            //看看pathGetClass读取的是jar包里面，还是jar包外面
            File file = new File(pathGetClass+"text.txt");
            BufferedReader reader = new BufferedReader(new FileReader(file));
            String tempString ="";
            while ((tempString = reader.readLine()) != null){
                System.out.println("pathGetClass: "+tempString);
            }



        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }

    }

}

```



