# 아이템28. 유효한 상태만 표현하는 타입을 지향하기

```tsx
function renderPage(state: State) {
  if (state.error) {
    return 'Error';
  } else if (state.isLoading) {
    return 'Loading';
  }
  return `Hi`;
}
```

코드를 보면 분기조건이 명확하게 분리되어 있지 않다는 것을 알 수 있다. `isLoading` 이 `true` 이며 동시에 `error` 가 있을 경우 오류 상태인지 로딩 중인 상태인지 명확히 구분할 수 없다.

```tsx
async function changePage(state: State, newPage: string) {
  state.isLoading = true;
  try  {
    const response = await fetch(getUrlForPage(newPage));
    if (!response.ok) {
      throw new Error('Unable to load');
    }
  } catch (e) {
    state.error = e;
  }
}
```

페이지를 전환하는 함수인 `changePage` 함수에는 더 많은 문제가 있다.

오류가 발생했을 때 `state.isLoading` 을 `false` 로 설정하는 로직이 빠져있다. `state.error` 를 초기화 하지 않았기 때문에 페이지 전환 중 로딩 메시지 대신에 과거의 오류 메시지를 보여준다. 그리고 페이지 로딩 중에 사용자가 화면을 바꾸면 어떤 일이 발생할지 예상하기 어렵다.

현재 `State` 타입은 `isLoading` 이 `true` 이면서 `error` 값이 설정되는 무효한 상태를 허용하고 있다. 무효한 상태가 존재하면 `renderPage()` 와 `changePage()` 를 둘 다 제대로 구현할 수 없다.

아래는 State 를 조금 더 자세히 표현하는 방법이다.

```tsx
interface RequestPending {
  state: 'pending';
}

interface RequestError {
  state: 'error';
  error: string;
}

interface RequestSuccess {
  state: 'ok';
  pageText: string;
}
type RequestState = RequestPending | RequestError | RequestSuccess;

interface State {
  currentPage: string;
  requests: {[page: string]: RequestState};
}
```

네트워크 요청 과정의 각각의 상태를 명시적으로 모델링한 **Tagged Union** 이 사용되었다.

```tsx
function renderPage(state: State) {
  const { currentPage } = state;
  const requestState = state.requests[currentPage];
  switch (requestState.state) {
    case "pending":
      return 'Loading';
    case "error":
      return 'Error';
    case "ok":
      return 'Hi';
  }
}

async function changePage(state: State, newPage: string) {
  state.requests[newPage] = { state: 'pending' };
  state.currentPage = newPage;
  try  {
    const response = await fetch(getUrlForPage(newPage));
    if (!response.ok) {
      throw new Error('Unable to load');
    }
    const pageText = await response.text();
    state.requests[newPage] = { state: 'ok', pageText };
  } catch (e) {
    state.requests[newPage] = { state: 'error', error: '' + e }
  }
}
```

기존 renderpage와 changePage의 모호함은 완전히 사라졌다.

타입을 설계할 때는 어떤 값들을 포함하고 어떤 값들을 제외할지 신중하게 생갹해야 한다. 유효한 상태를 표현하는 값만 허용한다면 코드를 작성하기 쉬워지고 타입 체크가 용이해진다. **유효한 상태만 허용**하는 것은 매우 일반적인 원칙이다.

이처럼 유효한 상태와 무효한 상태를 모두 포함하는 타입은 혼란을 초래하며 오류를 유발하게 된다. 따라서 유효한 상태만 표현하는 타입을 지향해야 한다. **코드가 길어지고, 표현하기 어려울 수도 있지만 결국은 시간을 절약하고 고통을 줄일 수 있다**.
