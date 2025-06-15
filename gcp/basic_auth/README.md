# Basic Auth HTML Site on Cloud Run

GCP Cloud Run + Nginx で Basic 認証付き HTML を配信する最小構成。

---

## 📁 ディレクトリ構成

```
basic_auth/
├── Dockerfile
├── default.conf
├── html/
│   └── index.html
└── auth/
    └── .htpasswd  # 後述の手順で生成
```

---

## 📝 各ファイル内容

### Dockerfile

```dockerfile
FROM nginx:alpine
COPY default.conf /etc/nginx/conf.d/default.conf
COPY html/ /usr/share/nginx/html/
COPY auth/.htpasswd /etc/nginx/.htpasswd
EXPOSE 8080
```

### default.conf

```nginx
server {
    listen 8080;
    server_name _;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        auth_basic "Restricted";
        auth_basic_user_file /etc/nginx/.htpasswd;
        try_files $uri $uri/ =404;
    }
}
```

### html/index.html

```html
<!doctype html>
<html>
  <head><title>Hello</title></head>
  <body><h1>It works behind Basic Auth 🎉</h1></body>
</html>
```

---

## 🔐 Basic 認証ユーザーの生成

```sh
mkdir -p auth
docker run --rm -v "$(pwd)/auth:/work" httpd:2.4-alpine \
  htpasswd -cB -b /work/.htpasswd alice supersecret
```

---

## 🚀 GCP へのデプロイ

### 1. コンテナをビルドして登録

```sh
gcloud builds submit --tag gcr.io/$(gcloud config get-value project)/basic-site
```

### 2. Cloud Run へデプロイ

```sh
gcloud run deploy basic-site \
  --image gcr.io/$(gcloud config get-value project)/basic-site \
  --platform managed \
  --region asia-northeast1 \
  --allow-unauthenticated
```

---

## 🌐 独自ドメイン（任意）

```sh
gcloud beta run domain-mappings create \
  --service basic-site \
  --domain example.com
```

```sh
Waiting for certificate provisioning. You must configure your DNS records for certificate issuance to begin.
NAME            RECORD TYPE  CONTENTS
gcp-basic-auth  CNAME       ghs.googlehosted.com.
```

---

## ✅ ポイント

- TLS 終端：Cloud Run（Google-managed cert 自動更新）
- Basic 認証：Nginx 内で `.htpasswd` によって実施
- 証明書やHTTPS設定は一切不要
