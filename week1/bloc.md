
# BLoC

BLoC(이하 블록)은 Business Logic Component의 약자로, 비즈니스 로직과 화면 영역의 분리를 목적으로 설계된 디자인 패턴입니다. Flutter에서 주로 사용되긴 하지만, 특정 프레임워크에 국한된 라이브러리 개념이 아닌 디자인 패턴이기 때문에 플랫폼에 상관없이 적용할 수 있는 구조입니다. 다트 컨퍼런스 2018에서 최초 공개되었고, Flutter와 AngularDart가 모든 UI 로직을 공유하는 멋진 예시를 보여주었다고 합니다. 그만큼 UI와 비즈니스 로직을 완벽히 분리하여 테스트를 용이하게 하는 패턴으로 보입니다.

블록에서 위젯은 가능한 수동적이여야하며 비즈니스 로직은 별도의 컴포넌트로 존재해야한다는 것이 블록 패턴의 핵심 개념입니다. 블록은 다음 두 가지를 제공합니다.

- 블록의 공개 API는 간단한 입출력을 사용합니다.
- 블록은 주입할 수 있어야합니다. 즉, 플랫폼 의존성이 없어야 한다는 뜻입니다. 따라서 블록을 플러터와 웹에서 모두 이용할 수 있습니다.

다소 추상적일 수 있지만, 블록에서 추구하는 기본 설계 원칙을 규칙을 보면 어느 정도 이해가 됩니다.

먼저 **앱 설계 규칙**은 다음과 같습니다.

1. 입출력은 오직 싱크와 스트림을 통해 이루어지며 함수, 상수, 변수를 사용하지 않습니다.
2. 반드시 의존성을 주입할 수 있어야 합니다. 
3. 플랫폼 분기는 허용하지 않습니다. 블록 내 `if (device == browser)`와 같은 표현식이 있다면 이는 잘못된 것입니다.

다음으로 **UI 기능 관련 규칙**입니다.

1. 보통 블록과 최상위 수준 플러터 패키지는 일대일 관계를 맺습니다. 
2. 컴포넌트는 위젯의 비즈니스 로직이 아니므로 입력을 있는 그대로 블록에 전송해야 합니다. 텍스트를 포맷하거나 모델을 직렬화하는 등의 작업은 블록 내부에서 수행합니다.
3. 출력은 위젯이 사용할 수 있는 형태로 전달합니다. 예를 들어 숫자에 화폐 단위를 붙어야 하는 상황이라면 이 작업은 블록에서 처리합니다.
4. 모든 분기는 단순 블록 Boolean 로직을 사용합니다. 한 개의 Boolean 스트림만 사용하돌고 블록을 제한해야 합니다. 예를 들어 `bloc.isDestructive ? Colors.red : Colors.blue` 와 같은 코드는 괜찮지만, 여러 개의 or 조건이 붙는 if 문을 사용하는 복잡한 Boolean 문은 권장하지 않습니다. 

여기까지 살펴봤을 때 기존 모바일 아키텍처에서 자주 사용되는 ViewModel의 개념과 크게 다를 바는 없어보입니다.
어떤 플랫폼이든 간에, 클라이언트 단에서 사용하는 아키텍처들의 공통적인 목표는 UI와 비즈니스 로직의 분리라고 생각되며 굳이 차이점을 찾자면 그 것을 어떤 방식으로 혹은 어떤 이름을 붙여가며 구현하느냐의 차이인 것 같습니다.

> **💡 BLoC과 ViewModel**
앞서 ‘앱 설계 규칙’에서 블록 패턴은 입출력을 오직 싱크와 스트림을 통해 이루어진다고 했습니다. 이러한 점을 블록과 뷰모델의 차이점으로 볼 수 있었는데, 사실 해당 규칙은 초기 블록 패턴에 한정되고 현재 시점에서 Best Practice는 아닌 것으로 보입니다.
입출력을 오직 싱크와 스트림을 통해 이루어지도록 했던 이유는 AngularDart 웹앱과 Flutter 모바일 앱 간 코드를 재사용할 수 있도록 하기 위함이었기 때문입니다. 하지만 실제로는 AngularDart 웹앱을 작성하는 일이 드물고, 일반적인 방법(함수/상수/변수를 노출하는 것)보다 덜 편리하기 때문에 이러한 원칙은 최근에 폐기되었습니다. 
현재 공식 BLoC 패키지의 블록은 좋은 VM과 마찬가지로 메서드를 노출합니다. 결과적으로 기존 모바일 아키텍처에서 사용하는 뷰모델 개념과 큰 차이가 없습니다.


이번 글에서는 Flutter-BLoC 라이브러리를 사용하지 않는 순수한 방식의 블록 구현 방식에 대해 알아보고, 다음 글에서는 라이브러리를 이용하여 좀 더 추상화된 방식으로 블록을 구현해보겠습니다.

블록은 크게 두 가지 역할을 수행합니다. 블록은 위젯이 상태를 갱신할 수 있도록 스트림을 제공하고 위젯으로 하여금 다시 화면을 그리도록 지시합니다. 
따라서 블록을 구현하기 위해서는 Stream의 개념에 대해 먼저 알아야합니다.

# Stream

Stream(이하 스트림)은 다트 프로그래밍에서 큰 비중을 차지합니다. 다트에서 스트림은 객체이면서 동시에 비동기 프로그래밍 개념을 지칭합니다. 많은 언어에서 이러한 스트림을 `Observable`이라고 표현하곤 합니다. 디자인 패턴을 공부하며 옵저버 패턴을 학습한 적이 있다면(혹은 Rx 라이브러리를 사용해본 적이 있다면) 이러한 스트림 개념이 낯설지 않을 것입니다. 
스트림은 새로운 값을 반복적으로 방출하며 다른 객체는 이러한 스트림을 ‘듣고’(*listen*) 있다가 스트림을 통해 흘러온 값을 얻습니다. 객체는 흘러들어온 값을 확인하고 그에 적합한 작업을 수행합니다. 
블록에 스트림 개념을 적용한다면, 여기서 말하는 객체는 위젯이 될 것이고 값에 맞는 적합한 작업은 UI 업데이트가 될 것입니다.

장바구니에 항목을 담을 때마다 장바구니 아이콘의 숫자를 바꾸는 기능을 스트림으로 구현해보겠습니다.

# BLoC과 Stream을 통해 장바구니 앱 구현하기

블록의 기본은 입출력이 무엇인지를 정의하는 것입니다. CartBloc의 입출력 API는 다음과 같습니다.

- 입력
    - 장바구니에 항목 추가
    - 장바구니에서 항목 제거
- 출력
    - 장바구니의 모든 항목 얻기
    - 장바구니의 항목 개수 얻기

## 입력 - 장바구니에 항목 추가하기

```dart
CartBloc.addProductSink.add(Product item, int qty)
```

위와 같은 형태로 장바구니에 아이템을 추가할 수 있도록 블록을 구현해보겠습니다.

**1. CartBloc.addProductSink를 구현합니다.**
싱크는 StreamController의 형태입니다. 싱크에 add를 호출하면 스트림으로 데이터가 추가됩니다.
    
 ```dart
class CartBloc {
    StreamController<AddToCartEvent> addProductSink = StreamController<AddToCartEvent>()
}
```
    
**2. addProductSink로부터 스트림을 듣습니다.**
StreamController.stream은 다른 객체가 들을 수 있는 스트림 객체입니다. 스트림의 listen 메서드는 콜백을 받으며 스트림 컨트롤러로 새 값이 전달되면 이를 호출합니다.
    
```dart
CartBloc(this._service) {
    addProductSink.stream.listen(
        (_handleAddItemsToCart)
    )
}
```
    
**3. 스트림으로부터 데이터가 흘러들어오면, 데이터베이스의 장바구니에 제품을 추가하는 서비스를 호출합니다.**
    
```dart
void _handleAddItemsToCart(AddToCartEvent e) {
    _service.addToCart(e.product, e.qty);
}
```
    

위젯 파일에서는 다음과 같이 제품을 스트림으로 추가합니다.

```dart
void _addToCart(Product product, int quantity, CartBloc _bloc) {
    _bloc.addProductSink.add(
        AddToCartEvent(product, quantity),
    )
}
```

## 출력 - 장바구니 항목 제거하기

사용자가 장바구니에 담긴 제품의 개수를 줄이고자 특정 버튼을 누르면 이는 입력 이벤트로 들어오고, 이 입력 이벤트에 맞게 데이터베이스에서 값을 감소시켜야하므로 적절한 백엔드 API를 호출할 것입니다. 반응형으로 값 변화를 클라이언트단에 통지하는 파이어스토어를 이용한다고 가정하고, 이러한 실시간 값 변화를 화면에 반영하는 출력 과정을 나타내보겠습니다. 

`사용자가 감소 버튼 터치함` -> `백엔드 API 호출하여 개수 줄임` -> `백엔드 DB 값 변화를 반응형으로 감지` -> `화면에 반영`

출력 이벤트를 처리하는 3, 4번째 과정을 코드로 다음과 같이 표현할 수 있습니다.

**1. 필요한 객체의 인스턴스를 만듭니다.**
```dart
StreamController _cartItemCountStreamController = BehaviorSubject<int>(seedValue: 0)
Stream<int> get cartItemCount => _cartItemCountStreamController.stream;
```

**2. 외부 소스로부터 스트림 컨트롤러로 데이터를 추가합니다. CartBloc 클래스의 생성자에서 이를 설정합니다.**
```dart
CartBloc(this._service) {
    _service
        .streamCartCount()
        .listen((int count) =>
        _cartItemCountStreamController.add(count))
}
```

## 고차 스트림(higher-order stream)을 이용한 입출력 구조 
함수를 인자로 받거나 결과를 반환하는 함수인 `고차 함수`의 개념과 유사하게, 고차 스트림은 스트림을 받아서 또 다른 형태의 스트림으로 반환합니다. 고차 스트림을 이용하여 카탈로그의 모든 제품을 보여주고, 새 제품을 추가하면 갱신된 제품 리스트를 보여주는 코드를 작성해보겠습니다.

```dart
class CatalogBloc {
    StreamController _productStreamController = StreamController<List<Product>>();
    StreamController<List<Product>>();
    Stream<List<Product>> get allProducts => _productStreamController.stream;

    // ...

    final _productInputController = StreamController<ProductEvent>.broadcast();

    // ...

    CatalogBloc(this._service) {
        _productInputController.stream
            .where((ProductEvent event) => event is UpdateProductEvent)
            .listen(_handleProductUpdate);

        _productInputController.stream
            .where((ProductEvent event) => event is AddProductEvent)
            .listen(_handleAddProduct);
    }
}
```

사용자 입력을 ProductEvent라는 클래스에 정의하고, 이벤트의 종류가 어떤 것인지 where 스트림으로 판별하고 그에 맞는 이벤트가 들어오면 정의해둔 handler 함수를 호출합니다. handler 함수에는 들어온 값에 맞게 화면을 업데이트 하는 코드가 들어가면 됩니다. 

여기까지 Flutter에서 사용하는 블록 패턴의 개념과 Dart의 Stream API를 이용한 간단한 구현 방식에 대해 알아보았습니다. 다음 글에서는 이러한 블록 패턴을 라이브러리화한 `Flutter_bloc` 라이브러리에 대해 알아보겠습니다.
