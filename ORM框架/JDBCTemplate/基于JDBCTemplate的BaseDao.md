```java
@Slf4j
public class BaseDao<T, P> {
    private JdbcTemplate jdbcTemplate;
    private Class<T> clazz;

    @SuppressWarnings(value = "unchecked")
    public BaseDao(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
        clazz = (Class<T>) ((ParameterizedType) this.getClass().getGenericSuperclass()).getActualTypeArguments()[0];
    }

    /**
     * 通用插入，自增列需要添加 {@link Pk} 注解
     *
     * @param t          对象
     * @param ignoreNull 是否忽略 null 值
     * @return 操作的行数
     */
    protected Integer insert(T t, Boolean ignoreNull) {
        //获取类名
        String table = getTableName(t);
        //获取字段名
        List<Field> filterField = getField(t, ignoreNull);
        //列信息集合
        List<String> columnList = getColumns(filterField);
        //以,为分隔符将list集合的多个对象转换为字符串。
        String columns = StrUtil.join(Const.SEPARATOR_COMMA, columnList);

        // 构造占位符，重复某个字符串并通过分界符连接 重复?，次数,分隔符为最后一个参数
        String params = StrUtil.repeatAndJoin("?", columnList.size(), Const.SEPARATOR_COMMA);

        // 构造值
        Object[] values = filterField.stream().map(field -> ReflectUtil.getFieldValue(t, field)).toArray();

        //拼接sql
        String sql = StrUtil.format("INSERT INTO {table} ({columns}) VALUES ({params})", Dict.create().set("table", table).set("columns", columns).set("params", params));
        log.debug("【执行SQL】SQL：{}", sql);
        log.debug("【执行SQL】参数：{}", JSONUtil.toJsonStr(values));
        return jdbcTemplate.update(sql, values);
    }

    /**
     * 通用根据主键删除
     *
     * @param pk 主键
     * @return 影响行数
     */
    protected Integer deleteById(P pk) {
        String tableName = getTableName();
        String sql = StrUtil.format("DELETE FROM {table} where id = ?", Dict.create().set("table", tableName));
        log.debug("【执行SQL】SQL：{}", sql);
        log.debug("【执行SQL】参数：{}", JSONUtil.toJsonStr(pk));
        return jdbcTemplate.update(sql, pk);
    }

    /**
     * 通用根据主键更新，自增列需要添加 {@link Pk} 注解
     *
     * @param t          对象
     * @param pk         主键
     * @param ignoreNull 是否忽略 null 值
     * @return 操作的行数
     */
    protected Integer updateById(T t, P pk, Boolean ignoreNull) {
        String tableName = getTableName(t);

        List<Field> filterField = getField(t, ignoreNull);

        List<String> columnList = getColumns(filterField);

        //追加字符串 update field = ? ,field2 = ?,
        List<String> columns = columnList.stream().map(s -> StrUtil.appendIfMissing(s, " = ?")).collect(Collectors.toList());
        String params = StrUtil.join(Const.SEPARATOR_COMMA, columns);

        // 构造值
        List<Object> valueList = filterField.stream().map(field -> ReflectUtil.getFieldValue(t, field)).collect(Collectors.toList());
        //主键值
        valueList.add(pk);

        Object[] values = ArrayUtil.toArray(valueList, Object.class);

        String sql = StrUtil.format("UPDATE {table} SET {params} where id = ?", Dict.create().set("table", tableName).set("params", params));
        log.debug("【执行SQL】SQL：{}", sql);
        log.debug("【执行SQL】参数：{}", JSONUtil.toJsonStr(values));
        return jdbcTemplate.update(sql, values);
    }

    /**
     * 通用根据主键查询单条记录
     *
     * @param pk 主键
     * @return 单条记录
     */
    public T findOneById(P pk) {
        String tableName = getTableName();
        String sql = StrUtil.format("SELECT * FROM {table} where id = ?", Dict.create().set("table", tableName));
        RowMapper<T> rowMapper = new BeanPropertyRowMapper<>(clazz);
        log.debug("【执行SQL】SQL：{}", sql);
        log.debug("【执行SQL】参数：{}", JSONUtil.toJsonStr(pk));
        return jdbcTemplate.queryForObject(sql, new Object[]{pk}, rowMapper);
    }

    /**
     * 根据对象查询
     *
     * @param t 查询条件
     * @return 对象列表
     */
    public List<T> findByExample(T t) {
        String tableName = getTableName(t);
        List<Field> filterField = getField(t, true);
        List<String> columnList = getColumns(filterField);

        List<String> columns = columnList.stream().map(s -> " and " + s + " = ? ").collect(Collectors.toList());

        String where = StrUtil.join(" ", columns);
        // 构造值
        Object[] values = filterField.stream().map(field -> ReflectUtil.getFieldValue(t, field)).toArray();

        String sql = StrUtil.format("SELECT * FROM {table} where 1=1 {where}", Dict.create().set("table", tableName).set("where", StrUtil.isBlank(where) ? "" : where));
        log.debug("【执行SQL】SQL：{}", sql);
        log.debug("【执行SQL】参数：{}", JSONUtil.toJsonStr(values));
        List<Map<String, Object>> maps = jdbcTemplate.queryForList(sql, values);
        List<T> ret = CollUtil.newArrayList();
        maps.forEach(map -> ret.add(BeanUtil.fillBeanWithMap(map, ReflectUtil.newInstance(clazz), true, false)));
        return ret;
    }

    /**
     * 获取表名
     *
     * @param t 对象
     * @return 表名
     */
    private String getTableName(T t) {
        //通过运行时对象获取类的注解
        Table tableAnnotation = t.getClass().getAnnotation(Table.class);
        //不为空则返回类的表名，为空则直接返回类名忽略大小写。
        if (ObjectUtil.isNotNull(tableAnnotation)) {
            return StrUtil.format("`{}`", tableAnnotation.name());
        } else {
            return StrUtil.format("`{}`", t.getClass().getName().toLowerCase());
        }
    }

    /**
     * 获取表名
     *
     * @return 表名
     */
    private String getTableName() {
        //无构造方法直接获取
        Table tableAnnotation = clazz.getAnnotation(Table.class);
        if (ObjectUtil.isNotNull(tableAnnotation)) {
            return StrUtil.format("`{}`", tableAnnotation.name());
        } else {
            return StrUtil.format("`{}`", clazz.getName().toLowerCase());
        }
    }

    /**
     * 获取列
     *
     * @param fieldList 字段列表
     * @return 列信息列表
     */
    private List<String> getColumns(List<Field> fieldList) {
        // 构造列
        List<String> columnList = CollUtil.newArrayList();
        //遍历获取列注解
        for (Field field : fieldList) {
            Column columnAnnotation = field.getAnnotation(Column.class);
            String columnName;
            //非空，则获取列名；否则获取字段名
            if (ObjectUtil.isNotNull(columnAnnotation)) {
                columnName = columnAnnotation.name();
            } else {
                columnName = field.getName();
            }
            //添加到列信息集合
            columnList.add(StrUtil.format("`{}`", columnName));
        }
        return columnList;
    }

    /**
     * 获取字段列表 {@code 过滤数据库中不存在的字段，以及自增列}
     *
     * @param t          对象
     * @param ignoreNull 是否忽略空值
     * @return 字段列表
     */
    private List<Field> getField(T t, Boolean ignoreNull) {
        // 获取所有字段，包含父类中的字段
        Field[] fields = ReflectUtil.getFields(t.getClass());
        //上面为第三方HuTools工具类的获取所有字段的方式，下方为常规获取
        // Field[] declaredFields = t.getClass().getDeclaredFields();

        // 过滤数据库中不存在的字段，以及自增列
        List<Field> filterField;
        //此处为JDK8中的集合流操作，同时使用了第三方工具类
        Stream<Field> fieldStream = CollUtil.toList(fields).stream().filter(field ->
            ObjectUtil.isNull(field.getAnnotation(Ignore.class)) || ObjectUtil.isNull(field.getAnnotation(Pk.class)));
        //非工具类使用
        /*
            Stream<Field> fieldStream1 = Arrays.stream(fields).filter(field ->
            Objects.isNull(field.getAnnotation(Ignore.class)) || Objects.isNull(field.getAnnotation(Pk.class)));
        * */
        // 是否过滤字段值为null的字段
        if (ignoreNull) {
            //不为空值的全部获取
            filterField = fieldStream.filter(field -> ObjectUtil.isNotNull(ReflectUtil.getFieldValue(t, field))).collect(Collectors.toList());
        } else {
            filterField = fieldStream.collect(Collectors.toList());
        }
        return filterField;
    }

}
```