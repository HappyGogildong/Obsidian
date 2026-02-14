# Contrastive Language–Image Pre‑training (CLIP)

OpenAI’s **CLIP** (Contrastive Language–Image Pre‑training) is a seminal framework that aligns images and natural‑language descriptions in a shared embedding space. Unlike classical classifiers that learn a fixed set of labels, CLIP learns to associate any image with free‑form text prompts by jointly training two encoders:

- **Image encoder:** Usually a ResNet or Vision Transformer (ViT) that processes an image into a fixed‑length feature vector. The image is divided into patches, embedded and passed through multiple transformer layers. The final representation is ℓ2‑normalized so that cosine similarity measures angular distance.
    
- **Text encoder:** A transformer model that tokenizes a sentence and outputs a contextualized embedding. Like the image branch, the output is normalized and projected into the common latent space.
    

_Diagram: CLIP uses separate image and text encoders to map inputs into a shared embedding space, where similarity is measured via cosine distance._

## Training Objective

CLIP is pre‑trained on **hundreds of millions of (image, caption) pairs** scraped from the internet. During training:

1. A minibatch contains NNN image–text pairs. Both encoders produce an embedding for each image ziimgz_i^{\text{img}}ziimg​ and caption zitextz_i^{\text{text}}zitext​.
    
2. Cosine similarities sij=⟨ziimg,zjtext⟩/(∥ziimg∥ ∥zjtext∥)s_{ij} = \langle z_i^{\text{img}}, z_j^{\text{text}}\rangle / (\|z_i^{\text{img}}\|\,\|z_j^{\text{text}}\|)sij​=⟨ziimg​,zjtext​⟩/(∥ziimg​∥∥zjtext​∥) are computed for all pairs.
    
3. A **contrastive loss** encourages the similarity for matched pairs (i=j)(i=j)(i=j) to be high and those for mismatched pairs (i≠j)(i \ne j)(i=j) to be low. In practice, InfoNCE or cross‑entropy loss is applied across rows and columns of the similarity matrix.
    

Because the model never sees class labels, it learns general visual–language concepts. At inference time, CLIP performs **zero‑shot classification** by comparing an image’s embedding to embeddings of textual class descriptions (e.g., “a photo of a cat”, “a photo of a dog”) and choosing the highest‑similarity label.

## Impact and Extensions

CLIP demonstrates that large‑scale pairing of web images and captions can replace manual labeling. Its flexibility spurred numerous follow‑ups:

- **OpenCLIP** and **EVA‑CLIP:** Open‑sourced reproductions trained on LAION and COYO datasets that scale up parameters and training data.
    
- **SigLIP:** Replaces the softmax contrastive loss with a sigmoid‑based loss and introduces captioning and masked‑image objectives.
    
- **MobileCLIP:** Distills CLIP knowledge into lightweight models suitable for mobile devices.
    

These models preserve the **dual‑encoder** architecture but explore different training losses, data curation strategies, and student–teacher distillation to push zero‑shot and retrieval performance further.