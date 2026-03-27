# ElevenLabs with Genesys Cloud(GC)
ElevenLabs 에서는 CCaaS 환경 구성으로 GC 를 가이드하고 있다. ([Genesys | ElevenLabs Doc](https://elevenlabs.io/docs/eleven-agents/phone-numbers/c-caa-s-integrations/genesys#overview#overview))
연동에는 별도의 개발은 필요 없고 GC 의 Audio Connector 를 통해 연동이 가능하며, GC Flow 에서 블록을 통해 호출할 수 있다. ElevenLabs 와 Audio Connector 는 `WebSocket` 을 통해 연동한다. 두 시스템은 실시간으로 양방향 오디오 스트리밍을 통해 STT 및 TTS 서비스를 제공한다. GC Flow 에서는 ElevenAgents 로 context 를 전달하기 위해 변수를 사용할 수 있으며 반대로 ElevenAgents 의 conetxt 도 변수로 전달 받을 수 있다.

## Requirements

1. Genesys Cloud CX license with bot flow capabilities
2. Administrator access to Genesys Cloud organization
3. A configured ElevenLabs account and ElevenLabs agent
4. ElevenLabs API key and Agent ID

## Agent configuration requirements

- TTS output format: Set to μ-law 8000 Hz in Agent Settings → Voice
- User input audio format: Set to μ-law 8000 Hz in Agent Settings → Advanced

## Session variables

### Input session variables

1. In Genesys flow: Define input session variables in your “Call Audio Connector” action
2. In ElevenLabs agent: Variables are available as dynamic variables and/or override configuration settings
3. Usage: Reference these variables in your agent’s conversation flow or system prompts

### Configuration overrides
기본적으로 ElevenAgents 의 "system" 으로 시작하는 변수는 업데이트할 수 없고, 아래 변수들만 가능하다.

- `system__override_system_prompt`: Overrides the agent’s system prompt.
- `system__override_first_message`: Overrides the agent’s first message to the user.
- `system__override_language`: Sets the agent’s language for this session.
- `system__override_voice_id`: Sets the agent’s voice ID for this session.

예시로 아래와 같이 ElevenAgents 의 시작 메시지를 설정 가능하다.

- `customer_name = "John Smith"`
- `system__override_first_message = "Hello! Welcome to our support line."`

### Output session variables
ElevenAgents 로 부터 받을 변수는 Data Collection 을 통해 수집한 데이터만 가능하다. 이 변수를 통하여 고객의 정보나 의도를 파악하여 GC Flow 에서 적절한 상담원으로 라우팅하도록 서비스 구현이 가능하다.
자세한 내용은 [Data Collection](#data-collection) 참고

# ElevenLabs with Avaya

# ElevenAgents

## System Prompt
System Prompt 는 AI Agent 의 성격과 정책을 나타내는 청사진이다. 고객사에서는 Agnet 의 역할, 목표, 도구에 대한 지침 및 에이전트가 해서는 안될 행동을 설명하는 guardrails 등 복잡한 구조가 필요하다. 이 Prompt 를 어떻게 구성하느냐가 Agent 의 신뢰성에 직접적인 영향일 미치기 때문에 충분한 고객과의 협의와 지속적인 모니터링을 통해 유지관리가 필요하다.

### Prompt engineering fundamentals

####  명확한 섹션 경계
마크다운 제목으로 지정된 섹션을 나누면 모델에게 도움이 됩니다. 특히 모델들은 다른 제목보다 `#Guardrails`를 신경쓰도록 설계되어 있습니다.
```
# Personality

You are a customer service agent for Acme Corp. You are polite, efficient, and solution-oriented.

# Goal

Help customers resolve issues quickly by looking up orders and processing refunds when appropriate.

# Guardrails

Never share sensitive customer data across conversations.
Always verify customer identity before accessing account information.

# Tone

Keep responses concise (under 3 sentences) unless the user requests detailed explanations.
```

#### 가능한 간결하게
모든 지시는 짧고 명확해야 하며 행동 중심으로 유지한다. 간결한 지침은 모호함과 토큰을 줄여준다.
```
# Tone

Speak in a friendly, conversational manner while maintaining professionalism.
```

#### 중요한 지시 강조
중요한 단계를 강조하고 싶은경우 "이 단계는 중요하다" 를 덧붙인다. Prompt 가 복잡하면 모델이 System Prompt 보다 맥락을 우선시 할 수 있으므로, 모델이 이러한 중요한 규칙들을 간과하지 않도록 보장한다.
```
# Goal

Verify customer identity before accessing their account. This step is important.
Look up order details and provide status updates.
Process refund requests when eligible.

# Guardrails

Never access account information without verifying customer identity first. This step is important.
```

#### 텍스트 정규화
TTS 모델, 특히 빠른 모델들은 알파벳과 같이 사람이 읽을 수 있는 문자에 대한 음성 생성에 적합하게 설계되어 있다. 따라서 "@" 과 같은 기호나 숫자들은 잘못된 발음이나 음성 환각을 일으킬 가능성이 높다. 이를 해결하기 위해 정규화 전략 선택이 필요할 수 있다. ElevenAgents 는 기본적으로 System Prompt 를 따르며 Voice 설정으로 ElevenLabs 의 [TTS normalizer](https://elevenlabs.io/docs/overview/capabilities/text-to-speech/best-practices#text-normalization) 를 적용할 수 있다.

#### 가드레일 지정
모델이 반드시 지켜야하는 규칙을 `# Guardrails` 에 나열한다. 효율적인 가드레일 설계는 [Guardrails](https://elevenlabs.io/docs/eleven-agents/best-practices/guardrails)를 참고한다.

## Models
ElevenAgents 에서 지원하는 모델은 다음과 같다. 각 모델 연동을 위해서는 별도의 API Key 가 필요 없다.
| Provider       | Model                  |
| -------------- | ---------------------- |
| **ElevenLabs** | GLM-4.5-Air            |
|                | Qwen3-30B-A3B          |
|                | GPT-OSS-120B           |
| **Google**     | Gemini 3 Pro Preview   |
|                | Gemini 3 Flash Preview |
|                | Gemini 2.5 Flash       |
|                | Gemini 2.5 Flash Lite  |
|                | Gemini 2.0 Flash       |
|                | Gemini 2.0 Flash Lite  |
| **OpenAI**     | GPT-5                  |
|                | GPT-5 Mini             |
|                | GPT-5 Nano             |
|                | GPT-4.1                |
|                | GPT-4.1 Mini           |
|                | GPT-4.1 Nano           |
|                | GPT-4o                 |
|                | GPT-4o Mini            |
|                | GPT-4 Turbo            |
|                | GPT-3.5 Turbo          |
| **Anthropic**  | Claude Sonnet 4.5      |
|                | Claude Sonnet 4        |
|                | Claude Haiku 4.5       |
|                | Claude 3.7 Sonnet      |
|                | Claude 3.5 Sonnet      |
|                | Claude 3 Haiku         |

## Workflows
Workflow 는 ElevenAgents에서 복잡한 대화 흐름을 설계할 수 있는 GUI 를 제공한다. 고객의 요구에 따라 고객사에서 동적으로 서비스할 수 있도록 ElevenAgents를 구현할 수 있다.
![Workflow Overview](https://files.buildwithfern.com/https://elevenlabs.docs.buildwithfern.com/docs/0b5b2cf9754c67ef469c08af5d13786f70ca8e0018d10e92595861abb4ed32cb/assets/images/conversational-ai/workflow-overview.png)

### Node Types

#### Subagent node
대화 흐름의 특정 시점에 Agent 동작을 수정할 수 있게 해줍니다. 각 Subagent 는 System Prompt 를 Override 하거나 자신만의 Prompt 를 정의할 수 있다. `Knowledge Base` 및 `Tools`도 별도로 지정 가능하다.

#### Say node
대화 흐름의 특정 시점에 Agent 가 고객에게 특정 음성을 안내할 수 있게 합니다. 특정 문자열을 안내할 수 도있고 Prompt 를 통해 LLM 으로 안내 가능하다.

#### Dispatch tool node
대화 흐름의 특정 시점에 도구를 실행합니다. Subagent 의 `Tools`와 달리 호출이 보장된다.

#### Agent transfer node
서로 다른 Agent 간의 대화 핸드오프를 지원합니다.

#### Transfer to number node

#### End node

## Data Collection
