# 아키텍처 문서 — DirectCloud FAQ 자동 분석 도구

> 워크숍 5단계 산출물

---

## 1. 시스템 전체 구조

```mermaid
graph TD
    A[브라우저<br/>index.html] -->|API Key 입력| B[Backlog REST API]
    A -->|문의 텍스트 전달| C[Claude API<br/>claude-sonnet-4]
    B -->|최근 12개월 문의 수집| A
    C -->|적합도 점수 + 카테고리<br/>그룹화 + FAQ 초안| A

    style A fill:#1e2333,stroke:#3b7ef8,color:#e2e8f4
    style B fill:#1e2333,stroke:#5c6a85,color:#e2e8f4
    style C fill:#1e2333,stroke:#22c55e,color:#e2e8f4
```

---

## 2. 데이터 흐름

```mermaid
sequenceDiagram
    actor 사용자 as 브릿지 엔지니어
    participant UI as index.html
    participant BL as Backlog API
    participant CL as Claude API

    사용자->>UI: 분석 실행 버튼 클릭
    UI->>BL: GET /api/v2/issues<br/>(프로젝트: DIRECTCLOUDINQUIRY)
    BL-->>UI: 문의 목록 반환 (최근 12개월)
    UI->>UI: 기간 필터 적용
    UI->>CL: 문의 텍스트 전달<br/>(FAQ 분석 요청)
    CL-->>UI: 적합도 점수 + 카테고리<br/>유사 문의 그룹 + FAQ 초안
    UI->>사용자: 결과 화면 표시
```

---

## 3. 주요 컴포넌트

```mermaid
graph LR
    subgraph HTML ["index.html (단일 파일)"]
        UI[UI 레이어<br/>카드 목록 / 필터 / 사이드바]
        BL_MOD[Backlog 모듈<br/>API 호출 + 기간 필터]
        CL_MOD[Claude 모듈<br/>분석 요청 + 응답 파싱]
        STATE[상태 관리<br/>문의 데이터 + 분석 결과]
    end

    UI <-->|렌더링| STATE
    BL_MOD -->|문의 저장| STATE
    CL_MOD -->|분석 결과 저장| STATE
    UI -->|실행 트리거| BL_MOD
    BL_MOD -->|텍스트 전달| CL_MOD

    style HTML fill:#0f1117,stroke:#2a3045,color:#e2e8f4
    style UI fill:#1e2333,stroke:#3b7ef8,color:#e2e8f4
    style BL_MOD fill:#1e2333,stroke:#5c6a85,color:#e2e8f4
    style CL_MOD fill:#1e2333,stroke:#22c55e,color:#e2e8f4
    style STATE fill:#1e2333,stroke:#f59e0b,color:#e2e8f4
```

---

## 4. Claude API 분석 프로세스

```mermaid
flowchart TD
    A[문의 텍스트 입력] --> B{FAQ 적합 여부 판단}
    B -->|적합도 1~2점| C[제외 — 개별 이슈]
    B -->|적합도 3~5점| D[카테고리 분류]
    D --> E[유사 문의 그룹화]
    E --> F{그룹 크기}
    F -->|1건| G[단독 FAQ 후보]
    F -->|2건 이상| H[묶음 FAQ 후보<br/>점수 가중치 상승]
    G --> I[FAQ 초안 Q&A 생성]
    H --> I
    I --> J[결과 화면 표시]

    style A fill:#1e2333,stroke:#3b7ef8,color:#e2e8f4
    style C fill:#1e2333,stroke:#ef4444,color:#e2e8f4
    style J fill:#1e2333,stroke:#22c55e,color:#e2e8f4
```

---

## 5. 기술 스택 요약

| 항목 | 기술 | 비고 |
|------|------|------|
| 실행 환경 | 브라우저 (Chrome 권장) | 서버 설치 불필요 |
| 프론트엔드 | HTML + CSS + Vanilla JS | 단일 파일 |
| AI 엔진 | Claude API (claude-sonnet-4) | FAQ 분석 전담 |
| 데이터 소스 | Backlog REST API v2 | DIRECTCLOUDINQUIRY 프로젝트 |
| 스타일 | CSS Variables | DirectCloud 다크 테마 |
| 버전 관리 | Git + GitHub | my-vibe-project 레포 |

---

## 6. 보안 고려사항

- **API 키 관리:** Backlog API 키 및 Claude API 키는 브라우저 내 입력 방식으로 처리 — 소스코드에 하드코딩 금지
- **데이터 범위:** 분석 대상은 DIRECTCLOUDINQUIRY 프로젝트 내 문의로 한정
- **단독 사용 도구:** 외부 공개 없이 로컬에서만 실행
