# Vision: Standardizing 3D Scene Generation Research

## The Problem

3D scene generation research is fragmented. 
Methods like [DiffuScene](https://tangjiapeng.github.io/projects/DiffuScene/), [Holodeck](https://yueyang1996.github.io/holodeck/), [HSM](https://3dlg-hcvc.github.io/hsm/), and [LayoutGPT](https://layoutgpt.github.io/) use formats, metrics, and evaluation protocols incompatible with each other. 
This fragmentation creates critical challenges:

- **Incomparability**: "Method A achieves X% on dataset D1" vs "Method B achieves Y% on dataset D2" - it is difficult to know which is better
- **Unreproducibility**: Each paper reimplements infrastructure from scratch, making validation difficult
- **Slow iteration**: Researchers spend time on boilerplate instead of novel algorithms
- **Component isolation**: Cannot test whether learned placement improves rule-based methods, or vice versa

**The field needs infrastructure that enables systematic comparison and accelerates research velocity.**

## Our Vision

[sisglib](https://github.com/3D-Intelligence/sisglib) aims to become **the standard infrastructure for 3D scene generation research** - analogous to what OpenAI Gym did for reinforcement learning or ImageNet/COCO did for computer vision.

### Core Principles

1. **Common Interface, Diverse Approaches**
   - Any method (linear, learned, agentic, hybrid) implements the same interface: `prompt → SceneState`
   - Internal implementation is unconstrained - methods can be fundamentally different while remaining interoperable

2. **Standards Enable Science**
   - **[sissf (Spatial Intelligence Scene State Format)](https://github.com/3D-Intelligence/sissf)**: Standard representation for scenes, enabling fair comparison
   - **SceneGenerationStrategy Interface**: Standard abstraction for algorithms, enabling component reuse
   - **Unified Evaluation**: Same benchmarks and metrics across all methods

3. **Composability Accelerates Innovation**
   - Mix components from different methods (e.g. [Holodeck](https://yueyang1996.github.io/holodeck/)'s object selection + [DiffuScene](https://tangjiapeng.github.io/projects/DiffuScene/)'s placement)
   - Build hybrid approaches by combining linear, learned, and agentic components
   - Ablate individual components without reimplementing entire systems

4. **Research to Production**
   - Prototype locally with minimal dependencies
   - Scale to production by swapping backends (storage, vectors, metadata)
   - Same research code, different configuration

## What We Enable

### Fair Method Comparison

```python
# Compare fundamentally different approaches on identical benchmarks
methods = [
    HolodeckStrategy.build(),      # Linear pipeline
    DiffuSceneStrategy.build(),    # End-to-end learned
    AgenticLLMStrategy.build(),    # LLM-directed
]

benchmark = SceneBenchmark(
    dataset=load_dataset("structured3d_prompts"),
    metrics=[CollisionRate(), SemanticCoherence(), SpatialPlausibility()]
)

for method in methods:
    results = await benchmark.evaluate(method)
    # Now we can actually answer: "Which method is better?"
```

### Component-Level Ablations

```python
# Question: Does learned placement beat constraint-based?
baseline = LinearStrategy.builder()
    .add_stage(RuleBasedArchitecture())
    .add_stage(ConstraintBasedPlacement())
    .build()

learned = LinearStrategy.builder()
    .add_stage(RuleBasedArchitecture())  # Same architecture
    .add_stage(LearnedPlacement())        # Only placement differs
    .build()

# Isolate the impact of placement method
compare([baseline, learned], metrics=[CollisionRate(), SceneQuality()])
```

### Cross-Method Innovation

```python
# "Does Holodeck's object selection improve DiffuScene?"
hybrid = DAGStrategy.builder()
    .add_node("holodeck_selection", HolodeckObjectSelection())
    .add_node("diffusion_placement", DiffusionPlacement())
    .add_edge("holodeck_selection", "diffusion_placement")
    .build()

# Test combinations impossible without a common framework
```

### Reproducibility by Default

```yaml
# Share complete experimental configuration
strategy:
  type: linear
  stages:
    - type: holodeck_architecture
      version: 1.0.2
    - type: learned_placement
      checkpoint: weights/best.pt

benchmark:
  dataset: structured3d_v2
  metrics: [collision, coherence]
  seed: 42
```

Anyone can reproduce your exact experiment with identical results.


## Success Metrics

[sisglib](https://github.com/3D-Intelligence/sisglib) succeeds when:

1. **Methods are comparable**: Researchers routinely compare their work against baselines using [sisglib](https://github.com/3D-Intelligence/sisglib) benchmarks
2. **Components are reusable**: Papers cite using "X's object selection with our placement method"
3. **Reproduction is trivial**: "We used config Y from the [sisglib](https://github.com/3D-Intelligence/sisglib) repository"
4. **Innovation accelerates**: Time from idea to validated result decreases significantly
5. **Community adopts**: Major papers implement their methods in [sisglib](https://github.com/3D-Intelligence/sisglib) alongside custom codebases

## Join Us

[sisglib](https://github.com/3D-Intelligence/sisglib) is open-source (Apache 2.0) because standardization only works through community adoption. We invite:

- **Researchers**: Implement your methods, contribute benchmarks
- **Labs**: Integrate with your existing work
- **Industry**: Build production systems on research foundations
- **Everyone**: Provide feedback, report issues, suggest improvements

**Together, we can accelerate 3D scene generation research.**

<!-- ---

*For technical details, see the [Architecture Documentation](docs/guides/architecture.md)*  
*For contribution guidelines, see [CONTRIBUTING.md](CONTRIBUTING.md)* -->