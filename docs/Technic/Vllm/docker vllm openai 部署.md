---
date created: 2025/5/8 11:40
date modified: 2025/6/6 17:1
---

[vLLM - vLLM](https://docs.vllm.ai/en/latest/)

AskAI

以下是 `vllm serve` 命令的中文参数说明：

### 基本模型参数

- **--model**：指定要加载的模型。
	
- **--task**：设置模型任务类型，可选值包括 `auto`、`classify`、`draft`、`embed`、`embedding`、`generate`、`reward`、`score`、`transcription`。
	
- **--tokenizer**：指定使用的分词器。
	
- **--tokenizer-mode**：设置分词器模式，可选值包括 `auto`、`custom`、`mistral`、`slow`。
	
- **--trust-remote-code / --no-trust-remote-code**：是否信任远程代码。
	
- **--dtype**：设置模型参数的数据类型，可选值包括 `auto`、`bfloat16`、`float`、`float16`、`float32`、`half`。
	
- **--seed**：随机种子。
	
- **--hf-config-path**：Hugging Face 配置文件路径。
	
- **--allowed-local-media-path**：允许访问的本地媒体路径。
	
- **--revision**：模型的代码版本。
	
- **--code-revision**：代码库的特定版本。
	
- **--rope-scaling**、**--rope-theta**：设置位置编码 (RoPE) 的缩放和角度参数。
	
- **--max-model-len**：设置模型的最大输入长度。
	
- **--quantization**：模型量化选项，例如 `gptq`、`bitsandbytes`、`fbgemm_fp8` 等。
	
- **--served-model-name**：指定服务的模型名称。
	
- **--config-format**：模型配置格式，例如 `auto`、`hf`、`mistral`。
	
- **--hf-token**：Hugging Face 访问令牌。
	
- **--hf-overrides**：自定义 Hugging Face 参数。
	
- **--override-neuron-config**、**--override-pooler-config**：覆盖神经网络或池化层的配置。
	
- **--model-impl**：模型实现类型，例如 `auto`、`vllm`、`transformers`。
	
- **--load-format**：模型加载格式，例如 `auto`、`pt`、`safetensors`、`npcache`、`dummy` 等。
	
- **--download-dir**：模型下载路径。

### 推理与生成设置

- **--logits-processor-pattern**：设置 logits 处理器的模式。
	
- **--generation-config**、**--override-generation-config**：配置文本生成参数。
	
- **--enable-reasoning / --no-enable-reasoning**：启用或禁用推理功能。
	
- **--reasoning-parser**：推理解析器，例如 `deepseek_r1`、`granite`、`qwen3`。
	
- **--enable-prompt-embeds / --no-enable-prompt-embeds**：启用或禁用提示嵌入。

### 并行与分布式设置

- **--distributed-executor-backend**：分布式执行后端，例如 `external_launcher`、`mp`、`ray`、`uni`。
	
- **--pipeline-parallel-size**：设置流水线并行度。
	
- **--tensor-parallel-size**、**--data-parallel-size**：张量和数据并行度。
	
- **--enable-expert-parallel / --no-enable-expert-parallel**：是否启用专家并行。
	
- **--max-parallel-loading-workers**：最大并行加载线程数。
	
- **--worker-cls**、**--worker-extension-cls**：自定义 worker 类。
	
- **--block-size**：模型块大小，可选 `1`、`8`、`16`、`32`、`64`、`128`。

### 性能优化与缓存

- **--cpu-offload-gb**：设置 CPU 缓存的最大占用空间。
	
- **--kv-cache-dtype**：KV 缓存的数据类型，例如 `auto`、`fp8`、`fp8_e4m3`、`fp8_e5m2`。
	
- **--enable-prefix-caching / --no-enable-prefix-caching**：启用或禁用前缀缓存。
	
- **--long-prefill-token-threshold**：设置长输入的 token 阈值。
	
- **--calculate-kv-scales / --no-calculate-kv-scales**：是否计算 KV 缩放。

### LoRA 设置

- **--enable-lora / --no-enable-lora**：启用或禁用 LoRA（低秩自适应）。
	
- **--max-loras**、**--max-lora-rank**：设置最大 LoRA 数量和秩。
	
- **--lora-extra-vocab-size**：LoRA 额外词表大小。
	
- **--long-lora-scaling-factors**：设置长 LoRA 的缩放因子。

### 其他配置

- **--device**：设备类型，例如 `auto`、`cpu`、`cuda`、`hpu`、`neuron`、`tpu`、`xpu`。
	
- **--speculative-config**：推测执行配置。
	
- **--use-tqdm-on-load / --no-use-tqdm-on-load**：加载时是否显示进度条。
	
- **--scheduling-policy**：调度策略，例如 `fcfs`（先到先服务）、`priority`（优先级）。
	
- **--disable-log-stats**：禁用日志统计。

---

```python
usage: vllm serve [-h] [--model MODEL]
                  [--task {auto,classify,draft,embed,embedding,generate,reward,score,transcription}]
                  [--tokenizer TOKENIZER]
                  [--tokenizer-mode {auto,custom,mistral,slow}]
                  [--trust-remote-code | --no-trust-remote-code]
                  [--dtype {auto,bfloat16,float,float16,float32,half}]
                  [--seed SEED] [--hf-config-path HF_CONFIG_PATH]
                  [--allowed-local-media-path ALLOWED_LOCAL_MEDIA_PATH]
                  [--revision REVISION] [--code-revision CODE_REVISION]
                  [--rope-scaling ROPE_SCALING] [--rope-theta ROPE_THETA]
                  [--tokenizer-revision TOKENIZER_REVISION]
                  [--max-model-len MAX_MODEL_LEN]
                  [--quantization {aqlm,awq,awq_marlin,bitblas,bitsandbytes,compressed-tensors,deepspeedfp,experts_int8,fbgemm_fp8,fp8,gguf,gptq,gptq_bitblas,gptq_marlin,gptq_marlin_24,hqq,ipex,marlin,modelopt,moe_wna16,neuron_quant,nvfp4,ptpc_fp8,qqq,quark,torchao,tpu_int8,None}]
                  [--enforce-eager | --no-enforce-eager]
                  [--max-seq-len-to-capture MAX_SEQ_LEN_TO_CAPTURE]
                  [--max-logprobs MAX_LOGPROBS]
                  [--disable-sliding-window | --no-disable-sliding-window]
                  [--disable-cascade-attn | --no-disable-cascade-attn]
                  [--skip-tokenizer-init | --no-skip-tokenizer-init]
                  [--enable-prompt-embeds | --no-enable-prompt-embeds]
                  [--served-model-name SERVED_MODEL_NAME [SERVED_MODEL_NAME ...]]
                  [--disable-async-output-proc]
                  [--config-format {auto,hf,mistral}] [--hf-token [HF_TOKEN]]
                  [--hf-overrides HF_OVERRIDES]
                  [--override-neuron-config OVERRIDE_NEURON_CONFIG]
                  [--override-pooler-config OVERRIDE_POOLER_CONFIG]
                  [--logits-processor-pattern LOGITS_PROCESSOR_PATTERN]
                  [--generation-config GENERATION_CONFIG]
                  [--override-generation-config OVERRIDE_GENERATION_CONFIG]
                  [--enable-sleep-mode | --no-enable-sleep-mode]
                  [--model-impl {auto,vllm,transformers}]
                  [--load-format {auto,pt,safetensors,npcache,dummy,tensorizer,sharded_state,gguf,bitsandbytes,mistral,runai_streamer,runai_streamer_sharded,fastsafetensors}]
                  [--download-dir DOWNLOAD_DIR]
                  [--model-loader-extra-config MODEL_LOADER_EXTRA_CONFIG]
                  [--ignore-patterns IGNORE_PATTERNS [IGNORE_PATTERNS ...]]
                  [--use-tqdm-on-load | --no-use-tqdm-on-load]
                  [--qlora-adapter-name-or-path QLORA_ADAPTER_NAME_OR_PATH]
                  [--pt-load-map-location PT_LOAD_MAP_LOCATION]
                  [--guided-decoding-backend {auto,guidance,lm-format-enforcer,outlines,xgrammar}]
                  [--guided-decoding-disable-fallback | --no-guided-decoding-disable-fallback]
                  [--guided-decoding-disable-any-whitespace | --no-guided-decoding-disable-any-whitespace]
                  [--guided-decoding-disable-additional-properties | --no-guided-decoding-disable-additional-properties]
                  [--enable-reasoning | --no-enable-reasoning]
                  [--reasoning-parser {deepseek_r1,granite,qwen3}]
                  [--distributed-executor-backend {external_launcher,mp,ray,uni,None}]
                  [--pipeline-parallel-size PIPELINE_PARALLEL_SIZE]
                  [--tensor-parallel-size TENSOR_PARALLEL_SIZE]
                  [--data-parallel-size DATA_PARALLEL_SIZE]
                  [--enable-expert-parallel | --no-enable-expert-parallel]
                  [--max-parallel-loading-workers MAX_PARALLEL_LOADING_WORKERS]
                  [--ray-workers-use-nsight | --no-ray-workers-use-nsight]
                  [--disable-custom-all-reduce | --no-disable-custom-all-reduce]
                  [--worker-cls WORKER_CLS]
                  [--worker-extension-cls WORKER_EXTENSION_CLS]
                  [--block-size {1,8,16,32,64,128}]
                  [--gpu-memory-utilization GPU_MEMORY_UTILIZATION]
                  [--swap-space SWAP_SPACE]
                  [--kv-cache-dtype {auto,fp8,fp8_e4m3,fp8_e5m2}]
                  [--num-gpu-blocks-override NUM_GPU_BLOCKS_OVERRIDE]
                  [--enable-prefix-caching | --no-enable-prefix-caching]
                  [--prefix-caching-hash-algo {builtin,sha256}]
                  [--cpu-offload-gb CPU_OFFLOAD_GB]
                  [--calculate-kv-scales | --no-calculate-kv-scales]
                  [--tokenizer-pool-size TOKENIZER_POOL_SIZE]
                  [--tokenizer-pool-type TOKENIZER_POOL_TYPE]
                  [--tokenizer-pool-extra-config TOKENIZER_POOL_EXTRA_CONFIG]
                  [--limit-mm-per-prompt LIMIT_MM_PER_PROMPT]
                  [--mm-processor-kwargs MM_PROCESSOR_KWARGS]
                  [--disable-mm-preprocessor-cache | --no-disable-mm-preprocessor-cache]
                  [--enable-lora | --no-enable-lora]
                  [--enable-lora-bias | --no-enable-lora-bias]
                  [--max-loras MAX_LORAS] [--max-lora-rank MAX_LORA_RANK]
                  [--lora-extra-vocab-size LORA_EXTRA_VOCAB_SIZE]
                  [--lora-dtype {auto,bfloat16,float16}]
                  [--long-lora-scaling-factors LONG_LORA_SCALING_FACTORS [LONG_LORA_SCALING_FACTORS ...]]
                  [--max-cpu-loras MAX_CPU_LORAS]
                  [--fully-sharded-loras | --no-fully-sharded-loras]
                  [--enable-prompt-adapter | --no-enable-prompt-adapter]
                  [--max-prompt-adapters MAX_PROMPT_ADAPTERS]
                  [--max-prompt-adapter-token MAX_PROMPT_ADAPTER_TOKEN]
                  [--device {auto,cpu,cuda,hpu,neuron,tpu,xpu}]
                  [--speculative-config SPECULATIVE_CONFIG]
                  [--show-hidden-metrics-for-version SHOW_HIDDEN_METRICS_FOR_VERSION]
                  [--otlp-traces-endpoint OTLP_TRACES_ENDPOINT]
                  [--collect-detailed-traces {all,model,worker,None} [{all,model,worker,None} ...]]
                  [--max-num-batched-tokens MAX_NUM_BATCHED_TOKENS]
                  [--max-num-seqs MAX_NUM_SEQS]
                  [--max-num-partial-prefills MAX_NUM_PARTIAL_PREFILLS]
                  [--max-long-partial-prefills MAX_LONG_PARTIAL_PREFILLS]
                  [--cuda-graph-sizes CUDA_GRAPH_SIZES [CUDA_GRAPH_SIZES ...]]
                  [--long-prefill-token-threshold LONG_PREFILL_TOKEN_THRESHOLD]
                  [--num-lookahead-slots NUM_LOOKAHEAD_SLOTS]
                  [--scheduler-delay-factor SCHEDULER_DELAY_FACTOR]
                  [--preemption-mode {recompute,swap,None}]
                  [--num-scheduler-steps NUM_SCHEDULER_STEPS]
                  [--multi-step-stream-outputs | --no-multi-step-stream-outputs]
                  [--scheduling-policy {fcfs,priority}]
                  [--enable-chunked-prefill | --no-enable-chunked-prefill]
                  [--disable-chunked-mm-input | --no-disable-chunked-mm-input]
                  [--scheduler-cls SCHEDULER_CLS]
                  [--compilation-config COMPILATION_CONFIG]
                  [--kv-transfer-config KV_TRANSFER_CONFIG]
                  [--kv-events-config KV_EVENTS_CONFIG]
                  [--additional-config ADDITIONAL_CONFIG]
                  [--use-v2-block-manager] [--disable-log-stats]
```