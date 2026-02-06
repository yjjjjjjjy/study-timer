# 공부 타이머 (Study Timer)

Nuxt.js로 만든 공부 타이머 앱입니다.

## 기능

- ⏰ 준비 시간: 5초
- 📚 공부 시간: 50초 (실제 사용 시 50분)
- ☕ 휴식 시간: 10초 (실제 사용 시 10분)
- 🔄 10세트 자동 반복
- 📊 세트 진행도 게이지
- ⏸️ 일시정지/재개 기능
- 🔔 단계 전환 시 알림음

## 설치 및 실행

```bash
# 의존성 설치
npm install

# 개발 서버 실행 (http://localhost:3000)
npm run dev

# 프로덕션 빌드
npm run build

# 정적 사이트 생성
npm run generate
```

## Vercel 배포 방법

1. GitHub에 코드 푸시
2. [Vercel](https://vercel.com) 로그인
3. "New Project" 클릭
4. GitHub 저장소 선택
5. Framework Preset: "Nuxt.js" 선택
6. "Deploy" 클릭

또는 Vercel CLI 사용:

```bash
npm install -g vercel
vercel
```

## 설정 변경

실제 사용을 위해 타이머 시간을 변경하려면 `app.vue` 파일의 다음 부분을 수정하세요:

```javascript
const PREPARE_TIME = 5     // 5초 → 5분으로 변경 시: 300
const STUDY_TIME = 50      // 50초 → 50분으로 변경 시: 3000
const BREAK_TIME = 10      // 10초 → 10분으로 변경 시: 600
```
