# ner_kram

## ToDo

### Questions

- Which HPO method do we trust/like?
  - Why? Why not the others?
- Which parameters do we vary?
  - Based on what?
- Do we see patterns in good/bad hyperparameter combinations?
- How do we continue when we do not do finetuning but pretraining?

### NER HPO Experiments

- HPO with ASHA / BOHB / PBT
  - original tags
    - cased
    - uncased
    - uncased-cased-mix
  - simple tags
    - cased
    - uncased
    - uncased-cased-mix
  - evaluate each model on for their respective tag family
    - cased
    - uncased
    - uncased-cased-mix
    - uncased-cased-both
    - ne-lower
    - ne-lower-cased-mix
    - ne-lower-cased-both
  - look at what hyperparameters and combination lead to good results

#### HPO Method

**Motivation for ASHA**: Authors demonstrate slight advantage over BOHB, even commonly on par with PBT. Huggingface provides an example for HPO with ray tune and BO + ASHA (https://huggingface.co/blog/ray-tune). Furthermore, ASHA is used in [https://arxiv.org/pdf/2106.09204.pdf] and seems to be a practical solution due to the native support of a grace period parameter. Potentially, PBT would provide a better performing model at the cost of having to rely on hyperparameter schedules and depressingly long HPO runs with several TB of disk usage. Perhaps we could overcome some of these issues by tinkering with the PBT parameters, but for now it does not seem to be worth the pain.

BO (TPE) + ASHA, 27 trials with a grace period of 1 epoch with the follow hyperparameter space, inspired by [https://arxiv.org/pdf/2106.09204.pdf]:

```
learning_rate: ~U(6e-6, 5e-6)
weight_decay: ~U(0.0, 0.2)
warmup_ratio: ~U(0.0, 0.12)
attention_probs_dropout_prob: ~U(0, 0.2)
hidden_dropout_prob: ~U(0, 0.2)
per_device_train_batch_size: ~Choice([16, 32, 64])
```

#### Results

##### Temporary Results For Reference

Trained & evaluated on uncased with default hyperparameters:

|         | BS=16  | BS=32  | BS=64      | BS=128 | BS=256 |
|---------|--------|--------|------------|--------|--------|
| F1-Dev  | 0.8651 | 0.8676 | **0.8683** | 0.8635 | 0.8572 |
| F1-Test | 0.8668 | 0.8663 | **0.867**  | 0.8624 | 0.8556 |

PBT, 20 trials, perturbation interval 1, uncased training & evaluation:<br>
F1-Dev=0.8684 (does not look very promising with the current settings as it took ~13h with 4 gpus)

##### Development

| Tag Family | Trained on        | HPO Alg | cased  | uncased | uncased-cased-mix | uncased-cased-both | ne-lower | ne-lower-cased-mix | ne-lower-cased-both  |
| ---------- | ----------------- | ------- | -----  | ------- | ----------------- | ------------------ | -------- | ------------------ | -------------------- |
| Original   | cased             | BOHB    | -      | -       | -                 | -                  | -        | -                  | -                    |
| Original   | uncased           | ASHA    | 0.77931| 0.87012 | 0.82533           | 0.82621            | 0.87044  | 0.82607            | 0.82640              |
| Original   | uncased-cased-mix | BOHB    | -      | -       | -                 | -                  | -        | -                  | -                    |
| Simple     | cased             | BOHB    | -      | -       | -                 | -                  | -        | -                  | -                    |
| Simple     | uncased           | BOHB    | -      | -       | -                 | -                  | -        | -                  | -                    |

##### Test

| Tag Family | Trained on        | HPO Alg | cased  | uncased | uncased-cased-mix | uncased-cased-both | ne-lower | ne-lower-cased-mix | ne-lower-cased-both  |
| ---------- | ----------------- | ------- | -----  | ------- | ----------------- | ------------------ | -------- | ------------------ | -------------------- |
| Original   | cased             | BOHB    | -      | -       | -                 | -                  | -        | -                  | -                    |
| Original   | uncased           | ASHA    | 0.78353| 0.86388 | 0.82469           | 0.82493            | 0.86206  | 0.82550            | 0.82406              |
| Original   | uncased-cased-mix | BOHB    | -      | -       | -                 | -                  | -        | -                  | -                    |
| Simple     | cased             | BOHB    | -      | -       | -                 | -                  | -        | -                  | -                    |
| Simple     | uncased           | BOHB    | -      | -       | -                 | -                  | -        | -                  | -                    |
| Simple     | uncased-cased-mix | BOHB    | -      | -       | -                 | -                  | -        | -                  | -                    |
| Simple     | uncased-cased-mix | BOHB    | -      | -       | -                 | -                  | -        | -                  | -                    |

#### Successful Hyperparameters

| Tag Family | Trained on        | HPO Alg | learning rate | weight decay | warmup ratio | attention dropout | hidden dropout | batch size | ??? |
| ---------- | ----------------- | ------- | ------------- | ------------ | ------------ | ----------------- | -------------- | ---------- | --- |
| Original   | cased             | BOHB    | -             | -            | -            | -                 | -              | -          | -   |
| Original   | uncased           | ASHA    | 2.326071e-05  | 0.144090228  | 0.066440712  | 0.148347450       | 0.140120178    | 16         | -   |
| Original   | uncased-cased-mix | BOHB    | -             | -            | -            | -                 | -              | -          | -   |
| Simple     | cased             | BOHB    | -             | -            | -            | -                 | -              | -          | -   |
| Simple     | uncased           | BOHB    | -             | -            | -            | -                 | -              | -          | -   |
| Simple     | uncased-cased-mix | BOHB    | -             | -            | -            | -                 | -              | -          | -   |

### Potentially Interesting Papers

- [Efficient Deep Learning: A Survey on Making Deep Learning Models Smaller, Faster, and Better](https://arxiv.org/abs/2106.08962)
- [An Empirical Study on Hyperparameter Optimization for Fine-Tuning Pre-trained Language Models](https://arxiv.org/abs/2106.09204)
- [Hyper-Parameter Optimization: A Review of Algorithms and Applications](https://arxiv.org/abs/2003.05689)
- [A disciplined approach to neural network hyper-parameters: Part 1 -- learning rate, batch size, momentum, and weight decay](https://arxiv.org/abs/1803.09820)
- [BOHB: Robust and Efficient Hyperparameter Optimization at Scale](http://proceedings.mlr.press/v80/falkner18a.html)
- [Population Based Training of Neural Networks](https://arxiv.org/abs/1711.09846)
- [A System for Massively Parallel Hyperparameter Tuning](https://arxiv.org/abs/1810.05934)
- [Deepspeed: Curriculum Learning](https://www.deepspeed.ai/tutorials/curriculum-learning/)
