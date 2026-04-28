# MCL Profiles (Docker)

一個用來練習 Docker 基本操作的小專案。每個人建立自己的個人詳細介紹頁面，並在首頁加上自己的連結卡片。

核心目標：練習 Docker 操作、練習 Docker Compose 操作、運用 Dockerfile 製作 Image

## 快速開始

### Step 1：Clone 專案

```bash
git clone git@github.com:OpenMCL/mcl-profiles-docker.git
cd mcl-profiles-docker
```

## Docker
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

在瀏覽器輸入：`http://<你的 VM IP>:8080` 

### Step 6： Docker 基本操作

以下是幾個常見的 Docker 指令：

*   **啟動容器**：`docker run` 加上 Image 名稱來啟動一個新的容器。
*   **查看容器**：`docker ps` 查看正在運行的容器，加上 `-a` 參數可以看到所有容器（包括已停止的）。
*   **停止容器**：`docker stop` 停止運行中的容器。
*   **重啟容器**：`docker restart` 重新啟動容器。
*   **刪除容器**：`docker rm` 刪除已停止的容器。
*   **進入容器**：`docker exec -it` 進入一個正在運行的容器中進行操作。
*   **查看日誌**：`docker logs` 加上容器 ID 或名稱，可以查看容器的運行日誌，對於 Debug 非常有幫助。

## Docker Compose
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
在瀏覽器輸入：`http://<你的 VM IP>:8081` 

### Step 10： Docker Compose 基本操作

以下是幾個常見的 Docker Compose 指令：

*   **一鍵啟動**：`docker compose up -d` 在背景執行所有服務。
*   **查看狀態**：`docker compose ps` 列出目前專案相關的容器狀態。
*   **停止並移除**：`docker compose down` 停止並移除所有相關容器與網路。
*   **重新啟動**：`docker compose restart` 重啟所有服務。
*   **進入容器**：`docker compose exec [服務名稱] sh` 進入指定的服務容器中。
*   **查看日誌**：`docker compose logs -f` 即時追蹤（Follow）容器的日誌輸出。


## 進階補充：使用 .dockerignore

當我們執行 `docker build` 時，Docker 會將當前目錄下的所有檔案發送到 Docker Daemon。為了避免將不必要的檔案（如 `.git`、`README.md` 或本地開發工具的設定）包進 Image 中，我們可以建立一個 `.dockerignore` 檔案。

這可以讓 Image 體積更小，並加快 Build 的速度。

### 如何實作：
在專案根目錄建立 `.dockerignore`，並填入以下內容：

```text
.git
.gitignore
README.md
Dockerfile
docker-compose.yml
```

---

## 檔案結構

```
mcl-profiles-docker/
├── Dockerfile              # 定義如何建立 Docker Image 的說明文件
├── docker-compose.yml      # 定義與啟動 Docker 容器的設定檔
├── .dockerignore           # 排除不需要包進 Image 的檔案 (選用)
├── index.html              # 團隊首頁
├── styles.css              # 共用樣式
├── profiles/
│   ├── template.html       # 個人頁面模板
│   └── yourname.html       # 個人詳細介紹頁面
├── assets/photos/          # 大頭照存放目錄
└── README.md
```
