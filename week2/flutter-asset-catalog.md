모바일 앱 개발을 하면, 한번 쯤 고민해보게 되는 항목이 있습니다.
바로 "Asset을 어떻게 관리할까?"에 대한 고민인데요.

Asset은 폰트, 색상, 이미지와 같이 컴포넌트를 구성하기 위해 필요한 디자인적 요소를 뜻합니다.

아름다운 앱을 만들기 위해서는 이러한 Asset들을 잘 활용하는 것이 필수적입니다.
디자이너와 협업을 한다면, 디자인 프로토타이핑 툴(Figma, Sketch, Zeplin)을 사용해서 소통을 합니다.


보통 디자인 시스템을 미리 디자이너가 정의를 하고, 이에 맞춰 앱을 구상하게 되는데
우리는 여기서 이러한 디자인 시스템을 어떻게 앱에 잘 녹여낼지에 대한 고민을 한번 쯤 하게 됩니다.


그래서 오늘, Flutter 엔진에서 이러한 Asset들을 관리하는지에 대해서 이야기 하고자 합니다.


### 1. Asset Catalog의 필요성

Asset Catalog는 디자이너와 개발자의 소통을 도와줍니다.
디자인 산출물을 바탕으로, 개발 환경에서 어떻게 구성되어 있어 있는지, 그리고 요구사항에 맞춰 동작해야하는지 등을 Asset Catalog를 통해 확인할 수 있습니다.

뿐만 아니라, 작업자가 여려 명일 경우 중복 코드를 예방할 수 있고 유지 보수도 가능하게 합니다.

이처럼 우리들은 Asset Catalog를 통해 작업자 간 소통할 수 있는 하나의 수단으로서 활용할 수 있기 때문에 되도록, 많은 작업자가 투입된 프로젝트에서는 만드는 것이 효율적이라고 생각합니다.


### 2. 디자인 산출물 이해하기

디자이너가 제시하는 디자인 산출물은 다음과 같습니다.

```
- 컬러 시스템 (light, dark)
- 컴포넌트 (inputBox, button, etc)
```

이러한 디자인 산출물은, 요구사항을 잘 이해하고 그에 맞춰 개발 하는 것이 중요합니다.
간단하게 확인 해야하는 요소들을 정리해보면 다음과 같습니다.

```
1) 컬러
- os 또는 서비스 테마 여부

2) 컴포넌트
- status(enable, disable, pressed ..) 여부
- 애니메이션 (touch, init motion ..)
- 조건에 따른 동작 여부
```

이를 바탕으로 우리는 프로젝트에 어떻게 녹여낼 것인지 고민하여 설계가 필요합니다.

최근 이러한 요구사항들을 잘 분석하지 않아서 서비스 중 유지보수 하는데 있어 많은 비용이 발생한 경험을 하였습니다.

컴포넌트 특성상 서비스 전반에서 공통적으로 사용되는 요소이기 때문에, 하나의 문제가 발생하였을 때 수정 시 영향도가 큽니다.
따라서 아무리 간단한 것이라도 요구사항을 잘 분석하고, 이를 설계하는 것이 중요합니다.



### 3. 구조 설계


구조 설계에 앞서, Flutter의 Widget에 대해 알아야 합니다.

```
Flutter Widget

1. 화면 구성은 Widget의 조합으로 구성
2. State 유무에 따라 build 메서드를 통해 Widget update
```

#### 1) 단순한 컴포넌트

단순한 컴포넌트의 경우, 객체의 생성을 단순화 하는 `Builder` 패턴을 이용하여 설계할 수 있습니다.

ListView의 `ListTile`을 예시로 설명하면 다음과 같습니다.


```dart

class ListTileBuilder {

  String? title;
  String? subTitle;
  Widget? leading;
  Widget? trailing;
  void Function() onTap;

  bool enabled = true;
  bool selected = false;
  Color? tileColor;

  ListTileBuilder(this.onTap);

  ListTile card() {

    return ListTile(
      title: title != null ? Text(title ?? "") : null,
      titleTextStyle: const TextStyle(fontSize: 14, color: Colors.black),
      subtitle: subTitle != null ? Text(subTitle ?? "") : null,
      subtitleTextStyle: const TextStyle(fontSize: 10, color: Colors.blueAccent),

      leading: leading,
      trailing: trailing,

      onTap: onTap,
      enabled: enabled,
      selected: selected,

      tileColor: tileColor,
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(20)),
      style: ListTileStyle.list,
    );
  }
}

```

```dart

@override
Widget build(BuildContext context) { 
    return SizedBox(
        height: 400,
        child: ListView.builder(
            itemCount: 5,
            itemBuilder: (context, index) {
                final widget = ListTileBuilder(
                    () {
                        print("onTap:: $index 번째");
                    })
                    ..title = "$index 번째 타일"
                    ..tileColor = Colors.blue[(index+1) * 100];

                return widget.card();
            }
      )
    );
}
```

`ListViewBuilder`  class
  - ListTile Widget을 생성해주는 클래스
  - `card()` 메서드를 통해 정해진 style 의 widget을 반환
  

여기서 `..title` , `..tileColor`와 같은 Dart 언어의 chaining을 통해 속성을 바로 접근하여 값을 변경해주었습니다.

이러한 Dart 언어를 이용하여, 자주 변경이 일어나는 값(enabled, Color 등)들을 사용하는 외부에서 주입하여 사용하도록 설계 할 수 있습니다.


---


#### 2) 상태를 가진 컴포넌트

사용자의 액션에 따라 컴포넌트 자체에서 상태를 갖는 경우도 있습니다.
예를 들어 button이나 inputBox가 그에 해당하는데요, 사용자의 액션에 따라 상태를 갖는 컴포넌트를 설계할 때 어떻게 고려하면 좋을까요?


먼저 컴포넌트의 상태를 정의하고, 이를 외부로 알려줄 수 있는 구조가 필요합니다.


그렇게 설계 하기 위해 컴포넌트의 상태를 관리하는 클래스가 별도로 존재 해야 합니다.

`GetX` 라는 Flutter 서드파티 프레임워크는, 상태를 관리하기 위해 TextfieldController와 같은 클래스에 Rx를 래핑해놓은 클래스를 별도로 구현해서 제시하기도 했습니다.


그렇다면, `GetX` 말고 다른 방법으로 구현할 수는 없을까요?

`flutter_bloc`을 사용하여 구현해 보면 다음과 같습니다.


```dart
class TextWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocBuilder<TextFieldBloc, TextFieldState>(
      builder: (context, state) {
        return Text(
          '현재 텍스트: ${state.text}',
          style: TextStyle(fontSize: 16),
        );
      },
    );
  }
}

class TextFieldComponent extends StatelessWidget {

  final TextEditingController _textEditingController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    context.read<TextFieldBloc>().stream.listen((event) {
      _textEditingController.value = TextEditingValue(
          text: event.text,
          selection: TextSelection.fromPosition(TextPosition(offset: event.cursorPosition))
      );
    });

    return BlocBuilder<TextFieldBloc, TextFieldState>(
      builder: (context, state) {
        return TextField(
          onChanged: (text) {
            context.read<TextFieldBloc>().add(ValueChangeEvent(text));
          },
          onTap: () {
            context.read<TextFieldBloc>().add(EditingDidBeginEvent());
          },
          onEditingComplete: () {
            context.read<TextFieldBloc>().add(EditingCompleteEvent());
          },
          onSubmitted: (text) {
            context.read<TextFieldBloc>().add(EditingDoneEvent());
          },
          decoration: InputDecoration(
            labelText: '텍스트 입력',
          ),
          controller: _textEditingController,
          enabled: true, // 필요한 경우 상태에 따라 동적으로 설정
        );
      },
    );
  }
}

```

```dart
import 'package:flutter_bloc/flutter_bloc.dart';

abstract class TextFieldEvent {}

class ValueChangeEvent extends TextFieldEvent {
  final String newValue;

  ValueChangeEvent(this.newValue);
}

class EditingDidBeginEvent extends TextFieldEvent {}

class EditingCompleteEvent extends TextFieldEvent {}

class EditingDoneEvent extends TextFieldEvent {}

class TextFieldState {
  final String text;
  final bool isFocused;
  final int cursorPosition;

  TextFieldState(this.text, this.isFocused, this.cursorPosition);
}

class TextFieldBloc extends Bloc<TextFieldEvent, TextFieldState> {
  TextFieldBloc() : super(TextFieldState('', false, 0));

  @override
  Stream<TextFieldState> mapEventToState(TextFieldEvent event) async* {
    if (event is ValueChangeEvent) {
      yield TextFieldState(event.newValue, state.isFocused, event.newValue.length);
      print("ValueChangeEvent ${event.newValue}");
    } else if (event is EditingDidBeginEvent) {
      yield TextFieldState(state.text, true, state.cursorPosition);
      print("EditingDidBeginEvent");
    } else if (event is EditingCompleteEvent) {
      yield TextFieldState(state.text, false, state.cursorPosition);
      print("EditingCompleteEvent");
    } else if (event is EditingDoneEvent) {
      // 여기에서 텍스트 필드의 작업이 완료된 상태에 대한 추가 로직을 수행할 수 있습니다.
      print("EditingDoneEvent");
      yield state;
    }
  }
}
```


`Bloc` 패턴의 가장 큰 특징인 Input(Event), Output(State)를 두어 관리하는 것인데요.

이러한 input/output 구조를 이용하면 사용자에게 받을 입력 값과 출력 값들을 서로 독립적으로 분리 할 수 있다는 장점이 있어, 상태를 가지는 컴포넌트를 설계하고자 하였을 때, 유용하게 사용될 수 있습니다.

TextField 라는 위젯의 콜백들을 bloc의 event를 통해 이 위젯을 사용하고 있는 클래스에서는, 단순히 state(output)만 구독하여 알아낼 수 있습니다.

---



#### 3) 서비스 전반에서 사용하는 상수 값 관리

디자인 시스템을 보면 폰트나 그림자, 그리고 색상 값 등 서비스에 사용될 스타일들을 미리 정의된 장표를 본 적이 있을 겁니다.

이렇게 서비스에서 사용되는 상수 값들은, 정적 변수로서 서비스 메모리에 영향을 주기도 하는데요,
물론 성능에 영향이 미칠 만큼 많은 데이터 값이 아니기 때문에 크게 성능과는 상관이 없지만 협업의 관점에서는 어떨까요?

먼저 Figma를 예시로 컬러 시스템을 살펴보면, 아래와 같이 트리 구조로 되어 있는 것을 확인할 수 있습니다.

```
Libraries
- main 
  ㄴ primary 
     ㄴ button
     ㄴ text 
  ㄴ gray
     ㄴ border
     ㄴ shadow
       ..
```

이렇게 트리 구조로 되어 있는 경우, Native로 Color Asset 등록 시 어떻게 해야할까요?

먼저, 하나의 트리(노드)를 기준으로 `_`과 같은 구분자를 넣어 Asset 이름을 구상하는 것도 방법이 되겠습니다.
하지만 이렇게 할 경우 디자이너와 협업 시 또는 개발자가 디자인 산출물을 기준으로 개발 시 컬러 값 하나를 확인하는데 여러 번 값을 확인해가며, 찾아야한다는 수고를 감수 해야한다는 단점이 있습니다.


그러면, 최대한 디자이너와의 시각을 줄이면서 코드화 하는 방법으로는 어떤 것이 있을까요?


가장 좋은 방법은 트리의 구조를 기반으로 Class를 나눠 관리하는 것입니다.

Dart 언어는 객체 지향 기반 언어입니다. 따라서, 객체 지향 프로그래밍의 특징들이 잘 녹여져 있는 것을 확인할 수 있는데요,

이러한 특징을 살려 코드 레벨로 구현해보면 다음과 같습니다.

```dart

class Colors { 
    static const primary = Primary();
    static const gray = Gray();
}

class Primary { 
    static const text = Colors.fromRGB(1, 1, 1, 1);
    ...
}

class Gray {
    static const shadow = Colors.fromRGB(1, 0.4, 0.3, 0.2);
    ...
}


void main { 
    const colors = Colors();
    print(colors.primary.text);
}
```


어떠신가요? Figma에 정의되어 있는 라이브러리 구조와 동일하지 않나요?
Swift로 만약 구현한다면 `struct` 자료구조를 이용하여 구성할 수 있지만, Dart의 class를 활용해도 충분히 그와 비슷한 구조로 구현해낼 수 있다는 것을 확인 할 수 있었습니다.

하지만, 단점이 있습니다.

Class는 생성자를 통해 인스턴스 생성 후 `static 변수`를 통해 접근하여 사용하기 때문에, 런타임 단에서 그 값이 결정 됩니다.

따라서 주로 Color 값이 필요한 Widget에서 `const` 키워드를 더 이상 사용할 수 없게 되는 것이죠.

> "const 키워드를 사용하지 못하는게 뭐 별거야?, 상수 키워드 final도 있잖아!"

라고 생각을 하신 분들이 분명 있을 겁니다.

Flutter 엔진의 특성상 re-build 시 상태 값이 변경된 위젯에서부터 그 하위 위젯 트리까지 모두 다시 그리는 구조를 택하고 있어 불필요하게 계속 위젯을 그리는 경우가 발생할 수 있습니다.

이러한 경우를 회피하기 위해서는, 우리는 적절히 상수 키워드를 사용하면 되는데,
`const` 사용 시 컴파일 단에서 이미 메모리에 적재가 되어 사용되기 때문에 리소스 낭비를 막을 수 있습니다.

따라서 성능이 크게 중요한 앱이라면, 클래스 구조로 색상 값들을 관리하는 것보다 정적 변수로서 값을 관리하는 것이 더 좋지 않을까 하는 생각이 듭니다.


#### 4) 기본 컴포넌트 확장하기

사실, 별도의 클래스를 두고 관리한다는 것은 프로젝트 규모가 크지 않는 이상 관리 포인트가 늘어난다는 점에서 불편하다고 생각하시는 분들도 계실 겁니다.


그렇기 때문에, material 에서 제공하는 컴포넌트를 확장 & 조합 하여 사용하는 방법에 대해서 다뤄보겠습니다.


먼저 스타일입니다. Flutter의 대부분 위젯에서는 `style` 속성 값을 사용합니다.

이러한 `style` 속성은 어쩌면 반복된 코드를 계속해서 붙여 넣어줘야 해서, 개발자에게 피로감과 실수를 발생시킬 수 있습니다.


저도 이런 귀차니즘이 매우 심해서.. ㅋㅋ 이 부분을 "어떻게 하면 해결할 수 있을까?"에 대한 고민을 한 끝에 고안해낸 방법이 `mixin`  키워드를 활용하여 개발하는 방법입니다.

```dart
mixin TextFieldDesign {

  TextStyle? get textStyle => const TextStyle(
    color: Colors.deepPurple
  );

  InputDecoration? inputDecoration(int? inputLength, int? maxLength) {
    return InputDecoration(
        isDense: true,
        contentPadding: const EdgeInsets.symmetric(vertical: 12, horizontal: 16),
        fillColor: Colors.grey,
        border: const OutlineInputBorder(
          borderRadius: BorderRadius.all(Radius.circular(22)),
        ),
        enabledBorder: const OutlineInputBorder(
          borderRadius: BorderRadius.all(Radius.circular(22)),
        ),
        disabledBorder: InputBorder.none,
        suffix: maxLength == null ? null : Text.rich(
            TextSpan(
                text: "$inputLength",
                style: const TextStyle(fontWeight: FontWeight.bold),
                children: <TextSpan>[
                  const TextSpan(text: "/", style: TextStyle(fontWeight: FontWeight.normal)),
                  TextSpan(text: "$maxLength", style: const TextStyle(fontWeight: FontWeight.normal))
                ]
            )
        ),
        counter: null,
        counterText: ""
    );
  }

}
```


위 코드는, Textfield의 style 부분을 정의해 놓은 `mixin class`입니다. textstyle이나 decorator 등을 하나의 mixin 클래스에서 정의한 것인데, 

이렇게 스타일만 별도로 분리해 놓은 클래스만 존재한다면, 만약 디자인이 변경 됐을 때 이 파일만 열고, 수정하여 대응할 수 있습니다.

만약 이러한 구조가 되어 있지 않으면.. 하나씩 `Command+F` 권법을 사용해서 바꿔줘야겠지요..?ㅠ
(생각만해도 암울하고 벌써 지루합니다.)

스타일을 간단하게 정의하고 관리하는 것은 알겠으나, 사실 어려운 mixin 개념으로 안하고 static 이라는 단순한 개념으로도 해결 할 수 있습니다.

여기서 제가 mixin 키워들을 굳이 꼽아서 설명한 이유는, 개발을 하다보니 아래와 같은 생각이 들었기 때문입니다.

> 스타일만 정의해 놓은 클래스가 있다. O.K, 근데 매번 정의하면서 사용하기 귀찮아! 


이런 요구사항이 생겼을 때 .. 어떻게 해결하실 건가요?

mixin 키워드를 사용하면 굉장히 간단하게 끝낼 수 있습니다.

```dart
class TextFieldView extends TextField with TextFieldDesign {

  const TextFieldView({
    Key? key,
    TextEditingController? controller,
    int? maxLength
  }) : super(
      key: key,
      controller: controller,
      maxLength: maxLength
  );

  @override
  TextStyle? get style => textStyle;

  @override
  InputDecoration? get decoration => inputDecoration(controller?.text.length ?? 0, maxLength);

}
```


하나의 컴포넌트를 만들 때 어떤 디자인을 채용할 것인지, 클래스의 헤더 부분에 정의하면 됩니다. 참 쉽죠?

이렇게 정의하여 사용한다면 누구나 쉽게 "아~, 이곳에서는 이런 스타일을 사용 중이구나"를 쉽고 빠르게 이해할 수 있을 것 입니다. 굳이 빌드를 하지 않아도요.

다트에서는 OOP 기반의 여러 가지 기법들을 개발자 스스로가 자유도를 가지고 사용 할 수 있게끔 유도합니다.
이러한 점이 Swift 개발자의 입장으로, 한 편으로는 편했지만 정말 불편하다고 생각이 든 순간들이 많았습니다. 특히 `enum`과 `struct`는 Swift가 정말 고민 많이하고 만들어진 오픈소스 프로젝트라는 것을 느끼게 해줬습니다.


#### 4. 마치며

이야기가 많이 길어졌지만, 초기 서비스를 개발하면서 컴포넌트 & 디자인 요소 갖추기는 꼭 필요한 작업입니다.

누구는 단순 반복 느낌의 작업이라 기피할 수 있겠지만, 어쩌면 Framework(또는 OS)에서 추구하는 Guide와 UI rendering에 대해서 깊게 탐구 해볼 수 있는 좋은 기회라고 생각합니다.


로직을 단순히 잘 짠다고 해서, 화면 단의 움직임에 대한 이해가 없으면 잘 짠 로직을 표현할 수 있는 방법이 없을 겁니다.
사용자에게 부드러운 UI와 경험을 주는 것이 모바일 엔지니어로서 계속 고민해야할 부분이 아닐까 싶습니다.


개인적으로 플러터를 사용하면서 OOP에 대한 개념들을 다시 살펴 볼 수 있어서 좋았는데요.
학창 시절에나 OOP~ OOP~ 하면서 달달 외운 것들이 다 였는데, 이렇게 실전으로 디자인 패턴도 하나씩 녹여가면서 공부할 수 있는 시간이여서 보람찼던 것 같습니다.

