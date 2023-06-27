# lesson24. TCP 의 구조

## 1. TCP 란?

- TCP 프로토콜을 사용해 데이터를 전송할 때 붙이는 헤더를 TCP 헤더라고 한다. 그리고 **TCP 헤더가 붙은 데이터를 세그먼트(segment)** 라고 한다.

![Untitled](https://user-images.githubusercontent.com/63203480/236835088-378984d1-b9ce-427b-a4e3-e2fc583e1a01.jpeg)

- TCP 프로토콜을 이용해서 데이터를 전송하기 위해서는 전송 전에 가상의 독점 통신로를 확보하는 연결(Connection) 을 해야 한다. 연결이 된 이후에 데이터를 전송할 수 있다.
- Code Bit 는 TCP 헤더의 107~112비트까지 총 6비트로 **연결의 제어 정보가 기록**되는 곳이다. 초기 값은 0이고 비트가 활성화되면 1이 된다. 이 중에 연결을 확립하기 위해서는 SYN(연결요청) 과 ACK(확인응답) 가 필요하다.

  ![Untitled2](https://user-images.githubusercontent.com/63203480/236835366-91ac5813-7f3d-4ae6-a235-8dffc83ba897.png)


## 2. 3-way handshake 란?

- 데이터를 전송하기 전에 3 번의 패킷전송이 발생한다. 데이터를 보내기 전에 연결을 확립하기 위해서 패킷 요청을 3 번 교환하는 것을 3-way handshake 라고 한다.

  ![Untitled3](https://user-images.githubusercontent.com/63203480/236835516-daf39dda-814e-40b1-afa5-bc1df287e1df.png)

    1. SYN: 통신을 하기 위해서는 컴퓨터 1이 컴퓨터 2에게 허가를 받아야 하기 때문에, 연결 확립 허가를 받기 위해 요청(SYN)을 보낸다.
    2. SYN + ACK: 컴퓨터 2는 컴퓨터 1이 보낸 요청을 받은 후에 허가를 위한 응답을 회신하기 위해서 연결 확립 응답(ACK) 을 보낸다. 동시에 컴퓨터 2도 컴퓨터 1에게 데이터 전송 허가를 받기 위해서는 연결 확립 요청(SYN)을 보낸다.
    3. ACK: 컴퓨터 2의 요청을 받은 컴퓨터 1은 컴퓨터 2로 연결을 허락한다는 응답으로 연결 확립 응답(ACK) 를 보낸다.

    ```
                URG ACK PSH RST SYN FIN
    1. SYN     : 0   0   0   0   1   0
    2. SYN+ACK : 0   1   0   0   1   0
    3. ACK     : 0   1   0   0   0   0 
    ```


## 3. 4-way handshake 란?

- 데이터 전송이 끝났다면 연결을 끊기 위한 요청을 서로 교환해야 한다. 연결을 끊을 때는 FIN 과 ACK 를 사용한다. FIN 은 연결 종료를 뜻한다. 그리고 이것을 4-way handshake 라고 한다.

  ![Untitled](https://user-images.githubusercontent.com/63203480/236835589-8f79ad19-bacf-4e7b-9c24-d84bbab9453b.png)

    1. 컴퓨터 1에서 컴퓨터 2에게 연결 종료 요청(FIN)을 보낸다.
    2. 컴퓨터 2가 컴퓨터 1에게 연결 종료 응답(ACK) 를 반환한다.
    3. 똑같이 컴퓨터 2도 컴퓨터 1에게 연결 종료 요청(FIN)을 보낸다.
    4. 컴퓨터 1이 컴퓨터 2에게 연결 종료 응답(ACK) 를 반환한다.
  ```
              URG ACK PSH RST SYN FIN
  1. FIN     : 0   0   0   0   0   1
  2. ACK     : 0   1   0   0   0   0
  3. FIN     : 0   0   0   0   0   1
  4. ACK     : 0   1   0   0   0   0
  ```
