---
layout: page
title: Textual Inversion with Captioning Initialization
description: An attempt to improve textual inversion (inverting concept in the image to text in context of text-to-image diffusion models, so that the concept can be re-contextualized with customized prompt) by using off-the-shelf captioning model. 
img: assets/textual_inversion_plus/teaser.png
importance: 1
category: generative modeling
related_publications: 
---


<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/textual_inversion_plus/teaser.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    <strong>Teaser.</strong> Inverting the input image(s) (left) to generate personalized outputs (right). With our method, the learned text embeddings from single image can do plausible personalization/re-contextualization, converging much faster than vanila TI.
</div>

# [Important!!!] Paper Writing
**For a more comprehensive treatment, I recommand the [paper/report](../../assets/textual_inversion_plus/an_image_is_worth_one_sentence.pdf) we've written, where we provide more backgrounds, intutions, and analysis.**

Nevertheless, I provide an overview below, assuming the knowledge of [vanila textual inversion](https://textual-inversion.github.io).

# Abstract
Text-driven image synthesis has emerged as a popular research area. Existing
approaches to invert image to text face challenges like requiring multiple images,
slow convergence, or overfitting thus suffering from editing capability. In this
paper, we propose a novel initialization method for inverting text using *off-the-shelf* (pre-trained)
classification or captioning models. This approach enables multi-token embedding
learning from a single input image while eliminating the need for fine-tuning
and ensuring faster convergence. We demonstrate a significant improvement in
convergence speed compared to vanilla TI.

# Method

Overall, we modify the original textual inversion by replacing the idea of 'capturing a concept with a word' to 'capturing a concept with a sentence', and proposed to initialize the inverted 'sentence' by off-the-shelf captioning model (classification model can also be used, and we have results related to it in the [report](../../assets/textual_inversion_plus/an_image_is_worth_one_sentence.pdf). In the following, we focus on the captioning initialization.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/textual_inversion_plus/labelled_diagram.pdf" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    <strong>Pipeline.</strong> (1) The input image is first classified/captioned, and the template containing placeholder tokens is generated to match captioning length. (2) The template is tokenized and converted to a matrix \( \mathbf{P} \), each row corresponds to a word embedding. The dotted lines show that each placeholder token is initialized to their corresponding output from the classification/captioning model. We only optimize parameters in the red region. (3) The matrix \( \mathbf{P} \) is converted into a vector by text-encoder \( \mathbf{c}_{\boldsymbol{\theta}}(.) \) then fed into the LDM generator. (4) Compute the LDM loss. The generated image and noised samples shown are for illustration, loss is computed in <em>latent space</em>.
</div>
 
### Single Image Setting 
Instead of using multiple captures of a concept (typically 3-5 images in original TI), we use the setting of single image/capture alone, as this is a more realistic setting for real-life use cases. We demonstrate that a single image suffices via some recontextualization examples in the [report](../../assets/textual_inversion_plus/an_image_is_worth_one_sentence.pdf). However, our main contribution is in the captioning initialization (w/ multi-token inversion).

### Captioning Initialization, and Mutli-token Inversion
 Instead of representing a concept using only one *single* "new word", i.e., a placeholder token `<s>` as in the vanilla TI, we allow constructing *multiple* placeholder tokens `<0>, <1>, ..., <n>`, and denote their corresponding optimizing embeddings as $$ \mathbf{v}^{(0)}, \mathbf{v}^{(1)}, \dots, \mathbf{v}^{(n)} $$. The specific number of tokens depends on the results of a captioning model as the initialization of word embeddings -- we will first fed the image into a captioning model to get a sequence of words.

# Results

For more comprehensive analysis like ablation study please refer to the [report](../../assets/textual_inversion_plus/an_image_is_worth_one_sentence.pdf). Below, we give the training loss convergence, in which we see that captioning converges the fastest out off three cases -- initialization by classification, captioning, or a single token `'*'`. We can observe that our captioning method converges the fastest out of the three. And note that there are many more metrics can be evaluated on generated recontextualized/reconstructed images, such as LPIPS -- please refer to the [report](../../assets/textual_inversion_plus/an_image_is_worth_one_sentence.pdf). 


<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/textual_inversion_plus/mse_convergence.pdf" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    <strong>Training Loss Convergence.</strong> The plot illustrates the training average mean squared error, i.e., the LDM loss over 12 chosen images. The red curve shows using caption as initialization, the green curve shows using object class as initialization, and blue curve shows `'*'` as initialization.
</div>

Below, we include some qualitive results of our method. These are generated images from prompts with learned embeddings. We use only *100 optimization steps on a single image* (v.s. vanilla textual inversion uses around 5000 steps on multiple iamges) and the results show promising personalization capabilities of our proposed method.
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/textual_inversion_plus/personalization.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    <strong>Some personalization results w/ our method.</strong> These are generated images from prompts with learned embeddings. We use only *100 optimization steps on a single image*
</div>


# Code

The implementation is simple. We just modify the part of TI's restricted single word embedding inversion to allowing mutliple word embeddings inversion in the [official Textual Inversion github repo](https://github.com/rinongal/textual_inversion). If you would like to know more details about how this is done feel free to [contact me](mailto:harry.qiu@mail.utoronto.ca) and I can send to the specific functions/parts to be replaced from the official repo -- we are maintaining the code for other extensions so would not release it yet.

