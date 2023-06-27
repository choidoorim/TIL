# lesson30. DNS 서버의 구조(이름 해석)

## 1. 도메인 이름이란?

- 컴퓨터에는 IP 주소가 있기 때문에 인터넷을 통해 웹 서버에 접속해서 웹 사이트를 볼 수 있었다. 하지만 브라우저에 `www.google.com` 이라는 URL 을 입력하면 구글 웹사이트를 볼 수 있다. IP 주소를 입력하지 않아도 웹 페이지를 볼 수 있는데, **DNS 는 URL 을 IP 주소로 변환하는 서비스(시스템)**이다.
- `www.google.com` 을 입력하면 DNS 서버가 웹 사이트 서버의 IP 주소를 알려주는데, 이것을 이름 해석(name resolution)이라고 한다.
- `www.google.com` 과 같이 컴퓨터나 네트워크를 식별하기 위해서 붙여진 이름을 Domain Name 이라고 하고, 도메인 이름 앞에 있는 www 는 Host Name(서버 이름)이다.
    - 파일 전송 서버일 경우 호스트 이름은 ftp 일 것이고, 메일 서버일 경우 mail 일 것이다.
- 컴퓨터와 DNS 서버 간에 일어나는 IP 주소 교환을 아래와 같다.
    1. 컴퓨터에서 도메인 이름(ex. `www.google.com`)의 IP 주소가 무엇인지 DNS 서버에 요청한다.
    2. DNS 서버는 해당 도메인 이름에 맞는 IP 주소를 알려 준다.
    3. 컴퓨터는 IP 주소로 서버에 접속한다.
- DNS 서버는 전 세계에 계층적으로 연결되어 있다. 따라서 DNS Server1 에서 도메인 네임에 해당하는 IP 주소를 찾지 못할 경우, DNS Server2, 3 등 에 요청을 해서 IP 주소를 찾을 수 있도록 한다.

  ![Untitledasd](https://github.com/choidoorim/TIL/assets/63203480/a3a6468e-8f84-4a74-905c-cdf6074ae2c0)
