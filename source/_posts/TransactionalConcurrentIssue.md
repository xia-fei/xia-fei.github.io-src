---
title: 记一次线上事务并发问题
date: 2018-07-07 21:00:29
tags: bug
---

今天一同事线上遇到一个问题，程序不明原因的进入了死循环。最后通过一步步分析代码的线程运行情况，定位出是事务产生的并发问题
<!-- more -->

##### 我们看下代码入口。
```java
    @Transactional
	public Result<Boolean> execute(Map<String, String> currentRow, Map<String, String> contextInfo) {
        ...
         synchronized (this) {
              userVipCardDO.setCardNo(userVipCardService.getVipCardNo(contextInfo.get("storeId")));
         }             
	}
```
>此方法可能多线程运行


#### 第2个方法代码    
```java
private int getVipCardNo(String storeId) {
        //获取数据库当前值
        SerialNumberDO serialNumberDO = serialNumberDao.getByKey(Long.parseLong(storeId), SerialBusiType.HY.getCode());
        if (serialNumberDO == null) {
            ...
            return 1;
        }
        //查出数据库当前值
        long counter = serialNumberDO.getCounter();
        SerialNumberDO updateDO = new SerialNumberDO();
        ...
        updateDO.setCounter(counter);
        int i = serialNumberDao.increase(updateDO);
        if(i == 0){
            return getVipCardNo(storeId);
        }else {
            return counter;
        }
    }
```
>相当于一个乐观锁 compare And Set  
>此处呢,由于外面synchronized单线程运行  
>这是一个递归调用  

#### 第3个dao层sql
```xml
 <update id="increase" parameterType="SerialNumberDO">
        update sh_serial_number
        set counter = counter+1
        where store_id = #{storeId}
        and busi_type = #{busiType}
        and counter = #{counter}
</update>
```
>乐观锁sql  

问题出在哪呢。  

1. 第一断代码，当时所有线程进来读到的值都是**8**  
2. `@Transactional` 默认事务隔离级别`Repeatable read` 可重复读  
    1. 这就意味着第一步多个线程进来后。读到的值，其它事务进行修改提交之后。它还是读的当 时进来的这个值**8**。   
1. 第一个线程读到**8** 当时数据库确实是**8** 所以修改成  **9**  修改成功  
1. 第一个之后的线程呢 。  
    1. 当时进来时开启事务数据库里是**8**
    1. synchronized等待
    1. 等第一个修改成功之后进去select
    1. 由于事务隔离级别`Repeatable read`的原因，虽然数据库里改成了**9**,它这里仍然读到是**8**
    1. 然后`update .... and counter=#{counter}` 比较的时候呢，却是**比较数据库真实的值**，所有 `8=9`一直不成功
    1. 一直重试递归调用 进入了死循环
    
##### 问题找到了，那么最简单的解决方案就是`@Transactional(isolation = Isolation.READ_COMMITTED)` 将隔离级别降低一级。读取已提交的   
最后分下这个代码

1. 如果不加`synchronized`其实也会出问题。只是出的几率小一点。    
1. 为了其它service方法使用默认隔离级别，调用`getVipCardNo` 不出类似问题。我觉得应该利用事务传播行为，
将`getVipCardNo`开启一个新的事务，再将隔离级别设置成读取已提交的