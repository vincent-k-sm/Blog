> ### 연관 글

-   [\[기획\] - 기획서(스토리보드) 작성 방법 | 01 - 시작하기](https://smkdir.tistory.com/1)
-   [\[기획\] - 기획서(스토리보드) 작성 방법 | 02 - 서비스 개요](https://smkdir.tistory.com/2)
-   [\[기획\] - 기획서(스토리보드) 작성 방법 | 03 - IA (Infromation Architecture)](https://smkdir.tistory.com/3)

<br><br>
> ### 서비스 프로세스 란

-   앞서 작성한 IA 를 기반으로 서비스의 주요 기능 및 제공되는 컨텐츠 등에 대해 도식화 하는단계입니다.
-   이 하 "프로세스 설계서" 로 통칭합니다
<br>
<br>
> ### 프로세스 설계서 작성의 목표

-   IA(Information Architecture) 와 비슷한 내용으로 작성되나 프로세스 정의 단계에서는 사용자 행위의 도식화를 목표합니다.
-   서비스 전반에 대한 흐름을 디자이너/개발자 가 가시적으로 확인할 수 있도록 유도합니다.
-   이 정의 단계를 통해 추 후 메뉴 트리 설계에 있어 자연스러운 흐름을 선행하여 기획할 수 있습니다

<br><br>
> ### 프로세스 설계서 작성 방법

-   프로세스 설계서는 사용자의 동선에 따른 큰 단위의 도식화를 초점으로 작성합니다.
-   이 전 IA 작성에서 다뤘던 프로젝트를 기준으로 예시를 들어보겠습니다
-   주요 포인트를 잡는다면 아래와 같습니다
    -   사용자의 동선에 따라 수립한다
    -   권한에 따른 프로세스 분기는 적용하되 '기능'에 대한 분기는 처리하지 않는다
        -   권한에 따른 기능분기는 추후 상세 기획에서 flow chart로 명시합니다

아래는 토이프로젝트로 진행하는 스터디 매칭 플랫폼의 예시입니다.  

더보기

[##_Image|kage@cveNbN/btqY7nPOuc5/4S3kQGKpVyAneVb1jd0nv1/img.png|alignCenter|data-filename="뱃지스터디_프로세스.png" data-origin-width="1072" data-origin-height="3627" data-ke-mobilestyle="widthContent"|||_##]

<br><br>

> #### 상기 예시 중 `스터디 구성 프로세스 中 (생성/참여)` 를 기준으로 자세한 구성요소를 풀어보겠습니다

[##_Image|kage@HjEHG/btqYW2TvBFm/s0s5YRtkvBKM74aFXjcx7K/img.png|alignCenter|data-filename="1.png" data-origin-width="1492" data-origin-height="827" data-ke-mobilestyle="widthContent"|||_##][##_Image|kage@4MElm/btqYLz5YqHc/U3Z3QMCcqOgigsZ78UHfTk/img.png|alignCenter|data-filename="2.png" data-origin-width="1492" data-origin-height="827" data-ke-mobilestyle="widthContent"|||_##][##_Image|kage@6UoAf/btqYN1A8mPH/BsHuEAtp6wUwFUdVQUdk00/img.png|alignCenter|data-filename="3.png" data-origin-width="1492" data-origin-height="827" data-ke-mobilestyle="widthContent"|||_##]

<br>
위 내용을 통해 붉은색으로 표시한 내용을 다시 정리 하면 아래와 같습니다

1.  역할이 명확히 구분될 수 있도록 한다
2.  조건문 (IF) 에 따른 분기는 상세 기획서 내 Flow Chart에서 처리한다
3.  프로세스 정의서에서는 전체적인 흐름을 보기 위함이며, 보다 깊은(deep) 프로세스 설명이 필요한 경우 별도로 분리한다
4.  프로세스 내 알림 등 부가적인 사항이 있다면 가능한 추가한다 (시스템 설계가 보다 용이해진다)

<br><br>
> ### 정리

**프로세스 정의서는 시스템 전반의 사용자 흐름(동선)을 정리한다**

-   앞서 IA(Information Architecture)는 기능 하나하나에 대한 정의가 이루어지며, 프로세스 정의서는 해당 기능들이 동작하는 흐름을 보여줄 수 있습니다.
-   이 과정에서 IA가 수정될 수 있습니다.
-   각 사용자의 동선을 중심으로 작성됩니다.

<br><br>

**프로세스의 단위는 정해져있지 않다**

-   후에 다른 글에서 다룰 예정이지만, 기획서는 `보는 사람을 위한 문서`입니다.
    -   `보는 사람`이 이 문서를 통해 인지하고 이해 할 수 있는 단위라면 조금 더 세분화 되거나 더 통합되어도 무방합니다.
-   **궁극적인 목표는 이 프로세스 정의서를 통해 서비스 전반에 대한 흐름을 이해할 수 있도록 하는데 중점을 둡니다**

<br><br>
> #### 이어지는 글
>
> -   이어지는 글에서는 메뉴트리(Menu Tree)에 대해 작성될 예정입니다

<br><br>

> ##### 본 블로그 글은 작성자의 기준에서 작성된 내용이며 참고용으로만 이용해주세요