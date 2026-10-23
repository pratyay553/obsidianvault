---
tags: [email, integration, interview]
---

# Email Services

Related: [[04 - AOP - Aspect Oriented Programming]] · [[05 - Retry Pattern]] · [[15 - Security - JWT OAuth2 MSAL]] · [[20 - Interview Cheat Sheet]]

## 🧩 What it is
The module both **sends** emails (birthday emails, alerts) and **reads** emails (`MailReaderSchedule` — likely processing inbound reports/attachments sent by partners).

## 🔑 Two Different Email Paths

### 1. Classic JavaMail (SMTP)
```java
@Service
public class MailService {
    private final JavaMailSender mailSender;

    public void send(String to, String subject, String body) {
        MimeMessage message = mailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(message, true);
        helper.setTo(to);
        helper.setSubject(subject);
        helper.setText(body, true); // true = HTML
        mailSender.send(message);
    }
}
```
Config in `application.yml`:
```yaml
spring:
  mail:
    host: smtp.example.com
    port: 587
    username: ${MAIL_USER}
    password: ${MAIL_PASS}
    properties.mail.smtp.starttls.enable: true
```

### 2. Microsoft 365 (Modern Auth)
Since Microsoft deprecated basic SMTP auth for M365, this module integrates via:
- **MSAL4J** (Microsoft Authentication Library for Java) — obtains an **OAuth2** access token (see [[15 - Security - JWT OAuth2 MSAL]])
- **EWS Java API** (Exchange Web Services) — uses that token to actually send/read mail against Exchange Online

```java
ConfidentialClientApplication app = ConfidentialClientApplication.builder(
        clientId, ClientCredentialFactory.createFromSecret(clientSecret))
        .authority("https://login.microsoftonline.com/" + tenantId)
        .build();

IAuthenticationResult result = app.acquireToken(
        ClientCredentialParameters.builder(Set.of("https://outlook.office365.com/.default")).build()
        ).get();

ExchangeService service = new ExchangeService(ExchangeVersion.Exchange2016);
service.getHttpHeaders().put("Authorization", "Bearer " + result.accessToken());
```

## 📥 Reading Mail — `MailReaderSchedule`
A scheduled job that connects to a mailbox, scans for new messages matching some criteria (subject/sender), extracts attachments (see [[12 - File Processing]]), and processes the data — a common pattern for partners who deliver daily reports as email attachments instead of an API.

## ❓ Interview Q&A

**Q: Why can't you just use username/password SMTP auth with Microsoft 365 anymore?**
A: Microsoft disabled Basic Authentication for Exchange Online (as of late 2022/2023 for most tenants) for security reasons; you must use **OAuth2 / Modern Authentication**, which is why this module pulls in `MSAL4J` (to get a token) alongside the EWS API.

**Q: How would you make email sending resilient to a temporary SMTP outage?**
A: Wrap the send call in the [[05 - Retry Pattern]], and consider queuing failed emails (e.g., a DB table or a Kafka/RabbitMQ topic — see [[09 - Kafka and RabbitMQ Messaging]]) for later reprocessing rather than losing them.

**Q: What does `MimeMessageHelper` give you over building a `MimeMessage` manually?**
A: A friendlier API for setting recipients, HTML content, attachments, and inline images without manually dealing with MIME multipart structure.

**Q: Where would AOP fit into email sending?**
A: A `MailAspect` (mentioned in the reference sheet) could log every send attempt/failure, measure latency, or even trigger the retry logic — keeping that cross-cutting concern out of `MailService` itself (see [[04 - AOP - Aspect Oriented Programming]]).
