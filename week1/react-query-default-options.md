## 💻 개요

`React Query` 는 현재 서버 상태(`Server State`) 를 관리하기 위한 라이브러리로 많이 사용되고 있습니다.  
이 글에서는 `React Query` 가 무엇이며, 사용하기 위해 기본으로 가지고 있는 설정값과 알아야할 용어들에 대해 살펴보겠습니다.

## 💻 React Query 란?

`React Query` 는 비동기 상태 관리자(`Async Data Manager`) 입니다.  
비동기 상태를 `Promise` 형태로 반환하는 함수를 전달하기만 하면, 어떤 형태의 비동기 상태든 관리할 수 있습니다. `React Query` 로 데이터 페칭(`Data Fetching`) 을 수행할 수 있는 것도 이러한 이유 때문입니다. `Axios` 나 `fetch` 와 같은 함수는 모두 데이터를 `Promise` 를 형태로 반환하기 때문에 `React Query` 와 함께 사용할 수 있습니다. 또한 데이터 페칭에서의 비동기 상태를 서버 상태(`Server State`) 라고 합니다.

## 💻 서버상태(`Server State`) 란?

서버 상태(`Server State`) 는 `Client` 가 소유하고 있지 않은 데이터를 의미하며 다음과 같은 특성을 지닙니다.

- `Client` 와 다른 곳에 위치하고 있어, 소유하거나 제어하지 못함
- 데이터를 가져오거나 업데이트를 하기위해서 비동기적으로 `API` 를 호출해야함
- 소유권이 없기때문에 여러사람이 업데이트를 할 수 있으며, 업데이트를 인지할 수 없음
- 따라서 현재 `Client` 에서 사용하고 있는 서버 상태는 최신이 아닐 수 (`out of date`) 있음

유튜브의 인기 영상을 예시로 들어보겠습니다.  
인기 영상의 경우 계속해서 댓글이 추가되기 때문에 특정 시점에 불러온 영상의 정보들은 최신이 아니게 됩니다.
따라서 서버 상태와의 효율적인 동기화가 필요한데, `React Query` 는 동기화(`synchronize`) 를 위한 다양한 수단들을 제공합니다.

## 💻 Stale vs Inactive

### 👨‍💻 Stale

`Stale` 의 사전적의미는 `신선하지 않은`, `탁한` 으로, `React Query` 에서 `stale` 은 데이터가 최신이 아님을 의미합니다. `React Query` 는 기본적으로 `fetch` 가 완료 되자마자 데이터들을 `fresh` 상태에서 `stale` 상태로 설정합니다.

`Stale` 상태의 데이터들은 아래와 같은 상황에서 `refetch` 를 통해 데이터를 다시 불러옵니다.

- 새로운 `query` 인스턴스가 마운트 되었을 때
- `Window` 에 `focus` 이벤트가 트리거 되었을 때
- `Network` 가 재연결 되었을 때
- `Query` 에 `refetchInterval` 옵션이 설정되었을 때

> 위 상황은, 각각 `refetchOnMount`, `refetchOnWindowFocus`, `refetchOnReconnect`, `refetchInterval` 옵션을 설정해 제어할 수 있습니다.

`Stale` 상태에 있는 데이터가 `refetch` 될때, 우선 메모리에 있는 데이터를 (`cached data`) 보여준 후 `refetch` 가 완료되었을 때, 새로운 데이터로 변경합니다.

`StaleTime` 을 통해, `fresh` 상태에서 `stale` 상태로 변경되는 시점을 설정할 수 있습니다. `staleTime` 은 `0` ms 가 기본값입니다. `fetch` 되자마자 `stale` 상태로 변경되는 것도 이 때문입니다.

```typescript
useQuery({
  queryFn: () => {},
  queryKey: [],
  staleTime: 1000 * 30,
});
```

위와 같이 `staleTime` 을 `10` 초로 설정하면, `fetch` 가 완료된 후 `10` 초 후에 `fresh` 상태에서 `stale` 상태로 변경됩니다. `fresh` 상태의 데이터는 항상 캐싱된 데이터에서만 값을 가져오며, 추가적인 네트워크 요청을 보내지 않습니다.

### 👨‍💻 Inactive

`Query` 의 결과가 사용되지 않거나, 활성화된 `query observer` 의 인스턴스를 갖지 않을 경우, `inactive` 상태가 되며, 추후에 사용할것을 대비해 일정 시간동안 캐시에 저장됩니다. 만약 일정 시간동안 사용되지 않을 경우, 저장된 데이터는 `garbage collector` 에 의해 사라지게 됩니다.  
위에서 언급한 `일정 시간` 은 `gcTime` 으로 설정할 수 있으며, 기본값은 `5` 분 (1000 \* 60 \* 5) 입니다.

## 💻 Retry

`Query` 가 실패하면, 기본적으로 3번의 재시도가 발생하게 되며, 매 시도마다 2배의 지연시간이 생기게 됩니다.  
이 동작은 `retry`, `retryDelay` 를 통해 재설정할 수 있습니다.