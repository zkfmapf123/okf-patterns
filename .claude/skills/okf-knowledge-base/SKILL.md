---
name: okf-knowledge-base
description: >
  지식을 OKF(Open Knowledge Format) 형식으로 저장·갱신·검색할 때 사용한다.
  구체적으로 (1) 대화에서 나온 결론·절차·결정·참고자료를 ~/kb/ 에 정리/저장할 때,
  (2) 기존 KB 문서를 수정/갱신할 때, (3) KB에서 정보를 검색·참조할 때 이 스킬을
  로드하여 형식 규칙과 저장 위치 규칙, 안전 규칙을 따른다.
  AWS/DevOps 등 팀 지식 베이스를 다루는 모든 작업에 적용된다.
---

# OKF Knowledge Base Skill

## OKF란 (이름 의존 없는 자기완결 설명)
OKF는 지식을 "YAML 프론트매터가 붙은 마크다운 파일들의 디렉토리"로 표현하는
개방형 형식이다. 새 언어·런타임·SDK가 아니라, 아래 규칙을 따르는 평범한 마크다운
파일 모음이다. "OKF"라는 이름을 몰라도 이 문서의 규칙만 따르면 된다.

## 저장 위치 규칙
- KB 루트: `~/kb/`
  - `~/kb/common/` — 팀 공용 KB. **직접 수정·삭제 금지. 읽기(참조)만 허용.**
  - `~/kb/local/`  — 개인 로컬 KB. 모든 신규 저장의 기본 위치.
- 별도 지정이 없으면 저장은 항상 `~/kb/local/` 에 한다.
- common 보강 내용은 `~/kb/local/` 에 초안으로 저장 후, 사용자에게 승격(PR)을 안내한다.

## 핵심 개념
- 파일 하나 = 개념 하나.
- **파일 경로 = 개념 ID.** 예: `aws/standard-vpc.md` → `aws/standard-vpc`.
- 개념 간 관계는 마크다운 링크, 번들 루트 기준 **절대경로**(`/...`) 권장.

## 문서 구조
1. YAML 프론트매터 (`---` 구분)
   - 필수: `type`
   - 권장: `title`, `description`, `resource`, `tags`, `timestamp`
2. 구조화된 마크다운 본문 (헤딩·표·리스트·코드블록 선호)
   - 관례 헤딩: `# Schema`, `# Examples`, `# Citations`

## type 분류 (팀 컨벤션)
- `Reference` 서비스·리소스·구성 설명
- `Runbook`   장애·운영 대응 절차
- `HowTo`     재사용 가능한 작업 방법
- `Decision`  아키텍처·운영 결정 기록
- `Policy`    보안·IAM 등 정책
(모르는 type 도 거부하지 말고 일반 개념으로 처리.)

## 예약 파일
- `index.md` 디렉토리 목차(점진적 공개). 개념 문서 아님.
- `log.md`   변경 이력. 최신이 위, 날짜는 ISO 8601 (YYYY-MM-DD).

## 검색·읽기 절차 (토큰 효율)
1. 먼저 `index.md` 로 무엇이 있는지 파악.
2. 프론트매터(type/title/description/tags)로 후보를 좁힘.
3. 관련 개념 문서만 선택적으로 연다 (lazy loading).
4. 번들 전체를 통째로 컨텍스트에 넣지 않는다.

## 저장·갱신 절차
1. 기본 저장 대상은 `~/kb/local/`. (common 직접 쓰기 금지)
2. 경로를 정한다(경로가 곧 개념 ID).
3. 같은 개념이 이미 있으면 **새로 만들지 말고 갱신**.
4. 생성·갱신 시 해당 범위 `log.md` 에 한 줄 기록.
5. 관련 개념은 절대경로 링크로 연결.
6. common 반영이 필요하면 local 초안 + 승격(PR) 안내.

## 하지 말 것 (Anti-patterns)
- `~/kb/common/` 을 직접 수정·삭제하지 않는다.
- 대화 로그·채팅 전문을 통째로 한 파일에 붙여넣지 않는다. (정제된 지식만)
- `type` 없이 개념 문서를 저장하지 않는다.
- 대화 단위로 파일을 만들지 않는다. **주제(개념) 단위**로 만든다.
- 민감정보(시크릿/키/자격증명)를 본문에 넣지 않는다.
- 사용자 확인 없이 임의로 저장하지 않는다.

## 예시 1 — Reference (경로: ~/kb/local/aws/standard-vpc.md)
```markdown
---
type: Reference
title: 표준 VPC 구성
description: 팀 표준 VPC 네트워크 레이아웃과 서브넷 분리 원칙.
resource: https://console.aws.amazon.com/vpc/
tags: [aws, network, vpc]
timestamp: 2026-06-22T00:00:00Z
---

# Overview
프로덕션 표준 VPC의 기본 레이아웃.

# Layout
| 구성요소        | 용도                          |
|---------------|-------------------------------|
| Public Subnet  | ALB, NAT Gateway             |
| Private Subnet | 애플리케이션 계층              |
| Data Subnet    | RDS 등 (외부 접근 차단)        |

# Related
- 배포는 [prod deploy](/runbooks/prod-deploy.md) 참조.

# Citations
[1] [AWS VPC 문서](https://docs.aws.amazon.com/vpc/)
```

## 예시 2 — Runbook (경로: ~/kb/local/runbooks/rds-failover.md)
```markdown
---
type: Runbook
title: RDS 장애 조치 대응
description: RDS Multi-AZ 장애 발생 시 대응 절차.
tags: [aws, rds, incident]
timestamp: 2026-06-22T00:00:00Z
---

# Symptoms
- 애플리케이션 DB 커넥션 타임아웃 급증.

# Steps
1. RDS 콘솔에서 인스턴스 상태/이벤트 확인.
2. Multi-AZ 자동 페일오버 진행 여부 확인.
3. 미진행 시 수동 페일오버 트리거.
4. 커넥션 풀 리셋 후 헬스체크 확인.

# Related
- 대상 DB는 [orders db](/aws/orders-db.md) 참조.
```

## 예시 3 — index.md / log.md
```markdown
# (index.md)
# AWS
* [표준 VPC 구성](aws/standard-vpc.md) - 팀 표준 VPC 레이아웃
# Runbooks
* [RDS 장애 조치](runbooks/rds-failover.md) - RDS 페일오버 대응
```
```markdown
# (log.md)
# Update Log
## 2026-06-22
* **Creation**: [RDS 장애 조치](/runbooks/rds-failover.md) 추가.
```