# Ollama + Open WebUI Docker Compose

使用 Docker Compose 快速部署 Ollama 與 Open WebUI，打造本地端 AI LLM 服務。

## 功能特色

-  一鍵部署 Ollama
- 🌐 Web 介面管理模型（Open WebUI）
- 🎮 支援 NVIDIA GPU 加速
- 💾 持久化模型與聊天資料
- 🔄 容器自動重啟
- 🐳 Docker Compose 管理

---

## 系統需求

### 必要條件

- Docker Engine 24+
- Docker Compose v2+
- NVIDIA GPU（選用）
- NVIDIA Container Toolkit

### 驗證 Docker

```bash
docker --version
docker compose version
```

### 驗證 GPU

```bash
nvidia-smi
```

---

## 專案結構

```text
.
├── docker-compose.yml
├── models/
└── README.md
```

---

## docker-compose.yml

```yaml
version: '3.9'

services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    restart: always
    ports:
      - "11434:11434"
    volumes:
      - ollama_data:/root/.ollama
    environment:
      - NVIDIA_VISIBLE_DEVICES=all
      - NVIDIA_DRIVER_CAPABILITIES=compute,utility
    networks:
      - ollama-net

  open-webui:
    image: ghcr.io/open-webui/open-webui:latest
    container_name: open-webui
    restart: always
    command: bash start.sh --host 0.0.0.0 --port 5000
    ports:
      - "5000:5000"
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
      - NVIDIA_VISIBLE_DEVICES=all
      - NVIDIA_DRIVER_CAPABILITIES=compute,utility
      - BIND_HOST=0.0.0.0
      - USE_CPU_ONLY=false
      - LOAD_8BIT=false
      - MAX_TOKENS=1024
    volumes:
      - webui_data:/app/backend/data
      - ./models:/app/backend/data/models
    depends_on:
      - ollama
    networks:
      - ollama-net

networks:
  ollama-net:

volumes:
  ollama_data:
  webui_data:
```

---

## 啟動服務

### 下載專案

```bash
git clone https://github.com/IsuzuRen5000/ollama-openwebui-docker.git
cd ollama-openwebui-docker.git
```

### 啟動容器

```bash
docker compose up -d
```

### 查看狀態

```bash
docker ps
```

### 查看日誌

```bash
docker compose logs -f
```

---

## 使用方式

### Open WebUI

瀏覽器開啟：

```text
http://localhost:5000
```

首次登入需建立管理員帳號。

### Ollama API

```text
http://localhost:11434
```

測試 API：

```bash
curl http://localhost:11434/api/tags
```

---

## 下載模型

進入 Ollama 容器：

```bash
docker exec -it ollama bash
```

安裝模型：

### Llama 3

```bash
ollama pull llama3
```

### Qwen 3

```bash
ollama pull qwen3
```

### DeepSeek R1

```bash
ollama pull deepseek-r1
```

### Gemma

```bash
ollama pull gemma3
```

查看已安裝模型：

```bash
ollama list
```

---

## GPU 驗證

確認 Ollama 可偵測 GPU：

```bash
docker exec -it ollama nvidia-smi
```

若顯示 NVIDIA 資訊則代表 GPU 已成功掛載。

---

## 停止服務

```bash
docker compose down
```

停止並刪除 Volume：

```bash
docker compose down -v
```

---

## 資料持久化

| Volume | 用途 |
|----------|----------|
| ollama_data | Ollama 模型資料 |
| webui_data | Open WebUI 使用者資料 |

---

## 常見問題

### Open WebUI 無法連線 Ollama

確認：

```bash
docker exec -it open-webui ping ollama
```

以及：

```bash
docker logs ollama
```

---

### GPU 沒有被使用

確認：

```bash
nvidia-smi
```

並安裝：

- NVIDIA Driver
- NVIDIA Container Toolkit

---

## License

MIT License
