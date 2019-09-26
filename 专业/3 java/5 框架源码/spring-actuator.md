# spring-actuator
<span style="color:red">红字</span>为主要执行路径，<span style="color:blue">蓝字</span>为暂不分析的代码

## 架构
spring-actuator 由各种 Endpoint 构成，而 Endpoint 由 WebMvcEndpointHandlerMapping 注册到 springMVC

## 主配置
- 参见 ManagementContextAutoConfiguration
- 参见 ServletManagementContextAutoConfiguration

        management.server.port=8080
        management.server.ssl.enabled=true
        management.server.ssl.key-store=classpath:management.jks
        management.server.ssl.key-password=secret


- 注意 @EnableManagementContext、EnableChildManagementContextConfiguration，他们会开启 WebMvcEndpointManagementContextConfiguration、ServletEndpointManagementContextConfiguration，而这两个类是Endpoint 与 springMVC 之间的衔接

        @Configuration
        @EnableManagementContext(ManagementContextType.SAME)
        static class EnableSameManagementContextConfiguration {

        }

        @Override
        public void onApplicationEvent(WebServerInitializedEvent event) {
            if (event.getApplicationContext().equals(this.applicationContext)) {
                ConfigurableWebServerApplicationContext managementContext = this.managementContextFactory
                        .createManagementContext(this.applicationContext,
                                EnableChildManagementContextConfiguration.class,
                                PropertyPlaceholderAutoConfiguration.class);
                managementContext.setServerNamespace("management");
                managementContext.setId(this.applicationContext.getId() + ":management");
                setClassLoaderIfPossible(managementContext);
                CloseManagementContextListener.addIfPossible(this.applicationContext, managementContext);
                managementContext.refresh();
            }
        }

## Endpoint配置
- EndpointAutoConfiguration
- JmxEndpointAutoConfiguration
- WebEndpointAutoConfiguration

### Metrics配置
- MetricsAutoConfiguration
- CompositeMeterRegistryAutoConfiguration
- SimpleMetricsExportAutoConfiguration

## 参考引用
0. [SpringBoot源码之Actuator - Evan_L的博客 - CSDN博客](https://blog.csdn.net/Evan_L/article/details/87191042)