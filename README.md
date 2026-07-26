# 숏폼 인기영상 도우미 v1.3.0

대한민국의 한국어 숏폼 인기 영상을 수집·검토·순위화하고 Google Sheets와 Notion에 동기화하는 반응형 운영 사이트입니다.

## v1.3.0 핵심

- Notion 공개 연결(OAuth)을 이용한 계정 로그인
- 허용된 Notion 워크스페이스·사용자 ID·이메일 제한
- 로그인 사용자의 이름·워크스페이스·프로필 표시
- 영상 카드, 상위 후보, 플랫폼별 순위에 썸네일 표시
- YouTube 썸네일 자동 생성
- TikTok·Instagram Webhook의 `thumbnailUrl` 저장 및 표시
- Google Sheets `Raw_Content!AJ`에 `Thumbnail URL` 저장
- 썸네일을 불러오지 못하면 플랫폼별 안전한 대체 이미지 표시

## 인증 구조

Notion OAuth는 사용자의 Notion 계정과 워크스페이스를 확인하는 로그인 수단입니다. 실제 운영 데이터의 읽기·쓰기는 서버에 등록된 내부 Notion 연결 토큰과 Google 서비스 계정으로 수행합니다. 브라우저에는 Notion 토큰이나 Google 개인키가 노출되지 않습니다.

Notion 공식 공개 연결을 만든 뒤 다음 값을 배포 환경변수에 등록합니다.

```env
NOTION_OAUTH_ENABLED=true
PASSWORD_LOGIN_ENABLED=false
NOTION_OAUTH_CLIENT_ID=...
NOTION_OAUTH_CLIENT_SECRET=...
NOTION_OAUTH_REDIRECT_URI=https://your-domain.example/api/auth/notion/callback
NOTION_ALLOWED_WORKSPACE_IDS=...
NOTION_ALLOWED_USER_IDS=...
NOTION_ALLOWED_EMAILS=owner@example.com
```

운영 환경에서는 워크스페이스·사용자 ID·이메일 중 적어도 하나의 허용 목록이 필요합니다.

## 썸네일 데이터 계약

YouTube는 Data API의 썸네일을 우선 사용하며 없을 경우 영상 ID로 썸네일 주소를 만듭니다.

TikTok·Instagram 수집 Webhook은 다음 키 중 하나를 보낼 수 있습니다.

```json
{
  "thumbnailUrl": "https://...",
  "thumbnail_url": "https://...",
  "coverUrl": "https://...",
  "cover": "https://..."
}
```

수집된 주소는 Google Sheets `Thumbnail URL` 열에 저장됩니다.

## 실행

```bash
cp .env.example .env
set -a && source .env && set +a
npm start
```

개발 검증:

```bash
npm run check
npm test
npm audit --omit=dev
```

## 배포

Dockerfile과 Render Blueprint가 포함되어 있습니다. HTTPS 주소를 만든 후 그 주소를 Notion 페이지에 `/embed`로 삽입합니다. Notion OAuth의 Redirect URI는 실제 배포 도메인과 정확히 일치해야 합니다.
