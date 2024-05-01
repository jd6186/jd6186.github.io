---
layout: post
title: "[Python] Java Email(Gmail) 발송 가이드"
tags: [BackEnd JAVA GMAIL EMAIL]
---

## Intro
안녕하세요 **Noah**입니다.<br/>
오늘은 확인해보니 Gmail 발송에 대해 최신정보가 반영된 블로그 글을 찾기 어려워 작성하게 되었습니다.<br/>
이 글에서는 구글 계정 설정이 예전과 달라진 것들이 있어(2024년 05월 01일 기준) 그 부분을 이미지로 다루고 있습니다. 꼭 참고 해주세요!(전 이걸 찾는게 더 오래걸렸습니다 ㅠㅠ)<br/>
기본적인 환경은 Java 17, Spring Boot 3.2.3을 사용했고 Gmail을 발송하는 방법에 대해 알아보겠습니다.<br/>
<br/><br/><br/><br/>

## 본문
### 1. Gmail 설정
* Gmail 설정에서 2차 인증을 설정합니다.
  * ![1.png](..%2F..%2F..%2Fassets%2Fimg%2FBackEnd%2FJava%2F2024-05-01-GmailSend%2F1.png)
* 앱 비밀번호를 생성합니다.
  * ![2.png](..%2F..%2F..%2Fassets%2Fimg%2FBackEnd%2FJava%2F2024-05-01-GmailSend%2F2.png)
* 앱을 생성해줍니다.
  * ![3.png](..%2F..%2F..%2Fassets%2Fimg%2FBackEnd%2FJava%2F2024-05-01-GmailSend%2F3.png)

<br/><br/>

### 2. 의존성 추가
```gradle
// Email
implementation group: 'com.sun.mail', name: 'javax.mail', version: '1.6.2'
implementation group: 'com.google.api-client', name: 'google-api-client', version: '2.4.1'
implementation 'com.google.oauth-client:google-oauth-client-jetty:1.34.1'
implementation 'com.google.apis:google-api-services-gmail:v1-rev20220404-2.0.0'
implementation 'com.google.auth:google-auth-library-oauth2-http:1.23.0'
```

<br/><br/>

### 3. 환경변수 설정
GMAIL_EMAIL, GMAIL_PASSWORD 환경변수를 설정합니다.

```dockerfile
# 베이스 이미지로 OpenJDK 17을 사용
FROM openjdk:17-slim as build

# Gradle 래퍼를 사용하기 위한 준비
WORKDIR /app
COPY gradlew .
COPY gradle gradle
COPY build.gradle .
COPY settings.gradle .
COPY src src

# gradlew에 실행 권한 부여
RUN chmod +x ./gradlew

# 애플리케이션 빌드
RUN ./gradlew build -x test

# 빌드 결과물을 실행할 새로운 스테이지
FROM openjdk:17-slim
WORKDIR /app
COPY --from=build /app/build/libs/app.jar app.jar

ENTRYPOINT java -jar app.jar -Dspring.profiles.active=${SPRING_PROFILES_ACTIVE} -Dspring.datasource.password=${SPRING_DATASOURCE_PASSWORD} -Djwt.secret=${JWT_SECRET} -Dtoss.api.secretKey=${TOSS_API_SECRETKEY} -Dserver.port=${SERVER_PORT} -Dgmail.email=${GMAIL_EMAIL} -Dgmail.password=${GMAIL_PASSWORD}
```

<br/><br/>

### 4. 메일 발송 서비스 구현
```java
import lombok.AllArgsConstructor;
import lombok.Getter;

@Getter
@AllArgsConstructor
public class GmailSendDto {
    private String to;
    private String isSuccess;
}
```

```java
import com.plapp.indist_front_api.core.code.IsSuccess;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import javax.mail.*;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;
import java.util.List;
import java.util.Properties;

@Component
@Slf4j
public class EmailClient {
    @Value("${gmail.email}")
    private String EMAIL_ADDRESS;
    @Value("${gmail.password}")
    private String PASSWORD;

    public GmailSendDto sendEmail(String toEmail, String subject, String bodyText){
        String isSuccess = IsSuccess.SUCCESS.getCode();
        Session session = getSession();

        // Email 발송
        try {
            MimeMessage message = new MimeMessage(session);
            message.setFrom(new InternetAddress(EMAIL_ADDRESS));
            message.addRecipient(Message.RecipientType.TO, new InternetAddress(toEmail));
            message.setSubject(subject); //메일 제목을 입력
            message.setText(bodyText);    //메일 내용을 입력
            Transport.send(message); ////전송
            log.info("[EMAIL SENT SUCCESS] Email: {}", toEmail);
        } catch (MessagingException e) {
            isSuccess = IsSuccess.FAIL.getCode();
            log.error("[EMAIL SENT ERROR] Email: {} \tReason: {}", toEmail, e.getMessage());
        }

        // Email 발송 결과 반환
        return new GmailSendDto(toEmail, isSuccess);
    }

    private Session getSession() {
        // Email 발송을 위한 설정
        Properties prop = new Properties();
        prop.put("mail.smtp.host", "smtp.gmail.com");
        prop.put("mail.smtp.port", 465);
        prop.put("mail.smtp.auth", "true");
        prop.put("mail.smtp.ssl.enable", "true");
        prop.put("mail.smtp.ssl.trust", "smtp.gmail.com");
        return Session.getDefaultInstance(prop, new Authenticator() {
            protected PasswordAuthentication getPasswordAuthentication() {
            return new PasswordAuthentication(EMAIL_ADDRESS, PASSWORD);
            }
        });
    }

    // 다중 발송
    public List<GmailSendDto> sendMultiEmail(List<String> toEmails, String subject, String bodyText){
        return List.of(
                toEmails.stream()
                        .map(toEmail -> sendEmail(toEmail, subject, bodyText))
                        .toArray(GmailSendDto[]::new)
        );
    }
}
```
<br/><br/><br/><br/>

## 글을 마치며
이번 포스트에서는 Java와 Spring Boot를 활용하여 Gmail 이메일 발송 기능을 구현하는 방법에 대해 알아보았습니다. 처음부터 Gmail 설정, 의존성 추가, 환경변수 설정, 그리고 실제 메일 발송 로직까지 단계별로 설명을 드렸습니다. 특히, 보안이 강화된 구글 계정 설정을 위해 2차 인증과 앱 비밀번호 생성이 필요하다는 점을 강조하고 실제 활용 예를 이미지로 보여드렸습니다.

본 가이드를 통해 개발자 여러분이 자신의 Java 애플리케이션에서 사용자의 이메일로 직접 알림을 보낼 수 있는 기능을 손쉽게 구현할 수 있기를 바랍니다. 또한, 이메일 발송 기능은 사용자와의 커뮤니케이션뿐만 아니라, 애플리케이션의 오류 알림, 사용자 인증 등 다양한 분야에서 활용될 수 있습니다.

혹시라도 설정 과정에서 문제가 발생하거나 추가적으로 궁금한 점이 있다면 언제든지 댓글을 통해 질문해 주시기 바랍니다. 앞으로도 유용한 개발 팁과 정보를 지속적으로 공유할 예정이니, 많은 관심과 응원 부탁드립니다.

다들 즐코하세요~ 🚀
<br/><br/>

