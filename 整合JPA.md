## 整合JPA
JPA:ORM (Object Relational Mapping)
1. 创建一个实体类和数据表映射
```java

// 使用jpa注解配置映射关系
@Entity     // 告诉jpa这是一个实体类(和数据表映射)
@Table(name = "tbl_user") //指定哪个数据表 如果省略表名默认是user
@Data
public class User {

    @Id //这是一个主键
    @GeneratedValue(strategy = GenerationType.IDENTITY)  //这是一个自增主键
    private Integer id;

    @Column(name = "last_name", length = 50)
    private String lastName;

    @Column  // 省略默认列名是属性名
    private String email;
}
```

2. 编写Dao接口来操作实体类对应的数据表(Repository)

```java
// 继承JpaRepository来完成对数据库的操作
public interface UserRepository extends JpaRepository<User,Integer> {
}

```

3. 基本配置

```yml
  jpa:
    hibernate:
#      更新或创建数据表结构
      ddl-auto: update
#	  控制台显示sql
    show-sql: true
```

**最好用阿里巴巴的Druid连接池**  
 

