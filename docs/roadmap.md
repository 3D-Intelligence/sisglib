# Technical Roadmap

## Current State: Linear Pipelines (v0.1)

### What's Supported

**Architecture**:
```python
strategy = LinearStrategy.builder()
    .add_stage(StageA())
    .add_stage(StageB())
    .add_stage(StageC())
    .build()
```

**Execution Model**: Sequential stage execution, each stage transforms the `SceneState`

**Supported Methods**:
- Linear generation (E.g. [Holodeck](https://yueyang1996.github.io/holodeck/))
- Rule-based pipelines
- Simple sequential workflows

**Limitations**:
- No branching or conditionals
- No parallel execution
- No iterative refinement loops
- Cannot express agentic strategies
- Difficult to implement end-to-end learned methods

### Reference Implementation: [Holodeck](https://yueyang1996.github.io/holodeck/)

```python
class HolodeckStrategy(LinearStrategy):
    """Complete Holodeck implementation as linear pipeline"""
    
    @classmethod
    def build(cls) -> "HolodeckStrategy":
        return cls.builder()
            .add_stage(HolodeckArchitectureStage())      # LLM → rooms, walls, doors, windows
            .add_stage(HolodeckObjectSelectionStage())   # Semantic retrieval
            .add_stage(HolodeckFloorStage())             # Generate & solve floor object constraints
            .add_stage(HolodeckWallStage())              # Generate & solve wall object constraints
            .add_stage(HolodeckSmallStage())             # Generate & solve small object constraints
            .build()
```

**Status**: ✅ Complete and validated

---

## Phase 1: DAG-Based Pipelines (v0.2)

**Target**: Q1 2026

### Goals

Enable non-linear execution patterns:
- Non-linear & agentic strategies
- Parallel stage execution
- Conditional branching
- Iterative refinement loops
- Complex dependencies between stages

### Architecture Design

```python
class DAGStrategy(SceneGenerationStrategy):
    """DAG-based strategy with flexible execution graph"""
    
    def __init__(
        self,
        nodes: Dict[str, Stage],
        edges: List[Tuple[str, str]],
        execution_policy: ExecutionPolicy = "topological"
    ):
        self.graph = build_execution_graph(nodes, edges)
        self.policy = execution_policy
    
    async def generate(self, prompt: str, **kwargs) -> SceneState:
        # Execute stages according to DAG topology
        return await self._execute_dag(prompt, **kwargs)
```

### Use Cases Enabled

#### 1. Parallel Exploration

Try multiple approaches simultaneously, pick the best result:

```python
strategy = DAGStrategy.builder()
    .add_node("arch", ArchitectureStage())
    .add_node("placement_learned", LearnedPlacement())
    .add_node("placement_solver", ConstraintSolver())
    .add_node("eval_learned", EvaluationStage())
    .add_node("eval_solver", EvaluationStage())
    .add_node("select_best", SelectionStage())
    .add_edges([
        ("arch", "placement_learned"),
        ("arch", "placement_solver"),
        ("placement_learned", "eval_learned"),
        ("placement_solver", "eval_solver"),
        ("eval_learned", "select_best"),
        ("eval_solver", "select_best"),
    ])
    .build()
```

#### 2. Iterative Refinement

Loop until convergence or constraints satisfied:

```python
strategy = DAGStrategy.builder()
    .add_node("initial_place", InitialPlacement())
    .add_node("check_collisions", CollisionDetector())
    .add_node("refine", RefinementStage())
    .add_conditional_edge(
        "check_collisions",
        condition=lambda state: state.has_collisions,
        true_target="refine",
        false_target="output"
    )
    .add_edge("refine", "check_collisions")  # Loop
    .build()
```

#### 3. Ensemble Methods

Combine multiple strategies:

```python
strategy = DAGStrategy.builder()
    .add_node("arch", ArchitectureStage())
    .add_node("method_a", MethodAObjectSelection())
    .add_node("method_b", MethodBObjectSelection())
    .add_node("method_c", MethodCObjectSelection())
    .add_node("ensemble", VotingEnsemble())
    .add_edges([
        ("arch", "method_a"),
        ("arch", "method_b"),
        ("arch", "method_c"),
        ("method_a", "ensemble"),
        ("method_b", "ensemble"),
        ("method_c", "ensemble"),
    ])
    .build()
```

---

## Phase 2: End-to-End Learned Methods (v0.3)

**Target**: Q1 2026

### Goals

Support methods where generation is a single forward pass through a neural network, with no explicit stages.

### Architecture Design

The `SceneGenerationStrategy` interface already supports this - learned methods just implement `generate()` differently:

```python
class LearnedStrategy(SceneGenerationStrategy):
    """Single forward-pass generation (e.g. diffusion models)"""
    
    def __init__(self, model: nn.Module, tokenizer: Tokenizer):
        self.model = model
        self.tokenizer = tokenizer
    
    async def generate(self, prompt: str, **kwargs) -> SceneState:
        # Encode prompt
        tokens = self.tokenizer.encode(prompt)
        
        # Single forward pass
        scene_params = await self._denoise(tokens)
        
        # Decode to standard format
        return self._decode_to_scene_state(scene_params)
```

### Use Cases Enabled

#### 1. Scene Diffusion Models

```python
class DiffuSceneStrategy(LearnedStrategy):
    """DiffuScene: learned object placement via diffusion"""
    
    def __init__(self, checkpoint_path: str):
        model = DiffusionModel.from_pretrained(checkpoint_path)
        super().__init__(model, tokenizer=CLIPTokenizer())
    
    async def _denoise(self, tokens):
        # Diffusion denoising process
        latent = self.model.encode(tokens)
        for t in reversed(range(self.num_steps)):
            latent = self.model.denoise_step(latent, t)
        return self.model.decode(latent)
```

#### 2. Transformer-Based Generation

```python
class TransformerStrategy(LearnedStrategy):
    """Autoregressive scene generation"""
    
    async def generate(self, prompt: str, **kwargs) -> SceneState:
        tokens = self.tokenizer.encode(prompt)
        
        # Autoregressive generation
        scene_tokens = []
        for _ in range(max_objects):
            next_token = self.model.predict_next(tokens + scene_tokens)
            if next_token == EOS:
                break
            scene_tokens.append(next_token)
        
        return self._tokens_to_scene_state(scene_tokens)
```

#### 3. Hybrid Linear-Learned

Combine explicit stages with learned components:

```python
class HybridStrategy(DAGStrategy):
    """Learned architecture + linear placement"""
    
    @classmethod
    def build(cls):
        return cls.builder()
            .add_node("arch", LearnedArchitectureModel())      # Neural network
            .add_node("objects", SemanticRetrieval())           # Traditional
            .add_node("place", LearnedPlacement())              # Neural network
            .add_edges([
                ("arch", "objects"),
                ("objects", "place")
            ])
            .build()
```

## Phase 3: Agentic Methods (v0.4)

**Target**: Q1 2026

### Goals

Support LLM-directed strategies where the execution plan is determined dynamically based on reasoning about the task.

### Architecture Design

```python
class AgenticStrategy(SceneGenerationStrategy):
    """LLM-directed tool use for scene generation"""
    
    def __init__(
        self,
        agent: LLMPlanner,
        tools: Dict[str, List[Stage]],
        max_iterations: int = 10
    ):
        self.agent = agent
        self.tools = tools
        self.max_iterations = max_iterations
    
    async def generate(self, prompt: str, **kwargs) -> SceneState:
        state = SceneState(prompt=prompt)
        
        for iteration in range(self.max_iterations):
            # Agent decides next action
            action = await self.agent.plan(state, self.tools)
            
            if action.type == "complete":
                return state
            
            # Execute selected tool
            tool = self.tools[action.category][action.tool_index]
            state = await tool.process(state)
            
            # Agent reflects on result
            await self.agent.observe(state)
        
        return state
```

### Use Cases Enabled

#### 1. Task-Specific Planning

Agent chooses different tools based on prompt:

```python
agent = LLMPlanner(
    model="claude-4-sonnet",
    system_prompt="""
    You are a scene generation planner. 
    Analyze the prompt and select appropriate tools.
    
    For simple rooms: use rule-based architecture + constraint solver
    For complex scenes: use learned architecture + agentic placement
    For outdoor scenes: use terrain generation + physics-based placement
    """
)

strategy = AgenticStrategy(
    agent=agent,
    tools={
        "architecture": [RuleBased(), Learned(), Terrain()],
        "placement": [ConstraintSolver(), AgenticPlacer(), PhysicsSim()],
        "refinement": [Aesthetic(), Collision(), Semantic()]
    }
)
```

#### 2. Error Recovery

Agent detects issues and replan:

```python
# Agent reasoning loop:
# 1. Initial placement
# 2. Check for collisions
# 3. If collisions: analyze cause
# 4. If too dense: remove objects
# 5. If poor layout: try different architecture
# 6. If still failing: use simpler solver
```

#### 3. Multi-Modal Reasoning

Agent considers multiple aspects:

```python
class MultiModalAgent(LLMPlanner):
    async def plan(self, state: SceneState, tools: Dict) -> Action:
        # Consider:
        # - Semantic coherence (living room needs sofa)
        # - Spatial constraints (furniture must fit)
        # - Aesthetic principles (balanced composition)
        # - User preferences (from prompt)
        # - Current state quality
        
        analysis = await self._analyze_state(state)
        return self._select_tool(analysis, tools)
```

---

## Phase 4: Unified Framework (v1.0)

**Target**: Q2 2026

### Goal

Seamlessly support all paradigms (linear, DAG, learned, agentic) with full interoperability.

### Key Features

**1. Paradigm Mixing**
```python
# DAG with learned and agentic components
strategy = DAGStrategy.builder()
    .add_node("arch", LearnedArchitectureModel())           # Learned
    .add_node("objects", AgenticObjectSelection())          # Agentic
    .add_node("place", ConstraintSolver())                  # Linear
    .add_node("refine", LearnedRefinement())                # Learned
    .build()
```

**2. Universal Evaluation**
```python
# Compare across all paradigms
benchmark.evaluate([
    LinearStrategy(...),      # Linear
    DAGStrategy(...),         # Complex pipeline
    LearnedStrategy(...),     # End-to-end
    AgenticStrategy(...),     # LLM-directed
    HybridStrategy(...)       # Mixed paradigm
])
```

**3. Component Library**
- 5+ complete strategies
- Multiple evaluation metrics
- Benchmark datasets

<!-- **4. Production Ready**
- Monitoring and logging
- Error handling and recovery
- Performance optimization
- Deployment guides -->

### Success Criteria

- ✅ 5+ research papers use sisglib
- ✅ 3+ research labs contribute implementations
- ✅ Community benchmark adopted
<!-- - ✅ Production deployments documented -->
- ✅ 200+ GitHub stars

---

## Beyond v1.0

### Potential Extensions

**Multi-Modal Input**
- Image-based scene generation
- Sketch-to-scene
- Voice prompts

**Domain Specialization**
- Architecture-specific pipelines
- Game level generation
- Robotic environment synthesis

<!-- ---

## Contributing to the Roadmap

This roadmap evolves based on community needs. We welcome:

- **Feature requests**: What paradigms are we missing?
- **Use case studies**: How would you use DAG/learned/agentic strategies?
- **Implementation contributions**: Help build the future versions
- **Feedback**: What would make sisglib more useful for your research?

**Join the discussion**: [GitHub Discussions](https://github.com/3D-intelligence/sisglib/discussions) -->