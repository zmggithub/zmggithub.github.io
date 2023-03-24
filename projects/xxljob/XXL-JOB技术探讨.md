# XXL-JOB技术探讨

先上图

![image.png](../../../pic/resize,w_1492-20191202185401274.png)


问题集（麻烦大家把问题汇总一下，填写一下）
### 路由策略（实现细节）

- 忙碌转移
  - com.xxl.job.admin.core.route.strategy.ExecutorRouteBusyover#route
```java
StringBuffer idleBeatResultSB = new StringBuffer();
for (String address : addressList) {
    // beat
    ReturnT<String> idleBeatResult = null;
    try {
        ExecutorBiz executorBiz = XxlJobScheduler.getExecutorBiz(address);
        idleBeatResult = executorBiz.idleBeat(triggerParam.getJobId());
    } catch (Exception e) {
        logger.error(e.getMessage(), e);
        idleBeatResult = new ReturnT<String>(ReturnT.FAIL_CODE, ""+e );
    }
    idleBeatResultSB.append( (idleBeatResultSB.length()>0)?"<br><br>":"")
        .append(I18nUtil.getString("jobconf_idleBeat") + "：")
        .append("<br>address：").append(address)
        .append("<br>code：").append(idleBeatResult.getCode())
        .append("<br>msg：").append(idleBeatResult.getMsg());

    // beat success
    if (idleBeatResult.getCode() == ReturnT.SUCCESS_CODE) {
        idleBeatResult.setMsg(idleBeatResultSB.toString());
        idleBeatResult.setContent(address);
        return idleBeatResult;
    }
}

return new ReturnT<String>(ReturnT.FAIL_CODE, idleBeatResultSB.toString());
```

  - com.xxl.job.core.biz.impl.ExecutorBizImpl#idleBeat
```java
 // isRunningOrHasQueue
        boolean isRunningOrHasQueue = false;
        JobThread jobThread = XxlJobExecutor.loadJobThread(jobId);
        if (jobThread != null && jobThread.isRunningOrHasQueue()) {
            isRunningOrHasQueue = true;
        }

        if (isRunningOrHasQueue) {
            return new ReturnT<String>(ReturnT.FAIL_CODE, "job thread is running or has trigger queue.");
        }
        return ReturnT.SUCCESS;
```

- 最不经常使用
  - com.xxl.job.admin.core.route.strategy.ExecutorRouteLFU#route

```java
// cache clear
if (System.currentTimeMillis() > CACHE_VALID_TIME) {
    jobLfuMap.clear();
    CACHE_VALID_TIME = System.currentTimeMillis() + 1000*60*60*24;
}

// lfu item init
HashMap<String, Integer> lfuItemMap = jobLfuMap.get(jobId);     // Key排序可以用TreeMap+构造入参Compare；Value排序暂时只能通过ArrayList；
if (lfuItemMap == null) {
    lfuItemMap = new HashMap<String, Integer>();
    jobLfuMap.putIfAbsent(jobId, lfuItemMap);   // 避免重复覆盖
}

// put new
for (String address: addressList) {
    if (!lfuItemMap.containsKey(address) || lfuItemMap.get(address) >1000000 ) {
        lfuItemMap.put(address, new Random().nextInt(addressList.size()));  // 初始化时主动Random一次，缓解首次压力
    }
}
// remove old
List<String> delKeys = new ArrayList<>();
for (String existKey: lfuItemMap.keySet()) {
    if (!addressList.contains(existKey)) {
        delKeys.add(existKey);
    }
}
if (delKeys.size() > 0) {
    for (String delKey: delKeys) {
        lfuItemMap.remove(delKey);
    }
}

// load least userd count address
List<Map.Entry<String, Integer>> lfuItemList = new ArrayList<Map.Entry<String, Integer>>(lfuItemMap.entrySet());
Collections.sort(lfuItemList, new Comparator<Map.Entry<String, Integer>>() {
    @Override
    public int compare(Map.Entry<String, Integer> o1, Map.Entry<String, Integer> o2) {
        return o1.getValue().compareTo(o2.getValue());
    }
});

Map.Entry<String, Integer> addressItem = lfuItemList.get(0);
String minAddress = addressItem.getKey();
addressItem.setValue(addressItem.getValue() + 1);

return addressItem.getKey();
```


### 分片问题

- 分片不合理，默认走第一台
- 是否走执行路由策略

分片本身就是路由策略的一种

```java

    FIRST(I18nUtil.getString("jobconf_route_first"), new ExecutorRouteFirst()),
    LAST(I18nUtil.getString("jobconf_route_last"), new ExecutorRouteLast()),
    ROUND(I18nUtil.getString("jobconf_route_round"), new ExecutorRouteRound()),
    RANDOM(I18nUtil.getString("jobconf_route_random"), new ExecutorRouteRandom()),
    CONSISTENT_HASH(I18nUtil.getString("jobconf_route_consistenthash"), new ExecutorRouteConsistentHash()),
    LEAST_FREQUENTLY_USED(I18nUtil.getString("jobconf_route_lfu"), new ExecutorRouteLFU()),
    LEAST_RECENTLY_USED(I18nUtil.getString("jobconf_route_lru"), new ExecutorRouteLRU()),
    FAILOVER(I18nUtil.getString("jobconf_route_failover"), new ExecutorRouteFailover()),
    BUSYOVER(I18nUtil.getString("jobconf_route_busyover"), new ExecutorRouteBusyover()),
    SHARDING_BROADCAST(I18nUtil.getString("jobconf_route_shard"), null); # 广播模式就是分片执行策略
```


### 任务配置超时时间，超时之后操作
中止当前任务线程，com.xxl.job.core.thread.JobThread#run

```java
// init
try {
    handler.init();
} catch (Throwable e) {
    logger.error(e.getMessage(), e);
}

// execute
while(!toStop){
    running = false;
    idleTimes++;

    TriggerParam triggerParam = null;
    ReturnT<String> executeResult = null;
    try {
        // to check toStop signal, we need cycle, so wo cannot use queue.take(), instand of poll(timeout)
        triggerParam = triggerQueue.poll(3L, TimeUnit.SECONDS);
        if (triggerParam!=null) {
            running = true;
            idleTimes = 0;
            triggerLogIdSet.remove(triggerParam.getLogId());

            // log filename, like "logPath/yyyy-MM-dd/9999.log"
            String logFileName = XxlJobFileAppender.makeLogFileName(new Date(triggerParam.getLogDateTim()), triggerParam.getLogId());
            XxlJobFileAppender.contextHolder.set(logFileName);
            ShardingUtil.setShardingVo(new ShardingUtil.ShardingVO(triggerParam.getBroadcastIndex(), triggerParam.getBroadcastTotal()));

            // execute
            XxlJobLogger.log("<br>----------- xxl-job job execute start -----------<br>----------- Param:" + triggerParam.getExecutorParams());

            if (triggerParam.getExecutorTimeout() > 0) {
                // limit timeout
                Thread futureThread = null;
                try {
                    final TriggerParam triggerParamTmp = triggerParam;
                    FutureTask<ReturnT<String>> futureTask = new FutureTask<ReturnT<String>>(new Callable<ReturnT<String>>() {
                        @Override
                        public ReturnT<String> call() throws Exception {
                            return handler.execute(triggerParamTmp.getExecutorParams());
                        }
                    });
                    futureThread = new Thread(futureTask);
                    futureThread.start();

                    executeResult = futureTask.get(triggerParam.getExecutorTimeout(), TimeUnit.SECONDS);
                } catch (TimeoutException e) {

                    XxlJobLogger.log("<br>----------- xxl-job job execute timeout");
                    XxlJobLogger.log(e);

                    executeResult = new ReturnT<String>(IJobHandler.FAIL_TIMEOUT.getCode(), "job execute timeout ");
                } finally {
                    futureThread.interrupt();
                }
            } else {
                // just execute
                executeResult = handler.execute(triggerParam.getExecutorParams());
            }

            if (executeResult == null) {
                executeResult = IJobHandler.FAIL;
            } else {
                executeResult.setMsg(
                    (executeResult!=null&&executeResult.getMsg()!=null&&executeResult.getMsg().length()>50000)
                    ?executeResult.getMsg().substring(0, 50000).concat("...")
                    :executeResult.getMsg());
                executeResult.setContent(null);	// limit obj size
            }
            XxlJobLogger.log("<br>----------- xxl-job job execute end(finish) -----------<br>----------- ReturnT:" + executeResult);

        } else {
            if (idleTimes > 30) {
                XxlJobExecutor.removeJobThread(jobId, "excutor idel times over limit.");
            }
        }
    } catch (Throwable e) {
        if (toStop) {
            XxlJobLogger.log("<br>----------- JobThread toStop, stopReason:" + stopReason);
        }

        StringWriter stringWriter = new StringWriter();
        e.printStackTrace(new PrintWriter(stringWriter));
        String errorMsg = stringWriter.toString();
        executeResult = new ReturnT<String>(ReturnT.FAIL_CODE, errorMsg);

        XxlJobLogger.log("<br>----------- JobThread Exception:" + errorMsg + "<br>----------- xxl-job job execute end(error) -----------");
    } finally {
        if(triggerParam != null) {
            // callback handler info
            if (!toStop) {
                // commonm
                TriggerCallbackThread.pushCallBack(new HandleCallbackParam(triggerParam.getLogId(), triggerParam.getLogDateTim(), executeResult));
            } else {
                // is killed
                ReturnT<String> stopResult = new ReturnT<String>(ReturnT.FAIL_CODE, stopReason + " [job running，killed]");
                TriggerCallbackThread.pushCallBack(new HandleCallbackParam(triggerParam.getLogId(), triggerParam.getLogDateTim(), stopResult));
            }
        }
    }
}

// callback trigger request in queue
while(triggerQueue !=null && triggerQueue.size()>0){
    TriggerParam triggerParam = triggerQueue.poll();
    if (triggerParam!=null) {
        // is killed
        ReturnT<String> stopResult = new ReturnT<String>(ReturnT.FAIL_CODE, stopReason + " [job not executed, in the job queue, killed.]");
        TriggerCallbackThread.pushCallBack(new HandleCallbackParam(triggerParam.getLogId(), triggerParam.getLogDateTim(), stopResult));
    }
}

// destroy
try {
    handler.destroy();
} catch (Throwable e) {
    logger.error(e.getMessage(), e);
}

logger.info(">>>>>>>>>>> xxl-job JobThread stoped, hashCode:{}", Thread.currentThread());
```


### 长/短/链式JOB注意事项

- 长JOB
  - 路由模式选择一致性HASH，让同一节点执行同一个JOB
  - 阻塞模式选择丢弃后续调度
- 短JOB
  - 路由模式选择随机
  - 阻塞模式选择覆盖之前调度，短JOB追求时效性
- 链式JOB
  - 采用主子JOB形式运行JOB，主JOB执行完毕之后，运行子JOB


### 高可用（HA）

- 注册中心
  - com.xxl.job.core.executor.XxlJobExecutor#initAdminBizList
- 执行节点（JOB）
  - 如果配置超时时间，另起一个Further线程执行Handler，在任务超时中止任务该线程（further调用get阻塞了自身线程，中止信号发送之后产生InterruptedException响应）
  - 如果没有配置超时时间，直接执行Handler
  - 如果任务调用出现异常（包括中止异常），生产错误信息，压入回调Queue
  - com.xxl.job.core.thread.TriggerCallbackThread会一直从回调Queue获取回调信息处理
  - 遍历执行器存储的调度中心列表
  - 调用com.xxl.job.core.biz.AdminBiz#callback方法（批量发送回调信息）
    - 获取已有调用JOB日志
    - 如果是成功调用的回调，并且有子任务，马上遍历触发子任务信息，子任务击发参照普通任务（**付为地提出主子任务运行，****子任务****适用什么任务配置**）
    - 处理回调信息，更新任务日志
  - 如果成功中止循环，记录回调成功日志（执行节点本地日志）
  - 注册中心启动时候会启动com.xxl.job.admin.core.thread.JobFailMonitorHelper#start
    - 获取错误执行日志信息（报警状态为0），改变报警状态到-1，防止重复处理（DB状态机）
    - 如果重试次数大于0，重新击发参照普通任务，重试次数-1，更新任务日志
    - 处理报警
    - 根据报警处理结果更新报警状态
