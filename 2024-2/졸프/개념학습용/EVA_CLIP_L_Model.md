# EVA‑02‑CLIP‑L Model

The **EVA‑CLIP** project from BAAI introduces a family of open‑source contrastive vision–language models that improve upon CLIP through more efficient training and larger scale. The **EVA‑02‑CLIP‑L** variant uses a **ViT‑L/14** backbone with approximately **428 million parameters** and achieves state‑of‑the‑art zero‑shot performance while remaining manageable for research and deployment.

## Training Recipe

1. **Massive data:** EVA‑CLIP models are trained on **Merged‑2B**, a dataset combining **1.6 billion samples from LAION‑2B** with **0.4 billion samples from COYO‑700M**. This diverse corpus provides broad coverage of visual concepts.
    
2. **Improved optimization:** The authors introduce techniques such as better masked image modeling (MIM) teachers, efficient scaling of batch sizes and learning rates, and customized regularization to train large models effectively. These changes allow the models to match or exceed OpenAI CLIP with fewer samples and compute.
    
3. **Contrastive and MIM supervision:** A masked image modeling teacher guides the image encoder, while contrastive learning aligns image and text embeddings. The synergy between these objectives yields robust representations.
    

## Performance and Scale

EVA‑CLIP is notable for its scalability: the **EVA‑02‑CLIP‑L** model achieves **79.8 % zero‑shot top‑1 accuracy on ImageNet‑1K** with 428 M parameters. Larger variants such as **EVA‑02‑CLIP‑E** and **EVA‑02‑CLIP‑E+** with 4.7 B and 5.0 B parameters push this figure to **81.9 % and 82.0 %** respectively.

The project notes that **EVA‑CLIP series are the most performant open‑sourced CLIP models at all scales**, especially on mainstream classification benchmarks like ImageNet. This makes them attractive as teachers or strong baselines for retrieval and zero‑shot tasks.

_Diagram: Distillation for EVA‑CLIP L. A large teacher (EVA‑CLIP E or CLIP‑ViT‑bigG) supervises the EVA‑CLIP L student to transfer knowledge._

## Distillation Strategy

While the EVA‑02‑CLIP‑L model can be trained from scratch, knowledge distillation (KD) helps reduce training cost and stabilize the student. Recommended **teacher models** include:

- **EVA‑02‑CLIP‑E (≈4.7 B parameters):** Trained on the same dataset, this model attains 81.9 % ImageNet zero‑shot accuracy and provides richer features to supervise the L variant.
    
- **CLIP‑ViT‑bigG or CLIP‑ViT‑H/14 (LAION‑trained):** These large OpenCLIP models offer complementary representations, and can serve as external teachers.
    

The distillation objective combines the standard CLIP contrastive loss with a matching loss (e.g., cosine or mean squared error) between the student and teacher embeddings. By aligning its representations with those of a larger teacher, **EVA‑02‑CLIP‑L** can approach or surpass its teacher’s performance while retaining a much smaller footprint.