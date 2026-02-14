# SigLIP 2 Base Model

**SigLIP 2** is Google DeepMind’s 2025 release of multilingual vision–language encoders that build on the original SigLIP framework. Where CLIP relies solely on contrastive softmax loss, SigLIP 2 blends several objectives to learn richer representations:

- **Captioning‑based pretraining:** The model uses a decoder to generate captions from images, providing supervised signal beyond contrastive alignment.
    
- **Self‑distillation and masked prediction:** During training, the model predicts the embeddings of its own previous iterations and reconstructs masked image patches. These self‑supervised objectives improve the quality of local features and enhance tasks like segmentation and depth estimation.
    
- **Sigmoid contrastive loss:** SigLIP 2 replaces the softmax cross‑entropy of CLIP with a **sigmoid‑based loss**. This alternative objective balances global and local feature learning and allows the model to learn robust similarities without requiring large batch sizes.
    

The training corpus contains primarily English captions with a smaller portion of non‑English captions; de‑biasing techniques reduce undesirable correlations. The model architecture remains compatible with Vision Transformers, and the **NaFlex** variant processes images at varying resolutions using a single checkpoint, preserving native aspect ratios.

_Diagram: Distillation setup for SigLIP 2. A larger teacher (SigLIP 2 SO400M) guides the student (SigLIP 2 Base) using a combination of contrastive and distillation losses._

## Why SigLIP 2?

SigLIP 2 delivers significant improvements over its predecessor and other open‑weight vision–language models:

- **Improved zero‑shot and retrieval performance:** Benchmarks such as ImageNet, ObjectNet and Crossmodal‑3600 show that SigLIP 2 consistently outperforms earlier models. The inclusion of captioning, self‑distillation and masked prediction yields a better balance between semantic understanding and localization.
    
- **Better dense features:** Self‑distillation encourages the model to learn fine‑grained spatial details, which benefits segmentation, depth estimation and open‑vocabulary detection.
    
- **Multilingual and fair:** The training data covers multiple languages and employs de‑biasing methods, reducing unfair object‑to‑gender associations.
    

## Distillation Strategy

For knowledge distillation (KD), a larger **teacher** model transfers its knowledge to a smaller **student**. In the SigLIP family:

- **Teacher:** SigLIP 2 SO400M or SigLIP 2 Giant (≈400 M–1 B parameters), trained with the same objectives. These models capture richer semantics and can supervise the student.
    
- **Student:** SigLIP 2 Base (≈86 M parameters), which seeks to match the teacher’s embeddings while maintaining a small footprint.
    

Training uses a weighted sum of the SigLIP contrastive loss (between student image and text embeddings) and the distillation loss (e.g., cosine similarity loss between student and teacher embeddings). The result is a compact model that inherits the teacher’s performance but is faster and easier to deploy on resource‑constrained devices.