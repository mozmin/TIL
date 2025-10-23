# EC2 설정 법

## 해당 글에서는 AWS의 EC2 설정법에 대해서 정리함

### 1. EC2 인스턴스 생성

AWS Management Console에 로그인한 후, 다음 단계에 따라 EC2 인스턴스를 생성.

1.1. 인스턴스 시작
EC2 대시보드로 이동합니다.

[인스턴스] 메뉴에서 [인스턴스 시작] 버튼을 클릭합니다.

1.2. AMI (Amazon Machine Image) 선택
애플리케이션 및 OS 이미지: Amazon Linux

Amazon Machine Image (AMI): Amazon Linux 2023 AMI (또는 Amazon Linux 2)

프리 티어 사용이 가능한 최신 버전 사용을 권장합니다.

1.3. 인스턴스 유형
인스턴스 유형: t2.micro (프리 티어)

테스트 및 소규모 프로젝트에 적합합니다.

1.4. 키 페어 (로그인)
EC2 인스턴스에 SSH로 접속하기 위한 필수 단계입니다.

**[새 키 페어 생성]**을 클릭합니다.

키 페어 이름: 식별하기 쉬운 이름 입력 (예: my-project-key)

키 페어 유형: RSA

프라이빗 키 파일 형식: .pem (OpenSSH 사용 시)

[키 페어 생성] 버튼을 클릭하면 .pem 파일이 로컬 PC에 다운로드됩니다.

[주의] 이 파일은 다시 다운로드할 수 없으므로, 안전한 곳에 보관해야 합니다. (예: ~/.ssh/ 디렉토리)

1.5. 네트워크 설정 (보안 그룹)
[네트워크 설정] 패널에서 [편집]을 클릭합니다.

**[보안 그룹 생성]**을 선택하고, 적절한 이름과 설명을 입력합니다. (예: my-project-sg)

인바운드 보안 그룹 규칙:

SSH (포트 22): EC2에 접속하기 위해 필요합니다.

소스 유형: 내 IP (보안을 위해 현재 접속한 IP만 허용)

HTTP (포트 80): 웹 서버 접근을 위해 필요합니다.

소스 유형: 위치 무관 (0.0.0.0/0)

HTTPS (포트 443): SSL 적용 시 필요합니다.

소스 유형: 위치 무관 (0.0.0.0/0)

(선택) 사용자 지정 TCP (포트 8080): Spring Boot 내장 톰캣 포트(기본값 8080)에 직접 접근할 경우 추가합니다.

소스 유형: 내 IP (테스트용) 또는 위치 무관

1.6. 스토리지 (볼륨)
기본값 (프리 티어 30GiB까지 가능)을 유지합니다. gp3 유형 8GiB로도 충분합니다.

1.7. 인스턴스 시작
모든 설정을 확인한 후 [인스턴스 시작] 버튼을 클릭합니다.

### 2. EC2 인스턴스 접속 (SSH)

생성된 인스턴스에 터미널(Git Bash, macOS/Linux 터미널)을 이용해 접속

2.1. 키 파일 권한 변경 (최초 1회)
다운로드한 .pem 키 파일에 올바른 권한을 부여해야 합니다. (macOS/Linux 기준)
```
# .pem 파일이 있는 디렉토리로 이동
# 예: cd ~/.ssh/

# 키 파일에 읽기 전용 권한 부여
chmod 400 your-key-name.pem
```

2.2. SSH 접속
1. EC2 인스턴스 목록에서 방금 생성한 인스턴스를 선택.

2. [세부 정보] 탭에서 퍼블릭 IPv4 주소를 복사.

3. 터미널에서 아래 명령어를 실행.
```
# ssh -i [키 파일 경로] [사용자명]@[퍼블릭 IP 주소]
# Amazon Linux 2023/2의 기본 사용자명은 'ec2-user'입니다.

ssh -i ~/.ssh/your-key-name.pem ec2-user@xx.xx.xx.xx
```
최초 접속 시 "Are you sure you want to continue connecting (yes/no/[fingerprint])?" 메시지가 나타나면 yes를 입력.

### 3. 서버 초기 설정 (Java, Git 설치)

EC2 인스턴스에 접속한 상태에서, Spring Boot 애플리케이션 실행에 필요한 기본 패키지를 설치.

참고: Amazon Linux 2023은 dnf를, Amazon Linux 2는 yum을 패키지 관리자로 사용합니다. 아래는 dnf (AL 2023) 기준.


3.1. 서버 패키지 업데이트
```
sudo dnf update -y
```

3.2. Java (JDK) 설치
Amazon Corretto 17 (Java 17)을 설치합니다.

```
# Amazon Corretto 17 JDK 설치
sudo dnf install java-17-amazon-corretto -y

# 설치 확인
java -version

```

3.3. Git 설치
소스 코드를 클론하기 위해 Git을 설치합니다.

```
# Git 설치
sudo dnf install git -y

# 설치 확인
git --version
```