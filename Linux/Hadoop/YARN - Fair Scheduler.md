```txt
假设在生产环境Yarn中，总共有四类用户需要使用集群，开发用户、测试用户、业务1用户、业务2用户。
为了使其提交的任务不受影响，我们在Yarn上规划配置了五个资源池，分别为：
    dev_group（开发用户组资源池）
    test_group（测试用户组资源池）
    business1_group（业务1用户组资源池）
    business2_group（业务2用户组资源池）
    default（只分配了极少资源）。并根据实际业务情况，为每个资源池分配了相应的资源及优先级等。

这样，每个用户组下的用户提交任务时候，会到相应的资源池中，而不影响其他业务
```
#### ResourceManager 的 fair-scheduler.xml
```xml
<?xml version="1.0"?>
<allocations>  
  <!-- users max running apps -->
  <userMaxAppsDefault>30</userMaxAppsDefault>
<queue name="root">
  <aclSubmitApps> </aclSubmitApps>
  <aclAdministerApps> </aclAdministerApps>
  
  <queue name="default">
          <minResources>2000mb,1vcores</minResources>
          <maxResources>10000mb,1vcores</maxResources>
          <maxRunningApps>1</maxRunningApps>
          <schedulingMode>fair</schedulingMode>
          <weight>0.5</weight>
          <aclSubmitApps>*</aclSubmitApps>
  </queue>
       
  <queue name="dev_group">
          <minResources>200000mb,33vcores</minResources>
          <maxResources>300000mb,90vcores</maxResources>
          <maxRunningApps>150</maxRunningApps>
          <schedulingMode>fair</schedulingMode>
          <weight>2.5</weight>
          <aclSubmitApps> dev_group</aclSubmitApps>
          <aclAdministerApps> hadoop,dev_group</aclAdministerApps>
  </queue>
  
  <queue name="test_group">
          <minResources>70000mb,20vcores</minResources>
          <maxResources>95000mb,25vcores</maxResources>
          <maxRunningApps>60</maxRunningApps>
          <schedulingMode>fair</schedulingMode>
          <weight>1</weight>
          <aclSubmitApps> test_group</aclSubmitApps>
          <aclAdministerApps> hadoop,test_group</aclAdministerApps>
  </queue>
                                                                          
  <queue name="business1_group">
          <minResources>75000mb,15vcores</minResources>
          <maxResources>100000mb,20vcores</maxResources>
          <maxRunningApps>80</maxRunningApps>
          <schedulingMode>fair</schedulingMode>
          <weight>1</weight>
          <aclSubmitApps> business1_group</aclSubmitApps>
          <aclAdministerApps> hadoop,business1_group</aclAdministerApps>
  </queue>
                                                             
                                                                          
  <queue name="business2_group">
      <minResources>75000mb,15vcores</minResources>
      <maxResources>102400mb,20vcores</maxResources>
      <maxRunningApps>80</maxRunningApps>
      <schedulingMode>fair</schedulingMode>
      <weight>1</weight>
      <aclSubmitApps> business2_group</aclSubmitApps>
      <aclAdministerApps> hadoop,business2_group</aclAdministerApps>
  </queue>
 
</queue>
  <queuePlacementPolicy>
      <rule name="primaryGroup" create="false" />
      <rule name="secondaryGroupExistingQueue" create="false" />
      <rule name="default" />
  </queuePlacementPolicy>
 
</allocations>
```
```txt
需要注意的是，所有客户端提交任务的用户和用户组的对应关系需要维护在ResourceManager上
ResourceManager在分配资源池时候，是从ResourceManager上读取用户和用户组的对应关系的，否则就会被分配到default资源池
在日志中出现”UserGroupInformation: No groups available for user”类似的警告。而客户端机器上的用户对应的用户组无关紧要
```
#### 使调度修改生效
```bash
#每次在ResourceManager上新增用户或者调整资源池配额后，需要执行下面的命令刷新使其生效
yarn rmadmin -refreshQueues
yarn rmadmin -refreshUserToGroupsMappings

```