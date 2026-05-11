## TIL: VS Code를 활용한 EC2 PostgreSQL 원격 연결 (SSH 터널링)

### 1. 필요 도구

* **VS Code Extension**: `PostgreSQL` 
* **준비물**: EC2 퍼블릭 IP, SSH 접속용 `.pem` 키, DB 접속 정보 (아이디/비밀번호)

---

### 2. 연결 설정 단계

VS Code 왼쪽 사이드바의 **PostgreSQL 코끼리 아이콘**을 클릭하고 `Add Connection`을 선택.

#### **Step 1: SSH 터널링 설정 (Tunneling)**

보안상 5432 포트를 외부에 개방하지 않았으므로, SSH를 먼저 경유해야 함.

* **Host**: `43.***.***.194` (EC2 public IP)
* **User**: `ubuntu`
* **Auth Method**: `Private Key` 선택 후 `.pem` 파일 경로 지정

#### **Step 2: DB 접속 정보 입력 (PostgreSQL)**

터널이 연결되면 VS Code는 EC2 내부에서 통신하는 것과 같습니다.

* **Host**: `localhost` (터널 내부 기준)
* **Port**: `5432`
  * 로컬 서버 DB와 배포 서버 DB를 구분하기 위해, 배포 서버의 포트는 5433으로 설정하여 접속할 수 있음.
* **Database**: `aka_db`
* **User**: `deploy`
* **Password**: `pw`

### SSH 터널링

SSH 터널링을 사용하면 **AWS 보안 그룹(Security Group)에서 5432 포트를 열지 않아도 됨.** 오직 22번(SSH) 포트만 열려 있으면 외부에서 안전하게 DB에 접근할 수 있다는 점이 보안상의 큰 장점.

#### 🔍 왜 Host를 localhost로 하나요?

SSH 터널링은 내 컴퓨터와 EC2 서버 사이에 비밀 통로를 만드는 것과 같음.

DBeaver가 내 컴퓨터의 특정 포트로 데이터를 보내면,

SSH 통로를 타고 EC2 서버 안으로 들어간 뒤,

그 안에서 다시 localhost:5432에 떠 있는 PostgreSQL 컨테이너로 전달됨.

따라서 AWS 보안 그룹에서 5432 포트를 0.0.0.0/0으로 열지 않아도 안전하게 접속할 수 있음.
