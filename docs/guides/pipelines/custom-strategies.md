# Custom Strategies Development Guide

## Overview

This guide walks you through developing custom scene generation strategies in sisglib. Whether you're implementing a novel research method or adapting an existing approach, sisglib's modular architecture makes it straightforward to create, test, and benchmark your algorithms.

## Strategy Basics

A **strategy** in sisglib defines how to generate a 3D scene from a text prompt. Strategies are composed of **stages** that transform a `SceneState` object sequentially.

```
Prompt → [Stage 1] → [Stage 2] → [Stage 3] → ... → Scene
          ↓           ↓           ↓
       SceneState  SceneState  SceneState
```

## Quick Start: Fluent Builder Pattern

The fastest way to create a custom strategy is using the fluent builder API:

```python
from sisglib.pipeline.generation import SceneGenerationStrategy, SceneGenerationPipeline
from sisglib.pipeline.generation.strategies.holodeck import (
    HolodeckArchitectureStage,
    HolodeckObjectSelectionStage
)

# Mix your custom stages with existing ones
strategy = (
    SceneGenerationStrategy.builder()
    .with_name("My Hybrid Strategy")
    .add_stage(HolodeckArchitectureStage())           # Reuse Holodeck's architecture
    .add_stage(MyCustomObjectSelectionStage())        # Your custom selection logic
    .add_stage(MyLearnedPlacementStage())             # Your ML-based placement
    .build()
)

# Use it immediately
pipeline = SceneGenerationPipeline(strategy=strategy)
scene = await pipeline.generate(prompt="A modern living room")
```

## Creating Custom Stages

Stages are the building blocks of strategies. Here's how to create one:

```python
from sisglib.pipeline.generation.core import PipelineStage
from sissf import SceneState

class MyCustomObjectSelectionStage(PipelineStage):
    """Select 3D objects based on custom logic."""

    def __init__(self, vector_adapter, metadata_adapter):
        super().__init__()
        self.vectors = vector_adapter
        self.metadata = metadata_adapter

    async def execute(self, state: SceneState, prompt: str) -> SceneState:
        """
        Transform the scene state based on your logic.

        Args:
            state: Current scene state (may be empty or partially filled)
            prompt: Original user prompt

        Returns:
            Modified scene state
        """
        # Your custom logic here
        # Example: Semantic search for objects mentioned in prompt

        # 1. Extract object categories from prompt (your logic)
        categories = self.extract_categories(prompt)

        # 2. Search for relevant assets
        for category in categories:
            results = await self.vectors.search(
                query_vector=self.encode_text(category),
                k=5,
                metadata_filter={"category": category}
            )

            # 3. Add objects to scene state
            for result in results:
                obj_metadata = await self.metadata.get(result.id)
                state.add_object(
                    id=result.id,
                    category=category,
                    metadata=obj_metadata
                )

        return state

    def extract_categories(self, prompt: str) -> list[str]:
        # Your implementation
        pass

    def encode_text(self, text: str):
        # Your implementation
        pass
```

## Production Pattern: Strategy Subclass

For production-grade implementations, subclass `SceneGenerationStrategy`:

```python
from sisglib.pipeline.generation import SceneGenerationStrategy
from typing import List

class MyResearchStrategy(SceneGenerationStrategy):
    """
    A custom strategy implementing [your method name].

    This strategy uses:
    - [Stage 1 description]
    - [Stage 2 description]
    - [Stage 3 description]
    """

    def __init__(
        self,
        vector_adapter,
        metadata_adapter,
        llm_config: dict,
        **kwargs
    ):
        # Define your stages
        stages = [
            ArchitectureGenerationStage(llm_config),
            MyCustomObjectSelectionStage(vector_adapter, metadata_adapter),
            MyLearnedPlacementStage(),
            RefinementStage(),
        ]

        super().__init__(
            name="My Research Strategy",
            stages=stages,
            **kwargs
        )

        self.vector_adapter = vector_adapter
        self.metadata_adapter = metadata_adapter

    @classmethod
    def build(
        cls,
        vector_adapter=None,
        metadata_adapter=None,
        llm_config=None,
        **kwargs
    ):
        """
        Factory method for easy configuration.

        This is the recommended way to instantiate your strategy,
        allowing for default values and validation.
        """
        # Set defaults
        if vector_adapter is None:
            from sisglib.adapters.vectors import NumpyAdapter
            vector_adapter = NumpyAdapter(dimension=768)

        if metadata_adapter is None:
            from sisglib.adapters.metadata import JSONLocalAdapter
            metadata_adapter = JSONLocalAdapter()

        if llm_config is None:
            llm_config = {"model": "gpt-4", "temperature": 0.7}

        return cls(
            vector_adapter=vector_adapter,
            metadata_adapter=metadata_adapter,
            llm_config=llm_config,
            **kwargs
        )
```

### Using Your Strategy

```python
# Simple instantiation
strategy = MyResearchStrategy.build()

# Custom configuration
strategy = MyResearchStrategy.build(
    vector_adapter=my_custom_vector_adapter,
    llm_config={"model": "gpt-4-turbo", "temperature": 0.5}
)

# Run it
pipeline = SceneGenerationPipeline(strategy=strategy)
scene = await pipeline.generate(prompt="A cozy bedroom")
```

## Common Stage Types

Note: These stages are for demonstration purposes.

### 1. Architecture Generation Stage

Generates the room layout, walls, floor, ceiling:

```python
class ArchitectureGenerationStage(PipelineStage):
    async def execute(self, state: SceneState, prompt: str) -> SceneState:
        # Generate room dimensions from prompt
        dimensions = self.llm_generate_dimensions(prompt)

        # Add to scene state
        self.set_room_dimensions(state, dimensions)

        # Generate walls, floor, ceiling
        ...

        self.add_architecture_elements(state, walls, floor, ceiling)

        return state
```

### 2. Object Selection Stage

Chooses which 3D assets to include:

```python
class ObjectSelectionStage(PipelineStage):
    async def execute(self, state: SceneState, prompt: str) -> SceneState:
        # Semantic search for relevant objects
        objects = await self.find_relevant_objects(prompt)

        for obj in objects:
            state.add_object(obj)

        return state
```

### 3. Spatial Placement Stage

Determines 3D positions and orientations:

```python
class SpatialPlacementStage(PipelineStage):
    async def execute(self, state: SceneState, prompt: str) -> SceneState:
        # Get unplaced objects
        objects = state.get_objects(placed=False)

        for obj in objects:
            # Your placement logic (constraint-based, learned, etc.)
            position = self.compute_position(obj, state)
            rotation = self.compute_rotation(obj, state)

            self.set_object_transform(
                state,
                obj.id,
                position=position,
                rotation=rotation
            )

        return state
```

### 4. Refinement Stage

Post-processes the scene (collision resolution, aesthetic adjustments):

```python
class RefinementStage(PipelineStage):
    async def execute(self, state: SceneState, prompt: str) -> SceneState:
        # Resolve collisions
        state = self.resolve_collisions(state)

        # Adjust for aesthetics
        state = self.improve_layout(state)

        return state
```

## Interoperability via sissf

All strategies operate on the `SceneState` format from [sissf](https://github.com/3D-Intelligence/sissf/wiki), which makes the following possible:

### Mix Strategies

```python
# Start with Holodeck's architecture, then use your custom placement
strategy = (
    SceneGenerationStrategy.builder()
    .add_stage(HolodeckArchitectureStage())
    .add_stage(HolodeckObjectSelectionStage())
    .add_stage(MyLearnedPlacementStage())  # Your ML model
    .build()
)
```

### Chain Strategies

```python
# Generate base scene with one strategy
base_scene = await pipeline1.generate(prompt)

# Refine with another strategy
refined_scene = await pipeline2.generate(
    prompt=prompt,
    initial_state=base_scene  # Start from existing scene
)
```

### Share Intermediate Results

```python
# Save intermediate state
with open ("intermediate.json", "w") as f:
    f.write(scene_state.to_dict())

# Load in different pipeline
with open ("intermediate.json", "r") as f:
    state_dict = json.load(f)

loaded_state = SceneState.from_dict(state_dict)
final_scene = await other_pipeline.generate(
    prompt=prompt,
    initial_state=loaded_state
)
```

## Benchmarking Strategies

Compare your strategy against baselines:

```python
from sisglib.pipeline.generation import HolodeckStrategy, SceneGenerationPipeline

# Your strategy
your_strategy = MyResearchStrategy.build()

# Baseline
holodeck_strategy = HolodeckStrategy.build()

# Benchmark prompts
prompts = [
    "A modern living room with a sofa and TV",
    "A cozy bedroom with a bed and nightstand",
    "An office with a desk and chair",
]

results = {}
for name, strategy in [("Yours", your_strategy), ("Holodeck", holodeck_strategy)]:
    pipeline = SceneGenerationPipeline(strategy=strategy)
    results[name] = []

    for prompt in prompts:
        scene = await pipeline.generate(prompt=prompt)
        metrics = evaluate_scene(scene, prompt)  # Your evaluation function
        results[name].append(metrics)

# Compare results
print_comparison(results)
```
