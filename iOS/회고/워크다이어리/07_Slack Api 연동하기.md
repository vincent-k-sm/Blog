
# [워크다이어리] Slack Api 연동하기

## 들어가며
초기 Slack Api 를 검토 할 당시 필요한 API가 있는지 여부에 대해서는 검증이 되었지만, <br>
실제 적용까지는 다소 난해함을 겪어서 결과를 정리하고자 추가한다

<br>

## 관련 정보의 부재 
Slack Api 는 관련 예제를 찾아보기 힘들었다, <br>
웹뷰 내에서 OAuth를 통해 연결해야 했고, 결과를 scheme으로 받아 redirect하는 등의 과정이 필요했는데 <br>
이러한 과정을 그냥 맨 바닥에서 진행하는데 있어 어려움을 겪었다..

## 구현 방법

### Slack Apps 생성하기
1. 사용중인 Slack계정으로 로그인 후 [앱 관리 페이지](https://api.slack.com/apps) 에서 앱을 생성한다
2. 이 후 과정은 https://api.slack.com/authentication/basics 를 따르면 무난하게 진행할 수 있었다.
<br>
밑에서 추가로 얘기하겠지만 가장 해맸던 부분은 생성한 앱의 permission 관련 사항이었는데 이걸 앱에 적용하기 위해서는 거쳐야 하는 과정이 있었다..

### Slack 로그인 하기
1. [OAuthSwift](https://github.com/OAuthSwift/OAuthSwift) 라이브러리를 채택하였다
2. Slack Api 만 사용하면 되어서 자체적으로 구현할까도 고민해보았으나, 쉬운길을 택했다..
<br>
무난하게 되는가 싶었으나 여기서 부터 문제는 시작되었다. 

#### 테스트 디바이스에서는 되는데 개인 디바이스에서는 안된다?
먼저 테스트 디바이스로 (iPhone XR) 구현하고 `슬랙 계정에 구글로그인`을 시도 했을때는 문제 없이 로그인에 성공했다.<br>
문제는 개인 디바이스 (iPhone mini 12)로 로그인을 시도하니 access denied 에러가 계속 발생했다.<br>
결론부터 얘기하면 `실험적 기능` 이 문제였다
> 경로 : 디바이스 설정 -> Safari -> 고급 -> `Experimental Features` 

<br>

이곳에서 켜져있는 기능이 두 디바이스가 상이했는데 이를 테스트 디바이스와 동일하게 설정한 뒤부터 정상적으로 동작되었다<br>
( 이 이슈를 찾기 위해 거의 1주일은 날린것 같다 ㅠ-ㅠ )

<br>

#### API 사용을 위해 userToken 발급받기
앞선 문제를 해결하고 실제 메시지 발송에 필요한 user_access_token을 발급받고자 했다.
이 때 [권한Scope](https://api.slack.com/scopes) 을 참고해서 로그인 시 scope 정보를 보냈어야 하는데 iOS 관련 가이드가 없어 위에서 채택한 `OAuthSwift` 의 예제 앱을 참고 했다 <br>
다만 해당 예제 앱은 OAuth 1.0 기준으로 되어있어 결국 우연히 안드로이드 관련 코드를 보고 해소할 수 있었다 
<br>
결과적으로 함께 요청해야 하는 scope은 아래과 같았다
```swift
"channels:history,channels:join,channels:manage,channels:read,chat:write,chat:write.customize,groups:history,groups:read,im:history,im:read,mpim:history,mpim:read,chat:write.public,users:read&user_scope=channels:history,channels:read,chat:write,groups:read,im:history,im:read,mpim:history,mpim:read,users:read,users:write,channels:write"
```

이 Scope 들은 앱 생성 시 설정한 Permission을 기준으로 설정했어야 하는데 처음에는 아무리 동일하게 맞춰도 `Invalid Scope` 에러가 뜨면서 적용되지 않았다.. 
![slack1](https://github.com/vincent-k-sm/Blog/raw/master/iOS/%ED%9A%8C%EA%B3%A0/%EC%9B%8C%ED%81%AC%EB%8B%A4%EC%9D%B4%EC%96%B4%EB%A6%AC/images/slack1.png)

<br>

후에 정말 우연히 Slack Console의 ManageDistribution에서 답을 찾았다..
* 아래 캡처의 `Sharable URL` 을 복사하고 다른곳에 붙여넣어 봤더니 뒤쪽에 scope 이 자동으로 설정되어 있었더라..
* 어느 가이드에도 나와있지 않아 너무 당황스러웠던 기억이 있다 (내가 못읽은걸까..ㅠㅠ)
 
<br>

![slack1](https://github.com/vincent-k-sm/Blog/raw/master/iOS/%ED%9A%8C%EA%B3%A0/%EC%9B%8C%ED%81%AC%EB%8B%A4%EC%9D%B4%EC%96%B4%EB%A6%AC/images/slack2.png)

## 마치며 
Slack Api는 별도의 SDK 제공없이 public 한 웹 프론트 기반 api로 document가 구성되어있어 예상보다 진입장벽이 좀 있었다,<br>
심지어 이 회고를 작성하면서 scope url 을 어떻게 찾았었는지 또 잊어버려 시간이 좀 걸렸다..
앞으로 이슈가 생겼을 때 바쁘더라도 정리하고 넘어가는 습관을 들여야겠다
