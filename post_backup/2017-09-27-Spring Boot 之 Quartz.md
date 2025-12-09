---
layout:     post
title:     	Spring Boot 之 Quartz
subtitle:    "\"Spring Boot 之 Quartz\""
date:       2017-09-27
author:     Mr Chang
header-img: img/post-bg-debug.png
catalog: true
tags:
    - Spring Boot
    - Quartz
    - 定时任务

---


## POM

	<dependencies>
         <dependency>
            <groupId>org.quartz-scheduler</groupId>
            <artifactId>quartz</artifactId>
            <version>2.2.1</version>
        </dependency>
    </dependencies>	 
	 
	
## application.properties
	
	quartz.enabled=true
	samplejob.frequency=2000
	
	

	
	
## quartz.properties
		

	org.quartz.scheduler.instanceName=spring-boot-quartz-demo
	org.quartz.scheduler.instanceId=AUTO
	org.quartz.threadPool.threadCount=5
	org.quartz.jobStore.class=org.quartz.impl.jdbcjobstore.JobStoreTX
	org.quartz.jobStore.driverDelegateClass=org.quartz.impl.jdbcjobstore.StdJDBCDelegate
	org.quartz.jobStore.useProperties=true
	org.quartz.jobStore.misfireThreshold=60000
	org.quartz.jobStore.tablePrefix=QRTZ_
	org.quartz.jobStore.isClustered=true
	org.quartz.jobStore.clusterCheckinInterval=20000	
	
	    
## SchedulerConfig

		@Configuration
		@ConditionalOnProperty(name = "quartz.enabled")
		public class SchedulerConfig {
		
		    private static JobDetailFactoryBean createJobDetail(Class jobClass) {
		        JobDetailFactoryBean factoryBean = new JobDetailFactoryBean();
		        factoryBean.setJobClass(jobClass);
		        // job has to be durable to be stored in DB:
		        factoryBean.setDurability(true);
		        return factoryBean;
		    }
		
		    private static SimpleTriggerFactoryBean createTrigger(JobDetail jobDetail, long pollFrequencyMs) {
		        SimpleTriggerFactoryBean factoryBean = new SimpleTriggerFactoryBean();
		        factoryBean.setJobDetail(jobDetail);
		        factoryBean.setStartDelay(0L);
		        factoryBean.setRepeatInterval(pollFrequencyMs);
		        factoryBean.setRepeatCount(SimpleTrigger.REPEAT_INDEFINITELY);
		        // in case of misfire, ignore all missed triggers and continue :
		        factoryBean.setMisfireInstruction(SimpleTrigger.MISFIRE_INSTRUCTION_RESCHEDULE_NEXT_WITH_REMAINING_COUNT);
		        return factoryBean;
		    }
		
		    // Use this method for creating cron triggers instead of simple triggers:
		    private static CronTriggerFactoryBean createCronTrigger(JobDetail jobDetail, String cronExpression) {
		        CronTriggerFactoryBean factoryBean = new CronTriggerFactoryBean();
		        factoryBean.setJobDetail(jobDetail);
		        factoryBean.setCronExpression(cronExpression);
		        factoryBean.setMisfireInstruction(SimpleTrigger.MISFIRE_INSTRUCTION_FIRE_NOW);
		        return factoryBean;
		    }
		
		    // injecting SpringLiquibase to ensure liquibase is already initialized and created the quartz tables:
		    @Bean
		    public JobFactory jobFactory(ApplicationContext applicationContext) {
		        AutowiringSpringBeanJobFactory jobFactory = new AutowiringSpringBeanJobFactory();
		        jobFactory.setApplicationContext(applicationContext);
		        return jobFactory;
		    }
		
		    @Bean
		    public SchedulerFactoryBean schedulerFactoryBean(DataSource dataSource, JobFactory jobFactory, @Qualifier("sampleJobTrigger") Trigger sampleJobTrigger) throws IOException {
		        SchedulerFactoryBean factory = new SchedulerFactoryBean();
		        // this allows to update triggers in DB when updating settings in config file:
		        factory.setOverwriteExistingJobs(true);
		        factory.setDataSource(dataSource);
		        factory.setJobFactory(jobFactory);
		        factory.setQuartzProperties(quartzProperties());
		
		        return factory;
		    }
		
		    @Bean
		    public Properties quartzProperties() throws IOException {
		        PropertiesFactoryBean propertiesFactoryBean = new PropertiesFactoryBean();
		        propertiesFactoryBean.setLocation(new ClassPathResource("/quartz.properties"));
		        propertiesFactoryBean.afterPropertiesSet();
		        return propertiesFactoryBean.getObject();
		    }
		
		    @Bean
		    public JobDetailFactoryBean sampleJobDetail() {
		        return createJobDetail(SampleJob.class);
		    }
		
		    @Bean(name = "sampleJobTrigger")
		    public SimpleTriggerFactoryBean sampleJobTrigger(@Qualifier("sampleJobDetail") JobDetail jobDetail, @Value("${samplejob.frequency}") long frequency) {
		        return createTrigger(jobDetail, frequency);
		    }
		
		}	
	}

	
## SampleJob
	
	public class SampleJob implements Job {
	
	    @Override
	    public void execute(JobExecutionContext jobExecutionContext) {
	        System.out.printf("SampleJob 执行 - 时间： " + System.currentTimeMillis());
	    }
	}
	

## JobController

	@Controller
	@RequestMapping(value = "jobController")
	@Api(value = "定时任务管理")
	public class JobController {
	    @SuppressWarnings("SpringJavaAutowiringInspection")
	    @Autowired
	    private SchedulerFactoryBean schedulerFactory;
	
	    @RequestMapping(value = "initJob", method = RequestMethod.GET)
	    @ResponseBody
	    @ApiOperation(value = "初始化定时任务", notes = "根据URL的num来初始化定时任务")
	    public ResponseMessage initJob() {
	
	        Scheduler scheduler = schedulerFactory.getScheduler();
	        JobDetail job = newJob(CleanUseFlowJob.class)
	                .withIdentity("清空Pro用户使用流量", "Pro组")
	                .build();
	
	
	        Trigger trigger = newTrigger()
	                .withIdentity("清空Pro用户使用流量", "Pro组")
	                .startNow()
	                .withSchedule(
	                        cronSchedule("0 0 1 1 * ?")//每月1号凌晨1点执行
	                ).build();
	        // Tell quartz to schedule the job using our trigger
	        try {
	            scheduler.scheduleJob(job, trigger);
	        } catch (SchedulerException e) {
	            e.printStackTrace();
	        }
		    
		    
## 参考

**代码：https://github.com/changdaye/spring-boot-demo/tree/master/spring-boot-quartz**


![](http://cdn-blog.jetbrains.org.cn/17-10-11/92200804.jpg)