# 1. 自定义查询辅助注解

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Query {

    /**
     * 基本对象的属性名 
     */
    String propName() default "";
    /**
     * 
     * 查询方式
     */
    Type type() default Type.EQUAL;

    /**
     * 多字段模糊搜索，仅支持String类型字段，多个用逗号隔开, 如@Query(blurry = "email,username")
     */
    String blurry() default "";

    enum Type {
        // 相等
        EQUAL
        // 大于等于
        , GREATER_THAN
        // 小于等于
        , LESS_THAN
        // 中模糊查询
        , INNER_LIKE
        // 左模糊查询
        , LEFT_LIKE
        //  右模糊查询
        , RIGHT_LIKE
        //  小于
        , LESS_THAN_NQ
        //  包含
        , IN
        // 不等于
        ,NOT_EQUAL
        // between
        ,BETWEEN
        // 不为空
        ,NOT_NULL
        // 为空
        ,IS_NULL
    }

}
```

# 2. 查询条件解析类

```java
@Slf4j
public class QueryHelper {
	public static <T, Q> QueryWrapper<T> getQueryWrapper(QueryWrapper<T> queryWrapper, Q query) {
		try {
			List<Field> fields = getAllFields(query.getClass(), new ArrayList<>());
			for (Field field : fields) {
				boolean accessible = field.isAccessible();
				// 设置对象的访问权限，保证对private的属性的访问
				field.setAccessible(true);
				Query q = field.getAnnotation(Query.class);
				if (q != null) {
					String propName = q.propName();
					String blurry = toUnderScoreCase(q.blurry());
					String attributeName = isBlank(propName) ? toUnderScoreCase(field.getName()) : propName;
					Object val = field.get(query);
					// 判断是否为null，或者空字符串
					if (ObjectUtil.isNull(val) || "".equals(val)) {
						continue;
					}
					// 模糊多字段
					if (ObjectUtil.isNotEmpty(blurry)) {
						// 获取用于模糊查询字段
						ArrayList<String> blurryList = new ArrayList<>(Arrays.asList(blurry.split(",")));
						// 处理成 AND(name LIKE ? OR create_by LIKE ?) 这种格式
						queryWrapper.and(i -> blurryList.forEach(bs -> i.like(bs, val).or()));
						// 匹配下一个字段
						continue;
					}
					// 普通查询
					switch (q.type()) {
						case EQUAL:
							queryWrapper.and(i -> i.eq(attributeName, val));
							break;
						case GREATER_THAN:
							queryWrapper.and(i -> i.ge(attributeName, val));
							break;
						case LESS_THAN:
							queryWrapper.and(i -> i.le(attributeName, val));
							break;
						case LESS_THAN_NQ:
							queryWrapper.and(i -> i.lt(attributeName, val));
							break;
						case INNER_LIKE:
							queryWrapper.and(i -> i.like(attributeName, val));
							break;
						case LEFT_LIKE:
							queryWrapper.and(i -> i.likeLeft(attributeName, val));
							break;
						case RIGHT_LIKE:
							queryWrapper.and(i -> i.likeRight(attributeName, val));
							break;
						case IN:
							if (CollUtil.isNotEmpty((Collection<?>) val)) {
								queryWrapper.and(i -> i.in(attributeName, val));
							}
							break;
						case NOT_EQUAL:
							queryWrapper.and(i -> i.ne(attributeName, val));
							break;
						case NOT_NULL:
							queryWrapper.and(i -> i.isNotNull(attributeName));
							break;
						case IS_NULL:
							queryWrapper.and(i -> i.isNull(attributeName));
							break;
						case BETWEEN:
							List<Object> between = new ArrayList<>((List<?>) val);
							if (CollUtil.isNotEmpty(between) && between.size() == 2) {
								queryWrapper.and(i -> i.between(attributeName, between.get(0), between.get(1)));
							}
							break;
						default:
							break;
					}
				}
				// 最后设置字段为原来的权限
				field.setAccessible(accessible);
			}
		} catch (Exception e) {
			log.error(e.getMessage(), e);
		}
		return queryWrapper;
	}

	private static boolean isBlank(final CharSequence cs) {
		int strLen;
		if (cs == null || (strLen = cs.length()) == 0) {
			return true;
		}
		for (int i = 0; i < strLen; i++) {
			if (!Character.isWhitespace(cs.charAt(i))) {
				return false;
			}
		}
		return true;
	}

	public static List<Field> getAllFields(Class<?> clazz, List<Field> fields) {
		if (clazz != null) {
			fields.addAll(Arrays.asList(clazz.getDeclaredFields()));
			getAllFields(clazz.getSuperclass(), fields);
		}
		return fields;
	}
    
    /**
     * 驼峰命名法
     * 将 helloWorld 转换为 hello_world
     * @param s
     * @return
     */
    public static String toUnderScoreCase(String s) {
        if (s == null) {
            return null;
        }

        StringBuilder sb = new StringBuilder();
        boolean upperCase = false;
        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);

            boolean nextUpperCase = true;

            if (i < (s.length() - 1)) {
                nextUpperCase = Character.isUpperCase(s.charAt(i + 1));
            }

            if ((i > 0) && Character.isUpperCase(c)) {
                if (!upperCase || !nextUpperCase) {
                    sb.append('_');
                }
                upperCase = true;
            } else {
                upperCase = false;
            }

            sb.append(Character.toLowerCase(c));
        }

        return sb.toString();
    }
}
```

# 3. 使用查询辅助注解

## 3.1 创建UserQuery类，用于查询User信息

```java
@Getter
@Setter
public class UserQuery {

    /**
     * 通过loginName,name模糊查询
     */
    @Query(blurry = "loginName,name")
    private String blurry;

    /**
     * 通过电话equal查询
     */
    @Query
    private String tel;

    /**
     * 通过登录时间equal查询
     */
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    @Query
    private LocalDateTime loginTime;

}
```

## 3.2 通过UserQuery进行查询

```java
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {

    @Autowired
    private UserMapper userMapper;

    @Autowired
    private UserMapstruct userMapstruct;

    @Override
    public Object findOfPage(PageDto pageDto, UserQuery query) {
        // 开启分页
        PageHelper.startPage(pageDto.getPage(), pageDto.getLimit());
        // 执行查询
        return new PageInfo<>(userMapstruct.toDto(userMapper.selectList(QueryHelper.getQueryWrapper(new QueryWrapper<User>(), query))));
    }
}
```

测试结果：

![image-20210908200900620](https://i.loli.net/2021/09/08/C6DB19L2kXGJTmA.png)