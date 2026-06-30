<!-- source: website/docs/reference/optional-habilidads-catalog.md -->
# Optional Habilidads Catalog

# Optional Habilidads Catalog

Optional habilidads ship with hermes-agente under `optional-habilidads/` but are **not active by default**. Install them explicitly:

```bash
hermes habilidads install official/<category>/<habilidad>
```

For example:

```bash
hermes habilidads install official/blockchain/solana
hermes habilidads install official/mlops/flash-attention
```

Each habilidad below links to a dedicated page with its full definition, setup, and usage.

To uninstall:

```bash
hermes habilidads uninstall <habilidad-name>
```

## autonomous-ai-agentes

| Habilidad | Description |
|-------|-------------|
| [**antigravity-cli**](/docs/guia-usuario/habilidads/optional/autonomous-ai-agentes/autonomous-ai-agentes-antigravity-cli) | Operate the Antigravity CLI (agy): complementos, auth, sandbox. |
| [**blackbox**](/docs/guia-usuario/habilidads/optional/autonomous-ai-agentes/autonomous-ai-agentes-blackbox) | Delegate coding tasks to Blackbox AI CLI agente. Multi-modelo agente with built-in judge that runs tasks through multiple LLMs and picks the best result. Requires the blackbox CLI and a Blackbox AI API key. |
| [**grok**](/docs/guia-usuario/habilidads/optional/autonomous-ai-agentes/autonomous-ai-agentes-grok) | Delegate coding to xAI Grok Build CLI (features, PRs). |
| [**honcho**](/docs/guia-usuario/habilidads/optional/autonomous-ai-agentes/autonomous-ai-agentes-honcho) | Configure and use Honcho memoria with Hermes -- cross-session user modeloing, multi-perfil peer isolation, observation config, dialectic reasoning, session summaries, and contexto budget enforcement. Use when setting up Honcho, troubleshoo... |
| [**openhands**](/docs/guia-usuario/habilidads/optional/autonomous-ai-agentes/autonomous-ai-agentes-openhands) | Delegate coding to OpenHands CLI (modelo-agnostic, LiteLLM). |

## blockchain

| Habilidad | Description |
|-------|-------------|
| [**evm**](/docs/guia-usuario/habilidads/optional/blockchain/blockchain-evm) | Read-only EVM client: wallets, tokens, gas across 8 chains. |
| [**hyperliquid**](/docs/guia-usuario/habilidads/optional/blockchain/blockchain-hyperliquid) | Hyperliquid market data, account history, trade review. |
| [**solana**](/docs/guia-usuario/habilidads/optional/blockchain/blockchain-solana) | Query Solana blockchain data with USD pricing — wallet balances, token portfolios with values, transaction details, NFTs, whale detection, and live network stats. Uses Solana RPC + CoinGecko. No API key required. |

## communication

| Habilidad | Description |
|-------|-------------|
| [**one-three-one-rule**](/docs/guia-usuario/habilidads/optional/communication/communication-one-three-one-rule) | Structured decision-making framework for technical proposals and trade-off analysis. When the user faces a choice between multiple approaches (architecture decisions, herramienta selection, refactoring strategies, migration paths), this habilidad p... |

## creative

| Habilidad | Description |
|-------|-------------|
| [**baoyu-article-illustrator**](/docs/guia-usuario/habilidads/optional/creative/creative-baoyu-article-illustrator) | Article illustrations: type × style × palette consistency. |
| [**baoyu-comic**](/docs/guia-usuario/habilidads/optional/creative/creative-baoyu-comic) | Knowledge comics (知识漫画): educational, biography, tutorial. |
| [**blender-mcp**](/docs/guia-usuario/habilidads/optional/creative/creative-blender-mcp) | Control Blender directly from Hermes via socket connection to the blender-mcp addon. Create 3D objects, materials, animations, and run arbitrary Blender Python (bpy) code. Use when user wants to create or modify anything in Blender. |
| [**concept-diagrams**](/docs/guia-usuario/habilidads/optional/creative/creative-concept-diagrams) | Generate flat, minimal light/dark-aware SVG diagrams as standalone HTML files, using a unified educational visual language with 9 semantic color ramps, sentence-case typography, and automatic dark mode. Best suited for educational and no... |
| [**creative-ideation**](/docs/guia-usuario/habilidads/optional/creative/creative-creative-ideation) | Generate ideas via named methods from creative practice. |
| [**hyperframes**](/docs/guia-usuario/habilidads/optional/creative/creative-hyperframes) | Create HTML-based video compositions, animated title cards, social overlays, captioned talking-head videos, audio-reactive visuals, and shader transitions using HyperFrames. HTML is the source of truth for video. Use when the user wants... |
| [**kanban-video-orchestrator**](/docs/guia-usuario/habilidads/optional/creative/creative-kanban-video-orchestrator) | Plan, set up, and monitor a multi-agente video production pipeline backed by Hermes Kanban. Use when the user wants to make ANY video — narrative film, product/marketing, music video, explainer, ASCII/terminal art, abstract/generative loo... |
| [**meme-generation**](/docs/guia-usuario/habilidads/optional/creative/creative-meme-generation) | Generate real meme images by picking a template and overlaying text with Pillow. Produces actual .png meme files. |
| [**pixel-art**](/docs/guia-usuario/habilidads/optional/creative/creative-pixel-art) | Pixel art w/ era palettes (NES, Game Boy, PICO-8). |

## devops

| Habilidad | Description |
|-------|-------------|
| [**inference-sh-cli**](/docs/guia-usuario/habilidads/optional/devops/devops-cli) | Run 150+ AI apps via inference.sh CLI (infsh) — image generation, video creation, LLMs, search, 3D, social automation. Uses the terminal herramienta. Triggers: inference.sh, infsh, ai apps, flux, veo, image generation, video generation, seedrea... |
| [**docker-management**](/docs/guia-usuario/habilidads/optional/devops/devops-docker-management) | Manage Docker containers, images, volumes, networks, and Compose stacks — lifecycle ops, debugging, cleanup, and Dockerfile optimization. |
| [**hermes-s6-container-supervision**](/docs/guia-usuario/habilidads/optional/devops/devops-hermes-s6-container-supervision) | Modify, debug, or extend the s6-overlay supervision tree inside the Hermes Agente Docker image — adding new services, debugging perfil puerta de enlaces, understanding the Architecture B main-program pattern. |
| [**pinggy-tunnel**](/docs/guia-usuario/habilidads/optional/devops/devops-pinggy-tunnel) | Zero-install localhost tunnels over SSH via Pinggy. |
| [**watchers**](/docs/guia-usuario/habilidads/optional/devops/devops-watchers) | Poll RSS, JSON APIs, and GitHub with watermark dedup. |

## dogfood

| Habilidad | Description |
|-------|-------------|
| [**adversarial-ux-test**](/docs/guia-usuario/habilidads/optional/dogfood/dogfood-adversarial-ux-test) | Roleplay the most difficult, tech-resistant user for your product. Browse the app as that persona, find every UX pain point, then filter complaints through a pragmatism layer to separate real problems from noise. Creates actionable ticke... |

## email

| Habilidad | Description |
|-------|-------------|
| [**agentemail**](/docs/guia-usuario/habilidads/optional/email/email-agentemail) | Give the agente its own dedicated email inbox via AgenteMail. Send, receive, and manage email autonomously using agente-owned email addresses (e.g. hermes-agente@agentemail.to). |

## finance

| Habilidad | Description |
|-------|-------------|
| [**3-statement-modelo**](/docs/guia-usuario/habilidads/optional/finance/finance-3-statement-modelo) | Build fully-integrated 3-statement modelos (IS, BS, CF) in Excel with working capital schedules, D&A roll-forwards, debt schedule, and the plugs that make cash and retained earnings tie. Pairs with excel-author. |
| [**comps-analysis**](/docs/guia-usuario/habilidads/optional/finance/finance-comps-analysis) | Build comparable company analysis in Excel — operating metrics, valuation multiples, statistical benchmarking vs peer sets. Pairs with excel-author. Use for public-company valuation, IPO pricing, sector benchmarking, or outlier detection. |
| [**dcf-modelo**](/docs/guia-usuario/habilidads/optional/finance/finance-dcf-modelo) | Build institutional-quality DCF valuation modelos in Excel — revenue projections, FCF build, WACC, terminal value, Bear/Base/Bull scenarios, 5x5 sensitivity tables. Pairs with excel-author. Use for intrinsic-value equity analysis. |
| [**excel-author**](/docs/guia-usuario/habilidads/optional/finance/finance-excel-author) | Build auditable Excel workbooks headless with openpyxl — blue/black/green cell conventions, formulas over hardcodes, named ranges, balance checks, sensitivity tables. Use for financial modelos, audit outputs, reconciliations. |
| [**lbo-modelo**](/docs/guia-usuario/habilidads/optional/finance/finance-lbo-modelo) | Build leveraged buyout modelos in Excel — sources & uses, debt schedule, cash sweep, exit multiple, IRR/MOIC sensitivity. Pairs with excel-author. Use for PE screening, sponsor-case valuation, or illustrative LBO in a pitch. |
| [**merger-modelo**](/docs/guia-usuario/habilidads/optional/finance/finance-merger-modelo) | Build accretion/dilution (merger) modelos in Excel — pro-forma P&L, synergies, financing mix, EPS impact. Pairs with excel-author. Use for M&A pitches, board materials, or deal evaluation. |
| [**pptx-author**](/docs/guia-usuario/habilidads/optional/finance/finance-pptx-author) | Build PowerPoint decks headless with python-pptx. Pairs with excel-author for modelo-backed decks where every number traces to a workbook cell. Use for pitch decks, IC memos, earnings notes. |
| [**stocks**](/docs/guia-usuario/habilidads/optional/finance/finance-stocks) | Stock quotes, history, search, compare, crypto via Yahoo. |

## gaming

| Habilidad | Description |
|-------|-------------|
| [**minecraft-modpack-server**](/docs/guia-usuario/habilidads/optional/gaming/gaming-minecraft-modpack-server) | Host modded Minecraft servers (CurseForge, Modrinth). |
| [**pokemon-player**](/docs/guia-usuario/habilidads/optional/gaming/gaming-pokemon-player) | Play Pokemon via headless emulator + RAM reads. |

## health

| Habilidad | Description |
|-------|-------------|
| [**fitness-nutrition**](/docs/guia-usuario/habilidads/optional/health/health-fitness-nutrition) | Gym workout planner and nutrition tracker. Search 690+ exercises by muscle, equipment, or category via wger. Look up macros and calories for 380,000+ foods via USDA FoodData Central. Compute BMI, TDEE, one-rep max, macro splits, and body... |
| [**neurohabilidad-bci**](/docs/guia-usuario/habilidads/optional/health/health-neurohabilidad-bci) | Connect to a running NeuroHabilidad instance and incorporate the user's real-time cognitive and emotional state (focus, relaxation, mood, cognitive load, drowsiness, heart rate, HRV, sleep staging, and 40+ derived EXG scores) into responses.... |

## mcp

| Habilidad | Description |
|-------|-------------|
| [**fastmcp**](/docs/guia-usuario/habilidads/optional/mcp/mcp-fastmcp) | Build, test, inspect, install, and deploy MCP servers with FastMCP in Python. Use when creating a new MCP server, wrapping an API or database as MCP herramientas, exposing resources or prompts, or preparing a FastMCP server for Claude Code, Cur... |
| [**mcporter**](/docs/guia-usuario/habilidads/optional/mcp/mcp-mcporter) | Use the mcporter CLI to list, configure, auth, and call MCP servers/herramientas directly (HTTP or stdio), including ad-hoc servers, config edits, and CLI/type generation. |

## migration

| Habilidad | Description |
|-------|-------------|
| [**openclaw-migration**](/docs/guia-usuario/habilidads/optional/migration/migration-openclaw-migration) | Migrate a user's OpenClaw customization footprint into Hermes Agente. Imports Hermes-compatible memories, SOUL.md, command allowlists, user habilidads, and selected workspace assets from ~/.openclaw, then reports exactly what could not be mig... |

## mlops

| Habilidad | Description |
|-------|-------------|
| [**huggingface-accelerate**](/docs/guia-usuario/habilidads/optional/mlops/mlops-accelerate) | Simplest distributed training API. 4 lines to add distributed support to any PyTorch script. Unified API for DeepSpeed/FSDP/Megatron/DDP. Automatic device placement, mixed precision (FP16/BF16/FP8). Interactive config, single launch comm... |
| [**axolotl**](/docs/guia-usuario/habilidads/optional/mlops/mlops-training-axolotl) | Axolotl: YAML LLM fine-tuning (LoRA, DPO, GRPO). |
| [**chroma**](/docs/guia-usuario/habilidads/optional/mlops/mlops-chroma) | Open-source embedding database for AI applications. Store embeddings and metadata, perform vector and full-text search, filter by metadata. Simple 4-function API. Scales from notebooks to production clusters. Use for semantic search, RAG... |
| [**clip**](/docs/guia-usuario/habilidads/optional/mlops/mlops-clip) | OpenAI's modelo connecting vision and language. Enables zero-shot image classification, image-text matching, and cross-modal retrieval. Trained on 400M image-text pairs. Use for image search, content moderation, or vision-language tasks w... |
| [**dspy**](/docs/guia-usuario/habilidads/optional/mlops/mlops-research-dspy) | DSPy: declarative LM programs, auto-optimize prompts, RAG. |
| [**faiss**](/docs/guia-usuario/habilidads/optional/mlops/mlops-faiss) | Facebook's library for efficient similarity search and clustering of dense vectors. Supports billions of vectors, GPU acceleration, and various index types (Flat, IVF, HNSW). Use for fast k-NN search, large-scale vector retrieval, or whe... |
| [**optimizing-attention-flash**](/docs/guia-usuario/habilidads/optional/mlops/mlops-flash-attention) | Optimizes transformer attention with Flash Attention for 2-4x speedup and 10-20x memoria reduction. Use when training/running transformers with long sequences (>512 tokens), encountering GPU memoria issues with attention, or need faster in... |
| [**guidance**](/docs/guia-usuario/habilidads/optional/mlops/mlops-guidance) | Control LLM output with regex and grammars, guarantee valid JSON/XML/code generation, enforce structured formats, and build multi-step workflows with Guidance - Microsoft Research's constrained generation framework |
| [**huggingface-tokenizers**](/docs/guia-usuario/habilidads/optional/mlops/mlops-huggingface-tokenizers) | Fast tokenizers optimized for research and production. Rust-based implementation tokenizes 1GB in &lt;20 seconds. Supports BPE, WordPiece, and Unigram algorithms. Train custom vocabularies, track alignments, handle padding/truncation. Integ... |
| [**instructor**](/docs/guia-usuario/habilidads/optional/mlops/mlops-instructor) | Extract structured data from LLM responses with Pydantic validation, retry failed extractions automatically, parse complex JSON with type safety, and stream partial results with Instructor - battle-tested structured output library |
| [**lambda-labs-gpu-cloud**](/docs/guia-usuario/habilidads/optional/mlops/mlops-lambda-labs) | Reserved and on-demand GPU cloud instances for ML training and inference. Use when you need dedicated GPU instances with simple SSH access, persistent filesystems, or high-performance multi-node clusters for large-scale training. |
| [**llava**](/docs/guia-usuario/habilidads/optional/mlops/mlops-llava) | Large Language and Vision Assistant. Enables visual instruction tuning and image-based conversations. Combines CLIP vision encoder with Vicuna/LLaMA language modelos. Supports multi-turn image chat, visual question answering, and instruct... |
| [**modal-serverless-gpu**](/docs/guia-usuario/habilidads/optional/mlops/mlops-modal) | Serverless GPU cloud platform for running ML workloads. Use when you need on-demand GPU access without infrastructure management, deploying ML modelos as APIs, or running batch jobs with automatic scaling. |
| [**nemo-curator**](/docs/guia-usuario/habilidads/optional/mlops/mlops-nemo-curator) | GPU-accelerated data curation for LLM training. Supports text/image/video/audio. Features fuzzy deduplication (16× faster), quality filtering (30+ heuristics), semantic deduplication, PII redaction, NSFW detection. Scales across GPUs wit... |
| [**obliteratus**](/docs/guia-usuario/habilidads/optional/mlops/mlops-obliteratus) | OBLITERATUS: abliterate LLM refusals (diff-in-means). |
| [**outlines**](/docs/guia-usuario/habilidads/optional/mlops/mlops-inference-outlines) | Outlines: structured JSON/regex/Pydantic LLM generation. |
| [**peft-fine-tuning**](/docs/guia-usuario/habilidads/optional/mlops/mlops-peft) | Parameter-efficient fine-tuning for LLMs using LoRA, QLoRA, and 25+ methods. Use when fine-tuning large modelos (7B-70B) with limited GPU memoria, when you need to train &lt;1% of parameters with minimal accuracy loss, or for multi-adapter se... |
| [**pinecone**](/docs/guia-usuario/habilidads/optional/mlops/mlops-pinecone) | Managed vector database for production AI applications. Fully managed, auto-scaling, with hybrid search (dense + sparse), metadata filtering, and namespaces. Low latency (&lt;100ms p95). Use for production RAG, recommendation systems, or se... |
| [**pytorch-fsdp**](/docs/guia-usuario/habilidads/optional/mlops/mlops-pytorch-fsdp) | Expert guidance for Fully Sharded Data Parallel training with PyTorch FSDP - parameter sharding, mixed precision, CPU offloading, FSDP2 |
| [**pytorch-lightning**](/docs/guia-usuario/habilidads/optional/mlops/mlops-pytorch-lightning) | High-level PyTorch framework with Trainer class, automatic distributed training (DDP/FSDP/DeepSpeed), callbacks system, and minimal boilerplate. Scales from laptop to supercomputer with same code. Use when you want clean training loops w... |
| [**qdrant-vector-search**](/docs/guia-usuario/habilidads/optional/mlops/mlops-qdrant) | High-performance vector similarity search engine for RAG and semantic search. Use when building production RAG systems requiring fast nearest neighbor search, hybrid search with filtering, or scalable vector storage with Rust-powered per... |
| [**sparse-autoencoder-training**](/docs/guia-usuario/habilidads/optional/mlops/mlops-saelens) | Provides guidance for training and analyzing Sparse Autoencoders (SAEs) using SAELens to decompose neural network activations into interpretable features. Use when discovering interpretable features, analyzing superposition, or studying... |
| [**simpo-training**](/docs/guia-usuario/habilidads/optional/mlops/mlops-simpo) | Simple Preference Optimization for LLM alignment. Reference-free alternative to DPO with better performance (+6.4 points on AlpacaEval 2.0). No reference modelo needed, more efficient than DPO. Use for preference alignment when want simpl... |
| [**slime-rl-training**](/docs/guia-usuario/habilidads/optional/mlops/mlops-slime) | Provides guidance for LLM post-training with RL using slime, a Megatron+SGLang framework. Use when training GLM modelos, implementing custom data generation workflows, or needing tight Megatron-LM integration for RL scaling. |
| [**stable-diffusion-image-generation**](/docs/guia-usuario/habilidads/optional/mlops/mlops-stable-diffusion) | State-of-the-art text-to-image generation with Stable Diffusion modelos via HuggingFace Diffusers. Use when generating images from text prompts, performing image-to-image translation, inpainting, or building custom diffusion pipelines. |
| [**tensorrt-llm**](/docs/guia-usuario/habilidads/optional/mlops/mlops-tensorrt-llm) | Optimizes LLM inference with NVIDIA TensorRT for maximum throughput and lowest latency. Use for production deployment on NVIDIA GPUs (A100/H100), when you need 10-100x faster inference than PyTorch, or for serving modelos with quantizatio... |
| [**distributed-llm-pretraining-torchtitan**](/docs/guia-usuario/habilidads/optional/mlops/mlops-torchtitan) | Provides PyTorch-native distributed LLM pretraining using torchtitan with 4D parallelism (FSDP2, TP, PP, CP). Use when pretraining Llama 3.1, DeepSeek V3, or custom modelos at scale from 8 to 512+ GPUs with Float8, torch.compile, and dist... |
| [**fine-tuning-with-trl**](/docs/guia-usuario/habilidads/optional/mlops/mlops-training-trl-fine-tuning) | TRL: SFT, DPO, PPO, GRPO, reward modeloing for LLM RLHF. |
| [**unsloth**](/docs/guia-usuario/habilidads/optional/mlops/mlops-training-unsloth) | Unsloth: 2-5x faster LoRA/QLoRA fine-tuning, less VRAM. |
| [**whisper**](/docs/guia-usuario/habilidads/optional/mlops/mlops-whisper) | OpenAI's general-purpose speech recognition modelo. Supports 99 languages, transcription, translation to English, and language identification. Six modelo sizes from tiny (39M params) to large (1550M params). Use for speech-to-text, podcast... |

## payments

| Habilidad | Description |
|-------|-------------|
| [**mpp-agente**](/docs/guia-usuario/habilidads/optional/payments/payments-mpp-agente) | Pay HTTP 402 APIs via Machine Payments Protocol (MPP). |
| [**stripe-link-cli**](/docs/guia-usuario/habilidads/optional/payments/payments-stripe-link-cli) | Agente payments via Stripe Link — cards, SPT, approvals. |
| [**stripe-projects**](/docs/guia-usuario/habilidads/optional/payments/payments-stripe-projects) | Provision SaaS services + sync creds via Stripe Projects. |

## productivity

| Habilidad | Description |
|-------|-------------|
| [**canvas**](/docs/guia-usuario/habilidads/optional/productivity/productivity-canvas) | Canvas LMS integration — fetch enrolled courses and assignments using API token autenticación. |
| [**here.now**](/docs/guia-usuario/habilidads/optional/productivity/productivity-here-now) | Publish static sites to &#123;slug&#125;.here.now and store private files in cloud Drives for agente-to-agente handoff. |
| [**memento-flashcards**](/docs/guia-usuario/habilidads/optional/productivity/productivity-memento-flashcards) | Spaced-repetition flashcard system. Create cards from facts or text, chat with flashcards using free-text answers graded by the agente, generate quizzes from YouTube transcripts, review due cards with adaptive scheduling, and export/impor... |
| [**shop**](/docs/guia-usuario/habilidads/optional/productivity/productivity-shop) | Shop catalog search, checkout, order tracking, returns. |
| [**shopify**](/docs/guia-usuario/habilidads/optional/productivity/productivity-shopify) | Shopify Admin & Storefront GraphQL APIs via curl. Products, orders, customers, inventory, metafields. |
| [**siyuan**](/docs/guia-usuario/habilidads/optional/productivity/productivity-siyuan) | SiYuan Note API for searching, reading, creating, and managing blocks and documents in a self-hosted knowledge base via curl. |
| [**telephony**](/docs/guia-usuario/habilidads/optional/productivity/productivity-telephony) | Give Hermes phone capabilities without core herramienta changes. Provision and persist a Twilio number, send and receive SMS/MMS, make direct calls, and place AI-driven outbound calls through Bland.ai or Vapi. |

## research

| Habilidad | Description |
|-------|-------------|
| [**bioinformatics**](/docs/guia-usuario/habilidads/optional/research/research-bioinformatics) | Puerta de enlace to 400+ bioinformatics habilidads from bioHabilidads and ClawBio. Covers genomics, transcriptomics, single-cell, variant calling, pharmacogenomics, metagenomics, structural biology, and more. Fetches domain-specific reference material on... |
| [**darwinian-evolver**](/docs/guia-usuario/habilidads/optional/research/research-darwinian-evolver) | Evolve prompts/regex/SQL/code with Imbue's evolution loop. |
| [**domain-intel**](/docs/guia-usuario/habilidads/optional/research/research-domain-intel) | Passive domain reconnaissance using Python stdlib. Subdomain discovery, SSL certificate inspection, WHOIS lookups, DNS records, domain availability checks, and bulk multi-domain analysis. No API keys required. |
| [**drug-discovery**](/docs/guia-usuario/habilidads/optional/research/research-drug-discovery) | Pharmaceutical research assistant for drug discovery workflows. Search bioactive compounds on ChEMBL, calculate drug-likeness (Lipinski Ro5, QED, TPSA, synthetic accessibility), look up drug-drug interactions via OpenFDA, interpret ADMET... |
| [**duckduckgo-search**](/docs/guia-usuario/habilidads/optional/research/research-duckduckgo-search) | Free web search via DuckDuckGo — text, news, images, videos. No API key needed. Prefer the `ddgs` CLI when installed; use the Python DDGS library only after verifying that `ddgs` is available in the current runtime. |
| [**gitnexus-explorer**](/docs/guia-usuario/habilidads/optional/research/research-gitnexus-explorer) | Index a codebase with GitNexus and serve an interactive knowledge graph via web UI + Cloudflare tunnel. |
| [**osint-investigation**](/docs/guia-usuario/habilidads/optional/research/research-osint-investigation) | Public-records OSINT investigation framework — SEC EDGAR filings, USAspending contracts, Senate lobbying, OFAC sanctions, ICIJ offshore leaks, NYC property records (ACRIS), OpenCorporates registries, CourtListener court records, Wayback... |
| [**parallel-cli**](/docs/guia-usuario/habilidads/optional/research/research-parallel-cli) | Optional vendor habilidad for Parallel CLI — agente-native web search, extraction, deep research, enrichment, FindAll, and monitoring. Prefer JSON output and non-interactive flows. |
| [**qmd**](/docs/guia-usuario/habilidads/optional/research/research-qmd) | Search personal knowledge bases, notes, docs, and meeting transcripts locally using qmd — a hybrid retrieval engine with BM25, vector search, and LLM reranking. Supports CLI and MCP integration. |
| [**scrapling**](/docs/guia-usuario/habilidads/optional/research/research-scrapling) | Web scraping with Scrapling - HTTP fetching, stealth browser automation, Cloudflare bypass, and spider crawling via CLI and Python. |
| [**searxng-search**](/docs/guia-usuario/habilidads/optional/research/research-searxng-search) | Free meta-search via SearXNG — aggregates results from 70+ search engines. Self-hosted or use a public instance. No API key needed. Falls back automatically when the web search herramientaset is unavailable. |

## security

| Habilidad | Description |
|-------|-------------|
| [**1password**](/docs/guia-usuario/habilidads/optional/security/security-1password) | Set up and use 1Password CLI (op). Use when installing the CLI, enabling desktop app integration, signing in, and reading/injecting secrets for commands. |
| [**godmode**](/docs/guia-usuario/habilidads/optional/security/security-godmode) | Jailbreak LLMs: Parseltongue, GODMODE, ULTRAPLINIAN. |
| [**oss-forensics**](/docs/guia-usuario/habilidads/optional/security/security-oss-forensics) | Supply chain investigation, evidence recovery, and forensic analysis for GitHub repositories. Covers deleted commit recovery, force-push detection, IOC extraction, multi-source evidence collection, hypothesis formation/validation, and st... |
| [**sherlock**](/docs/guia-usuario/habilidads/optional/security/security-sherlock) | OSINT username search across 400+ social networks. Hunt down social media accounts by username. |
| [**web-pentest**](/docs/guia-usuario/habilidads/optional/security/security-web-pentest) | Authorized web application penetration testing — reconnaissance, vulnerability analysis, proof-based exploitation, and professional reporting. Adapts Shannon's "No Exploit, No Report" methodology with hard guardrails for scope, authoriza... |

## software-development

| Habilidad | Description |
|-------|-------------|
| [**code-wiki**](/docs/guia-usuario/habilidads/optional/software-development/software-development-code-wiki) | Generate wiki docs + Mermaid diagrams for any codebase. |
| [**rest-graphql-debug**](/docs/guia-usuario/habilidads/optional/software-development/software-development-rest-graphql-debug) | Debug REST/GraphQL APIs: status codes, auth, schemas, repro. |
| [**subagente-driven-development**](/docs/guia-usuario/habilidads/optional/software-development/software-development-subagente-driven-development) | Execute plans via delegate_task subagentes (2-stage review). |

## web-development

| Habilidad | Description |
|-------|-------------|
| [**cloudflare-temporary-deploy**](/docs/guia-usuario/habilidads/optional/web-development/web-development-cloudflare-temporary-deploy) | Deploy a Worker live, no account, via wrangler --temporary. |
| [**page-agente**](/docs/guia-usuario/habilidads/optional/web-development/web-development-page-agente) | Embed alibaba/page-agente into your own web application — a pure-JavaScript in-page GUI agente that ships as a single &lt;script> tag or npm package and lets end-users of your site drive the UI with natural language ("click login, fill userna... |

---

## Contribuyendo Optional Habilidads

To add a new optional habilidad to the repository:

1. Create a directory under `optional-habilidads/<category>/<habilidad-name>/`
2. Add a `SKILL.md` with standard frontmatter (name, description, version, author)
3. Include any supporting files in `references/`, `templates/`, or `scripts/` subdirectories
4. Submit a pull request — the habilidad will appear in this catalog and get its own docs page once merged

---