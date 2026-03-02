# redmine_superset_mariadb
Redmine+Superset Docker based MariaDB, Redmine Theme, Plugins .Good Luck...!

📦 Redmine & Superset Analytics Integrated Package for Synology
이 패키지는 Synology NAS Docker(Container Manager) 환경에서 Redmine(프로젝트 관리)과 Superset(데이터 시각화)을 연동하고, 업무 효율을 높이는 필수 플러그인을 포함한 통합 환경을 구축합니다.
1. 📂 사전 준비 (Synology File Station)
설치 전, NAS의 /volume2/ssd_docker/redmine_maria/ 경로에 아래 폴더들을 미리 생성하고 Docker 실행 권한(Read/Write)을 부여하세요.
mysql, redmine_files, redmine_plugins, redmine_themes, superset_home
2. 🐳 Docker Compose 설정 (docker-compose.yml)
yaml
version: '3.8'

services:
  # [1] MariaDB: 데이터 저장소
  db:
    image: mariadb:10.11
    container_name: redmine-mariadb
    restart: always
    networks:
      - analytics-net
    ports:
      - "3306:3306" # 192.168.0.* 대역 DBeaver 접속 허용
    environment:
      MYSQL_ROOT_PASSWORD: 'your_root_password'
      MYSQL_DATABASE: 'redmine'
      MYSQL_USER: 'redmine'
      MYSQL_PASSWORD: 'redmine_password'
    volumes:
      - /volume2/ssd_docker/redmine_maria/mysql:/var/lib/mysql

  # [2] Redmine: 프로젝트 관리 (플러그인 포함)
  redmine:
    image: redmine:latest
    container_name: redmine-app
    restart: always
    networks:
      - analytics-net
    ports:
      - "8080:3000"
    environment:
      REDMINE_DB_MYSQL: db
      REDMINE_DB_DATABASE: redmine
      REDMINE_DB_USERNAME: redmine
      REDMINE_DB_PASSWORD: redmine_password
      # 생성방법: 터미널에서 'openssl rand -hex 64' 실행 후 결과값 입력
      REDMINE_SECRET_KEY_BASE: 'f1a2b3...your_generated_redmine_key...'
    volumes:
      - /volume2/ssd_docker/redmine_maria/redmine_files:/usr/src/redmine/files
      - /volume2/ssd_docker/redmine_maria/redmine_plugins:/usr/src/redmine/plugins
      - /volume2/ssd_docker/redmine_maria/redmine_themes:/usr/src/redmine/public/themes
    depends_on:
      - db

  # [3] Apache Superset: 데이터 분석 및 시각화
  superset:
    image: apache/superset:latest
    container_name: superset-app
    restart: always
    networks:
      - analytics-net
    ports:
      - "8088:8088"
    user: "root"
    environment:
      # 생성방법: 터미널에서 'openssl rand -base64 42' 실행 후 결과값 입력
      SUPERSET_SECRET_KEY: 's1u2p3...your_generated_superset_key...'
    volumes:
      - /volume2/ssd_docker/redmine_maria/superset_home:/app/superset_home
    depends_on:
      - db

networks:
  analytics-net:
    driver: bridge
코드를 사용할 때는 주의가 필요합니다.

3. 🛠️ 설치 및 설정 매뉴얼 (GitHub Style)
Step 1: 보안키 생성 (Secret Key Generation)
보안을 위해 각 서비스의 고유 키를 생성합니다. Synology SSH 또는 터미널에서 실행하세요.
Redmine용: openssl rand -hex 64 실행 결과를 REDMINE_SECRET_KEY_BASE에 입력.
Superset용: openssl rand -base64 42 실행 결과를 SUPERSET_SECRET_KEY에 입력.
Step 2: 플러그인 및 테마 수동 설치
요청하신 플러그인들은 정해진 폴더에 다운로드 후 압축을 풀어야 합니다.
/volume2/ssd_docker/redmine_maria/redmine_plugins/ 경로에 아래 플러그인 업로드:
Clipboard Image Paste
Projects Treeview
View Customize (워크플로우 및 UI 커스텀 필수)
설치 후 Redmine 컨테이너 터미널에서 실행:
bash
bundle exec rake redmine:plugins:migrate RAILS_ENV=production
docker restart redmine-app
코드를 사용할 때는 주의가 필요합니다.

Step 3: Superset 초기화 (First Run)
컨테이너가 실행된 후 다음 명령어를 순차적으로 실행하여 관리자 계정을 생성합니다.
bash
docker exec -it superset-app superset fab create-admin --username admin --firstname admin --lastname admin --email admin@example.com --password admin

docker exec -it superset-app superset db upgrade

docker exec -it superset-app superset init

코드를 사용할 때는 주의가 필요합니다.

Step 4: DBeaver 연결 (192.168.0. 대역)*
Host: NAS IP (예: 192.168.0.10)
Port: 3306
Database: redmine
Auth: 사용자 redmine / 비밀번호 redmine_password
Tip: MariaDB 드라이버 사용 권장.
4. 💡 주요 사용법 및 팁
Clipboard Image: 이슈 작성 시 Ctrl+V로 이미지를 즉시 첨부할 수 있습니다.
Workflow View (D3): View Customize Plugin을 통해 D3.js 스크립트를 삽입하여 고급 워크플로우 시각화를 구현합니다.
Superset 연동: Superset 웹(:8088) 접속 후 Settings > Database Connections에서 호스트명을 db(Docker 내부 네트워크 이름)로 지정하여 Redmine 데이터를 실시간 분석하세요.
