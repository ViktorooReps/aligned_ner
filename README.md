# Quick explanation
![Exécution_de_Marie_Antoinette_le_16_octobre_1793_cropped](https://github.com/user-attachments/assets/7fce97dd-4208-480c-9a16-15463ecf4167)

The fine-tuning procedure introduced in the infamous BERT paper [Devlin et al.](https://aclanthology.org/N19-1423.pdf) has become a de-facto standard in sequence tagging 
(NER, PoS, etc) using pretrained Transformers. This repository aims to challenge the status quo by providing the evidence that Masked Language Modelling (MLM) produces 
a suboptimal non-mask token representations in the final Transformer layer.

The solution is elegant and time tested: the strategical removal of ~~the head~~ the last Transformer block.

# The Issue with the Last Block

One can consider token representations in Transformer as information channels, one per token in the context. Most of the operations on the channels are done independently of the other channels: initialization with embeddings, positional information encoding, normalization, and MLP. The only exception is the attention mechanism, which can be viewed as a channel mixing procedure with weights assigned to each pair of channels.

This view makes it convenient to explore how gradients affect token representations at each layer of the Transformer network.

BERT is trained with MLM: at each training step, some of the tokens are masked (replaced with a special `<mask>` token), and the model is prompted to reproduce them. In practice, this means that the loss is computed only at the mask tokens.

During the backward pass, the gradients travel through the channels, the opposite way of the information flow. For MLM, the gradients start propagating only through channels associated with mask tokens. This leads to an absence of any mechanism for adjusting the weights that operate on channels past the last attention operation – the gradients simply cannot reach them.

# Why Conventional Fine-Tuning is Flawed

The goal of any fine-tuning procedure is to utilize the knowledge encoded in model weights to solve a downstream task. This is done by "aligning" the weights with the task-specific inputs and outputs, minimally changing them in the process using a small learning rate and a minimal number of steps. Ideally, one would not need to add any new components to the network, as these would need to be trained from scratch, potentially requiring a larger learning rate and more steps to shift them from their initial state to one that is "synchronized" with the rest of the network. Additionally, if these new components do not see the full spectrum of possible inputs from the pretrained parts of the network during training, there is a considerable risk of performance degradation due to domain shifts. 

Conventional fine-tuning for sequence tagging tasks involves the addition of a linear layer on top of the pretrained language model (LM). This added layer projects the hidden state of the model to label scores for **each** token. 

As described in detail before, if the aformentioned LM was trained with MLM, the last Transformer block **is not trained to process the information channels, associated with non-mask tokens**. Practically, it means that one has to train from scratch not only a linear layer, but the last Transformer block as well.

# Empirical Evidence

## NER Knowledge Transfer across Languages

The goal of the experiment is to find evidence that the removal of the last Transformer block leads to better initial state of the network.

1. Multiple training configurations are tested, for each configuration two models are fine-tuned, `l12` and `l11`, the latter has the last Transformer block removed.
2. Multilingual [XML-V](https://aclanthology.org/2023.emnlp-main.813/) is fine-tuned on [CoNLL 2003](https://aclanthology.org/W03-0419/) and tested on [Afrikaans NER](https://repo.sadilar.org/handle/20.500.12185/299) dataset.
3. The results for multiple learning rates are reported. The training steps are adjusted so both models with the same learning rate can converge.
<img width="960" alt="ner_knowledge_transfer" src="https://github.com/user-attachments/assets/9259b66a-1b40-43ee-820d-5049336e9ece">

Given the same training configuration, `l11` consistently outperforms `l12`, although the effect is decreased with the learning rate increase. The latter may be the evidence of `l12` requiring higher learning rate due to the need to adjust more weights. Since these two networks are the same, except `l11` has the last Transformer block removed, this leads to the conclusion that the last block is suboptimally initialized.  

The effect is especially evident on extremely low learning rates: `5e-6` and `1e-6`:
<img width="960" alt="low_lr_conll03" src="https://github.com/user-attachments/assets/7692be16-f350-47aa-9ad4-2f89ee7c0483">
The `l12` model takes a lot longer to converge, and in case of `1e-6` learning rate, converges to a significantly lower F1 score.

# Contribution

Any contribution is highly encouraged! Have a look at [Discussions](https://github.com/ViktorooReps/vive_la_ner/discussions) tab to suggest new experiments, critique the results or propose help.

# Goal

* Improve the understanding of fine-tuning process and domain shift challenges.
* Establish and popularize a better simple fine-tuning procedure.

## Citation

If you use this repository in your research or projects, please cite it using the following format:
```bibtex
@misc{shcherbakov_vive_la_ner_2024,
  title = {Vive la NER},
  author = {Viktor Shcherbakov},
  year = {2024},
  publisher = {GitHub},
  journal = {GitHub repository},
  howpublished = {\url{https://github.com/ViktorooReps/your-repository}},
  version = {v1.0}
}
```
