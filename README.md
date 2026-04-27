# MCL Profiles

一個用來練習 Docker 基本操作的小專案。每個人建立自己的個人詳細介紹頁面，並在首頁加上自己的連結卡片。

核心目標：練習 docker 操作、練習 docker compose 操作、運用Dockerfile製作image

## 快速開始

### Step 1：Clone 專案

```bash
git clone git@github.com:OpenMCL/mcl-profiles-docker.git
cd mcl-profiles-docker
```

## docker
### Step 2：撰寫 Dockerfile 
在專案根目錄下建立一個名為 Dockerfile 的檔案（注意拼字且無副檔名），內容如下：
```bash
# 使用 Nginx 最輕量的版本作為地基
FROM nginx:alpine

# 將本地的網頁檔案搬進 Nginx 的預設路徑
COPY . /usr/share/nginx/html

# 聲明服務埠號為 80
EXPOSE 80
```

### Step 3：Build Image

```bash
# -t 代表標記名稱，最後的「.」代表當前目錄
docker build -t mcl-docker-image:v1 .
# 顯示現在有什麼image
docker images
```

### Step 4：Run Container

```bash
# -d: 背景執行, -p: 電腦 8080 對接容器 80
docker run -d -p 8080:80 --name docker-web mcl-docker-image:v1
```
### Step 5：查看成果

打開chrome打:`http://vm's ip:port` 

### Step 6： docker 基本操作

以下是幾個常見的 Docker command 來管理容器的生命周期：

啟動容器： `docker run` 加上 image 名來啟動一個新的容器。

查看容器： `docker ps` 查看正在運行的容器，加上 -a 參數可以看到所有容器，包括停止的。

停止容器： `docker stop`

重啟容器： `docker restart`

刪除容器： `docker rm` 命令來刪除停止的容器。

進入容器： `docker exec` 加上 -it 參數來進入一個正在運行的容器中。

查看日誌 : `docker logs` 加上 容器ID或名稱 可以讓你查看 container 的 log，通常如果網頁有 bug 報 internal server error 或 500 的話看這個 log 應該可以看到 error
## docker compose

### Step 7：撰寫 docker-compose.yml
在專案根目錄下建立一個名為 docker-compose.yml 的檔案（注意拼字），內容如下：
```bash
version: '3.8'

services:
  web:
    image: mcl-docker-image:v1
    container_name: docker-compose-web
    ports:
      - "8081:80"
    restart: always
```

### Step 8：啟動 container 

```bash
docker-compose up -d
```

### Step 9：查看成果
打開chrome打:`http://vm's ip:port` 

### Step 10： docker compose 基本操作

以下是幾個常見的 Docker compose command 來管理多個容器的生命周期：

一鍵啟動： `docker compose up -d` -d 代表在背景執行（Detach），不會佔用你的 Terminal。

查看狀態： `docker compose ps`  只顯示跟這個專案有關的容器。

停止並移除： `docker compose down`

重新啟動： `docker compose restart`

進入容器： `docker compose exec [服務名稱] sh`  這裡用的是 yml 裡的 service 名稱，不是那一串亂碼 ID。

查看日誌： `docker compose logs -f` 加上 -f 可以即時追蹤（Follow），對於 Debug 網頁噴出的 500 錯誤非常有幫助。


## 檔案結構

```
mcl-profiles-docker/
├── Dockerfile              
├── docker-compose.yml              
├── index.html              # 團隊首頁
├── styles.css              # 共用樣式
├── profiles/
│   ├── template.html       # 個人頁面模板
│   └── yourname.html       # 個人詳細介紹頁面
├── assets/photos/          # 大頭照存放目錄
└── README.md
```
