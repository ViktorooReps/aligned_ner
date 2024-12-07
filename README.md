# Quick explanation
![Exécution_de_Marie_Antoinette_le_16_octobre_1793_cropped](https://github.com/user-attachments/assets/7fce97dd-4208-480c-9a16-15463ecf4167)

The fine-tuning procedure introduced in the infamous BERT paper [Devlin et al.](https://aclanthology.org/N19-1423.pdf) has become a de-facto standard in sequence tagging 
(NER, PoS, etc) using pretrained Transformers. This repository aims to challenge the status quo by providing the evidence that Masked Language Modelling (MLM) produces 
a suboptimal non-mask token representations in the final Transformer layer.

The solution is elegant and time tested: the strategical removal of ~~the head~~ the last Transformer block.
