## 1. CORS跨域配置

```java
@Configuration
public class CORSConfig {

	@Bean
	public CorsFilter corsFilter() {
		UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
		CorsConfiguration corsConfiguration = new CorsConfiguration();
		corsConfiguration.setMaxAge(1728000L);
		corsConfiguration.setAllowCredentials(true);
		corsConfiguration.addAllowedOrigin("*");
		corsConfiguration.addAllowedHeader("*");
		corsConfiguration.addAllowedMethod("*");
		source.registerCorsConfiguration("/**", corsConfiguration);
		return new CorsFilter(source);
	}

	@Bean
	public WebMvcConfigurer corsConfigurer() {
		return new WebMvcConfigurer() {
			@Override
			public void addCorsMappings(CorsRegistry registry) {
				registry
				.addMapping("/**")
				.allowedOriginPatterns("*")
				.allowedHeaders("*")
				.allowCredentials(true)
				.allowedMethods("GET", "POST", "DELETE", "PUT", "PATCH")
				.maxAge(3600);
			}
		};
	}

}
```
## 2. springboot运行成功后的提示信息
（1）使用前需要先引入hutool工具包。

```xml
<dependency>
	<groupId>cn.hutool</groupId>
	<artifactId>hutool-all</artifactId>
	<version>5.3.4</version>
</dependency>
```
（2）springboot main函数配置。
```java
public static void main(String[] args) {
	ConfigurableApplicationContext applicationContext = SpringApplication.run(ProjectManageApplication.class, args);
	AppInfoUtils.printTips(applicationContext);
}
```
（3）ToolHttpInfo 类

```java
public class ToolHttpInfo {

	private static final String UNKNOWN = "unknown";
	private static final String HOST_1 = "0.0.0.0";
	private static final String HOST_2 = "0:0:0:0:0:0:0:1";
	private static final String HOST_3 = "localhost";
	private static final String HOST_4 = "127.0.0.1";

	/**
	 *获取客户端的 IP
	 * 
	 * @param request
	 * @return
	 */
	public static String getClientIp(HttpServletRequest request) {
		String ip = request.getHeader("X-Real-IP");
		if (ip == null || "".equals(ip.trim()) || UNKNOWN.equalsIgnoreCase(ip)) {
			ip = request.getHeader("X-Forwarded-For");
		}
		if (ip == null || ip.length() == 0 || UNKNOWN.equalsIgnoreCase(ip)) {
			ip = request.getHeader("Proxy-Client-IP");
		}
		if (ip == null || ip.length() == 0 || UNKNOWN.equalsIgnoreCase(ip)) {
			ip = request.getHeader("WL-Proxy-Client-IP");
		}
		if (ip == null || ip.length() == 0 || UNKNOWN.equalsIgnoreCase(ip)) {
			ip = request.getRemoteAddr();
		}
		if (HOST_1.equals(ip) || HOST_2.equals(ip) || HOST_3.equals(ip) || HOST_4.equals(ip)) {
			ip = "127.0.0.1";
		}
		return ip;
	}

	/**
	 * 获取当前机器的IP
	 *
	 * @return /
	 */
	public static String getLocalIp() {
		InetAddress addr;
		try {
			addr = InetAddress.getLocalHost();
		} catch (UnknownHostException e) {
			return "unknown";
		}
		byte[] ipAddr = addr.getAddress();
		StringBuilder ipAddrStr = new StringBuilder();
		for (int i = 0; i < ipAddr.length; i++) {
			if (i > 0) {
				ipAddrStr.append(".");
			}
			ipAddrStr.append(ipAddr[i] & 0xFF);
		}
		return ipAddrStr.toString();
	}

	/**
	 * detect request if Ajax
	 * 
	 * @param request
	 * @return
	 */
	public static boolean ajax(HttpServletRequest request) {
		String accept = request.getHeader("accept");
		return accept != null && accept.contains("application/json") || (request.getHeader("X-Requested-With") != null
				&& request.getHeader("X-Requested-With").contains("XMLHttpRequest"));
	}

	/**
	 * 获得浏览器信息
	 * 
	 * @param request
	 * @return
	 */
	public static String getBrowser(HttpServletRequest request) {
		UserAgent userAgent = UserAgentParser.parse(request.getHeader("User-Agent"));
		Browser browser = userAgent.getBrowser();
		return browser.getName();
	}

}
```

（4）AppInfoUtils 工具类
```java
public class AppInfoUtils {
    public static void printTips(ConfigurableApplicationContext application) {

        Environment env = application.getEnvironment();

        String ip = ToolHttpInfo.getLocalIp();

        String ssl = env.getProperty("server.ssl.key-store");
        String protocol = "http";
        if (ssl != null) {
            protocol = "https";
        }
        String port = env.getProperty("server.port");
        System.out.println("\n----------------------------------------------------------\n\t"
                + "Application  started ! Access URLs:\n\t" + "Local: \t" + protocol + "://localhost:" + port + "/\n\t"
                + "External: " + protocol + "://" + ip + ":" + port + "/\n\t" + "swagger-bootstrap-ui: " + protocol + "://" + ip
                + ":" + port + "/doc.html" + "\n\t" + "swagger-ui: " + protocol + "://" + ip
                + ":" + port + "/swagger-ui.html\n" + "----------------------------------------------------------");
    }

}
```

## 3. PageDto
```java
public class PageDto {

	@ApiModelProperty(value = "currentPage", name = "page", example = "1")
	private Integer page;

	@ApiModelProperty(value = "pagesize", name = "limit", example = "5")
	private Integer limit;

}
```
## 4. 日志文件配置，保存输出日志文件。logback.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <!-- 日志输出路径，最好使用绝对路径 -->
    <property name="LOG_HOME" value="./logs" />

    <!--项目名称，用于自动拼接日志前缀-->
    <property name="PROJECT_NAME" value="pm"></property>

    <!-- 控制台输出 -->
    <appender name="CONSOLE_OUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 按照每天生成日志文件 -->
    <appender name="FILE_OUT_PER_DAY" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <FileNamePattern>${LOG_HOME}/${PROJECT_NAME}_log.%d{yyyy-MM-dd}.log</FileNamePattern>
            <!--日志文件保留天数-->
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
        <!--日志文件最大的大小-->
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <MaxFileSize>10MB</MaxFileSize>
        </triggeringPolicy>
    </appender>

    <!-- Hibernate 定制化配置 -->
    <!-- <logger name="org.hibernate.type.descriptor.sql.BasicBinder"  level="TRACE" />
    <logger name="org.hibernate.type.descriptor.sql.BasicExtractor"  level="DEBUG" />
    <logger name="org.hibernate.SQL" level="DEBUG" />
    <logger name="org.hibernate.engine.QueryParameters" level="DEBUG" />
    <logger name="org.hibernate.engine.query.HQLQueryPlan" level="DEBUG" /> -->
    <!--myibatis 定制化配置-->
    <!-- <logger name="com.apache.ibatis" level="TRACE"/>
    <logger name="java.sql.Connection" level="DEBUG"/>
    <logger name="java.sql.Statement" level="DEBUG"/>
    <logger name="java.sql.PreparedStatement" level="DEBUG"/> -->
    <!--监控sql日志输出 -->
    <logger name="jdbc.sqlonly" level="INFO" additivity="false">
        <appender-ref ref="FILE_OUT_PER_DAY" />
    </logger>
    <logger name="jdbc.resultset" level="ERROR" additivity="false">
        <appender-ref ref="FILE_OUT_PER_DAY" />
    </logger>
    <!--  如想看到表格数据，将OFF改为INFO  -->
    <logger name="jdbc.resultsettable" level="OFF" additivity="false">
        <appender-ref ref="CONSOLE_OUT" />
    </logger>
    <logger name="jdbc.connection" level="OFF" additivity="false">
        <appender-ref ref="CONSOLE_OUT" />
        <appender-ref ref="FILE_OUT_PER_DAY" />
    </logger>
    <logger name="jdbc.sqltiming" level="OFF" additivity="false">
        <appender-ref ref="CONSOLE_OUT" />
        <appender-ref ref="FILE_OUT_PER_DAY" />
    </logger>
    <logger name="jdbc.audit" level="OFF" additivity="false">
        <appender-ref ref="CONSOLE_OUT" />
        <appender-ref ref="FILE_OUT_PER_DAY" />
    </logger>

    <!--日志异步到数据库 -->
    <!--<appender name="DB" class="ch.qos.logback.classic.db.DBAppender">-->
    <!--&lt;!&ndash;日志异步到数据库 &ndash;&gt;-->
    <!--<connectionSource class="ch.qos.logback.core.db.DriverManagerConnectionSource">-->
    <!--&lt;!&ndash;连接池 &ndash;&gt;-->
    <!--<dataSource class="com.mchange.v2.c3p0.ComboPooledDataSource">-->
    <!--<driverClass>com.mysql.jdbc.Driver</driverClass>-->
    <!--<url>jdbc:mysql://127.0.0.1:3306/databaseName</url>-->
    <!--<user>root</user>-->
    <!--<password>root</password>-->
    <!--</dataSource>-->
    <!--</connectionSource>-->
    <!--</appender>-->
    <!-- 按级别统一设置日志 -->
    <root level="INFO">
        <appender-ref ref="CONSOLE_OUT" />
        <appender-ref ref="FILE_OUT_PER_DAY" />
    </root>
</configuration>

```
## 5. 自定义响应实体类
（1）自定义的ResponseCode类

```java
@NoArgsConstructor
@AllArgsConstructor
public enum ResponseCode {

	SUCCESS(200, "操作成功"), FAIL(-1, "操作失败"), SIGN_IN_SUCCESS(200, "登录成功"), SIGN_IN_FAIL(207, "登录失败"),
	NO_SIGN_IN_FAIL(208, "未登录，请重新登录"), SIGN_IN_USERNAME_PASSWORD_FAIL(200, "用户名或密码错误"), USER_ISLOCKED(206, "用户已被锁定"),
	SIGN_IN_USERNAME_PASSWORD_EMPTY(201, "用户名或密码为空"), SIGN_OUT_SUCCESS(202, "登出成功"),
	PERMISSIN_FAIL(203, "没有足够的权限,请重新登录"), TOKEN_EXPIRED(204, "凭证过期，请重新登录"), TOKEN_AUTHENTICATION_FAIL(205, "凭证校验失败"),
	UPLOAD_FAIL(-1, "凭证过期，请重新登录"), UPLOAD_SUCCESS(200, "凭证过期，请重新登录"), NONE_LAST_FLOW_FAIL,;

	public Integer code;
	public String msg;

}
```
（2）ResponseResult 类

```java
@NoArgsConstructor
@AllArgsConstructor
@Builder
@Getter
@Setter
@ApiModel(value = "系统响应")
public class ResponseResult {

	@ApiModelProperty(value = "响应码")
	private Integer code;
	@ApiModelProperty(value = "响应消息")
	private String msg;
	@ApiModelProperty(value = "响应数据")
	private Object data;

	/****
	 * 实例初始化的方式构造返回结果
	 * 
	 * @param responseCode
	 */
	public ResponseResult(ResponseCode responseCode) {
		this.code = responseCode.code;
		this.msg = responseCode.msg;
	}

	public ResponseResult(ResponseCode responseCode, Object data) {
		this.code = responseCode.code;
		this.msg = responseCode.msg;
		this.data = data;
	}

	/****
	 * 操作方法的方式返回
	 * 
	 * @param responseCode
	 */
	public static ResponseResult e(ResponseCode responseCode) {
		return e(responseCode, null);
	}

	public static ResponseResult e(HttpStatus httpstatus) {
		return e(httpstatus, null);
	}

	public static ResponseResult e(ResponseCode responseCode, Object data) {
		return ResponseResult.builder().code(responseCode.code).msg(responseCode.msg).data(data).build();
	}

	public static ResponseResult e(HttpStatus httpstatus, Object data) {
		return ResponseResult.builder().code(httpstatus.value()).msg(httpstatus.getReasonPhrase()).data(data).build();
	}

}
```
## 6. 驼峰命名法之间转换的工具类

```java
public class StringUtils extends org.apache.commons.lang3.StringUtils {
	
    private static final char SEPARATOR = '_';

    /**
     * 驼峰命名法工具
     *
     * @return toCamelCase(" hello_world ") == "helloWorld"
     *         toCapitalizeCamelCase("hello_world") == "HelloWorld"
     *         toUnderScoreCase("helloWorld") = "hello_world"
     */
    public static String toCamelCase(String s) {
        if (s == null) {
            return null;
        }

        s = s.toLowerCase();

        StringBuilder sb = new StringBuilder(s.length());
        boolean upperCase = false;
        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);

            if (c == SEPARATOR) {
                upperCase = true;
            } else if (upperCase) {
                sb.append(Character.toUpperCase(c));
                upperCase = false;
            } else {
                sb.append(c);
            }
        }

        return sb.toString();
    }

    /**
     * 驼峰命名法工具
     *
     * @return toCamelCase(" hello_world ") == "helloWorld"
     *         toCapitalizeCamelCase("hello_world") == "HelloWorld"
     *         toUnderScoreCase("helloWorld") = "hello_world"
     */
    public static String toCapitalizeCamelCase(String s) {
        if (s == null) {
            return null;
        }
        s = toCamelCase(s);
        return s.substring(0, 1).toUpperCase() + s.substring(1);
    }

    /**
     * 驼峰命名法工具
     *
     * @return toCamelCase(" hello_world ") == "helloWorld"
     *         toCapitalizeCamelCase("hello_world") == "HelloWorld"
     *         toUnderScoreCase("helloWorld") = "hello_world"
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
                    sb.append(SEPARATOR);
                }
                upperCase = true;
            } else {
                upperCase = false;
            }

            sb.append(Character.toLowerCase(c));
        }

        return sb.toString();
    }

    /**
     * 获得当天是周几
     */
    public static String getWeekDay() {
        String[] weekDays = { "Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat" };
        Calendar cal = Calendar.getInstance();
        cal.setTime(new Date());

        int w = cal.get(Calendar.DAY_OF_WEEK) - 1;
        if (w < 0) {
            w = 0;
        }
        return weekDays[w];
    }

}
```
## 7. PageHelper使用
（1）引入依赖
```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.3.1</version>
</dependency>
```
***注意：版本和SpringBoot是否兼容，一般最新版的就和最新版的SpringBoot结合使用。***
（2）在需要查询的代码前插入该代码，开启分页

```java
PageHelper.startPage(page, limit);
```
## 8. Mybatis Plus自定义通用查询
（1）自定义查询辅助注解
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
        //相等
        EQUAL
        // 2017/8/7 大于等于
        , GREATER_THAN
        // 2017/8/7 小于等于
        , LESS_THAN
        // 2017/8/7 中模糊查询
        , INNER_LIKE
        // 2017/8/7 左模糊查询
        , LEFT_LIKE
        //  2017/8/7 右模糊查询
        , RIGHT_LIKE
        //  2017/8/7 小于
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
（2）查询条件解析类

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
					String blurry = StringUtils.toUnderScoreCase(q.blurry());
					String attributeName = isBlank(propName) ? StringUtils.toUnderScoreCase(field.getName()) : propName;
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
}
```

## 9. Spring Data JPA自定义通用查询
（1）自定义查询辅助注解

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
     * 连接查询的属性名，如User类中的dept
     */
    String joinName() default "";

    /**
     * 默认左连接
     */
    Join join() default Join.LEFT;

    /**
     * 多字段模糊搜索，仅支持String类型字段，多个用逗号隔开, 如@Query(blurry = "email,username")
     */
    String blurry() default "";

    enum Type {
        //相等
        EQUAL
        // 2017/8/7 大于等于
        , GREATER_THAN
        // 2017/8/7 小于等于
        , LESS_THAN
        // 2017/8/7 中模糊查询
        , INNER_LIKE
        // 2017/8/7 左模糊查询
        , LEFT_LIKE
        //  2017/8/7 右模糊查询
        , RIGHT_LIKE
        //  2017/8/7 小于
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

    /**
     * 适用于简单连接查询，复杂的请自定义该注解，或者使用sql查询
     */
    enum Join {
        /**2019-6-4 13:18:30 */
        LEFT,
        RIGHT,
        INNER
    }

}
```

（2）查询条件解析类

```java
@Slf4j
@SuppressWarnings({ "unchecked", "all" })
public class QueryHelp {

	public static <R, Q> Predicate getPredicate(Root<R> root, Q query, CriteriaBuilder cb) {
		List<Predicate> list = new ArrayList<>();
		if (query == null) {
			return cb.and(list.toArray(new Predicate[0]));
		}
		// 数据权限验证
		DataPermission permission = query.getClass().getAnnotation(DataPermission.class);
		if (permission != null) {
			// 获取数据权限,All,this_level,cus
			List<Long> dataScopes = SecurityUtils.getCurrentUserDataScope();
			// ???TODO
			if (CollectionUtil.isNotEmpty(dataScopes)) {
				if (StringUtils.isNotBlank(permission.joinName()) && StringUtils.isNotBlank(permission.fieldName())) {
					Join join = root.join(permission.joinName(), JoinType.LEFT);
					list.add(getExpression(permission.fieldName(), join, root).in(dataScopes));
				} else if (StringUtils.isBlank(permission.joinName())
						&& StringUtils.isNotBlank(permission.fieldName())) {
					list.add(getExpression(permission.fieldName(), null, root).in(dataScopes));
				}
			}
		}
		try {
			List<Field> fields = getAllFields(query.getClass(), new ArrayList<>());
			for (Field field : fields) {
				boolean accessible = field.isAccessible();
				// 设置对象的访问权限，保证对private的属性的访问
				field.setAccessible(true);
				Query q = field.getAnnotation(Query.class);
				if (q != null) {
					String propName = q.propName();
					String joinName = q.joinName();
					String blurry = q.blurry();
					String attributeName = isBlank(propName) ? field.getName() : propName;
					Class<?> fieldType = field.getType();
					Object val = field.get(query);
					if (ObjectUtil.isNull(val) || "".equals(val)) {
						continue;
					}
					Join join = null;
					// 模糊多字段
					if (ObjectUtil.isNotEmpty(blurry)) {
						String[] blurrys = blurry.split(",");
						List<Predicate> orPredicate = new ArrayList<>();
						for (String s : blurrys) {
							orPredicate.add(cb.like(root.get(s).as(String.class), "%" + val.toString() + "%"));
						}
						Predicate[] p = new Predicate[orPredicate.size()];
						list.add(cb.or(orPredicate.toArray(p)));
						continue;
					}
					// 连接查询
					if (ObjectUtil.isNotEmpty(joinName)) {
						String[] joinNames = joinName.split(">");
						for (String name : joinNames) {
							switch (q.join()) {
							case LEFT:
								if (ObjectUtil.isNotNull(join) && ObjectUtil.isNotNull(val)) {
									join = join.join(name, JoinType.LEFT);
								} else {
									join = root.join(name, JoinType.LEFT);
								}
								break;
							case RIGHT:
								if (ObjectUtil.isNotNull(join) && ObjectUtil.isNotNull(val)) {
									join = join.join(name, JoinType.RIGHT);
								} else {
									join = root.join(name, JoinType.RIGHT);
								}
								break;
							case INNER:
								if (ObjectUtil.isNotNull(join) && ObjectUtil.isNotNull(val)) {
									join = join.join(name, JoinType.INNER);
								} else {
									join = root.join(name, JoinType.INNER);
								}
								break;
							default:
								break;
							}
						}
					}
					// 普通查询
					switch (q.type()) {
					case EQUAL:
						list.add(cb.equal(
								getExpression(attributeName, join, root).as((Class<? extends Comparable>) fieldType),
								val));
						break;
					case GREATER_THAN:
						list.add(cb.greaterThanOrEqualTo(
								getExpression(attributeName, join, root).as((Class<? extends Comparable>) fieldType),
								(Comparable) val));
						break;
					case LESS_THAN:
						list.add(cb.lessThanOrEqualTo(
								getExpression(attributeName, join, root).as((Class<? extends Comparable>) fieldType),
								(Comparable) val));
						break;
					case LESS_THAN_NQ:
						list.add(cb.lessThan(
								getExpression(attributeName, join, root).as((Class<? extends Comparable>) fieldType),
								(Comparable) val));
						break;
					case INNER_LIKE:
						list.add(cb.like(getExpression(attributeName, join, root).as(String.class),
								"%" + val.toString() + "%"));
						break;
					case LEFT_LIKE:
						list.add(cb.like(getExpression(attributeName, join, root).as(String.class),
								"%" + val.toString()));
						break;
					case RIGHT_LIKE:
						list.add(cb.like(getExpression(attributeName, join, root).as(String.class),
								val.toString() + "%"));
						break;
					case IN:
						if (CollUtil.isNotEmpty((Collection<Long>) val)) {
							list.add(getExpression(attributeName, join, root).in((Collection<Long>) val));
						}
						break;
					case NOT_EQUAL:
						list.add(cb.notEqual(getExpression(attributeName, join, root), val));
						break;
					case NOT_NULL:
						list.add(cb.isNotNull(getExpression(attributeName, join, root)));
						break;
					case IS_NULL:
						list.add(cb.isNull(getExpression(attributeName, join, root)));
						break;
					case BETWEEN:
						List<Object> between = new ArrayList<>((List<Object>) val);
						list.add(cb.between(
								getExpression(attributeName, join, root)
										.as((Class<? extends Comparable>) between.get(0).getClass()),
								(Comparable) between.get(0), (Comparable) between.get(1)));
						break;
					default:
						break;
					}
				}
				field.setAccessible(accessible);
			}
		} catch (Exception e) {
			log.error(e.getMessage(), e);
		}
		int size = list.size();
		return cb.and(list.toArray(new Predicate[size]));
	}

	@SuppressWarnings("unchecked")
	private static <T, R> Expression<T> getExpression(String attributeName, Join join, Root<R> root) {
		if (ObjectUtil.isNotEmpty(join)) {
			return join.get(attributeName);
		} else {
			return root.get(attributeName);
		}
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

	public static List<Field> getAllFields(Class clazz, List<Field> fields) {
		if (clazz != null) {
			fields.addAll(Arrays.asList(clazz.getDeclaredFields()));
			getAllFields(clazz.getSuperclass(), fields);
		}
		return fields;
	}
}
```
## 10. JSR330 在SpringBoot中的使用
在springboot3后，springboot不在提供相关依赖。
（1）引入依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
</dependency>
```
（2）实体类中使用JSR注解验证

```java
@Getter
@Setter
@TableName(value = "t_user")
@ToString(callSuper = true)
@ApiModel(value = "用户类")
public class User extends BaseEntity {
	
	/**
	 * 
	 */
	private static final long serialVersionUID = 2439061274512192367L;
	
	private Integer id;
	
	@ApiModelProperty("姓名")
	@NotEmpty(message = "姓名不能为空")
	@Length(min = 4, max = 30, message = "用户名要在4-30位之间")
	private String name;
	
	@ApiModelProperty("年龄")
	@NotNull(message = "年龄不能为空")
	@Range(message = "年龄大小应为15-70岁", min = 15, max = 70)
	private Integer age;
	
	
	
	@Override
	public int hashCode() {
		final int prime = 31;
		int result = super.hashCode();
		result = prime * result + ((id == null) ? 0 : id.hashCode());
		return result;
	}
	@Override
	public boolean equals(Object obj) {
		if (this == obj)
			return true;
		if (!super.equals(obj))
			return false;
		if (getClass() != obj.getClass())
			return false;
		User other = (User) obj;
		if (id == null) {
			if (other.id != null)
				return false;
		} else if (!id.equals(other.id))
			return false;
		return true;
	}

}
```
（3）增加配置类，让其验证时遇到一个验证不通过就返回，即不会去验证剩下的字段了。

```java
@Configuration
public class ValidatorConfig {
	
    @Bean
    public javax.validation.Validator validator(){
        ValidatorFactory validatorFactory = Validation.byProvider(HibernateValidator.class)
                .configure()
                .addProperty( "hibernate.validator.fail_fast", "true" )
                .buildValidatorFactory();
        return validatorFactory.getValidator();
 
    }

}
```
（4）测试，查看页面验证不通过时返回结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/b5873d465ec14d47a6adc5787da0afaf.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAQFRBTkdA,size_20,color_FFFFFF,t_70,g_se,x_16)
可以看到返回了**MethodArgumentNotValidException**异常信息，但这信息太多，有很多是我们不需要的。所以我们需要增加全局异常处理器。
（5）全局异常处理

```java
@Getter
@Setter
@NoArgsConstructor
public class ExceptionResult {
	private Integer code;
	private String message;
	private Long times;
	
	public ExceptionResult(Integer code, String message) {
		this.code = code;
		this.message = message;
		this.times = System.currentTimeMillis();
	}
}

@RestControllerAdvice
public class GolbalExceptionHandler {
	@ExceptionHandler(MethodArgumentNotValidException.class)
	public ResponseEntity<ExceptionResult> MethodArgumentNotValidExceptionHandler(MethodArgumentNotValidException exception) {
		return ResponseEntity.status(HttpStatus.BAD_REQUEST).
				body(new ExceptionResult(400, exception.getAllErrors().get(0).getDefaultMessage()));
	}
}
```
（6）再次测试
![在这里插入图片描述](https://img-blog.csdnimg.cn/23609e09113747398db7aa8259e123b7.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAQFRBTkdA,size_20,color_FFFFFF,t_70,g_se,x_16)
看到是我们想要的返回信息。
## 11. MapStruct通用配置
前提：maven版本要是3.6.0+。lombok版本要是1.16.16+。
（1）引入依赖

```xml
<properties>
    <org.mapstruct.version>1.4.2.Final</org.mapstruct.version>
    <org.projectlombok.version>1.18.20</org.projectlombok.version>
</properties>
<!-- mapstruct实体类之间映射 -->
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>${org.mapstruct.version}</version>
</dependency>
<!-- lombok dependencies should not end up on classpath -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>${org.projectlombok.version}</version>
    <optional>true</optional>
    <scope>provided</scope>
</dependency>

 <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <source>1.8</source> <!-- depending on your project -->
        <target>1.8</target> <!-- depending on your project -->
        <annotationProcessorPaths>
            <path>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${org.projectlombok.version}</version>
            </path>
            <path>
                <groupId>org.mapstruct</groupId>
                <artifactId>mapstruct-processor</artifactId>
                <version>${org.mapstruct.version}</version>
            </path>
            <!-- other annotation processors -->
        </annotationProcessorPaths>
    </configuration>
</plugin>
```
（2）配置通用mapstruct

```java
public interface BaseMapstruct<D, E> {
    /**
     * DTO转Entity
     * @param dto /
     * @return /
     */
    E toEntity(D dto);

    /**
     * Entity转DTO
     * @param entity /
     * @return /
     */
    D toDto(E entity);

    /**
     * DTO集合转Entity集合
     * @param dtoList /
     * @return /
     */
    List <E> toEntity(List<D> dtoList);

    /**
     * Entity集合转DTO集合
     * @param entityList /
     * @return /
     */
    List <D> toDto(List<E> entityList);
}
```
（3）定义一个需要使用的mapstruct继承basestruct类

```java
@Mapper(componentModel = "spring",unmappedTargetPolicy = ReportingPolicy.IGNORE)
public interface UserMapstruct extends BaseMapstruct<UserDto, User> {

}
```
## 12. 日期转换工具类

```java

package cn.com.ncsi.pm.common.utils;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.time.OffsetDateTime;
import java.time.ZoneId;
import java.time.format.DateTimeFormatter;
import java.util.Date;

public class DateUtils {

    public static final DateTimeFormatter DFY_MD_HMS = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    public static final DateTimeFormatter DFY_MD = DateTimeFormatter.ofPattern("yyyy-MM-dd");
    private final static DateTimeFormatter EXCEL_DATE_PATTERN = DateTimeFormatter.ofPattern("M/dd/yyy");

    /**
     * LocalDateTime 转时间戳
     *
     * @param localDateTime /
     * @return /
     */
    public static Long getTimeStamp(LocalDateTime localDateTime) {
        return localDateTime.atZone(ZoneId.systemDefault()).toEpochSecond();
    }

    /**
     * 时间戳转LocalDateTime
     *
     * @param timeStamp /
     * @return /
     */
    public static LocalDateTime fromTimeStamp(Long timeStamp) {
        return LocalDateTime.ofEpochSecond(timeStamp, 0, OffsetDateTime.now().getOffset());
    }

    /**
     * LocalDateTime 转 Date
     * Jdk8 后 不推荐使用 {@link Date} Date
     *
     * @param localDateTime /
     * @return /
     */
    public static Date toDate(LocalDateTime localDateTime) {
        return Date.from(localDateTime.atZone(ZoneId.systemDefault()).toInstant());
    }

    /**
     * LocalDate 转 Date
     * Jdk8 后 不推荐使用 {@link Date} Date
     *
     * @param localDate /
     * @return /
     */
    public static Date toDate(LocalDate localDate) {
        return toDate(localDate.atTime(LocalTime.now(ZoneId.systemDefault())));
    }


    /**
     * Date转 LocalDateTime
     * Jdk8 后 不推荐使用 {@link Date} Date
     *
     * @param date /
     * @return /
     */
    public static LocalDateTime toLocalDateTime(Date date) {
        return LocalDateTime.ofInstant(date.toInstant(), ZoneId.systemDefault());
    }

    /**
     * 日期 格式化
     *
     * @param localDateTime /
     * @param patten /
     * @return /
     */
    public static String localDateTimeFormat(LocalDateTime localDateTime, String patten) {
        DateTimeFormatter df = DateTimeFormatter.ofPattern(patten);
        return df.format(localDateTime);
    }

    /**
     * 日期 格式化
     *
     * @param localDateTime /
     * @param df /
     * @return /
     */
    public static String localDateTimeFormat(LocalDateTime localDateTime, DateTimeFormatter df) {
        return df.format(localDateTime);
    }
    

    /**
     * 日期格式化 yyyy-MM-dd HH:mm:ss
     *
     * @param localDateTime /
     * @return /
     */
    public static String localDateTimeFormatyMdHms(LocalDateTime localDateTime) {
        return DFY_MD_HMS.format(localDateTime);
    }
    
    /**
     * 日期格式化 M/dd/yyyy
     *
     * @param localDateTime /
     * @return /
     */
    public static String localDateTimeFormatMdy(LocalDateTime localDateTime) {
        return EXCEL_DATE_PATTERN.format(localDateTime);
    }

    /**
     * 日期格式化 yyyy-MM-dd
     *
     * @param localDateTime /
     * @return /
     */
    public String localDateTimeFormatyMd(LocalDateTime localDateTime) {
        return DFY_MD.format(localDateTime);
    }

    /**
     * 字符串转 LocalDateTime ，字符串格式 yyyy-MM-dd
     *
     * @param localDateTime /
     * @return /
     */
    public static LocalDateTime parseLocalDateTimeFormat(String localDateTime, String pattern) {
        DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern(pattern);
        return LocalDateTime.from(dateTimeFormatter.parse(localDateTime));
    }

    /**
     * 字符串转 LocalDateTime ，字符串格式 yyyy-MM-dd
     *
     * @param localDateTime /
     * @return /
     */
    public static LocalDateTime parseLocalDateTimeFormat(String localDateTime, DateTimeFormatter dateTimeFormatter) {
        return LocalDateTime.from(dateTimeFormatter.parse(localDateTime));
    }

    /**
     * 字符串转 LocalDateTime ，字符串格式 yyyy-MM-dd HH:mm:ss
     *
     * @param localDateTime /
     * @return /
     */
    public static LocalDateTime parseLocalDateTimeFormatyMdHms(String localDateTime) {
        return LocalDateTime.from(DFY_MD_HMS.parse(localDateTime));
    }

    /**
     * 
     * @param yearValue
     * @param datePattern
     * @return
     */
	public static String localDateFormat(LocalDate yearValue, String datePattern) {
		 DateTimeFormatter df = DateTimeFormatter.ofPattern(datePattern);
	     return df.format(yearValue);
	}
	
    /**
     * 字符串转 LocalDate ，字符串格式 yyyy-MM-dd
     *
     * @param localDate /
     * @return /
     */
    public static LocalDate parseLocalDateFormat(String localDate) {
    	return LocalDate.from(DFY_MD.parse(localDate));
    }
    
    /**
     * 
     * @param localDate
     * @param pattern
     * @return
     */
    public static LocalDate parseLocalDateFormat(String localDate, String pattern) {
    	DateTimeFormatter df = DateTimeFormatter.ofPattern(pattern);
    	return LocalDate.from(df.parse(localDate));
    }
    
    /**
	 * 日期格式化为字符串
	 * @param date
	 * @param pattern
	 * @return
	 */
	public static String dateToStr(Date date, String pattern) {
		SimpleDateFormat sdf =  new SimpleDateFormat(pattern);
		return date == null ? null : sdf.format(date);
	}
	
	/**
	 * 字符串转日期
	 * @param str
	 * @param pattern
	 * @return
	 */
	public static Date strToDate(String str, String pattern) {
		SimpleDateFormat sdf =  new SimpleDateFormat(pattern);
		try {
			return sdf.parse(str);
		} catch (ParseException e) {
			// TODO Auto-generated catch block
			throw new RuntimeException(e.getMessage());
		}
	}
}

```
## 13. 快速拷贝vue打包文件到springboot资源下。
**文件名pack-script.bat**
```bash
@echo on
echo "删除template下的index.html"
del ..\file\src\main\resources\templates\index.html
echo "删除static下的所有文件"
del /q ..\file\src\main\resources\static\css\*.*
del /q ..\file\src\main\resources\static\fonts\*.*
del /q ..\file\src\main\resources\static\img\*.*
del /q ..\file\src\main\resources\static\js\*.*
del /q ..\file\src\main\resources\static\*.*
echo "删除static"
rd ..\file\src\main\resources\static\*
rd ..\file\src\main\resources\static
echo "创建static及下方文件夹"
mkdir ..\file\src\main\resources\static
mkdir ..\file\src\main\resources\static\css
mkdir ..\file\src\main\resources\static\fonts
mkdir ..\file\src\main\resources\static\img
mkdir ..\file\src\main\resources\static\js
echo "拷贝文件"
copy dist\static\*.* ..\file\src\main\resources\static
copy dist\static\css\*.* ..\file\src\main\resources\static\css
copy dist\static\fonts\*.* ..\file\src\main\resources\static\fonts
copy dist\static\img\*.* ..\file\src\main\resources\static\img
copy dist\static\js\*.* ..\file\src\main\resources\static\js
copy dist\index.html ..\file\src\main\resources\templates\index.html
pause

```

