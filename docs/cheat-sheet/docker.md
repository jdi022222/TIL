# Docker

## Docker

### **1. Images**

*   Dockerfile로부터 이미지 빌드하기:

    ```
    docker build -t <이미지_이름> .
    ```
*   캐시 없이 Dockerfile로부터 이미지 빌드하기:

    ```
    docker build -t <이미지_이름> . --no-cache
    ```
*   로컬 이미지 목록 확인하기:

    ```
    docker images
    ```
*   이미지 삭제하기:

    ```
    docker rmi <이미지_이름>
    ```
*   사용하지 않는 이미지 모두 삭제하기:

    ```
    docker image prune
    ```



### **2. Docker Hub**

*   Docker에 로그인하기:

    ```
    docker login -u <사용자명>
    ```
*   Docker Hub에 이미지 발행하기:

    ```
    docker push <이미지_이름>
    ```
*   Hub에서 이미지 검색하기:

    ```
    docker search <이미지_이름>
    ```
*   Docker Hub에서 이미지 가져오기:

    ```
    docker pull <이미지_이름>
    ```



### **3. Containers**

*   커스텀 이름으로 이미지로부터 컨테이너 생성 및 실행하기:

    ```
    docker run --name <컨테이너_이름> <이미지_이름>
    ```
*   컨테이너의 포트를 호스트에 게시하여 실행하기:

    ```
    docker run -p <호스트_포트>:<컨테이너_포트> <이미지_이름>
    ```
*   백그라운드에서 컨테이너 실행하기:

    ```
    docker run -d <이미지_이름>
    ```
*   기존 컨테이너 시작 또는 중지하기:

    ```
    docker start|stop <컨테이너_이름>
    ```
*   중지된 컨테이너 삭제하기:

    ```
    docker rm <컨테이너_이름>
    ```
*   실행 중인 컨테이너 내부의 쉘 열기:

    ```
    docker exec -it <컨테이너_이름> sh
    ```
*   컨테이너 로그 가져오기 및 실시간 추적하기:

    ```
    docker logs -f <컨테이너_이름>
    ```
*   실행 중인 컨테이너 검사하기:

    ```
    docker inspect <컨테이너_이름>
    ```
*   현재 실행 중인 컨테이너 목록 보기:

    ```
    docker ps
    ```
*   모든 Docker 컨테이너(실행 중인 것과 중지된 것 모두) 목록 보기:

    ```
    docker ps --all
    ```
*   리소스 사용량 통계 보기:

    ```
    docker container stats
    ```
