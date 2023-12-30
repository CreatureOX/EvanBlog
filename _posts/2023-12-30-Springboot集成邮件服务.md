---
title: "Springboot集成邮件服务"
share: false
categories:
  - 后端技术研究
tags:
  - Spring Cloud
  - Spring Boot
  - 邮件
---

# 集成步骤
1. 添加starter依赖
2. 添加相关配置
3. 调用JavaMailSender发送邮件
   
## 添加starter依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

## 添加相关配置
### Gmail
  选用了587端口。密码若直接使用google账号密码会提示使用应用专属密码，提示信息中有超链接，点开按流程设置即可获得专属密码
  ```
  spring.mail.host=smtp.gmail.com
  spring.mail.port=587
  spring.mail.username={google账号}@gmail.com
  spring.mail.password={google应用专属密码}
  spring.mail.properties.mail.smtp.auth=true
  spring.mail.properties.mail.smtp.starttls.required=true
  spring.mail.properties.mail.smtp.starttls.enable=true
  ```
  
### QQ邮箱
  开启SSL时使用587端口时无法连接QQ邮件服务器
  ```
  spring.mail.host=smtp.qq.com
  spring.mail.username={QQ号}@qq.com
  spring.mail.password={客户端授权码}
  spring.mail.properties.mail.smtp.auth=true
  spring.mail.properties.mail.smtp.port=465
  spring.mail.properties.mail.smtp.starttls.required=true
  spring.mail.properties.mail.smtp.starttls.enable=true
  ```

## 调用JavaMailSender发送邮件
JavaMailSender需要装配才可使用

```java
@Configuration
public class EmailConfig {

    @Resource
    private JavaMailSender mailSender;

}
```

```java
    /**
     * @param to 接收邮箱
     * @param subject 邮件主题
     * @param content 邮件正文
     */
    public void sendSimpleMail(String to, String subject, String content) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom(from);
        message.setTo(to);
        message.setSubject(subject);
        message.setText(content);
        mailSender.send(message);
    }
```
