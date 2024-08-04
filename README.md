Health Check, CRUD 기능이 있는 간단한 Flask API Server입니다.
# app.py
- DB 연동을 위해 DB Endpoint가 필요합니다.
- 따라서 terraform output -json > terraform_outputs.json 명령으로 output이 담긴 json 파일을 생성해야 합니다.
- 이 부분을 자동화하고 싶었지만 그러지 못했습니다.
- json 파일을 읽어서 output의 db endpoint, db user 등의 정보로 mysql_uri를 만들었습니다.

- Item 테이블에는 id와 name column이 있습니다.

- CRUD의 엔드포인트와 HTTP 메서드는 다음과 같습니다.
  - Create: /items POST
  - Read: /items GET
  - Update: /items/<int:item_id> PUT
  - Delete: /items/<int:item_id> DELETE

- /health 엔드포인트에 GET 요청 시 {"status":"ok"}를 반환합니다. 

- 이를 이용해 Jenkins pipeline에서 헬스 체크를 할 수 있습니다.

# Jenkinsfile
깃 트리거를 구성해서, git push 시 다음과 같은 순서로 자동으로 파이프라인을 수행합니다
1. docker --version을 실행해 docker agent가 도커 명령을 수행할 수 있는지 확인합니다.
2. Docker build 명령으로 새 이미지를 빌드합니다.
3. Jenkins에 등록된 Credential을 이용해 도커 허브에 로그인합니다.
4. Docker Hub에 이미지를 푸시합니다.
5. 헬스 체크를 위해 컨테이너를 실행합니다.
6. response 환경변수에 /healthCheck 엔드포인트에 curl 명령한 결과를 저장합니다.
   - {"status":"ok"}를 반환하지 않을 경우 파이프라인을 실패시킵니다.

# Dockerfile
- flask server 실행을 위해 베이스 이미지로 python:3.9-slim을 사용합니다.
- app.py와 flask, DB 의존성 패키지가 저장된 request.txt, curl 명령을 수행하는 healthCheck.sh 파일을 복사합니다.
- 5000번 포트를 노출하고, 의존성 패키지 설치 후 flask server를 실행합니다.
  
# Health Check와 CRUD 명령 수행 결과
다음 순서로 테스트를 진행했습니다.
1. health check
2. CREATE: test_item1
3. CREATE: test_item2
4. READ: 현재 DB의 item 목록 조회
5. UPDATE: id가 4인 item을 updated_item4로 변경
6. DELETE: id가 3인 item 삭제
7. READ: 현재 DB의 item 목록 조회
   
![CRUD](https://github.com/user-attachments/assets/6baaf259-271b-45a2-8000-68eb09bb701a)



  
