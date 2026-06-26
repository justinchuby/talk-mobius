Hi, I am Justin, a software engineer at Microsoft working our inference and optimization stack for edge AI.

At Microsoft, we use ONNX as both the starting point for optimizing AI models, and as the format we deliver those optimized models in.

To acquire the ONNX models, we have been relying on the PyTorch exporter to trace the model architecture and convert the operators into ONNX representations. And with these AI models, typically from Hugging Face.

Now, to convert these models into ONNX using the Torch exporter, conversion is still hard for a few reasons:

- **Dynamic shapes** - models specializes shape information in runtime (breaks export)
- **Unsupported operators** - control flow, Python-only logic, new PyTorch ops
- **Graph clean up and node fusion** - patterns change with `transformers` and PyTorch updates / varies among models
- **Model signature / architecture changes** Some models require input/output/tensor layout changes to fit onnx conventions
- **Quantized models** Low-bit representation doesn't have a standard tracible representation

# The Deeper Problem: Fragmentation

<v-clicks>

- **No single source of truth** — same model, different ONNX graphs depending on who exported it
- **Structural inconsistency** — CUDA export ≠ WebGPU export ≠ CPU export
- **Quality variance** — some exports work, some silently produce wrong outputs
- **Duplicated effort** — every team re-discovers the same export pitfalls

<div v-click class="mt-8 p-4 bg-blue-50 rounded-lg text-center text-xl">
💡 We need <strong>one canonical construction</strong> per architecture.
</div>

All of these leads to longer development time and and we miss performance optimization opportunities by having models with patterns the optimizers do not know.

This is why we need a way to ensure canonical constructions and avoid fragmentation.

---

# The Paradigm: Construction

<div class="grid grid-cols-2 gap-8 mt-8">

<div class="border-2 border-gray-300 rounded-lg p-4">

### Translation (Export)

```
PyTorch Model
    ↓ trace
Intermediate Repr
    ↓ convert ops
ONNX Graph
    ↓ cleanup / fusion / graph transform
ONNX Model
```

Great for general purpose.
Model code dependent. Hard to standardize.

</div>

<div class="border-2 border-green-300 rounded-lg p-4">

### ✅ Construction

```
HuggingFace Config
    ↓ read architecture
ONNX Graph (declarative)
    ↓ apply weights
ONNX Model
```

Deterministic. Composable.
**Canonical across architectures.**

</div>

</div>

<!--
- Whereas we translated the models and were constrained by modeling code and the PyTorch framework, we started to explore directly constructing these models based on what we know about the model architecture and what we expected as the eventual clean ONNX representation.
- This is the approach pioneered by the Model Builder in ONNX Runtime GenAI
-->


---


<ScaleAnimation />

With Model Builder, we are able to build ~20 text to text architectures. But the HuggingFace ecosystem has over 300 across every modality.

How do we scale construction to the entire ecosystem?

# The Answer

<div class="text-2xl text-gray-500 mt-4">
Design construction for AI — from the ground up.
</div>


<!--

Our answer: scale using AI agents. And design the system for agentic development from day one.

To open the top of the funnel for the ONNX ecosystem, and to make device-targeted optimization easier, we need a way to scalably bring models into ONNX — and provide a stable, uniform representation as the starting point, regardless of model architecture.

That's what Mobius does.
-->
