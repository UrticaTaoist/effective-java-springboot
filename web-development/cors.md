# Cors

CORS（跨域资源共享），这是由W3C提出的。

浏览器有同源策略， 源\[origin\]就是协议、域名和端口号,同时同源策略还对http方法有约束。

部分代码引自 [https://spring.io/blog/2015/06/08/cors-support-in-spring-framework](https://spring.io/blog/2015/06/08/cors-support-in-spring-framework)

1.控制器CORS配置：  
你可以@RequestMapping所拦截的方法上添加@CrossOrigin，默认情况下允许所有源和方法。

```text
@RestController
@RequestMapping("/account")
public class AccountController {

	@CrossOrigin
	@GetMapping("/{id}")
	public Account retrieve(@PathVariable Long id) {
		// ...
	}

	@DeleteMapping("/{id}")
	public void remove(@PathVariable Long id) {
		// ...
	}
}
```

也可以为整个Controller启用CORS

```text
@CrossOrigin(origins = "http://domain2.com", maxAge = 3600)
@RestController
@RequestMapping("/account")
```

也可以组合配置

```text
@CrossOrigin(maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

	@CrossOrigin(origins = "http://domain2.com")
	@GetMapping("/{id}")
```

如果使用了Spring Security，就可以在Security级别启用CORS。  
[https://docs.spring.io/spring-security/site/docs/current/reference/html/cors.html](https://docs.spring.io/spring-security/site/docs/current/reference/html/cors.html)

以上是细粒度的CORS，也可以启用全局的CORS。

2.JavaConfig

为整个应用程序启用CORS

```text
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

	@Override
	public void addCorsMappings(CorsRegistry registry) {
		registry.addMapping("/**");
	}
}
```

如果是使用了SpringBoot，则可以这样使用

```text
@Configuration
public class MyConfiguration {

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurerAdapter() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/**");
            }
        };
    }
}
```

也可以定制配置

```text
@Override
public void addCorsMappings(CorsRegistry registry) {
	registry.addMapping("/api/**")
		.allowedOrigins("http://domain2.com")
		.allowedMethods("PUT", "DELETE")
			.allowedHeaders("header1", "header2", "header3")
		.exposedHeaders("header1", "header2")
		.allowCredentials(false).maxAge(3600);
}
```

3.基于过滤器的CORS  
 Spring Framework还提供了[CorsFilter](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/filter/CorsFilter.html)

```text
@Configuration
public class MyConfiguration {

	@Bean
	public FilterRegistrationBean corsFilter() {
		UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
		CorsConfiguration config = new CorsConfiguration();
		config.setAllowCredentials(true);
		config.addAllowedOrigin("http://domain1.com");
		config.addAllowedHeader("*");
		config.addAllowedMethod("*");
		source.registerCorsConfiguration("/**", config);
		FilterRegistrationBean bean = new FilterRegistrationBean(new CorsFilter(source));
		bean.setOrder(0);
		return bean;
	}
}
```





