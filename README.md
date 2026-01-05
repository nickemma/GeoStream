# GeoStream

<div align="center">

![Status](https://img.shields.io/badge/status-early%20development-orange)
![Go Version](https://img.shields.io/badge/go-1.25-blue)
![License](https://img.shields.io/badge/license-MIT-green)
[![CI](https://github.com/nickemma/geostream/workflows/CI/badge.svg)](https://github.com/nickemma/geostream/actions)

**A Geo-Distributed Platform with Intelligent Routing and Fault Tolerance**

_Building distributed systems from first principles, one request at a time._

[Architecture](docs/architecture.md) • [Roadmap](docs/roadmap.md) • [API Reference](docs/API_Reference.md)

</div>

---

## 🎯 What is GeoStream?

GeoStream is a **geo-distributed routing platform** built to understand how global-scale systems route traffic intelligently across multiple regions. Think of it as learning distributed systems by building the infrastructure that powers CDNs, global load balancers, and multi-region applications.

A complete implementation demonstrating mastery of:

- **Intelligent geo-routing** (latency-based, cost-aware, availability-driven)
- **Fault tolerance patterns** (circuit breakers, health checks, automatic failover)
- **Distributed caching** (Redis with cache-aside and read-through strategies)
- **Hexagonal architecture** (clean domain boundaries, testable without infrastructure)
- **Consensus-based coordination** (Raft protocol for leader election - future phase)
- **Observability** (metrics, logging, distributed tracing)

**⚠️ Important:** GeoStream is a **learning project** built to prove one engineer can understand and implement the routing intelligence behind systems like Cloudflare, AWS Route53, or Google Cloud Load Balancer. It's not production-ready.

---

## 🚀 Status

| Component                   | Status         | Description                             |
| --------------------------- | -------------- | --------------------------------------- |
| **Project Structure**       | ✅ Complete    | Hexagonal architecture, CI/CD pipeline  |
| **Geo-Routing Engine**      | 🔄 In Progress | Latency-based routing, region selection |
| **Health Monitoring**       | 🔄 In Progress | Circuit breakers, health checks         |
| **Distributed Caching**     | 📋 Planned     | Redis integration, cache strategies     |
| **Observability Stack**     | 📋 Planned     | Prometheus, Grafana, OpenTelemetry      |
| **Multi-Region Deployment** | 📋 Planned     | Deploy to 3+ AWS regions                |
| **Raft Consensus**          | 📋 Planned     | Leader election, configuration sync     |
| **Chaos Engineering**       | 📋 Planned     | Fault injection, partition testing      |

**Current Milestone:** Building core geo-routing logic and fault tolerance patterns

---

## 🏗️ Architecture

GeoStream follows a **hexagonal architecture** (ports & adapters) with clear separation of concerns:

```
┌─────────────────────────────────────────────────────────┐
│                    HTTP API Layer                        │
│              (REST endpoints, middleware)                │
└─────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────┐
│                  Application Layer (Go)                  │
│           Use Cases • Service Orchestration              │
└─────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────┬──────────────────┬──────────────────┐
│  Geo-Routing     │   Health         │   Storage        │
│  Domain          │   Domain         │   Domain         │
│                  │                  │                  │
│  - Location      │  - Circuit       │  - Region        │
│    Detection     │    Breaker       │    Config        │
│  - Routing       │  - Health        │  - Cache         │
│    Strategy      │    Check         │    Management    │
│  - Latency       │  - Failover      │                  │
│    Calc          │                  │                  │
└──────────────────┴──────────────────┴──────────────────┘
                              ↓
┌──────────────────┬──────────────────┬──────────────────┐
│  Repository      │  Repository      │  Repository      │
│  Interfaces      │  Interfaces      │  Interfaces      │
│  (Ports)         │  (Ports)         │  (Ports)         │
└──────────────────┴──────────────────┴──────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────┐
│              Infrastructure Layer (Adapters)             │
│   PostgreSQL • Redis • HTTP Client • Monitoring          │
└─────────────────────────────────────────────────────────┘
```

**Design Principles:**

- **Hexagonal Architecture** - Domain logic isolated from infrastructure
- **Dependency Inversion** - Domain depends on interfaces, not concrete implementations
- **Single Responsibility** - Each module has one reason to change
- **Testability First** - Business logic testable without databases or networks

For detailed architecture, see [`docs/architecture.md`](docs/architecture.md).

---

## 🎓 What You'll Learn

Building GeoStream teaches you the same concepts used at Cloudflare (edge routing), AWS (Route53), and Google (Global Load Balancer):

<details>
<summary><b>Geo-Routing & Traffic Management</b></summary>

- Latency-based routing algorithms
- Cost-aware traffic distribution
- Availability-driven failover
- Geographic proximity calculation (haversine formula)
- DNS-based geo-routing (Route53 patterns)

</details>

<details>
<summary><b>Fault Tolerance Patterns</b></summary>

- Circuit breaker implementation (open/half-open/closed states)
- Health check strategies (active vs passive)
- Automatic failover (<3s recovery time)
- Retry logic with exponential backoff + jitter
- Rate limiting (token bucket algorithm)

</details>

<details>
<summary><b>Distributed Caching</b></summary>

- Cache-aside pattern
- Read-through and write-through strategies
- Cache invalidation (TTL-based and event-driven)
- Redis clustering and replication
- Cache warming and priming

</details>

<details>
<summary><b>Distributed Systems (Future Phases)</b></summary>

- Raft consensus protocol (leader election, log replication)
- Handling network partitions and split-brain
- Strong consistency vs eventual consistency trade-offs
- Distributed configuration management

</details>

<details>
<summary><b>Observability & Operations</b></summary>

- Prometheus metrics (request latency, error rates, cache hit ratio)
- Distributed tracing with OpenTelemetry
- Structured logging (JSON logs with context)
- Grafana dashboards for real-time monitoring
- Chaos engineering and fault injection

</details>

---

## 📍 Roadmap

### **Phase 1: Core Routing Logic** (Current)

- [x] Project structure with hexagonal architecture
- [x] CI/CD pipeline (GitHub Actions)
- [ ] Geo-routing domain logic (latency calculation, region selection)
- [ ] Location detection (GeoIP, IP geolocation)
- [ ] Basic health checks (HTTP ping)
- [ ] In-memory region configuration

**Goal:** Single-region system with intelligent routing logic

### **Phase 2: Fault Tolerance**

- [ ] Circuit breaker implementation
- [ ] Automatic failover (route away from unhealthy regions)
- [ ] Retry logic with backoff
- [ ] Rate limiting (protect backend services)
- [ ] PostgreSQL for persistent configuration

**Goal:** Resilient system that handles failures gracefully

### **Phase 3: Caching & Performance**

- [ ] Redis integration (distributed cache)
- [ ] Cache-aside pattern implementation
- [ ] Read-through caching
- [ ] Cache invalidation strategies
- [ ] Observability stack (Prometheus, Grafana)

**Goal:** Sub-100ms P99 latency globally

### **Phase 4: Multi-Region Deployment**

- [ ] Deploy identical instances to 3+ AWS regions
- [ ] DNS-based geo-routing (Route53)
- [ ] Regional databases (no cross-region coordination yet)
- [ ] Monitor data consistency issues
- [ ] Measure real-world latencies

**Goal:** Multi-region deployment with observed trade-offs

### **Phase 5: Consensus & Coordination** (Future)

- [ ] Implement Raft consensus protocol from scratch
- [ ] Deploy Raft cluster (3-5 nodes) for config management
- [ ] Leader election and automatic failover
- [ ] Strong consistency for critical data
- [ ] Cross-region coordination

**Goal:** Consensus-based coordination (only if Phase 4 shows need)

For detailed milestones, see [`docs/roadmap.md`](docs/roadmap.md).

---

## 🛠️ Tech Stack

| Layer              | Technology                         | Why                                          |
| ------------------ | ---------------------------------- | -------------------------------------------- |
| **Application**    | Go 1.21+                           | Excellent concurrency, network libraries     |
| **Database**       | PostgreSQL 14+                     | JSONB for flexible schemas, reliability      |
| **Cache**          | Redis 7+                           | In-memory performance, clustering            |
| **RPC**            | REST (future gRPC)                 | Simple to start, gRPC for service extraction |
| **Cloud**          | AWS Multi-Region                   | Real-world deployment constraints            |
| **Observability**  | Prometheus, Grafana, OpenTelemetry | Industry-standard monitoring                 |
| **Infrastructure** | Terraform, Docker                  | Infrastructure as code, containerization     |

---

## 🚦 Quick Start

### Prerequisites

- Go 1.25
- PostgreSQL 14+
- Redis 7+
- Docker & Docker Compose (optional)

### Run Locally

```bash
# Clone the repository
git clone https://github.com/nickemma/geostream.git
cd geostream

# Set up environment
cp .env.example .env
# Edit .env with your configuration

# Run with Docker Compose (recommended)
docker-compose up

# Or run manually
docker-compose up postgres redis -d
make migrate-up
make run
```

### Verify It's Working

```bash
# Health check
curl http://localhost:8080/health

# Get optimal region for a location
curl http://localhost:8080/api/v1/route \
  -H "Content-Type: application/json" \
  -d '{"client_ip": "8.8.8.8", "service": "api"}'

# Response
{
  "region": "us-west-2",
  "latency_ms": 45,
  "reason": "lowest_latency"
}
```

---

## 📖 Documentation

- **[Architecture Overview](docs/architecture.md)** - Hexagonal architecture deep dive, module boundaries
- **[API Reference](docs/API_Reference.md)** - REST API endpoints, request/response formats
- **[Roadmap](docs/roadmap.md)** - Detailed phase-by-phase milestones

---

## 🧪 Testing

```bash
# Run all tests
make test

# Run with race detector (recommended)
make test-race

# Run with coverage
make test-coverage

# Run integration tests
make test-integration

# Run benchmarks
make benchmark
```

### Testing Strategy

- **Unit Tests** - Domain logic, pure business rules
- **Integration Tests** - Repository implementations, database interactions
- **Load Tests** - Performance under realistic traffic patterns

---

## 🌟 Why This Exists

> "I'm fascinated by how global-scale routing systems work, but most engineers never get to build them from scratch. GeoStream is my answer: a complete implementation that proves one person can still understand and build the kind of infrastructure that powers Cloudflare's edge network, AWS Route53, or Google's Global Load Balancer."

**If this project demonstrates anything, it's that:**

- Deep technical work still matters in an age of managed services
- Understanding distributed systems from first principles beats clicking buttons in AWS console
- One engineer with focus can build infrastructure that teaches fundamental concepts

This project is my **demonstration of expertise** - not just theoretical knowledge, but hands-on implementation of:

- Geo-routing algorithms used by CDNs
- Fault tolerance patterns from SRE literature
- Distributed caching strategies from high-scale systems
- Consensus protocols from academic papers (Raft)

**"I don't just use global load balancers. I build them from scratch."**

---

## 🎯 Who This Is For

- **Engineers learning distributed systems** - Follow along, ask questions, contribute
- **Infrastructure engineers** - See real-world implementation of routing patterns
- **Students** - Bridge theory (papers) with practice (running code)
- **Hiring managers** - This is what mastery looks like

---

## 🧠 Learning Approach

This project is both a **product** and a **learning journey**.

### Parallel Tracks

**Track 1: Building (70%)**

- Implement features incrementally (modular monolith first)
- Measure performance and identify real bottlenecks
- Iterate based on data, not assumptions

**Track 2: Studying Theory (20%)**

- MIT 6.824 Distributed Systems course
- Papers: Raft, Spanner, Dynamo, CAP theorem
- Understanding consistency models, replication strategies

**Track 3: Experimentation (10%)**

- Implement patterns in isolation (toy systems)
- Build distributed algorithms without production pressure
- Learn failure modes through controlled experiments

### Why This Approach?

**Build Simple → Study Deep → Scale Smart**

- ✅ Validate ideas quickly with simple architecture
- ✅ Understand theory deeply through parallel study
- ✅ Apply patterns only when data shows need
- ✅ Make informed architectural decisions based on trade-offs

---

## 📊 Key Metrics

### Performance Targets

| Metric                     | Phase 1 | Phase 3 | Phase 4 |
| -------------------------- | ------- | ------- | ------- |
| **P99 Latency**            | <200ms  | <100ms  | <150ms  |
| **Throughput**             | 1K RPS  | 10K RPS | 50K RPS |
| **Cache Hit Rate**         | N/A     | 80%+    | 85%+    |
| **Failover Time**          | N/A     | <3s     | <3s     |
| **Uptime (Single Region)** | 99.9%   | 99.95%  | 99.95%  |

### Success Metrics (End of Project)

- ✅ **Functional** - Multi-region deployment routing traffic intelligently
- ✅ **Performant** - Sub-100ms P99 latency for geo-routing decisions
- ✅ **Resilient** - Survives region failures with <3s recovery
- ✅ **Documented** - 20-30 page architecture document
- ✅ **Demonstrable** - Live demo routing traffic across regions

---

## 👤 Author

**[@nickemma](https://github.com/nickemma)** • Building distributed systems from first principles

💼 **Open to opportunities** at companies building serious infrastructure: Cloudflare, AWS, Google Cloud, Fastly, Akamai, or any team tackling global-scale routing and distributed systems.

📧 **Contact:** nicholasemmanuel321@gmail.com  
🐦 **Twitter:** [@techieemma](https://twitter.com/techieemma)  
💼 **LinkedIn:** [Nicholas Emmanuel](https://linkedin.com/in/techieemma)

---

## ⭐ Support

If you believe one engineer can still build production-grade distributed infrastructure, **star this repo** and follow along.

Let's prove that deep technical work still matters.

---

## 📜 License

Licensed under the Apache License - see the [LICENSE](LICENSE) file for details.

---

<div align="center">

**Building Systems, Building Knowledge - One Request at a Time**

_"Whatever you do, work at it with all your heart, as working for the Lord." - Colossians 3:23_

[⬆ Back to Top](#geostream)

</div>
