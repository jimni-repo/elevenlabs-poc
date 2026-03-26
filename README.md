# ElevenLabs POC

## ElevenLabs with Genesys Cloud(GC)
ElevenLabs 에서는 CCaaS 환경 구성으로 GC 를 가이드하고 있다. ([Genesys | ElevenLabs Doc](https://elevenlabs.io/docs/eleven-agents/phone-numbers/c-caa-s-integrations/genesys#overview#overview))
연동에는 별도의 개발은 필요 없고 GC 의 Audio Connector 를 통해 연동이 가능하며, GC Flow 에서 블록을 통해 호출할 수 있다. ElevenLabs 와 Audio Connector 는 `WebSocket` 을 통해 연동한다. 두 시스템은 실시간으로 양방향 오디오 스트리밍을 통해 STT 및 TTS 서비스를 제공한다. GC Flow 에서는 ElevenAgents 로 context 를 전달하기 위해 변수를 사용할 수 있으며 반대로 ElevenAgents 의 conetxt 도 변수로 전달 받을 수 있다.
### Requirements
1. Genesys Cloud CX license with bot flow capabilities
2. Administrator access to Genesys Cloud organization
3. A configured ElevenLabs account and ElevenLabs agent
4. ElevenLabs API key and Agent ID
### Agent configuration requirements
- TTS output format: Set to μ-law 8000 Hz in Agent Settings → Voice
- User input audio format: Set to μ-law 8000 Hz in Agent Settings → Advanced
### Session variables
#### Input session variables
1. In Genesys flow: Define input session variables in your “Call Audio Connector” action
2. In ElevenLabs agent: Variables are available as dynamic variables and/or override configuration settings
3. Usage: Reference these variables in your agent’s conversation flow or system prompts
#### Configuration overrides
기본적으로 ElevenAgents 의 "system" 으로 시작하는 변수는 업데이트할 수 없고, 아래 변수들만 가능하다.
- `system__override_system_prompt`: Overrides the agent’s system prompt.
- `system__override_first_message`: Overrides the agent’s first message to the user.
- `system__override_language`: Sets the agent’s language for this session.
- `system__override_voice_id`: Sets the agent’s voice ID for this session.
예시로 아래와 같이 ElevenAgents 의 시작 메시지를 설정 가능하다.
- `customer_name = "John Smith"`
- `system__override_first_message = "Hello! Welcome to our support line."`
#### Output session variables
ElevenAgents 로 부터 받을 변수는 Data Collection 을 통해 수집한 데이터만 가능하다. 이 변수를 통하여 고객의 정보나 의도를 파악하여 GC Flow 에서 적절한 상담원으로 라우팅하도록 서비스 구현이 가능하다.

자세한 내용은 [Data Collection](https://github.com/jimni-repo/elevenlabs-poc/edit/main/README.md#data-collection) 참고

## ElevenLabs with Avaya

## ElevenAgents
### Data Collection
