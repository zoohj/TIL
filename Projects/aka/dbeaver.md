현지님, 로컬 워커가 EC2 DB에 성공적으로 붙었으니 DBeaver에서도 똑같이 연결해서 데이터를 확인해 보면 훨씬 편할 거예요.

EC2에 있는 PostgreSQL은 보안상 외부 접속을 막아두었기 때문에, 아까 Celery 설정할 때처럼 SSH 터널링 방식을 사용하는 것이 가장 안전하고 깔끔합니다.

🛠️ DBeaver에서 EC2 DB 연결하는 방법
DBeaver를 실행하고 새 연결(New Connection)을 누른 뒤 아래 순서대로 입력해 보세요.

1. Main 탭 (데이터베이스 기본 정보)
여기에 입력하는 정보는 EC2 내부(도커 컨테이너 기준) 정보입니다.

Host: localhost (SSH 터널을 통해 들어갈 것이므로 localhost로 적습니다)

Port: 5432

Database: aka_db (또는 현지님이 만드신 DB 이름)

Username: deploy (또는 .env에 설정한 유저 이름)

Password: jwCZeffJ8indBc1cs (설정하신 비밀번호)

2. SSH 탭 (가장 중요!)
여기서 현지님의 EC2 서버로 들어가는 통로를 설정합니다.

Use SSH Tunnel: 체크 ✅

SSH Host: 43.200.164.194 (EC2 퍼블릭 IP)

Port: 22

User Name: ubuntu

Authentication Method: Public Key 선택

Private Key: 현지님이 사용하시는 .pem 키 파일 경로 선택


✅ 연결 확인
설정을 마친 후 좌측 하단의 Test Connection을 눌러보세요. "Connected"가 뜨면 성공입니다!

이제 knowledge 테이블이나 youtube_metadata 테이블을 열어서, 아까 로컬 워커가 성공적으로 저장한 자막 데이터들이 잘 들어와 있는지 확인해 보세요. 자막 데이터가 잘 보이나요? :)


이것보다 visualstudio로 연결하는게 더 나은듯!



## TIL: VS Code를 활용한 EC2 PostgreSQL 원격 연결 (SSH 터널링)

DBeaver 같은 별도 툴 설치 없이, 평소 사용하던 **VS Code**에서 바로 DB 데이터를 확인하는 방법입니다. Docker 컨테이너로 실행 중인 DB라도 SSH 터널링을 이용하면 로컬 환경처럼 안전하게 접근할 수 있습니다.

---

### 1. 필요 도구

* **VS Code Extension**: `PostgreSQL` (Microsoft 공식 또는 Chris Kolkman 제작 권장)
* **준비물**: EC2 퍼블릭 IP, SSH 접속용 `.pem` 키, DB 접속 정보 (아이디/비밀번호)

---

### 2. 연결 설정 단계

VS Code 왼쪽 사이드바의 **PostgreSQL 코끼리 아이콘**을 클릭하고 `Add Connection`을 선택합니다.

#### **Step 1: SSH 터널링 설정 (Tunneling)**

보안상 5432 포트를 외부에 개방하지 않았으므로, SSH를 먼저 경유해야 합니다.

* **Host**: `43.200.164.194` (EC2 IP)
* **User**: `ubuntu`
* **Auth Method**: `Private Key` 선택 후 `.pem` 파일 경로 지정

#### **Step 2: DB 접속 정보 입력 (PostgreSQL)**

터널이 연결되면 VS Code는 EC2 내부에서 통신하는 것과 같습니다.

* **Host**: `localhost` (터널 내부 기준)
* **Port**: `5432`
* **Database**: `aka_db`
* **User**: `deploy`
* **Password**: `jwCZeffJ8indBc1cs`

---

### 3. 왜 VS Code 연결이 더 유리한가?

* **컨텍스트 스위칭 감소**: 코드 작성과 데이터 확인을 한 화면에서 처리할 수 있어 흐름이 끊기지 않습니다.
* **경량화**: DBeaver 같은 무거운 Java 기반 툴을 별도로 실행할 필요가 없습니다.
* **SQL 파일 관리**: 작성한 쿼리문을 `.sql` 파일로 만들어 바로 Git에 커밋하거나 프로젝트 폴더 내에 보관하기 용이합니다.

---

### 4. 데이터 확인 (Verification)

연결이 완료되면 Explorer 탭에서 테이블 목록을 확인할 수 있습니다.

* `youtube_metadata` 테이블: 로컬 워커가 수집한 유튜브 자막 데이터가 정상적으로 적재되었는지 확인.
* `knowledge` 테이블: 가공된 지식 베이스가 잘 들어갔는지 SQL 쿼리로 조회.

---

### 💡 핵심 포인트

SSH 터널링을 사용하면 **AWS 보안 그룹(Security Group)에서 5432 포트를 열지 않아도 됩니다.** 오직 22번(SSH) 포트만 열려 있으면 외부에서 안전하게 DB에 접근할 수 있다는 점이 보안상의 큰 장점입니다.

🔍 왜 Host를 localhost로 하나요?
SSH 터널링은 현지님의 컴퓨터와 EC2 서버 사이에 비밀 통로를 만드는 것과 같습니다.

DBeaver가 현지님 컴퓨터의 특정 포트로 데이터를 보내면,

SSH 통로를 타고 EC2 서버 안으로 들어간 뒤,

그 안에서 다시 localhost:5432에 떠 있는 PostgreSQL 컨테이너로 전달됩니다.

따라서 AWS 보안 그룹에서 5432 포트를 0.0.0.0/0으로 열지 않아도 안전하게 접속할 수 있습니다.
