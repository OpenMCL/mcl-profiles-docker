# MCL Profiles (docker)

一個用來練習 docker 基本操作的小專案。每個人建立自己的個人詳細介紹頁面，並在首頁加上自己的連結卡片。

核心目標：練習 docker 操作、練習 docker compose 操作、運用 Dockerfile 製作 image

## 快速開始

### Step 1：Clone 專案

```bash
git clone git@github.com:OpenMCL/mcl-profiles-docker.git
cd mcl-profiles-docker
```

### Step 2：安裝 Docker 指令

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

> [!NOTE]
> **幾個補充提醒：**
> - 裝完後預設只有 root 能跑 docker，如果你想讓一般使用者也能用，要加進 `docker` 群組：
>   ```bash
>   sudo usermod -aG docker $USER
>   ```
>   然後重新登入才會生效。
> - 這個腳本適合開發/測試環境快速部署。Docker 官方自己也說，正式生產環境建議用手動安裝步驟，這樣你對裝了什麼有更明確的掌控。
> - 如果之後需要確認版本：
>   ```bash
>   docker --version
>   docker compose version
>   ```

## Docker

### Step 3：撰寫 Dockerfile

在專案根目錄下建立一個名為 Dockerfile 的檔案（注意拼字且無副檔名），內容如下：

```bash
# 使用 Nginx 最輕量的版本作為地基
FROM nginx:alpine

# 將本地的網頁檔案搬進 Nginx 的預設路徑
COPY . /usr/share/nginx/html

# 聲明服務埠號為 80
EXPOSE 80
```

### Step 4：Build Image

```bash
# -t 代表標記名稱，最後的「.」代表當前目錄
docker build -t mcl-docker-image:v1 .

# 顯示現在有什麼image
docker images
```

### Step 5：Run Container

```bash
# -d: 背景執行, --rm: 停止後自動刪除容器, -p: 電腦 8080 對接容器 80
docker run -d --rm -p 8080:80 --name docker-web mcl-docker-image:v1
```

### Step 6：查看成果

在瀏覽器輸入：`http://<你的 VM IP>:8080`

### Step 7： docker 基本操作

以下是幾個常見的 Docker 指令：

- **啟動容器**：`docker run` 加上 Image 名稱來啟動一個新的容器。
- **查看容器**：`docker ps` 查看正在運行的容器，加上 `-a` 參數可以看到所有容器（包括已停止的）。
- **停止容器**：`docker stop` 停止運行中的容器。
- **重啟容器**：`docker restart` 重新啟動容器。
- **刪除容器**：`docker rm` 刪除已停止的容器。
- **進入容器**：`docker exec -it` 進入一個正在運行的容器中進行操作。
- **查看日誌**：`docker logs` 加上容器 ID 或名稱，可以查看容器的運行日誌，對於 Debug 非常有幫助。

## docker compose

### Step 8：撰寫 docker-compose.yml

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

### Step 9：啟動 container

```bash
docker-compose up -d
```

### Step 10：查看成果

在瀏覽器輸入：`http://<你的 VM IP>:8081`

### Step 11： docker compose 基本操作

以下是幾個常見的 Docker Compose 指令：

- **一鍵啟動**：`docker compose up -d` 在背景執行所有服務。
- **查看狀態**：`docker compose ps` 列出目前專案相關的容器狀態。
- **停止並移除**：`docker compose down` 停止並移除所有相關容器與網路。
- **重新啟動**：`docker compose restart` 重啟所有服務。
- **進入容器**：`docker compose exec [服務名稱] sh` 進入指定的服務容器中。
- **查看日誌**：`docker compose logs -f` 即時追蹤（Follow）容器的日誌輸出。

## 進階補充 1：docker volumes (即時開發模式)

在開發時，如果每次修改 HTML 都要重新 `docker build` 會非常沒效率。我們可以使用 **Volume** 功能，將本地的資料夾直接「掛載」到容器內，達成**修改存檔，網頁立即更新**的效果。

docker command:

```bash
# -v 代表掛載路徑：[本地絕對路徑]:[容器內路徑]
# $(pwd) 會自動抓取你目前所在的目錄
docker run -d --rm -p 8080:80 -v $(pwd):/usr/share/nginx/html --name docker-dev mcl-docker-image:v1
```

docker-compose.yml 更改:

```bash
version: '3.8'

services:
  web:
    image: nginx:alpine
    container_name: docker-compose-web
    ports:
      - "8081:80"
    volumes:
      # [本地路徑]:[容器內部的 Nginx 網頁路徑]
      # 假設你的 docker-compose.yml 跟 index.html 放在同一個資料夾
      - .:/usr/share/nginx/html
    restart: always
```

## 進階補充 2：使用 .dockerignore

當我們執行 `docker build` 時，docker 會將當前目錄下的所有檔案發送到 docker daemon。為了避免將不必要的檔案（如 `.git`、`README.md` 或本地開發工具的設定）包進 image 中，我們可以建立一個 `.dockerignore` 檔案。

這可以讓 image 體積更小，並加快 build 的速度。

### 如何實作：

在專案根目錄建立 `.dockerignore`，並填入以下內容：

```text
.git
.gitignore
README.md
Dockerfile
docker-compose.yml
```

## 進階補充 3：發佈 image 到 GHCR

大家都有 GitHub 帳號，我們可以直接將 image 上傳到 **GitHub Container Registry (GHCR)**。請按照以下步驟操作：

1.  **登入 GHCR**：
    我們需要使用 GitHub 的 **個人存取權杖 (PAT)** 來登入：
    1.  至 GitHub [Settings > Developer settings > Personal access tokens (classic)](https://github.com/settings/tokens) 產生一個 Token。
    2.  點擊 **Generate new token (classic)**，Note 隨意填寫，並勾選 **`write:packages`** 權限後點擊產生。
    3.  複製該 Token (`ghp_` 開頭的字串)。
    4.  回到終端機執行登入：
        ```bash
        docker login ghcr.io -u <你的 GitHub 帳號>
        # 密碼處請貼上剛才複製的 Token
        ```

> [!TIP]
> 如果登入失敗，可以先執行 `docker logout ghcr.io` 清除舊的快取。

2.  **重新標記 image (tag)**：
    將你的本地 image 標記為 GHCR 的路徑。**請將 tag 命名為你的名字**。

    ```bash
    # 格式：docker tag [本地 image] ghcr.io/[帳號或組織名]/[專案名]:[你的名字]
    docker tag mcl-docker-image:v1 ghcr.io/openmcl/mcl-profiles-docker:<你的名字>
    ```

3.  **推送到 GitHub**：
    ```bash
    docker push ghcr.io/openmcl/mcl-profiles-docker:<你的名字>
    ```

> [!IMPORTANT]
> 推送成功後，你可以在這個 GitHub Repository 的 **Packages** 頁面看到大家上傳的 image！

## 進階補充 4：常用清理與監控指令

- **查看資源佔用**：`docker stats` (可以看到各容器的 CPU/記憶體使用量)。
- **一鍵大掃除**：`docker system prune` (清理所有停止的容器、沒用的網路與未被標記的 image，釋放硬碟空間)。

---

## 檔案結構

```
mcl-profiles-docker/
├── Dockerfile              # 定義如何建立 docker image 的說明文件
├── docker-compose.yml      # 定義與啟動 docker 容器的設定檔
├── .dockerignore           # 排除不需要包進 image 的檔案 (選用)
├── index.html              # 團隊首頁
├── styles.css              # 共用樣式
├── profiles/
│   ├── template.html       # 個人頁面模板
│   └── yourname.html       # 個人詳細介紹頁面
├── assets/photos/          # 大頭照存放目錄
└── README.md
```
