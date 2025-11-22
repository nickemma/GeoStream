# GeoStream

> Geo-distributed platform with intelligent routing and consensus-based coordination

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Go Version](https://img.shields.io/badge/Go-1.21+-00ADD8?logo=go)](https://go.dev/)
[![Project Status](https://img.shields.io/badge/Status-Active%20Development-green)]()

**Live Demo:** [Coming Soon]  
**Documentation:** [Link to docs]

---

## 📖 Overview

GeoStream is a geo-distributed platform designed to route requests intelligently across multiple regions while maintaining low latency and high availability. Built as a **modular monolith** with clear service boundaries for future extraction based on real scaling needs.

### The Problem

Modern applications need to serve global users with low latency, but building distributed systems is complex. Common challenges include:

- **High latency** for users far from servers
- **Single points of failure** causing regional outages
- **Data consistency** across multiple regions
- **Complex coordination** between distributed services
- **Over-engineered solutions** that add unnecessary complexity

### The Solution

GeoStream provides intelligent geo-routing with fault tolerance, starting simple and scaling based on measured bottlenecks:

- 🌍 **Intelligent Geo-Routing:** Routes requests to optimal regions based on latency, cost, and availability
- 🛡️ **Fault Tolerance:** Circuit breakers, health checks, and automatic failover
- ⚡ **Performance:** Distributed caching with Redis for sub-100ms response times
- 📊 **Observability:** Built-in metrics, logging, and distributed tracing
- 🔄 **Future-Ready:** Designed for consensus-based coordination when scale demands it

---

## 🏗️ Architecture

### Current Phase: Modular Monolith
```
┌─────────────────────────────────────────────────┐
│               GeoStream Monolith                │
│                                                 │
│  ┌──────────────┐  ┌──────────────┐           │
│  │  API Layer   │  │ Middleware   │           │
│  └──────┬───────┘  └──────┬───────┘           │
│         │                  │                    │
│  ┌──────▼──────────────────▼───────┐           │
│  │      Core Business Logic         │           │
│  │                                  │           │
│  │  ┌─────────────────────────┐    │           │
│  │  │  Geography Module       │    │           │
│  │  │  - Location Detection   │    │           │
│  │  │  - Routing Strategy     │    │           │
│  │  │  - Latency Calculation  │    │           │
│  │  └─────────────────────────┘    │           │
│  │                                  │           │
│  │  ┌─────────────────────────┐    │           │
│  │  │  Health Module          │    │           │
│  │  │  - Region Monitoring    │    │           │
│  │  │  - Circuit Breakers     │    │           │
│  │  │  - Automatic Failover   │    │           │
│  │  └─────────────────────────┘    │           │
│  │                                  │           │
│  │  ┌─────────────────────────┐    │           │
│  │  │  Storage Module         │    │           │
│  │  │  - Data Access Layer    │    │           │
│  │  │  - Cache Management     │    │           │
│  │  └─────────────────────────┘    │           │
│  └──────────────────────────────────┘           │
│         │                  │                    │
│  ┌──────▼──────┐    ┌──────▼──────┐           │
│  │ PostgreSQL  │    │    Redis    │           │
│  └─────────────┘    └─────────────┘           │
└─────────────────────────────────────────────────┘
```

**Design Principles:**

1. **Clear Module Boundaries:** Each module has well-defined interfaces and responsibilities
2. **Dependency Injection:** Modules depend on interfaces, not concrete implementations
3. **Single Process:** All modules run in the same binary for simplicity and performance
4. **Observability First:** Metrics and logging at every layer to identify bottlenecks
5. **Future Extraction Ready:** Modules can become services when data shows the need

### Why Modular Monolith?

**Current Benefits:**
- ✅ Faster development (no network calls between modules)
- ✅ Simpler debugging (single codebase, unified logs)
- ✅ Lower operational overhead (one deployment, one database)
- ✅ Easier testing (integration tests without distributed complexity)
- ✅ Better performance (in-process function calls vs network RPCs)

**Future-Ready:**
- ✅ Clear module boundaries make service extraction straightforward
- ✅ Metrics identify which modules need independent scaling
- ✅ Interface-based design allows swapping implementations (local → remote)

---

## 🎯 Key Features

### 1. Intelligent Geo-Routing

Routes incoming requests to the optimal region based on multiple factors:

- **Latency:** Minimizes round-trip time to user
- **Cost:** Balances traffic across regions to optimize cloud spend
- **Availability:** Automatically routes away from degraded regions
- **Load:** Distributes traffic to prevent hotspots

**Target Performance:** <100ms P99 latency globally

### 2. Fault Tolerance

Built-in resilience patterns to handle failures gracefully:

- **Circuit Breakers:** Prevent cascading failures by failing fast
- **Health Checks:** Continuous monitoring of region availability
- **Automatic Failover:** Routes traffic away from unhealthy regions within <3s
- **Retry Logic:** Exponential backoff with jitter for transient failures
- **Rate Limiting:** Token bucket algorithm protects backend services

**Target Availability:** 99.95% uptime

### 3. Distributed Caching

Multi-layer caching strategy for optimal performance:

- **Cache-Aside Pattern:** Application controls cache population
- **Read-Through:** Automatic cache population on miss
- **Write-Through:** Synchronous cache updates on writes
- **Intelligent Invalidation:** TTL-based and event-driven invalidation

**Target Cache Hit Rate:** 80%+

### 4. Observability

Comprehensive monitoring and debugging:

- **Structured Logging:** JSON logs with request context
- **Metrics:** Prometheus-compatible metrics for all operations
- **Distributed Tracing:** Request flow across modules (ready for future services)
- **Health Endpoints:** `/health` and `/metrics` for monitoring

---

## 🛣️ Roadmap

### Phase 1: Modular Monolith MVP ✅ (Current)

**Status:** In Progress  
**Timeline:** Months 1-3  
**Goals:**
- ✅ Core geo-routing logic
- ✅ Basic health checks and circuit breakers
- 🔄 Redis caching layer
- 🔄 Observability stack (Prometheus/Grafana)
- ⏳ Deploy to single AWS region

**Success Metrics:**
- Route 1K requests/second with <100ms P99 latency
- 99.9% uptime over 30 days
- 80% cache hit rate

### Phase 2: Multi-Region Deployment (Planned)

**Status:** Not Started  
**Timeline:** Months 4-6  
**Goals:**
- Deploy identical monoliths to 3+ regions (US, EU, Asia)
- DNS-based geo-routing (Route53)
- Regional databases (no cross-region coordination yet)
- Monitor data consistency issues

**What This Teaches:**
- Real-world latency between regions
- Data consistency challenges
- Network partition scenarios
- Cost of multi-region infrastructure

**When to Start:** After Phase 1 metrics show single-region bottlenecks

### Phase 3: Consensus-Based Coordination (Future)

**Status:** Research Phase  
**Timeline:** Months 7-12  
**Prerequisites:**
- Phase 2 complete
- Observed data inconsistency problems
- Clear need for cross-region coordination

**Goals:**
- Implement Raft consensus protocol from scratch in Go
- Deploy Raft cluster (3-5 nodes) for configuration management
- Use Raft for leader election and automatic failover
- Maintain strong consistency for critical data

**Research Activities (Parallel):**
- Complete MIT 6.824 Distributed Systems course
- Implement Raft in isolation (separate learning repo)
- Study Raft paper and related consensus protocols (Paxos, Multi-Paxos)
- Build toy distributed systems to understand failure modes

**When to Start:** Only if Phase 2 metrics show coordination is bottleneck

### Phase 4: Selective Service Extraction (If Needed)

**Status:** Hypothetical  
**Timeline:** Month 12+  
**Triggers:** Metrics showing specific modules as bottlenecks

**Candidates for Extraction:**
1. **Geo-routing service** (if CPU-bound and blocking other operations)
2. **Health monitoring service** (if needs different scaling characteristics)
3. **Caching layer** (if single Redis instance becomes bottleneck)

**What Stays in Monolith:**
- Core business logic (routing decisions)
- Data access layer (unless database sharding needed)
- Simple CRUD operations

---

## 🧠 Learning Approach

This project is both a **product** and a **learning journey** in distributed systems.

### Parallel Learning Tracks

**Track 1: Building (70%)**
- Implement features in modular monolith
- Measure performance and identify bottlenecks
- Iterate based on real data

**Track 2: Studying Theory (20%)**
- MIT 6.824 Distributed Systems lectures and labs
- Reading foundational papers: Raft, Paxos, Spanner, Dynamo, Bigtable
- Understanding CAP theorem, consistency models, replication strategies

**Track 3: Experimentation (10%)**
- Implement distributed patterns in isolation (separate repos)
- Build toy systems to understand failure modes
- Test consensus algorithms without production pressure

### Why This Approach?

**Traditional Approach (Build First, Learn Later):**
- ❌ Makes costly architectural mistakes
- ❌ Over-engineers solutions without understanding trade-offs
- ❌ Struggles to debug distributed failures

**Academic Approach (Learn First, Never Build):**
- ❌ Theory without practical context
- ❌ Doesn't understand real-world constraints
- ❌ Can't apply knowledge to production systems

**This Approach (Build Simple, Study Deep, Scale Smart):**
- ✅ Validates ideas quickly with simple architecture
- ✅ Understands theory deeply through parallel study
- ✅ Applies patterns only when data shows need
- ✅ Makes informed architectural decisions based on trade-offs

---

## 🔧 Tech Stack

### Core Languages
- **Go:** Primary language for performance and concurrency
- **Rust:** System-level components requiring memory safety (future)

### Data Storage
- **PostgreSQL:** Primary data store (JSONB for flexible schemas)
- **Redis:** Caching layer (single instance → cluster when needed)

### Infrastructure
- **Kubernetes:** Container orchestration (future multi-region deployment)
- **Terraform:** Infrastructure as code
- **Docker:** Containerization

### Observability
- **Prometheus:** Metrics collection and alerting
- **Grafana:** Metrics visualization
- **OpenTelemetry:** Distributed tracing (ready for future microservices)

### Cloud Providers
- **AWS:** Primary cloud (EC2, RDS, ElastiCache, Route53, CloudWatch)
- **GCP:** Secondary cloud for multi-cloud strategy (future)

---

## 📊 Metrics & Monitoring

### Key Performance Indicators (KPIs)

**Latency Metrics:**
- P50, P95, P99 response times by region
- Cache hit/miss latency
- Database query latency

**Availability Metrics:**
- Uptime percentage per region
- Failed health checks
- Circuit breaker trips
- Failover events

**Throughput Metrics:**
- Requests per second
- Cache requests per second
- Database connections

**Business Metrics:**
- Geographical distribution of users
- Cost per request by region
- Infrastructure spend

### Monitoring Dashboards

**1. Real-Time Operations Dashboard**
- Current request rate
- Active regions and health status
- Circuit breaker states
- Cache hit rates

**2. Performance Dashboard**
- Latency heatmaps by region
- P99 latency trends
- Slow query analysis
- Cache performance

**3. Cost Dashboard**
- Spend by region
- Cost per 1M requests
- Infrastructure utilization

---

## 🚀 Quick Start

### Prerequisites
- Go 1.21+
- PostgreSQL 14+
- Redis 7+
- Docker & Docker Compose (optional)

### Local Development

1. **Clone the repository**
```bash
git clone https://github.com/yourusername/geostream.git
cd geostream
```

2. **Set up environment**
```bash
cp .env.example .env
# Edit .env with your configuration
```

3. **Run with Docker Compose (Recommended)**
```bash
docker-compose up
```

4. **Or run manually**
```bash
# Start dependencies
docker-compose up postgres redis -d

# Run migrations
make migrate-up

# Start server
make run
```

5. **Verify it's working**
```bash
curl http://localhost:8080/health
```

### Configuration

Key environment variables:
```bash
# Server
PORT=8080
ENVIRONMENT=development

# Database
DATABASE_URL=postgresql://user:pass@localhost:5432/geostream

# Redis
REDIS_URL=redis://localhost:6379

# Monitoring
METRICS_ENABLED=true
LOG_LEVEL=info
```

---

## 🧪 Testing
```bash
# Run all tests
make test

# Run with coverage
make test-coverage

# Run integration tests
make test-integration

# Run benchmarks
make benchmark
```

### Testing Strategy

- **Unit Tests:** Each module's business logic
- **Integration Tests:** Module interactions and database
- **Load Tests:** Performance under realistic traffic
- **Chaos Tests:** Behavior during failures (future)

---

## 📚 Documentation

- **[Architecture](docs/architecture.md):** Deep dive into system design
- **[API Reference](docs/api.md):** REST API documentation
- **[Deployment](docs/deployment.md):** Production deployment guide
- **[Contributing](CONTRIBUTING.md):** How to contribute
- **[Learning Path](docs/learning.md):** Distributed systems resources

---

## 🤝 Contributing

Contributions are welcome! This project is both a product and a learning exercise.

**Ways to Contribute:**
- 🐛 **Bug Reports:** Found an issue? Open an issue with reproduction steps
- 💡 **Feature Requests:** Ideas for improvement? Start a discussion
- 📝 **Documentation:** Help improve docs or add examples
- 🔧 **Code:** Submit PRs for bug fixes or features
- 📚 **Learning:** Share distributed systems resources or insights

**Before Contributing:**
1. Read [CONTRIBUTING.md](CONTRIBUTING.md)
2. Check existing issues and PRs
3. Open an issue to discuss major changes

---

## 📖 Learning Resources

Resources that shaped this project:

**Courses:**
- [MIT 6.824 Distributed Systems](https://pdos.csail.mit.edu/6.824/)
- [CMU 15-445 Database Systems](https://15445.courses.cs.cmu.edu/)

**Papers:**
- [Raft Consensus Algorithm](https://raft.github.io/raft.pdf)
- [Google Spanner](https://research.google/pubs/pub39966/)
- [Amazon Dynamo](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf)
- [CAP Theorem](https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed/)

**Books:**
- *Designing Data-Intensive Applications* by Martin Kleppmann
- *Database Internals* by Alex Petrov

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 💬 Contact

**Nicholas Emmanuel**

- Email: nicholasemmanuel321@gmail.com
- GitHub: [@nickemma](https://github.com/nickemma)
- LinkedIn: [Your LinkedIn](your-linkedin-url)

---

## 🙏 Acknowledgments

- MIT 6.824 course staff for excellent distributed systems education
- The Raft authors for a clear, implementable consensus algorithm
- Martin Kleppmann for *Designing Data-Intensive Applications*
- The Go community for excellent tooling and libraries

---

**Built with ❤️ and a commitment to understanding distributed systems deeply**

*"Whatever you do, work at it with all your heart, as working for the Lord." - Colossians 3:23*
