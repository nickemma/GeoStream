# GeoStream Architecture

**Status:** Early Development  
**Last Updated:** December 2025  
**Author:** [@Nicholas Emmanuel](https://github.com/nickemma)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Design Principles](#design-principles)
- [System Architecture](#system-architecture)
- [Module Breakdown](#module-breakdown)
- [Data Flow](#data-flow)
- [Network Architecture](#network-architecture)
- [Failure Handling](#failure-handling)
- [Future Evolution](#future-evolution)

---

## Overview

GeoStream is a **geo-distributed routing platform** built from first principles to demonstrate mastery of distributed systems concepts. The architecture is designed to be:

- **Learnable** - Clear module boundaries with explicit dependencies
- **Evolvable** - Incremental complexity (single-region → multi-region → consensus)
- **Testable** - Hexagonal architecture enables unit testing without infrastructure
- **Realistic** - Production-grade patterns from Cloudflare, AWS Route53, Google Cloud Load Balancer

**Core Philosophy:** Build the simplest thing that works, measure real bottlenecks, then evolve complexity as needed.

---

## Design Principles

### 1. Hexagonal Architecture (Ports & Adapters)

```
┌─────────────────────────────────────────┐
│         Domain Layer (Pure Logic)       │
│  - No external dependencies             │
│  - Business rules & entities            │
│  - Testable without mocks               │
└─────────────────────────────────────────┘
                    ↑
┌─────────────────────────────────────────┐
│      Application Layer (Use Cases)      │
│  - Orchestrates domain logic            │
│  - Depends on repository interfaces     │
└─────────────────────────────────────────┘
                    ↑
┌─────────────────────────────────────────┐
│    Repository Layer (Port Interfaces)   │
│  - Contracts for external systems       │
│  - Storage, cache, network              │
└─────────────────────────────────────────┘
                    ↑
┌─────────────────────────────────────────┐
│   Infrastructure Layer (Adapters)       │
│  - Concrete implementations             │
│  - PostgreSQL, Redis, HTTP client       │
└─────────────────────────────────────────┘
```

**Why Hexagonal Architecture?**

This pattern allows replacing implementations (e.g., in-memory cache → Redis) without touching business logic. The domain layer has **zero dependencies** on frameworks, databases, or external services.

**Key Benefits:**

- **Testability** - Domain logic can be tested without starting a database
- **Flexibility** - Swap implementations (PostgreSQL → MySQL) without changing business logic
- **Maintainability** - Changes to infrastructure don't cascade to domain layer
- **Clarity** - Clear boundaries between "what" (domain) and "how" (infrastructure)

### 2. Domain-Driven Design (DDD)

Each module represents a **bounded context** with:

- **Ubiquitous Language** - Terminology matches domain experts (latency, circuit breaker, failover)
- **Aggregates** - Transactional consistency boundaries (Region, Route, HealthStatus)
- **Entities & Value Objects** - Clear identity semantics
- **Domain Events** - Communication between bounded contexts

**Example Bounded Contexts:**

- **Geo-Routing Context** - Everything about routing decisions
- **Health Monitoring Context** - Everything about service health
- **Storage Context** - Everything about caching and persistence

### 3. Dependency Inversion Principle

```
High-Level Policy (Domain)
         ↓ depends on
    Interface (Port)
         ↑ implements
Low-Level Detail (Adapter)
```

- **High-level modules** (domain logic) never depend on **low-level modules** (infrastructure)
- Both depend on **abstractions** (repository interfaces)
- Enables testing business logic without databases

**Example:**

```go
// Domain layer defines the interface
type RegionRepository interface {
    GetHealthyRegions() ([]Region, error)
}

// Infrastructure layer implements it
type PostgresRegionRepo struct {
    db *sql.DB
}

func (r *PostgresRegionRepo) GetHealthyRegions() ([]Region, error) {
    // PostgreSQL-specific implementation
}
```

### 4. Single Responsibility Principle

Each module has **one reason to change**:

- **Geo-routing** - Changes when routing algorithms evolve
- **Health monitoring** - Changes when health check strategies evolve
- **Storage** - Changes when persistence requirements evolve

---

## System Architecture

### High-Level Overview

```
┌────────────────────────────────────────────────────────────┐
│                    Client Applications                      │
│              (Web Browsers, Mobile Apps, APIs)             │
└────────────────────────────────────────────────────────────┘
                            ↓ HTTP/HTTPS
┌────────────────────────────────────────────────────────────┐
│                      API Gateway (Go)                       │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                │
│  │  Auth    │→ │ Rate     │→ │  CORS    │                │
│  │  Middleware │ Limiting │  │ Middleware│               │
│  └──────────┘  └──────────┘  └──────────┘                │
└────────────────────────────────────────────────────────────┘
                            ↓
┌────────────────────────────────────────────────────────────┐
│                Application Layer (Go)                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │   Routing    │  │   Health     │  │   Config     │    │
│  │   Service    │  │   Service    │  │   Service    │    │
│  └──────────────┘  └──────────────┘  └──────────────┘    │
└────────────────────────────────────────────────────────────┘
                            ↓
┌────────────────────────────────────────────────────────────┐
│                   Domain Layer (Go)                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │  Geo-Routing │  │    Health    │  │   Storage    │    │
│  │   Domain     │  │    Domain    │  │   Domain     │    │
│  │              │  │              │  │              │    │
│  │ - Location   │  │ - Circuit    │  │ - Region     │    │
│  │   Detection  │  │   Breaker    │  │   Config     │    │
│  │ - Routing    │  │ - Health     │  │ - Cache      │    │
│  │   Strategy   │  │   Check      │  │   Mgmt       │    │
│  │ - Latency    │  │ - Failover   │  │              │    │
│  │   Calc       │  │              │  │              │    │
│  └──────────────┘  └──────────────┘  └──────────────┘    │
└────────────────────────────────────────────────────────────┘
                            ↓
┌────────────────────────────────────────────────────────────┐
│              Repository Interfaces (Ports)                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │ RegionRepo   │  │  HealthRepo  │  │  CacheRepo   │    │
│  │ Interface    │  │  Interface   │  │  Interface   │    │
│  └──────────────┘  └──────────────┘  └──────────────┘    │
└────────────────────────────────────────────────────────────┘
                            ↓
┌────────────────────────────────────────────────────────────┐
│              Infrastructure Layer (Adapters)                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │  PostgreSQL  │  │     Redis    │  │   HTTP       │    │
│  │   Adapter    │  │    Adapter   │  │   Client     │    │
│  └──────────────┘  └──────────────┘  └──────────────┘    │
└────────────────────────────────────────────────────────────┘
                            ↓
┌────────────────────────────────────────────────────────────┐
│                    External Systems                         │
│           PostgreSQL • Redis • Backend Services             │
└────────────────────────────────────────────────────────────┘
```

### Layer Responsibilities

| Layer                    | Responsibility                          | Example Components                |
| ------------------------ | --------------------------------------- | --------------------------------- |
| **API Layer**            | HTTP routing, middleware, serialization | Chi router, auth middleware       |
| **Application Layer**    | Use case orchestration                  | RoutingService, HealthService     |
| **Domain Layer**         | Business logic, pure functions          | LatencyCalculator, CircuitBreaker |
| **Repository Layer**     | Contracts for external systems          | RegionRepository interface        |
| **Infrastructure Layer** | Concrete implementations                | PostgresRegionRepo, RedisCache    |

---

## Module Breakdown

### Project Structure

```
geostream/
├── .github/
│   └── workflows/
│       ├── ci.yml                 # Automated testing
│       ├── cd.yml                 # Deployment pipeline
│       └── create_release.yml     # Release automation
├── cmd/
│   └── main.go                    # Entry point, dependency injection
├── internal/
│   ├── georouting/                # Geo-routing bounded context
│   │   ├── domain/                # Business logic (pure)
│   │   │   ├── location.go        # Location entity
│   │   │   ├── region.go          # Region entity
│   │   │   ├── route.go           # Routing strategy
│   │   │   └── latency.go         # Latency calculation
│   │   ├── application/           # Use cases
│   │   │   └── routing_service.go # Route selection orchestration
│   │   ├── repository/            # Port interfaces
│   │   │   └── region_repo.go     # RegionRepository interface
│   │   └── infrastructure/        # Adapters
│   │       ├── postgres_region.go # PostgreSQL implementation
│   │       └── memory_region.go   # In-memory (testing)
│   ├── health/                    # Health monitoring bounded context
│   │   ├── domain/
│   │   │   ├── health_status.go   # Health status entity
│   │   │   ├── circuit_breaker.go # Circuit breaker logic
│   │   │   └── failover.go        # Failover strategy
│   │   ├── application/
│   │   │   └── health_service.go  # Health check orchestration
│   │   ├── repository/
│   │   │   └── health_repo.go     # HealthRepository interface
│   │   └── infrastructure/
│   │       ├── http_health_check.go # HTTP checker
│   │       └── postgres_health.go   # PostgreSQL adapter
│   └── storage/                   # Storage bounded context
│       ├── domain/
│       │   └── cache_entry.go     # Cache entry entity
│       ├── application/
│       │   └── cache_service.go   # Cache management
│       ├── repository/
│       │   └── cache_repo.go      # CacheRepository interface
│       └── infrastructure/
│           ├── redis_cache.go     # Redis adapter
│           └── memory_cache.go    # In-memory adapter
├── pkg/                           # Shared utilities
│   ├── logger/                    # Structured logging
│   ├── metrics/                   # Prometheus metrics
│   └── config/                    # Configuration management
├── docs/
│   ├── architecture.md            # This document
│   ├── API_Reference.md           # API documentation
│   └── roadmap.md                 # Project roadmap
├── test/
│   ├── unit/                      # Unit tests
│   └── integration/               # Integration tests
├── migrations/                    # Database migrations
├── config/                        # Configuration files
│   └── config.yaml
├── monitoring/                    # Observability configs
│   ├── prometheus.yml
│   └── grafana/
│       └── dashboards/
└── util/                          # Utility scripts
    └── seed_regions.sh
```

---

### Current Modules (Phase 1-2)

#### 1. `internal/georouting` - Geo-Routing Module

**Purpose:** Intelligently route requests to optimal regions based on latency, cost, and availability.

**Domain Layer (`domain/`):**

```go
// location.go - Value Object
type Location struct {
    Latitude  float64
    Longitude float64
    IPAddress string
}

func (l Location) IsValid() bool {
    return l.Latitude >= -90 && l.Latitude <= 90 &&
           l.Longitude >= -180 && l.Longitude <= 180
}

// region.go - Entity (has identity)
type Region struct {
    ID          string
    Name        string
    Location    Location
    Endpoint    string
    IsHealthy   bool
    Capacity    int
    CurrentLoad int
}

func (r Region) LoadPercentage() float64 {
    if r.Capacity == 0 {
        return 0
    }
    return float64(r.CurrentLoad) / float64(r.Capacity) * 100
}

func (r Region) CanAcceptTraffic() bool {
    return r.IsHealthy && r.LoadPercentage() < 90
}

// route.go - Strategy Pattern
type RoutingStrategy interface {
    SelectRegion(clientLoc Location, regions []Region) (*Region, error)
}

// Latency-based routing
type LatencyBasedStrategy struct{}

func (s *LatencyBasedStrategy) SelectRegion(clientLoc Location, regions []Region) (*Region, error) {
    if len(regions) == 0 {
        return nil, errors.New("no regions available")
    }

    var optimal *Region
    minLatency := time.Duration(math.MaxInt64)

    for _, region := range regions {
        if !region.CanAcceptTraffic() {
            continue
        }

        latency := CalculateLatency(clientLoc, region.Location)
        if latency < minLatency {
            minLatency = latency
            optimal = &region
        }
    }

    if optimal == nil {
        return nil, errors.New("no healthy regions available")
    }

    return optimal, nil
}

// Cost-aware routing
type CostAwareStrategy struct{}

func (s *CostAwareStrategy) SelectRegion(clientLoc Location, regions []Region) (*Region, error) {
    // Balance between latency and cost
    // Prefer regions with lower load (lower cost)
    // Implementation details...
}

// Availability-first routing
type AvailabilityFirstStrategy struct{}

func (s *AvailabilityFirstStrategy) SelectRegion(clientLoc Location, regions []Region) (*Region, error) {
    // Prioritize healthy regions over latency
    // Implementation details...
}

// latency.go - Pure function (no dependencies)
func CalculateLatency(from, to Location) time.Duration {
    // Haversine formula for great-circle distance
    const earthRadius = 6371.0 // km

    lat1Rad := toRadians(from.Latitude)
    lat2Rad := toRadians(to.Latitude)
    deltaLat := toRadians(to.Latitude - from.Latitude)
    deltaLon := toRadians(to.Longitude - from.Longitude)

    a := math.Sin(deltaLat/2)*math.Sin(deltaLat/2) +
         math.Cos(lat1Rad)*math.Cos(lat2Rad)*
         math.Sin(deltaLon/2)*math.Sin(deltaLon/2)

    c := 2 * math.Atan2(math.Sqrt(a), math.Sqrt(1-a))
    distance := earthRadius * c

    // Approximate latency: ~1ms per 100km (speed of light in fiber)
    // Add base latency for processing
    baseLatency := 5 * time.Millisecond
    propagationLatency := time.Duration(distance/100) * time.Millisecond

    return baseLatency + propagationLatency
}

func toRadians(degrees float64) float64 {
    return degrees * math.Pi / 180
}
```

**Application Layer (`application/`):**

```go
// routing_service.go - Use Case Orchestration
type RoutingService struct {
    regionRepo repository.RegionRepository
    strategy   domain.RoutingStrategy
    logger     logger.Logger
    metrics    metrics.Collector
}

func NewRoutingService(
    regionRepo repository.RegionRepository,
    strategy domain.RoutingStrategy,
    logger logger.Logger,
    metrics metrics.Collector,
) *RoutingService {
    return &RoutingService{
        regionRepo: regionRepo,
        strategy:   strategy,
        logger:     logger,
        metrics:    metrics,
    }
}

func (s *RoutingService) GetOptimalRegion(clientIP string, service string) (*domain.Region, error) {
    startTime := time.Now()
    defer func() {
        s.metrics.RecordLatency("routing_service.get_optimal_region", time.Since(startTime))
    }()

    // 1. Detect client location from IP
    clientLoc, err := s.detectLocation(clientIP)
    if err != nil {
        s.logger.Error("failed to detect location", "ip", clientIP, "error", err)
        return nil, fmt.Errorf("location detection failed: %w", err)
    }

    // 2. Fetch available regions from repository
    regions, err := s.regionRepo.GetHealthyRegions()
    if err != nil {
        s.logger.Error("failed to get regions", "error", err)
        return nil, fmt.Errorf("failed to fetch regions: %w", err)
    }

    if len(regions) == 0 {
        s.metrics.Increment("routing_service.no_healthy_regions")
        return nil, errors.New("no healthy regions available")
    }

    // 3. Apply routing strategy
    optimal, err := s.strategy.SelectRegion(clientLoc, regions)
    if err != nil {
        s.logger.Error("route selection failed", "error", err)
        return nil, fmt.Errorf("routing strategy failed: %w", err)
    }

    s.logger.Info("optimal region selected",
        "region", optimal.ID,
        "client_ip", clientIP,
        "service", service,
    )

    s.metrics.IncrementLabeled("routing_service.region_selected", map[string]string{
        "region": optimal.ID,
    })

    return optimal, nil
}

func (s *RoutingService) detectLocation(ip string) (domain.Location, error) {
    // Use GeoIP service (ipapi.co, MaxMind, etc.)
    // For now, placeholder implementation
    // TODO: Integrate real GeoIP service
    return domain.Location{
        Latitude:  37.7749,  // San Francisco
        Longitude: -122.4194,
        IPAddress: ip,
    }, nil
}
```

**Repository Layer (`repository/`):**

```go
// region_repo.go - Port Interface (Contract)
type RegionRepository interface {
    GetHealthyRegions() ([]domain.Region, error)
    GetRegionByID(id string) (*domain.Region, error)
    GetAllRegions() ([]domain.Region, error)
    UpdateRegionHealth(id string, healthy bool) error
    UpdateRegionLoad(id string, load int) error
}
```

**Infrastructure Layer (`infrastructure/`):**

```go
// postgres_region.go - Adapter (Concrete Implementation)
type PostgresRegionRepo struct {
    db *sql.DB
}

func NewPostgresRegionRepo(db *sql.DB) *PostgresRegionRepo {
    return &PostgresRegionRepo{db: db}
}

func (r *PostgresRegionRepo) GetHealthyRegions() ([]domain.Region, error) {
    query := `
        SELECT id, name, latitude, longitude, endpoint,
               is_healthy, capacity, current_load
        FROM regions
        WHERE is_healthy = true
        ORDER BY current_load ASC
    `

    rows, err := r.db.Query(query)
    if err != nil {
        return nil, fmt.Errorf("query failed: %w", err)
    }
    defer rows.Close()

    var regions []domain.Region
    for rows.Next() {
        var region domain.Region
        err := rows.Scan(
            &region.ID,
            &region.Name,
            &region.Location.Latitude,
            &region.Location.Longitude,
            &region.Endpoint,
            &region.IsHealthy,
            &region.Capacity,
            &region.CurrentLoad,
        )
        if err != nil {
            return nil, fmt.Errorf("scan failed: %w", err)
        }
        regions = append(regions, region)
    }

    return regions, nil
}

func (r *PostgresRegionRepo) UpdateRegionHealth(id string, healthy bool) error {
    query := `UPDATE regions SET is_healthy = $1, updated_at = NOW() WHERE id = $2`
    _, err := r.db.Exec(query, healthy, id)
    return err
}

// memory_region.go - In-Memory Adapter (Testing)
type InMemoryRegionRepo struct {
    regions map[string]domain.Region
    mu      sync.RWMutex
}

func NewInMemoryRegionRepo() *InMemoryRegionRepo {
    return &InMemoryRegionRepo{
        regions: make(map[string]domain.Region),
    }
}

func (r *InMemoryRegionRepo) GetHealthyRegions() ([]domain.Region, error) {
    r.mu.RLock()
    defer r.mu.RUnlock()

    var healthy []domain.Region
    for _, region := range r.regions {
        if region.IsHealthy {
            healthy = append(healthy, region)
        }
    }
    return healthy, nil
}

func (r *InMemoryRegionRepo) Seed(regions []domain.Region) {
    r.mu.Lock()
    defer r.mu.Unlock()

    for _, region := range regions {
        r.regions[region.ID] = region
    }
}
```

**Key Concepts:**

- **Latency-Based Routing** - Select region with lowest estimated latency (haversine distance)
- **Cost-Aware Routing** - Balance traffic across regions to minimize cloud egress fees
- **Availability-First Routing** - Always route to healthy regions (circuit breaker integration)
- **Load-Aware Routing** - Avoid overloaded regions

---

#### 2. `internal/health` - Health Monitoring Module

**Purpose:** Monitor backend services, detect failures, and trigger automatic failover.

**Domain Layer (`domain/`):**

```go
// health_status.go - Entity
type HealthStatus struct {
    RegionID     string
    IsHealthy    bool
    LastCheck    time.Time
    ResponseTime time.Duration
    ErrorCount   int
}

func (h HealthStatus) IsStale(maxAge time.Duration) bool {
    return time.Since(h.LastCheck) > maxAge
}

// circuit_breaker.go - State Machine (Pure Logic)
type State int

const (
    Closed State = iota  // Normal operation
    Open                 // Failing, fast-fail requests
    HalfOpen            // Testing if service recovered
)

type CircuitBreaker struct {
    state         State
    failureCount  int
    successCount  int
    lastFailTime  time.Time
    threshold     int           // Failures before opening
    timeout       time.Duration // Time before attempting half-open
    mu            sync.RWMutex
}

func NewCircuitBreaker(threshold int, timeout time.Duration) *CircuitBreaker {
    return &CircuitBreaker{
        state:     Closed,
        threshold: threshold,
        timeout:   timeout,
    }
}

func (cb *CircuitBreaker) Call(fn func() error) error {
    cb.mu.Lock()
    defer cb.mu.Unlock()

    switch cb.state {
    case Open:
        // Check if timeout elapsed
        if time.Since(cb.lastFailTime) > cb.timeout {
            cb.state = HalfOpen
            cb.successCount = 0
        } else {
            return errors.New("circuit breaker open")
        }
    }

    // Execute function
    err := fn()

    if err != nil {
        cb.recordFailure()
    } else {
        cb.recordSuccess()
    }

    return err
}

func (cb *CircuitBreaker) recordFailure() {
    cb.failureCount++
    cb.lastFailTime = time.Now()

    if cb.state == HalfOpen {
        // Failure in half-open → back to open
        cb.state = Open
        cb.failureCount = 0
    } else if cb.failureCount >= cb.threshold {
        // Threshold exceeded → open
        cb.state = Open
        cb.failureCount = 0
    }
}

func (cb *CircuitBreaker) recordSuccess() {
    cb.failureCount = 0
    cb.successCount++

    if cb.state == HalfOpen && cb.successCount >= 2 {
        // Successful recovery → close
        cb.state = Closed
        cb.successCount = 0
    }
}

func (cb *CircuitBreaker) State() State {
    cb.mu.RLock()
    defer cb.mu.RUnlock()
    return cb.state
}

// failover.go - Strategy Pattern
type FailoverStrategy interface {
    SelectFallbackRegion(failedRegion string, regions []Region) (*Region, error)
}

type NearestHealthyStrategy struct{}

func (s *NearestHealthyStrategy) SelectFallbackRegion(
    failedRegion string,
    regions []Region,
) (*Region, error) {
    // Find nearest healthy region (excluding failed one)
    var fallback *Region
    for _, region := range regions {
        if region.ID != failedRegion && region.IsHealthy {
            if fallback == nil {
                fallback = &region
            }
            // Could compare distances here
        }
    }

    if fallback == nil {
        return nil, errors.New("no fallback region available")
    }

    return fallback, nil
}
```

**Application Layer (`application/`):**

```go
// health_service.go - Use Case Orchestration
type HealthService struct {
    healthRepo     repository.HealthRepository
    regionRepo     repository.RegionRepository
    circuitBreakers map[string]*domain.CircuitBreaker
    logger         logger.Logger
    mu             sync.RWMutex
}

func NewHealthService(
    healthRepo repository.HealthRepository,
    regionRepo repository.RegionRepository,
    logger logger.Logger,
) *HealthService {
    return &HealthService{
        healthRepo:      healthRepo,
        regionRepo:      regionRepo,
        circuitBreakers: make(map[string]*domain.CircuitBreaker),
        logger:          logger,
    }
}

func (s *HealthService) CheckHealth(regionID string) error {
    // Get or create circuit breaker for this region
    cb := s.getCircuitBreaker(regionID)

    // Get region details
    region, err := s.regionRepo.GetRegionByID(regionID)
    if err != nil {
        return fmt.Errorf("failed to get region: %w", err)
    }

    // Use circuit breaker to prevent cascading failures
    startTime := time.Now()
    err = cb.Call(func() error {
        return s.pingEndpoint(region.Endpoint)
    })
    responseTime := time.Since(startTime)

    if err != nil {
        s.logger.Warn("health check failed",
            "region", regionID,
            "error", err,
            "circuit_state", cb.State(),
        )

        // Mark region unhealthy
        s.regionRepo.UpdateRegionHealth(regionID, false)

        // Record failed health check
        s.healthRepo.RecordHealthCheck(domain.HealthStatus{
            RegionID:     regionID,
            IsHealthy:    false,
            LastCheck:    time.Now(),
            ResponseTime: responseTime,
            ErrorCount:   1,
        })

        return err
    }

    // Update health status
    s.healthRepo.RecordHealthCheck(domain.HealthStatus{
        RegionID:     regionID,
        IsHealthy:    true,
        LastCheck:    time.Now(),
        ResponseTime: responseTime,
        ErrorCount:   0,
    })

    // Mark region healthy
    s.regionRepo.UpdateRegionHealth(regionID, true)

    return nil
}

func (s *HealthService) StartPeriodicHealthChecks(interval time.Duration) {
    ticker := time.NewTicker(interval)
    defer ticker.Stop()

    for range ticker.C {
        regions, err := s.regionRepo.GetAllRegions()
        if err != nil {
            s.logger.Error("failed to get regions", "error", err)
            continue
        }

        for _, region := range regions {
            go func(r domain.Region) {
                if err := s.CheckHealth(r.ID); err != nil {
                    s.logger.Warn("periodic health check failed",
                        "region", r.ID,
                        "error", err,
                    )
                }
            }(region)
        }
    }
}

func (s *HealthService) getCircuitBreaker(regionID string) *domain.CircuitBreaker {
    s.mu.RLock()
    cb, exists := s.circuitBreakers[regionID]
    s.mu.RUnlock()

    if exists {
        return cb
    }

    s.mu.Lock()
    defer s.mu.Unlock()

    // Double-check after acquiring write lock
    if cb, exists := s.circuitBreakers[regionID]; exists {
        return cb
    }

    cb = domain.NewCircuitBreaker(5, 30*time.Second)
    s.circuitBreakers[regionID] = cb
    return cb
}

func (s *HealthService) pingEndpoint(endpoint string) error {
    client := &http.Client{
        Timeout: 5 * time.Second,
    }

    resp, err := client.Get(endpoint + "/health")
    if err != nil {
        return fmt.Errorf("health check request failed: %w", err)
    }
    defer resp.Body.Close()

    if resp.StatusCode != http.StatusOK {
        return fmt.Errorf("unhealthy status code: %d", resp.StatusCode)
    }

    return nil
}
```

**Why Circuit Breaker?**

Prevents wasting resources on repeatedly calling failing services. After threshold failures, "opens" circuit and fast-fails requests, giving the service time to recover.

---

#### 3. `internal/storage` - Storage & Caching Module

**Purpose:** Manage configuration persistence and distributed caching (Phase 3).

**Domain Layer (`domain/`):**

```go
// cache_entry.go - Entity
type CacheEntry struct {
    Key       string
    Value     []byte
    ExpiresAt time.Time
    HitCount  int
    CreatedAt time.Time
    UpdatedAt time.Time
}

func (e CacheEntry) IsExpired() bool {
    return time.Now().After(e.ExpiresAt)
}

func (e CacheEntry) TTL() time.Duration {
    if e.IsExpired() {
        return 0
    }
    return time.Until(e.ExpiresAt)
}

// cache_strategy.go - Strategy Pattern
type CacheStrategy interface {
    Get(key string) (*CacheEntry, error)
    Set(key string, value []byte, ttl time.Duration) error
    Invalidate(key string) error
}
```

**Application Layer (`application/`):**

```go
// cache_service.go - Use Case Orchestration
type CacheService struct {
    cacheRepo repository.CacheRepository
    logger    logger.Logger
    metrics   metrics.Collector
}

func NewCacheService(
    cacheRepo repository.CacheRepository,
    logger logger.Logger,
    metrics metrics.Collector,
) *CacheService {
    return &CacheService{
        cacheRepo: cacheRepo,
        logger:    logger,
        metrics:   metrics,
    }
}

// Cache-aside pattern
func (s *CacheService) GetWithFallback(
    key string,
    fetcher func() ([]byte, error),
) ([]byte, error) {
    // 1. Try cache first
    entry, err := s.cacheRepo.Get(key)
    if err == nil && !entry.IsExpired() {
        s.metrics.Increment("cache.hit")
        s.logger.Debug("cache hit", "key", key)
        return entry.Value, nil
    }

    s.metrics.Increment("cache.miss")
    s.logger.Debug("cache miss", "key", key)

    // 2. Cache miss - fetch from source
    value, err := fetcher()
    if err != nil {
        return nil, fmt.Errorf("fetcher failed: %w", err)
    }

    // 3. Populate cache
    if err := s.cacheRepo.Set(key, value, 5*time.Minute); err != nil {
        s.logger.Warn("failed to populate cache", "key", key, "error", err)
        // Don't fail the request, just return the value
    }

    return value, nil
}

// Read-through pattern
func (s *CacheService) ReadThrough(
    key string,
    loader func(key string) ([]byte, error),
) ([]byte, error) {
    // Always try cache first, populate on miss
    return s.GetWithFallback(key, func() ([]byte, error) {
        return loader(key)
    })
}

// Write-through pattern (future)
func (s *CacheService) WriteThrough(
    key string,
    value []byte,
    persister func(key string, value []byte) error,
) error {
    // 1. Write to cache
    if err := s.cacheRepo.Set(key, value, 5*time.Minute); err != nil {
        s.logger.Warn("cache write failed", "key", key, "error", err)
    }

    // 2. Write to persistent store
    return persister(key, value)
}
```

**Infrastructure Layer (`infrastructure/`):**

```go
// redis_cache.go - Redis Adapter
type RedisCacheRepo struct {
    client *redis.Client
}

func NewRedisCacheRepo(addr string) *RedisCacheRepo {
    client := redis.NewClient(&redis.Options{
        Addr: addr,
    })
    return &RedisCacheRepo{client: client}
}

func (r *RedisCacheRepo) Get(key string) (*domain.CacheEntry, error) {
    ctx := context.Background()

    val, err := r.client.Get(ctx, key).Bytes()
    if err == redis.Nil {
        return nil, errors.New("cache miss")
    }
    if err != nil {
        return nil, fmt.Errorf("redis get failed: %w", err)
    }

    ttl, err := r.client.TTL(ctx, key).Result()
    if err != nil {
        return nil, fmt.Errorf("ttl check failed: %w", err)
    }

    return &domain.CacheEntry{
        Key:       key,
        Value:     val,
        ExpiresAt: time.Now().Add(ttl),
    }, nil
}

func (r *RedisCacheRepo) Set(key string, value []byte, ttl time.Duration) error {
    ctx := context.Background()
    return r.client.Set(ctx, key, value, ttl).Err()
}

func (r *RedisCacheRepo) Invalidate(key string) error {
    ctx := context.Background()
    return r.client.Del(ctx, key).Err()
}

// memory_cache.go - In-Memory Adapter (Testing)
type InMemoryCacheRepo struct {
    cache map[string]domain.CacheEntry
    mu    sync.RWMutex
}

func NewInMemoryCacheRepo() *InMemoryCacheRepo {
    return &InMemoryCacheRepo{
        cache: make(map[string]domain.CacheEntry),
    }
}

func (r *InMemoryCacheRepo) Get(key string) (*domain.CacheEntry, error) {
    r.mu.RLock()
    defer r.mu.RUnlock()

    entry, exists := r.cache[key]
    if !exists || entry.IsExpired() {
        return nil, errors.New("cache miss")
    }

    return &entry, nil
}

func (r *InMemoryCacheRepo) Set(key string, value []byte, ttl time.Duration) error {
    r.mu.Lock()
    defer r.mu.Unlock()

    r.cache[key] = domain.CacheEntry{
        Key:       key,
        Value:     value,
        ExpiresAt: time.Now().Add(ttl),
        CreatedAt: time.Now(),
    }

    return nil
}
```

---

### Future Modules (Phase 5)

#### 4. `internal/consensus` - Raft Consensus (Phase 5)

**Purpose:** Coordinate configuration changes across multiple GeoStream instances using Raft.

**When to Implement:** Only after Phase 4 shows data inconsistency problems requiring strong consistency.

**Domain Layer (`domain/`):**

```go
// raft_state.go
type RaftState int

const (
    Follower RaftState = iota
    Candidate
    Leader
)

type LogEntry struct {
    Index   uint64
    Term    uint64
    Command []byte // Configuration change command
}

// election.go
type ElectionTimer struct {
    timeout       time.Duration
    lastHeartbeat time.Time
    mu            sync.RWMutex
}

func (t *ElectionTimer) Reset() {
    t.mu.Lock()
    defer t.mu.Unlock()
    t.lastHeartbeat = time.Now()
}

func (t *ElectionTimer) IsExpired() bool {
    t.mu.RLock()
    defer t.mu.RUnlock()
    return time.Since(t.lastHeartbeat) > t.timeout
}
```

**Application Layer (`application/`):**

```go
// raft_service.go
type RaftService struct {
    state        RaftState
    currentTerm  uint64
    votedFor     string
    log          []LogEntry
    commitIndex  uint64
    lastApplied  uint64

    peers        []string
    electionTimer *ElectionTimer
    mu           sync.RWMutex
}

func (s *RaftService) StartElection() {
    s.mu.Lock()
    s.state = Candidate
    s.currentTerm++
    s.votedFor = s.id
    s.mu.Unlock()

    // Request votes from peers
    // Implementation details...
}

func (s *RaftService) AppendEntries(entries []LogEntry) error {
    // Replicate log entries
    // Implementation details...
}
```

**Why Raft?**

Simpler than Paxos, proven correctness, widely used (etcd, CockroachDB). Only needed when configuration must be strongly consistent across regions.

---

## Data Flow

### Request Flow (Phase 1-2: Single Region)

```
Client Request (IP: 8.8.8.8)
      ↓
[API Gateway] → POST /api/v1/route
      ↓
Middleware: Authentication → Rate Limiting → Logging
      ↓
[Application Layer]
      ↓
RoutingService.GetOptimalRegion(clientIP)
      ↓
[Domain Layer]
      ↓
1. DetectLocation(clientIP) → Location{lat: 37.77, lon: -122.41}
2. RegionRepo.GetHealthyRegions() → []Region{us-west, us-east, eu-west}
3. For each region:
   - CalculateLatency(clientLoc, region.Location)
   - us-west: 45ms, us-east: 120ms, eu-west: 140ms
4. LatencyBasedStrategy.SelectRegion() → us-west (lowest latency)
      ↓
[Infrastructure Layer]
      ↓
PostgreSQL: SELECT * FROM regions WHERE is_healthy = true
      ↓
Return Response:
{
  "region": "us-west-2",
  "latency_ms": 45,
  "reason": "lowest_latency"
}
```

### Health Check Flow (Phase 2)

```
[Periodic Health Checker] (goroutine, every 30s)
      ↓
HealthService.CheckHealth("us-west-2")
      ↓
[Circuit Breaker Check]
      ↓
If state == Open:
  - Check if timeout elapsed (30s)
  - If yes → HalfOpen, attempt test request
  - If no → Fast-fail, return error
      ↓
If state == Closed or HalfOpen:
  - Attempt health check
      ↓
HTTP GET https://us-west-2.example.com/health
      ↓
Response Time: 14ms, Status: 200 OK
      ↓
If Success:
  - CircuitBreaker.recordSuccess()
  - If state == HalfOpen and 2+ successes → Closed
  - RegionRepo.UpdateRegionHealth("us-west-2", true)
  - HealthRepo.RecordHealthCheck(HealthStatus{...})
      ↓
If Failure:
  - CircuitBreaker.recordFailure()
  - If failure_count >= threshold → Open
  - RegionRepo.UpdateRegionHealth("us-west-2", false)
  - Trigger failover (route traffic to us-east-1)
```

### Caching Flow (Phase 3)

```
Client Request → API Gateway
      ↓
RoutingService.GetOptimalRegion(clientIP)
      ↓
CacheService.GetWithFallback("region_config")
      ↓
1. Check Redis: GET region_config
      ↓
   Cache Hit → Return cached regions (2ms latency)
      ↓
   Cache Miss → Fetch from PostgreSQL (45ms latency)
      ↓
2. Store in Redis: SETEX region_config 300 <data>
      ↓
3. Return data to caller
      ↓
Apply routing strategy → Return optimal region
```

**Performance Impact:**

- **Without cache:** 45ms database query per request
- **With cache (80% hit rate):** 2ms _ 0.8 + 45ms _ 0.2 = 10.6ms average

---

## Network Architecture

### Single-Region Deployment (Phase 1-3)

```
┌─────────────────────────────────────────┐
│          AWS us-west-2 (Oregon)         │
│                                         │
│  ┌────────────────────────────────┐    │
│  │     GeoStream Instance         │    │
│  │  (Go application)              │    │
│  │                                │    │
│  │  - API Gateway (port 8080)    │    │
│  │  - Routing Service            │    │
│  │  - Health Service             │    │
│  │                                │    │
│  │  Connected to:                │    │
│  │  - PostgreSQL (RDS)           │    │
│  │  - Redis (ElastiCache)        │    │
│  └────────────────────────────────┘    │
│                                         │
└─────────────────────────────────────────┘
            ↑
            │ HTTPS (port 443)
            │
      [Global Users]
```

### Multi-Region Deployment (Phase 4)

```
┌────────────────┐  ┌────────────────┐  ┌────────────────┐
│  us-west-2     │  │  us-east-1     │  │  eu-west-1     │
│  (Oregon)      │  │  (Virginia)    │  │  (Ireland)     │
│                │  │                │  │                │
│  GeoStream     │  │  GeoStream     │  │  GeoStream     │
│  Instance      │  │  Instance      │  │  Instance      │
│  + PostgreSQL  │  │  + PostgreSQL  │  │  + PostgreSQL  │
│  + Redis       │  │  + Redis       │  │  + Redis       │
└────────┬───────┘  └────────┬───────┘  └────────┬───────┘
         │                   │                   │
         └───────────────────┴───────────────────┘
              AWS VPC Peering (~50-150ms RTT)
                        ↑
                        │
                Route53 (DNS-based geo-routing)
                Latency-based routing policy
                        │
                        ↓
                  [Global Users]

User in California → us-west-2 (20ms)
User in New York → us-east-1 (15ms)
User in London → eu-west-1 (10ms)
```

**Key Characteristics:**

- **Independent Deployments:** Each region runs identical GeoStream instances
- **Regional Databases:** PostgreSQL and Redis per region (no cross-region coordination yet)
- **DNS Routing:** Route53 directs traffic to nearest region based on latency
- **VPC Peering:** Private network between regions (for future coordination)

**Challenges:**

- **Latency:** Cross-region database queries >100ms
- **Data Consistency:** Regional databases may diverge (eventual consistency)
- **Failover:** DNS propagation delay (TTL ~60s)

### Multi-Region with Raft (Phase 5)

```
┌────────────────┐  ┌────────────────┐  ┌────────────────┐
│  us-west-2     │  │  us-east-1     │  │  eu-west-1     │
│  (Leader)      │  │  (Follower)    │  │  (Follower)    │
│                │  │                │  │                │
│  GeoStream     │  │  GeoStream     │  │  GeoStream     │
│  + Raft Node   │  │  + Raft Node   │  │  + Raft Node   │
│  + PostgreSQL  │  │  + PostgreSQL  │  │  + PostgreSQL  │
│  + Redis       │  │  + Redis       │  │  + Redis       │
└────────┬───────┘  └────────┬───────┘  └────────┬───────┘
         │                   │                   │
         └───────────────────┴───────────────────┘
              Raft Consensus (gRPC heartbeats)
                 Leader sends heartbeats every 50ms
                 Followers acknowledge
                        ↑
                        │
                  [Configuration Changes]
                  - Add/Remove Region
                  - Update Circuit Breaker Thresholds
                  - Replicated via Raft log
```

**Benefit:** Configuration changes are strongly consistent across all nodes. If leader adds a new region, all followers see the same state.

---

## Failure Handling

### Failure Taxonomy

| Failure Type          | Detection                  | Recovery                        | Recovery Time | Example               |
| --------------------- | -------------------------- | ------------------------------- | ------------- | --------------------- |
| **Service Crash**     | Health check timeout (5s)  | Route to backup region          | <3s           | App server OOM        |
| **Network Partition** | HTTP request timeout       | Circuit breaker opens, failover | <3s           | AWS AZ outage         |
| **Slow Response**     | Latency threshold exceeded | Circuit breaker trips           | <30s          | Database overload     |
| **Database Failure**  | Connection error           | Use cached data, alert operator | Manual        | RDS instance failure  |
| **Region Failure**    | All health checks fail     | Redirect all traffic to backup  | <60s (DNS)    | Entire region down    |
| **Cache Failure**     | Redis connection error     | Fallback to database            | Transparent   | ElastiCache node down |

### Circuit Breaker State Machine

```
           ┌───────────────┐
           │    CLOSED     │
           │ (Normal ops)  │
           │               │
           │ Requests pass │
           │ through       │
           └───────┬───────┘
                   │
                   │ failure_count >= 5
                   ↓
           ┌───────────────┐
           │     OPEN      │
           │ (Fast-fail)   │
           │               │
           │ All requests  │
           │ rejected      │
           └───────┬───────┘
                   │
                   │ timeout = 30s
                   ↓
           ┌───────────────┐
           │  HALF-OPEN    │
           │ (Testing)     │
           │               │
           │ Allow test    │
           │ requests      │
           └───────┬───────┘
                   │
            ┌──────┴──────┐
            │             │
    success │             │ failure
            ↓             ↓
        CLOSED          OPEN
```

**State Transitions:**

1. **Closed → Open:** After 5 consecutive failures
2. **Open → Half-Open:** After 30 seconds timeout
3. **Half-Open → Closed:** After 2 successful requests
4. **Half-Open → Open:** After any failure

**Parameters (Configurable):**

- **Failure Threshold:** 5 consecutive failures
- **Timeout:** 30 seconds
- **Success Threshold (Half-Open):** 2 requests
- **Request Timeout:** 5 seconds

### Automatic Failover (Phase 2)

**Scenario: Region us-west-2 Becomes Unhealthy**

```
Time T:   us-west-2 healthy, receiving 60% of traffic
          us-east-1 healthy, receiving 40% of traffic

Time T+1s: Health check fails (HTTP 500 error)
           - CircuitBreaker.recordFailure() (count = 1)

Time T+2s: Health check fails (timeout)
           - CircuitBreaker.recordFailure() (count = 2)

Time T+5s: Health check fails (3rd consecutive)
           - CircuitBreaker.recordFailure() (count = 3)

Time T+8s: Health check fails (4th consecutive)
           - CircuitBreaker.recordFailure() (count = 4)

Time T+11s: Health check fails (5th consecutive)
            - CircuitBreaker opens (count = 5 >= threshold)
            - RegionRepo.UpdateRegionHealth("us-west-2", false)
            - RoutingService automatically selects us-east-1
            - Metric: circuit_breaker_state{region="us-west-2"} = 1

Time T+12s: New requests routed to us-east-1 (100% traffic)
            - Latency increases slightly (more load on us-east-1)

Time T+41s: CircuitBreaker timeout elapsed (30s)
            - State → Half-Open
            - Attempt test request to us-west-2

Time T+42s: Test request succeeds (HTTP 200, 15ms)
            - CircuitBreaker.recordSuccess() (count = 1)

Time T+43s: Second test request succeeds
            - CircuitBreaker.recordSuccess() (count = 2)
            - State → Closed
            - RegionRepo.UpdateRegionHealth("us-west-2", true)
            - Resume normal traffic distribution
```

**Recovery Time:** <3 seconds from detection to failover

**Observability:**

- Metrics: `circuit_breaker_state`, `region_health_status`
- Logs: "Region us-west-2 marked unhealthy, failover to us-east-1"
- Alerts: PagerDuty notification for degraded service

---

## Future Evolution

### Phase 1 → Phase 2: Basic Routing → Fault Tolerance

**Changes:**

- Add circuit breaker logic in health domain
- Implement health check service (periodic goroutine)
- Add PostgreSQL for persistent region configuration
- Automatic failover on region failure

**No Changes:**

- Routing algorithms (still latency-based)
- Single-region deployment
- API endpoints

**Migration Path:**

1. Deploy PostgreSQL, run migrations
2. Migrate in-memory regions to database
3. Start health check service
4. Monitor circuit breaker state

### Phase 2 → Phase 3: Fault Tolerance → Caching

**Changes:**

- Add Redis caching layer
- Implement cache-aside pattern in storage domain
- Add observability (Prometheus, Grafana)
- Optimize for <100ms P99 latency

**No Changes:**

- Routing logic
- Health monitoring
- Circuit breaker behavior

**Migration Path:**

1. Deploy Redis (ElastiCache)
2. Add cache service (fallback to DB if Redis down)
3. Instrument code with Prometheus metrics
4. Create Grafana dashboards

### Phase 3 → Phase 4: Single-Region → Multi-Region

**Changes:**

- Deploy to 3+ AWS regions using Terraform
- DNS-based geo-routing (Route53)
- Regional databases (independent, no coordination)
- Measure cross-region latencies

**No Changes:**

- Application code (same binary deployed everywhere)
- API contracts
- Caching strategy

**Migration Path:**

1. Provision infrastructure with Terraform
2. Deploy same GeoStream binary to all regions
3. Configure Route53 latency-based routing
4. Monitor cross-region traffic patterns

### Phase 4 → Phase 5: Independent Regions → Consensus

**Changes:**

- Implement Raft consensus from scratch
- Deploy Raft cluster for configuration management
- Strong consistency for critical data
- Cross-region coordination

**No Changes:**

- User-facing API
- Routing algorithms
- Health monitoring

**Trigger:** Only if Phase 4 metrics show:

- Configuration inconsistencies across regions
- Need for atomic configuration updates
- Split-brain scenarios

**Migration Path:**

1. Implement Raft in isolation (separate repo, thoroughly tested)
2. Add Raft service to GeoStream
3. Migrate configuration management to Raft log
4. Test extensively with chaos engineering

---

## 📚 References

**Architecture Patterns:**

- **Hexagonal Architecture:** [Alistair Cockburn](https://alistair.cockburn.us/hexagonal-architecture/)
- **Domain-Driven Design:** by Eric Evans
- **Clean Architecture:** by Robert C. Martin

**Distributed Systems:**

- **Raft Paper:** [In Search of an Understandable Consensus Algorithm](https://raft.github.io/raft.pdf)
- **CAP Theorem:** [Brewer's Conjecture](https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed/)
- **Circuit Breaker Pattern:** [Martin Fowler](https://martinfowler.com/bliki/CircuitBreaker.html)

**Cloud Infrastructure:**

- **AWS Route53:** [Routing Policies](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html)
- **AWS Multi-Region Architecture:** [Best Practices](https://docs.aws.amazon.com/whitepapers/latest/aws-multi-region-fundamentals/aws-multi-region-fundamentals.html)

**Books:**

- _Designing Data-Intensive Applications_ by Martin Kleppmann
- _Site Reliability Engineering_ by Google
- _Building Microservices_ by Sam Newman

---

## 🤝 Contributing

Architecture decisions are documented in ADRs (Architecture Decision Records). Before proposing changes:

1. Read existing architecture docs
2. Understand hexagonal architecture pattern
3. Consider testability and maintainability
4. Open an issue for discussion

**Questions?** Open an issue with the `question` label.

---

<div align="center">

**Built with ❤️ by [@Nicholas Emmanuel](https://github.com/nickemma)**

[⬆ Back to Top](#geostream-architecture)

</div>
