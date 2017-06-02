---
layout: "post"
title: "跨域问题(spring/spring-boot)"
date: "2017-06-02 10:33"
---

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

	- [一. 简介](#一-简介)
	- [二.什么是跨域](#二什么是跨域)
	- [三.跨域产生的原因](#三跨域产生的原因)
	- [四.控制器方法级别的 CORS配置](#四控制器方法级别的-cors配置)
	- [五.全局级别配置CORS](#五全局级别配置cors)
	- [六.基于Filter的CORS](#六基于filter的cors)
	- [七.自定义的处理器](#七自定义的处理器)

<!-- /TOC -->

## 一. 简介

```
由于安全原因,浏览器禁止Ajax 调用/访问非当前域下的资源.例如,在一个标签页上,你正在检查你的银行卡账户,你可以在另一个标签页中打开evil.com 网站.来自evil.com域下的脚本不能通过Ajax请求来访问你的银行的API(从你的账户中提取现金)通过使用你的证书.
```
## 二.什么是跨域

cross-orgin resouce sharing(cors)是一个w3c标准被大多数浏览器所实现,通过一种灵活的方式来指定哪一种请求被授权,而不是使用一种不安全和不太强的方式如iframe或者jsonp.

在spring-framework 4.2 ,跨域已经在盒子外面被支持了.跨域请求被自动的分配到了各种各样的已经注册过的HandlerMappings.CorsProcessor处理CORS预检请求并且拦截 简单的和实际的CORS 请求.同时添加相关的cors响应头基于你添加的cors 配置.

> 由于CORS 请求是自动的被分配,你不需要改变DispatcherServlet /DispatchOptionRequest 的初始化参数,使用默认值是一个建议的做法.

## 三.跨域产生的原因
1. 请求的发起者与请求的接受者的
  2. 域名不同
  3. 端口号不同

## 四.控制器方法级别的 CORS配置

1. 通过@CrossOrigin 注解在@RequestMapping注解上就可以打开CORS.通过默认的 @CrossOrigin允许所有的域和Http方法访问特定的@RequestMapping的注解.

```
@RestController
@RequestMapping("/account")
public class AccountController {

	@CrossOrigin
	@RequestMapping("/{id}")
	public Account retrieve(@PathVariable Long id) {
		// ...
	}

	@RequestMapping(method = RequestMethod.DELETE, path = "/{id}")
	public void remove(@PathVariable Long id) {
		// ...
	}
}

```

2. 也可以通过在控制器方法级别来处理跨域问题

```
@CrossOrigin(origins = "http://domain2.com", maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

	@RequestMapping("/{id}")
	public Account retrieve(@PathVariable Long id) {
		// ...
	}

	@RequestMapping(method = RequestMethod.DELETE, path = "/{id}")
	public void remove(@PathVariable Long id) {
		// ...
	}
}
```

## 五.全局级别配置CORS
1. 为了添加细粒土的/基于注解的配置,你讲可以定义一些全局的CORS配置.这个跟filters很像,但是可以声明在SpringMVC里,并且可以有更细粒度的配置跨域配置.

```
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

	@Override
	public void addCorsMappings(CorsRegistry registry) {
		registry.addMapping("/api/**")
			.allowedOrigins("http://domain2.com")
			.allowedMethods("PUT", "DELETE")
			.allowedHeaders("header1", "header2", "header3")
			.exposedHeaders("header1", "header2")
			.allowCredentials(false).maxAge(3600);
	}
}
```

2. 基于配置文件的


```
<mvc:cors>

	<mvc:mapping path="/api/**"
		allowed-origins="http://domain1.com, http://domain2.com"
		allowed-methods="GET, PUT"
		allowed-headers="header1, header2, header3"
		exposed-headers="header1, header2" allow-credentials="false"
		max-age="123" />

	<mvc:mapping path="/resources/**"
		allowed-origins="http://domain1.com" />

</mvc:cors>
```


## 六.基于Filter的CORS
 为了支持CORS基于安全框架像spring-Security,或者其他的库不支持本地CORS,SPring Framework也提供了CorsFilter.替代使用@CrossOrgin or WebMvcConfigurerAdapter#addCorsMappings(CorsRegistry),就需要
 定义个自定义的过滤器.

```
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

public class MyCorsFilter extends CorsFilter {

	public MyCorsFilter() {
		super(configurationSource());
	}

	private static UrlBasedCorsConfigurationSource configurationSource() {
		CorsConfiguration config = new CorsConfiguration();
		config.setAllowCredentials(true);
		config.addAllowedOrigin("http://domain1.com");
		config.addAllowedHeader("*");
		config.addAllowedMethod("*");
		UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
		source.registerCorsConfiguration("/**", config);
		return source;
	}
}

```

## 七.自定义的处理器

corsConfiguration 允许你来配置什么样的cors请求应该被处理
1. 允许的域(origins)
2. header信息
3. methods方法
同时他还可以提供更多的方式:
1. AbstractHandlerMapping#setCorsConfiguration() 允许你描述一个Map集合,使用一些cors的配置实例来匹配
2. 通过子类提供自己的corsConfiguration() 来重写AbstractHandlerMapping#getConfiguration(Object,HttpServletRequest)方法
3. 处理器快车实现corsConfiguration接口就像ResourceHttpRequestHandler来为每一个请求提供一个实例
