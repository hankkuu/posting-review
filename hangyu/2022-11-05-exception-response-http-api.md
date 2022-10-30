# Exception 발생과 HTTP Response 형태 
---
> ## 지난번에 Exception 과 HTTP Status Code 에 대한 부분에 의논이 있었고 이제는 Response Format 에 대해 정해봅니다 
---
예외가 발생되었고 그 예외는 내부에서 잡아서 시스템에 정의된 응답 format 으로 만들어서 나가게 됩니다 

이 부분에서 스프링도 자체적인 Default Format 이 있고 다양한 규모있는 서비스 회사들은 회사만의 일관성 있는 응답 스타일로 API 오류를 처리합니다 

```
- spring default 
{
  "timestamp": "2019-02-15T21:48:44.447+0000",
  "status": 404,
  "error": "Not Found",
  "message": "No message available",
  "path": "/123"
}
```

이 부분에서 웹 서비스를 하는 회사는 어쨌든 균일한 처리를 위해 컨벤션이나 표준과 같은 부분이 필요합니다 

---
REST API 에러 핸들링을 표준화 하기 위한 노력으로 IETF (국제 인터넷 표준화 기구) 는 일반화된 에러 핸들링 스키마를 만드는 RFC 7807 을 고안했습니다 
- https://www.rfc-editor.org/rfc/rfc7807
```
- IETF (국제 인터넷 표준화 기구)
{
    "type": "/errors/incorrect-user-pass",
    "title": "Incorrect username or password.",
    "status": 403,
    "detail": "Authentication failed due to incorrect username or password.",
    "instance": "/login/log/abc123"
}
```  
---
위의 표준화 방안도 실제 스프링의 스타일과 달라서 뭔가 어렵네요.. 

그러면 다른 글로벌 빅테크 들은 어떤지 한번 봐야 하겠습니다 (아래 내용은 현행화가 안되었을 수 있습니다)

```
- google 

성공 : Status Code - 200

{
 "kind": "drive#fileList",
 "etag": "\"G9loKy74Mg0FQ-YRqtCj_yTTrpg/AnWmN3F7Chubr4pCYB95O2lqWbk\"",
 "selfLink": "https://content.googleapis.com/drive/v2/files?maxResults=1",
 "nextPageToken": "!!|~EAIakwELEgBSigEKgAEKaPjzroj_____wxO8LC_vdqAA_wH__kNvc21vL....",
 "nextLink": "https://content.googleapis.com/drive/v2/files?maxResults=1&pageToken=!!%7C~EAIakwELEgBSig....",
 "items": [
  ...
 ]
}

실패 : Status Code - 401

{
 "error": {
  "errors": [
   {
    "domain": "global",
    "reason": "required",
    "message": "Login Required",
    "locationType": "header",
    "location": "Authorization"
   }
  ],
  "code": 401,
  "message": "Login Required"
 }
} 
```

```
- facebook 

성공 : Status Code - 200

{
  "data": [
     ... Endpoint data is here
  ],
  "paging": {
    "cursors": {
      "after": "MTAxNTExOTQ1MjAwNzI5NDE=",
      "before": "NDMyNzQyODI3OTQw"
    },
    "previous": "https://graph.facebook.com/me/albums?limit=25&before=NDMyNzQyODI3OTQw"
    "next": "https://graph.facebook.com/me/albums?limit=25&after=MTAxNTExOTQ1MjAwNzI5NDE="
  }
}

실패 : Status Code - 400 
- https://gist.github.com/subicura/8329759

{
  "error":{
    "type":"OAuthException",
    "message":"(#506) Duplicate status message"
  }
}

----- 현재 상황
{
  "error": {
    "message": "Message describing the error", 
    "type": "OAuthException", 
    "code": 190,
    "error_subcode": 460,
    "error_user_title": "A title",
    "error_user_msg": "A message",
    "fbtrace_id": "EJplcsCHuLu"
  }
}

```

```
- twiter 

성공 : Status Code - 200

{
  "created_at": "Sun Jan 05 12:43:39 +0000 2014",
  "id": 419811428917194750,
  "id_str": "419811428917194752",
  ...
}

실패 : Status Code - 400
- https://gist.github.com/subicura/8329759

{
  "errors":  [
     {
      "message": "Bad Authentication data",
      "code": 215
    }
  ]
}

--- 현재 상황 
{
  "errors": [
    {
      "parameter": "start_time",
      "details": "invalid date",
      "code": "INVALID_PARAMETER",
      "value": "",
      "message": "Expected time, got \"\" for start_time"
    }
  ],
  "request": {
    "params": {
      "account_id": "hkk5"
    }
  }
}

```

```
- github 

성공 : Status Code - 200

{
  "name": "Hello-World",
  "description": "This is your first repository",
  "homepage": "https://github.com",
  "private": false,
  "has_issues": true,
  "has_wiki": true,
  "has_downloads": true
}

실패 : Status Code - 400

{
  "message":"Problems parsing JSON"
}

실패 : Status Code - 422

{
   "message": "Validation Failed",
   "errors": [
     {
       "resource": "Issue",
       "field": "title",
       "code": "missing_field"
     }
   ]
}

실패 : Status Code - 403 

{
  "message": "Maximum number of login attempts exceeded. Please try again later.",
  "documentation_url": "http://developer.github.com/v3"
}

```
```
- Spotify 

실패 - Status Code : 400

{
    "error": {
        "status": 400,
        "message": "invalid id"
    }
}

```
```
- AWS S3 
- 믿기 어렵지만 아직도 XML 스타일 이네요.. 
- https://docs.aws.amazon.com/AmazonS3/latest/API/ErrorResponses.html

<?xml version="1.0" encoding="UTF-8"?>
<Error>
  <Code>NoSuchKey</Code>
  <Message>The resource you requested does not exist</Message>
  <Resource>/mybucket/myfoto.jpg</Resource> 
  <RequestId>4442587FB7D0A2F9</RequestId>
</Error>
```

```
- Naver

{
 	"errorMessage": "Authentication failed (인증에 실패하였습니다.)",
 	"errorCode": "024"
}

```

--- 

정리해 보니 더 혼란스럽네요 IETF나 스프링의 고민한 정도는 새발의 피였습니다 

실제 비즈니스를 하는 회사의 유형을 보니 꽤 복잡했습니다 

일단 실패 사례만 추리려고 했다가 성공 사례도 같이 붙여 봤습니다 왜냐하면 실무적인 부분에서 이 부분에 대해서도 논의가 있었습니다

실제 경험에서도 한 회사에서는 Common Response 를 만들고 Header, Body를 HTTP Body 내부에 속성으로 감싸서 사용했고 한 회사에서는 HTTP Body 에 오직 DB의 결과를 바로 담아 사용했습니다 
```
1번 
Common Response 
{
    "result" = "ok"
    "data" = [
        { "key" : "value" },
        { "key" : "value" }
    ]
}
```
```
2번 
[
    { "key" : "value" },
    { "key" : "value" }
]

```
위의 차이는 어떤 걸로 보이시나요? 

규모있는 회사는 1번을 사용하는 것 같습니다. 

지금 개발 하는 부분은 그리 규모 있는 급이 아니라는 토론이 있었고 불필요한 구조를 명분없이 넣을 이유는 없을 것 같아 공통 응답으로 설득할 이유가 없어서 2번을 사용했습니다 

뒤에서 오류 부분과 같이 얘기하겠지만 응답을 한번 감싼 다는 것은 프론트에 부가 정보를 보낸다는 것과 같다고 봅니다 

즉, 프론트에서 백엔드의 응답 결과를 재가공 하려고 한다면 추가적인 정보가 필요할 수 있습니다 

하지만 프론트는 그냥 데이터의 가공이나 비즈니스 적인 응답을 처리하는 것이 아닌 데이터를 보여주는 것에 집중한다면 굳이 부가정보는 필요해 보이지 않습니다 

그래서 공통 응답 체계가 필요없다는 부분을 인정하기로 했습니다 

(백엔드에서 주는 메시지 자체도 전혀 필터링 없이 그대로 보여 줍니다 - 백엔드에서 잘 작성해야 하는 이유)

하지만 500 Error 정도는 프론트에서 일 한번 더 해야 하지 않나 프론트에서도 Error Handling 을 개발해야 하지 않을가 싶긴 합니다 

--- 
# Error Response 

다시 Error Response 로 돌아와서 위의 이유들로 결국 혼자 정하는 것이 아닌 회사 구성원들이 모여서 어떻게 응답을 내보낼 지 협의 과정이 필요하게 됩니다 

(아무렇게나 다 각각 다르게 응답이 된다는 것은 그 회사의 기술 수준이 표준화도 안되어 있고 외부에서 여긴 왜 이렇지? 이런 생각을 심어주게 될 것 같네요...)

즉 일관성있는 코드 스타일을 유지할 수 있는 Exception 처리와 그에 따른 Response Format 을 만들어야 하는 이유가 생깁니다 

그렇지 않으면 클라이언트에서는 예외 처리를 항상 동일 로직으로 처리하기가 어렵습니다 

일반적으로 Spring 에서 Exception Handler 를 사용하는 코드는 아래와 같습니다 
```

```

현재 사용하고 있는 응답 스타일은 아래와 같습니다 

```
{
    "code" : "BadRequest",
    "message" : "빈 값 보내지 말아 주세요 제발.."
}
```

실제 Exception을 만들게 되는 경우를 보면 Business Exception 을 처리하는 경우가 많습니다 

처음에 의논한 부분은 굳이 Case 마다 Excepiton을 만드는 행위를 하지 말자가 있었습니다 

이미 Java or Spring or JPA 등등 각각의 Core 개발자 들이 만들어놓은 Exception 들이 있습니다 

이미 만들어진 다양한 Exception 들이 있는데 이걸 재활용하고 따로 Business Exception을 재정의 하지 말자의 의견이 있었고 전혀 거부감이 없었습니다 

하지만 지금은 조금 달라 졌는데 Business Error Handling 은 Exception 객체가 중요한게 아니라 Error Code 나 Error Message가 중요한 것으로 판단됩니다 

즉 Business Exception 은 내용이 중요하고 정형화 하기 힘들기 때문에 다양한 케이스의 문자화가 중요하게 판단되었습니다 

그리고 두 번째 이유로는 라이브러리의 Exception 과 Businsess Exception 이 언제까지 운이 좋게 맞아 떨어진다고 연속형으로 볼 수 있을지에 대한 막연함이었습니다 

기본적으로 IllegalArgumentException 이나 IllegalStateException JDK7의 Exception이나 JPA의 EntityNotFoundException 의 경우 재활용이 가능한 부분이 확실하지만 JPA를 안쓰는 경우도 있어 정확히 Not Found 의 Message 에 의존하는 것이 더 낫다고 생각 되었습니다 

그래서 Error Code 의 하나 하나 항목이 모두 Business Exeption 의 Case 이고 여기 하나의 파일로 모두 묶어 두어 해당 내용이 인수인계의 대상 데이터로 보는 것이 더 합리적이라고 생각되 었습니다 

Error Code 의 경우는 아래와 같이 사용됩니다 

```

```

Error Response 객체는 아래와 같이 생겼습니다. 
 많이 복잡합니다 

```

```

여기서 추가로 고민하고 싶은 부분은 Error Logging 과 Alarm Sending 에 대한 처리가 추가로 진행 해야 할 부분입니다 

Error 를 만들면 자동 로깅와 자동 알람 발생을 넣는게 맞을까요? 

Error 를 공통적으로 처리하기 때문에 Spring의 @RestControllerAdvice를 사용하게 되고 @ExceptionHandler 를 사용하게 되는데 

여기 메소드 안에서 로깅/알람/응답 이 3개의 경우를 각각 분리해야 할지 아니면 통합해서 처리해야 할지 고민이 생겼습니다 
```
Error Handling Method
{
    log.error("hadle bind exception", exception)
    val error = ErrorResponse.of(exeption)
    Alarm.send(request.uri, error)
    return error.toResponseEntity()
}
```
이 부분은 사내 개발자들과 한번 의논해 볼 예정입니다 





REST API의 에러핸들링에 대한 best practice를 알아보았습니다.

마지막으로 내용을 정리해보자면 
 

- 특정 상태코드를 내려주기
- 응답 바디에 추가적인 정보를 담아주기
- 통일된 방식으로 에러 처리를 하기


왜 오류에 집착하나요? 

- 오류를 빨리 발견하고 고치거나 에러 복구를 실현해 보고 싶습니다 

MSA 환경에서도 서비스가 분산되었을 때 Business Exception 과 별개로 Api Exception을 정의하는 경우도 있습니다 

해당 경우에 대해서는 분산 환경에서 어떻게 Error Handling을 할지 고민 후 진행해보겠습니다 

---
### 참고한 내용 

- https://blog.ull.im/engineering/2019/03/14/service-providers-errors.html
- https://supawer0728.github.io/2019/04/04/spring-error-handling/
- https://gist.github.com/subicura/8329759
- https://gist.github.com/subicura/8329767
- https://pjh3749.tistory.com/273
- https://cheese10yun.github.io/spring-guide-exception/