### @MappedSuperclass

在Jpa里, 当我们在定义多个实体类时, 可能会遇到这几个实体类都有几个共同的属性, 这时就会出现很多重复代码.
这时我们可以选择编写一个父类,将这些共同属性放到这个父类中, 并且在父类上加上@MappedSuperclass注解.注意:标注为@MappedSuperclass的类将不是一个完整的实体类，他将不会映射到数据库表，但是他的属性都将映射到其子类的数据库字段中。

> 标注为@MappedSuperclass的类不能再标注@Entity或@Table注解，也无需实现序列化接口.

### @EntityListeners(AuditingEntityListener.class)

(1) 该注解用于监听实体类，在save、update之后的状态
(2) 使用了@EntityListeners(AuditingEntityListener.class)之后，记得在Application
启动类上加@EnableJpaAuditing，不然@CreateDate，@LastModifiedBy不生效

### @JoinTable()多对多关系映射

```java
/**
 * 关联部门表
 * 1、关系维护端，负责多对多关系的绑定和解除
 * 2、@JoinTable注解的name属性指定关联表的名字，joinColumns指定外键的名字，关联到关系维护端(User)
 * 3、inverseJoinColumns指定外键的名字，要关联的关系被维护端(Department)
 * 4、其实可以不使用@JoinTable注解，默认生成的关联表名称为主表表名+下划线+从表表名，
 * 即表名为user_department
 * 关联到主表的外键名：主表名+下划线+主表中的主键列名,即user_id,这里使用referencedColumnName指定
 * 关联到从表的外键名：主表中用于关联的属性名+下划线+从表的主键列名,department_id
 * 主表就是关系维护端对应的表，从表就是关系被维护端对应的表
 */
@ManyToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER)
@JoinTable(name = "orm_user_dept", joinColumns = @JoinColumn(name = "user_id", referencedColumnName = "id"), inverseJoinColumns = @JoinColumn(name = "dept_id", referencedColumnName = "id"))
private Collection<Department> departmentList;
```

### @Embedded和@Embeddable注解

- @Embeddable：一般用于需要作为属性出现在另外一个实体类的普通实体类出现。添加Embeddable的实体类不会生成对应的数据库表，而另外一个生成数据库表的实体类中具备的实体类作为属性可以被正常解析。

- @Embedded：在主表中的POJO实体类的属性上面添加该注解，效果与第一个注解相同。
- 两个注解同时使用产生的效果也一致。

### 覆盖`@Embeddable`类中字段的列属性

这里就要使用另外的两个注解`@AttributeOverrides`和`@AttributeOverride`，这两个注解是用来覆盖`@Embeddable`类中字段的属性的。

- `@AttributeOverrides`：里面只包含了`@AttributeOverride`类型数组；
- `@AttributeOverride`：包含要覆盖的`@Embeddable`类中字段名name和新增的`@Column`字段的属性；

使用如下：
Person类如下：

```java
@Entity
public class Person implements Serializable{
    private static final long serialVersionUID = 8849870114127659929L;

    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private Integer age;

    @Embedded
    @AttributeOverrides({@AttributeOverride(name="country", column=@Column(name = "person_country", length = 25, nullable = false)),
                        @AttributeOverride(name="city", column = @Column(name = "person_city", length = 15))})
    private Address address;

    //setter、getter
}
```

Address类如下：

```java
@Embeddable
public class Address implements Serializable{
    private static final long serialVersionUID = 8849870114128959929L;

    @Column(nullable = false)
    private String country;
    @Column(length = 30)
    private String province;
    @Column(unique = true)
    private String city;
    @Column(length = 50)
    private String detail;
    //setter、getter
}
```

### @EmbeddedId

复合主键类，用@Embeddable注解声明

```java
@Embeddable
public class MasCpcusPK implements Serializable {
    private static final long serialVersionUID = 1L;

    private BigDecimal crAcctNbr; // 持卡人代号
    private BigDecimal crOrgNbr; // 机构号

    public MasCpcusPK() {
    }  
    @Basic
    @Column(name="CR_ACCT_NBR")
    public BigDecimal getCrAcctNbr() {
        return this.crAcctNbr;
    }
    public void setCrAcctNbr(BigDecimal crAcctNbr) {
        this.crAcctNbr = crAcctNbr;
    }
    @Basic
    @Column(name="CR_ORG_NBR")
    public BigDecimal getCrOrgNbr() {
        return this.crOrgNbr;
    }
    public void setCrOrgNbr(BigDecimal crOrgNbr) {
        this.crOrgNbr = crOrgNbr;
    }
}
```

PO类，用复合主键代替那两个属性，在get方法上，使用@EmbeddedId注解

```java
@Entity
@Table(name = "mkt_mas_cpcus", schema = "market", catalog = "")
public class MktMasCpcus implements Serializable {

    private static final long serialVersionUID = -8509123963537411929L;

    private MasCpcusPK id; // 复合主键

    private BigDecimal crStatus; // 客户状态

    private String crShortName; // 姓名

    @EmbeddedId
    public MasCpcusPK getId() {
        return id;
    }

    public void setId(MasCpcusPK id) {
        this.id = id;
    }
    @Basic
    @Column(name = "CR_STATUS")
    public BigDecimal getCrStatus() {
        return crStatus;
    }

    public void setCrStatus(BigDecimal crStatus) {
        this.crStatus = crStatus;
    }

    @Basic
    @Column(name = "CR_SHORT_NAME")
    public String getCrShortName() {
        return crShortName;
    }

    public void setCrShortName(String crShortName) {
        this.crShortName = crShortName;
    }
}
```

