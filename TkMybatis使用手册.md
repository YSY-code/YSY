# [TkMybatis使用手册](https://confluence.hisense.com/pages/viewpage.action?pageId=28481823#tkmybatis使用手册)

## [应用场景](https://confluence.hisense.com/pages/viewpage.action?pageId=28481823#应用场景)

TkMybatis是基于Mybatis框架开发的一个工具，通过调用它提供的方法实现对单表的数据操作，不需要写任何sql语句，这极大地提高了项目开发效率。

## [框架配置](https://confluence.hisense.com/pages/viewpage.action?pageId=28481823#框架配置)

配置的话非常简单，SpringBoot可直接在pom.xml文件中引入：

```xml
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-spring-boot-starter</artifactId>
    <version>2.1.5</version>
</dependency>

<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper</artifactId>
    <version>4.1.5</version>
</dependency>
```

## [实体类配置](https://confluence.hisense.com/pages/viewpage.action?pageId=28481823#实体类配置)

创建数据库表对应实体类对象

```java
@Table(name = "base_user")
public class User implements Serializable {
    private static final long serialVersionUID = -2786301994259082323L;
    
    @Id
    @KeySql(genId = SnowflakeGenId.class)
    private Long id;

    private String username;
   
    @Column(name = "mobile_phone")
    private String mobilePhone;

    @Transient
    private String departName;
}
```

实体类中，使用了以下注解：

@Table
描述数据库表信息，主要属性有name(表名)、schema、catalog、uniqueConstraints等。

@Id
指定表主键字段，无属性值。

@KeySql(genId = SnowflakeGenId.class)
框架新版本统一使用该注解生成主键

@Column
描述数据库字段信息，主要属性有name(字段名)、columnDefinition、insertable、length、nullable(是否可为空)、precision、scale、table、unique、updatable等。

@Transient
标识该属性不进行数据库持久化操作，无属性。

## [Mapper数据库操作接口](https://confluence.hisense.com/pages/viewpage.action?pageId=28481823#mapper数据库操作接口)

此框架为我们实现这些功能所有的改动都在Mapper层面，所有的Mapper都继承了 tk.mybatis.mapper.common.Mapper 及 tk.mybatis.mapper.common.ids.SelectByIdsMapper

```java
public interface WorkerMapper extends Mapper<Worker>, SelectByIdsMapper<Worker> {}
```

## [Tkmybatis数据库操作方法](https://confluence.hisense.com/pages/viewpage.action?pageId=28481823#tkmybatis数据库操作方法)

### [增](https://confluence.hisense.com/pages/viewpage.action?pageId=28481823#增)

insert

```java
   /**
     * 保存一个实体，null的属性也会保存，不会使用数据库默认值
     *
     * @param record
     * @return
     */
    @InsertProvider(type = BaseInsertProvider.class, method = "dynamicSQL")
    int insert(T record);
```

insertSelective（插入数据常用此方法）

```java
    /**
     * 保存一个实体，null的属性不会保存，会使用数据库默认值
     *
     * @param record
     * @return
     */
    @InsertProvider(type = BaseInsertProvider.class, method = "dynamicSQL")
    int insertSelective(T record);
```

### [删](https://confluence.hisense.com/pages/viewpage.action?pageId=28481823#删)

delete

```java
   /**
     * 根据实体属性作为条件进行删除，查询条件使用等号
     *
     * @param record
     * @return
     */
    @DeleteProvider(type = BaseDeleteProvider.class, method = "dynamicSQL")
    int delete(T record);
```

deleteByPrimaryKey（物理删除数据常用此方法）

```java
    /**
     * 根据主键字段进行删除，方法参数必须包含完整的主键属性
     *
     * @param key
     * @return
     */
    @DeleteProvider(type = BaseDeleteProvider.class, method = "dynamicSQL")
    int deleteByPrimaryKey(Object key);
```

### [改](https://confluence.hisense.com/pages/viewpage.action?pageId=28481823#改)

updateByPrimaryKey

```java
    /**
     * 根据主键更新实体全部字段，null值会被更新
     *
     * @param record
     * @return
     */
    @UpdateProvider(type = BaseUpdateProvider.class, method = "dynamicSQL")
    @Options(useCache = false, useGeneratedKeys = false)
    int updateByPrimaryKey(T record);
```

updateByPrimaryKeySelective（唯一主键时，更新数据常用此方法）

```java
    /**
     * 根据主键更新属性不为null的值
     *
     * @param record
     * @return
     */
    @UpdateProvider(type = BaseUpdateProvider.class, method = "dynamicSQL")
    @Options(useCache = false, useGeneratedKeys = false)
    int updateByPrimaryKeySelective(T record);
```

updateByExample（复合主键时，更新数据用此方法，同时此方法可配置多属性作为查询条件）

```java
    /**
     * 根据Example条件更新实体`record`包含的全部属性，null值会被更新
     *
     * @param record
     * @param example
     * @return
     */
    @UpdateProvider(type = ExampleProvider.class, method = "dynamicSQL")
    @Options(useCache = false, useGeneratedKeys = false)
    int updateByExample(@Param("record") T record, @Param("example") Object example);
    /**
     * Mybatis中的Criteria条件查询（以customerId和projectCode为条件更新实体数据）
    */

    /**
     * mybatis的逆向工程中会生成实例及实例对应的example,example用于添加条件，相当于where后面的部分
    */
    Example example = new Example(MdProjectWeekMtc.class);
    /**
     * Criteria包含一个Cretiron的集合,每一个Criteria对象内包含的Cretiron之间是由AND连接的,是逻辑与的关系。
     * Criterion是最基本,最底层的Where条件，用于字段级的筛选
    */
    Example.Criteria criteria = example.createCriteria();
    /**
     * andEqualTo相当于在sql中拼接一个“AND customerId ='传入值'”
    */
    criteria.andEqualTo("customerId", mdProjectWeekMtc.getCustomerId());
    criteria.andEqualTo("projectCode", mdProjectWeekMtc.getProjectCode());
    mapper.updateByExample(mdProjectWeekMtc, example);
```

Criteria的部分方法及说明

| 方法                                       | 说明                                    |
| :----------------------------------------- | :-------------------------------------- |
| criteria.andXxxIsNull                      | 添加字段xxx为null的条件                 |
| criteria.andXxxIsNotNull                   | 添加字段xxx不为null的条件               |
| criteria.andXxxEqualTo(value)              | 添加xxx字段等于value条件                |
| criteria.andXxxNotEqualTo(value)           | 添加xxx字段不等于value条件              |
| criteria.andXxxGreaterThan(value)          | 添加xxx字段大于value条件                |
| criteria.andXxxGreaterThanOrEqualTo(value) | 添加xxx字段大于等于value条件            |
| criteria.andXxxLessThan(value)             | 添加xxx字段小于value条件                |
| criteria.andXxxLessThanOrEqualTo(value)    | 添加xxx字段小于等于value条件            |
| criteria.andXxxIn(List<？>)                | 添加xxx字段值在List<？>条件             |
| criteria.andXxxNotIn(List<？>)             | 添加xxx字段值不在List<？>条件           |
| criteria.andXxxLike(“%”+value+”%”)         | 添加xxx字段值为value的模糊查询条件      |
| criteria.andXxxNotLike(“%”+value+”%”)      | 添加xxx字段值不为value的模糊查询条件    |
| criteria.andXxxBetween(value1,value2)      | 添加xxx字段值在value1和value2之间条件   |
| criteria.andXxxNotBetween(value1,value2)   | 添加xxx字段值不在value1和value2之间条件 |

### [查](https://confluence.hisense.com/pages/viewpage.action?pageId=28481823#查)

selectOne

```java
    /**
     * 根据实体中的属性进行查询，只能有一个返回值，有多个结果是抛出异常，查询条件使用等号
     *
     * @param record
     * @return
     */
    @SelectProvider(type = BaseSelectProvider.class, method = "dynamicSQL")
    T selectOne(T record);
```

select

```java
    /**
     * 根据实体中的属性值进行查询，查询条件使用等号
     *
     * @param record
     * @return
     */
    @SelectProvider(type = BaseSelectProvider.class, method = "dynamicSQL")
    List<T> select(T record);
```

selectAll

```java
    /**
     * 查询全部结果
     *
     * @return
     */
    @SelectProvider(type = BaseSelectProvider.class, method = "dynamicSQL")
    List<T> selectAll();
```

selectByPrimaryKey（根据单据ID查询详细信息常用此方法）

```java
    /**
     * 根据主键字段进行查询，方法参数必须包含完整的主键属性，查询条件使用等号
     *
     * @param key
     * @return
     */
    @SelectProvider(type = BaseSelectProvider.class, method = "dynamicSQL")
    T selectByPrimaryKey(Object key);
```

selectByExample（多条件分页查询常用此方法）

```java
    /**
     * 根据Example条件进行查询
     *
     * @param example
     * @return
     */
    @SelectProvider(type = ExampleProvider.class, method = "dynamicSQL")
    List<T> selectByExample(Object example);
```

selectByIds

```java
    /**
     * 根据主键字符串进行查询，类中只有存在一个带有@Id注解的字段
     *
     * @param ids 如 "1,2,3,4"
     * @return
     */
    @SelectProvider(type = IdsProvider.class, method = "dynamicSQL")
    List<T> selectByIds(String ids);
```

## [注意事项](https://confluence.hisense.com/pages/viewpage.action?pageId=28481823#注意事项)

- 定义参数时使用 @param(" ") 显式定义;
- 能用#{}的地方尽量使用#{}，有sql注入风险的禁止使用${};
- *表名作为参数时，必须使用${};
- *order by 时，必须使用${};
- 使用${}时要注意何时加或不加单引号；
- 批量操作时要根据数据条数来确定单次提交的数量，当数据条数较大时，建议每200条数据提交一次，防止撑爆mysql IO;
- 开启tk mapper的安全删除和更新机制（**重要**），防止删除或者更新全表，启用表安全配置对tk mapper的版本有要求，具体为tk.mybatis:mapper-spring-boot-starter版本为2.0.0及以上，tk.mybatis:mapper版本为4.0.0及以上安全删除配置才生效
  在application.yml文件中加入如下配置：

```yaml
  mapper:
      mappers: com.savor.security.common.mapper.CommonMapper
        #启用安全删除和安全更新机制
        safe-delete: true
        safe-update: true
```

- 在使用delete(Object)和update(Object)方法时（**重要**），必须对查询参数做非空校验，防止查询参数错误误删除数据

```java
  // 根据Object的A、B属性删除
  if (object != null && StringUtils.isNotEmpty(object.getA()) && StringUtils.isNotEmpty(object.getB())) {
      mapper.delete(object);
  }
```