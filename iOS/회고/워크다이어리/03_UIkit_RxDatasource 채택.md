
# [워크다이어리] UIKit vs SwiftUI | RxDatasource vs DiffableDataSource?

## 들어가며
언어는 당연히 Swift를 사용하지만, UI관련해서 고민을 좀 해 보았다
<br>

## 어떤 언어로 적용해볼까
UIKit or SwiftUI 두가지 밖에 없었다

<br>

## 선택은?
이번 프로젝트에서는 SwiftUI를 도입해볼까 고민 중이었으나,

최근 [Swift UI 3.0](https://smkdir.tistory.com/8)을 정리하면서도 생각한대로 iOS 15 미만에서 아직 미 지원되는 기능이 많고

[iOS 버전 별 점유율](https://developer.apple.com/kr/support/app-store/)을 참고할 때 아직 30%가량이 15버전 미만을 사용하고 있어 시기상조로 판단해서 이번 프로젝트 까지는 UIKit으로 선정했다.

UIKit을 사용하되 조금 더 경험하지 못한 기능을 도입해볼 방법은 없을까?
<br>

## Discussion
* 이번 프로젝트에는 Rxswift가 아닌 Combine을 사용하여 구현해볼 수 있을까?
  * DiffableDatasource를 사용해보면 어떨까

### DiffableDatasource
* DiffableDatasource는 [Advances in UI Data Sources](https://developer.apple.com/videos/play/wwdc2019/220/)에서 잘 설명되어있다
* `performBatchUpdates` 가 사라지고 [apply](https://developer.apple.com/documentation/uikit/uitableviewdiffabledatasource/3375811-apply)라는 단일 메소드가 있다
* `Snapshot` 이란 개념이 도입된다
  * Snapshot은 간단히 말해서 현재 UI state의 truth라고 볼 수 있다
  * `section` 과 `item` 에 대해 각각이 `Unique Identifier`가 있으며, `IndexPath`가 아닌 `Unique Identifier`로 업데이트 하게 된다

![snapshot1](https://github.com/vincent-k-sm/Blog/raw/master/iOS/%ED%9A%8C%EA%B3%A0/%EC%9B%8C%ED%81%AC%EB%8B%A4%EC%9D%B4%EC%96%B4%EB%A6%AC/images/snapshot1.png)
![snapshot2](https://github.com/vincent-k-sm/Blog/raw/master/iOS/%ED%9A%8C%EA%B3%A0/%EC%9B%8C%ED%81%AC%EB%8B%A4%EC%9D%B4%EC%96%B4%EB%A6%AC/images/snapshot2.png)

<br>

## Conclusion
이번에는 UIKit + Combine + DiffableDatsource로 적용하기로 결정했다
<br>
Combine을 채택한 건 향 후 SwiftUI를 반영할 때 Combine에 바로 연결하기 위해서 선택했다

DiffableDatasource의 경우 가장 큰 목적은 아래와 같다
* 최소한의 라이브러리로 구현하고 싶다
  * Rx를 안쓰고 만들어보고 싶다
* 최소버전 iOS 13부터 사용가능하나 그동안 도입해본적이 없다 
    
<br>

## 진짜 회고
* 현재 이 글을 작성하는 시점에서는 상기 기준에 따라 모두 작업이 완료 되었다,<br>
결과적으로 UIKit과 Combine 자체의 결합도는 그리 크지 않았는데
* 구현하는 UI는 모두 `Snapkit`을 사용하여 코드 베이스로 구현되었고,<br>
이벤트 바인딩 `(eg. Button, textfield)` 등에 대한 부분은 별도의 Combine Wrapping을 통해 rx와 유사하게 사용할 수 있었다.

<br>

* DiffableDatasource를 사용하면서 장점으로는 rxswift와 유사하게 viewModel의 이벤트 바인딩으로 보다 간결한 코드 관리가 가능했다.

<br>

* 만약 기본 CollectionView, TableView Delegate 를 사용했다면, ViewModel의 데이터를 바인딩하여 relaod 하는 방식으로 해야했지만, <br>
DiffableDatasource는 SnapShot관리를 통해 자연스럽게 바인딩이 가능하여 보다 명확한 코드 관리가 가능했다. 
<br>

* 이 때 조금 어려웠던 부분이 있었는데 RxDatasource와 마찬가지로 각 데이터가 Unique 한 identifire을 가지고 있어야 하고, 나는 각 `Section과 Cell` 을 `enum` 으로 관리하고 `enum associated value` 를 기반ㅇ로 바인딩 부분을 간결하게 하고자 했는데 ,
* 이 때 class와 struct중 어떤 것을 채택하는가에 따라 shapshot이 다르게 동작하는 경우가 있었다.

<br>

* 간략하게 남겨보자면 `apply snapshot` 시 애니메이션 부분에서 이슈가 있었는데, <br>
개인적으로는 데이터 수정이 발생했을 때 `fade` 효과를 선호했지만 `diffable은 Indentifier` 값에 따라 신규 객체로 인지되면 오른쪽으로 사라지고 왼쪽에서 들어오는 애니메이션을 보여주었다

<br>

* 결론적으로 fade 효과를 넣기 위해서는 각 identifier 값을 별도로 저장하거나 class 형태로 값은 보존 시키는 방법이 필요했는데 여기에서 많은 삽질이 있었다.

<br>

* 상기 [DiffableDatasource](#diffabledatasource)에서 보았듯이 이는 자동으로 이루어지는 부분이고 cell에 대한 Associcated Value를 일일히 직접 접근해서 처리하는건 너무 부하가 커보여서 여러가지 방법을 채택해보았다.

* 기존 데이터 객체를 관리하는 별도의 변수를 만들어 해당 데이터를 apply한다
* 이번에는 `Hashable`, `Equtable`을 채택하여 사용햇는데 다음에는 `Identifiable`을 채택해볼 수 있을 것 같다

CASE 1
* 데이터 목록 전체가 갱신되어야 하는경우
* 아래 케이스는 검색어에 따른 리스트를 필터하여 배치하는 경우이다

```swift
let updateChannelList: [SlackChannelInfo] = self.slackChannelList.filter({ $0.name.contains(searchText) })
self.filteredSlackChannelList = searchText.isEmpty ? self.slackChannelList : updateChannelList

var snapShot = self.dataSource.snapshot()
snapShot.deleteSections([.slackChannelList])
snapShot.insertSections([.slackChannelList], afterSection: .slackChannelSearchSection)

let newCells: [SlackManageCellTypes] = self.filteredSlackChannelList.map { item -> SlackManageCellTypes in
    return .slackChannelInfo(item: item)
}

snapShot.appendItems(newCells, toSection: .slackChannelList)

DispatchQueue.main.async {
    self.dataSource.apply(snapShot, animatingDifferences: true)
    
}
```

CASE 2
* 각각의 데이터가 수정되어야 하는 경우
* 아래 케이스는 리스트에서 선택 한 채널의 Selected 값을 변경하는 경우이다.

```swift
let id = reportChannelId
let updateChannelList: [SlackChannelInfo] = self.slackChannelList.map { channel in
    var newChannel = channel
    newChannel.isReporting = (newChannel.id == id)
    
    return newChannel
}

self.slackChannelList = updateChannelList
self.dataSource.snapshot.reloadSections([.slackChannelList])
DispatchQueue.main.async {
    self.dataSource.apply(self.dataSource.snapshot, animatingDifferences: true)
}
```


## 마치며
* diffable datasource를 enum 타입으로 사용할 때 데이터 갱신이 가장 어려웠는데 관련 예제를 찾기가 너무 어려웠다
* 다음 iOS 15 점유율이 더욱 높아지면 iOS 15에서 사용가능한 [applySnapshotUsingReloadData](#https://developer.apple.com/documentation/uikit/views_and_controls/collection_views/updating_collection_views_using_diffable_data_sources)를 채택해봐야겠다
* 22년 하반기 전까지는 Swift UI 3.0 으로 미리 리팩토링을 준비해봐야 할 것 같다