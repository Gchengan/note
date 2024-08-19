# Gateway三大核心

## 1.Route(路由)

路由是构建网关的基本模块，它由ID，目标URL，一系列的断言和过滤器组成，如果断言为true则匹配该路由

## 2.Predicate(断言)

参考的是Java8的java.util.function.Predicate，开发者可以匹配Http请求中的所有内容（例如请求头或请求参数），如果请求与断言想匹配则进行路由

## 3.Filter(过滤)



指的是Spring框架中的GatewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改

## 官方文档地址：[Glossary :: Spring Cloud Gateway](https://docs.spring.io/spring-cloud-gateway/reference/spring-cloud-gateway/glossary.html)



# Gateway工作流程

## 官网地址：[How It Works :: Spring Cloud Gateway](https://docs.spring.io/spring-cloud-gateway/reference/spring-cloud-gateway/how-it-works.html)


## 工作流程总结

客户端向Spring Cloud Gateway发出请求，然后在Gateway Handler Mapping中找到与请求相匹配的路由，将其发送到Gateway Web Handler，Handler再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。过滤器用虚线分隔的原因是过滤器可以在发送代理请求之前（Pre）或之后（Post）执行业务逻辑。