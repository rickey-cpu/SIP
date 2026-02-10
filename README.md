# Mermaid Architecture Pack

## AI Callbot – Bot → Phone (Stringee Integration)

> Full technical architecture documentation using Mermaid diagrams
> Use in GitHub, GitLab, Notion, MkDocs, Docusaurus, Obsidian

---

# 1. System Architecture

```mermaid
flowchart LR
    CRM[CRM / ERP / Database]
    USER[Phone User ☎]

    ORCH[Call Orchestrator<br/>(Workflow + State Machine)]
    ADAPTER[Stringee Adapter<br/>(Call API + Webhook)]
    STRINGEE[Stringee Platform]
    PSTN[PSTN Gateway]

    MEDIA[Media Stream Service<br/>(RTP/WebRTC Bridge)]
    BOT[AI Bot Engine<br/>(STT / TTS / NLP / LLM)]

    CRM --> ORCH
    ORCH --> ADAPTER
    ADAPTER --> STRINGEE
    STRINGEE --> PSTN
    PSTN --> USER

    USER --> PSTN
    PSTN --> STRINGEE
    STRINGEE --> ADAPTER
    ADAPTER --> ORCH

    STRINGEE <--> MEDIA
    MEDIA <--> BOT
```

---

# 2. Call Flow Sequence

```mermaid
sequenceDiagram
    participant Scheduler
    participant Orchestrator
    participant Stringee
    participant PSTN
    participant User
    participant Media
    participant Bot

    Scheduler->>Orchestrator: create_call()
    Orchestrator->>Stringee: POST /call2/call
    Stringee->>PSTN: Dial phone
    PSTN->>User: Ringing
    User->>PSTN: Answer
    PSTN->>Stringee: Call connected
    Stringee->>Orchestrator: webhook(call.answered)

    Stringee->>Media: attach stream
    Media->>Bot: audio stream

    Bot->>Media: TTS audio
    Media->>Stringee: RTP audio
    Stringee->>PSTN: Voice
    PSTN->>User: Speak
```

---

# 3. Bot State Machine

```mermaid
stateDiagram-v2
    [*] --> IDLE
    IDLE --> CALLING
    CALLING --> CONNECTED
    CONNECTED --> GREETING
    GREETING --> LISTENING
    LISTENING --> UNDERSTANDING
    UNDERSTANDING --> DECISION

    DECISION --> TRANSFER : qualified
    DECISION --> END : rejected
    DECISION --> ANSWER : question

    ANSWER --> LISTENING
    TRANSFER --> END
    END --> [*]
```

---

# 4. Media Flow

```mermaid
flowchart LR
    MIC[Phone Mic] --> PSTN
    PSTN --> STRINGEE[Stringee]
    STRINGEE --> MEDIA[Media Stream Service]
    MEDIA --> BOT[AI Bot Engine (STT)]

    BOT --> MEDIA
    MEDIA --> STRINGEE
    STRINGEE --> PSTN
    PSTN --> SPK[Phone Speaker]
```

---

# 5. Microservice Map

```mermaid
flowchart TB
    subgraph Core
        ORCH[call-orchestrator]
        ADAPTER[stringee-adapter]
        STATE[state-machine]
    end

    subgraph AI
        BOT[bot-engine]
        NLP[nlp-service]
        TTS[tts-service]
        STT[stt-service]
    end

    subgraph Media
        MEDIA[media-stream]
    end

    subgraph Data
        REDIS[Redis]
        DB[(Postgres)]
        KAFKA[Kafka]
    end

    ORCH --> ADAPTER
    ORCH --> STATE
    ORCH --> MEDIA

    MEDIA --> BOT
    BOT --> NLP
    BOT --> STT
    BOT --> TTS

    ORCH --> REDIS
    ORCH --> DB
    ORCH --> KAFKA
```

---

# 6. Network Topology

```mermaid
flowchart LR
    Internet --> LB[Load Balancer]
    LB --> API[API Gateway]

    API --> ORCH
    API --> ADAPTER

    ORCH --> MEDIA
    MEDIA --> BOT

    ADAPTER --> STRINGEE[Stringee Cloud]
    STRINGEE --> PSTN[PSTN Network]
```

---

# 7. Deployment (Kubernetes)

```mermaid
flowchart TB
    subgraph K8s_Cluster
        ORCH[call-orchestrator]
        ADAPTER[stringee-adapter]
        MEDIA[media-stream]
        BOT[bot-engine]
        NLP[nlp]
        STT[stt]
        TTS[tts]
    end

    subgraph Infra
        REDIS[(Redis)]
        DB[(Postgres)]
        KAFKA[(Kafka)]
    end

    ORCH --> ADAPTER
    ORCH --> MEDIA
    MEDIA --> BOT
    BOT --> NLP
    BOT --> STT
    BOT --> TTS

    ORCH --> REDIS
    ORCH --> DB
    ORCH --> KAFKA
```

---

# 8. Data Flow

```mermaid
flowchart LR
    AudioIn[Audio Input] --> STT
    STT --> Text
    Text --> NLP
    NLP --> Intent
    Intent --> Decision[Decision Engine]
    Decision --> TTS
    TTS --> AudioOut[Audio Output]
```

---

# 9. Business Flow

```mermaid
flowchart LR
    Start([Start Call]) --> Greet[Bot Greeting]
    Greet --> Ask[Ask Question]
    Ask --> Classify[Intent Classification]

    Classify -->|Interested| Transfer[Transfer to Sale]
    Classify -->|Not Interested| EndCall[End Call]
    Classify -->|Question| Answer[Bot Answer]

    Answer --> Ask
    Transfer --> EndCall
    EndCall --> Stop([End])
```

---

# 10. Security Flow

```mermaid
flowchart LR
    API --> JWT[JWT Auth]
    JWT --> ORCH

    ORCH --> HMAC[HMAC Verify]
    HMAC --> ADAPTER

    MEDIA --> MTLS[mTLS]
    MTLS --> BOT
```

---

# END OF ARCHITECTURE PACK
