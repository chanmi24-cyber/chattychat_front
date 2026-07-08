# ChattyCat 🐱

## 프로젝트 소개
랜덤 매칭 1대1 채팅 서비스

## 기술 스택
- Backend: Java, Spring Boot, WebSocket
- ✅Frontend: Vue 3
- 메모리 기반 (DB 미사용)

## 주요 기능
- 닉네임 입력 후 입장
- 랜덤 1대1 매칭
- 실시간 채팅
- 채팅 종료 후 재대기 가능

## 기술적 고민
- DB 없이 ConcurrentHashMap으로 메모리 관리
- WebSocket 세션으로 사용자 식별
- Queue로 대기열 관리
