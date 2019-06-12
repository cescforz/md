Spring 事务

```
public interface TransactionDefinition {
    int PROPAGATION_REQUIRED = 0;
    int PROPAGATION_SUPPORTS = 1;
    int PROPAGATION_MANDATORY = 2;
    int PROPAGATION_REQUIRES_NEW = 3;
    int PROPAGATION_NOT_SUPPORTED = 4;
    int PROPAGATION_NEVER = 5;
    int PROPAGATION_NESTED = 6;
    int ISOLATION_DEFAULT = -1;
    int ISOLATION_READ_UNCOMMITTED = 1;
    int ISOLATION_READ_COMMITTED = 2;
    int ISOLATION_REPEATABLE_READ = 4;
    int ISOLATION_SERIALIZABLE = 8;
    int TIMEOUT_DEFAULT = -1;
    
    int getPropagationBehavior();//返回定义的事务传播行为
	int getIsolationLevel();//返回事务隔离级别
	int getTimeout();//返回定义的事务超时时间
	boolean isReadOnly();//返回定义的事务是否是只读的
	String getName();//返回事务名称
}

public interface TransactionStatus extends SavepointManager, Flushable {
    boolean isNewTransaction();//返回当前事务是否是新的事务
    boolean hasSavepoint();//返回当前事务是否有保存点
	void setRollbackOnly();//设置事务回滚
	boolean isRollbackOnly();//设置当前事务是否应该回滚
	void flush();//用于刷新底层会话中的修改到数据库，一般用于刷新如Hibernate/JPA的会话，可能对如JDBC类型的事务无任何影响；
	boolean isCompleted();//返回事务是否完成
}
事务的传播特性
1.PROPAGATION_REQUIRED：支持当前事务，如当前没有事务，则新建一个。
2.PROPAGATION_SUPPORTS：支持当前事务，如当前没有事务，则已非事务性执行。
3.PROPAGATION_MANDATORY：支持当前事务，如当前没有事务，则抛出异常（强制一定要在一个已经存在的事务中执行，业务方法不可独自发起自己的事务）。
4.PROPAGATION_REQUIRES_NEW：始终新建一个事务，如当前原来有事务，则把原事务挂起。
5.PROPAGATION_NOT_SUPPORTED：不支持当前事务，始终已非事务性方式执行，如当前事务存在，挂起该事务。
6.PROPAGATION_NEVER：不支持当前事务；如果当前事务存在，则引发异常。
7.PROPAGATION_NESTED：如果当前事务存在，则在嵌套事务中执行，如果当前没有事务，则执行与 PROPAGATION_REQUIRED 类似的操作（注意：当应用到JDBC时，只适用JDBC 3.0以上驱动）。

事物的隔离级别
① Serializable (串行化)：可避免脏读、不可重复读、幻读的发生。
② Repeatable read (可重复读)：可避免脏读、不可重复读的发生。
③ Read committed (读已提交)：可避免脏读的发生。
④ Read uncommitted (读未提交)：最低级别，任何情况都无法保证。

数据库事务的隔离级别有4种，由低到高分别为Read uncommitted 、Read committed 、Repeatable read 、Serializable 。而且，在事务的并发操作中可能会出现脏读，不可重复读，幻读。下面通过事例一一阐述它们的概念与联系。

Read uncommitted

读未提交，顾名思义，就是一个事务可以读取另一个未提交事务的数据。

事例：老板要给程序员发工资，程序员的工资是3.6万/月。但是发工资时老板不小心按错了数字，按成3.9万/月，该钱已经打到程序员的户口，但是事务还没有提交，就在这时，程序员去查看自己这个月的工资，发现比往常多了3千元，以为涨工资了非常高兴。但是老板及时发现了不对，马上回滚差点就提交了的事务，将数字改成3.6万再提交。

分析：实际程序员这个月的工资还是3.6万，但是程序员看到的是3.9万。他看到的是老板还没提交事务时的数据。这就是脏读。

那怎么解决脏读呢？Read committed！读提交，能解决脏读问题。

Read committed

读提交，顾名思义，就是一个事务要等另一个事务提交后才能读取数据。

事例：程序员拿着信用卡去享受生活（卡里当然是只有3.6万），当他埋单时（程序员事务开启），收费系统事先检测到他的卡里有3.6万，就在这个时候！！程序员的妻子要把钱全部转出充当家用，并提交。当收费系统准备扣款时，再检测卡里的金额，发现已经没钱了（第二次检测金额当然要等待妻子转出金额事务提交完）。程序员就会很郁闷，明明卡里是有钱的…

分析：这就是读提交，若有事务对数据进行更新（UPDATE）操作时，读操作事务要等待这个更新操作事务提交后才能读取数据，可以解决脏读问题。但在这个事例中，出现了一个事务范围内两个相同的查询却返回了不同数据，这就是不可重复读。

那怎么解决可能的不可重复读问题？Repeatable read ！

Repeatable read

重复读，就是在开始读取数据（事务开启）时，不再允许修改操作

事例：程序员拿着信用卡去享受生活（卡里当然是只有3.6万），当他埋单时（事务开启，不允许其他事务的UPDATE修改操作），收费系统事先检测到他的卡里有3.6万。这个时候他的妻子不能转出金额了。接下来收费系统就可以扣款了。

分析：重复读可以解决不可重复读问题。写到这里，应该明白的一点就是，不可重复读对应的是修改，即UPDATE操作。但是可能还会有幻读问题。因为幻读问题对应的是插入INSERT操作，而不是UPDATE操作。

什么时候会出现幻读？

事例：程序员某一天去消费，花了2千元，然后他的妻子去查看他今天的消费记录（全表扫描FTS，妻子事务开启），看到确实是花了2千元，就在这个时候，程序员花了1万买了一部电脑，即新增INSERT了一条消费记录，并提交。当妻子打印程序员的消费记录清单时（妻子事务提交），发现花了1.2万元，似乎出现了幻觉，这就是幻读。

那怎么解决幻读问题？Serializable！

Serializable 序列化

Serializable 是最高的事务隔离级别，在该级别下，事务串行化顺序执行，可以避免脏读、不可重复读与幻读。但是这种事务隔离级别效率低下，比较耗数据库性能，一般不使用。

值得一提的是：大多数数据库默认的事务隔离级别是Read committed，比如Sql Server , Oracle。Mysql的默认隔离级别是Repeatable read。
```



编程式事务

```java
1.配置事务管理
<!-- (事务管理)transaction manager, use JtaTransactionManager for global tx -->
	<bean id="transactionManager"
		class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"/>
	</bean>
	<tx:annotation-driven transaction-manager="transactionManager"/>
	
2.依赖注入
@Resource(name="transactionManager")
private DataSourceTransactionManager transactionManager;

public void testdelivery(){  
        //获取事务定义
        DefaultTransactionDefinition def = new DefaultTransactionDefinition();
        //定义事务隔离级别
        def.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED); 
        //定义事务传播行为 
        def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);    
        TransactionStatus status = transactionManager.getTransaction(def);   
        
        try {
			//逻辑代码，可以写上你的逻辑处理代码
			transactionManager.commit(status);
		} catch (Exception e) {
			transactionManager.rollback(status);
		}
} 
```

声明式事务

```java
@Transactional(rollbackFor = Exception.class)
public boolean update() {
   try {
      // 操作A
      this.doA();
      // 操作B
      this.doB()；
   } catch (Exception e) {
      log.error("事务失败", e);
      return Boolean.FALSE;
   }
   return Boolean.TRUE;
}

=====》以上事务不会回滚《=====
操作A成功，B抛出异常，但是被catch住并处理，这样异常并没有抛出来，而Spring的声明式事务是基于AOP的，所以也就认为这段操作没有异常，虽然返回的失败，但是A操作已经被提交。

====================================================================================
@Transactional(rollbackFor = Exception.class)
public boolean update() {
   try {
      // 操作A
      this.doA();
      // 操作B
      this.doB()；
   } catch (Exception e) {
      log.error("事务失败", e);
      throw new RuntimeException(); 
   }
   return Boolean.TRUE;
}
======》以上事务会回滚《======
原因：
Spring aop异常捕获原理：被拦截的方法需显式抛出异常，并不能经任何处理，这样aop代理才能捕获到方法的异常，才能进行回滚，默认情况下aop只捕获runtimeexception的异常，但可以通过配置来捕获特定的异常并回滚  
换句话说不使用try catch 或者在catch中最后加上throw new runtimeexcetpion（），这样程序异常时才能被aop捕获进而回滚.
    
====================================================================================
声明式事务的手动控制回滚：
Spring的事务一般分为声明式事务（或叫注解式事务）和编程式事务。
1.编程式事务比较灵活，可以将事务的粒度控制的更细，并且可以控制何时提交，哪种情况回滚。
2.声明式事务使用@Transactional注解，使用起来没有那么灵活，但是对业务代码没有入侵，而且本身支持一些异常情况下的回滚。但是这个异常是不可以捕获的，如果代码里捕获了，那么事务中已经执行了的部分会提交(事务不会回滚)

操作A成功，B抛出异常，但是被catch住并处理，这样异常并没有抛出来，而Spring的声明式事务是基于AOP的，所以也就认为这段操作没有异常，虽然返回的失败，但是A操作已经被提交。
那么如果想要自己控制异常的时候的返回结果，由希望事务回滚要怎么操作呢？
其实只要加一段代码就可以，上面的catch语句中修改为如下：

@Transactional(rollbackFor = Exception.class)
public boolean update() {
   try {
      // 操作A
      this.doA();
      // 操作B
      this.doB()；
   } catch (Exception e) {
      log.error("事务失败", e);
      TransactionAspectSupport.currentTransactionStatus().setRollbackOnly(); 
   }
   return Boolean.TRUE;
}
```
