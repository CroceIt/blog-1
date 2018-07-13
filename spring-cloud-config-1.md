##**Spring-Cloud-Config���**
Spring-Cloud-Config��Sping-Cloud�����ڷֲ�ʽ���ù����������ֳ���������ɫConfig-Server��Config-Client��Config-Server�˼���ʽ�洢/���������ļ����������ṩ�ӿڷ���Config-Client���ʣ��ӿ�ʹ��HTTP�ķ�ʽ�����ṩ���ʣ�Config-Clientͨ���ӿڻ�ȡ�����ļ���Ȼ�������Ӧ����ʹ�ã�Config-Server�洢/����������ļ��������Ա����ļ���Զ��Git�ֿ��Լ�Զ��Svn�ֿ⣻

##**Config-Server��**
###1.Config-Server����

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
    <version>2.0.0.RELEASE</version>
</dependency>
```
ע��2.0�Ժ�İ汾��Ҫjdk1.8�����ϰ汾

###2.׼��������������ļ�
Spring-Cloud-Config�ṩ�˶Զ��ֻ��������ļ���֧�֣����磺�������������Ի��������������ȣ�Ϊ�˸���ȫ���ģ�⣬׼���������÷ֱ����£�

```
config-dev.properties
config-test.properties
config-pro.properties
```
�ֱ��ǿ����������Լ������������ļ�������Ҳ�Ƚϼ�������ʾ��

```
foo=hello dev/test/pro
```
###3.׼�����������ļ�
������������ļ��������Զ���ط��������������ļ���Զ��Git�ֿ��Լ�Զ��Svn�ֿ⣬����ֱ���resources/application.properties��������;
####3.1�����ļ�

```
spring.application.name=config-server
server.port=8888
spring.profiles.active=native
spring.cloud.config.server.native.searchLocations=file:E:/github/spring-cloud-config-repo
```
ָ����server�������˿�Ϊ8888���ļ�����E:/github/spring-cloud-config-repo�����������ļ����ڴ�Ŀ¼��

####3.2Զ��Git�ֿ�

```
spring.application.name=config-server
server.port=8888
spring.profiles.active=git
spring.cloud.config.server.git.uri=https://github.com/ksfzhaohui/spring-cloud-config-repo
spring.cloud.config.server.git.default-label=master
```
spring.profiles.activeĬ��ֵ��git��git.uriָ����ַ��git�ֿ�default-labelĬ��ֵ��master��

####3.3Զ��svn�ֿ�

```
spring.profiles.active=subversion
spring.cloud.config.server.svn.uri=https://NJD9YZGJ2-PC.99bill.com:8443/svn/spring-cloud-config-repo
spring.cloud.config.server.svn.username=root
spring.cloud.config.server.svn.password=root
spring.cloud.config.server.svn.default-label=
```
������svn���û��������룬svn�ֿ�default-labelĬ��ֵ��trunk����Ϊ�˴��Խ���svn������default-labelΪ�գ���������Ϊ��ֵ���ɣ�

###4.׼��������

```
@SpringBootApplication
@EnableConfigServer
public class ConfigServer {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServer.class, args);
    }
}
```
@EnableConfigServer�������÷�������

###5.����
����ʹ�����ϵ����ַ�ʽ���ã�������ͨ��ʹ��http�ķ�ʽ���ʣ�http���������¼��ַ�ʽ������Դ��

```
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```
application��ʵ���ж�Ӧconfig��profile��ʾʹ�����ֻ����������ļ������������dev��test��pro��label��ѡ�ı�ǩ��git�ֿ�Ĭ��ֵmaster��svn�ֿ�Ĭ��ֵ��trunk��

5.1����http://localhost:8888/config/dev/master��������£�

```
{"name":"config","profiles":["dev"],"label":"master","version":"e9884489051c3b962840ac0a710f0f949a82d0ea","state":null,"propertySources":[{"name":"https://github.com/ksfzhaohui/spring-cloud-config-repo/config-dev.properties","source":{"foo":"hello dev"}}]}
```
���ؽ����������ϸ����Ϣ������source�����������ļ����ݣ�

5.2����http://localhost:8888/config-dev.yml��������£�

```
foo: hello dev
```
���ַ�ʽ���ʽ���ʾ�����ļ����ݣ�ͬ��properties��׺��Ҳ����ʾ�����ļ����ݣ�ֻ����ʾ�ĸ�ʽ��һ����

5.3����git���ļ����ݣ�����http://localhost:8888/config-dev.yml��������£�

```
foo: hello dev update
```
��ȡ�������µ����ݣ���ʵÿ���������ʱ�򶼻�ȥԶ�ֿ̲��и���һ�����ݣ���־���£�

```
2018-07-13 09:43:07.606  INFO 14040 --- [nio-8888-exec-7] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@34107ea3: startup date [Fri Jul 13 09:43:07 CST 2018]; root of context hierarchy
2018-07-13 09:43:07.610  INFO 14040 --- [nio-8888-exec-7] o.s.c.c.s.e.NativeEnvironmentRepository  : Adding property source: file:/C:/Users/HUIZHA~1.CFS/AppData/Local/Temp/config-repo-1042810186024067185/config-dev.properties
2018-07-13 09:43:07.611  INFO 14040 --- [nio-8888-exec-7] s.c.a.AnnotationConfigApplicationContext : Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@34107ea3: startup date [Fri Jul 13 09:43:07 CST 2018]; root of context hierarchy
```
�����ݸ��µ����ص�Temp·���£�

##**Config-Client��**
###1.Config-Client����

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
    <version>2.0.0.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.0.0.RELEASE</version>
</dependency>
```

###2.���������ļ�
�������ļ�resources/bootstrap.properties�����������ã�

```
spring.application.name=config
spring.cloud.config.label=master
spring.cloud.config.profile=test
spring.cloud.config.uri= http://localhost:8888/
server.port=8889
```
spring.application.name����Ӧ{application}����ʵ������config��
spring.cloud.config.label����Ӧ{label}��ָ��server�����õķ�֧���˴���master���ɣ�
spring.cloud.config.profile����Ӧ{profile}��ָ��client��ǰ�Ļ�������ѡֵ��dev��test��pro��
spring.cloud.config.uri��server�˵�ַ��
server.port��client�����˿ڣ�

###3.׼��������

```
@SpringBootApplication
public class ConfigClient {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClient.class, args);
    }
}
 
@RestController
public class HelloController {
 
    @Value("${foo}")
    String foo;
 
    @RequestMapping(value = "/hello")
    public String hello() {
        return foo;
    }
 
}
```
���ʵ�ַ��http://localhost:8889/hello�����ؽ�����£�

```
hello test
```
##**����Spring-Cloud-Config���õĸ���**
###1.Client�˳�ʼ�������ļ�
Client����������ʱ�򣬿��Է���Server������ȡ�����ļ�����־��

```
2018-07-13 12:47:36.330  INFO 13884 --- [nio-8888-exec-1] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@1917a8ad: startup date [Fri Jul 13 12:47:36 CST 2018]; root of context hierarchy
2018-07-13 12:47:36.399  INFO 13884 --- [nio-8888-exec-1] o.s.c.c.s.e.NativeEnvironmentRepository  : Adding property source: file:/C:/Users/HUIZHA~1.CFS/AppData/Local/Temp/config-repo-1261377317774171312/config-test.properties
2018-07-13 12:47:36.400  INFO 13884 --- [nio-8888-exec-1] s.c.a.AnnotationConfigApplicationContext : Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@1917a8ad: startup date [Fri Jul 13 12:47:36 CST 2018]; root of context hierarchy
```
###2.Server�����ݸ��£�Client��θ���
����git��config-test.properties������http://localhost:8888/config-test.yml��������£�

```
foo: hello test update
```
Client����http://localhost:8889/hello��������£�

```
hello test
```
���Է���Server���Ѿ����£�����Client��û�л�ȡ�����µ����ݣ�����ʹ�õĻ���������ݣ�
Spring-Cloud-Config�ṩ�˶���ˢ�»��ƣ����濴һ������ֶ�ˢ�£�

####2.1��������

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
    <version>2.0.0.RELEASE</version>
</dependency>
```
####2.2��¶ȫ��endpoints
��bootstrap.properties�����

```
management.endpoints.web.exposure.include=*
```
####2.3.�޸�HelloController

```
@RefreshScope
@RestController
public class HelloController {
 
    @Value("${foo}")
    String foo;
 
    @RequestMapping(value = "/hello")
    public String hello() {
        return foo;
    }
}
```
@RefreshScope���ֶ�ִ��ˢ�µ�ʱ�����´˱���

####2.4.����
�۲�������־��������һ��ӳ�����£�

```
2018-07-13 15:54:16.959  INFO 11372 --- [           main] s.b.a.e.w.s.WebMvcEndpointHandlerMapping : Mapped "{[/actuator/refresh],methods=[POST],produces=[application/vnd.spring-boot.actuator.v2+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.web.servlet.AbstractWebMvcEndpointHandlerMapping$OperationHandler.handle(javax.servlet.http.HttpServletRequest,java.util.Map<java.lang.String, java.lang.String>)
```
/actuator/refresh�ṩ���ֶ�ˢ�µĹ��ܣ����ұ���ʹ��POST��ʽ��

####2.5.����
���ʵ�ַ��http://localhost:8889/hello�����ؽ�����£�

```
hello test
```
����git�ϵ������ļ�������ֵΪfoo=hello test update��

���ʵ�ַ��http://localhost:8889/hello�����ؽ�����£�

```
hello test
```
ִ���ֶ�ˢ�²�����

```
C:\curl-7.61.0\I386>curl -X POST http://localhost:8889/actuator/refresh
["config.client.version","foo"]
```
���ʵ�ַ��http://localhost:8889/hello�����ؽ�����£�

```
hello test update
```

###3.����Զ�����
�����������²�����ÿ�ζ�ȥ�ֶ�����refresh��github�ṩ��webhook���ܣ���ĳ���¼�����ʱ��ͨ������http�ķ�ʽ���߽��շ��������Ϳ����ڽ��յ��¼���ʱ�򴥷�refresh����

##**��������������**
###1.���Client�ڵ���θ���
���������Client���кܶ���ڵ㣬���ҽڵ��������ߺ����ߣ����ͬʱ֪ͨÿ���ڵ㣬Spring-Cloud-Config�ṩ��Spring Cloud Bus����������

###2.���»���
��ִ��refresh��ʱ��ֻ��ѱ䶯�Ĳ������͸�Client�ˣ�û�б䶯�Ĳ��ᷢ�ͣ���Լ��������������������ļ��������ͬ��Clientʹ�ã��Ƿ����ֲ���ɵĲ����ᷢ�͸�ÿ��Client��

###3.�������ļ���֧��
Server����ͬʱ���ض�������ļ���ClientҲ����֧�ֶ�������ļ���

###4.Server����α�֤���ݵĿɿ���
Server�˼��й������ã����Է���Ŀɿ��Ժ���Ҫ��

##**�ܽ�**
���ķֱ��Server�˺�Client�˽��ʵ��������Spring-Cloud-Config���ʹ�ã�Ȼ�������������ʹ�ø��¹��ܣ�������˼��������������⣬�������з�����