# 나는 무한도전 누구? — 기획안

## Context
쓸데없지만 한 번 만들어보고 싶은 재미용 웹앱. "무한도전 멤버 매칭 테스트"를 하나만 빠르게 만들어 반응을 본 뒤 확장한다.
제약: 비용 0원(런타임 AI/Claude API 미사용), Next.js 웹, 링크 공유가 핵심. 개발 작업은 Claude 클라우드 세션에서 진행.

## 제품 정의
- 질문 8~10개(각 2~4지선다)에 답하면 **별명 캐릭터(persona)** 1개와 매칭된 결과 카드가 나온다.
- persona = 멤버 + 별명 변형. 같은 멤버도 여러 persona를 가진다. 예: "거성 박명수", "쩌리짱 정준하", "정총무"(정준하의 다른 변형 등).
- 대상: 전성기 핵심 멤버 7~8명(유재석, 박명수, 정준하, 정형돈, 노홍철, 하하, 길 등), persona 총 20개 안팎.
- 결과 URL(`/r/[personaId]`)을 공유하면 미리보기 카드(OG 이미지)가 뜬다.
- 결과 화면: persona 별명, 한 줄 설명, 닮은 점 3개, 찰떡/상극 persona, 다시하기, 링크 복사.
- 톤: 사용자가 별명 중심으로 방향을 정함. 놀리는 드립은 별명 자체의 느낌을 따르되 과하게 비하하지 않는다.

## 핵심 결정
1. **채점**: 각 선택지가 persona별 점수를 올리고 합산 최고점이 결과. 동점이면 마지막 질문 가중치 우선. 순수 함수 `scoreAnswers()`로 단위 테스트.
2. **이미지/저작권**: 방송 캡처·실제 사진은 저장소에 넣지 않는다. 사용자가 직접 구한 이미지를 `public/personas/<personaId>.jpg`에 넣으면 표시. 없으면 색상 + 이모지 자리표시를 쓴다. 이미지 사용 책임은 사용자에게 있음을 README에 명시.
3. **상태**: 서버 DB 없음. 결과는 URL 경로만, 진행 상태는 클라이언트 로컬 상태.
4. **OG 이미지**: `next/og` `ImageResponse`로 `/r/[personaId]/opengraph-image`.
5. **배포**: Vercel 무료 플랜.

## 폴더 구조 (feature-first)
```
app/
  page.tsx                        # 랜딩 + 시작
  quiz/page.tsx                   # 퀴즈 진행
  r/[personaId]/page.tsx          # 결과 (정적 생성)
  r/[personaId]/opengraph-image.tsx
features/quiz/
  domain/        # Persona, Question, Choice 타입, scoreAnswers()
  data/          # personas.ts, questions.ts
  presentation/  # QuizScreen, ResultCard, ShareButton, 진행바, 퀴즈 상태
core/            # 공통 상수, 카피 문자열
public/personas/ # 사용자가 넣는 이미지
```

## 구현 순서
1. Next.js(App Router, TypeScript strict, Tailwind) 생성.
2. 도메인 타입 + `scoreAnswers()` + 테스트(정상, 동점, 일부 미응답).
3. persona 목록과 질문 초안 작성(사용자와 함께 다듬음).
4. 퀴즈 UI: 한 화면 한 질문, 44pt 이상 터치 영역, 진행바, 뒤로가기.
5. 결과 페이지 + 링크 복사 + OG 이미지.
6. 접근성, 모바일 우선 레이아웃.
7. Vercel 배포.

## 검증
- `scoreAnswers` 단위 테스트 통과.
- 모든 persona가 최소 한 가지 답변 조합으로 도달 가능한지 테스트(결과 쏠림 방지).
- 로컬 실행 후 375px 폭에서 시작→결과→공유 링크 열기 확인.
- 결과 URL 직접 접속 시 OG 이미지 생성 확인.
