# GitHub PR Template
## PR(Pull Request) 이란
```
PR 은 GitHub Repository 에 있는 Branch 에 푸시한 변경사항을 팀원들에게 알릴 수 있는 기능입니다.
```

이를 통해 많은 기업들과 사람들이 개발 프로세스에 대한 문서역할과 기능의 변경사항을 함께 논의하고 검토할 수 있습니다.   
또한 PR 을 통해 미래의 다른 팀원, 개발자들에게 변경된 코드에 대한 기록을 남길 수 있습니다.  

## 그렇다면 왜 좋은 PR 을 작성해야 할까?
좋고 깔끔한 PR 을 작성함으로써 다른 사람이 코드가 아닌 작성된 PR 내용을 통해 변경되거나 추가된 이유를 확인할 수 있고, 
어떻게 문제를 해결했고, 특정 구현방법을 왜 사용했는지 대한 내용을 알 수 있습니다.   
좋은 PR 은 아래의 내용들이 포함되어 있어야 합니다.


1. 코드를 추가한 이유와 무엇을 해결하였는가?
2. 어떻게 문제를 해결하였는가?
3. 문제를 해결하면서 무엇인 변경되었는가?(인프라, 새로운 모델, 새로운 패턴 등...)
4. 추가 및 변경된 사항에 관련된 스크린 샷


이러한 내용들을 통해 아래와 같은 이점들을 얻을 수 있습니다.


1. 리뷰어가 더 빠르고, 더 나은 코드리뷰가 가능합니다. 리뷰어가 작성된 PR 내용들을 통해 코드에 대한 빠른 이해가 가능하기에, 집중적인 설명과 토론을 제공하기에 더 쉽습니다. 
2. 미래에 나를 포함한 개발자들에 대한 문서입니다. 미래의 개발자는 이전 코드가 작성되었을 때 프로젝트의 상태를 되돌아보고 상기할 수 있기 때문에 새로운 변경사항이 시스템에 어떤 영향을 미칠지에 대해 더 잘 알 수 있습니다.
3. 작업한 코드에 대한 이유를 설명함으로써 코드를 더 강력하고 읽기 쉽게 구성하고 테스트할 수 있도록 정리할 수 있는 부분들을 찾을 수 있습니다.

## PR 은 크기는 얼마가 적합할까?
- PR 의 크기는 작은 하나의 기능에 하나의 PR 로 진행합니다.
- PR 은 200 줄 정도가 적합하다고 합니다.

https://smallbusinessprogramming.com/optimal-pull-request-size/

## GitHub Repository 에서 PR Template 를 만드는 방법

https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository

위의 링크에서 만드는 방법에 대해 잘 설명되어 있습니다. 저는 ```.github/pull_request_template``` 디렉토리 구조로 템플릿을 구성했습니다.

### 참고자료
https://blog.carbonfive.com/why-write-good-pull-requests/  

