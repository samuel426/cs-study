# CS Study

매일 하나의 CS 주제를 공부하고, 확인 문제에 답하며 이해한 내용과 오개념을 기록한다.

## 새 채팅에서 이어서 공부하기

이 저장소를 프로젝트로 연 새 채팅에서 다음처럼 요청한다.

> 오늘 CS 공부 시작하자. AGENTS.md와 STUDY_STATE.md를 읽고 현재 진도부터 이어서 진행해 줘.

| 문서 | 역할 |
|---|---|
| [AGENTS.md](AGENTS.md) | 새 채팅이 따를 학습 방식과 기록·커밋·푸시 규칙 |
| [CURRICULUM.md](CURRICULUM.md) | 전체 목차와 완료 여부 |
| [CS_PRACTICE_PLAN.md](CS_PRACTICE_PLAN.md) | CS 개념 설명과 적용 연습을 위한 집중 학습 계획 |
| [STUDY_STATE.md](STUDY_STATE.md) | 최근 진도, 복습 포인트, 진행 중 질문, 다음 시작점 |

Codex가 프로젝트 지침을 읽는 `AGENTS.md` 방식을 사용한다. [OpenAI 공식 안내](https://learn.chatgpt.com/docs/agent-configuration/agents-md)

## 학습 방식

1. 작은 주제 하나를 정해 개념을 공부한다.
2. 확인 문제 총 5개를 하나씩 풀고 답변에 대한 피드백을 받는다.
3. 이해한 내용과 수정한 생각을 Markdown으로 남긴다.
4. 목차와 현재 학습 상태를 갱신하고, 학습 기록이나 복습 내용을 커밋·푸시한다.

## 학습 기록

| 날짜 | 분야 | 주제 |
|---|---|---|
| 2026-10-05 | 운영체제 · Java | [프로세스와 스레드, 메모리 공유와 동기화](notes/2026-10-05-process-and-thread.md) |
| 2026-10-06 | 운영체제 | [컨텍스트 스위칭과 전환 비용](notes/2026-10-06-context-switching.md) · [TLB와 주소 변환](notes/2026-10-06-tlb.md) |
| 2026-10-07 | 운영체제 | [CPU 스케줄링 기초: 상태·선점·성능 기준·기아](notes/2026-10-07-cpu-scheduling-basics.md) |
| 2026-10-11 | 운영체제 | [스케줄링 알고리즘: FCFS·SJF·라운드 로빈](notes/2026-10-11-scheduling-algorithms.md) |
| 2026-10-12 | 운영체제 · Java | [동기화와 임계 구역: 공유 데이터·락·뮤텍스·세마포어](notes/2026-10-12-synchronization-critical-section.md) |
| 2026-10-19 | 운영체제 | [교착 상태: 필요 조건·락 획득 순서·기아와의 차이](notes/2026-10-19-deadlock.md) |
| 2026-10-20~21 | 운영체제 | [가상 메모리와 페이징: 주소 변환·페이지 테이블·TLB](notes/2026-10-21-virtual-memory-paging.md) |
| 2026-11-10 | 데이터베이스 | [SQL JOIN과 집계: NULL·DISTINCT·중복 집계·성능 검증](notes/2026-11-10-sql-joins-aggregation.md) |
| 2026-11-12 | 데이터베이스 | [인덱스와 조회 성능: 탐색·유지 비용·복합 인덱스·전체 스캔](notes/2026-11-12-indexes-query-performance.md) |
| 2026-11-12 | 데이터베이스 | [트랜잭션과 동시성: ACID·갱신 유실·잠금·복구](notes/2026-11-12-transactions-concurrency.md) |
| 2026-11-14 | Redis · 동시성 | [요청 제한: Sorted Set·시간 구간·동시 요청·TTL](notes/2026-11-14-redis-rate-limiting.md) |
