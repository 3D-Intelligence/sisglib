# Value Propositions

## For Researchers

### Core Benefits

**1. Focus on Your Novel Contribution**
- Stop reimplementing asset management, semantic search, and evaluation infrastructure
- Implement only what's unique about your method
- Compare against baselines in hours, not months

**Example**: Implementing a new spatial reasoning algorithm
```python
# Your novel contribution
class MyNovelPlacementStage(Stage):
    async def process(self, state: SceneState) -> SceneState:
        # Your algorithm here
        return optimized_state

# Instantly comparable to existing methods
strategy = LinearStrategy.builder()
    .add_stage(HolodeckArchitecture())      # Reuse existing
    .add_stage(HolodeckObjectSelection())   # Reuse existing
    .add_stage(MyNovelPlacementStage())     # Your contribution
    .build()
```

**2. Reproducible Research**
- Share strategy configurations, not entire codebases
- Others can validate your results exactly
- Build on prior work without re-implementation

**3. Component-Level Ablations**
- Isolate the impact of individual components
- Answer: "Does my architecture generator improve Method X's results?"
- Test hypotheses systematically

**4. Fair Benchmarking**
- Evaluate all methods on identical datasets with identical metrics
- No more apples-to-oranges comparisons
- Track progress objectively

### What You Can Build

- **Novel scene generation algorithms**: Implement your approach using the standard interface
- **Hybrid methods**: Combine rule-based, learned, and agentic components
- **Specialized stages**: Contribute object selection, placement solvers, refinement strategies
- **Evaluation protocols**: Define new metrics and benchmarks for the community

### Time Savings

**Without [sisglib](https://github.com/3D-Intelligence/sisglib)**:
- Implement asset loading: 1-2 weeks
- Build semantic search: 1-2 weeks  
- Create evaluation harness: 1-2 weeks
- Reproduce baseline: 2-4 weeks
- **Total**: 5-10 weeks before research begins

**With [sisglib](https://github.com/3D-Intelligence/sisglib)**:
- Load pre-built infrastructure: 1 hour
- Implement your novel component: 1-2 weeks
- Compare to baselines: 1 hour
- **Total**: 1-2 weeks to validated results

## For Research Labs

### Strategic Benefits

**1. Standardize Your Research Ecosystem**
- Common codebase across lab members and projects
- Run large-scale ablation studies across multiple methods
- Build comprehensive benchmarks for your research area

**2. Maximize Research Impact**
- Papers include reproducible implementations
- Other researchers build on your work instead of reimplementing
- Your methods become widely adopted through ease of use

<!-- ## For ML Engineers & Applied Scientists

### Deployment Benefits

**1. Research to Production Path**
- Prototype with local files and in-memory vectors
- Scale with S3, Pinecone, MongoDB - same code
- No refactoring between research and production

**Example**:
```python
# Research prototype
storage = await StorageAdapter.from_url("file:///local/assets")
vectors = NumpyAdapter(dimension=768)

# Production deployment (same strategy code)
storage = await StorageAdapter.from_url("s3://prod-assets")
vectors = PineconeAdapter(api_key=PINECONE_KEY)
```

**2. Backend Flexibility**
- Swap storage: filesystem → S3 → Azure → GCS
- Swap vectors: numpy → ChromaDB → Pinecone → Weaviate
- Swap metadata: JSON → MongoDB → PostgreSQL
- All via configuration, zero code changes

**3. Production-Ready Infrastructure**
- Async/await throughout for efficient I/O
- Lazy loading for massive datasets
- Batch processing for embeddings
- Error handling and retry logic
- Monitoring and logging hooks

**4. Testability**
- Mock-friendly architecture with dependency injection
- Component isolation for unit testing
- Integration test suites included
- Reproducible test scenarios

### What You Can Build

- **Production scene generation APIs**: Deploy research models at scale
- **Custom pipelines**: Combine stages optimized for your use case
- **Domain-specific applications**: Architecture, games, VR/AR, robotics
- **Hybrid systems**: Mix traditional algorithms with ML models -->

## For the Research Community

### Collective Benefits

**1. Accelerated Progress**
- Everyone builds on a common foundation
- Methods are directly comparable
- Best practices are codified and shared
- Reproducibility becomes the norm

**2. Lower Barriers to Entry**
- New researchers can start contributing immediately
- No need to understand every implementation detail
- Focus on ideas, not infrastructure

**3. Cumulative Knowledge**
- "What works" becomes clear through systematic comparison
- Component-level insights transfer across methods
- Research compounds instead of restarting

### Community Standards

[sisglib](https://github.com/3D-Intelligence/sisglib) proposes two key standards:

**[sissf (Spatial Intelligence Scene State Format)](https://github.com/3D-Intelligence/sissf)**
- Common representation for scenes at all generation stages
- Enables intermediate result sharing
- Supports diverse output formats (USD, glTF, custom)

**SceneGenerationStrategy Interface**
- Unified abstraction: `prompt → SceneState`
- Supports linear, learned, and agentic approaches
- Enables fair comparison and component reuse

These standards are open, and succeed only through community adoption and evolution.

## Comparison to Existing Tools

### vs. Custom Implementations

| Aspect               | Custom Code | [sisglib](https://github.com/3D-Intelligence/sisglib) |
|--------              |-------------|---------                                              |
| Time to first result | Weeks       | Hours                                                 |
| Reproducibility      | Manual      | Automatic                                             |
| Comparability        | Difficult   | Built-in                                              |
| Backend flexibility  | None        | Full                                                  |
| Maintenance          | You         | Community                                             |

### vs. [Holodeck](https://yueyang1996.github.io/holodeck/), [DiffuScene](https://tangjiapeng.github.io/projects/DiffuScene/), etc.

These are scene generation *methods*. [sisglib](https://github.com/3D-Intelligence/sisglib) is a *framework* to enables implementing, evaluating, and comparing methods like these.

<!-- ## Key Design Principles

### 1. Interface Over Implementation

We standardize *what* methods must do (prompt → scene), not *how* they do it. This preserves research freedom while enabling interoperability.

### 2. Composability First

Every component is designed to work with others. Mix and match stages, strategies, and backends without friction.

### 3. Research-Production Continuum

Same code from prototype to deployment. Configuration changes, algorithms stay the same.

### 4. Community-Driven Evolution

Standards and interfaces evolve based on community needs. Early adoption shapes the future.

## Get Started

### Researchers
1. Clone the repository
2. Run the quickstart example
3. Implement your method as a custom strategy
4. Benchmark against baselines
5. Share your results

### Labs
1. Review integration documentation
2. Wrap one existing method as proof-of-concept
3. Establish lab conventions on top of [sisglib](https://github.com/3D-Intelligence/sisglib)
4. Contribute reference implementations
5. Collaborate on benchmarks

### Engineers
1. Explore example strategies
2. Configure backends for your infrastructure
3. Build application-specific pipelines
4. Deploy to production
5. Share production insights

---

**Questions? See our [FAQ](docs/FAQ.md) or [open an issue](https://github.com/3D-intelligence/sisglib/issues).** -->