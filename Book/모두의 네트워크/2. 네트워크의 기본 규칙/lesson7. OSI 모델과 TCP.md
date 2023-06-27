# lesson7. OSI 모델과 TCP/IP 모델

## 1. OSI 모델

- 옛날에는 A 사의 컴퓨터와 B 사의 컴퓨터가 통신을 할 수 없었다. 그리고 케이블을 연결하는 커넥터도 다르기 때문에 공통으로 사용할 수 있는 표준 규격을 정해야 했다.
- ISO 라는 국제표준화기구에서 OSI 모델이라는 표준규격을 제정했다.
- 컴퓨터간 데이터를 송수신할 때 컴퓨터 내부에서는 일곱 개의 계층(Layer)이 여러 가지의 일을 한다.
- 일곱개의 계층은 `(상위)Application → Presentation → Session → Transport → Network → Data Link → Physical(하위) Layer` 로 나눠져있다. 통신 할 때는 Application Layer 에서부터 순차적으로 아래 계층으로 전달된다.

![Untitled](https://user-images.githubusercontent.com/63203480/235929408-5e7759f9-1625-49ba-9b10-e1a99223aad5.png)

- 데이터를 전송하는 송신측에서는 상위 계층에서 하위 계층으로 데이터를 전달한다. 각 계층은 독립적이기 때문에 데이터가 전달되는 동안 다른 계층의 영향을 받지 않는다. 데이터를 받는 수신 측에서는 하위 계층에서 상위 계층으로 전달된 데이터를 받게 된다.

## 2. TCP/IP 모델

- 4 개의 계층으로 나누어져있다. TCP/IP 모델에서는 Presentation, Session Layer 를 Application Layer 에 포함되어 있다.

![Untitled](https://user-images.githubusercontent.com/63203480/235929539-4fc2db0b-7127-4277-9578-ca4045b55aeb.jpeg)
