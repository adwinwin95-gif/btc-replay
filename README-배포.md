# BTC 리플레이 트레이너 — 서버 배포

순수 정적 사이트입니다. 백엔드도 빌드도 필요 없고, 이 폴더를 그대로 올리면 됩니다.
전체 약 150MB. 첫 로딩은 3.3MB(1시간봉)뿐이고, 1·3·5·15·30분봉은
고른 연도의 1분봉 파일 하나(압축 약 5MB)만 추가로 받습니다.

## 1. Cloudflare Pages — 가장 간단 (무료)

1. https://dash.cloudflare.com → Workers & Pages → Create → Pages
2. "Upload assets" 선택하고 이 폴더를 통째로 드래그
3. 끝. `프로젝트이름.pages.dev` 주소가 바로 나옵니다

`_headers` 파일이 캐시 설정을 자동으로 잡아줍니다. gzip·brotli도 자동입니다.
CLI를 쓰려면: `npx wrangler pages deploy . --project-name btc-replay`

## 2. Netlify — 드래그 한 번 (무료)

https://app.netlify.com/drop 에 이 폴더를 드래그하면 끝입니다.
`_headers` 를 똑같이 인식합니다.

## 3. 직접 쓰는 VPS (nginx)

```bash
scp -r btc-replay-web/ 서버주소:/var/www/btc-replay
```

`nginx.conf.sample` 내용을 `/etc/nginx/sites-available/btc-replay` 로 복사하고,

```bash
ln -s /etc/nginx/sites-available/btc-replay /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
```

**gzip 을 꼭 켜세요.** JSON 이 2~3배로 줄어서 로딩이 눈에 띄게 빨라집니다.

## 4. Fly.io

`Dockerfile` 하나면 됩니다:

```dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
COPY nginx.conf.sample /etc/nginx/conf.d/default.conf
```

```bash
fly launch --no-deploy
fly deploy
```

## 5. 로컬에서 바로 확인

```bash
cd btc-replay-web
python -m http.server 8080
```

브라우저에서 http://localhost:8080

> `index.html` 을 더블클릭해서 `file://` 로 여는 건 **안 됩니다.**
> 브라우저가 로컬 파일 fetch 를 막기 때문입니다.
> 그 용도로는 바탕화면의 `BTC-리플레이-트레이너.html`(단일 파일)을 쓰세요.

## 데이터 출처

Bitstamp BTC/USD 실제 체결 데이터 (2012-01-01 ~ 2026-11-07).
1분봉을 5분봉 157만 개와 1시간봉 13만 개로 집계했습니다.
가격은 센트 단위 정수로 차분 인코딩되어 있어 원본 대비 40% 정도 작습니다.
