Health Check, CRUD 기능이 있는 간단한 Flask API Server입니다.
# app.py
DB 연동을 위해 DB Endpoint가 필요합니다.
따라서 terraform output -json > terraform_outputs.json 명령으로 json 파일을 생성해서 테라폼의 output을 저장해야 합니다. 

이 부분을 자동화하고 싶었지만 그러지 못했습니다.

CRUD의 엔드포인트와 HTTP 메서드는 다음과 같습니다.
- Create: /items POST
- Read: /items GET
- Update: /items/<int:item_id> PUT
- Delete: /items/<int:item_id> DELETE

/health 엔드포인트에 GET 요청 시 {"status":"ok"}를 반환합니다. 

이를 이용해 Jenkins pipeline에서 헬스 체크를 할 수 있습니다.

# Jenkinsfile
깃 트리거를 구성해서, git push 시 자동으로 다음과 같은 순서로 파이프라인으로 수행합니다
1. docker --version을 실행해 docker agent가 도커 명령을 수행할 수 있는지 확인합니다.
2. Docker build 명령으로 새 이미지를 빌드합니다.
3. Jenkins에 등록된 Credential을 이용해 도커 허브에 로그인합니다.
4. Docker Hub에 이미지를 푸시합니다.
5. 헬스 체크를 위해 컨테이너를 실행합니다.
6. response 환경변수에 /healthCheck 엔드포인트에 curl 명령한 결과를 저장합니다. {"status":"ok"}를 반환하지 않을 경우 파이프라인을 실패시킵니다.

# Dockerfile
- flask server 실행을 위해 베이스 이미지로 python:3.9-slim을 사용합니다.
- app.py와 flask, DB 의존성 패키지가 저장된 request.txt, curl 명령을 수행하는 healthCheck.sh 파일을 복사합니다.
- 5000번 포트를 노출하고, 의존성 설치 후 flask server를 실행합니다.
  
# Health Check와 CRUD 명령 수행 결과
![CRUD](https://github.com/user-attachments/assets/6baaf259-271b-45a2-8000-68eb09bb701a)



  
