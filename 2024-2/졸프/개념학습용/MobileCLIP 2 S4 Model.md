# MobileCLIP 2 S4 Model

**MobileCLIP 2** is an evolution of Apple’s MobileCLIP series, designed to bring CLIP‑level performance to **edge and mobile devices**. While the original MobileCLIP achieved 3–15 ms latency with 50–150 M parameters, MobileCLIP 2 improves both the training recipe and the resulting models.

## Key Innovations in MobileCLIP 2

1. **Better CLIP teacher ensemble:** The authors train stronger teacher models on the **DFN dataset**, a large curated collection of high‑quality image–text pairs. Distilling from these teachers yields more accurate students.
    
2. **Improved captioner teachers:** Captioner models trained on DFN and fine‑tuned on diverse high‑quality datasets generate synthetic captions to reinforce text supervision. Combining captions from multiple models improves diversity and robustness.
    
3. **Training insights:** Ablation studies reveal that tuning the temperature parameter in the contrastive loss and carefully weighting synthetic captions lead to significant gains. New models are trained with these settings to form **MobileCLIP 2**.
    
4. **Performance gains:** MobileCLIP 2 B achieves a **2.2 percentage point improvement** in ImageNet‑1k zero‑shot accuracy over the original MobileCLIP B. The **S4** variant matches the zero‑shot accuracy of SigLIP‑SO400M/14 while being **2× smaller** and delivers higher accuracy than DFN ViT‑L/14 at **2.5× lower latency**.
    

_Diagram: MobileCLIP 2 S4 uses two teacher models—a large SigLIP teacher and a DFN ViT‑L/14 CLIP teacher. The student learns from both image and caption guidance._

## Distillation Setup

MobileCLIP 2 S4 leverages a **dual‑teacher distillation** framework:

- **Teacher 1:** A powerful SigLIP‑SO400M or Meta CLIP 2 (H/14) model trained on multilingual and high‑quality data. This teacher provides rich semantic embeddings.
    
- **Teacher 2:** A DFN‑trained ViT‑L/14 CLIP model, offering complementary features and robust contrastive supervision.
    
- **Student:** A small ViT‑S4 backbone with ~67 M parameters and efficient convolutional/attention blocks for low latency.
    

During training, the student minimizes a weighted sum of:

1. **Contrastive loss** between the student’s image and text embeddings (as in CLIP).
    
2. **Distillation losses** from each teacher (e.g., cosine or mean squared error) to match their embeddings.
    
3. **Caption losses** from captioner teachers, encouraging the student to align with synthetic captions.
    

This combination allows the student to inherit the teachers’ zero‑shot capabilities while remaining fast enough for deployment on mobile devices. The DFN dataset and improved captioners help the model generalize to diverse visual concepts without manual annotation.