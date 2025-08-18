---
title: "Docker"
slug: "docker"
order: 1
source: "docker"
---

## docke コマンド

```
//run
docker run イメージ

//run - port
docker run -p 80:80 -d httpd

//run - apache - 80port
docker run --name my_apache -p 80:80 -d httpd

//run - mysql - env


//レイヤー確認
docker inspect {{}} イメージ


//image to file
docker save nginx > nginx.tar

//file to load
docker load < nginx.tar

//タグで起動。名前もつける
docker run -d --name my-nginx-container my-nginx:v1

//exec

//tagが便利な点。
stg, prodとかで切れるから

//memory, cpu制御して起動
docker run -d --name nginx-limited --memory 256m --cpus 0.5 nginx

//削除
docker rm nginx-custom

//envを使って再ビルド
docker build -t my-nginx-env .
docker build -t polyglot-whale .

//レイヤーをhistoryで確認したい。
docker build コンテナ名

```

## Dockerfile

```
//ENVを設定
ENV WHALE_LANGUAGE="English"

//COPYをする
COPY entrypoint.sh /entrypoint.sh

//権限を与える
RUN chmod +x /entrypoint.sh

//echoでhellowolrd
CMD ["echo",  "Hello, World!"]

//pipinstallしてみる
RUN pip install --no-cache-dir -r requirements.txt




```

##　マルチステージビルド

### ゴミが含まれない、ベストプラクティス

```
# Build stage
# ここは設定だけ
FROM python:3.9-slim AS builder

WORKDIR /app

COPY requirements.txt .

RUN pip install --user --no-cache-dir -r requirements.txt

# Final stage
# ここが実際に使われる
FROM python:3.9-slim

WORKDIR /app

# Copy only the installed packages from the builder stage
COPY --from=builder /root/.local /root/.local
COPY app.py .

ENV PATH=/root/.local/bin:$PATH
ENV ENVIRONMENT=production

CMD ["python", "app.py"]

EXPOSE 5000

LABEL maintainer="Your Name <your.email@example.com>"
LABEL version="1.0"
LABEL description="Flask app demo with multi-stage build"
```

## 他にも高度な Dockrefile の使い方

```
//ユーザ作成
RUN useradd -m appuser

//色々入れる。あとapt キャッシュを削減
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

//Python バージョンを動的に決定
RUN PYTHON_VERSION=$(python -c 'import sys; print(f"{sys.version_info.major}.{sys.version_info.minor}")') && ...

//マルチステージビルド
COPY --from=builder /root/.local/lib/python3.9/site-packages "${SITE_PACKAGES_PATH}"

//実行ファイルをコピー
COPY --from=builder /root/.local/bin /home/appuser/.local/bin

```

## .dockerignore で不要なファイルを除外

## network を作成

```
//作成 custom bridge create
docker network create --driver bridge my-network

//一覧
docker network ls

//コンテナ立ててネットワークに接続
docker run --name container1 --network my-network -d nginx

//ネットワークへ接続
docker network connect my-network2 container2

//切断
docker network disconnect my-network2 container2

//ネットワークに接続されてるコンテナがあるか確認
docker network inspect my-network2

//netowrk削除
docker network rm my-network2


//同ネットワークエイリアス上にコンテナ作成



```

## volume

```

//volume作成


//volume ls
docker volume ls

//volume mountでコンテナを起動
docker run -d --name my_container -v my_data:/app/data ubuntu:latest sleep infinity

//backupを取るためにコンテナを削除する
docker rm my_container

//backupするときに一時的なコンテナを使うのが良い
//Dockerボリューム（my_data）の中身をホスト側のディレクトリにバックアップ
docker run --rm -v my_data:/source:ro -v $(pwd):/backup ubuntu tar cvf /backup/my_data_backup.tar -C /source.

//
docker run -d --name volume_mounter -v data_volume:/app alpine sh -c "echo 'Hello, Docker volumes.' > /app/hello.txt && tail -f /dev/null"


```

## kubanetes

```
//clusterの状態確認
kubectl cluster-info

//クラスタが正常起動してるか確認する。1. クラスタのノードを確認
kubectl get nodes

//2.  すべてのポッドの状態を確認 でもいい
kubectl get pods --all-namespaces

//cluster up (minikube)
minikube start


```
