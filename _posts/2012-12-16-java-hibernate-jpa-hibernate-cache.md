---
layout: post
title:  "JPA+Hibernate各种缓存的配置演示"
desc: "JPA+Hibernate各种缓存的配置演示"
keywords: "JPA+Hibernate各种缓存的配置演示,cache,缓存,hibernate,java,kinglyjn"
date: 2012-12-16
categories: [Java]
tags: [java]
icon: fa-coffee
---

> <i style="font-size:12px;">JPA itself is just a specification, not a product; it cannot perform persistence or anything else by itself.</i> <br>
> <i style="font-size:12px;">JPA仅仅只是一个规范，而不是产品；使用JPA本身是不能做到持久化的。</i> <br>

JPA只是一系列定义好的持久化操作的接口，在系统中使用时，需要真正的实现者，在这里，我们使用Hibernate作为实现者。<br>

JPA规范中定义了很多的缓存类型：一级缓存，二级缓存，对象缓存，数据缓存，等等一系列概念，此文章示例基于hibernate提供的JPA缓存实现。很多其他的JPA实现者，比如toplink（EclipseLink），也许还有其他的各种缓存实现，在此就不说了。 <br>

hibernate实现中只有三种缓存类型：<br> 

* 一级缓存
* 二级缓存
* 查询缓存 

在hibernate的实现概念里，他把什么集合缓存之类的统一放到二级缓存里去了。<br><br>


### 一级缓存测试

```xml
<bean id="entityManagerFactory"  
    class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">  
    <property name="dataSource" ref="dataSource" />  
    <property name="jpaVendorAdapter" ref="hibernateJpaVendorAdapter" />  
    <property name="packagesToScan" value="com.restjplat.quickweb" />  
    <property name="jpaProperties">  
        <props>  
            <prop key="hibernate.show_sql">${hibernate.show_sql}</prop>  
            <prop key="hibernate.format_sql">true</prop>  
        </props>  
    </property>  
</bean>  
```

可见没有添加任何配置项。 

```java
private void firstCacheTest(){    
    EntityManager em = emf.createEntityManager();  
    Dict d1 = em.find(Dict.class, 1); //find id为1的对象  
    Dict d2 = em.find(Dict.class, 1); //find id为1的对象  
    logger.info((d1==d2)+""); //true  
  
    EntityManager em1 = emf.createEntityManager();  
    Dict d3 = em1.find(Dict.class, 1); //find id为1的对象  
    EntityManager em2 = emf.createEntityManager();  
    Dict d4 = em2.find(Dict.class, 1); //find id为1的对象  
    logger.info((d3==d4)+""); //false  
}  


//运行结果
输出为：因为sql语句打出来太长，所以用*号代替  
Hibernate: ***********  
2014-03-17 20:41:44,819  INFO [main] (DictTest.java:76) - true  
Hibernate: ***********  
Hibernate: ***********  
2014-03-17 20:41:44,869  INFO [main] (DictTest.java:84) - false  
```

由此可见：同一个session内部，一级缓存生效，同一个id的对象只有一个。不同session，一级缓存无效。 <br><br>



### 二级缓存测试


文件配置： <br>

1:实体类直接打上 javax.persistence.Cacheable 标记。<br>

```java
@Entity  
@Table(name ="dict")  
@Cacheable  
public class Dict extends IdEntity{}  
``` 

2：配置文件修改，在 jpaProperties <br>下添加，用ehcache来实现二级缓存，另外因为加入了二级缓存，我们将hibernate的统计打开来看看到底是不是被缓存了。<br>

```xml
<prop key="hibernate.cache.region.factory_class">org.hibernate.cache.ehcache.EhCacheRegionFactory</prop> 
<prop key="javax.persistence.sharedCache.mode">ENABLE_SELECTIVE</prop>  
<prop key="hibernate.generate_statistics">true</prop>  
```

注1：如果在配置文件中加入了 <prop key="javax.persistence.sharedCache.mode">ENABLE_SELECTIVE</prop>，则不需要在实体内配置hibernate的 @cache标记，只要打上JPA的@cacheable标记即可默认开启该实体的2级缓存。<br>

注2：如果不使用javax.persistence.sharedCache.mode配置，直接在实体内打@cache标记也可以。<br>

```java
@Cache(usage = CacheConcurrencyStrategy.READ_ONLY)  
public class Dict extends IdEntity{}  
``` 

至于 hibernate的 hibernate.cache.use_second_level_cache这个属性，文档里是这么写的： <br>

> <i style="font-size:12px;"> Can be used to completely disable the second level cache, which is enabled by default for classes which specify a \<cache\> mapping.</i><br>

即打上只要有@cache标记，自动开启。<br>


`所以有两种方法配置开启二级缓存`： <br>

1. 第一种不使用hibernate的@cache标记，直接用@cacheable标记和缓存映射配置项。<br> 
2. 第二种用hibernate的@cache标记使用。 <br><br>

另外javax.persistence.sharedCache.mode的其他配置如下：<br>

The javax.persistence.sharedCache.mode property can be set to one of the following values: <br>

* ENABLE_SELECTIVE (Default and recommended value): entities are not cached unless explicitly marked as cacheable.
* DISABLE_SELECTIVE: entities are cached unless explicitly marked as not cacheable.
* NONE: no entity are cached even if marked as cacheable. This option can make sense to disable second-level cache altogether.
* ALL: all entities are always cached even if marked as non cacheable. 

如果用all的话，连实体上的@cacheable都不用打，直接默认全部开启二级缓存 <br><br>

测试代码： <br>

```java
private void secondCachetest(){  
    EntityManager em1 = emf.createEntityManager();  
    Dict d1 = em1.find(Dict.class, 1); //find id为1的对象  
    logger.info(d1.getName());  
    em1.close();  
      
    EntityManager em2 = emf.createEntityManager();  
    Dict d2 = em2.find(Dict.class, 1); //find id为1的对象  
    logger.info(d2.getName());  
    em2.close();  
}  


//运行结果
Hibernate: **************  
a  
a  
===================L2======================  
com.restjplat.quickweb.model.Dict : 1  
```

可见二级缓存生效了，只输出了一条sql语句，同时监控中也出现了数据。<br>

另外也可以看看如果是配置成ALL，并且把@cacheable删掉,输出如下： <br>

```java
Hibernate: ************  
a  
a  
===================L2======================  
com.restjplat.quickweb.model.Children : 0  
com.restjplat.quickweb.model.Dict : 1  
org.hibernate.cache.spi.UpdateTimestampsCache : 0  
org.hibernate.cache.internal.StandardQueryCache : 0  
com.restjplat.quickweb.model.Parent : 0  
=================query cache=================  
``` 

并且可以看见，所有的实体类都加入二级缓存中去了。 <br><br>



### 查询缓存测试

一，二级缓存都是根据对象id来查找，如果需要加载一个List的时候，就需要用到查询缓存。<br>
在Spring-data-jpa实现中，也可以使用查询缓存。 <br>

文件配置：<br> 
在 jpaProperties 下添加，这里必须明确标出增加查询缓存。<br>

```xml
<prop key="hibernate.cache.use_query_cache">true</prop>  
```

然后需要在方法内打上@QueryHint来实现查询缓存，我们写几个方法来测试如下： <br>

```java
public interface DictDao extends JpaRepository<Dict, Integer>,JpaSpecificationExecutor<Dict>{  
  
    // spring-data-jpa默认继承实现的一些方法，实现类为  
    // SimpleJpaRepository。  
    // 该类中的方法不能通过@QueryHint来实现查询缓存。  
    @QueryHints({ @QueryHint(name = "org.hibernate.cacheable", value ="true") })  
    List<Dict> findAll();  
      
    @Query("from Dict")  
    @QueryHints({ @QueryHint(name = "org.hibernate.cacheable", value ="true") })  
    List<Dict> findAllCached();  
      
    @Query("select t from Dict t where t.name = ?1")  
    @QueryHints({ @QueryHint(name = "org.hibernate.cacheable", value ="true") })  
    Dict findDictByName(String name);  
}  
```

测试方法： <br>

```java
private void QueryCacheTest(){  
    //无效的spring-data-jpa实现的接口方法  
    //输出两条sql语句  
    dao.findAll();  
    dao.findAll();  
    System.out.println("================test 1 finish======================");  
    //自己实现的dao方法可以被查询缓存  
    //输出一条sql语句  
    dao.findAllCached();  
    dao.findAllCached();  
    System.out.println("================test 2 finish======================");  
    //自己实现的dao方法可以被查询缓存  
    //输出一条sql语句  
    dao.findDictByName("a");  
    dao.findDictByName("a");  
    System.out.println("================test 3 finish======================");  
}  


//运行结果
Hibernate: **************  
Hibernate: **************  
================test 1 finish======================  
Hibernate: ***********  
================test 2 finish======================  
Hibernate: ***********  
================test 3 finish======================  
===================L2======================  
com.restjplat.quickweb.model.Dict : 5  
org.hibernate.cache.spi.UpdateTimestampsCache : 0  
org.hibernate.cache.internal.StandardQueryCache : 2  
=================query cache=================  
select t from Dict t where t.name = ?1  
select generatedAlias0 from Dict as generatedAlias0  
from Dict  
```

很明显，查询缓存生效。但是为什么第一种方法查询缓存无法生效，原因不明，只能后面看看源代码了。 
<br><br>


### 集合缓存测试

根据hibernate文档的写法，这个应该是算在2级缓存里面。 <br>

测试类： <br>

```java
@Entity  
@Table(name ="parent")  
@Cacheable  
public class Parent extends IdEntity {  
      
    private static final long serialVersionUID = 1L;  
    private String name;  
    private List<Children> clist;  
      
    public String getName() {  
        return name;  
    }  
    public void setName(String name) {  
        this.name = name;  
    }  
      
    @OneToMany(fetch = FetchType.EAGER,mappedBy = "parent")  
        @Cache(usage = CacheConcurrencyStrategy.NONSTRICT_READ_WRITE)  
    public List<Children> getClist() {  
        return clist;  
    }  
    public void setClist(List<Children> clist) {  
        this.clist = clist;  
    }  
}  
  
@Entity  
@Table(name ="children")  
@Cacheable  
public class Children extends IdEntity{  
  
    private static final long serialVersionUID = 1L;  
    private String name;  
    private Parent parent;  
      
    @ManyToOne(fetch = FetchType.LAZY)  
    @JoinColumn(name = "parent_id")  
    public Parent getParent() {  
        return parent;  
    }  
  
    public void setParent(Parent parent) {  
        this.parent = parent;  
    }  
  
    public String getName() {  
        return name;  
    }  
  
    public void setName(String name) {  
        this.name = name;  
    }     
}  
```

测试方法： 

```java
private void cellectionCacheTest(){  
    EntityManager em1 = emf.createEntityManager();  
    Parent p1 = em1.find(Parent.class, 1);  
    List<Children> c1 = p1.getClist();  
    em1.close();  
    System.out.println(p1.getName()+" ");  
    for (Children children : c1) {  
        System.out.print(children.getName()+",");  
    }  
    System.out.println();  
    EntityManager em2 = emf.createEntityManager();  
    Parent p2 = em2.find(Parent.class, 1);  
    List<Children> c2 = p2.getClist();  
    em2.close();  
    System.out.println(p2.getName()+" ");  
    for (Children children : c2) {  
        System.out.print(children.getName()+",");  
    }  
    System.out.println();  
} 


//运行结果
Hibernate: ********************  
Michael   
kate,Jam,Jason,Brain,  
Michael   
kate,Jam,Jason,Brain,  
===================L2======================  
com.restjplat.quickweb.model.Children : 4  
com.restjplat.quickweb.model.Dict : 0  
org.hibernate.cache.spi.UpdateTimestampsCache : 0  
com.restjplat.quickweb.model.Parent.clist : 1  
org.hibernate.cache.internal.StandardQueryCache : 0  
com.restjplat.quickweb.model.Parent : 1  
=================query cache=================  
```
在统计数据里可见二级缓存的对象数量。 <br>

本文我们不讨论关于缓存的更新策略，脏数据等等的东西，只是讲解配置方式。 <br><br>



### 源代码篇 

理清楚各种配置以后，我们来看一下hibernate和spring-data-jpa的一些缓存实现源代码。 <br>

上面有个遗留问题，为什么spring-data-jpa默认实现的findAll() 方法无法保存到查询缓存？只能啃源代码了。 <br>

打断点跟踪吧 <br>

入口方法是spring-data-jpa里的 SimpleJpaRepository类 <br>

```java
public List<T> findAll() {  
    return getQuery(null, (Sort) null).getResultList();  
}  
  
//然后到 QueryImpl<X>类的  
private List<X> list() {  
    if (getEntityGraphQueryHint() != null) {  
        SessionImplementor sessionImpl = (SessionImplementor) getEntityManager().getSession();  
        HQLQueryPlan entityGraphQueryPlan = new HQLQueryPlan( getHibernateQuery().getQueryString(), false, sessionImpl.getEnabledFilters(), sessionImpl.getFactory(), getEntityGraphQueryHint() );  
        // Safe to assume QueryImpl at this point.  
        unwrap( org.hibernate.internal.QueryImpl.class ).setQueryPlan( entityGraphQueryPlan );  
    }  
    return query.list();  
}  
  
//进入query.list();  
//query类的代码解析google一下很多，于是直接到最后：  
//进入QueryLoader的list方法。  
protected List list(  
        final SessionImplementor session,  
        final QueryParameters queryParameters,  
        final Set<Serializable> querySpaces,  
        final Type[] resultTypes) throws HibernateException {  

    final boolean cacheable = factory.getSettings().isQueryCacheEnabled() &&  
        queryParameters.isCacheable();  

    if ( cacheable ) {  
        return listUsingQueryCache( session, queryParameters, querySpaces, resultTypes );  
    }  
    else {  
        return listIgnoreQueryCache( session, queryParameters );  
    }  
}  
```
果然有个cacheable，值为false，说明的确是没有从缓存里取数据。用自定义的jpa查询方法测试后发现，这个值为true。于是接着看cacheable的取值过程： 

```java
final boolean cacheable = factory.getSettings().isQueryCacheEnabled() &&  
            queryParameters.isCacheable();  
```

factory.getSettings().isQueryCacheEnabled() 这个一定是true，因为是在配置文件中打开的。那只能是queryParameters.isCacheable() 这个的问题了。 <br>


```java
//在query.list()的方法内部：  
public List list() throws HibernateException {  
    verifyParameters();  
    Map namedParams = getNamedParams();  
    before();  
    try {  
        return getSession().list(  
                expandParameterLists(namedParams),  
                getQueryParameters(namedParams)  
            );  
    }  
    finally {  
        after();  
    }  
}  
  
//getQueryParameters(namedParams)这个方法实际获取的是query对象的cacheable属性的值，也就是说，query对象新建的时候cacheable的值决定了这个query方法能不能被查询缓存。  
``` 

接下来query的建立过程： 

```java
//在 SimpleJpaRepository 类中 return applyLockMode(em.createQuery(query));  
//直接由emcreate，再跟踪到 AbstractEntityManagerImpl中  
@Override  
public <T> QueryImpl<T> createQuery(  
        String jpaqlString,  
        Class<T> resultClass,  
        Selection selection,  
        QueryOptions queryOptions) {  
    try {  
        org.hibernate.Query hqlQuery = internalGetSession().createQuery( jpaqlString );  

        ....  
        return new QueryImpl<T>( hqlQuery, this, queryOptions.getNamedParameterExplicitTypes() );  
    }  
    catch ( RuntimeException e ) {  
        throw convert( e );  
    }  
}  
//即通过session.createQuery(jpaqlString ) 创建初始化对象。  
  
//在query类定义中  
public abstract class AbstractQueryImpl implements Query {  
    private boolean cacheable;  
}  
//cacheable不是对象类型，而是基本类型，所以不赋值的情况下默认为“false”。  
```
也就是说spring-data-jpa接口提供的简单快速的各种接口实现全是不能使用查询缓存的，完全不知道为什么这么设计。 <br>
接下来看看我们自己实现的查询方法实现： <br>
直接找到query方法的setCacheable（）方法打断点，因为肯定改变这个值才能有查询缓存。 <br>

```java
于是跟踪到 SimpleJpaQuery类中  
protected Query createQuery(Object[] values) {  
    return applyLockMode(applyHints(doCreateQuery(values), method), method);  
}  
```
在返回query的过程中通过applyHints（）方法读取了方法上的QueryHint注解从而设置了查询缓存。 <br>

















