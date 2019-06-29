# Mybatis中的主键生成策略(上篇)

[TOC]

## 前言

在实际项目开发中，经常有这样的需求，即在插入一条数据后需要获取该条记录的主键这就涉及到主键的生成策略问题了；

1. 在一个单系统中常见的方法M是设置表中主键为自动递增，每次插入后，mybatis会将自动生成的主键赋值给指定的实体类字段
2. 在分布式系统中，则需要生成全局唯一主键ID

方案1又根据数据库是否支持自动生成主键分为2中情况：

- 若数据库支持自动生成主键的字段（比如 MySQL和 SQL Server），则可以设置useGeneratedKeys=”true”，然后再把keyProperty 设置到目标属性上
- 对于不支持自增型主键的数据库（例如Oracle），则要先通过序列来模拟自增，每次插入数据前先从序列中拿到自增ID

方案1的局限性很大、不利于项目后期扩展，实际开发中不推荐使用；

方案2无论是单例项目还是分布式项目都适用；

本文基于mysql数据库分上下两篇，分别对方案1 和方案2进行介绍

##　1.修改代码

在上篇文章代码的基础上，我们对项目做如下修改

DepartmentMapper.java中新增insertAutoId接口

```java
int insertAutoId(Department record);
```

DepartmentMapper.xml中接口对应sql如下

```xml
  <insert id="insertAutoId" useGeneratedKeys="true" keyProperty="id" parameterType="com.wg.demo.po.Department">
    insert into department (dept_name, descr,create_time)
    values ( #{deptName,jdbcType=VARCHAR}, #{descr,jdbcType=VARCHAR},
      #{createTime,jdbcType=TIMESTAMP})
  </insert>
```

> useGeneratedKeys设置为true后，mybatis会使用JDBC的getGeneratedkeys方法获取由数据库内部自动生成的主键，并将该值赋值给由keyProperty指定的字段；

新建DepartmentService.java

```java
@Service
public class DepartmentService {
    @Autowired
    private DepartmentMapper departmentMapper;

    public int insertDept(Department department)
    {
        return departmentMapper.insertAutoId(department);
    }
}
```

新建DepartmentController.java

```java
@Api(description = "department")
@RestController
@RequestMapping("dept")
public class DepartmentController {
    @Autowired
    public DepartmentService departmentService;

    @ApiOperation(value = "新增部门")
    @PostMapping("new")
    public ResultMsg newDepartment(@RequestBody Department department)
    {
        departmentService.insertDept(department);
        return ResultMsg.getMsg(department);
    }
}
```

## 2.设置表主键为自动递增

最后一定要将数据库department表的主键设置为自动递增,否则会报java.sql.SQLException: Field 'id' doesn't have a default value异常

设置方法如下：

![](http://i67.tinypic.com/fogo4g.png)

##　3.测试

![](http://i63.tinypic.com/2q99lyr.png)

返回结果如下，自增主键为27

![](http://i67.tinypic.com/2zqzfqh.png)



## 

