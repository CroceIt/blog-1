## **ϵ������**

[Spring����Quartz�ֲ�ʽ����](https://my.oschina.net/OutOfMemory/blog/1790200)

[Quartz���ݿ�����](https://my.oschina.net/OutOfMemory/blog/1799185)

[Quartz����Դ�����](https://my.oschina.net/OutOfMemory/blog/1800560)

**ǰ��**  
��һƪ����[Spring����Quartz�ֲ�ʽ����](https://my.oschina.net/OutOfMemory/blog/1790200)������Quartzͨ�����ݿ�ķ�ʽ��ʵ�ֲַ�ʽ���ȣ�ͨ��ʹ�����ݿ����洢trigger��job����Ϣ��������ͣ��������ʱ�����¼����ϴ�trigger��״̬����֤�������ԣ���һ����ͨ�����ݿ���ʵ����������ʵ�ֲַ�ʽ���ȣ�QuartzĬ���ṩ��11�ű����Ľ����⼸�ű�����Ҫ�ķ�����

**����Ϣ**

```sql
1.qrtz_blob_triggers
2.qrtz_cron_triggers
3.qrtz_simple_triggers
4.qrtz_simprop_triggers
5.qrtz_fired_triggers
6.qrtz_triggers
7.qrtz_job_details
8.qrtz_calendars
9.qrtz_paused_trigger_grps
10.qrtz_scheduler_state
11.qrtz_locks
```

��11�ű�ǰ6�Ŷ��ǹ��ڸ���triggers����Ϣ���������job��������������״̬����Ϣ����ر��������StdJDBCDelegate�У����sql�����StdJDBCConstants�У�

1.qrtz\_blob\_triggers  
�Զ����triggersʹ��blog���ͽ��д洢�����Զ����triggers�������ڴ˱��У�Quartz�ṩ��triggers������CronTrigger��CalendarIntervalTrigger��  
DailyTimeIntervalTrigger�Լ�SimpleTrigger���⼸��trigger��Ϣ�ᱣ���ں���ļ��ű��У�

2.qrtz\_cron\_triggers  
�洢CronTrigger����Ҳ������ʹ�����Ĵ��������������ļ������������ã�������qrtz\_cron\_triggers���ɼ�¼��

```xml
<bean id="firstCronTrigger"
    class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
    <property name="jobDetail" ref="firstTask" />
    <property name="cronExpression" value="0/6 * * ? * *" />
    <property name="group" value="firstCronGroup"></property>
</bean>
<bean id="firstTask"
    class="org.springframework.scheduling.quartz.JobDetailFactoryBean">
    <property name="jobClass" value="zh.maven.SQuartz.task.FirstTask" />
    <property name="jobDataMap">
        <map>
            <entry key="firstService" value-ref="firstService" />
        </map>
    </property>
</bean>
<bean id="firstService" class="zh.maven.SQuartz.service.FirstService"></bean>
```

���ʽָ����ÿ��6��ִ��һ�Σ�Ȼ��ָ����Ҫִ�е�task��taskָ����Ҫִ�е�ҵ������֮����Բ鿴���ݱ�

```
mysql> select * from qrtz_cron_triggers;
+-------------+------------------+----------------+-----------------+---------------+
| SCHED_NAME  | TRIGGER_NAME     | TRIGGER_GROUP  | CRON_EXPRESSION | TIME_ZONE_ID  |
+-------------+------------------+----------------+-----------------+---------------+
| myScheduler | firstCronTrigger | firstCronGroup | 0/6 * * ? * *   | Asia/Shanghai |
+-------------+------------------+----------------+-----------------+---------------+

```

myScheduler���ڶ���SchedulerFactoryBeanʱָ�������ƣ������ֶζ�������������������ҵ���

3.qrtz\_simple\_triggers  
�洢SimpleTrigger���������ļ������������ã�������qrtz\_simple\_triggers���ɼ�¼��

```xml
<bean id="firstSimpleTrigger"
    class="org.springframework.scheduling.quartz.SimpleTriggerFactoryBean">
    <property name="jobDetail" ref="firstSimpleTask" />
    <property name="startDelay" value="1000" />
    <property name="repeatInterval" value="2000" />
    <property name="repeatCount" value="5"></property>
    <property name="group" value="firstSimpleGroup"></property>
</bean>
<bean id="firstSimpleTask"
    class="org.springframework.scheduling.quartz.JobDetailFactoryBean">
    <property name="jobClass" value="zh.maven.SQuartz.task.SimpleFirstTask" />
    <property name="jobDataMap">
        <map>
            <entry key="firstService" value-ref="simpleFirstService" />
        </map>
    </property>
</bean>
<bean id="simpleFirstService" class="zh.maven.SQuartz.service.SimpleFirstService"></bean>

```

ָ���˿�ʼ�ӳ�ʱ�䣬�ظ����ʱ���Ѿ��ظ��Ĵ������ƣ��鿴�����£�

```sql
mysql> select * from qrtz_simple_triggers;
+-------------+--------------------+------------------+--------------+-----------------+-----------------+
| SCHED_NAME  | TRIGGER_NAME       | TRIGGER_GROUP    | REPEAT_COUNT | REPEAT_INTERVAL | TIMES_TRIGGERED |
+-------------+--------------------+------------------+--------------+-----------------+-----------------+
| myScheduler | firstSimpleTrigger | firstSimpleGroup |            5 |            2000 |               1 |
+-------------+--------------------+------------------+--------------+-----------------+-----------------+

```

TIMES\_TRIGGERED������¼ִ���˶��ٴ��ˣ���ֵ��������SimpleTriggerImpl�У�ÿ��ִ��+1�����ﶨ���REPEAT\_COUNT=5��ʵ�������ִ��6�Σ�������Բ鿴SimpleTriggerImplԴ�룺

```java
public Date getFireTimeAfter(Date afterTime) {
        if (complete) {
            return null;
        }
 
        if ((timesTriggered > repeatCount)
                && (repeatCount != REPEAT_INDEFINITELY)) {
            return null;
        }
        ......
}
```

timesTriggeredĬ��ֵΪ0����timesTriggered > repeatCountֹͣtrigger�����Ի�ִ��6�Σ���ִ�����֮��˼�¼�ᱻɾ����

4.qrtz\_simprop\_triggers  
�洢CalendarIntervalTrigger��DailyTimeIntervalTrigger�������͵Ĵ�������ʹ��CalendarIntervalTrigger���������ã�

```xml
<bean id="firstCalendarTrigger" class="org.quartz.impl.triggers.CalendarIntervalTriggerImpl">
    <property name="jobDataMap">
        <map>
            <entry key="jobDetail" value-ref="firstCalendarTask"></entry>
        </map>
    </property>
    <property name="key" ref="calendarTriggerKey"></property>
    <property name="repeatInterval" value="1" />
    <property name="group" value="firstCalendarGroup"></property>
</bean>
<bean id="firstCalendarTask"
    class="org.springframework.scheduling.quartz.JobDetailFactoryBean">
    <property name="jobClass" value="zh.maven.SQuartz.task.CalendarFirstTask" />
    <property name="jobDataMap">
        <map>
            <entry key="firstService" value-ref="calendarFirstService" />
        </map>
    </property>
</bean>
<bean id="calendarFirstService" class="zh.maven.SQuartz.service.CalendarFirstService"></bean>

```

CalendarIntervalTriggerû�ж�Ӧ��FactoryBean��ֱ������ʵ����CalendarIntervalTriggerImpl��ָ�����ظ�������1��Ĭ�ϵ�λ���죬Ҳ����ÿ��ִ��һ�Σ��鿴�����£�

```
mysql> select * from qrtz_simprop_triggers;
+-------------+--------------------+--------------------+------------+---------------+------------+------------+------------+-------------+-------------+------------+------------+-------------+-------------+
| SCHED_NAME  | TRIGGER_NAME       | TRIGGER_GROUP      | STR_PROP_1 | STR_PROP_2    | STR_PROP_3 | INT_PROP_1 | INT_PROP_2 | LONG_PROP_1 | LONG_PROP_2 | DEC_PROP_1 | DEC_PROP_2 | BOOL_PROP_1 | BOOL_PROP_2 |
+-------------+--------------------+--------------------+------------+---------------+------------+------------+------------+-------------+-------------+------------+------------+-------------+-------------+
| myScheduler | calendarTriggerKey | firstCalendarGroup | DAY        | Asia/Shanghai | NULL       |          1 |          1 |           0 |           0 |       NULL |       NULL | 0           | 0           |
+-------------+--------------------+--------------------+------------+---------------+------------+------------+------------+-------------+-------------+------------+------------+-------------+-------------+

```

�ṩ��3��string���͵Ĳ�����2��int���͵Ĳ�����2��long���͵Ĳ�����2��decimal���͵Ĳ����Լ�2��boolean���͵Ĳ���������ÿ��������ʲô���壬���ݲ�ͬ��trigger���ʹ�Ÿ��ԵĲ�����

5.qrtz\_fired\_triggers  
�洢�Ѿ�������trigger�����Ϣ��trigger����ʱ�������״̬�����仯��ֱ�����triggerִ����ɣ��ӱ��б�ɾ������SimpleTriggerΪ���ظ�3��ִ�У���ѯ��

```sql
mysql> select * from qrtz_fired_triggers;
+-------------+----------------------------------------+--------------------+------------------+---------------------------+---------------+---------------+----------+-----------+-----------------+-----------+------------------+-------------------+
| SCHED_NAME  | ENTRY_ID                               | TRIGGER_NAME       | TRIGGER_GROUP    | INSTANCE_NAME             | FIRED_TIME    | SCHED_TIME    | PRIORITY | STATE     | JOB_NAME        | JOB_GROUP | IS_NONCONCURRENT | REQUESTS_RECOVERY |
+-------------+----------------------------------------+--------------------+------------------+---------------------------+---------------+---------------+----------+-----------+-----------------+-----------+------------------+-------------------+
| myScheduler | NJD9YZGJ2-PC15241041777351524104177723 | firstSimpleTrigger | firstSimpleGroup | NJD9YZGJ2-PC1524104177735 | 1524104178499 | 1524104178472 |        0 | EXECUTING | firstSimpleTask | DEFAULT   | 0                | 0                 |
| myScheduler | NJD9YZGJ2-PC15241041777351524104177724 | firstSimpleTrigger | firstSimpleGroup | NJD9YZGJ2-PC1524104177735 | 1524104180477 | 1524104180472 |        0 | EXECUTING | firstSimpleTask | DEFAULT   | 0                | 0                 |
| myScheduler | NJD9YZGJ2-PC15241041777351524104177725 | firstSimpleTrigger | firstSimpleGroup | NJD9YZGJ2-PC1524104177735 | 1524104180563 | 1524104182472 |        0 | ACQUIRED  | NULL            | NULL      | 0                | 0                 |
+-------------+----------------------------------------+--------------------+------------------+---------------------------+---------------+---------------+----------+-----------+-----------------+-----------+------------------+-------------------+

```

��ͬ��trigger��task��ÿ����һ�ζ��ᴴ��һ��ʵ�����Ӹձ�������ACQUIRED״̬����EXECUTING״̬�����ִ��������ݿ���ɾ����

6.qrtz_triggers  
�洢�����trigger�����϶��������triggersΪ�����ֱ��ǣ�firstSimpleTrigger��firstCalendarTrigger��firstCronTrigger������֮��鿴���ݿ⣺

```
mysql> select * from qrtz_triggers;
+-------------+--------------------+--------------------+-------------------+-----------+-------------+----------------+----------------+----------+---------------+--------------+---------------+----------+---------------+---------------+----------+
| SCHED_NAME  | TRIGGER_NAME       | TRIGGER_GROUP      | JOB_NAME          | JOB_GROUP | DESCRIPTION | NEXT_FIRE_TIME | PREV_FIRE_TIME | PRIORITY | TRIGGER_STATE | TRIGGER_TYPE | START_TIME    | END_TIME | CALENDAR_NAME | MISFIRE_INSTR | JOB_DATA |
+-------------+--------------------+--------------------+-------------------+-----------+-------------+----------------+----------------+----------+---------------+--------------+---------------+----------+---------------+---------------+----------+
| myScheduler | calendarTriggerKey | firstCalendarGroup | firstCalendarTask | DEFAULT   | NULL        |  1524203884719 |  1524117484719 |        5 | WAITING       | CAL_INT      | 1524117484719 |        0 | NULL          |             0 |          |
| myScheduler | firstCronTrigger   | firstCronGroup     | firstTask         | DEFAULT   | NULL        |  1524117492000 |  1524117486000 |        0 | ACQUIRED      | CRON         | 1524117483000 |        0 | firstCalendar |             0 |          |
| myScheduler | firstSimpleTrigger | firstSimpleGroup   | firstSimpleTask   | DEFAULT   | NULL        |             -1 |  1524117488436 |        0 | COMPLETE      | SIMPLE       | 1524117484436 |        0 | NULL          |             0 |          |
+-------------+--------------------+--------------------+-------------------+-----------+-------------+----------------+----------------+----------+---------------+--------------+---------------+----------+---------------+---------------+----------+

```

��qrtz\_fired\_triggers��ŵĲ�һ��������trigger�����˶��ٴζ�ֻ��һ����¼��TRIGGER_STATE������ʶ��ǰtrigger��״̬��firstCalendarTaskÿ��ִ��һ�Σ�ִ����֮��һֱ��WAITING״̬��firstCronTriggerÿ6��ִ��һ��״̬��ACQUIRED״̬��firstSimpleTrigger�ظ�ִ��6�κ�״̬ΪCOMPLETE�����һᱻɾ����

7.qrtz\_job\_details  
�洢jobDetails��Ϣ�������Ϣ�ڶ����ʱ��ָ���������涨���JobDetailFactoryBean����ѯ���ݿ⣺

```
mysql> select * from qrtz_job_details;
+-------------+-------------------+-----------+-------------+-----------------------------------------+------------+------------------+----------------+-------------------+----------+
| SCHED_NAME  | JOB_NAME          | JOB_GROUP | DESCRIPTION | JOB_CLASS_NAME                          | IS_DURABLE | IS_NONCONCURRENT | IS_UPDATE_DATA | REQUESTS_RECOVERY | JOB_DATA |
+-------------+-------------------+-----------+-------------+-----------------------------------------+------------+------------------+----------------+-------------------+----------+
| myScheduler | firstCalendarTask | DEFAULT   | NULL        | zh.maven.SQuartz.task.CalendarFirstTask | 0          | 0                | 0              | 0                 | |
| myScheduler | firstSimpleTask   | DEFAULT   | NULL        | zh.maven.SQuartz.task.SimpleFirstTask   | 0          | 0                | 0              | 0                 | |
| myScheduler | firstTask         | DEFAULT   | NULL        | zh.maven.SQuartz.task.FirstTask         | 0          | 0                | 0              | 0                 | |
+-------------+-------------------+-----------+-------------+-----------------------------------------+------------+------------------+----------------+-------------------+----------+

```

JOB_DATA��ŵľ��Ƕ���taskʱָ����jobDataMap���ԣ����Դ�������Ҫʵ��Serializable�ӿڣ�����־û������ݿ⣻

8.qrtz_calendars  
QuartzΪ�����ṩ�������Ĺ��ܣ������Լ�����һ��ʱ��Σ����Կ��ƴ����������ʱ����ڴ������߲������������ṩ6�����ͣ�AnnualCalendar��CronCalendar��DailyCalendar��HolidayCalendar��MonthlyCalendar��WeeklyCalendar������ʹ��CronCalendarΪ����

```xml
<bean id="firstCalendar" class="org.quartz.impl.calendar.CronCalendar">
    <constructor-arg value="0/5 * * ? * *"></constructor-arg>
</bean>
<bean id="firstCronTrigger"
    class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
    <property name="jobDetail" ref="firstTask" />
    <property name="cronExpression" value="0/6 * * ? * *" />
    <property name="group" value="firstCronGroup"></property>
    <property name="calendarName" value="firstCalendar"></property>
</bean>
<bean id="scheduler"
    class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
    <property name="schedulerName" value="myScheduler"></property>
    <property name="dataSource" ref="dataSource" />
    <property name="configLocation" value="classpath:quartz.properties" />
    <property name="triggers">
        <list>
            <ref bean="firstCronTrigger" />
        </list>
    </property>
    <property name="calendars">
        <map>
            <entry key="firstCalendar" value-ref="firstCalendar"></entry>
        </map>
    </property>
</bean>
```

������һ���ų�ÿ��5���CronCalendar��Ȼ����firstCronTrigger��ָ����calendarName��������Ҫ��SchedulerFactoryBean�ж���calendars����ΪfirstCronTriggerÿ6��ִ��һ�Σ���CronCalendar�ų�ÿ��5�룬���Ի����firstCronTrigger�ڵ�5�δ�����ʱ����Ҫ�ȴ�12�룬������£�

```
20180419 15:09:06---start FirstService
20180419 15:09:08---end FirstService
20180419 15:09:12---start FirstService
20180419 15:09:14---end FirstService
20180419 15:09:18---start FirstService
20180419 15:09:20---end FirstService
20180419 15:09:24---start FirstService
20180419 15:09:26---end FirstService
20180419 15:09:36---start FirstService
20180419 15:09:38---end FirstService
```

��ѯ�����������е�CronCalendar��

```
mysql> select * from qrtz_calendars;
+-------------+---------------+----------+
| SCHED_NAME  | CALENDAR_NAME | CALENDAR |
+-------------+---------------+----------+
| myScheduler | firstCalendar | |
+-------------+---------------+----------+
```

CALENDAR��ŵ���CronCalendar���л�֮������ݣ�

9.qrtz\_paused\_trigger_grps  
�����ͣ���Ĵ������������ֶ���ͣfirstCronTrigger���������£�

```java
public class App {
    public static void main(String[] args) {
        final AbstractApplicationContext context = new ClassPathXmlApplicationContext("quartz.xml");
        final StdScheduler scheduler = (StdScheduler) context.getBean("scheduler");
        try {
            Thread.sleep(4000);
            scheduler.pauseTriggers(GroupMatcher.triggerGroupEquals("firstCronGroup"));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

����֮���ӳ�4�����ͣfirstCronTrigger�����ﴫ�ݵĲ���group��Ȼ��鿴���ݿ⣺

```
mysql> select * from qrtz_paused_trigger_grps;
+-------------+----------------+
| SCHED_NAME  | TRIGGER_GROUP  |
+-------------+----------------+
| myScheduler | firstCronGroup |
+-------------+----------------+
```

��Ϊ�Ѿ���⣬��������֮��firstCronGroup���Ǵ�����ͣ״̬��firstCronTrigger�������У�

10.qrtz\_scheduler\_state  
�洢���нڵ��scheduler���ᶨ�ڼ��scheduler�Ƿ�ʧЧ���������scheduler����ѯ���ݿ⣺

```
mysql> select * from qrtz_scheduler_state;
+-------------+---------------------------+-------------------+------------------+
| SCHED_NAME  | INSTANCE_NAME             | LAST_CHECKIN_TIME | CHECKIN_INTERVAL |
+-------------+---------------------------+-------------------+------------------+
| myScheduler | NJD9YZGJ2-PC1524209095408 |     1524209113973 |             1000 |
| myScheduler | NJD9YZGJ2-PC1524209097649 |     1524209113918 |             1000 |
+-------------+---------------------------+-------------------+------------------+
```

��¼��������µļ��ʱ�䣬��quartz.properties��������CHECKIN_INTERVALΪ1000��Ҳ����ÿ����һ�Σ�

11.qrtz_locks  
Quartz�ṩ������Ϊ����ڵ�����ṩ�ֲ�ʽ����ʵ�ֲַ�ʽ���ȣ�Ĭ����2������

```
mysql> select * from qrtz_locks;
+-------------+----------------+
| SCHED_NAME  | LOCK_NAME      |
+-------------+----------------+
| myScheduler | STATE_ACCESS   |
| myScheduler | TRIGGER_ACCESS |
+-------------+----------------+
```

STATE_ACCESS��Ҫ����scheduler���ڼ���Ƿ�ʧЧ��ʱ�򣬱�ֻ֤��һ���ڵ�ȥ�����Ѿ�ʧЧ��scheduler��  
TRIGGER_ACCESS��Ҫ����TRIGGER�����ȵ�ʱ�򣬱�ֻ֤��һ���ڵ�ȥִ�е��ȣ�

**�ܽ�**  
���Ķ���11�ű����˼�Ҫ�ķ�����������ÿ�ű�����������洢ʲô�ģ����Ҹ��˼򵥵�ʵ������ʵ���Ҫʵ��һ��trigger�Ĺ���ϵͳ����ʵҲ���Ƕ��⼸�ű��ά����