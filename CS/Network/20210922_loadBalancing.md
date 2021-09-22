## Load Balancing 
부하 분산 또는 로드 밸런싱이라고 하며, 컴퓨터 네트워크 기술 중 하나로, 
둘 혹은 셋 이상의 CPU 나 저장장치와 같은 컴퓨터의 자원들에게 일을 나누는 것을 의미한다.   

![1](https://user-images.githubusercontent.com/63203480/134292957-6be1d020-8d1d-48ce-8e1a-fcdd13d1cd05.png)

요즘 시대에는 웹사이트에 접속하는 인원이 급격하게 늘고 있다.   
따라서 이 사람들에 대해 모든 트래픽을 감당하기엔 1대의 서버로는 부족합니다. 이에 대한 대응방안으로는 2가지가 있다.
- Scale-Up: 하드웨어의 성능을 올린다.
- Scale-Out: 여러대의 서버가 나눠서 일하도록 한다.   

<img src = "https://user-images.githubusercontent.com/63203480/134293406-f2d834dd-388f-424c-81be-aee7a147899a.jpg" width="50%" height="50%">

하드웨어 향상 비용이 더욱 비싸기도 하고, 서버가 여러대면 무중단 서비스를 제공하는 환경 구성이 용이하므로 
Scale-Out 이 효과적이다. 이때 **여러 서버에게 균등하게 트래픽을 분산** 시켜주는 것이 바로 **로드 밸런싱** 이다.   

로드 밸런싱은 여러 서버에게 부하(Load) 를 나눠주는 역할을 한다. Load Balancer 를 클라이언트와 서버 사이에 두고, 
부하가 일어나지 않도록 여러 서버에 분산시켜주는 방식이다.   
서비스를 운영하는 어플리케이션의 규모에 따라 웹 서버를 추가로 증설하면서 로드 밸런서를 통해 관리해주면
웹 서버의 부하를 해결할 수 있다.

### Load Balancer 종류
- L2 : Max 주소를 기반으로 Load Balancing
- L3 : IP 주소를 기반으로 Load Balancing
- L4 : Transport Layer(IP, Port) Level 에서 Load Balancing
    - TCP, UDP

![1](https://user-images.githubusercontent.com/63203480/134293798-9415c46b-d3d6-431e-83be-3d781cc8f026.png)

- L7 : Application Layer(사용자의 Request) Level 에서 Load Balancing
    - HTTP, HTTPS, FTP ..

![1](https://user-images.githubusercontent.com/63203480/134294534-821c8ccf-4afc-4f5f-8638-6c6c474a64c7.png)

### Load Balancer 가 서버를 선택하는 방법
- Round Robin : CPU 스케쥴링의 라운드 로빈 방식 활용
- Least Connection : 연결 개수가 가장 적은 서버를 선택(트래픽으로 인해 세션이 길어지는 경우 권장)
- Source : 사용자 IP 를 해싱하여 분배(특정 사용자가 항상 같은 서버로 연결되도록 보장)
- Load Balancer 장애대비 : 서버를 분배하는 로드 밸런서에 문제가 생길 수 있기 때문에 로드 밸런서를 이중화하여 대비한다.
    - 데이터 이중화 시스템: **Active** 된 데이터서버가 Fail 될 경우 실시간 복제를 계속 수행한
      **Passive** 서버를 Active 상태로 변환시켜 서비스의 단절을 방지한다.
    

