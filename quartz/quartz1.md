## **ϵ������**

[Spring����Quartz�ֲ�ʽ����](https://my.oschina.net/OutOfMemory/blog/1790200)

[Quartz���ݿ�����](https://my.oschina.net/OutOfMemory/blog/1799185)

[Quartz����Դ�����](https://my.oschina.net/OutOfMemory/blog/1800560)

**ǰ��**  
Ϊ�˱�֤Ӧ�õĸ߿��ú͸߲����ԣ�һ�㶼�Ჿ�����ڵ㣻���ڶ�ʱ�������ÿ���ڵ㶼ִ���Լ��Ķ�ʱ����һ����ķ���ϵͳ��Դ����һ������Щ������ִ�У���������Ӧ���߼����⣬������Ҫһ���ֲ�ʽ�ĵ���ϵͳ����Э��ÿ���ڵ�ִ�ж�ʱ����

**Spring����Quartz**  
Quartz��һ��������������ϵͳ��Spring��Quartz���˼��ݣ����㿪�������濴������������ϣ�  
1.Maven�����ļ�

```
<dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>4.3.5.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>4.3.5.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>4.3.5.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>4.3.5.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.quartz-scheduler</groupId>
            <artifactId>quartz</artifactId>
            <version>2.2.3</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.29</version>
        </dependency>
    </dependencies>
```

��Ҫ����Spring��ؿ⡢quartz���Լ�mysql�����⣬ע���ֲ�ʽ������Ҫ�õ����ݿ⣬����ѡ��mysql��

2.����job  
�ṩ�����ַ�ʽ������job���ֱ��ǣ�MethodInvokingJobDetailFactoryBean��JobDetailFactoryBean  
2.1MethodInvokingJobDetailFactoryBean  
Ҫ�����ض�bean��һ��������ʱ��ʹ�ã������������£�

```xml
<bean id="firstTask" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">  
    <property name="targetObject" ref="firstService" />  
    <property name="targetMethod" value="service" />  
</bea>
```

2.2JobDetailFactoryBean  
���ַ�ʽ�������������ô��ݲ������������£�

```xml
<bean id="firstTask"
        class="org.springframework.scheduling.quartz.JobDetailFactoryBean">
        <property name="jobClass" value="zh.maven.SQuartz.task.FirstTask" />
        <property name="jobDataMap">
            <map>
                <entry key="firstService" value-ref="firstService" />
            </map>
        </property>
</bean>
```

jobClass����������࣬�̳�QuartzJobBean��ʵ��executeInternal������jobDataMap������job��������;

3.���õ���ʹ�õĴ�����  
ͬ���ṩ�����ִ��������ͣ�SimpleTriggerFactoryBean��CronTriggerFactoryBean  
�ص㿴CronTriggerFactoryBean���������͸������������£�

```xml
<bean id="firstCronTrigger"
    class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
    <property name="jobDetail" ref="firstTask" />
    <property name="cronExpression" value="0/5 * * ? * *" />
</bean>
```

jobDetailָ���ľ����ڲ���2�����õ�job��cronExpression������ÿ5��ִ��һ��job��

4.����Quartz��������SchedulerFactoryBean  
ͬ���ṩ�����ַ�ʽ���ڴ�RAMJobStore�����ݿⷽʽ  
4.1�ڴ�RAMJobStore  
job�������Ϣ�洢���ڴ��ÿ���ڵ�洢���Եģ�������룬�������£�

```xml
<bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
    <property name="triggers">
        <list>
            <ref bean="firstCronTrigger" />
        </list>
    </property>
</bean>
```

4.2���ݿⷽʽ  
job�������Ϣ�洢�����ݿ��У����нڵ㹲�����ݿ⣬ÿ���ڵ�ͨ�����ݿ���ͨ�ţ���֤һ��jobͬһʱ��ֻ����һ���ڵ���ִ�У�����  
���ĳ���ڵ�ҵ���job�ᱻ���䵽�����ڵ�ִ�У������������£�

```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource"
        destroy-method="close">
        <property name="driverClass" value="com.mysql.jdbc.Driver" />
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/quartz" />
        <property name="user" value="root" />
        <property name="password" value="root" />
    </bean>
    <bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <property name="configLocation" value="classpath:quartz.properties" />
        <property name="triggers">
            <list>
                <ref bean="firstCronTrigger" />
            </list>
        </property>
    </bean>
```

dataSource������������Դ�����ݱ������Ϣ�����Ե���������gz����sql�ļ���·����docs\\dbTables�£������ṩ���������ݿ��sql�ļ����ܹ�11�ű�  
configLocation���õ�quartz.properties�ļ���quartz.jar��org.quartz���£������ṩ��һЩĬ�ϵ����ݣ�����org.quartz.jobStore.class

```
org.quartz.jobStore.class: org.quartz.simpl.RAMJobStore
```

������Ҫ��quartz.properties����������һЩ�޸ģ������޸����£�

```
org.quartz.scheduler.instanceId: AUTO
org.quartz.jobStore.class: org.quartz.impl.jdbcjobstore.JobStoreTX
org.quartz.jobStore.isClustered: true
org.quartz.jobStore.clusterCheckinInterval: 1000
```

5.�����

```java
public class FirstTask extends QuartzJobBean {

    private FirstService firstService;

    @Override
    protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
        firstService.service();
    }

    public void setFirstService(FirstService firstService) {
        this.firstService = firstService;
    }
}
```

FirstTask�̳�QuartzJobBean��ʵ��executeInternal����������FirstService;

```java
public class FirstService implements Serializable {

    private static final long serialVersionUID = 1L;

    public void service() {
        System.out.println(new SimpleDateFormat("YYYYMMdd HH:mm:ss").format(new Date()) + "---start FirstService");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(new SimpleDateFormat("YYYYMMdd HH:mm:ss").format(new Date()) + "---end FirstService");
    }
}
```

FirstService��Ҫ�ṩ���л��ӿڣ���Ϊ��Ҫ���������ݿ��У�

```java
public class App {
    public static void main(String[] args) {
        AbstractApplicationContext context = new ClassPathXmlApplicationContext("quartz.xml");
    }
}
```

������������quartz�����ļ���

**���Էֲ�ʽ����**  
1.ͬʱ����App���Σ��۲���־��

```
20180405 14:48:10---start FirstService
20180405 14:48:12---end FirstService
20180405 14:48:15---start FirstService
20180405 14:48:17---end FirstService
```

����A1����־�����A2û�У���ͣ��A1�Ժ�A2����־�����

2.����µ�job�ֱ��½���SecondTask��SecondService��ͬʱ�����������ļ�������App�۲���־��  
A1��־���£�

```
20180405 15:03:15---start FirstService
20180405 15:03:15---start SecondService
20180405 15:03:17---end FirstService
20180405 15:03:17---end SecondService
20180405 15:03:20---start FirstService
20180405 15:03:22---end FirstService
20180405 15:03:25---start FirstService
20180405 15:03:27---end FirstService
```

A2��־���£�

```
20180405 15:03:20---start SecondService
20180405 15:03:22---end SecondService
20180405 15:03:25---start SecondService
20180405 15:03:27---end SecondService
```

���Է���A1��A2����ִ�����񣬵���ͬһ����ͬһʱ��ֻ����һ���ڵ�ִ�У�����ֻ����ִ�н�������п��ܷ��䵽�����ڵ㣻

3.������ʱ��С������ִ��ʱ�䣬��������ĳ�sleep(6000)  
A1��־���£�

```
20180405 15:14:40---start FirstService
20180405 15:14:45---start FirstService
20180405 15:14:46---end FirstService
20180405 15:14:50---start FirstService
20180405 15:14:50---start SecondService
20180405 15:14:51---end FirstService
```

A2��־���£�

```
20180405 15:14:40---start SecondService
20180405 15:14:45---start SecondService
20180405 15:14:46---end SecondService
20180405 15:14:51---end SecondService
```

���ʱ����5�룬������ִ����Ҫ6�룬�۲���־���Է��֣�����û�н������µ������Ѿ���ʼ�����������������Ӧ�õ��߼����⣬��ʵ���������ܲ���֧�ִ��е����⣻

4.@DisallowConcurrentExecutionע�Ᵽ֤����Ĵ���  
��FirstTask��SecondTask�Ϸֱ����@DisallowConcurrentExecutionע�⣬��־������£�  
A1��־���£�

```
20180405 15:32:45---start FirstService
20180405 15:32:51---end FirstService
20180405 15:32:51---start FirstService
20180405 15:32:51---start SecondService
20180405 15:32:57---end FirstService
20180405 15:32:57---end SecondService
20180405 15:32:57---start FirstService
20180405 15:32:57---start SecondService
```

A2��־���£�

```
20180405 15:32:45---start SecondService
20180405 15:32:51---end SecondService
```

�۲���־���Է��֣�����ֻ����end�Ժ󣬲ŻῪʼ�µ�����ʵ��������Ĵ��л���

**�ܽ�**  
����ּ�ڶ�Spring+Quartz�ֲ�ʽ������һ��ֱ�۵��˽⣬ͨ��ʵ�ʵ�ʹ����������⣬��Ȼ���ܻ��кܶ����ʱ���������ε��ȵģ����ݿ�������˻���ô���ȵȣ�����Ҫ������������˽⡣