---
layout: post
title: "[AWS] AWS CloudFront 내 WAF 설정 간단 설명"
tags: [AWS, CloudFront, WAF, Security]
---

## 소개
안녕하세요 **Noah**입니다.

오늘은 AWS CloudFront에서 WAF(Web Application Firewall)를 설정하여 웹 애플리케이션을 보호하는 방법에 대해 자세히 설명해드리겠습니다. 보안은 모든 웹 서비스의 중요한 부분이며, 이 글을 통해 여러분의 서비스를 더욱 안전하게 보호할 수 있기를 바랍니다.

## CloudFront에서 WAF 설정이 필요한 이유
CloudFront는 AWS에서 제공하는 CDN(Content Delivery Network) 서비스로, 전 세계에 콘텐츠를 빠르게 제공합니다. 하지만, 이러한 공개적 접근성은 다양한 보안 위협에 노출될 수 있습니다. SQL 인젝션, 크로스 사이트 스크립팅(XSS), 사이트 간 요청 위조(CSRF) 등 다양한 공격으로부터 웹 애플리케이션을 보호하는 것이 중요합니다.

WAF는 이러한 공격을 탐지하고 차단할 수 있는 규칙 기반의 보안 계층을 제공하여, 애플리케이션의 보안을 강화합니다.

## WAF 설정 시 중점적으로 다루어야 할 부분
### 1. SQL 인젝션 및 XSS 공격 방지
가장 흔하게 발생할 수 있는 보안 위협인 SQL 인젝션과 XSS 공격을 방지하기 위한 WAF 규칙을 설정하는 것이 필수적입니다. AWS WAF에서는 이러한 공격 패턴을 식별하여 차단할 수 있는 다양한 규칙 세트를 제공합니다.

### 2. 지리적 차단 설정
특정 국가에서 오는 트래픽을 차단하거나 허용하고 싶을 때 지리적 차단 규칙을 설정할 수 있습니다. 이는 특정 지역에서의 악의적인 시도를 예방하는 데 유용합니다.

### 3. 접근 제어 및 속도 제한
비정상적인 트래픽 증가가 감지되었을 때 자동으로 트래픽을 제한하거나 특정 IP 주소 또는 IP 범위에서 오는 요청을 차단하는 규칙을 설정할 수 있습니다. 이는 DDoS 공격 같은 위협으로부터 웹 애플리케이션을 보호하는 데 중요합니다.

## WAF에서 맞춤 관리가 필요한 주요 부분
- **IP 주소 차단**: 특정 IP 주소 또는 범위에서 오는 접근을 차단하는 규칙을 설정하여 악의적인 접근을 차단합니다.
- **문자열 및 SQL 쿼리 필터링**: 웹 요청 중 의심스러운 문자열 패턴이나 SQL 쿼리가 포함된 경우 이를 차단하는 규칙을 설정합니다.
- **HTTP 헤더 검사**: HTTP 헤더를 검사하여 악의적인 스크립트나 명령어 삽입 시도를 차단합니다.
- **세션 토큰 유효성 검사**: 세션 토큰의 유효성을 검사하여 세션 하이재킹 공격을 방지합니다.

## WAF 설정 방법
1. AWS Management Console에서 WAF 서비스로 이동합니다.
2. WAF 서비스 대시보드에서 "CloudFront"로 접속 후 적용하길 원하는 CloudFront를 선택합니다.
3. Security 탭에서 Edit을 누릅니다.
4. 가능하면 아래 이미지와 같이 설정을 진행해줍니다. WAF를 통해 SQL 인젝션, 통신 시간 등을 관리하는 목적으로 사용됩니다.
   - ![create_cloudFront_WAF.png](..%2F..%2F..%2Fassets%2Fimg%2FDevOps%2FAWS%2F2024-04-18-AWS_CloudFront_WAF%2Fcreate_cloudFront_WAF.png)
5. 설정이 완료되면 "Save"를 눌러 설정을 저장합니다.
6. 설정이 적용된 CloudFront를 통해 애플리케이션에 접속하여 WAF가 제대로 동작하는지 확인합니다.
7. 이 때 아래 영역 내 존재하는 로그 확성화 버튼을 클릭하면 다음과 같은 이미지로 Log들일 활성화 된 것을 확인 가능합니다.(다만 요청당 비용이 발생하므로 신중하게 선택해주시기 바랍니다. 글 작성일 기준 100만 요청당 0.76$)
   - ![active_WAF_log.png](..%2F..%2F..%2Fassets%2Fimg%2FDevOps%2FAWS%2F2024-04-18-AWS_CloudFront_WAF%2Factive_WAF_log.png)
   - 로그 기록 확인 방법(CloudWatch Lop Group에서 확인 가능)
     - CloudFront는 글로벌 서비스기 때문에 Log 활성화했을 시 Virginia 리전에 로그 그룹이 자동으로 생성됩니다. 따라서 리전은 버지니아로 선택해야 합니다.
       - ![cloudWatch_log_group.png](..%2F..%2F..%2Fassets%2Fimg%2FDevOps%2FAWS%2F2024-04-18-AWS_CloudFront_WAF%2FcloudWatch_log_group.png)
8. 설정이 완료되면 WAF가 제대로 동작하는지 CloudFront Security 탭 및 CloudWatch를 통해 주기적으로 모니터링하고, 필요에 따라 규칙을 업데이트하거나 추가하는 것이 좋습니다. 모니터링 시에는 아래 순서대로 WebACL에 접근해 Rule 조작 및 로그 내용을 확인해줍니다. 다음은 Rule을 관리하는 예시입니다.
   - 예시 파일 사이즈 제한
       - AWS Management Console에서 WAF 서비스로 이동합니다.
         - ![WAF_Web_ACL.png](..%2F..%2F..%2Fassets%2Fimg%2FDevOps%2FAWS%2F2024-04-18-AWS_CloudFront_WAF%2FWAF_Web_ACL.png)
       - 원하는 WebACL을 선택하여 Rule을 확인하고, 필요에 따라 추가 또는 수정합니다.
         - 기존 Rule에서 FileSize제한 관련 내용은 Block에서 Allow로 변경
           - ![origin_file_rule.png](..%2F..%2F..%2Fassets%2Fimg%2FDevOps%2FAWS%2F2024-04-18-AWS_CloudFront_WAF%2Forigin_file_rule.png)
         - 새로운 Rule 생성
           - ![ex_file_size.png](..%2F..%2F..%2Fassets%2Fimg%2FDevOps%2FAWS%2F2024-04-18-AWS_CloudFront_WAF%2Fex_file_size.png)
         - 새로운 Rule이 적용된 내용 확인
           - ![custom_WAF_Rule.png](..%2F..%2F..%2Fassets%2Fimg%2FDevOps%2FAWS%2F2024-04-18-AWS_CloudFront_WAF%2Fcustom_WAF_Rule.png)


## 글을 마치며
AWS CloudFront와 WAF를 통해 웹 애플리케이션의 보안을 강화하는 방법에 대해 진짜 간단하게 알아보았습니다. 이 설정을 통해 사용자의 데이터와 서비스를 다양한 방법으로 보호할 수 있습니다. 하지만 지나치게 보안 위주로 설정할 경우 서비스 운용이 어려워질 수 있으므로 보안팀과 적절한 논의를 통해 서비스를 안전하게 운용하셨으면 좋겠습니다.

또한 보안 설정은 지속적인 관리와 업데이트가 필요하므로, AWS의 최신 보안 동향과 업데이트를 주기적으로 확인하는 것은 필수겠습니다.

앞으로도 개발과 관련된 다양한 지식과 경험을 공유할 예정이니, 관심 있으신 분들은 자주 방문해 주세요. 또한, 이 포스트를 통해 생긴 궁금증이나 추가적으로 알고 싶은 내용이 있다면 언제든지 댓글로 남겨주세요. 여러분의 피드백은 제가 더 나은 콘텐츠를 만드는 데 큰 도움이 됩니다.

이 글이 여러분의 프로젝트에 도움이 되길 바라며, 글을 마치겠습니다. 건승하세요! ^^