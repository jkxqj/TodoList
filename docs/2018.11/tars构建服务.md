## tars服务端开发



### 1、创建一个 webapp maven 项目，pom.xml 导入依赖

```xml
<dependency>
    <groupId>com.tencent.tars</groupId>
    <artifactId>tars-server</artifactId>
    <version>1.0.3</version>
    <type>jar</type>
</dependency>
```

### 2、pom添加插件

```xml
<plugin>
     <groupId>com.tencent.tars</groupId>
     <artifactId>tars-maven-plugin</artifactId>
     <version>1.0.3</version>
     <configuration>
          <tars2JavaConfig>
              <!-- 项目中 tars 文件位置 -->
              <tarsFiles>
                <tarsFile>${basedir}/src/main/resources/push.tars</tarsFile>
              </tarsFiles>
              <!-- 源文件编码 -->
              <tarsFileCharset>UTF-8</tarsFileCharset>
              <!-- 生成服务端代码 -->
              <servant>true</servant>
              <!-- 生成源代码编码 -->
              <charset>UTF-8</charset>
              <!-- 生成的源代码目录 -->
              <srcPath>${basedir}/src/main/java</srcPath>
              <!-- 生成源代码包前缀 -->
              <packagePrefixName>com.vivo.</packagePrefixName>
           </tars2JavaConfig>
       </configuration>
</plugin>
```

### 3、在 src/main/resources 中新建 push.tars 文件

```java
module Push {
    interface Hello{
        string hello(int no, string name);
    };
};
```

### 4、在工程目录下运行 mvn tars:tars2java 命令来生成接口文件

tars 文件决定了生成的接口，命名上追加 Servant ；内容也基本类似。具体如下：

```java
@Servant
public interface HelloServant {
   public String hello(int no, String name);
}
```

### 5、继承接口，实现方法

```java
public class HelloServantImpl implements HelloServant {
    public String hello(int no, String name) {
        return String.format("hello no=%s, name=%s, time=%s", no, name, System.currentTimeMillis());
    }
}
```

### 6、在 src/main/resources 中新建 servants.xml 文件，目的是暴露服务。

注意官方文档写的是在WEB-INF下创建servants.xml是错的，巨坑。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<servants>
    <servant name="HelloObj">
        <home-api>com.vivo.push.HelloServant</home-api>
        <home-class>com.vivo.push.impl.HelloServantImpl</home-class>
    </servant>
</servants>
```

注意文件路径别搞错了。

### 7、打成 war 包，上传到tars管理平台

- 在【运维管理】选择【服务部署】，应用和服务名称根据自己的需求来填写，服务类型选择【tars_java】，模板选择【tars.tarsjava.default】，节点选择你要部署的节点，OBJ名称选择你在servants.xml中配置的名称

- 点击提交后到导航菜单【服务管理】-》 【发布管理】，选中服务和节点，上传 war 包，点击发布

- 过一段时间后刷新服务列表，如果当前状态显示为绿色【active】即为发布成功。

- 如果当前状态显示为红色的【inactive】，请登录tars服务器到  /usr/local/app/tars/app_log/AppName/ServerName下查看详细的日志信息。


# 客户端开发

### 1、新建 maven webapp 项目，导入依赖

```xml
<dependency>
  <groupId>com.tencent.tars</groupId>
  <artifactId>tars-client</artifactId>
  <version>1.0.3</version>
  <type>jar</type>
</dependency>
```

### 2、添加插件

```xml
<plugin>
  <groupId>com.tencent.tars</groupId>
  <artifactId>tars-maven-plugin</artifactId>
  <version>1.0.3</version>
  <configuration>
    <tars2JavaConfig>
      <!-- tars文件位置 -->
      <tarsFiles>
        <tarsFile>${basedir}/src/main/resources/push.tars</tarsFile>
      </tarsFiles>
      <!-- 源文件编码 -->
      <tarsFileCharset>UTF-8</tarsFileCharset>
      <!-- 生成代码，PS：客户端调用，这里需要设置为false -->
      <servant>false</servant>
      <!-- 生成源代码编码 -->
      <charset>UTF-8</charset>
      <!-- 生成的源代码目录 -->
      <srcPath>${basedir}/src/main/java</srcPath>
      <!-- 生成源代码包前缀 -->
      <packagePrefixName>com.vivo.</packagePrefixName>
    </tars2JavaConfig>
  </configuration>
</plugin>
```

和服务端基本一致，区别是 **<servant>** 标签里是 **false** ，和生成源代码包前缀

### 3、在 src/main/resources 中新建 hello.tars 文件

文件和服务端的 tars 文件一致，拷贝即可。

### 4、工程目录下运行 mvn tars:tars2java 命令生成接口

和服务端一致

### 5、测试，直接运行该文件即可

1）**同步调用**

```java
public class Test {
    public static void main(String[] args) {
        CommunicatorConfig cfg = new CommunicatorConfig();
        //构建通信器
        Communicator communicator = CommunicatorFactory.getInstance().getCommunicator(cfg);
        //通过通信器，生成代理对象
        HelloPrx proxy = communicator.stringToProxy(HelloPrx.class, "Push.HelloServer.HelloObj@tcp -h 10.101.97.57 -p 10030 -t 3000");
        String ret = proxy.hello(1000, "HelloTars");
        System.out.println(ret);
    }
}
```

2）**异步调用**

```java
public class TestAysnc {
    public static void main(String[] args) {
        CommunicatorConfig cfg = new CommunicatorConfig();
        //构建通信器
        Communicator communicator = CommunicatorFactory.getInstance().getCommunicator(cfg);
        //通过通信器，生成代理对象
        HelloPrx proxy = communicator.stringToProxy(HelloPrx.class, "Push.HelloServer.HelloObj@tcp -h 10.101.97.57 -p 10030 -t 3000");
        proxy.async_hello(new HelloPrxCallback() {
              @Override
              public void callback_expired () {
              }
              @Override
              public void callback_exception (Throwable ex){
              }
              @Override
              public void callback_hello (String ret){
                  System.out.println(ret);
              }
            },1000,"Hello Tars");
    }
}
```

