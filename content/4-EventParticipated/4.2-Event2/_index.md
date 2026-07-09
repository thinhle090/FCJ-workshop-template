---
title: "Event 2 - AWS Vietnam Community Day 2026"
date: 2026-05-23
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

| Info | Details |
|---|---|
| Date | 23/05/2026 |
| Location | Floor 26, Bitexco Financial Tower, Sài Gòn Ward, Ho Chi Minh City |
| Role | Attendee |

This page summarizes the sessions from **AWS Vietnam Community Day 2026**, covering AI context design, edge infrastructure, hackathon experience, LLM reliability, and enterprise-grade multi-agent systems.


### Overall Theme

> AI is not just a demo tool. It requires proper context, architecture, and deployment processes.

- AWS serves as the foundation for infrastructure, security, operations, and AI system scaling
- All sessions emphasized practicality: from prompt crafting and hackathons to CloudFront optimization and enterprise multi-agent systems


### 1. Introduction

AWS Vietnam Community Day 2026 brought together practitioners sharing real-world experience building AI and cloud systems on AWS. Sessions covered a wide range of topics from individual productivity with AI, business assistant tools, CDN infrastructure, hackathon project building, LLM reliability concerns, and enterprise-grade credit scoring with multi-agent architecture.


### 2. Session Summaries

#### 2.1 Tinh Truong - Build Second Brain

This session focused on working effectively with AI through proper **context management**. The speaker emphasized that AI models are already powerful, but results are often poor because users either provide insufficient context or provide context that misses the point.

**What good context includes:**
- The goal to be achieved
- The current situation
- Technical constraints
- Relevant evidence

**Common mistakes:**
- Providing too many unfiltered documents
- Copy-pasting entire long files
- Stating obvious things the AI already knows

**Core principle:**
> Context quality matters more than context quantity.

**Second AI Brain concept:**
A personal knowledge organization system that helps you recall projects and retrieve the right information before querying AI.

**Key takeaway:** A skilled AI user is someone who can convert vague requests into a task with a clear objective, data, and expected output.

#### 2.2 Phạm Nguyễn Hải Anh - Friendly AI Assistant with Amazon Quick Suite

This session addressed the pain points of business users and PMs: managing many documents, meetings, emails, data, and repetitive tasks. **Amazon Quick Suite** was introduced as an AI-powered assistant built on Bedrock, web search, and internal data to streamline these workflows.

**Amazon Quick Suite features:**
- Chat-based Q&A
- Research and intelligent search
- BI dashboards
- Workflow automation
- API embedding into enterprise workflows
- Powered by: Amazon Bedrock + Web Search + Internal Data

**Demo use case:**
An AI PM assistant that automatically creates Meeting Minutes (MoM), sends emails to stakeholders, and schedules the next meeting.

**Key value:**
- Reduces time spent on repetitive tasks and information gathering
- Lets users focus on decisions and team coordination

**Key takeaway:** AI delivers the most value when embedded in the right workflow, understands internal data, and supports next actions.


#### 2.3 Nguyễn Tuấn Thịnh - From Edge To Origin: CloudFront as Your Foundation

This session explored **Amazon CloudFront** as a complete foundation from edge to origin, covering cost, security, performance, and reliability at scale.

**Cost model:**
- Fixed-price bundle including CDN, WAF, DDoS, DNS, and logging
- Predictable pricing suitable for small website owners, business users, and scaling businesses
- Handles traffic spikes without unexpected cost surges

**Security features:**

| Feature | Description |
|---|---|
| DDoS Protection | Shield against volumetric attacks |
| WAF | Web Application Firewall |
| DNS | Route 53 integration |
| TLS / mTLS | Free TLS plus mutual TLS for encrypted connections |
| Signed URL | Authenticated content delivery |
| Origin Cloaking | Hide the origin server from public access |

**Performance optimizations:**
- Multi-layer caching at edge to reduce origin load and optimize bandwidth
- HTTP/3 support
- Compression
- Persistent connections to reduce origin load
- Edge functions for low-latency logic execution

**Reliability features:**
- Stale content serving during origin issues
- Origin failover
- Intelligent routing

**Key takeaway:** CloudFront is not just a CDN. It is a foundation layer for cost optimization, security, performance, and fault tolerance.


#### 2.4 Team VIB - 36 Hours with LotusHacks: Building UTMorpho from Idea to Reality

Team VIB shared the real story of participating in the **LotusHacks hackathon** over 36 hours, from having no idea to building a working demo of **UTMorpho**.

**What UTMorpho does:**
Users can take a photo, draw, or upload a UI sketch, and AI will generate a web interface from it.

**Architecture used:**
```text
CloudFront -> API Gateway -> Lambda -> Bedrock -> S3 / DynamoDB
```

**AI agent pipeline:**
1. **Vision Analyst** - Interprets the sketch input
2. **UI Designer** - Converts it into a design specification
3. **Coder** - Generates the actual code

**Key challenges faced:**
- Token limits from LLM context windows
- AI overgeneration producing noisy output
- Pitch time pressure
- Scope creep from too many ideas

**Key takeaway:** Real frustration generates real ideas. Hackathons require strong teamwork, tight scope control, and focus on one genuinely useful core experience.


#### 2.5 Đào Đức - Non-Determinism of "Deterministic" LLM Settings

This session tackled a critical technical question: **Why do LLMs still produce different outputs even with `temperature=0`?** This matters greatly for high-stakes systems like legal, financial, and medical information retrieval.

**How LLMs generate tokens:**
- Logit computation -> Softmax -> Sampling
- `temperature` only adjusts the probability distribution; it does not eliminate all sources of non-determinism

**Experiments conducted:**
- **5 models tested:** GPT-3.5, GPT-4o, Llama-3 70B, Llama-3 8B, Mixtral 8x7B
- **8 tasks x 10 runs** per model; results showed significant accuracy variance across identical runs

**Technical causes:**

| Cause | Explanation |
|---|---|
| Floating-point arithmetic | GPU operations are not perfectly deterministic |
| Parallel execution order | GPU thread scheduling varies |
| Batching inference | Provider-side batching affects the order of operations |

**Mitigation strategies:**
- Run multiple times and use **majority voting**
- Use **structured output** such as JSON, regex, or grammar-based constraints
- Implement **regression testing** for output stability
- **Self-host** the model when full inference control is needed
- Design systems to **tolerate variance** from the ground up

**Sweet spot:** `temperature ~= 0.1` balances stability and output quality better than strict `temperature=0`

**Key takeaway:** `temperature=0` is not a reliability guarantee. Systems must be designed to handle variance from the start.


#### 2.6 Vy Lam - Enterprise-Grade Multi-Agent System: Startup Credit Scoring

This session presented a **multi-agent credit scoring system** for startups, a domain where traditional credit assessment models fail because startup data is fundamentally different from established businesses.

**Why traditional credit scoring fails for startups:**
- Requires long financial history, collateral, and stable revenue patterns
- Startups typically only have traction, team quality, IP, and unstructured data

**Startup data dimensions:**

| Dimension | Examples |
|---|---|
| Financial | Revenue, burn rate, runway |
| Market | TAM, competitive landscape |
| Team | Experience, background, diversity |
| Traction | User growth, retention, partnerships |

**System design - Virtual Credit Committee:**

| Agent Role | Responsibility |
|---|---|
| Manager | Orchestrates the overall assessment |
| Financial Analyst | Evaluates financial metrics |
| Market Analyst | Assesses market opportunity |
| Team Evaluator | Reviews the founding team |
| Risk Assessor | Identifies risk factors |
| Compliance Agent | Ensures regulatory compliance |

**Output requirements:**
- Credit score
- Risk rating
- Confidence level
- Audit trail with explainable decisions

**Enterprise-grade considerations - 6 Pillars:**

| Pillar | Scope |
|---|---|
| Security | Authentication, authorization, encryption |
| Data Governance | Data lineage, access control, retention |
| Networking | VPC isolation, private endpoints |
| Operations | Monitoring, alerting, incident response |
| Human Factors | Explainability, reviewer workflows |
| Compliance | Regulatory alignment, audit readiness |

**Guardrails - Three Layers (Input -> Processing -> Output):**

| Layer | Controls |
|---|---|
| Input | Content filtering, PII detection, injection prevention |
| Processing | Model selection controls, inference constraints |
| Output | Response validation, compliance verification |

**Deployment progression:**
```text
Local App / CrewAI -> AgentCore -> Docker -> ECR -> Bedrock -> API Gateway
                    + VPC, IAM, Secrets, Monitoring, Autoscaling, DR Strategy
```

**Expected ROI:**
- Processing time: weeks -> hours
- Reduced analyst hours
- Higher approval accuracy through multi-dimensional evaluation


![Photo from Event 2](/images/b2.jpg)


### 4. Conclusion

AWS Vietnam Community Day 2026 demonstrated that AI's value lies not in the model alone, but in how context is provided, how architecture is designed, how security and guardrails are implemented, and how systems are operated at real-world scale.

| Audience | Key Takeaway |
|---|---|
| Individuals | Learn to provide quality context and build a Second Brain for AI |
| Product Teams | Prioritize real problems, limit scope, and embed AI in specific workflows |
| Infrastructure | Use CloudFront and AWS services for performance, security, and reliability |
| Enterprise | Multi-agent systems need guardrails, audit trails, compliance, and clear ROI before production |

> **Overall message:** AI only truly creates value when combined with product thinking, the right system architecture, and reliable operational processes.
