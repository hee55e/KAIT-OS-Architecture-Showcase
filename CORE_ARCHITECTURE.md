# KAIT OS Core Architecture (Layer 0-5)

> **Document is Interface, Code is Truth.**
> 본 문서는 KAIT OS의 핵심 개념 레이어를 설명하는 인터페이스 명세입니다.

KAIT는 완성된 '집'이 아닌, 어떤 환경에서도 조립 가능한 **'AI 건축 부품'과 '설계도'**를 제공합니다.

## Layered Architecture Overview
```text
┌─────────────────────────────────────────────────────────────┐
│  Layer 5: Direct Interfaces (Reusable Providers)            │
│  LLM/Embedding/STT/TTS/OCR/RAG/Image Editing                 │
├─────────────────────────────────────────────────────────────┤
│  Layer 4: Session                                           │
│  PersonalSession, WorldSession, EphemeralSession            │
├─────────────────────────────────────────────────────────────┤
│  Layer 3: AI Instance                                       │
│  AIInstance/ProcessorUnit 선언 + Runtime Assembly           │
├─────────────────────────────────────────────────────────────┤
│  Layer 2.5: Execution State                                 │
│  ExecutionState (DB), ExecutionContext (Memory)             │
├─────────────────────────────────────────────────────────────┤
│  Layer 2: Graph Orchestration                               │
│  GraphExecutor, Node/Edge Routing                           │
├─────────────────────────────────────────────────────────────┤
│  Layer 1: Execution Units                                   │
│  ProcessorType (LLM, Script, Tool 등), Context Filter       │
├─────────────────────────────────────────────────────────────┤
│  Layer 0: Foundation                                        │
│  Neural Knowledge Graph, UCF, UTCP, Segment, Checkpoint     │
└─────────────────────────────────────────────────────────────┘
```

### Layer 0: Foundation (기반층)
- **UCF (Universal Content Format):** 텍스트, 이미지, 음성 등 모든 멀티모달 콘텐츠를 묶어내는 단일 표준 데이터 규격입니다.
- **UTCP (Universal Tool Calling Protocol):** HTTP, WebSocket, CLI, MCP 등 파편화된 백엔드 실행 환경을 JSON 매뉴얼 하나로 통합합니다.

### Layer 1: Execution Units (실행 단위층)
- 독립적으로 재사용 가능한 지능 부품(ProcessorUnit)입니다. 
- LLM에 전달되는 인식 창(Perception Window)을 동적 파이프라인으로 구성합니다.

### Layer 2: Graph Orchestration (그래프 조율층)
- 단일 프롬프트가 아닌, 노드(Node)와 엣지(Edge) 기반의 제어 흐름입니다.
- 조건 분기, 루프 실행, 다중 입력 대기(Await), 병렬 브랜치 병합을 완벽히 지원합니다.

### Layer 2.5: Execution State (상태 관리층)
- 비동기 그래프 실행의 진행 상황을 DB에 영구 기록합니다. (pending → running → completed/interrupted)
- **HITL(Human-in-the-loop):** 승인 대기 지점에 도달하면 상태를 중단(Interrupted)하고, 관리자의 승인 토큰이 인가되면 체크포인트부터 다시 실행을 재개합니다.

### Layer 3 & 4: AI Instance & Session
- Graph와 Processor들을 조립하여 최종 AI 인스턴스를 런타임에 띄웁니다.
- 1:1 대화, 다자간 협업(WorldSession) 등 히스토리 상태를 안전하게 격리 보관합니다.
