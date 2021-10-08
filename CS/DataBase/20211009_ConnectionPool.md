## Connection Pool 

만약 Pool 을 사용하지 않는다면 DB서버에 최초로 연결하여 Connection 객체를 생성는 작업을 하기 때문에 비용적인 측면에서**큰 성능 저하**를 겪게 된다. 이와 같은 문제를 해결하기 위한 것이 Connection Pool 이다.

> **Connection Pool 이란?**

동시 접속자가 연결 될 수 있는 Connection을 Pool이라는 컨테이너에 하나로 모아서 관리하는 개념입니다.

WAS 실행 시 일정량의 Connection 객체를 생성하여 미리 Pool 이라는 캐시 공간에 저장해둔다.

만약 누군가 DataBase 에 접근한다면 Pool에 남아 있는 Connection을 제공하고,

만약 제공할 Connection이 없다면 클라이언트를 **대기 상태로 전환**시키고 Pool에 Connection이 반환된다면 대기 상태에 있는 클라이언트에게 순서대로 제공합니다.

즉, DB와 연결된 Connection을 미리 만들어서 pool 안에 저장해 두고 있다가 필요할 때 Connection을 Pool에서 가져와 사용 후 반납하는 것입니다.

![1](https://user-images.githubusercontent.com/63203480/136589564-f48268a0-445f-4053-a155-172f32dfd7be.png)

Connection Pool의 크기를 너무 크게하면 메모리의 소모가 크고, 적게 하면 Client의 대기 시간이 자주 발생할 수 있기 때문에 접속자 수나, 서버 부하에 따라 Connection Pool에 적절한 Connection을 생성해야 합니다.

어플리케이션이 실행되면 DB와 연결 된 Connection 객체를 미리 Pool에 생성해놓고 가져와 사용하고 반납하는 방식입니다.

> **Connection Pool 특징**

1. Connection Pool 에 DB와 연결 한 객체를 두고 필요할 때마다 Connection을 빌려오기 때문에 Connection을 생성하는 시간이 소비되지 않습니다.

2. Connection 을 생성하고 닫는 것이 아닌, Pool 안에 미리 생성 된 Connection을 사용하고 반납하기 때문에 어플리케이션의 실행 속도가 빨라집니다

3. 한 번에 생성될 수 있는 Connection 수를 제어할 수 있기 때문에 동시 접속자 수가 커져도 어플리케이션이 쉽게 다운되지 않습니다

4. Connection 이 객체이기 때문에 메모리를 차지하므로, 무작정 Connection을 늘리면 메모리를 많이 차지하게 되어 오히려 성능이 떨어질 수 있습니다
