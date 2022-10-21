# RowMapper实现类

## RowMapper匿名类使用

这里直接一个个设置返回[实体类](https://so.csdn.net/so/search?q=实体类&spm=1001.2101.3001.7020)的属性

```java
 public Student getStudentByName(String name) {
        String sql = "select id,name from student where name = ?";
        Student student = this.jdbcTemplate.queryForObject(sql, new Object[]{name}, new RowMapper<Student>() {
            @Override
            public Student mapRow(ResultSet rs, int i) throws SQLException {
                Student s = new Student();
                s.setId(rs.getString("id"));
                s.setName(rs.getString("name"));
                return s;
            }
        });
        return student;
    }
```

## SingleColumnRowMapper使用

当查询单个字段时，可以直接使用SingleColumnRowMapper

```java
   public String getStudentNameById(String id) {
       String sql = "select name from test_student where id = ?";
       return this.jdbcTemplate.queryForObject(sql, new Object[]{id},
               new SingleColumnRowMapper<>(String.class));
   }
```

## BeanPropertyRowMapper使用

当查询字段多，或者你想直接偷懒，就用他，他源代码的实现方法已经实现了忽略大小写、驼峰命名转换的查询，不用再自己一个个set属性，方便简单，什么狗屁效率，快速开发就完事了。

```java
public Student getStudentByName(String name) {
        String sql = "select id,name from student where name = ?";
        Student student = this.jdbcTemplate.queryForObject(sql, new Object[]{name}, new BeanPropertyRowMapper<>(Student.class));
        return student;
	}
```

