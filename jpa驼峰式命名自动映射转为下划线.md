springdatajpa 使用的默认策略是 ImprovedNamingStrategy

在application.properties里面加上:
#解决Spring Boot集成JPA的Column注解命名字段会自动生成下划线的问题（命名规则）
spring.jpa.hibernate.naming.physical-strategy = org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
