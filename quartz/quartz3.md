## **ϵ������**

[Spring����Quartz�ֲ�ʽ����](https://my.oschina.net/OutOfMemory/blog/1790200)

[Quartz���ݿ�����](https://my.oschina.net/OutOfMemory/blog/1799185)

[Quartz����Դ�����](https://my.oschina.net/OutOfMemory/blog/1800560)

## **ǰ��**

��һƪ����[Quartz���ݿ�����](https://my.oschina.net/OutOfMemory/blog/1799185)������QuartzĬ���ṩ��11�ű����Ľ��������Quartz����ε��ȵģ������ͨ�����ݿ�ķ�ʽ�����ڷֲ�ʽ���ȡ�

## **�����߳�**

Quartz�ڲ��ṩ�ĵ�������QuartzScheduler����QuartzScheduler��ί��QuartzSchedulerThreadȥʵʱ���ȣ�����������Ҫȥִ��job��ʱ��QuartzSchedulerThread��û��ֱ��ȥִ��job��  
���ǽ���ThreadPoolȥִ��job������ʹ��ʲôThreadPool����ʼ�������̣߳������������ļ��н������ã�

```
org.quartz.threadPool.class: org.quartz.simpl.SimpleThreadPool
org.quartz.threadPool.threadCount: 10
org.quartz.threadPool.threadPriority: 5
```

���õ��̳߳���SimpleThreadPool������Ĭ��������10���̣߳���SimpleThreadPool�ᴴ��10��WorkerThread����WorkerThreadȥִ�о����job��

## **���ȷ���**

QuartzSchedulerThread�ǵ��ȵĺ����࣬����Quartz�����ʵ�ֵ��ȵģ����Բ鿴QuartzSchedulerThread����Դ�룺

```
public void run() {
    boolean lastAcquireFailed = false;
 
    while (!halted.get()) {
        try {
            // check if we're supposed to pause...
            synchronized (sigLock) {
                while (paused && !halted.get()) {
                    try {
                        // wait until togglePause(false) is called...
                        sigLock.wait(1000L);
                    } catch (InterruptedException ignore) {
                    }
                }
 
                if (halted.get()) {
                    break;
                }
            }
 
            int availThreadCount = qsRsrcs.getThreadPool().blockForAvailableThreads();
            if(availThreadCount > 0) { // will always be true, due to semantics of blockForAvailableThreads...
 
                List<OperableTrigger> triggers = null;
 
                long now = System.currentTimeMillis();
 
                clearSignaledSchedulingChange();
                try {
                    triggers = qsRsrcs.getJobStore().acquireNextTriggers(
                            now + idleWaitTime, Math.min(availThreadCount, qsRsrcs.getMaxBatchSize()), qsRsrcs.getBatchTimeWindow());
                    lastAcquireFailed = false;
                    if (log.isDebugEnabled()) 
                        log.debug("batch acquisition of " + (triggers == null ? 0 : triggers.size()) + " triggers");
                } catch (JobPersistenceException jpe) {
                    if(!lastAcquireFailed) {
                        qs.notifySchedulerListenersError(
                            "An error occurred while scanning for the next triggers to fire.",
                            jpe);
                    }
                    lastAcquireFailed = true;
                    continue;
                } catch (RuntimeException e) {
                    if(!lastAcquireFailed) {
                        getLog().error("quartzSchedulerThreadLoop: RuntimeException "
                                +e.getMessage(), e);
                    }
                    lastAcquireFailed = true;
                    continue;
                }
 
                if (triggers != null && !triggers.isEmpty()) {
 
                    now = System.currentTimeMillis();
                    long triggerTime = triggers.get(0).getNextFireTime().getTime();
                    long timeUntilTrigger = triggerTime - now;
                    while(timeUntilTrigger > 2) {
                        synchronized (sigLock) {
                            if (halted.get()) {
                                break;
                            }
                            if (!isCandidateNewTimeEarlierWithinReason(triggerTime, false)) {
                                try {
                                    // we could have blocked a long while
                                    // on 'synchronize', so we must recompute
                                    now = System.currentTimeMillis();
                                    timeUntilTrigger = triggerTime - now;
                                    if(timeUntilTrigger >= 1)
                                        sigLock.wait(timeUntilTrigger);
                                } catch (InterruptedException ignore) {
                                }
                            }
                        }
                        if(releaseIfScheduleChangedSignificantly(triggers, triggerTime)) {
                            break;
                        }
                        now = System.currentTimeMillis();
                        timeUntilTrigger = triggerTime - now;
                    }
 
                    // this happens if releaseIfScheduleChangedSignificantly decided to release triggers
                    if(triggers.isEmpty())
                        continue;
 
                    // set triggers to 'executing'
                    List<TriggerFiredResult> bndles = new ArrayList<TriggerFiredResult>();
 
                    boolean goAhead = true;
                    synchronized(sigLock) {
                        goAhead = !halted.get();
                    }
                    if(goAhead) {
                        try {
                            List<TriggerFiredResult> res = qsRsrcs.getJobStore().triggersFired(triggers);
                            if(res != null)
                                bndles = res;
                        } catch (SchedulerException se) {
                            qs.notifySchedulerListenersError(
                                    "An error occurred while firing triggers '"
                                            + triggers + "'", se);
                            //QTZ-179 : a problem occurred interacting with the triggers from the db
                            //we release them and loop again
                            for (int i = 0; i < triggers.size(); i++) {
                                qsRsrcs.getJobStore().releaseAcquiredTrigger(triggers.get(i));
                            }
                            continue;
                        }
 
                    }
 
                    for (int i = 0; i < bndles.size(); i++) {
                        TriggerFiredResult result =  bndles.get(i);
                        TriggerFiredBundle bndle =  result.getTriggerFiredBundle();
                        Exception exception = result.getException();
 
                        if (exception instanceof RuntimeException) {
                            getLog().error("RuntimeException while firing trigger " + triggers.get(i), exception);
                            qsRsrcs.getJobStore().releaseAcquiredTrigger(triggers.get(i));
                            continue;
                        }
 
                        // it's possible to get 'null' if the triggers was paused,
                        // blocked, or other similar occurrences that prevent it being
                        // fired at this time...  or if the scheduler was shutdown (halted)
                        if (bndle == null) {
                            qsRsrcs.getJobStore().releaseAcquiredTrigger(triggers.get(i));
                            continue;
                        }
 
                        JobRunShell shell = null;
                        try {
                            shell = qsRsrcs.getJobRunShellFactory().createJobRunShell(bndle);
                            shell.initialize(qs);
                        } catch (SchedulerException se) {
                            qsRsrcs.getJobStore().triggeredJobComplete(triggers.get(i), bndle.getJobDetail(), CompletedExecutionInstruction.SET_ALL_JOB_TRIGGERS_ERROR);
                            continue;
                        }
 
                        if (qsRsrcs.getThreadPool().runInThread(shell) == false) {
                            // this case should never happen, as it is indicative of the
                            // scheduler being shutdown or a bug in the thread pool or
                            // a thread pool being used concurrently - which the docs
                            // say not to do...
                            getLog().error("ThreadPool.runInThread() return false!");
                            qsRsrcs.getJobStore().triggeredJobComplete(triggers.get(i), bndle.getJobDetail(), CompletedExecutionInstruction.SET_ALL_JOB_TRIGGERS_ERROR);
                        }
 
                    }
 
                    continue; // while (!halted)
                }
            } else { // if(availThreadCount > 0)
                // should never happen, if threadPool.blockForAvailableThreads() follows contract
                continue; // while (!halted)
            }
 
            long now = System.currentTimeMillis();
            long waitTime = now + getRandomizedIdleWaitTime();
            long timeUntilContinue = waitTime - now;
            synchronized(sigLock) {
                try {
                  if(!halted.get()) {
                    // QTZ-336 A job might have been completed in the mean time and we might have
                    // missed the scheduled changed signal by not waiting for the notify() yet
                    // Check that before waiting for too long in case this very job needs to be
                    // scheduled very soon
                    if (!isScheduleChanged()) {
                      sigLock.wait(timeUntilContinue);
                    }
                  }
                } catch (InterruptedException ignore) {
                }
            }
 
        } catch(RuntimeException re) {
            getLog().error("Runtime error occurred in main trigger firing loop.", re);
        }
    } // while (!halted)
 
    // drop references to scheduler stuff to aid garbage collection...
    qs = null;
    qsRsrcs = null;
}
```

### 1.halted��paused

��������booleanֵ�ı�־�������ֱ��ʾ��ֹͣ����ͣ��haltedĬ��Ϊfalse����QuartzSchedulerִ��shutdown()ʱ�Ż����Ϊtrue��pausedĬ����true����QuartzSchedulerִ��start()ʱ  
����Ϊfalse����������֮��QuartzSchedulerThread�Ϳ�������ִ���ˣ�

### 2.availThreadCount

��ѯSimpleThreadPool�Ƿ��п��õ�WorkerThread�����availThreadCount>0�������¼���ִ�������߼������������飻

### 3.acquireNextTriggers

��ѯһ��ʱ���ڽ�Ҫ�����ȵ�triggers��������3���Ƚ���Ҫ�Ĳ����ֱ��ǣ�idleWaitTime��maxBatchSize��batchTimeWindow����3�������������������ļ��н������ã�

```
org.quartz.scheduler.idleWaitTime:30000
org.quartz.scheduler.batchTriggerAcquisitionMaxCount:1
org.quartz.scheduler.batchTriggerAcquisitionFireAheadTimeWindow:0
```

idleWaitTime:�ڵ��ȳ����ڿ���״̬ʱ�����ȳ��������²�ѯ���ô�����֮ǰ�ȴ���ʱ�������Ժ���Ϊ��λ����Ĭ����30��;  
batchTriggerAcquisitionMaxCount:������ȳ���ڵ�һ�λ�ȡ�����ڴ������Ĵ����������������Ĭ����1;  
batchTriggerAcquisitionFireAheadTimeWindow:������������Ԥ���Ļ���ʱ��֮ǰ����ȡ�ʹ�����ʱ�䣨���룩��ʱ������Ĭ����0;

���¼����鿴acquireNextTriggers����Դ�룺

```
public List<OperableTrigger> acquireNextTriggers(final long noLaterThan, final int maxCount, final long timeWindow)
    throws JobPersistenceException {
     
    String lockName;
    if(isAcquireTriggersWithinLock() || maxCount > 1) { 
        lockName = LOCK_TRIGGER_ACCESS;
    } else {
        lockName = null;
    }
    return executeInNonManagedTXLock(lockName, 
            new TransactionCallback<List<OperableTrigger>>() {
                public List<OperableTrigger> execute(Connection conn) throws JobPersistenceException {
                    return acquireNextTrigger(conn, noLaterThan, maxCount, timeWindow);
                }
            },
            ......
            });
}
```

���Է���ֻ����������acquireTriggersWithinLock����batchTriggerAcquisitionMaxCount>1����²�ʹ��LOCK\_TRIGGER\_ACCESS����Ҳ����˵��Ĭ�ϲ������õ�����£�������û��ʹ�����ģ�  
��ô�������ڵ�ͬʱȥִ��acquireNextTriggers���᲻�����ͬһ��trigger�ڶ���ڵ㶼��ִ�У�  
ע��acquireTriggersWithinLock�����������ļ��н������ã�

```
org.quartz.jobStore.acquireTriggersWithinLock=true
```

acquireTriggersWithinLock����ȡtriggers��ʱ���Ƿ���Ҫʹ������Ĭ����false�����batchTriggerAcquisitionMaxCount>1���ͬʱ����acquireTriggersWithinLockΪtrue��

������������鿴TransactionCallback�ڲ���acquireNextTrigger����Դ�룺

```
protected List<OperableTrigger> acquireNextTrigger(Connection conn, long noLaterThan, int maxCount, long timeWindow)
    throws JobPersistenceException {
    if (timeWindow < 0) {
      throw new IllegalArgumentException();
    }
     
    List<OperableTrigger> acquiredTriggers = new ArrayList<OperableTrigger>();
    Set<JobKey> acquiredJobKeysForNoConcurrentExec = new HashSet<JobKey>();
    final int MAX_DO_LOOP_RETRY = 3;
    int currentLoopCount = 0;
    do {
        currentLoopCount ++;
        try {
            List<TriggerKey> keys = getDelegate().selectTriggerToAcquire(conn, noLaterThan + timeWindow, getMisfireTime(), maxCount);
             
            // No trigger is ready to fire yet.
            if (keys == null || keys.size() == 0)
                return acquiredTriggers;
 
            long batchEnd = noLaterThan;
 
            for(TriggerKey triggerKey: keys) {
                // If our trigger is no longer available, try a new one.
                OperableTrigger nextTrigger = retrieveTrigger(conn, triggerKey);
                if(nextTrigger == null) {
                    continue; // next trigger
                }
                 
                // If trigger's job is set as @DisallowConcurrentExecution, and it has already been added to result, then
                // put it back into the timeTriggers set and continue to search for next trigger.
                JobKey jobKey = nextTrigger.getJobKey();
                JobDetail job;
                try {
                    job = retrieveJob(conn, jobKey);
                } catch (JobPersistenceException jpe) {
                    try {
                        getLog().error("Error retrieving job, setting trigger state to ERROR.", jpe);
                        getDelegate().updateTriggerState(conn, triggerKey, STATE_ERROR);
                    } catch (SQLException sqle) {
                        getLog().error("Unable to set trigger state to ERROR.", sqle);
                    }
                    continue;
                }
                 
                if (job.isConcurrentExectionDisallowed()) {
                    if (acquiredJobKeysForNoConcurrentExec.contains(jobKey)) {
                        continue; // next trigger
                    } else {
                        acquiredJobKeysForNoConcurrentExec.add(jobKey);
                    }
                }
                 
                if (nextTrigger.getNextFireTime().getTime() > batchEnd) {
                  break;
                }
                // We now have a acquired trigger, let's add to return list.
                // If our trigger was no longer in the expected state, try a new one.
                int rowsUpdated = getDelegate().updateTriggerStateFromOtherState(conn, triggerKey, STATE_ACQUIRED, STATE_WAITING);
                if (rowsUpdated <= 0) {
                    continue; // next trigger
                }
                nextTrigger.setFireInstanceId(getFiredTriggerRecordId());
                getDelegate().insertFiredTrigger(conn, nextTrigger, STATE_ACQUIRED, null);
 
                if(acquiredTriggers.isEmpty()) {
                    batchEnd = Math.max(nextTrigger.getNextFireTime().getTime(), System.currentTimeMillis()) + timeWindow;
                }
                acquiredTriggers.add(nextTrigger);
            }
 
            // if we didn't end up with any trigger to fire from that first
            // batch, try again for another batch. We allow with a max retry count.
            if(acquiredTriggers.size() == 0 && currentLoopCount < MAX_DO_LOOP_RETRY) {
                continue;
            }
             
            // We are done with the while loop.
            break;
        } catch (Exception e) {
            throw new JobPersistenceException(
                      "Couldn't acquire next trigger: " + e.getMessage(), e);
        }
    } while (true);
     
    // Return the acquired trigger list
    return acquiredTriggers;
}
```

���ȿ�һ����ִ��selectTriggerToAcquire����ʱ�������µĲ�����misfireTime=��ǰʱ��-MisfireThreshold��MisfireThreshold�����������ļ��н������ã�

```
org.quartz.jobStore.misfireThreshold:?60000
```

misfireThreshold���д�������ʱ��������10���̣߳�������11��������������һ�������ӳ�ִ���ˣ��������Ϊ��������������������ʱ��ʱ�䣻����Ĳ�ѯSQL������ʾ��

```
SELECT TRIGGER_NAME, TRIGGER_GROUP, NEXT_FIRE_TIME, PRIORITY
  FROM qrtz_TRIGGERS
 WHERE SCHED_NAME = 'myScheduler'
   AND TRIGGER_STATE = 'WAITING'
   AND NEXT_FIRE_TIME <= noLaterThan
   AND (MISFIRE_INSTR = -1 OR
       (MISFIRE_INSTR != -1 AND NEXT_FIRE_TIME >= noEarlierThan))
 ORDER BY NEXT_FIRE_TIME ASC, PRIORITY DESC
```

�����noLaterThan=��ǰʱ��+idleWaitTime+batchTriggerAcquisitionFireAheadTimeWindow��  
noEarlierThan=��ǰʱ��-MisfireThreshold��  
�ڲ�ѯ��֮�󣬻����ִ��updateTriggerStateFromOtherState()��������trigger��״̬��STATE\_WAITING��STATE\_ACQUIRED�����һ��ж�rowsUpdated�Ƿ����0�������������ڵ㶼��ѯ����ͬ��trigger�����ǿ϶�ֻ����һ���ڵ���³ɹ���������״̬֮����qrtz\_fired\_triggers���в���һ����¼����ʾ��ǰtrigger�Ѿ�������״̬ΪSTATE_ACQUIRED��

### 4.executeInNonManagedTXLock

Quartz�ķֲ�ʽ�������ںܶ�ط���������忴һ��Quartz�����ʵ�ֲַ�ʽ���ģ�executeInNonManagedTXLock����Դ�����£�

```
protected <T> T executeInNonManagedTXLock(
        String lockName, 
        TransactionCallback<T> txCallback, final TransactionValidator<T> txValidator) throws JobPersistenceException {
    boolean transOwner = false;
    Connection conn = null;
    try {
        if (lockName != null) {
            // If we aren't using db locks, then delay getting DB connection 
            // until after acquiring the lock since it isn't needed.
            if (getLockHandler().requiresConnection()) {
                conn = getNonManagedTXConnection();
            }
             
            transOwner = getLockHandler().obtainLock(conn, lockName);
        }
         
        if (conn == null) {
            conn = getNonManagedTXConnection();
        }
         
        final T result = txCallback.execute(conn);
        try {
            commitConnection(conn);
        } catch (JobPersistenceException e) {
            rollbackConnection(conn);
            if (txValidator == null || !retryExecuteInNonManagedTXLock(lockName, new TransactionCallback<Boolean>() {
                @Override
                public Boolean execute(Connection conn) throws JobPersistenceException {
                    return txValidator.validate(conn, result);
                }
            })) {
                throw e;
            }
        }
 
        Long sigTime = clearAndGetSignalSchedulingChangeOnTxCompletion();
        if(sigTime != null && sigTime >= 0) {
            signalSchedulingChangeImmediately(sigTime);
        }
         
        return result;
    } catch (JobPersistenceException e) {
        rollbackConnection(conn);
        throw e;
    } catch (RuntimeException e) {
        rollbackConnection(conn);
        throw new JobPersistenceException("Unexpected runtime exception: "
                + e.getMessage(), e);
    } finally {
        try {
            releaseLock(lockName, transOwner);
        } finally {
            cleanupConnection(conn);
        }
    }
}
```

���·ֳ�3�����裺��ȡ����ִ���߼����ͷ�����getLockHandler().obtainLock��ʾ��ȡ��txCallback.execute(conn)��ʾִ���߼���commitConnection(conn)��ʾ�ͷ���  
Quartz�ķֲ�ʽ���ӿ�����Semaphore��Ĭ�Ͼ����ʵ����StdRowLockSemaphore������ӿ����£�

```
public interface Semaphore {
    boolean obtainLock(Connection conn, String lockName) throws LockException;
    void releaseLock(String lockName) throws LockException;
    boolean requiresConnection();
}
```

���忴һ��obtainLock()����λ�ȡ���ģ�Դ�����£�

```
public boolean obtainLock(Connection conn, String lockName)
    throws LockException {
    if (!isLockOwner(lockName)) {
        executeSQL(conn, lockName, expandedSQL, expandedInsertSQL);
        getThreadLocks().add(lockName);
        
    } else if(log.isDebugEnabled()) {
        
    }
    return true;
}
 
protected void executeSQL(Connection conn, final String lockName, final String expandedSQL, final String expandedInsertSQL) throws LockException {
    PreparedStatement ps = null;
    ResultSet rs = null;
    SQLException initCause = null;
     
    int count = 0;
    do {
        count++;
        try {
            ps = conn.prepareStatement(expandedSQL);
            ps.setString(1, lockName);
             
            rs = ps.executeQuery();
            if (!rs.next()) {
                getLog().debug(
                        "Inserting new lock row for lock: '" + lockName + "' being obtained by thread: " + 
                        Thread.currentThread().getName());
                rs.close();
                rs = null;
                ps.close();
                ps = null;
                ps = conn.prepareStatement(expandedInsertSQL);
                ps.setString(1, lockName);
 
                int res = ps.executeUpdate();
                 
                if(res != 1) {
                   if(count < 3) {
                        try {
                            Thread.sleep(1000L);
                        } catch (InterruptedException ignore) {
                            Thread.currentThread().interrupt();
                        }
                        continue;
                    }
                }
            }
             
            return; // obtained lock, go
        } catch (SQLException sqle) {
            ......
    } while(count < 4);
 
}
```

obtainLock�����ж��Ƿ��Ѿ���ȡ���������û��ִ�з���executeSQL��������������Ҫ��SQL���ֱ��ǣ�expandedSQL��expandedInsertSQL����SCHED_NAME = ��myScheduler��Ϊ����

```
SELECT * FROM QRTZ_LOCKS WHERE SCHED_NAME = 'myScheduler' AND LOCK_NAME = ? FOR UPDATE
INSERT INTO QRTZ_LOCKS(SCHED_NAME, LOCK_NAME) VALUES ('myScheduler', ?)
```

select�����������FOR UPDATE�����LOCK_NAME���ڣ�������ڵ�ȥִ�д�SQLʱ��ֻ�е�һ���ڵ��ɹ��������Ľڵ㶼������ȴ���  
���LOCK_NAME�����ڣ�����ڵ�ͬʱִ��expandedInsertSQL��ֻ����һ���ڵ����ɹ���ִ�в���ʧ�ܵĽڵ㽫�������ԣ�����ִ��expandedSQL��  
txCallbackִ����֮��ִ��commitConnection������������ǰ�ڵ���ͷ���LOCK_NAME�������ڵ���Ծ�����ȡ�������ִ����releaseLock��

### 5.triggersFired

��ʾ����trigger������������£�

```
protected TriggerFiredBundle triggerFired(Connection conn,
        OperableTrigger trigger)
    throws JobPersistenceException {
    JobDetail job;
    Calendar cal = null;
 
    // Make sure trigger wasn't deleted, paused, or completed...
    try { // if trigger was deleted, state will be STATE_DELETED
        String state = getDelegate().selectTriggerState(conn,
                trigger.getKey());
        if (!state.equals(STATE_ACQUIRED)) {
            return null;
        }
    } catch (SQLException e) {
        throw new JobPersistenceException("Couldn't select trigger state: "
                + e.getMessage(), e);
    }
 
    try {
        job = retrieveJob(conn, trigger.getJobKey());
        if (job == null) { return null; }
    } catch (JobPersistenceException jpe) {
        try {
            getLog().error("Error retrieving job, setting trigger state to ERROR.", jpe);
            getDelegate().updateTriggerState(conn, trigger.getKey(),
                    STATE_ERROR);
        } catch (SQLException sqle) {
            getLog().error("Unable to set trigger state to ERROR.", sqle);
        }
        throw jpe;
    }
 
    if (trigger.getCalendarName() != null) {
        cal = retrieveCalendar(conn, trigger.getCalendarName());
        if (cal == null) { return null; }
    }
 
    try {
        getDelegate().updateFiredTrigger(conn, trigger, STATE_EXECUTING, job);
    } catch (SQLException e) {
        throw new JobPersistenceException("Couldn't insert fired trigger: "
                + e.getMessage(), e);
    }
 
    Date prevFireTime = trigger.getPreviousFireTime();
 
    // call triggered - to update the trigger's next-fire-time state...
    trigger.triggered(cal);
 
    String state = STATE_WAITING;
    boolean force = true;
     
    if (job.isConcurrentExectionDisallowed()) {
        state = STATE_BLOCKED;
        force = false;
        try {
            getDelegate().updateTriggerStatesForJobFromOtherState(conn, job.getKey(),
                    STATE_BLOCKED, STATE_WAITING);
            getDelegate().updateTriggerStatesForJobFromOtherState(conn, job.getKey(),
                    STATE_BLOCKED, STATE_ACQUIRED);
            getDelegate().updateTriggerStatesForJobFromOtherState(conn, job.getKey(),
                    STATE_PAUSED_BLOCKED, STATE_PAUSED);
        } catch (SQLException e) {
            throw new JobPersistenceException(
                    "Couldn't update states of blocked triggers: "
                            + e.getMessage(), e);
        }
    } 
         
    if (trigger.getNextFireTime() == null) {
        state = STATE_COMPLETE;
        force = true;
    }
 
    storeTrigger(conn, trigger, job, true, state, force, false);
 
    job.getJobDataMap().clearDirtyFlag();
 
    return new TriggerFiredBundle(job, trigger, cal, trigger.getKey().getGroup()
            .equals(Scheduler.DEFAULT_RECOVERY_GROUP), new Date(), trigger
            .getPreviousFireTime(), prevFireTime, trigger.getNextFireTime());
}
```

���Ȳ�ѯtrigger��״̬�Ƿ�STATE\_ACQUIRED״̬���������ֱ�ӷ���null��Ȼ��ͨ��ͨ��jobKey��ȡ��Ӧ��jobDetail�����¶�Ӧ��FiredTriggerΪEXECUTING״̬������ж�job��DisallowConcurrentExecution�Ƿ�������������˲��ܲ���ִ��job����ôtrigger��״̬ΪSTATE\_BLOCKED״̬������ΪSTATE\_WAITING�����״̬ΪSTATE\_BLOCKED����ô�´ε���  
��Ӧ��trigger���ᱻ��ȡ��ֻ�еȶ�Ӧ��jobִ����֮�󣬸���״̬ΪSTATE_WAITING֮��ſ���ִ�У���֤��job�Ĵ��У�

### 6.ִ��job

ͨ��ThreadPool��ִ�з�װjob��JobRunShell��

## **�������**

������[Spring����Quartz�ֲ�ʽ����](https://my.oschina.net/OutOfMemory/blog/1790200)�У�������˼��β��Էֲ�ʽ���ȣ����ڿ���������Ӧ�Ľ���

### 1.ͬһtriggerͬһʱ��ֻ����һ���ڵ�ִ��

�����п��Է���Quartzʹ���˷ֲ�ʽ����״̬����ֻ֤��һ���ڵ���ִ�У�

### 2.����û��ִ���꣬�������¿�ʼ

��Ϊ�����̺߳�����ִ���߳��Ƿֿ��ģ���Ϊִ����Threadpool��ִ�У����಻Ӱ�죻

### 3.ͨ��DisallowConcurrentExecutionע�Ᵽ֤����Ĵ���

��triggerFired�����ʹ����DisallowConcurrentExecution��������STATE_BLOCKED״̬����֤����Ĵ��У�

## **�ܽ�**

���Ĵ�Դ��ĽǶȴ��½�����һ��Quartz���ȵ����̣���Ȼ̫ϸ�ڵĶ���û��ȥ���룻ͨ�����Ĵ��¿��ԶԶ�ڵ���Ȳ�����������һ������Ľ��͡�

## **ϵ������**

[Spring����Quartz�ֲ�ʽ����](https://my.oschina.net/OutOfMemory/blog/1790200)  
[Quartz���ݿ�����](https://my.oschina.net/OutOfMemory/blog/1799185)  
[Quartz����Դ�����](https://my.oschina.net/OutOfMemory/blog/1800560)