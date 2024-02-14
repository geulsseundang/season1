# RxJS - Subject

## Hot & Cold Observable

### Hot Observable

hot observable은 observable을 생성하자마자 데이터를 흘려보내는 observable이다.
계속 흘려보내는 값을 구독하게 되면 그 시점으로부터 해당 값을 받는 것을 기본으로 한다.

### Cold Observable

cold observable은 일반적인 옵저버 형태를 말하며 누가 구독해주지 않으면 데이터를 배출해주지 않는다. 내가 요청하면 결과를 받는 과정을 거친다.

RxJS는 함수형 프로그래밍 패러다임에 근간을 두고 있어 지연 평가 방식을 사용한다.
그래서 Obserbvable의 세상에서는 생성 시에 선언만 하고 구독이 요청한 시점에 해당 코드를 실행하는 Cold 방식을 갖고 있다.

Observable은 데이터를 요청할 때마다 함수가 실행되어 결과를 전달해주는 방식이라 여러 군데에서 같은 값을 사용하면 데이터를 공유하지 않고 그때마다 선언된 함수를 호출하는 방식이 된다.

이걸 잘 이해하지 못하면 api를 observable로 만들고 여러 군데에서 해당 데이터를 사용했을 때 API 콜이 중복으로 계속 호출되는 문제가 발생하게 된다. 서버 데이터나 상태관리와 같이 api를 다루는 방식엔 hot 방식이 더 잘 어울린다.

그렇지만 RxJS에서 cold한 Observable을 hot하게 만들기 위한 과정이 상당히 복잡하다.
RxJS Observable의 기본이 cold인데 multicasting을 쓰지 않고 Hot Observable을 만들려면 어떻게 해야할까.

이를 위해 Hot Observable 전용 객체인 Subject와 BehaviorSubject가 존재한다.
Hot이 필요할 때 복잡한 multicasting보다는 subject를 사용하면 좋다.

프로그램에서 값을 계속 공유해서 사용해야 하는 경우엔 아래와 같이 사용하면 좋다.

1. 이벤트나 동작과 같이 시점이 중요한 경우 : Subject
2. API 등의 동작 결과값을 계속 공유해야 하는 경우 : BehaviorSubject

## Subject

Subject는 Observable을 상속하고 있다. 이는 Subject가 Observable이자 Observer라는 것을 의미한다.

Observer의 성격을 지니고 있어 Observable을 구독할 수 있다.

동시에 Observable이기 때문에 이벤트를 발행할 수 있다.

그럼. Observable과 subject의 차이는 뭘까?

subject는 multicast 방식으로 여러 개의 Observer에게 이벤트를 발행할 수 있다.
Observable은 unicast 방식으로 하나의 observer에게만 이벤트를 발행한다.

Observable이 여러 번의 구독을 할 수 없다는 것이 아니라 Observer가 구독하게 될 때마다 새로 create되어 발행하게 된다는 것이다. 반면에 Subject는 여러번 구독하여도 같은 값이 공유된다.

## Subject 종류

PublishSubject, BehaviorSubject, ReplaySubject, AsyncSubject 이렇게 4가지 subject가 있다

- **PublishSubject**

    가장 기본적인 Subject이다. 구독을 시작하는 순간부터 이벤트를 옵저버에게 전달하기 때문에, subscribe 이전에 일어난 이벤트를 몰라도 되는 경우에 사용하기 좋다.

    빈 상태로 생성되고, 새로운 element가 subscribe 이후의 event만 subscriber에게 전달된다.

- **BehaviorSubject**

    PublishSubject와 달리 기본값을 가진 채로 생성되고, 초기값 혹은 최신 element 값을 방출한다. observable이 가장 최근에 발행한 항목을 한번 방출하고 그 이후부터 Obsrvable에 의해 방출된 항목들을 옵저버에게 전달한다.

    behavior subjects는 subscribe가 시작됨과 동시에 최근의 element를 방출하기 때문에 subject가 생성되는 당시에 무조건 초기값을 설정해야 한다는 특징이 있다.

    최신 데이터로 화면을 그리는 경우에 유용하다.

- **ReplaySubject**

    정해진 buffer size를 가진 채로 생성되고, 새로운 구독자에게 방출한다.
    최신 이벤트를 버퍼 사이즈에 맞게 저장하고,Observer가 구독할 시 버퍼에 있는 이벤트를 모두 전달한다.
    BehaviorSubject는 가장 최근에 방출된 값을 딱 한번 방출해줬다면 ReplaySubject는 두 개 이상의 이벤트를 저장하고 싶을 때 사용한다. 하지만 초기값을 설정해주지 않기 때문에 아무 항목도 방출하지 않았을 때는 해당 Subject를 구독해도 아무런 값도 방출되지 않는다.

    버퍼의 크기를 직접 지정해주는 만큼 딱 필요한 만큼만 지정해주는 것이 메모리 관리상 중요하다.

- **AsyncSubject**

    completed 이벤트가 전달되기 전까지 어떠한 이벤트도 발생되지 않으며, complete가 되면 가장 최근에 전달된 값을 방출한다.
