
# [워크다이어리] WebView Component 모듈화 하기

## 들어가며
아.. 생각해보니 UI Component 외에 Webview도 계속 사용하고 있었다<br>
또한 향후 웹앱 몇가지에 대한 계획이 있어서 이번에 SPM(Swift Package Manager) 작업을 한김에 웹뷰 컴포넌트도 만들어보고자 했다(와이프가 프론트엔드 개발자이다)



<br>

## 기능 선정
생각한 컴포넌트는 아래와 같다
1. 기본적인 url기반 웹페이지 출력
2. `app scheme` (host)를 활용해 새창 띄우기
3. `app scheme`의 url parameter 기반으로 상단 `네비게이션 바` 세팅
4. Custom `UserScript` 설정
5. Header & Cookie 설정
6. 웹뷰 간 `ProcessPool` 공유 
7. `JavaScript Interface` 연동 

<br>

## 기술 검토 & Conclusion
마찬가지로 SPM(`Swift Package Manager`) 으로 채택! <br>
추가로 이번 프레임워크의 핵심은 앱의 부가적인 처리 없이 `scheme`만으로 현재 페이지 `dismiss(pop)`, `push`, `popToRoot` 하는에 중점을 두었다<br>
이는 새로운 페이지를 내부 페이지 redirect로 처리하는 경우 깜박거리거나 web 환경에 따라 history가 남지 않아 backSwipe로 돌아가는 등의 사용자 경험이 좋아지지 않고<br>이에 보다 자연스러운 UX 제공을(네이티브 같은..?) 위함이었다

<br>

## 구현과정
* 이 전 경험을 토대로 이번엔 처음부터 xib없이 작업을 시작했다(사실 웹뷰 컴포넌트 하나라 큰 부담도 없었다)
* app Scheme Parameter에 대한 기준을 수립했다
```swift
/// Webview를 세팅하는 MKWebViewConfiguration 입니다
/// - Parameters:
///   - host: deeplink 에 활용되는 host가 적용됩니다 (plist 내 등록된 url Types를 모두 체크합니다)
///   - urlString: landing url 을 설정합니다
///   - title: 네비게이션 바 내 페이지 타이틀을 설정합니다
///   - leftBtn: 좌측 버튼 (<-) 을 활성화 합니다
///   - rightBtn: 우측 버튼 (x) 을 활성화 합니다
///   - navigationColor: 네비게이션 바의 색상을 지정합니다 (hexString)
///   - statusBarColor: 상단 스테이터스 바의 색상을 지정합니다 (hexString)
///   - tintColor: 네비게이션 바 내 버튼, 타이틀 등의 색상을 지정합니다 (hexString)
///   - removeStack:기존 stack 에 쌓여있는 모든  controller를 제거합니다
///   - navigationBarIsEnable: host, urlString, navigationBarIsEnable, removeStack 값에 따라 네비게이션바 노출 여부를 설정합니다
///   - important:host, urlString, navigationBarIsEnable, removeStack 을 제외하고 모든 value가 nil 인 경우 네비게이션바가 노출되지 않습니다
public struct MKWebViewConfiguration: Codable {
    public var host: String                = ""
    public var urlString: String           = ""
    public var title: String?              = nil
    public var leftBtn: Bool?              = nil
    public var rightBtn: Bool?             = nil
    public var navigationColor: String?    = nil
    public var statusBarColor: String?     = nil
    public var tintColor: String?          = nil
    public var removeStack: Bool?          = nil
    public var navigationBarIsEnable: Bool = false
```
* 이 외 선정한 기능의 상세 명세는 해당 프레임워크의 예제 앱과 페이지에서 확인할 수 있도록 정리하였다
  * Git : https://github.com/vincent-k-sm/MKWebviewBridge

* 테스트할 웹뷰페이지를 정적으로 구현하였다
  * Test Web page : https://smbh.kr/mk_bridge/sample


* iOS 14이상부터 웹 <-> 앱 간 통신 시 promise를 사용할 수 있음을 알게되었다
```swift
public func userContentController(_ userContentController: WKUserContentController,
                                    didReceive message: WKScriptMessage,
                                    replyHandler: @escaping (Any?, String?) -> Void
) {
        var bodyMsg = ""
        if let body = message.body as? String {
            bodyMsg = body
        }
        for (key, value) in _replycallbacks {
            if message.name == key {
                
                let v = value.handler
                replyHandler(v.result, v.error)
                value.callBack(bodyMsg)
                break
            }
        }
                
        SystemUtils.shared.print("name: \(message.name), body: \(bodyMsg)", self)
    }
```
* 기존 방식은 웹에서 데이터 요청 시 callback 으로 데이터를 가져갈 수 없어서 `Trigger` 메소드와 적용 메소드를 페어로 관리했어야 하는데 사용성이 더욱 좋아진 것 같다


#### AS-IS
```javascript
//- javascript
function getToken() {
    webkit.messageHandlers.getToken.postMessage('');
}

// Return
function setToken(token) {
    document.getElementById("passDataResult").innerHTML = `${token}`;
    return "success"
}

//- swift
addPostMessageHandler("getToken") { (res) in
    let value: [String: Any] = ["token": UUID().uuidString]
    self.callJavaScript(script: "setToken", value: value)
}
```
#### TO-BE
```javascript

//- javascript
function handleInfoFromApp(fromApp) {
    document.getElementById("passDataResult").innerHTML = `Promise button taaped: ${fromApp}`;
}

function handleError(err) {
}

function sampleMethodTheHTMLCanCall(inputInfo, successFunc, errorFunc) {
    var promise = window.webkit.messageHandlers.testWithPromise.postMessage(inputInfo);
    promise.then(
        function (result) {
            console.log(result); // "Stuff worked!"
            successFunc(result)
        },
        function (err) {
            console.log(err); // Error: "It broken"
            errorFunc(err)
        });
}


//- swift
let promiseResult: ReplyHandler = ("\(UUID().uuidString)", nil)
if #available(iOS 14.0, *) {
    addPostMessageReplyHandler("testWithPromise", handler: promiseResult, result: { result in
        print("Called")
    })
}
```


## 마치며
* 위 과정에서 테스트 웹뷰 페이지에 앱 에서 설정된 `header`도 보여주도록 많은 시간을 투자했지만 `javascript + html` 만으로는 헤더 정보를 출력할 수 없었다
    * 여유가 생긴다면 Node.js 를 사용해서 나중에 반영해볼까 고민중이다
* iOS 14이상부터 사용가능한 API지만 Promise를 적용할 수 있다는 사실을 배웠다
* 테스트 웹페이지를 만들면서 Jquery를 처음써봤는데 잘 쓰면 간단하게 이것저것 만들어볼 수 있겠는걸..?

## 비고
전체 코드는 [github-MKWebviewBridge](https://github.com/vincent-k-sm/MKWebviewBridge#load-url-based) 에서 확인할 수 있습니다 :)
* 영상 : https://user-images.githubusercontent.com/24787667/160655332-d618e441-6147-40f9-b263-f002eae52a35.mov
