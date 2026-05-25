# KET 데이터베이스 설계 문서
**K-Entertainment Tier — Data Schema v1.0**
작성일: 2024-05-24

---

## 1. 개요

KET 웹사이트의 예능인 데이터를 구조화하기 위한 스키마 설계 문서.  
현재는 JSON 파일 기반 정적 운용을 목표로 하며, 추후 백엔드 도입 시 관계형 DB로 마이그레이션 가능하도록 설계한다.

---

## 2. 도메인 상수 정의

### 2-1. 티어 (Tier)

| 코드 | 명칭 | 설명 |
|------|------|------|
| `S` | 예능 대장군 | 혼자서 예능 프로그램 단독으로 진행 가능한 수준 |
| `A` | 예능 장군 | 예능 프로그램 MC 가능한 수준, 단독 진행은 부족 |
| `B` | 예능 용병 | 게스트 및 고정 멤버 모두 참여 가능, 제 몫의 웃음분량 만들 수 있는 수준 |
| `C` | 예능 병사 | 게스트로 참여하여 간간히 웃음을 만들어 낼 수 있는 수준, 고정멤버로는 부족 |
| `D` | 예능 훈련병 | 예능의 의지와 열정이 있는 수준 |

### 2-2. 유형 (Type)

| 코드 | 명칭 | 설명 |
|------|------|------|
| `muryok` | 무력형 | 개인기·기세·신체적 임팩트로 웃음을 만드는 유형. 에너지 자체가 콘텐츠가 된다. |
| `jiryak` | 지략형 | 꽁트 및 설계, 상황 판단으로 웃음을 설계하는 유형. 머리로 판을 짠다. |
| `bonneung` | 본능형 | 즉흥·반응·감각으로 웃음을 만드는 유형. 계획 없이 터뜨리는 것이 강점이다. |

### 2-3. 포지션 (Position)

| 코드 | 명칭 | 설명 |
|------|------|------|
| `dealer` | 딜러 | 직접 웃음을 생산하는 포지션. 공격력이 핵심이다. |
| `tanker` | 탱커 | 맞아주고 버텨주며 상황을 유지하는 포지션. 내구성이 핵심이다. |
| `supporter` | 서포터 | 다른 출연자의 웃음을 받쳐주는 포지션. 보이지 않는 기여가 핵심이다. |
| `stealer` | 스틸러 | 상황을 가로채 임팩트 있는 한 방을 날리는 포지션. 타이밍이 핵심이다. |

---

## 3. 엔티티 설계

### 3-1. Entertainer (예능인)

핵심 엔티티. 티어표의 각 카드 하나에 해당한다.

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| `id` | string | ✅ | 고유 식별자. 이름 기반 slug (예: `kang-hodong`) |
| `name` | string | ✅ | 이름 (한글) |
| `name_en` | string | | 영문 이름 |
| `tier` | enum(S/A/B/C/D) | ✅ | 현재 티어 |
| `type` | enum(muryok/jiryak/bonneung) | ✅ | 예능 유형 |
| `position` | enum(dealer/tanker/supporter/stealer/mc) | ✅ | 예능 포지션 |
| `initial` | string | ✅ | 이름 첫 글자 (프로필 카드 폴백 표시용) |
| `profile_image` | string | | 프로필 이미지 파일명 (예: `강호동.jpg`) |
| `debut_year` | number | | 예능 데뷔 연도 |
| `active` | boolean | ✅ | 현재 활동 여부 |
| `description` | string | | 1~2줄 소개 |
| `analysis` | string | | 상세 분석 텍스트 (detail 페이지용) |
| `stats` | object | | 6개 스탯 수치 (아래 Stats 스키마 참고) |
| `programs` | string[] | | 대표 출연 프로그램 목록 |
| `strengths` | string[] | | 강점 키워드 목록 |
| `weaknesses` | string[] | | 약점 키워드 목록 |
| `rep_score` | number | | 대표성 점수 (0–100, 티어 내 순위 산정용) |
| `tags` | string[] | | 검색·필터용 태그 |
| `tier_history` | TierHistory[] | | 티어 변경 이력 |
| `updated_at` | string | ✅ | 마지막 데이터 수정일 (ISO 8601) |

#### Stats 스키마

6개 지표는 유형/포지션과 별개로 측정되는 예능 어빌리티 수치다.

| 필드 키 | 표시명 | 측정 기준 | 강세 포지션 |
|---------|--------|-----------|------------|
| `gaein` | 개인기 | 단독 퍼포먼스, 고유 기술 | 개그맨·캐릭터형 |
| `sunbal` | 순발력 | 즉흥 받아치기 속도 | 딜러·스틸러 |
| `pandan` | 판단력 | 메타인지, 웃긴 선택 감각 | 전 포지션 변별기 |
| `gihoek` | 기획력 | 꽁트·상황 설계, 구조 구성 | 개그맨 출신 |
| `reaction` | 리액션 | 타인에 반응하는 질·타이밍 | 서포터·탱커 |
| `eonbyeon` | 언변 | 말 구사력, 위트, 표현 | MC·진행형 |

```json
"stats": {
  "gaein":    0,   // 개인기   0–100
  "sunbal":   0,   // 순발력   0–100
  "pandan":   0,   // 판단력   0–100
  "gihoek":   0,   // 기획력   0–100
  "reaction": 0,   // 리액션   0–100
  "eonbyeon": 0    // 언변     0–100
}
```

#### TierHistory 스키마

```json
{
  "tier":    "A",
  "date":    "2023-01-01",
  "reason":  "무한도전 종영 이후 활동량 감소"
}
```

---

### 3-2. LegendClip (레전드 클립)

레전드관 페이지에 표시되는 명장면 클립 데이터.

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| `id` | string | ✅ | 고유 식별자 |
| `rank` | number | ✅ | 레전드 순위 |
| `title` | string | ✅ | 클립 제목 |
| `description` | string | | 클립 설명 |
| `entertainer_ids` | string[] | ✅ | 출연 예능인 id 목록 |
| `program` | string | ✅ | 프로그램명 |
| `year` | number | | 방영 연도 |
| `youtube_url` | string | | YouTube 영상 URL |
| `thumbnail` | string | | 썸네일 이미지 파일명 |
| `duration` | string | | 영상 길이 (예: `3:24`) |
| `views` | number | | 조회수 |
| `category` | enum | ✅ | 카테고리 (토크쇼/관찰예능/게임예능/리액션/개그) |

---

### 3-3. YoutubeVideo (유튜브 영상)

유튜브 페이지에 표시되는 KET 채널 영상 데이터.

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| `id` | string | ✅ | 고유 식별자 |
| `title` | string | ✅ | 영상 제목 |
| `description` | string | | 영상 설명 |
| `youtube_url` | string | ✅ | YouTube 영상 URL |
| `youtube_id` | string | | YouTube 영상 ID (URL에서 추출) |
| `thumbnail` | string | | 썸네일 이미지 파일명 |
| `duration` | string | | 영상 길이 |
| `views` | number | | 조회수 |
| `upload_date` | string | ✅ | 업로드일 (ISO 8601) |
| `series` | string | | 시리즈 카테고리 (티어 분석/비교 분석/예능 부검 등) |
| `featured_ids` | string[] | | 주요 출연 예능인 id 목록 |
| `is_featured` | boolean | | 홈 히어로 섹션 노출 여부 |

---

## 4. JSON 파일 구조

정적 사이트에서 사용할 JSON 파일 구성.

```
yessir/
└── data/
    ├── entertainers.json    ← 예능인 전체 데이터
    ├── legend_clips.json    ← 레전드 클립 데이터
    └── youtube_videos.json  ← 유튜브 영상 데이터
```

---

## 5. entertainers.json 샘플

```json
{
  "entertainers": [
    {
      "id": "kang-hodong",
      "name": "강호동",
      "name_en": "Kang Ho-dong",
      "tier": "S",
      "type": "muryok",
      "position": "tanker",
      "initial": "강",
      "profile_image": "강호동.jpg",
      "debut_year": 1993,
      "active": true,
      "description": "기세와 에너지로 예능판을 장악하는 무력형 탱커의 원조.",
      "analysis": "강호동은 신체적 존재감과 압도적인 기세로 상황을 통제한다...",
      "stats": {
        "gaein":    92,
        "sunbal":   95,
        "pandan":   80,
        "gihoek":   62,
        "reaction": 90,
        "eonbyeon": 84
      },
      "programs": ["1박2일", "무릎팍도사", "아는 형님", "강심장"],
      "strengths": ["압도적 기세", "순발력", "감정 폭발"],
      "weaknesses": ["세밀한 조율 부족", "게스트 배려"],
      "rep_score": 97,
      "tags": ["무한도전", "1박2일", "씨름", "레슬링"],
      "tier_history": [
        { "tier": "S", "date": "2024-01-01", "reason": "현역 유지" }
      ],
      "updated_at": "2024-05-24"
    },
    {
      "id": "yoo-jaesuk",
      "name": "유재석",
      "name_en": "Yoo Jae-suk",
      "tier": "S",
      "type": "jiryak",
      "position": "supporter",
      "initial": "유",
      "profile_image": "유재석.jpg",
      "debut_year": 1991,
      "active": true,
      "description": "20년간 예능판 정상을 지킨 지략형 MC의 교과서.",
      "analysis": "유재석의 예능 전략은 철저한 상황 관리와 게스트 배려에 있다...",
      "stats": {
        "gaein":    72,
        "sunbal":   88,
        "pandan":   96,
        "gihoek":   90,
        "reaction": 85,
        "eonbyeon": 97
      },
      "programs": ["무한도전", "런닝맨", "유퀴즈", "놀면 뭐하니"],
      "strengths": ["상황 설계", "게스트 배려", "위기 대처"],
      "weaknesses": ["즉흥 폭발력 부족"],
      "rep_score": 99,
      "tags": ["무한도전", "런닝맨", "유퀴즈"],
      "tier_history": [
        { "tier": "S", "date": "2024-01-01", "reason": "현역 유지" }
      ],
      "updated_at": "2024-05-24"
    }
  ]
}
```

---

## 6. 관계형 DB 설계 (향후 확장)

백엔드 도입 시 아래 ERD 기준으로 마이그레이션한다.

```
Entertainer (1) ──< TierHistory (N)
Entertainer (1) ──< EntertainersPrograms (N) >── Program (1)
Entertainer (1) ──< LegendClipEntertainer (N) >── LegendClip (1)
Entertainer (1) ──< VideoEntertainer (N) >── YoutubeVideo (1)
```

### 테이블 목록

| 테이블 | 설명 |
|--------|------|
| `entertainers` | 예능인 기본 정보 |
| `entertainer_stats` | 예능인 스탯 (1:1) |
| `tier_history` | 티어 변경 이력 |
| `programs` | 프로그램 마스터 |
| `entertainer_programs` | 예능인-프로그램 중간 테이블 |
| `legend_clips` | 레전드 클립 |
| `legend_clip_entertainers` | 클립-예능인 중간 테이블 |
| `youtube_videos` | 유튜브 영상 |
| `video_entertainers` | 영상-예능인 중간 테이블 |

---

## 7. 현재 등록 예능인 현황

| 티어 | 예능인 | 유형 | 포지션 |
|------|--------|------|--------|
| S | 강호동 | 무력형 | 딜러 |
| S | 유재석 | 지략형 | 딜러 |
| S | 이경규 | 무력형 | 딜러 |
| S | 신동엽 | 본능형 | 딜러 |
| A | 장동민 | 본능형 | 딜러 |
| A | 탁재훈 | 본능형 | 딜러 |
| A | 김구라 | 지략형 | 딜러 |
| A | 허경환 | 무력형 | 탱커 |
| A | 유세윤 | 무력형 | 딜러 |
| A | 전현무 | 지략형 | 서포터 |
| A | 서장훈 | 지략형 | 탱커 |
| A | 이수근 | 본능형 | 딜러 |
| A | 하하 | 본능형 | 딜러 |
| A | 노홍철 | 본능형 | 딜러 |
| A | 김국진 | 지략형 | 서포터 |
| A | 정형돈 | 지략형 | 딜러 |
| B | 박명수 | 본능형 | 딜러 |
| B | 정준하 | 무력형 | 탱커 |
| B | 문세윤 | 무력형 | 딜러 |
| B | 김종민 | 본능형 | 스틸러 |
| B | 이광수 | 본능형 | 탱커 |
| B | 김영철 | 무력형 | 탱커 |
| B | 지상렬 | 본능형 | 스틸러 |
| B | 지석진 | 지략형 | 탱커 |
| B | 신기루 | 무력형 | 딜러 |
| B | 장도연 | 지략형 | 서포터 |
| B | 신봉선 | 무력형 | 탱커 |
| B | 조세호 | 본능형 | 탱커 |
| B | 김희철 | 무력형 | 딜러 |
| B | 데프콘 | 본능형 | 탱커 |
| C | 황광희 | 본능형 | 탱커 |
| C | 딘딘 | 본능형 | 딜러 |
| C | 이상민 | 무력형 | 탱커 |
| C | 민경훈 | 본능형 | 스틸러 |
| C | 길 | 무력형 | 스틸러 |
| D | 윤형빈 | 본능형 | 딜러 |

---

## 8. 다음 단계 권장사항

1. **`data/entertainers.json` 생성** — 위 샘플 기반으로 전체 36명 데이터 입력
2. **JS 데이터 로더 작성** — `fetch('data/entertainers.json')`으로 카드를 동적 렌더링 (HTML 하드코딩 제거)
3. **`detail.html` 동적화** — URL 파라미터(`?id=kang-hodong`)로 해당 예능인 데이터 로드
4. **프로필 이미지 수집** — `img/` 폴더에 각 예능인 사진 추가
5. **스탯 수치 입력** — 36명 전원의 6개 스탯 수치 결정 및 입력
