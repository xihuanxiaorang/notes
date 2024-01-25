# SpringSecurity 源码剖析（一）

## 简介

## 环境搭建

添加 `spring-boot-starter-security` 依赖，如下所示：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

核心组件：

- SecurityBuilder

- HttpSecurity

- WebSecurity

- SecurityConfigurer

- Authentication
- AuthenticationManager
- ProviderManager
- AuthenticationProvider
- AbstractUserDetailsAuthenticationProvider
- DaoAuthenticationProvider
- UserDetailsService
- UserDetails
- AbstractAuthenticationProcessingFilter
- UsernamePasswordAuthenticationFilter
- AuthenticationSuccessHandler
- AuthenticationFailureHandler



## 参考资料🎁

- 文档
  - [Spring Security - 官方英文文档](https://spring.io/projects/spring-security#learn)
  - [Spring Security 中文文档](https://springdoc.cn/spring-security/index.html)
  - [江南一点雨 - SpringSecurity系列](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI1NDY0MTkzNQ==&action=getalbum&album_id=1319828555819286528&scene=173&from_msgid=2247488106&from_itemidx=1&count=3&nolastread=1#wechat_redirect)
  - [Spring Security | 码农小胖哥的博客](https://www.felord.cn/categories/spring-security/)
- 视频
  - [从源码开始精通SpringSecurity](https://www.bilibili.com/video/BV1Rv4y1w74n/?spm_id_from=333.999.top_right_bar_window_history.content.click&vd_source=bf3d4320498e90d36e1361cc18b45e48)，十分推荐！