---
layout: post
title: 'SpringBoot Quartz 2.3.0'
date: 2018-07-30
author: yunfei
tags: SpringBoot Quartz
---

### 1、添加依赖
````xml
<dependency>
   <groupId>org.quartz-scheduler</groupId>
   <artifactId>quartz</artifactId>
   <version>2.3.0</version>
</dependency>
<dependency>
   <groupId>org.quartz-scheduler</groupId>
   <artifactId>quartz-jobs</artifactId>
   <version>2.3.0</version>
</dependency>
<dependency>
   <groupId>org.springframework</groupId>
   <artifactId>spring-context-support</artifactId>
</dependency>
````

### 2、SchedulerConfig
````java
import java.io.IOException;
import java.util.Properties;
 
import org.quartz.Scheduler;
import org.quartz.ee.servlet.QuartzInitializerListener;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.config.PropertiesFactoryBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;
import org.springframework.scheduling.quartz.SchedulerFactoryBean;
 
import javax.sql.DataSource;
 
/**
*
* @author wangzengfeng
* @date  2018/7/30 11:06
*/
@Configuration
public class SchedulerConfig {
 
    @Qualifier("dataSource")
    @Autowired
    private DataSource dataSource;
    
    @Bean(name="SchedulerFactory")
    public SchedulerFactoryBean schedulerFactoryBean() throws IOException {
        SchedulerFactoryBean factory = new SchedulerFactoryBean();
        // 设置dataSource 跟项目使用同一个dataSource，会覆盖quartz.properties里面的配置
        factory.setDataSource(dataSource);
        factory.setQuartzProperties(quartzProperties());
        return factory;
    }
 
    @Bean
    public Properties quartzProperties() throws IOException {
        PropertiesFactoryBean propertiesFactoryBean = new PropertiesFactoryBean();
        propertiesFactoryBean.setLocation(new ClassPathResource("/quartz.properties"));
        //在quartz.properties中的属性被读取并注入后再初始化对象
        propertiesFactoryBean.afterPropertiesSet();
        return propertiesFactoryBean.getObject();
    }
 
  
    /**
    *  quartz初始化监听器
    */
    @Bean
    public QuartzInitializerListener executorListener() {
       return new QuartzInitializerListener();
    }
 
    /**
     * 通过SchedulerFactoryBean获取Scheduler的实例
     */
    @Bean(name="Scheduler")
    public Scheduler scheduler() throws IOException {
        return schedulerFactoryBean().getScheduler();
    }
}
````
### 3、quartz.properties
```yaml
# 固定前缀org.quartz
# 主要分为scheduler、threadPool、jobStore、plugin等部分
org.quartz.scheduler.instanceName=DefaultQuartzScheduler
org.quartz.scheduler.rmi.export=false
org.quartz.scheduler.rmi.proxy=false
org.quartz.scheduler.wrapJobExecutionInUserTransaction=false
# 实例化ThreadPool时，使用的线程类为SimpleThreadPool
org.quartz.threadPool.class=org.quartz.simpl.SimpleThreadPool
# threadCount和threadPriority将以setter的形式注入ThreadPool实例
# 并发个数
org.quartz.threadPool.threadCount=5
# 优先级
org.quartz.threadPool.threadPriority=5
org.quartz.threadPool.threadsInheritContextClassLoaderOfInitializingThread=true
org.quartz.jobStore.misfireThreshold=5000
# 默认存储在内存中
#org.quartz.jobStore.class = org.quartz.simpl.RAMJobStore
#持久化
org.quartz.jobStore.class=org.quartz.impl.jdbcjobstore.JobStoreTX
#持久化方式配置数据驱动，MySQL数据库
org.quartz.jobStore.driverDelegateClass = org.quartz.impl.jdbcjobstore.StdJDBCDelegate
org.quartz.jobStore.tablePrefix=QRTZ_
org.quartz.jobStore.dataSource=qzDS
org.quartz.dataSource.qzDS.driver=com.mysql.jdbc.Driver
org.quartz.dataSource.qzDS.URL=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF-8
org.quartz.dataSource.qzDS.user=root
org.quartz.dataSource.qzDS.password=root
org.quartz.dataSource.qzDS.maxConnections=10
management.security.enabled=false
```
### 4、SchedulerJob 任务执行实现
````java
import org.quartz.Job;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
import java.util.Date;
 
/**
 * @author wangzengfeng
 * @date 2018/7/26 11:09
 */
public class SchedulerJob implements Job {
    private Logger logger = LoggerFactory.getLogger(this.getClass());
 
    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
 
        //这里可以获取控制器绑定的值，实际应用中可以设置为某个活动的id,以便进行数据库操作
        Object jobName = jobExecutionContext.getJobDetail().getKey();
        logger.info("定时任务[{}]开始执行>{}", jobName, new Date());
    }
}
````
### 5、使用例子
````java
  public class JobTest{
    @Autowired
    @Qualifier("Scheduler")
    private Scheduler scheduler;
     
    //添加开启任务
    @Override
    public Object jobStart(String jobName, String groupName, String corn) {
        //配置定时任务对应的Job，这里执行的是ScheduledJob类中定时的方法
        JobDetail jobDetail = JobBuilder
                .newJob(SchedulerJob.class)
                .usingJobData("jobName", jobName)
                .withIdentity(jobName, groupName)
                .build();
     
        CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule(corn);
        CronTrigger cronTrigger = TriggerBuilder.newTrigger()
                .withIdentity("trigger-" + jobName, groupName)
                .withSchedule(scheduleBuilder)
                .build();
     
        try {
            scheduler.scheduleJob(jobDetail, cronTrigger);
        } catch (SchedulerException e) {
            logger.error("任务启动失败>{}", e);
        }
        return jobName;
    }
    @Override
        public void jobPause(String jobName, String groupName) {
            try {
                scheduler.pauseJob(JobKey.jobKey(jobName, groupName));
            } catch (SchedulerException e) {
                logger.error("任务暂停失败>{}", e);
            }
        }
     
        @Override
        public void jobDelete(String jobName, String groupName) {
            try {
                scheduler.pauseTrigger(TriggerKey.triggerKey(jobName, groupName));
                scheduler.unscheduleJob(TriggerKey.triggerKey(jobName, groupName));
                scheduler.deleteJob(JobKey.jobKey(jobName, groupName));
            } catch (SchedulerException e) {
                logger.error("任务删除失败>{}", e);
            }
        }
     
        @Override
        public void jobResume(String jobName, String groupName) {
            try {
                scheduler.resumeJob(JobKey.jobKey(jobName, groupName));
            } catch (SchedulerException e) {
                logger.error("任务重置失败>{}", e);
            }
        }
     
        @Override
        public void jobReschedule(String jobName, String groupName, String cron) {
            try {
                TriggerKey triggerKey = TriggerKey.triggerKey(jobName, groupName);
                // 表达式调度构建器
                CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule(cron);
                CronTrigger trigger = (CronTrigger) scheduler.getTrigger(triggerKey);
                // 按新的cronExpression表达式重新构建trigger
                trigger = trigger.getTriggerBuilder().withIdentity(triggerKey).withSchedule(scheduleBuilder).build();
                // 按新的trigger重新设置job执行
                scheduler.rescheduleJob(triggerKey, trigger);
            } catch (SchedulerException e) {
                logger.error("更新任务失败>{}", e);
            }
        }
     
        @Override
        public Object getJobs() {
            Set<JobKey> jobKeys = new HashSet<JobKey>();
            try {
                Set<TriggerKey> triggerKeys = scheduler.getTriggerKeys(GroupMatcher.anyTriggerGroup());
                //获取所有的job集合
                jobKeys = scheduler.getJobKeys(GroupMatcher.anyJobGroup());
            } catch (SchedulerException e) {
                logger.error("获取任务列表失败>{}", e);
            }
            return jobKeys;
        }
     
        @Override
        public String getJobStatus(String jobName, String groupName) {
            TriggerKey triggerKey = TriggerKey.triggerKey(jobName, groupName);
            String name = null;
            try {
                Trigger.TriggerState state = scheduler.getTriggerState(triggerKey);
                name = state.name();
                logger.info(state.name());
            } catch (SchedulerException e) {
                logger.error(e.getMessage(), e);
            }
            return name;
        }
     
        @Override
        public ReturnDto getJobsAndStatus() {
            List<Map<String, Object>> data = new ArrayList<Map<String, Object>>();
            ReturnDto returnDto = new ReturnDto();
            try {
                Set<TriggerKey> triggerKeys = scheduler.getTriggerKeys(GroupMatcher.anyTriggerGroup());
                for (TriggerKey triggerKey : triggerKeys) {
                    Trigger.TriggerState state = scheduler.getTriggerState(triggerKey);
                    Map<String, Object> job = new HashMap<>(5);
                    job.put("triggerName", triggerKey.getName());
                    job.put("group", triggerKey.getGroup());
                    job.put("statusKey", state.name());
                    job.put("statusName", JobStatusEnum.getStatusName(state.name()));
                    data.add(job);
                }
     
    //            Set<JobKey> jobKeys = scheduler.getJobKeys(GroupMatcher.anyJobGroup());
    //            for(JobKey jobKey : jobKeys){
    //                JobDetail jobDetail = scheduler.getJobDetail(jobKey);
    //            }
            } catch (SchedulerException e) {
                logger.error("获取任务列表失败>{}", e);
            }
            returnDto.setData(data);
            returnDto.setCode(Constants.ReturnType.CODE_SUCCESS);
            returnDto.setMessage(Constants.ReturnType.MESSAGE_SUCCESS);
            return returnDto;
        }
     
        @Override
        public ReturnDto getCurrentExecutingJobs() {
            ReturnDto returnDto = new ReturnDto();
            try {
                List<JobExecutionContext> jobs = scheduler.getCurrentlyExecutingJobs();
                returnDto.setData(jobs);
                returnDto.setCode(Constants.ReturnType.CODE_SUCCESS);
                returnDto.setMessage(Constants.ReturnType.MESSAGE_SUCCESS);
            } catch (SchedulerException e) {
                logger.error("获取当前正在执行任务列表失败>{}", e);
            }
            return returnDto;
        }
  }
````


