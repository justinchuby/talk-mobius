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
