# 레거시 코드 개선기 (1)

작년 7월 경 팀에 정식으로 배치를 받고 약 3개월 간의 기간동안 주어졌던 온보딩 과제이자, 사실상 첫 업무였던 사내 레거시 코드를 개선했던 경험을 공유하려고 합니다.

제가 배치되었던 팀은 TF라는 특성 상, 한정적인 리소스로 짧은 주기 안에 많은 기능을 만들어내야하는 어려움을 가진 상태였습니다. 따라서 개발자 리소스가 부족했기에 외주 인력을 통해 작성되어 퀄리티가 떨어지는 코드가 다수 존재했습니다.

이런 상황에서 저는 로그인 / 회원가입 / 프로필 생성 등과 같이 앱에 존재하는 모든 인증과 관련된 코드를 개선하는 업무를 받게 되었습니다.   

## AS-IS 분석

개선된 코드를 작성하기에 앞서, 먼저 **우리 서비스의 요구사항을 이해하고 이 요구사항을 현재 프로젝트에서 어떻게 만족시키고 있는 지를 파악하는 것**이 선행되어야 했습니다.

사실 처음에는 클라이언트 단에서 인증과 관련된 코드들이 많아봤자 얼마나 많겠어? 하는 생각을 가지고 있었으나 다소 안일한 생각이었음을 머지않아 알게 되었습니다.

비즈니스 레벨의 앱에서는 단순히 정확한 아이디와 비밀번호를 입력해서 로그인을 성공하는 것 외에도 `서드 파티 로그인`, `캡챠`, `약관`, `제재 유저 처리` 등 고려해야 할 동선이 많았습니다. 본 개선 프로젝트 초기에 들었던 조언 중 ‘인증 관련 개발은 정확한 아이디 비밀번호를 입력한 모범 유저를 들여보내는 것보다 적합하지 않은 유저를 걸러내는 것이 더 초점이 된다’ 라는 말이 기억에 남는데, 프로젝트를 진행하며 그 말에 공감을 많이 했던 것 같습니다.


### 다른 사람의 코드 읽기

코드 수정 작업을 들어가기에 앞서, 현재 코드에 어떤 문제가 있는 지를 팀 내 공유하기 위해 이전 작성자가 짠 코드에 대한 시퀀스 다이어그램을 작성하기로 결정했습니다. 

처음으로 다른 사람의 코드를 하나 하나 읽으며 들었던 생각은 다음과 같습니다.
- **'다른 사람이 읽었을 때 어떻게 생각할까?'를 끊임없이 생각하면서 코드 작성하기**
  - 지금은 레거시일지라도 당시의 상황을 고려했을 때 최선의 코드였을 수 있기에 누구도 함부로 비난해서는 안된다고 생각합니다. 그렇지만 로직 자체가 효율적이지 않은 것을 떠나서, _어떤 목적을 위해 만든 변수이고 함수인지를 파악할 수 있게끔 적절히 이름을 짓는 습관_ 은 꼭 들여야겠다고 생각했습니다.
- **생각보다 중요한 코딩 컨벤션**
  - 나의 개인 사이드 프로젝트가 아니라 언제든지 다른 사람에게 인계될 수 있는 **회사의 프로젝트**이기 때문에, 작업자가 여럿일지라도 한 사람이 작성한 코드처럼 하나의 통일된 스타일을 추구하는 것이 중요함을 느꼈습니다. 띄어쓰기 한 줄에도 의도를 포함할 수 있다고 생각하기 때문에, 명확히 정립된 일관된 가이드 없이 짜여지는 코드들은 추후 인계자의 혼란을 야기할 수 있기 때문입니다. (Lint는 필수인 것 같습니다.)
- **최소한의 문서화**
  - 문서화는 정말 개발자에게 어려운 숙제인 것 같습니다. 특히 저희 TF 팀처럼 스프린트가 몰아치는 상황이라면 더욱 더 어려운 일일 것입니다. 그럼에도 불구하고 한 번의 스프린트마다 분량이 짧을 지라도 '최소 한 개의 기능 정도에 대해서만이라도 문서화를 해보자' 라는 목표를 갖게 되었습니다. (거의 없었다고 봐야하지만, 남아있는 1-2개의 문서가 빛과 소금처럼 느껴졌습니다.)
- **커밋 메시지는 최대한 자세히, 적절한 기능 단위로**
  - 인수자도 없고, 기능에 대한 문서도 없는 상황에 유일하게 의존할 수 있는 기록은 단연 커밋 히스토리 입니다. 이 상황에서 커밋의 단위도 광범위하고, 의미가 모호한 메시지로 기록이 남아있다면 더욱 더 절망적인 상황이 될 것입니다. 따라서 기능을 개발할 때 커밋의 단위와 메시지에 신경을 쓰고, description도 최대한 잘 활용해야겠다고 생각했습니다. 
  종종 '이건 너무 쪼개는 거 아닐까?' 라는 생각을 하기도 했는데,  다른 사람의 커밋 히스토리를 추적하며 코드를 파악하다보니 하나의 통 커밋보다는 '지나칠지라도 잘게 쪼갠 여러 커밋이 더 낫다'라고 결론을 내렸습니다.

사실 어찌보면 당연시 여겨지는 것들이지만, 지켜지지 않은 부분이 생각보다 많았고 지켜지지 않았을 때의 나비 효과는 생각보다 컸습니다. 따라서 앞으로 개발을 할 때 아무리 급한 상황이여도 최소한 위의 상황들은 꼭 유념하며 개발을 해야겠다고 느꼈습니다.

### 시퀀스 다이어그램 작성하기

인계자인 제가 현재 어떤 문제점을 가지고 있는지 파악하는 것도 물론 중요하지만, 앞으로 함께 프로젝트를 진행할 팀원들도 우리 프로젝트가 현재 어떤 문제점을 가지고 있는지 아는 것도 중요한 일입니다. 따라서 모두가 AS-IS 문제 상황에 대해 좀 더 잘 이해할 수 있도록 기존 코드에 대한 시퀀스 다이어그램을 작성하고 공유하는 시간을 가졌습니다. 

학부 시절 소프트웨어 공학 과목을 들었을 때 잠깐 겉핥기로 보았던 경험만 있지, 실제로 시퀀스 다이어그램을 작성해보는 것은 처음이라 이 부분에서 조금 헤맸던 것 같습니다. 저는 시퀀스 다이어그램에 대한 다양한 피드백을 들으며 여러 차례 수정하는 과정을 가졌는데, 그 과정 속에서 시퀀스 다이어그램을 작성할 때 다음과 같은 부분을 고려하며 작성하는 것이 좋겠다는 생각을 했습니다.

**1. 해당 시퀀스 다이어그램의 용도는 무엇인가?**

초기 버전의 시퀀스 다이어그램에서는 클래스명과 변수명들을 그대로 옮겨와서, 어떤 시점에서 어떤 함수를 호출했고 어떤 변수를 사용했는 지를 하나 하나 작성했는데요. 이를 팀원들에게 공유하니 '그냥 코드를 읽는 것 같다'라는 피드백을 받았습니다. 

이런 방식의 시퀀스 다이어그램은 추후 그 코드를 인계받는 사람의 입장에서는 유익한 정보가 될 수 있겠지만 단순히 현재 구조에 어떤 비효율이 있는 지를 공유 받아야하는 입장인 팀원들에게는 불필요한 정보가 너무 많아 가독성이 떨어지는 시퀀스 다이어그램이 되었습니다.

이러한 상황에서는 핵심적이지 않은 자잘한 개체들은 쳐내고 핵심을 파악할 수 있는 요소들 위주로 시퀀스를 구성하는 것이 적절했겠지만, 읽는 사람을 제대로 고려하지 않은 것입니다.

비단 시퀀스 다이어그램 뿐만 아니라, 코드를 작성할 때와 마찬가지로 어떤 문서를 작성할 때 이 문서를 읽는 독자를 먼저 생각하고 작성하는 습관이 중요함을 다시 한 번 느꼈습니다.

**2. 시퀀스의 시작과 끝을 적절하게 설정하기**

초기에는 '인증'이라는 거대한 기능을 하나의 시퀀스에 녹여내고자 했습니다. 시작과 끝에 대한 명확한 기준점을 선정하지 않고 그리기를 시작하다보니, 한바닥으로 보기 어려울 만큼 길어져 이해를 쉽게 하기 위해 그리고자 했던 본래의 목적과 멀어지는 상황이 발생했습니다. 파일로 추출했을 때, 이미지의 width가 너무 커져 깨알같은 글씨가 되버린 것입니다. 이를 확대를 하면서 보니 이 개체가 어떤 개체와 상호작용 하고 있는지를 하나하나 옮겨가며 봐야하는 매우 비효율적인 다이어그램이 되었습니다. 

이러한 문제점을 개선하기 위해 시퀀스를 로그인, 회원가입, 약관 동의, 프로필 설정 등의 기능 단위로 잘게 다이어그램을 쪼갰습니다. 이렇게 개선하니 보고 싶은 부분 위주로 볼 수 있고, 옆으로 휠을 돌려가지 않아도 한바닥 내에서 볼 수 있는 가독성 있는 여러 장의 다이어그램을 산출하게 되었습니다.

**3. 효율적인 도구 활용**

PlantUML, Mermaid 등 시퀀스 다이어그램을 쉽게 그릴 수 있는 매우 다양한 도구들이 존재합니다. 처음에는 이에 대해 잘 몰라서 너무 많은 시간을 소비했었는데, 도구를 활용하니 훨씬 시간을 효율적으로 사용할 수 있었습니다. 적절한 상황에서 유용한 도구를 잘 활용할 줄 아는 것도 능력이기 때문에 어떤 작업을 시작할 때 무작정 진입하기 보다는 적절한 도구가 무엇이 있을 지 알아보고 이를 활용하면 좋을 것입니다.
