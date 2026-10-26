# Awarri AI Model requirements

1. Model
We will be deploying separate STT and TTS models.

STT: Custom-built by Awarri and licensed under Awarri’s license.
TTS: Diffusion-based fine-tuned model based on the OmniVoice architecture, exported into a custom implementation.
2. Artifacts

STT: Model format is ONNX. Container images have already been built and are available within the MTN environment.
TTS: Container images have also already been built and are available within the MTN environment.
3. Serving
STT:

Runtime: Custom
API: REST
Streaming: WebSockets or TCP
Batch inference: HTTP
TTS:

Runtime: vLLM-Omni
API: REST
Streaming: WebSockets
4. Compute
GPU compute is required. Our current requirement is:

2 × NVIDIA A100 GPUs
40 GB VRAM per GPU
5. Performance

STT:

Expected concurrency: approximately 175 concurrent users per 40 GB A100
First partial latency: <500 ms at p95
Drain latency: <800 ms at p95
TTS:

Expected concurrency: approximately 50 concurrent calls per 40 GB A100
Request rate: approximately 5 requests/second
Latency target: ≤1.0 second at p95
6. Availability
The required availability and scaling configuration can be aligned with MTN’s expected usage patterns and infrastructure requirements. We would be happy to discuss the appropriate replica and scaling strategy during the call.

7. Data and Security

We do not retain any user data through the models.
There are currently no specific requirements around region or data residency from our side.
The models were purpose-built for MTN, so the authentication layer can be determined by MTN based on its existing security and access-control requirements.
Internet access is required for the models to be used.
8. Quality / Supported Languages
Both the STT and TTS models support the following languages and accent variants:

Currently supported:

Nigerian-accented English
Yoruba
Igbo
Hausa
Nigerian Pidgin English
African-accented French
In production/deployment:

Ghanaian-accented English
Twi
