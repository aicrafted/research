# Current State of Spiking Neural Networks 2024–2026

## Landscape overview

Spiking Neural Networks (SNNs) in 2024–2026 are best described as a *maturing-but-still-specialized* paradigm: they are no longer confined to small toy problems, but they also have not displaced conventional artificial neural networks (ANNs) as a default choice for mainstream vision/NLP workloads. The mainstream research view is that SNNs are most compelling when **temporal structure and sparsity are intrinsic to the data** (e.g., event cameras, always-on sensing, streaming inference), and when the deployment target can exploit **event-driven computation** (neuromorphic chips, or tightly optimized sparse hardware). In contrast, when SNNs are evaluated as time-unrolled simulations on GPUs, they often lose much of the practical efficiency rationale (time-unrolling overhead, dense tensor kernels, and substantial memory/time costs). citeturn17view0turn18view0turn11view0

A key change in this period is that **deep SNN training with gradient-based methods is now routine** in the literature and increasingly “off-the-shelf” from a tooling perspective (PyTorch-like environments, surrogate-gradient training, increasingly standardized codebases). citeturn17view0turn4search4turn2search28 At the same time, the community remains split—sometimes sharply—on what constitutes a *meaningful* advantage. Many papers report **operation-count proxies** (e.g., synaptic operations, MAC vs AC counts, timesteps) or “theoretical energy,” while fewer report **end-to-end measured energy and latency** on a real neuromorphic system including I/O and preprocessing. This measurement gap is precisely what motivated standardized benchmarking efforts such as NeuroBench, which stresses system-level accounting (including pre/post processing) and comparable power/latency measurement practice. citeturn18view0

The most commonly cited bottlenecks limiting wider adoption in 2024–2026 cluster into four categories:

First, **programmability and deployment maturity**: for many years, deploying to neuromorphic hardware required highly specialized expertise, and even today the toolchain and compiler layers are still evolving (with progress being fast but not yet at the “CUDA for spikes” level). citeturn17view0turn20view0

Second, **training cost and temporal credit assignment**: training SNNs with backpropagation-through-time (BPTT) scales poorly with sequence length because intermediate states must be stored; long-horizon temporal credit assignment remains a fundamental choke point (especially for real-time, long-duration learning). Recent work explicitly frames the lack of “online learning with low memory footprint over long horizons” as a missing ingredient and proposes new systems to address it (e.g., BrainTrace, 2026). citeturn25view0

Third, **dataset and evaluation fit**: much of ML benchmarking remains based on frame-based datasets that do not inherently reward event-driven temporal sparsity. Surveys of learning rules emphasize that ANN-oriented datasets can underutilize SNNs’ spatiotemporal advantages and can lead to “SNNs as costly RNNs” outcomes unless carefully encoded. citeturn6search26turn18view0

Fourth, **the simulation–hardware gap**: differences in discretization, spike timing conventions, limited numeric precision, routing constraints, and chip-specific neuron/synapse implementations create mismatches between GPU simulations and deployed behavior. A concrete example: Neuromorphic Intermediate Representation (NIR) reports systematic differences such as timestep-delayed spiking behavior in specific simulator/hardware combinations—small at the unit level, but capable of compounding in deep temporal pipelines. citeturn20view0

## Active research directions

This section summarizes active directions in 2024–2026, each with motivation, key drivers, maturity assessment, and open issues.

**Temporal coding schemes** (maturing → still fragmented).  
Core idea: move beyond rate-based coding toward **precise spike timing** (e.g., time-to-first-spike, rank-order, phase-based or burst-based codes) to exploit temporal sparsity and reduce latency/energy. A representative 2025 line of work focuses on **Time-to-First-Spike (TTFS)** as an “at most one spike per neuron” regime and develops training stabilizers (initialization, normalization, temporal decoding) to mitigate vanishing gradients and sparse-firing instability; the method reports strong TTFS results on MNIST/Fashion-MNIST/CIFAR10 and DVS Gesture. citeturn22view0  
Known limitations: TTFS and other sparse timing codes can be brittle (training instability; sensitivity to noise and discretization), and it is still unclear how universally such codes scale to large models and complex multimodal tasks without reintroducing dense surrogate constructs or heavy architectural scaffolding. citeturn22view0turn25view0

**Neuromorphic hardware co-design** (maturing).  
Core idea: co-design SNN algorithms with real hardware constraints—routing, limited fan-in/out, state precision, on-chip learning primitives, and I/O. Key drivers include standardized intermediate representations and multi-backend toolchains (NIR; Lava), and scaling systems such as Hala Point. NIR’s explicit goal is backend interoperability across simulators (e.g., Norse, snnTorch, Rockpool) and hardware (Loihi 2 via Lava, SpiNNaker2, Xylo). citeturn20view0  
On the systems side, entity["company","Intel","semiconductor company"]’s 2024 Hala Point system exemplifies the “scale-up” push: it uses 1,152 Loihi 2 processors and is positioned as a research platform to probe scalable, energy-sustainable neuromorphic computation. citeturn8search0turn8search11  
Open problems: mapping and compilation remain hard; “efficient on paper” models can underperform if the deployment pipeline includes heavy event-to-frame preprocessing or if workloads do not generate enough sparsity. NeuroBench explicitly notes that pre/post processing can dominate system time/energy and must be measured. citeturn18view0turn17view0

**Hybrid ANN–SNN architectures** (maturing; strong practical traction).  
Core idea: combine fast, sparse, event-driven front-ends (SNN blocks) with dense ANN modules where training stability and representational power are needed. This direction is strongly pragmatic: it treats SNNs as an efficient temporal “sensor encoder/backbone” and uses ANN layers for high-accuracy late fusion or prediction.  
A concrete example: entity["people","Asude Aydin","event vision researcher"] and colleagues (CVPR Workshops 2024) propose an ANN “state initializer” that periodically seeds an SNN to avoid long transients/decay; they report large power reductions with modest accuracy degradation for event-based human pose estimation. citeturn23search3turn23search0  
Another example: entity["people","Soikat Hasan Ahmed","neuromorphic vision researcher"] et al. (CVPR 2025) propose attention-based SNN–ANN bridge modules for event-based object detection and demonstrate partial deployment of the SNN block on Loihi 2, reporting measured power/time. citeturn12view0turn11view0  
Limitations: hybrids can dilute the “pure spiking” motivation and often move the hard compute back into dense ANN blocks (sometimes on GPUs/cloud), raising questions about where the net system-level efficiency truly comes from. citeturn11view0turn18view0

**Large-scale SNN scaling and architecture modernization** (emerging → maturing, with rapid progress and some contested claims).  
Core idea: adopt modern deep learning architectures (ResNets, ViTs, attention variants, masked self-supervised pretraining) into spiking form while controlling the timestep/latency cost. An important marker is ImageNet-scale accuracy with **very small timesteps**.  
A particularly visible milestone: entity["people","X Shi","computer vision researcher"] et al. (CVPR 2024) report SpikingResformer with **79.40% top-1 on ImageNet** using **4 time-steps**, positioning it as state-of-the-art among several directly trained SNN families at that time. citeturn7search2turn7search0  
In parallel, entity["people","Zhaokun Zhou","spiking transformer researcher"] et al. (arXiv 2024) describe Spikformer V2 and claim **80.38% top-1 (4 timesteps)**, and **81.10% top-1 with 1 timestep** after masked self-supervised pretraining. (These are influential, but some results are preprint-based; community consensus on exact “SOTA” ordering shifts quickly.) citeturn23search2turn23search4  
Open issues: scaling often pushes SNNs toward “deep learning with spikes,” raising concerns about whether improvements come from carefully engineered ANN-like tricks (normalization, surrogate gradients, discretization choices) rather than uniquely spiking computation. This is an active—and sometimes contested—debate. citeturn17view0turn25view0

**Biological plausibility vs engineering pragmatism** (maturing; ongoing tension).  
Core idea: manage the tradeoff between biologically plausible learning (local synaptic rules, online learning, neuromodulated plasticity) and engineering-effective training (global loss, backprop, large minibatches). A widely cited position is that local rules (STDP variants) are valuable but insufficient for arbitrary application programming, driving the field toward ML-like example-driven training. citeturn17view0turn24search0  
A major 2026 inflection point in this debate is BrainTrace, which explicitly targets **online learning over long horizons** with a **linear-memory** approach plus compiler automation, aiming to reconcile usability and biological fidelity constraints that historically forced researchers into narrow neuron models or quadratic-memory eligibility traces. citeturn25view0  
Open issues: whether such online-learning systems can match offline BPTT accuracy at scale across diverse tasks remains unsettled; many approaches show promising results but with nontrivial complexity or hardware assumptions. citeturn25view0turn18view0

**In-memory and event-driven computing** (maturing in hardware; exploratory in full stack integration).  
Core idea: attack the von Neumann bottleneck by placing memory close to compute (digital SRAM-distributed designs like NorthPole; or analog/memristive compute-in-memory). This direction is broader than “spikes,” but strongly adjacent because SNNs naturally emphasize sparse communication and localized state.  
entity["company","IBM","technology company"]’s NorthPole program (Science 2023) is often cited as a memory–compute co-designed inference architecture emphasizing energy/space/time efficiency against conventional CPU/GPU baselines on vision workloads. Importantly, NorthPole is brain-inspired but not a native spiking event-driven SNN processor in the Loihi/SpiNNaker sense—so its relevance is partly “architectural adjacency.” citeturn14search0turn13view0  
On the neuromorphic side, Muir & Sheik (2025) argue that in-memory computing designs are approaching commercial maturity and that digital neuromorphic designs are increasingly replacing analog/mixed-signal approaches to simplify deployment while keeping key efficiency benefits. citeturn17view0

## Training methods

SNN training in 2024–2026 is best understood as a spectrum: from local unsupervised rules (high biological plausibility, limited task reach) to full gradient-based training (high performance, heavy compute and weaker biological plausibility), plus several hybrid and emerging methods attempting to bridge the two.

**STDP and variants (local plasticity)**  
Mechanism: Spike-Timing-Dependent Plasticity updates synapses from local temporal correlations between pre/post spikes (often with lateral inhibition and homeostasis for competition).  
Benchmark effectiveness: historically strong on small datasets in unsupervised settings; a canonical reference is entity["people","Peter U. Diehl","computational neuroscience researcher"] & entity["people","Matthew Cook","computational neuroscience researcher"] (Frontiers in Computational Neuroscience, 2015), reporting **~95%** MNIST accuracy with an STDP-trained network. citeturn24search0turn24search2  
Limitations: STDP alone is widely viewed as insufficient for “programmable general applications,” typically requiring hand-engineered architectures and struggling to scale to large supervised benchmarks without substantial additional machinery. citeturn17view0turn6search26  
Maturity: maturing scientifically (well-studied), but practical applicability is mostly niche (unsupervised feature learning, adaptation, small-scale edge tasks).

**ANN-to-SNN conversion**  
Mechanism: train an ANN with standard backprop, then convert activations/weights to spiking neurons; often requires calibration (threshold balancing, bias corrections) and can require many timesteps to approximate ANN activations via spike rates.  
Effectiveness: conversion remains one of the highest-accuracy routes on large static datasets because it leverages mature ANN training pipelines. A key line of work by entity["people","Tong Bu","spiking conversion researcher"] et al. (ICLR 2022) targets **high-accuracy, ultra-low-latency conversion**, explicitly addressing accuracy collapse at small timesteps. citeturn16search11turn16search3  
In 2024, entity["people","Xiaofeng Wu","computer vision researcher"] et al. (ECCV 2024) propose Forward Temporal Bias Correction (FTBC) to correct temporal bias effects and report improved conversion outcomes on CIFAR and ImageNet. citeturn6search7turn6search3  
Emerging (2024–2026): conversion has expanded beyond CNNs to Transformers; for example, Jiang et al. (ICLR 2024) propose a *training-free* spiking conversion route for Transformers (STA). citeturn6search35  
Limitations: conversion can produce highly accurate SNNs, but **energy/latency benefits depend on timestep count**, hardware support, and the degree to which spike sparsity is preserved end-to-end. It can also inherit ANN inductive biases that may not be optimal for event-driven sensing. citeturn18view0turn17view0  
Maturity: maturing, and arguably the most “production-aligned” training path today when the target is an existing ANN pipeline.

**BPTT and surrogate-gradient training (direct training)**  
Mechanism: unroll the SNN over time and apply backpropagation through time; because spikes are non-differentiable, use surrogate gradients or differentiable approximations for the spike function.  
Effectiveness: this family is the main driver behind rapidly improving ImageNet-scale results with small timesteps. For example, SpikingResformer (CVPR 2024) reports **79.40% ImageNet top-1 at 4 timesteps** using direct training. citeturn7search2turn7search0 Spikformer V2 (arXiv 2024) reports **80.38% at 4 timesteps**, plus **81.10% at 1 timestep** after self-supervised pretraining (noting its preprint status and that exact SOTA standing is fluid). citeturn23search2turn23search4  
Limitations: BPTT’s memory grows with the number of timesteps, making long-horizon training expensive; simulator fidelity and discretization choices can strongly impact results and portability to hardware. citeturn25view0turn20view0  
Maturity: maturing rapidly; increasingly standard in SNN research.

**Surrogate-gradient design and theory**  
Mechanism: choose surrogate derivatives (piecewise-linear, sigmoid-like, stochastic) to stabilize gradient flow through discontinuous spikes.  
Representative works: Neftci, Mostafa & Zenke (IEEE Signal Processing Magazine, 2019) remains a foundational tutorial-style exposition; Eshraghian et al. (Proc. IEEE, 2023) provides updated practical guidance and emphasizes deep-learning-style engineering for SNNs. citeturn16search2turn4search8turn4search4  
A 2025 Neural Computation paper analyzes theoretical underpinnings for surrogate derivatives in stochastic SNNs, reflecting ongoing interest in “why surrogates work.” citeturn16search36  
Limitations: surrogate choices can be architecture- and task-sensitive; empirical success does not automatically translate to biologically plausible learning.

**Exact or event-based gradient formulations**  
Mechanism: derive gradients in continuous-time spiking dynamics (adjoint methods) to avoid heuristic surrogates; compute gradients at event times.  
Representative works: EventProp (Wunderlich & Pehle, Scientific Reports 2021) derives exact gradient computation for spiking systems and motivates event-based learning implementations. citeturn16search5turn16search1  
Limitations: implementing exact-event training efficiently and scalably is hard; integration into mainstream deep SNN stacks is still emerging.

**Hybrid learning rules and online learning**  
Mechanism: combine global loss training with local plasticity or eligibility traces; aim for online updates without storing full trajectories.  
Representative 2026 inflection point: BrainTrace (Wang et al., Nature Communications 2026) proposes a model-agnostic, compiler-supported, linear-memory online learning framework, positioning itself against BPTT’s memory scaling limits. citeturn25view0  
Limitations: many online methods trade accuracy, generality, or assume simplified neuron dynamics; BrainTrace’s longer-term impact will depend on adoption and replication across tasks/hardware.

**Neuroevolution and architecture search**  
Mechanism: evolve architectures/parameters, often for hardware constraints or on-device learning; can be resilient to non-differentiabilities but computationally expensive.  
Representative works: SpikeNAS (Putra & Shafique, 2024) frames architecture search as memory-aware under embedded constraints, reflecting the hardware–algorithm co-design trend. citeturn6search1turn6search13  
Limitations: search cost and reproducibility; unclear scaling to very large SNNs compared to gradient-based methods.

## Best-fit tasks and demonstrated performance

SNNs in 2024–2026 look most competitive—or credibly *better* in at least one key metric—when tasks are naturally temporal, sparse, and latency constrained, and when performance is evaluated with system-level constraints in mind.

Event-based vision remains the flagship domain because event cameras generate sparse streams of asynchronous events that align structurally with spiking computation. Yet even here, the best-performing practical approaches often end up hybrid (SNN front-end + ANN/RNN back-end), implying that “pure SNN superiority” is not a settled story. citeturn11view0turn17view0

In event-based object detection, Ahmed et al. (CVPR 2025) report a hybrid SNN–ANN detector with results comparable to ANN/RNN baselines while enabling deployment of the SNN block on Loihi 2; they directly report Loihi 2 measurements for the SNN block: **~1.7 W** and **~1.9 ms per step** (in one configuration, across 6 chips), and provide an energy comparison in mJ across model families. citeturn12view0turn11view0 This is an unusually concrete, hardware-tethered result compared to purely simulated energy proxies.

For event-based 2D/3D pose estimation, Aydin et al. (CVPRW 2024) explicitly report that their hybrid ANN–SNN approach consumes **88% less power** with only a **~4% performance decrease** compared to fully ANN baselines at the same inference rate (task: event-based pose). While details of measurement context matter (power domains, hardware platform), this is another strong “best-fit domain” indicator: mixed temporal workloads with tight response constraints. citeturn23search0turn23search3

On event-based classification, TTFS training work targets latencies and sparsity by design. Che et al. (arXiv 2025) report TTFS state-of-the-art among TTFS methods, including **95.83% on DVS Gesture**, alongside strong MNIST/Fashion-MNIST/CIFAR10 results. citeturn22view0 For DVS-CIFAR10, other 2025–2026 preprints report **~82%** ranges with low timesteps (for example, a refractory-period–inspired neuron dynamics paper reports **82.40% on CIFAR10-DVS** and **97.22% on DVS Gesture**, but as a preprint this should be treated as provisional). citeturn10search2

For temporal/sequential modeling beyond event vision, there is credible progress but less consensus on “clear superiority.” Lv et al. (ICML 2024) propose an SNN-based framework for time-series forecasting and claim comparable or superior forecasting performance with lower estimated energy consumption, emphasizing the importance of temporal alignment/encoding choices. citeturn5search12turn5search0 On long-range sequence benchmarks, Stan et al. (Scientific Reports 2024) argue that SSM-based SNNs can outperform Transformers on Long Range Arena tasks, suggesting a plausible niche where spiking dynamics and state-space structure reinforce each other. citeturn5search6turn5search2 Follow-on work formalizes “spiking state space models” as a direction (e.g., SpikingSSMs at AAAI 2025). citeturn5search17

Edge inference and IoT are repeatedly cited as the most likely early adoption targets—*but performance is highly hardware-dependent*. Muir & Sheik (2025) explicitly argue for ultra-low-power neuromorphic adoption in battery-powered systems, IoT, and wearables, contingent on solving programmability and scalable deployment. citeturn17view0 NeuroBench’s system track reinforces why: even if an SNN core is efficient, preprocessing and system integration can dominate, especially when inputs/outputs are non-standard event modalities. citeturn18view0

In reinforcement learning and control, the evidence base is expanding but remains mixed. SpiNNaker2-related work reports hardware-aware fine-tuning of spiking Q-networks with energy comparisons against GPU baselines, claiming up to **32× energy reduction** with latency “on par” in their evaluated setup (details matter: task, model size, measurement domain). citeturn21search26turn15view0 Separately, evolutionary/heterosynaptic learning work claims competitive training outcomes on MNIST and Atari-like tasks, suggesting non-gradient routes remain viable for certain RL-style regimes. citeturn6search28turn6search36

For anomaly detection and sparse signal processing, 2025–2026 pipelines increasingly emphasize end-to-end deployment constraints. An arXiv 2025 work on radio-frequency interference detection reports an SNN pipeline mapped onto SynSense Xylo hardware with **on-chip power measurements** and “instrument-scaled inference at 100 mW” (a rare measured deployment detail). citeturn9search24

## Neuromorphic hardware ecosystem

The 2024–2026 hardware ecosystem spans “true spiking neuromorphic processors” and “brain-inspired memory–compute architectures.” The former directly targets event-driven spiking computation; the latter often pursues in-memory acceleration for dense networks while borrowing brain-inspired architectural principles.

Among spiking-centric platforms, Loihi 2 remains a central research workhorse. NIR describes Loihi 2 as supporting up to **1 million neurons and 120 million synapses per chip**, with support for custom neuron models and **three-factor learning rules**, and notes its tight integration with the Lava software stack. citeturn20view1turn20view2 At the systems scale, Intel’s Hala Point—announced in April 2024 and deployed at entity["organization","Sandia National Laboratories","Albuquerque, NM, US"]—packages 1,152 Loihi 2 processors and is described as supporting **1.15 billion neurons and 128 billion synapses**, with **maximum power consumption of 2,600 W** (system-level). citeturn8search0turn8search11

entity["company","BrainChip","neuromorphic company"]’s Akida line is positioned as a commercial neuromorphic edge accelerator emphasizing temporal/event-based processing. The Akida 2 Processor IP product brief emphasizes “spatio-temporal properties” with Temporal Event-based Neural Nets and a co-processor-style integration model. citeturn21search3 BrainChip also highlights M.2-form-factor deployment with a **~1 W power budget** framing for edge configurations (board-level context). citeturn21search11 (Power/efficiency claims for Akida vary across marketing materials; the most reliable interpretations are those tied to specific evaluated workloads and measured domains.)

For analog/mixed-signal accelerated neuromorphic platforms, BrainScaleS-2 represents a distinct design point: fast accelerated emulation for neuroscience and hybrid ML experimentation. Pehle et al. (Frontiers in Neuroscience, 2022) describe BrainScaleS-2 as running **~1000× faster than biological real time** (relative to modeled biological dynamics), enabling accelerated experimentation with plasticity. citeturn21search0turn21search4 Public technical materials describe BrainScaleS-2 as combining analog/mixed-signal neuron/synapse emulation with digital connectivity, programmable plasticity processors, and a machine-learning acceleration unit. citeturn21search12turn21search36 Tooling is increasingly ML-friendly: hxtorch.snn integrates hardware-in-the-loop training into PyTorch-style workflows. citeturn21search1turn21search9

SpiNNaker2 targets massive-scale event-based computation in a more software-defined, many-core digital architecture. A 2024 SpiNNaker2 system paper describes a chip with **152 ARM Cortex-M4F cores**, event-based interrupt-driven processing, a packet router for spike communication, and scaling toward large multi-chip systems (with a reported manufacturing scale of **>35k chips** forming a large “brain-like supercomputer” installation). citeturn15view0turn21search6 Tooling includes py-spinnaker2, described as a lightweight interface for running experiments on SpiNNaker2. citeturn21search2turn20view0

NorthPole sits in the “brain-inspired, memory–compute co-design” category rather than spiking neuromorphic processing. The Science 2023 NorthPole paper argues for frontier performance on energy/space/time metrics for neural inference via a spatial, memory-centric architecture. citeturn14search0 An IBM ISSCC 2024 invited talk page describes a 12nm chip with **22B transistors**, **256-core array**, and **192MB distributed SRAM**, with TOPS figures parameterized by precision. citeturn13view0 IBM also reports system-level prototypes scaling NorthPole cards for LLM inference; these are promising but should be interpreted carefully because comparisons depend on batching, power instrumentation, and pipeline boundaries. citeturn14search2turn14search3turn14search19

Software and deployment stacks are becoming the “glue layer” for the ecosystem. SpikingJelly (Science Advances, 2023) is positioned as an open-source infrastructure for spike-based intelligence. citeturn2search21 snnTorch positions itself as a PyTorch-based library for gradient-based SNN learning. citeturn4search0 Lava and Lava-DL are positioned as Intel’s open-source stack for describing neuromorphic processes and mapping them to CPU or Loihi 2 backends, including deep event-based network training paths. citeturn4search25turn20view1 NIR explicitly aims to connect multiple software frameworks and hardware backends through a unified instruction set/graph representation. citeturn20view0

image_group{"layout":"carousel","aspect_ratio":"16:9","query":["Intel Loihi 2 neuromorphic chip","IBM NorthPole chip","BrainScaleS-2 neuromorphic system","SpiNNaker2 neuromorphic board"],"num_per_query":1}

## Competitive benchmarks and measurement practice

The strongest head-to-head comparisons in 2024–2026 share a common theme: they attempt to **standardize what counts** as an evaluation boundary (preprocessing included or excluded), and they report multiple metrics rather than accuracy alone.

NeuroBench (Nature Communications, 2025) is the most explicit attempt to formalize this. It defines algorithm and system tracks, stresses that **pre- and post-processing must be accounted for** (because event modalities often require nontrivial conversions), and standardizes metrics including execution rate, latency, power/energy, and operation counts—explicitly separating multiply-accumulate operations (MACs) from “accumulates” (ACs) to better reflect sparse/event-driven computation. citeturn18view0 NeuroBench also provides baselines spanning conventional hardware and neuromorphic systems, including Loihi 2 and Xylo (with measurement methodology details). citeturn18view0turn20view0

At the single-workload comparison level, NorthPole’s Science 2023 evaluation is among the most visible head-to-heads (though not strictly SNN vs ANN—it is *dense inference* on a brain-inspired architecture). It frames comparison using energy/space/time metrics and evaluates on models including ResNet-50 and YOLOv4 against conventional CPU/GPU-style architectures. citeturn14search0turn14search1 Because NorthPole is not a native spiking processor, one should treat such results as *architectural competition* rather than “SNN algorithm superiority.”

Within “SNN/hybrid vs ANN” comparisons on event-based workloads, Ahmed et al. (CVPR 2025) provide an unusually strong package: they compare mAP against ANN/RNN baselines, provide MAC/AC analyses, report energy-per-inference style estimates in mJ for several model families, and demonstrate on-hardware execution of the SNN block on Loihi 2 with measured power/time. citeturn12view0turn11view0 This paper also illustrates a recurring conclusion: pure SNN backbones often lag in accuracy, while hybrids can approach ANN-like performance with a more credible efficiency argument when the spiking block is actually mapped to neuromorphic hardware. citeturn11view0

Aydin et al. (CVPRW 2024) similarly report power–performance tradeoffs for a hybrid pose estimation pipeline (large reported power savings with modest performance cost). citeturn23search0turn23search3

A key contested point across the literature is **how to translate operation counts into energy claims**. Many papers still present “theoretical energy” as a function of timesteps and spike rate, while NeuroBench argues that meaningful progress requires reproducible system-level measurement that includes conversion/feature extraction and I/O. citeturn18view0turn17view0

## Outlook and open problems

The near-term trajectory in the 3–5 year horizon (from a 2026 vantage point) is moderately optimistic but conditional. A prominent perspective by Muir & Sheik (2025) argues that neuromorphic technology is positioned for commercial uptake primarily in **battery-powered systems, IoT devices, and wearables**, and that progress depends on solving two key problems: **how to program general neuromorphic applications** and **how to deploy at scale**. citeturn17view0 This outlook is aligned with NeuroBench’s emphasis on end-to-end system benchmarking: real adoption requires not just fast neuron updates, but robust pipelines, measurement practice, and clear value at an application boundary. citeturn18view0

Fundamental open problems remain, and multiple sources agree they are not merely “engineering polish”:

Long-horizon temporal credit assignment remains central. BrainTrace (2026) treats the absence of an online learning system for rich spiking dynamics with low memory footprint as a major gap, and frames BPTT’s linear-in-sequence-length memory as a barrier to long-duration learning at scale. citeturn25view0 Even as deep SNNs improve on static tasks, long-horizon online learning is likely where “spiking-native” advantages would matter most (adaptive control, continual learning, streaming personalization)—and it remains challenging.

Dataset and benchmark fit is still imperfect. Surveys of learning rules and benchmarking frameworks emphasize that many popular datasets were not designed to stress the properties SNNs are supposed to exploit (asynchrony, precise timing, event sparsity); without better datasets, progress risks optimizing for ANN-like behavior with extra overhead. citeturn6search26turn18view0

The simulation–hardware gap is not solved. NIR highlights concrete, reproducible mismatches in timing conventions and numerical integration across backends; these are subtle but matter for deep temporal systems and for reproducible deployment. citeturn20view0

Tooling and deployment are improving quickly, but fragmentation remains. NIR’s explicit cross-backend scope (Loihi 2, SpiNNaker2, Xylo; multiple PyTorch-like simulators) suggests the ecosystem is converging toward compiler-like abstractions—but this is still early compared to mature GPU stacks. citeturn20view0turn17view0

As for likely “first adoption” domains (a reasoned projection, not a certainty): always-on sensing and event-driven perception (audio keyword spotting, simple vision triggers, industrial monitoring, low-latency event-based detection) remain the most plausible early adopters because their data is sparse, temporal, and they reward energy proportionality. citeturn17view0turn18view0turn12view0

Several 2024–2026 results look like credible inflection points (while noting that “inflection” is sometimes debated):

Hala Point (2024) is a scale milestone for Loihi 2-based neuromorphic systems, potentially enabling a new class of large-scale experimental work on mapping and distributed event-driven computing. citeturn8search0turn8search11

NIR (2024) is a concrete step toward a unifying “compiler layer” for neuromorphic backends, addressing a foundational deployment obstacle. citeturn20view0

ImageNet-level SNN results with very small timesteps (e.g., SpikingResformer CVPR 2024; Spikformer V2 arXiv 2024) show that accuracy gaps can narrow dramatically—though the extent to which that translates to real hardware efficiency depends on sparsity and end-to-end pipeline cost. citeturn7search2turn7search0turn23search2

NeuroBench (2025) may be a meta-inflection point: it changes *what the field measures*, potentially improving comparability and reproducibility. citeturn18view0

BrainTrace (2026) is a notable attempt to break the long-horizon online learning bottleneck with a model-agnostic, linear-memory system plus compiler automation—exactly the type of “missing infrastructure” many researchers have pointed to. citeturn25view0

## Key takeaways

- SNNs in 2024–2026 are increasingly *trainable at scale* (surrogate-gradient + deep-learning-style engineering), but remain best-positioned as a specialized tool rather than a general ANN replacement. citeturn17view0turn4search4  
- The strongest practical successes cluster around **event-based sensing and low-latency streaming**, where hybrids (SNN front-end + ANN/RNN back-end) often deliver the best accuracy–efficiency compromise, including on-chip demonstrations on Loihi 2. citeturn12view0turn23search3  
- ImageNet-scale SNN architectures reached **high accuracy with few timesteps** (e.g., SpikingResformer 79.4% @ 4 steps), narrowing the historical accuracy gap—yet many “energy” claims are still dominated by proxies rather than measured end-to-end system data. citeturn7search2turn18view0  
- The community increasingly treats **benchmarking methodology** as a first-class research problem: NeuroBench formalizes metrics and insists on measuring preprocessing and full system power/latency, not just neuron-core computation. citeturn18view0  
- Deployment friction is decreasing via **toolchain unification efforts** (e.g., NIR spanning multiple simulators and hardware backends), but the simulation–hardware gap and backend-specific quirks remain nontrivial. citeturn20view0  
- Long-horizon learning and continual adaptation remain fundamental bottlenecks; BrainTrace (2026) is a notable step toward linear-memory online learning with compiler support, potentially enabling more realistic streaming/continual SNN training. citeturn25view0  
- Near-term adoption is most plausible in **battery-powered, always-on edge systems** and sparse temporal workloads—provided programming models, end-to-end pipelines, and reproducible measurements continue to mature. citeturn17view0turn18view0