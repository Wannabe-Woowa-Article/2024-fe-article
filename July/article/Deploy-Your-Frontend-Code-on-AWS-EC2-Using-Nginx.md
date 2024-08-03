## 🔗 [Frontend 코드를 AWS EC2의 Nginx을 사용하여 배포하는 방법](https://medium.com/@jpan0831/deploy-your-frontend-code-on-aws-ec2-using-nginx-98bdb96ccbe4)

### 🗓️ 번역 날짜: 2024.07.28

### 🧚 번역한 크루: 소하(최소연)

---

### Step1: 프론트엔드 소스 코드를 프로덕션 코드로 빌드

client-side rendering을 위한 대부분의 프론트엔드 프레임워크, 예를 들어 React와 Vue에서는 개발 작업을 프로덕션 준비 코드로 컴파일해야 합니다. 이 과정을 거치면 웹 브라우저에서 읽을 수 있는 HTML, CSS, JavaScript 파일이 생성됩니다. Create React App을 사용하는 예를 들어보면, `npm run build` 명령어를 사용하여 컴파일하고 다음과 같은 파일을 생성할 수 있습니다.

<img width="500px" src="https://miro.medium.com/v2/resize:fit:1376/format:webp/1*8oyw6-LW8EjRMr9tL3kdxQ.png"/>

### Step 2: Nginx 설정

#### Nginx이란

Nginx은 인기 있는 오픈소스 웹 서버, reverse proxy 서버, 로드 밸런서입니다. 많은 동시 연결을 효율적으로 처리하도록 설계되어 있어 정적 콘텐츠 제공, 웹 트래픽 처리, 클라이언트와 백엔드 서버 간의 proxy 역할에 적합합니다.

#### 웹 서버로 Nginx 설정

간단한 몇 가지 명령어로 Nginx을 웹 서버로 사용하여 HTML, CSS, JavaScript 및 정적 파일을 제공할 수 있습니다.

```
http {
  # 파일 인식을 위한 공통 MIME 유형을 포함합니다.
  include mime.types;

  server {
    # 수신 연결을 위해 8080번 port(default port)에서 수신합니다.
    listen 8080;

    # 파일을 제공할 root directory를 설정합니다.
    # 빌드 폴더 대상 경로
    root /Users/jaypan/workspace/nginx/client/build;

    location / {
      # 요청된 파일($uri)을 찾고, 찾지 못한 경우 index.html을 제공합니다.
      # SPA에서는 routing이 클라이언트 측에서 처리됩니다.
      # 따라서 요청된 리소스가 누락된 경우 클라이언트 앱에서 자체적으로 처리하도록 합니다.
      try_files $uri /index.html;
    }
  }
}

events {}
```

#### 대체 접근법 (VanillaJS)

바닐라 자바스크립트(프레임워크 없이)를 사용하여 개발하고 있고, 프로젝트에 index.html, about.html과 같은 여러 정적 페이지가 포함되어 있다면 다음과 같이 구성을 수정할 수 있습니다.

```
http {

  include mime.types;

  server {
    listen 8080;
    root /Users/jaypan/workspace/nginx;

    location ~* /users/[0-9] {
      root /Users/jaypan/workspace/nginx;
      try_files /index.html =404;
    }

    location /about {
      root /Users/jaypan/workspace/nginx;
    }
  }
}

events {}
```

아래 제공된 crash course를 살펴보세요. nginx 컨텍스트와 지시어에 대한 이해를 높이는 데 매우 도움이 되었습니다.

<iframe width="500" height="315" src="https://www.youtube.com/embed/7VAI73roXaY?si=GeKiuy2lJf-2AvMs" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

#### 로컬 머신에서 테스트

macOS 사용자라면 앞서 설명한 대로 로컬 컴퓨터의 nginx.conf 파일을 수정하려면 다음 단계를 따르면 됩니다. 컴퓨터에 Nginx을 설치하지 않았다면 Homebrew를 사용하여 간편하게 설치할 수 있습니다.

```bash
vim /usr/local/etc/nginx/nginx.conf
```

<img width="500px" src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*VJLWOZbzT_K0iEDzP75uwQ.png"/>

이제 웹사이트를 로컬에 성공적으로 배포했으니, 다음 단계는 콘텐츠를 공개적으로 액세스할 수 있도록 만드는 것입니다.

### Step3: AWS EC2 인스턴스

#### EC2란?

Elastic Compute Cloud(EC2)는 AWS에서 제공하는 인프라 서비스(IaaS)입니다. 아마존 데이터 센터에 위치한 물리 서버에서 실행되는 가상 머신이라고 생각하시면 됩니다. 각 인스턴스(가상 머신)에는 고유한 공용 IP 주소가 부여됩니다.

#### 목표

목표는 이 인스턴스 내에 동일한 Nginx 구성을 구현하는 것입니다. 이렇게 하면 인터넷을 통해 웹사이트에 액세스할 수 있으며, 로컬 액세스에 제한되지 않습니다.

#### EC2 인스턴스 생성

AWS 계정이 이미 있다고 가정하고, EC2 대시보드로 이동하여 오른쪽 상단 모서리에 있는 "Launch Instance" 버튼을 클릭합니다.

Ubuntu 버전을 선택하고 네트워크 설정에서 HTTP 트래픽을 허용하는 옵션을 활성화합니다. 현재로서는 HTTPS를 구성하지 않을 것임을 참고해 주세요. key pair를 생성하는 것을 잊지 마세요. 이는 인스턴스에 적절한 액세스 권한을 부여하는 데 중요합니다.

<img width="500px" src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*qTHUKNMMlyhvEfBUqmb_Vg.png"/>

_ubuntu 프리티어 선택_

<img width="500px" src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*dy1s6vAzzqsLIYSIG6ltuA.png"/>

_http traffic 허용_

<img width="500px" src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*s3AfVmFTcgUzESeNtdau7w.png"/>

_key pair 생성_

#### 보안 그룹(방화벽) 구성

인스턴스 생성 시 "allow HTTP traffic" 옵션을 선택하지 않은 경우, 보안 그룹 설정을 수정하여 이를 수정할 수 있습니다. 인바운드 규칙을 추가하여 HTTP 트래픽을 허용하세요.

<img width="500px" src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Ssm_hlMfN2KLVPkUiutBqQ.png"/>

### Step4: SSH를 사용하여 인스턴스에 접근

SSH는 command line을 사용하여 원격 서버 및 장치(Linux, Mac, Window 10+)에 안전하게 접근하고 관리할 수 있는 방법을 제공합니다.

EC2 인스턴스 페이지에서 "Connect" 버튼을 클릭하고 제공된 지침을 따르세요. 타임아웃 오류가 발생하는 경우 인스턴스에 올바른 접근 권한이 없다는 것을 의미합니다.

```bash
ssh -i "ubuntu.pem" ubuntu@ec2-15-168-62-96.ap-northeast-3.compute.amazonaws.com
```

<img width="500px" src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*33FoDc8jDL6p93RfgXn6Tg.png"/>

### Step5: Ubuntu Linux에 Nginx 설치

```bash
// Ubuntu 시스템의 패키지 목록을 업데이트합니다.
sudo apt-get update

sudo apt install nginx

// nginx이 성공적으로 설치되었는지 확인합니다.
nginx -v

// Nginx 서비스를 시작합니다.
sudo systemctl start nginx

// 시스템 부팅 시 자동으로 시작되도록 설정합니다.
sudo systemctl enable nginx

// Nginx 서비스의 상태를 확인합니다.
sudo systemctl status nginx
```

### Step6: 기본 Nginx 파일 수정

기본 Nginx 구성 파일을 Step2의 지침에 맞게 수정하세요. 하지만 이것을 잊지 마세요.

```bash
vim /etc/nginx/nginx.conf
```

### Step7: 리눅스에 빌드 파일 복사

`scp`는 Secure Copy Protocol의 약자로, 로컬 호스트와 원격 호스트 간에 파일을 안전하게 전송하는 데 사용되는 command line 도구입니다. 이를 사용하여 빌드 폴더를 인스턴스로 복사해 보겠습니다.

```bash
// scp -i {key-pair.pem} -r {build folder} ubuntu@{dns}:{destination-folder}
scp -i "ubuntu.pem" -r /Users/jaypan/workspace/nginx/client/build ubuntu@ec2-15-168-39-19.ap-northeast-3.compute.amazonaws.com:/var/www/html/

// "permission denied"와 같은 오류가 발생하는 경우 -v를 추가하여 자세한 정보를 확인할 수 있습니다.
```

"build" 폴더가 성공적으로 복사되었는지 확인하세요. 기본적으로 제한된 권한을 가진 `/usr`에 파일을 복사하면 권한 문제가 발생할 수 있습니다. 웹 서버 파일을 수용하기 위해 일반적으로 더 적은 권한으로 구성된 `/var/www/html`에 복사하는 것이 좋습니다.

### Step8: Nginx `root` directive 업데이트

그런 다음 Nginx root directive를 업데이트하세요.

```
http {
  ...

  server {
    ...

    root /var/www/html/build;

    ...
  }
}
```

### Step9: 공용 IP에서 확인

이것으로 끝입니다! 이제 Nginx를 사용하여 AWS EC2에 프로덕션 프론트엔드 코드를 배포하는 방법을 알게 되었습니다.

<img width="500px" src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*_h2GIVsEsBfKeef_w08_QQ.png"/>

### 결론

프로덕션 코드를 배포하는 기본 원리를 이해하는 것은 매우 가치가 있습니다. 비록 업데이트가 필요할 때마다 수동 단계가 필요하더라도 말입니다. 앞으로는 이 과정을 간소화하고 최적화하기 위해 Docker를 사용하는 방법을 알아볼 계획입니다.
