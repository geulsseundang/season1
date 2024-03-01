## 💻 개요

`Re-rendering` 은 `리액트` 가 컴포넌트를 새로운 데이터로 업데이트 하는 과정으로, 이 과정이 있어야 에플리케이션을 사용자의 행동(버튼 클릭 등)에 반응하게끔 만들 수 있습니다.

이 글에서는 `re-rendering` 이 발생하는 이유와 불필요한 `re-rendering` 을 막을 수 있는 기본적인 방법 몇가지를 살펴보겠습니다.

## 💻 컴포넌트의 생명 주기

본 내용에 들어가기 앞서 `리액트` 컴포넌트의 생명 주기를 먼저 알아보겠습니다.

`리액트` 컴포넌트의 생명주기는 아래와 같이 크게 세가지 단계로 단순화 할 수 있습니다.

- `Mount`  
  `Mount` 단계에서는 컴포넌트의 인스턴스가 최초로 생성됩니다. 이때, 컴포넌트의 상태의 초기화가 일어나며, `hook` 들이 실행되고 요소들이 `DOM` 에 추가됩니다.

- `Re-render`  
   기존에 존재하고 있던 컴포넌트를 새로운 정보로 업데이트하는 단계입니다.  
  이미 생성된 인스턴스를 재사용하지만, 컴포넌트 내부에 있는 `hook` 들과 로직들은 재실행됩니다.

- `Unmount`  
  `Unmount` 단계는 컴포넌트의 인스턴스와 그와 관련된 모든것들이 제거되는 단계입니다. 이 단계에서 요소가 `DOM` 에서 제거됩니다.

## 💻 `Re-render` 의 원인

`Re-render` 가 발생하는 **직접적인** 원인은 컴포넌트가 소유하고 있는 `state` 값의 변경입니다. 즉 `props` 의 변경여부는 특정상황을 제외하고는 고려대상이 아닙니다.

```javascript
const App = () => {
  const [counter, setCounter] = useState(0);

  return <div onClick={() => setCounter((prev) => prev + 1)}>{counter}</div>;
};
```

위 코드에서, `div` 태그를 클릭하게되면 `setCounter` 가 실행됩니다. `setCounter` 가 실행되면, `counter` 의 값이 변경(`state` 업데이트)되고 그로 인해 `state` 를 소유한 `App` 컴포넌트가 `re-render` 가 됩니다.

`App` 컴포넌트에 여러 컴포넌트가 중첩되어 있는 경우에는 `counter` 의 의존성 여부와 상관없이 하위의 모든 컴포넌트들이 연쇄적으로 `re-render` 됩니다. 다시말해, `re-render` 가 최초로 시작된 컴포넌트를 포함하여 모든 하위의 컴포넌트들이 `re-render` 됩니다.

```javascript
const App = () => {
  const [counter, setCounter] = useState(0);

  return (
    <>
      <div onClick={() => setCounter((prev) => prev + 1)}>{counter}</div>
      <Component1 />
      <Component2 counter={counter} />
    </>
  );
};
```

`App` 컴포넌트에 `Component1`, `Component2` 가 중첩되어 있으며, `Component2` 는 `counter` 를 `props` 로 받고 있습니다. 이때, `setCounter` 를 이용하여 `counter` 를 업데이트하게되면, `counter` 데이터의 사용여부와 관계없이 `Component1`, `Component2` 가 모두 `re-render` 되는것을 알 수 있습니다.

![nested-component-re-render](https://github.com/geulsseundang/season1/assets/35404137/d8eb64f8-e296-4697-a7c0-577997e9adfa)

## 💻 불필요한 `re-render` 막기

아래와 같은 코드가 있다고 가정해보겠습니다.

```javascript
// App
const App = () => {
  const [counter, setCounter] = useState(0);

  return (
    <>
      <div onClick={() => setCounter((prev) => prev + 1)}>{counter}</div>
      <SlowComponent />
    </>
  );
};

// SlowComponent
const SlowComponent = () => {
  console.log("[ARTIFICIALLY SLOW] Rendering");
  let startTime = performance.now();
  while (performance.now() - startTime < 500) {
    // Do nothing for 500 ms to emulate extremely slow code
  }

  return null;
};
```

`SlowComponent` 는 의도적으로 `500ms` 동안 콜 스택을 점유해 다른 코드가 실행되지 안도록 만든 코드입니다. 해당 컴포넌트를 `App` 컴포넌트에 추가한 후, `counter` 를 업데이트하게 되면, `500ms` 동안 `re-render` 가 발생하지 않게됩니다.

`SlowComponent` 는 업데이트되는 `counter` 와 무관하지만, `counter` 를 소유하고 있는 `App` 컴포넌트의 자식 컴포넌트라는 이유로 `re-render` 가 발생합니다. 이를 해결할 수 있는 방법 두가지를 살펴보겠습니다.

### 👨‍💻 `React.memo`

최초로 `re-render` 를 발생시키는 원인은 `state` 의 변경이며, `props` 의 변경은 무관합니다. 하지만, `props` 의 변경을 비교해 자식 컴포넌트로의 `re-render` 의 전파는 막을 수 있습니다.

`React.memo` 를 사용한 고차컴포넌트는 부모로 부터 전달받은 `props` 들을 얕게 비교하며, 만약 하나라도 변경이 되었다면 `re-render` 가 멈추고 그렇지 않으면 계속 전파됩니다.

```javascript
// App
const App = () => {
  const [counter, setCounter] = useState(0);

  return (
    <>
      <div onClick={() => setCounter((prev) => prev + 1)}>{counter}</div>
      <SlowComponent />
    </>
  );
};

// SlowComponent
const SlowComponent = memo(() => {
  console.log("[ARTIFICIALLY SLOW] Rendering");
  let startTime = performance.now();
  while (performance.now() - startTime < 500) {
    // Do nothing for 500 ms to emulate extremely slow code
  }

  return null;
});
```

`React.memo` 로 감싼 `SlowComponent` 는 `props` 가 변하지 않았기 때문에, `counter` 가 업데이트 되어도 `re-render` 되지 않으며 아래로 전파되지도 않습니다.

![React.memo re-render](https://github.com/geulsseundang/season1/assets/35404137/c7f4e949-91fa-4ff3-a600-1f1e95d8ca73)

### 👨‍💻 `State` 분리하여, 아래로 끌어 내리기

`counter` `state` 는 `div` 태그에서만 사용되고 있기때문에, 컴포넌트로 분리해 고립시킬 수 있습니다.

```javascript
// App
const App = () => {
  return (
    <>
      <Counter />
      <SlowComponent />
    </>
  );
};

// Counter
const Counter = () => {
  const [counter, setCounter] = useState(0);

  return <div onClick={() => setCounter((prev) => prev + 1)}>{counter}</div>;
};

// SlowComponent
const SlowComponent = () => {
  console.log("[ARTIFICIALLY SLOW] Rendering");
  let startTime = performance.now();
  while (performance.now() - startTime < 500) {
    // Do nothing for 500 ms to emulate extremely slow code
  }

  return null;
};
```

위 코드는 `Counter` 컴포넌트를 분리하여 `counter` `state` 를 갖도록 만든 코드입니다. 이 경우, `counter` 를 업데이트하게 되면 `Counter` 컴포넌트에서 `re-render` 가 시작되며 하위 컴포넌트들로 전파되게 됩니다. 즉, `SlowComponent` 는 `re-render` 와 무관함으로 `React.memo` 로 감싸지 않아도 됩니다.

![Moving state down](https://github.com/geulsseundang/season1/assets/35404137/679f39b4-afd9-48a7-800e-1b66121068b6)
