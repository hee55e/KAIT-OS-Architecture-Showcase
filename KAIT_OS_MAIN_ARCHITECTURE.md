# KAIT OS Architecture

> **⚠️ 중요: 코드가 진실입니다 (Code is Truth)**
> 이 문서는 KAIT 아키텍처의 개요를 제공하지만, **실제 구현은 코드에 있습니다**.
> 문서와 코드 간 불일치가 있는 경우, 항상 **코드를 우선**으로 참고하세요.
> 문서는 가이드이며, 코드는 유일한 진실의 원천(Single Source of Truth)입니다.

> **Document is Interface, Code is Truth**
> 이 문서는 KAIT 아키텍처의 **인터페이스 명세**를 제공합니다.
> 구현 상세가 필요한 경우, 명시된 **파일 경로**의 실제 코드를 참조하세요.
> 문서는 지도(Map)이며, 코드는 영토(Territory)입니다.

> 이 문서는 **개념 레이어(0~5 + 2.5 + Optional Domain)** 관점의 아키텍처입니다.  
> 폴더/모듈 구조 기준 레이어는 이 문서의 **"코드 기준 레이어 매핑"** 섹션을 단일 진실로 사용합니다. (별도 문서 분리 예정)
> 문서의 섹션 번호(예: 6, 7, 8)는 레이어 번호와 무관합니다.
>
> 방법론 요약 문서: [`0. 아키텍처/온톨로지_슬롯_메커니즘.md`](0.%20아키텍처/온톨로지_슬롯_메커니즘.md)  
> (SSOT → 온톨로지 → 그래프 → 실행데이터 기반 진화 루프)

---

## 1. Core Philosophy (중요한건 아니고 풀네임을 가정하지 않게 하기 위한 그냥 임시 개념 정의)

**KAIT = Knowledge → Architect → Intelligence → Technology**

KAIT는 AI를 위한 운영체제입니다. 완성된 '집'이 아닌, 어떤 환경에서도 조립 가능한 **'AI 건축 부품'과 '설계도'**를 제공합니다.

---

## 2. Layered Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│  Optional Domain: Binding AI (Layer outside core stack)       │
│  /api/v1/binding/fill, /api/v1/binding/fill/stream            │
│  (routers/binding/binding.py → core/binding/service.py)       │
├─────────────────────────────────────────────────────────────┤
│  Layer 5: Direct Interfaces (Reusable Providers)            │
│  LLM/Embedding/STT/TTS/OCR/RAG/Image Editing + core/directs  │
├─────────────────────────────────────────────────────────────┤
│  Layer 4: Session                                           │
│  PersonalSession, WorldSession, EphemeralSession, ...       │
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
│  ProcessorUnit, ProcessorType, Context Filter               │
├─────────────────────────────────────────────────────────────┤
│  Layer 0: Foundation                                        │
│  Neural Knowledge Graph, UCF, UTCP, Viewport, Segment(+Registry) │
└─────────────────────────────────────────────────────────────┘
```

Direct Providers are reusable low-level modules referenced by Processor/Tool/Filter/Input/Output paths.
They are not a "top" runtime layer above Sessions; they are shared building blocks.

### ⚠️ Layer 번호의 의미 (중요)

**Layer 번호는 호출 순서가 아니라 "추상화/재사용 수준"을 나타냅니다.**

| Layer | 의미 | 설명 |
|-------|------|------|
| Layer 외부 (Optional Domain) | **선택적 인터페이스 결합** | Binding AI (`fill` 중심, `action`/`suggest` 미구현), 기본 Conversation 경로와 독립 |
| Layer 5 | 재사용 가능한 저수준 빌딩 블록 | LLM/Embedding/STT/TTS/OCR/RAG 등 외부 API 래퍼 |
| Layer 4 | 상태 컨테이너 | 대화 히스토리/세션 관리 |
| Layer 3 | 런타임 조립 단위 | AI Instance = Graph + Processors |
| Layer 2 | **실행 제어 중심** | GraphExecutor가 모든 흐름을 조율 |
| Layer 1 | 실행 유닛 | ProcessorType, Context Filter |
| Layer 0 | 데이터 규격 | UCF, UTCP, Viewport, Segment, Checkpoint (기반 규격) |

> Binding은 Layer 번호 체계 밖의 Optional Interface Domain으로 관리됩니다.

### 코드 기준 레이어 매핑 (현행 구현)

> 아래 표는 **개념 레이어 ↔ 실제 코드 위치**를 1:1로 연결한 “현행 코드 기준 지도”입니다.

| 개념 Layer | 주요 코드 위치 (현재 코드 기준) |
|-----------|--------------------------------|
| Layer 외부 (Binding Domain) | `kait_os_main/core/binding/`, `kait_os_main/routers/binding/binding.py` |
| Layer 5 (Direct Interfaces) | `kait_os_main/core/llm_direct/`, `kait_os_main/core/embedding_direct/`, `kait_os_main/core/stt_direct/`, `kait_os_main/core/tts_direct/`, `kait_os_main/core/ocr_direct/`, `kait_os_main/core/rag_direct/`, `kait_os_main/core/image_editing_direct/`, `kait_os_main/core/directs/` |
| Layer 4 (Session) | `kait_os_main/core/sessions/` (`personal/`, `world/`, `ephemeral/`), `kait_os_main/db_models/chat_session.py`, `kait_os_main/db_models/world_session.py`, `kait_os_main/db_models/session_image.py` |
| Layer 3 (AI Instance) | `kait_os_main/db_models/cognitive_architecture.py` (AIInstance, ProcessorUnit, `cognitive_pipeline` 선언) |
| Layer 2.5 (Execution State) | `kait_os_main/db_models/execution_state.py`, `kait_os_main/db_models/checkpoint_record.py`, `kait_os_main/core/orchestration/graph/execution_context.py` |
| Layer 2 (Orchestration) | `kait_os_main/core/orchestration/graph/**`, `kait_os_main/core/orchestration/pipeline_orchestrator.py`, `kait_os_main/core/orchestration/pipeline_orchestrator_parts/execution/streaming.py` |
| Layer 1 (Execution Units) | `kait_os_main/core/execution/processor_types/`, `kait_os_main/core/execution/processor_types/llm/compaction.py`(compact 실행 SSOT), `kait_os_main/core/execution/processor_types/llm/compaction_policy.py`(compact 트리거/예산 정책 SSOT), `kait_os_main/core/context/filters/`, `kait_os_main/tools/tool_executor.py`(UTCP 실행), `kait_os_main/db_models/cognitive_architecture.py` (ProcessorUnit) |
| Layer 0 (Foundation) | `kait_os_main/core/common/content.py`(UCF), `kait_os_main/core/common/segment.py` + `kait_os_main/core/common/cognitive_segment.py`(Segment), `kait_os_main/core/common/segment_registry/`(Segment 계약 SSOT), `kait_os_main/core/common/checkpoints/`(Checkpoint 계약), `kait_os_main/utcp/**`(UTCP 규격), `kait_os_main/core/viewport/`(Viewport), `kait_os_main/db_models/corpus.py`(Neural Knowledge Graph) |

**레이어 경계 불변식 (코드 기준):**
- Layer 3는 `AIInstance.cognitive_pipeline` 같은 **선언적 그래프 정의(무엇을 실행할지)**를 소유합니다.
- Entry Layer(`ConversationRouterOrchestrationService`/`ConversationService`/`OrchestratorManager`)는 실행 컨텍스트를 조립하고, Layer 2 진입점인 `CognitiveOrchestratorService.stream_turn`에 위임합니다.
- Layer 2는 `GraphLoader` + `GraphExecutor.execute_graph`로 **실행 제어(어떻게 실행할지)**를 수행합니다.
- Entry Layer(라우터/서비스 경계)는 실행 진입점이지만, 번호 레이어(0~5)에는 포함되지 않습니다.

**왜 이런 구조인가?**
- KAIT는 "레고 조립" 철학을 따릅니다
- Layer 5(빌딩 블록)를 Layer 2(조립 엔진)가 조립하여 Layer 3(AI)를 만듭니다
- 일반적인 계층 구조(Presentation → Business → Data)와 다릅니다

### 실제 제어 흐름 (Request → Response)

```
HTTP Request (POST /api/v1/conversation/stream)
    │
    ▼
┌─────────────────────────────────────────────────────────────────┐
│  Entry Layer (문서에 미포함, 실제 진입점)                        │
│  handle_conversation_stream                                      │
│   → ConversationRouterOrchestrationService.stream_turn           │
│   → ConversationService.stream_turn                              │
│   → ConversationOrchestrationStreamMixin._stream_orchestrator_events │
│   → OrchestratorManager.get_orchestrator                         │
│  위치: routers/conversation/conversation.py                       │
│       services/conversation/router/core/orchestration.py          │
│       services/conversation/interaction/flows/orchestration_stream.py │
│       services/execution/orchestrator_manager.py                  │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  Layer 2: Graph Orchestration (제어 중심)                        │
│  CognitiveOrchestratorService.stream_turn                         │
│   → GraphLoader.load_graph → GraphExecutor.execute_graph          │
│  GraphExecutor 내부: NodeRouter + EdgeManager/ConditionEvaluator  │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐         │
│  │ Node 1  │ → │ Node 2  │ → │ Node 3  │ → │ Node N  │         │
│  └────┬────┘   └────┬────┘   └────┬────┘   └────┬────┘         │
└───────┼─────────────┼─────────────┼─────────────┼───────────────┘
        │             │             │             │
        ▼             ▼             ▼             ▼
┌─────────────────────────────────────────────────────────────────┐
│  Layer 1: Execution Units                                        │
│  ProcessorRouter → ProcessorType (LLM/Tool/Script...)           │
│  ContextBuilder → Filters (system_prompt, history, user_prompt) │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  Layer 5: Direct Providers (저수준 빌딩 블록 - 호출됨)           │
│  LLM/Embedding/STT/TTS/OCR/RAG/Image Editing Direct              │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  Layer 0: Foundation (데이터 규격 - 모든 레이어에서 참조)        │
│  UCF (데이터 형식), UTCP (도구 규격), Segment (출력 형식)        │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ▼
                    SSE Response Stream
```

**핵심 통찰:**
- **제어 흐름**: Entry → ConversationRouterOrchestrationService → Layer 2 → Layer 1 → Layer 5 (위에서 아래로 호출)
- **데이터 흐름**: Layer 5 → Layer 1 → Layer 2 → ConversationRouterOrchestrationService → Entry (아래에서 위로 반환)
- **Layer 0**: 모든 레이어에서 횡단적으로 참조되는 데이터 규격

---

## 3. Layer 0: Foundation (기반층)

### 3.1 Neural Knowledge Graph

**위치**: `kait_os_main/db_models/corpus.py`

**상세 문서**: 분리 예정 (현재 이 섹션이 단일 진실)

---

### 3.2 UCF (Universal Content Format)

**위치**: `kait_os_main/core/common/content.py`

모든 콘텐츠의 표준 형식. 텍스트, 이미지, 음성, 문서 등을 통합 표현합니다.

```python
ContentPart = Dict[str, Any]
UniversalContent = Union[str, List[ContentPart]]
```

**주요 유틸리티:**
- `ensure_ucf()` - 임의 값을 UCF로 정규화
- `extract_text()` - UCF에서 텍스트만 추출
- `is_multimodal()` - 멀티모달 여부 확인

**ParsedInput** (`kait_os_main/core/input/base.py`): 입력 파서의 출력 형식으로, `text`, `input_type`, `attachments`, `images` 등 메타데이터 포함.

---

### 3.3 UTCP (Universal Tool Calling Protocol)

**위치(규격)**: `kait_os_main/utcp/categories/{category}/{tool_name}.json`  
**위치(실행)**: `kait_os_main/tools/tool_executor.py` *(Layer 1 Execution Units에서 호출됨)*

다양한 백엔드(HTTP, WebSocket, gRPC, CLI 등)를 **JSON 매뉴얼 하나로 통합**하는 도구 정의 표준입니다.

**UTCP 매뉴얼 구조:**
```json
{
  "name": "tool_name",
  "description": "What the tool does",
  "tool_provider": {
    "provider_type": "http|native|mcp|llm|subagent|...",
    // provider_type별 설정 필드
  },
  "inputs": { "type": "object", "properties": {...}, "required": [...] },
  "outputs": { "type": "object", "properties": {...} }
}
```

**Provider Types (예시):**

| Provider Type | 용도 |
|---------------|------|
| `http`, `https` | REST API 호출 |
| `websocket`, `ws`, `wss` | WebSocket 통신 |
| `sse`, `http_stream` | 스트리밍 |
| `grpc`, `graphql` | RPC/GraphQL |
| `cli` | 커맨드라인 실행 |
| `tcp`, `udp`, `webrtc` | 소켓/RTC 통신 |
| `native` | Python 함수 직접 실행 |
| `terminal` | PTY/터미널 실행 |
| `mcp` | Model Context Protocol |
| `llm`, `subagent` | LLM/에이전트 호출 |
| `computer_use`, `gemini_computer_use`, `electron_browser` | 브라우저 자동화 |

**도구 저장 위치:**
- 시스템 도구: `utcp/categories/{category}/{tool_name}.json`
- 사용자 도구: DB `user_tools` 테이블

---

### 3.4 Handler Registry & Dispatcher

**위치**: `kait_os_main/tools/handlers/registry.py`, `kait_os_main/tools/tool_executor.py`

UTCP 도구 실행은 **HandlerRegistry 기반 Dispatcher 패턴**으로 동작합니다.
KAIT의 레지스트리 시스템은 **`RegistryBase` 싱글톤 패턴**으로 일원화되어 가고 있습니다.

**Registry Status (단일 진실 소스):**

| 레지스트리 | 상속 (RegistryBase) | 특징 |
|------------|--------------------|------|
| **HandlerRegistry** | ✅ Yes | 모든 UTCP 핸들러 관리 |
| **EdgeRegistry** | ✅ Yes | 그래프 엣지 타입 관리 |
| **FilterRegistry** | ✅ Yes | 컨텍스트 필터 관리 |
| **SessionRegistry** | ✅ Yes | 세션 타입 관리 |
| **ViewportSlotRegistry** | ✅ Yes | 뷰포트 슬롯(Browser, Terminal 등) 관리 |
| **ProviderRegistry** | ✅ Yes | LLM Direct 공급자 관리 |
| **InputParserRegistry** | ✅ Yes | 입력 파서 관리 (manifest 기반 auto-discovery) |
| **ProcessorTypeRegistry** | ✅ Yes | 프로세서 타입 관리 (manifest 기반 auto-discovery) |
| **NodeTypeRegistry** | ✅ Yes | RegistryBase 상속 + node manifest 직접 discovery/plugin 지원 |
| **SegmentRegistry** | ✅ Yes (specialized) | RegistryBase 기반 + **JSON 타입/스타일 메타데이터** 관리 (class entrypoint 없이 manifest 직접 로딩) |

**RegistryBase 상속 시 이점**:
- `get_instance()`, `reset_instance()`, `add_plugin_path()`, `refresh()`, `reload_type()`, `reload_changed_paths()` 공통 인터페이스
- `manifest.json` 기반 자동 발견(Auto-discovery) + 중앙 refresh SSOT
- `create_snapshot()` + `version` 기반 요청 단위 스냅샷 격리
- `reload_type()`는 unregister 선행 없이 overwrite 방식으로 교체(실행 중 참조 보호)

**Runtime Snapshot Isolation (코드 기준):**
- `GraphExecutor` 실행 시작 시 `NodeTypeRegistry`/`HandlerRegistry` 스냅샷을 확보하고 세션 종료까지 고정합니다.
- 도구 매뉴얼은 요청 시작 시 request-scoped snapshot으로 동결되어 재귀 루프 도중 파일 변경이 있어도 현재 실행 계약이 바뀌지 않습니다.
- 런타임 갱신은 다음 요청부터 새 스냅샷으로 반영됩니다.

**Runtime Auto Hot-Reload (코드 기준):**
- `PluginRuntimeWatchService`가 파일 변경을 debounce 배치한 뒤 단일 오케스트레이터로 반영합니다.
- 변경 경로는 `runtime plugin`/`handler`/`filter`/`UTCP`로 분류됩니다.
- 플러그인은 `refresh_runtime_plugins_for_app`, 핸들러/필터는 `RegistryBase.reload_changed_paths()` SSOT 경로로 manifest 단위 증분 reload, UTCP는 증분(`reload_tool`) 또는 전체(`reload_all`)로 적용됩니다.
- 현재 실행 중 세션은 스냅샷 격리로 보호되고, 자동 리로드 결과는 다음 요청부터 일관되게 반영됩니다.

**상세 문서**: 분리 예정 (현재 이 섹션이 단일 진실)
- 전체 핸들러 목록 및 상태
- Manifest 구조 및 확장 가이드
- Category 기반 인스턴싱 전략
- Deprecated 핸들러 마이그레이션
- Testing 및 Best Practices


---

### 3.5 Security & Configuration

**코드 기준 단일 진실 요약**

#### Tool 실행 가드레일 (실행 순서)
1.  **허용 도구 필터**: `processor.allowed_tools` 또는 `processor_config.tools/allowed_tools` 기준으로 선별 (`core/execution/components/tool_handler_batch.py`).
2.  **승인/정책 평가**: `ToolExecutorRuntimeMixin._enforce_policy()` → `ApprovalPolicyService.evaluate()`에서 다음 순서로 결정:
    - pre-approved cache
    - node/user policy
    - connector policy
    - runtime capability profile (`decide_tool_allowed`)
    - tool metadata risk
    - `ToolPermissionService.check_permission`
3.  **입력 스키마 보정/검증**: `get_input_schema` → `apply_schema_defaults` → `adapt_arguments_for_execution` (`strict|lenient`).

#### Validation Mode 우선순위
`_resolve_input_validation_mode` 기준:
1.  `KAIT_UTCP_SCHEMA_MODE`
2.  매뉴얼/metadata 선언값
3.  `cloud_strict` 기본값(`strict`)
4.  기본값(`lenient`)

#### `${env.VAR}` 해석 우선순위
도구 실행 경로 기준(`_load_user_env_variables` + `resolve_env_expressions`):
1.  `client_secrets` (request context)
2.  `UserExecutorConfig.encrypted_secrets`
3.  `UserExecutorConfig.env_variables`
4.  시스템 환경변수(`os.getenv`)
5.  `env_keys` fallback 후보

#### User Executor Config / Attachment Parsing
- SSOT 모델: `db_models/user_executor_config.py`
- 핵심 필드: `max_iterations`, `persist_attachments`, `save_parsed_to_history`, `encrypted_secrets`, `env_variables`, `custom_settings`
- 첨부파일 파싱 정책 SSOT: `core/common/attachment_parsing_policy.py` (`custom_settings["parser_settings"]`)
- 실행 시 `persist_attachments`, `custom_settings`, `attachment_parse_timeout`, `attachment_parse_concurrency`를 함께 반영

#### Native Code Sandbox
- preset 기반 기본값: `use_restricted=True`, `use_isolation=True`
- `sandbox_config.use_isolation=false` 요청은 안전상 무시
- `client_local`: `LocalSandbox`(RestrictedPython 기반 `PythonSandbox`)
- RestrictedPython 초기화 실패 시 fail-closed (`permission_denied`)
- `cloud_strict`: `DockerSandbox` 경로를 사용하며 현재 구현은 fail-closed(실행 차단)
- 로컬 신뢰 환경에서만 `KAIT_LOCAL_UNRESTRICTED_SANDBOX=true`로 무제한 모드 opt-in 가능

---

### 3.6 MCP Architecture

**코드 기준 단일 진실 요약**

- 매니저: `UserMCPManager` (싱글톤)
- 용량 한계:
  - `MAX_POOLS=10000`
  - `MAX_PROCESSES_PER_USER=3`
- 수명/정리:
  - `PROCESS_TTL_SECONDS=300`
  - `POOL_TTL_SECONDS=3600`
  - `SSE_CLIENT_TTL_SECONDS=600`
  - `HTTP_CLIENT_TTL_SECONDS=1800`
  - `SSE_PING_INTERVAL=60`
  - cleanup loop 주기: 60초
- stdio 풀 동작:
  - 사용자별 `UserMCPPool` 분리
  - 사용자 풀 내 프로세스 초과 시 `last_used` 기준 LRU 축출
- transport별 관리:
  - `stdio`: 사용자/서버별 subprocess
  - `http`: 사용자별 `httpx.AsyncClient` TTL 재사용
  - `sse`: `user_id:server_name` 단위 클라이언트 + ping/reconnect 관리
- 자격증명/환경치환:
  - 서버 env 템플릿(`${VAR}`)은 `env_resolver`를 통해 사용자 env(`encrypted_secrets`/`env_variables`) + 시스템 env로 해석
  - HTTP 인증 실패(401/403 등) 시 OAuth token auto-refresh 경로 포함

---

### 3.7 OutputSegment

**위치**: `kait_os_main/core/common/segment.py`

UCF가 "무엇(what)"을 정의한다면, OutputSegment는 "어떻게(how)" 표현하고 저장할지 정의합니다.

```
UCF (콘텐츠)
    ↓
OutputSegment (백엔드 처리용)
    - type, content, persist, stream, style
    ↓
cognitiveSegments (프론트엔드 렌더링용)
    - to_cognitive_segments() 변환
```

**SegmentType / 기본 동작 SSOT (코드 기준):**
- 단일 진실 소스: `kait_os_main/core/common/segment_registry/types/*.json`
- `SegmentType` Enum: `kait_os_main/core/common/enums_parts/contracts/segment.py`에서 SegmentRegistry를 기준으로 동적 생성
- 기본 persist/stream 값은 SegmentRegistry manifest(`behavior.*`)를 따릅니다.
- 별칭 해석은 manifest의 `aliases`를 우선 사용하며, `kait_os_main/core/common/enums_parts/contracts/segment.py`의 enum 하위호환 alias(`HITL_REQUEST -> hitl_input`, `TOOL_RESULT -> tool_output`)가 추가로 존재합니다.

| 타입 | 설명 | 기본 Persist | 기본 Stream | 별칭 |
|------|------|-------------|------------|------|
| `content` | 텍스트/마크다운 콘텐츠 | ✅ | ✅ | - |
| `thinking` | LLM 사고 과정 (extended thinking) | ❌ | ✅ | - |
| `tool_call` | UTCP 도구 호출 메타 | ❌ | ✅ | - |
| `tool_output` | 도구 실행 결과 | ❌ | ✅ | `tool_result` |
| `native_tool_call` | 네이티브 도구 호출 (provider native) | ❌ | ✅ | - |
| `native_tool_result` | 네이티브 도구 결과 | ❌ | ✅ | - |
| `api_response` | API 호출 결과 | ❌ | ❌ | - |
| `script_result` | 스크립트 실행 결과 | ❌ | ✅ | - |
| `image_generation` | 이미지 생성 결과 | ✅ | ❌ | - |
| `audio_generation` | 오디오 생성 결과 | ✅ | ❌ | - |
| `video_generation` | 비디오 생성 결과 | ✅ | ❌ | - |
| `visual_block` | 리치 미디어 블록 | ❌ | ✅ | - |
| `app` | AI 생성 UI(Application) | ✅ | ❌ | - |
| `hitl_input` | HITL 요청/입력 | ✅ | ✅ | `hitl`, `hitl_request` |
| `perception_log` | 퍼셉션 추론 로그 | ❌ | ❌ | - |
| `processor_start` | 프로세서 실행 상태 지시 | ❌ | ✅ | `processor_status` |
| `session_compaction` | 세션 요약/경계 메타 | ✅ | ✅ | - |
| `error` | 실행 에러 정보 | ✅ | ✅ | - |
| `merge_result` | 병렬 브랜치 병합 결과 | ❌ | ❌ | - |

`SegmentType` enum 하위호환 멤버: `SegmentType.HITL_REQUEST`, `SegmentType.TOOL_RESULT`

**OutputSegmentStyle 필드:**
`collapsible`, `collapsed_by_default`, `syntax`, `language`, `icon`, `color`, `max_height`, `scrollable`

---

### 3.7.1 Segment is Truth (엄격)

- **Segment는 저장/스트리밍/복원의 단일 진실**이다.
- SSE 이벤트는 **델타**이며, `cognitive_segments_update` 스냅샷이 오면 UI/클라이언트는 **세그먼트 기준으로 재구성**해야 한다.
- **순서 보존이 핵심**이며, 병합/가공은 **표현 계층에서만** 허용된다(정보 손실 금지).
- **알 수 없는 Segment 타입은 소실 금지**: UI는 raw segment를 그대로 보존하고, 렌더는 Unknown/Fallback로 처리한다.

---

### 3.8 Segment Registry

**위치**: `kait_os_main/core/common/segment_registry/`

Segment 타입 정의는 **manifest 기반 레지스트리**로 관리됩니다.
`types/*.json`가 단일 소스이며, `type_name`, `behavior.persist.default`, `behavior.stream`, `style`, `aliases`를 중앙에서 조회합니다.
현재 등록된 canonical 타입(코드 기준): `content`, `thinking`, `tool_call`, `tool_output`, `native_tool_call`, `native_tool_result`, `api_response`, `script_result`, `image_generation`, `audio_generation`, `video_generation`, `visual_block`, `app`, `hitl_input`, `perception_log`, `processor_start`, `session_compaction`, `error`, `merge_result`.
주요 alias: `tool_result -> tool_output`, `hitl/hitl_request -> hitl_input`, `processor_status -> processor_start`.

---

### 3.9 Multimodal Asset Architecture

**위치**: `kait_os_main/core/common/handlers/multimodal.py`

**핵심 철학: Reference by ID, Value by Request**

KAIT는 DB 부하를 줄이고 유연성을 확보하기 위해 **JIT(Just-In-Time) Resolution** 패턴을 사용합니다. 로컬 에셋은 참조 URL(`/api/v1/image-search/assets/{id}`, `/api/v1/video-search/assets/{id}`, `/files/...`)로 유지되고, LLM 호출 직전에 `MultimodalHandler`가 이를 가로채 실제 Base64 데이터로 변환/주입합니다.

**Hybrid Resolution Strategy:**
- **Structured Object**: `json{"type": "image_url"}` 감지 및 변환
- **Markdown String**: 텍스트 내 `![alt](/api/...)` 패턴 감지 → `[Text, Image, Text]` 시퀀스로 분할 변환

**상세 문서**: 분리 예정 (현재 이 섹션이 단일 진실)

---

### 3.10 Multimodal Data Pipeline & Safety (Updated 2026-01-05)


**4-Layer Agent Safety Net:**
1.  **Standardization**: 모든 도구 결과(Dict/Media)를 `RecursionMixin`과 `ChatHistoryFilter`에서 즉시 UCF 표준으로 자동 변환
2.  **System Isolation**: 시스템 메시지(`sender_type='system'`)를 히스토리에서 격리하여 컨텍스트 오염 방지
3.  **ID Integrity**: 누락된 `tool_call_id` 자동 생성 및 복구
4.  **Orphan Protection**: 결과가 유실된 도구 호출에 대해 "Result Missing" 경고를 주입하여 대화 흐름 유지

**상세 문서**: 분리 예정 (현재 이 섹션이 단일 진실)

---

## 4. Layer 1: Execution Units (실행 단위층)

### 4.1 ProcessorUnit

**위치**: `kait_os_main/db_models/cognitive_architecture.py`, `kait_os_main/core/execution/processor_types/`

재사용 가능한 지능 부품. 독립적으로 저장되며 여러 AI에서 공유됩니다.

**핵심 필드:**

| 필드 | 타입 | 설명 |
|------|------|------|
| `name` | str | 프로세서 이름 |
| `description` | str | 프로세서 설명 |
| `processor_type` | str | `llm`, `utcp`, `script`, `chain`, `corpus` (`tool`/`api` alias는 제거됨) |
| `config` | Dict | 타입별 설정 |
| `perception_config` | Dict | 인식 설정 (history_limit 등) |
| `perception_pipeline` | List[UUID] | 컨텍스트 필터 파이프라인 |
| `allowed_tools` | List[str] | 허용된 도구 목록 |
| `exclude_from_final_context` | bool | 최종 응답 컨텍스트 제외 여부 |
| `category` | str | Intelligence Taxonomy 카테고리 |

> `category` 계약 SSOT: `kait_os_main/contracts/intelligence_taxonomy.py`  
> - 정규화/검증/기본값(`_uncategorized`)은 이 모듈에서만 정의  
> - 입력 경로(`schemas/cognitive_architecture.py`, `/api/v1/processors`, `/api/v1/ai-instances`)와
>   분류 관리 경로(`/api/intelligence-taxonomy/**`)가 동일 계약을 사용

**ProcessorType별 역할:**

| 타입 | 위치 | 역할 |
|------|------|------|
| **LLM** | `kait_os_main/core/execution/processor_types/llm/orchestrator.py` | AI 모델 실행, RecursionMixin으로 도구 재귀 |
| **UTCP** | `kait_os_main/core/execution/processor_types/utcp/utcp.py` | UTCP 도구 실행 또는 Ad-hoc HTTP 호출 (통합) |
| **Script** | `kait_os_main/core/execution/processor_types/script/script.py` | 샌드박스/타임아웃 기반 변환 실행 |
| **Chain** | `kait_os_main/core/execution/processor_types/chain/chain.py` | 순차 프로세서 실행 |
| **Corpus** | `kait_os_main/core/execution/processor_types/corpus/corpus.py` | 정적 지식 저장소 (Retriever). NeuralImprints 검색 담당 |
| **Custom (Plugin)** | `config.PROCESSOR_PLUGIN_DIRS`로 주입된 경로 | RegistryBase 플러그인 auto-discovery로 로드 |

> `processor_type`의 `tool`/`api` alias는 제거되어, 정규화 단계에서 오류 처리됩니다 (`kait_os_main/core/execution/processor_type_normalizer.py`).

> **ProcessorTypeRegistry**: `RegistryBase`를 상속하며 `manifest.json` 기반 auto-discovery를 사용합니다.

---

### 4.2 Context Filter

**위치**: `kait_os_main/core/context/builder.py`, `kait_os_main/core/context/filters/`

LLM에 전달되는 **인식 창(Perception Window)**을 구성합니다. **FilterRegistry (RegistryBase 상속)**를 통한 동적 로딩.

**Manifest 구조** (`kait_os_main/core/context/filters/**/manifest.json`):
```json
{
  "filter_type": "pipeline",
  "output_type": "system_prompt_parts",
  "entry_point": "pipeline_filter.py",
  "class_name": "PipelineFilter",
  "dependencies": []
}
```

**output_type별 역할:**

| output_type | 역할 | 반환 형식 |
|-------------|------|----------|
| `system_prompt_parts` | 시스템 프롬프트 구성 | List[`ContextPart`] (`type="system_prompt"`) |
| `history_parts` | 대화 히스토리 | List[`ContextPart`] (`type="chat_history_messages"`, `chat_history_summary`, `world_session`) |
| `user_prompt_parts` | 사용자 입력 | List[`ContextPart`] (`type="user_prompt"`) |

필터의 반환 타입은 `kait_os_main/schemas/context.py`의 `ContextPart`이며,
`ContextBuilder._assemble_response()`가 `ContextPart.type`을 `BuildContextResponse`로 조립합니다.

필터의 “정확한 목록/구성”은 **manifest + DB**가 결정합니다.
- 코드 기반 필터: `kait_os_main/core/context/filters/**/manifest.json`
- DB 기반 필터: `crud_context_filter`를 통해 로드되며, `sandbox_*`/`custom_*`는 동적 로드 경로를 가집니다.

**병렬 실행**: manifest의 `dependencies`를 DAG로 해석해 의존성 없는 필터를 동시 실행합니다 (`kait_os_main/core/context/parallel_executor.py`)

---

### 4.3 RecursionMixin

**위치**: `kait_os_main/core/execution/processor_types/mixins/recursion/`

LLM ProcessorType의 **도구 호출-응답 재귀 루프**.

- **Loop Orchestration**: `recursion/loop.py` (전체 흐름 제어)
- **Iteration Limit**: `RecursionState.max_iterations` 기본값은 `0`(무제한)이며, 런타임 하드캡은 **10,000회** (`_UNLIMITED_MAX`)입니다.
- **LLM 설정 기본값(코드 기준)**: LLM Processor manifest 기본값은 `enable_tool_recursion=true`, `max_iterations=200`이며, Inline LLM Node는 `enable_tool_recursion=false`일 때 `max_iterations`를 `1`로, `true`일 때 `200`으로 보정합니다.
- **HITL Integration**: 실행 전 승인이 필요한 도구에 대해 루프를 중지하고 사용자 결정을 대기 (`recursion/hitl.py`)
- **Structured Output Mode**: 도구 없이 특정 JSON 스키마로만 응답해야 하는 경우의 특화된 재귀 처리 (`recursion/structured_output.py`)
- **State Management**: `recursion/state.py` (ID 발급 및 이력 관리)

---

### 4.4 Tool Binding

**위치**: `kait_os_main/core/execution/tool_binding_handler.py`

`allowed_tools`를 LLM이 이해할 수 있는 형식으로 변환.

```
ProcessorUnit.allowed_tools
    ↓
UTCP Manuals 로드
    ↓
Provider별 변환 (OpenAI/Anthropic/Google)
    ↓
llm.bind_tools()
```

---

### 4.5 LLM Provider

**위치**: `kait_os_main/core/llm_direct/base.py`, `kait_os_main/core/llm_direct/providers/`, `kait_os_main/services/llm/llm_factory.py`

**BaseLLMProvider 인터페이스:**
- `ainvoke()` - 비스트리밍 호출
- `astream()` - 스트리밍 호출
- `bind_tools()` - 도구 바인딩
- `_convert_utcp_to_native()` - UTCP → Provider 형식 변환

**지원 Provider(코드 기준):**
- 구현체 디렉토리: `kait_os_main/core/llm_direct/providers/` (openai/anthropic/google/custom/deepseek/grok/kimi/antigravity/mock, OAuth transport 포함: openai_codex_oauth/google_gemini_cli_oauth)
- Provider Profile(`kait_os_main/core/llm_direct/providers/profiles/`)은 base provider에 매핑되어 동작합니다.
- Vertex AI는 별도 provider가 아니라 Google provider의 옵션(`vertexai`)으로 동작합니다.

**Config 구조:**
```json
{
  "provider": "openai",
  "model": "gpt-4o",
  "temperature": 0.7,
  "max_tokens": 4096,
  "api_key": "...",  // 또는 api_key_env
  "base_url": "..."  // custom만
}
```

---
## 5. Layer 2: Graph Orchestration (그래프 조율층)

### 5.1 Graph (인지 파이프라인)

**위치**: `kait_os_main/core/orchestration/graph/`

노드와 엣지로 구성된 **실행 흐름 설계도**.

```json
{
  "nodes": {"node_id": {"type": "llm_node|tool_node|processor|...", ...}},
  "edges": [...],
  "entry_node_id": "start_node"
}
```

---

### 5.1 Node Types & Registry

**위치**: `kait_os_main/core/orchestration/graph/nodes/`, `kait_os_main/core/orchestration/graph/nodes/registry.py`

그래프의 실행 단위를 정의합니다. **NodeTypeRegistry**가 관리하며, 각 NodeExecutor는 `HANDLED_TYPES`를 선언합니다.
(참고: `NodeTypeRegistry`는 `RegistryBase`를 상속하며, 인스턴스 직접 등록과 manifest 기반 등록을 함께 지원합니다.)

| 클래스 | NodeType | 특징 |
|--------|----------|------|
| **ProcessorNode** | `processor` | ProcessorUnit 참조 실행. LLM ProcessorType는 config의 `max_iterations`를 사용하며, manifest 기본값은 `200` (`0`은 무제한 요청, 내부 상한 `10000`) |
| **LLMNode** | `llm_node` *(alias: `llm`)* | 인라인 LLM 호출. 템플릿(`${input}`) 지원. 도구 재귀는 기본 비활성 (루프는 그래프에서 명시) |
| **ToolNode** | `tool_node` *(alias: `tool`)* | UTCP 도구 직접 실행 |
| **SubGraphNode** | `subgraph_node` *(alias: `subgraph`)* | 하위 그래프 중첩 실행 |
| **SandboxNode** | `sandbox_node` *(alias: `sandbox`)* | 샌드박스 환경 내 코드 실행 |
| **ScriptNode** | `script_node` *(alias: `script`)* | Python 데이터 변환 실행 |
| **InputNode** | `input_node` *(alias: `input`)* | 사용자 입력/매칭 노드 |
| **HITLNode** | `hitl_node` *(alias: `hitl`)* | Human-in-the-loop (승인 대기) 인터럽트 발생 |
| **MergeNode** | `merge_node` *(alias: `merge`)* | 병렬 브랜치 결과 병합 (Strategy: list, dict, concat) |
| **APINode** | `api_node` *(alias: `api`)* | 가벼운 HTTP 호출 노드 |
| **NoteNode** | `note_node` *(alias: `note`)* | 메모 전용 (Pass-through) |
| **OutputNode** | `output` / `outputgate` *(alias: `output_node`)* | 그래프 종료 및 결과 반환 |

> 참고: `NodeType` Enum에는 `smart_unit`이 존재하지만, 기본 manifest executor는 등록되어 있지 않습니다.

---

### 5.2 Edge Types & Registry

**위치**: `kait_os_main/core/orchestration/graph/edges/`, `kait_os_main/core/orchestration/graph/edges/registry.py`

노드 간의 데이터 및 제어 흐름을 정의합니다. **EdgeRegistry (RegistryBase 상속)**가 관리합니다.

| 타입 | 설명 | 용도 |
|------|------|------|
| **linear** | A → B 순차 실행 | 기본 흐름 |
| **parallel** | A → [B, C, D] 병렬 실행 | 동시 처리 |
| **conditional** | 조건 분기 (if/else) | 표현식 기반 분기 |
| **selective** | `routing`의 하위호환 alias (Deprecated) | 기존 그래프 호환 유지 |
| **routing** | 출력 기반 라우팅 | output_parsing, semantic_match |
| **cyclic** | 루프 실행 | 반복 처리 |
| **error** | 에러 핸들링 | try-catch 패턴 |
| **interrupt** | Human-in-the-loop | HITL 노드와 연계 |
| **await** | 다중 입력 대기 | 동기화 지점 |

---

### 5.3 Session Registry

**위치**: `kait_os_main/core/sessions/registry.py`

**SessionRegistry (RegistryBase 상속)**는 `personal`, `world`, `ephemeral` 세션 타입을 관리하며, manifest를 통해 세션 클래스를 로드합니다.
이 레지스트리는 개념상 Layer 4 구성요소지만, 실행 경로에서 참조되므로 Layer 2 문맥에도 나타납니다.

---



---

## 6. Recursion & Agent Safety (2025 AI Standards)

**위치**: `kait_os_main/core/execution/processor_types/mixins/recursion/loop.py`

코드 기준으로 재귀 정책은 "기본 제한 + 명시적 활성화" 원칙입니다.

#### 6.1 Recursion Policy
- **Inline LLM Node 기본값** (`core/orchestration/graph/nodes/llm/llm_node_runtime.py`)
  - `enable_tool_recursion=False`
  - `max_iterations` 미지정 시: 재귀 비활성 `1`, 재귀 활성 `200`
- **LLM Processor 기본값** (`core/execution/processor_types/llm/manifest.json`, `core/execution/processor_types/llm/strategies/recursion_strategy.py`)
  - manifest 기본값: `enable_tool_recursion=true`, `max_iterations=200`
  - 런타임에서 `max_iterations` 누락 시 fallback은 `0`(무제한 요청)이며, 내부 실행 상한은 `_UNLIMITED_MAX=10000`
- **무제한 요청 가드레일**
  - `max_iterations=0`이면 내부 실행 상한을 `_UNLIMITED_MAX=10000`으로 고정 (`loop_runner.py`)
- **종료 조건**
  - 도구 호출이 없으면 종료
  - 모든 tool result가 `break_recursion=true`이면 종료
  - `enable_tool_recursion=false`면 첫 iteration 후 종료
  - `RecursionState.MAX_ITERATIONS_REACHED` 상태를 명시적으로 기록

#### 6.2 Intelligent Multi-Tenant MCP
MCP 멀티테넌트 상세는 `3.6 MCP Architecture` 섹션을 단일 진실로 사용합니다.

---

## 7. Multimodal Data Pipeline & Safety

**위치**: `kait_os_main/core/common/handlers/multimodal.py`

**코드 기준 핵심 동작**

- **멀티모달 조립**
  - 텍스트 + 첨부파일 파트를 UCF content로 조립 (`build_multimodal_content`)
- **첨부파일 파싱 게이트**
  - `persist_attachments` + `custom_settings["parser_settings"]` + URL/MIME 정책으로 파싱 여부 결정
  - `attachment_parse_timeout`(기본 30s, 최대 60s), `attachment_parse_concurrency`(기본 4) 반영
- **미디어 타입 처리**
  - 이미지: `image_url` data URL 변환
  - 오디오/비디오: `inline_data`(mime_type + base64) 파트 생성
- **로컬 에셋 참조 해석 (JIT)**
  - `/api/v1/image-search/assets/{id}`, `/api/v1/video-search/assets/{id}`, `/files/...`(이미지) 경로를 LLM 직전에 base64/data URL로 치환 (`resolve_local_assets`)
- **안전 동작**
  - `filename`/`content_base64` 누락 첨부는 crash 대신 오류 엔트리로 처리
  - 파싱 실패 시 오류 텍스트로 degrade
  - 첨부 텍스트 길이 제한(`attachment_parsing.max_chars`) 적용

**범위 명시**
- `tool_call_id` 복구/Orphan 보호/시스템 메시지 분리는 멀티모달 모듈이 아니라 `recursion`/`message_repository` 계층에서 처리됩니다.

---

## 8. Layer 2.5: Execution State (실행 상태 관리)

**단일 책임 원칙(SRP) 준수**
- `ExecutionState`는 **영구 상태의 기록과 전이**만 담당하고, 실행 로직 자체는 보유하지 않는다.
- `ExecutionContext`는 **런타임 메모리 컨텍스트**만 담당하며, 영속화 책임을 갖지 않는다.
- 체크포인트 저장 계약은 Layer 0 공용 계약에 위임해 상태 모델을 비대화하지 않는다.

### 8.1 ExecutionState (DB)

**위치**: `kait_os_main/db_models/execution_state.py`

비동기 그래프 실행을 위한 **영구 상태 추적**.

**주요 필드:**

| 필드 | 설명 |
|------|------|
| `execution_id` | 고유 실행 ID (클라이언트에게 즉시 반환) |
| `session_id` | 연관된 채팅 세션 ID (선택) |
| `ai_instance_id` | 실행 대상 AI 인스턴스 ID |
| `user_id` | 실행 요청 사용자 ID |
| `status` | pending → running → completed/failed/timeout/interrupted |
| `current_node_id` | 현재 실행 중인 노드 |
| `current_step` / `total_steps` | 진행 상황 |
| `input_data` | 원본 요청 데이터 |
| `run_cache` | 노드 출력 캐시 |
| `final_output` | 최종 실행 결과 |
| `error_info` / `error_traceback` | 에러 정보 및 스택트레이스 |
| `retry_count` / `retry_of` | 재시도 추적 |

**관련 저장소 (체크포인트)**:
- `kait_os_main/db_models/checkpoint_record.py` (`checkpoints` 테이블) — Graph/HITL/Recursion 스냅샷/재개 저장
- 저장 계약: `kait_os_main/core/common/checkpoints/` *(Layer 0 공용 계약)*

**상태 전이:**
```
pending → running → completed
                 → failed
                 → timeout
                 → interrupted (HITL/취소) → running → ...
```

### 8.2 Execution Flow

```
1. POST /api/execution/execute
   → ExecutionState 생성 (status=pending)
   → HTTP 200 OK {execution_id, status, events_url, status_url}
   → 클라이언트 연결 즉시 종료 가능

2. Execution Backend 실행
   → `resolve_execution_backend()==local`: in-process task
   → 기본: arq Worker (Redis 큐)
   → status = running
   → 노드별 ExecutionState 업데이트
   → SSE 이벤트 스트리밍

3. HITL(승인/입력) 노드 도달 시
   → status = interrupted (또는 timeout)
   → 체크포인트 저장
   → (스트리밍 흐름) POST /api/v1/conversation/resume 로 재개

4. 완료
   → status = completed | failed | timeout
   → final_output / error_info 저장
```

### 8.3 API Endpoints

| Method | Endpoint | 설명 |
|--------|----------|------|
| POST | `/api/execution/execute` | 비동기 실행 시작 (200 OK) |
| GET | `/api/execution/events/{execution_id}` | SSE 이벤트 스트림 |
| WS | `/api/execution/ws/{execution_id}` | WebSocket 이벤트 스트림 |
| GET | `/api/execution/status/{execution_id}` | 현재 상태 조회 |
| GET | `/api/execution/result/{execution_id}` | 최종 결과 조회 |
| GET | `/api/execution/list` | 실행 목록 조회 |
| DELETE | `/api/execution/cancel/{execution_id}` | 실행 취소 |
| POST | `/api/execution/inject/{execution_id}` | 실행 중 입력 주입 (running 상태에서만 허용) |

### 8.4 ExecutionContext (Memory)

**위치**: `kait_os_main/core/orchestration/graph/execution_context.py`

**요청별 메모리 객체**로 실행 흐름 관리. ExecutionState(DB)와 달리 요청 스코프.

---

### 8.5 HITL Registry (Human-in-the-Loop)

**위치**: `kait_os_main/core/execution/hitl_registry.py`, `kait_os_main/core/permissions/approval_policy.py`

HITL은 도구 실행 승인/거절을 위해 **대기 토큰을 발급하고 결정 수신**하는 레지스트리를 사용합니다.
- 실행 백엔드에 따라 Redis(arq) / In-Memory(local) 하이브리드로 동작
- 결정 키 TTL은 `HITL_TTL=600`(10분)

---

## 9. Layer 3: AI Instance (AI 인스턴스)

**위치**: `kait_os_main/db_models/cognitive_architecture.py`

그래프를 중심으로 런타임에서 조립되는 **AI 실행 단위**.

```
AI Instance (DB)
├── cognitive_pipeline (그래프 JSON)
│   ├── nodes
│   ├── edges
│   └── entry_node_id
├── tags / category / profile_image_url
└── ...
```

`AIInstance.category` 역시 `kait_os_main/contracts/intelligence_taxonomy.py`를 단일 계약으로 사용합니다.
생성/수정/마이그레이션 경로 모두 dot-path canonical form으로 정규화됩니다.

### 9.1 Trigger

**위치**: `kait_os_main/triggers/`

| 타입 | 위치 | 설명 |
|------|------|------|
| **API Trigger** | `kait_os_main/triggers/api_trigger.py` | 기존 HTTP 엔드포인트를 트리거로 사용 (추가 wiring 없음) |
| **Schedule Trigger** | `kait_os_main/triggers/schedule_trigger.py` | 스케줄러 서비스에 cron 트리거 등록 |
| **Event Bus Trigger** | `kait_os_main/triggers/event_bus_trigger.py` | 이벤트 토픽 구독 후 dispatch |
| **Webhook Trigger** | `kait_os_main/triggers/webhook_trigger.py` | 웹훅 레지스트리 등록 + 서명 검증 설정 지원 |

### 9.2 Connector

**위치**: `kait_os_main/connectors/manifest.json`, `kait_os_main/db_models/user_connector.py`

- `manifest.json`: 시스템 커넥터 카탈로그/인증 방식/OAuth 설정 정의
- `user_connector.py`: 사용자별 커넥터 인스턴스/토큰/상태 저장

---

## 10. Layer 4: Session (세션층)

**위치**: `kait_os_main/core/sessions/`

Session은 **대화 히스토리/첨부파일의 상태 컨테이너**입니다.
세션 타입은 `SessionRegistry`(RegistryBase 상속, manifest 자동 발견)로 관리됩니다.

### 10.1 세션 타입

| 타입 | 위치 | 저장 방식 | 용도 |
|------|------|----------|------|
| **PersonalSession** | `kait_os_main/core/sessions/personal/personal_session.py` | DB | 1:1 대화 |
| **WorldSession** | `kait_os_main/core/sessions/world/world_session.py` | DB | 다자간 협업 (Private/Public) |
| **EphemeralSession** | `kait_os_main/core/sessions/ephemeral/ephemeral_session.py` | 인메모리 | 테스트/디버깅, DB 없이 동작 |

---

## 11. Layer 5: Direct Providers (저수준 재사용 모듈)

**위치**: `kait_os_main/core/*_direct/`, `kait_os_main/core/directs/`

Direct Provider는 **ProcessorUnit/Graph와 독립적으로 재사용 가능한 저수준 API 모듈**입니다.
- 고정 Direct 계열: `llm_direct`, `embedding_direct`, `stt_direct`, `tts_direct`, `ocr_direct`, `rag_direct`, `image_editing_direct`
- 확장 Direct 계열: `core/directs/DirectRegistry`를 통한 manifest/plugin 기반 커스텀 direct

**주요 모듈:**
- `llm_direct`, `embedding_direct`
- `stt_direct`, `tts_direct`, `ocr_direct`
- `rag_direct`, `image_editing_direct`
- `directs` (커스텀 direct 레지스트리)

---

## 12. Binding Domain: Optional Interface (Layer 외부)

**위치**: `kait_os_main/core/binding/`, `kait_os_main/routers/binding/binding.py`

Binding은 핵심 레이어(0~5) 위에 놓인 필수 실행축이 아니라, **선택적으로 붙는 인터페이스 도메인**입니다.
기본 대화 경로(`/api/v1/conversation/stream`)는 Binding 라우터와 독립적으로
`ConversationRouterOrchestrationService.stream_turn -> ConversationService.stream_turn`로 실행됩니다.

### 12.1 코드 기준 역할
- **전용 엔드포인트**: `/api/v1/binding/fill`, `/api/v1/binding/fill/stream`
- **API Prefix 계약 SSOT**: `kait_os_main/contracts/api_routes.py`에서 `/api`, `/api/v1`, voice/binding 등 공개 prefix를 선언하고, `kait_os_main/startup/router_specs.py`는 해당 상수를 참조해 등록합니다.
- **옵셔널 라우터 등록**: `kait_os_main/startup/router_setup.py`의 `_register_optional_router(...)`로 등록되며, 이는 `app.py`의 `setup_routers(...)`에서 호출됩니다. 단, 라우터 부팅은 기본 fail-close이며 `KAIT_ALLOW_BROKEN_STARTUP=true`를 명시한 경우에만 broken startup(부분 라우터 상태)을 허용합니다.
- **수명주기 부팅 정책**: `kait_os_main/startup/lifecycle.py`는 startup check를 수집하고 `context_builder`, `database_schema_bootstrap` 실패를 critical로 취급합니다. 기본 fail-close이며, `KAIT_ALLOW_DEGRADED_STARTUP=true`를 명시한 경우에만 critical failure 상태로 제한적으로 기동을 허용합니다.
- **오케스트레이터 위임 래퍼**: `BindingService`는 별도 실행 엔진이 아니라 `OrchestratorManager`/`stream_turn`/`process_one_shot`을 호출하는 어댑터입니다.

### 12.2 실행 모드 정합성 (코드 기준)
- **`BindingRequest.mode`**: `fill` / `action` / `suggest`
- 라우터 계약은 `fill`만 공개(`BindingFillRequest.mode: Literal["fill"]`)
- `action`/`suggest`는 `BindingService`에 엔트리는 있으나 현재 실패 반환(미지원) 상태입니다.
- **Ask Mode**: 별도 `mode`가 아니라 `BindingContext.state.generationMode == "ask"`일 때 프롬프트 레벨 규칙으로 적용됩니다.

---

## 13. Layer 0 (Extension): Viewport Architecture

**위치**: `kait_os_main/core/viewport/`, `kait_os_main/core/viewport/VIEWPORT_ARCHITECTURE_GUIDE.md`

**핵심 개념: Keep-Alive, Instance-Based, Multi-Slot Universal Viewport**

Viewport는 단순한 UI 렌더링 시스템이 아닌, **지능과 환경이 만나는 접점(Shared World)**입니다.

### 13.1 Shared Interaction Space
- **Context Manifestation**: 각 뷰포트 세션은 `get_binding_context()`로 상태를 Binding 컨텍스트로 노출합니다.
- **Action Projection**: 뷰포트 세션은 `apply_binding_output()`으로 Binding 출력 payload를 자신의 환경에 반영합니다.

### 13.2 Viewport Slots
현재 지원되는 주요 슬롯:
- `file`: 파일 편집/조회 슬롯
- `terminal`: 실시간 PTY 터미널 세션 (xterm.js)
- `electron_browser` (`browser` alias): Electron 기반 브라우저 제어 슬롯
- `preview`: 멀티미디어 프리뷰 (Image/PDF/Video)
- `artifact`: AI 생성 HTML/JS 실시간 렌더링
- `corpus`: 지식 그래프/코퍼스 탐색 슬롯

---

## 14. Cross-Cutting Systems

### 14.1 Expression System
**위치**: `kait_os_main/core/common/utils/expression_resolver.py`

노드 간 데이터 전달을 위한 **동적 표현식**.
```
${input}                         # 현재 입력(current_input)
${input.field}                   # 현재 입력의 특정 필드
${user_input}                    # 사용자 최초 입력
${step}                          # 현재 step
${node_id.field}                 # 특정 노드 출력 참조 (nodes[node_id] 단축)
${run_cache.key}                 # 실행 캐시 참조
${await.source_id.field}         # await 수집 결과 참조
${input.missing:default_value}   # 기본값 폴백
${input.images:multimodal}       # multimodal payload 원본 해석
{{input.field}}                  # 중괄호 이중 문법도 지원
```
- 보안 규칙: `__`로 시작하는 경로 접근은 차단됨

---

## 15. Performance & Principles

1. **DB 의존성 최소화 + 선택 경로 무DB**: `EphemeralSession`/일부 로컬 실행은 무DB 경로가 가능하지만, 기본 Conversation/Execution API는 DB 상태를 기준으로 동작.
2. **Manifest-Driven Auto-Discovery**: `ProcessorTypeRegistry`, `FilterRegistry`, `NodeTypeRegistry`, `EdgeRegistry`, `SessionRegistry`, `ViewportSlotRegistry`, `DirectRegistry`는 manifest 기반 자동 발견을 사용.
3. **RegistryBase Consistency**: 주요 레지스트리가 `RegistryBase` 싱글톤 패턴 + 중앙 `refresh/reload` 경로를 공유하며, 요청 단위 `create_snapshot()`으로 실행 중 상태 격리를 보장합니다. `SegmentRegistry`는 Python class entrypoint 대신 JSON 타입/스타일 메타데이터 중심으로 동작.
4. **Code is Truth**: 문서는 구현 가이드이며, 최종 계약은 코드에 의해 결정.

---

# Workbench 자동화 시스템 개선 TIP:

	•	오류의 정의 공간 설계
	•	정합성의 기준 설계
	•	수정 가능 범위의 경계 설계
	•	“무엇이 바뀌면 안 되는가”를 먼저 고정


  오류를 오류로 정의할 수 있는 능력
	•	실체 없는 것을 기준 삼아도 시스템이 붕괴하지 않게 만드는 능력
	•	수정 가능성과 불변성을 구분하는 능력

  	•	정합성
	  •	불변량
	  •	오류 정의
	  •	수렴 조건

	•	이 세계관에서 불변량이 무엇인가
	•	이 가정이 참일 때 반드시 따라야 할 귀결은 무엇인가
	•	그 귀결이 실제 출력에 위반되었는가
	•	위반되었다면 → 오류

시스템의 공백(Void)을 설계하고, 그 공백이 뱉어내는 비명(Error)을 워크벤치에게 먹이로 던져주는 방식
