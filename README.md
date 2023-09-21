# Fast inference from transformers via speculative decoding

This repository implements speculative sampling for large language model (LLM) decoding. It utilizes two models during the decoding process: a target model and an approximation model. The approximation model is a smaller model, while the target model is a larger one. The approximation model generates token guesses, and the target model corrects these guesses. This approach allows for decoding by running the target model in parallel on the outputs of the approximation models, resulting in improved efficiency compared to decoding with the target model alone.

The speculative sampling is proposed by Google and Deepmind independently. So I implement two slightly different versions of speculative sampling: [Google's](https://arxiv.org/abs/2211.17192) and [Deepmind's](https://arxiv.org/abs/2302.01318).

## Update Logs

- 2023.09.21: Add serving features. Support more models, i.e. llama-7B and llama-1B.

- 2023.09.19: Add KV Cache Optimization to the Google's version.

- 2023.08.16: First release, implement the paper's algorithm. Support Bloom-560M and Bloomz-7B1.

## Usage
### Inference
You need prepare a pair of models using the same embedding and vocabulary. The approximation model should be smaller than the target model. Here are some
tested model pairs.

<center>

| Approx Model | Target Model |
|--------------|--------------|
| [bloomz-7b1](https://huggingface.co/bigscience/bloomz-7b1/tree/main) | [bloom-560m](https://huggingface.co/bigscience/bloom-560m/tree/main) |
| [TinyLlama-1.1B](https://huggingface.co/PY007/TinyLlama-1.1B-step-50K-105b) | llama-7b |

</center>

In the sample, I use [bloomz-7b1](https://huggingface.co/bigscience/bloomz-7b1/tree/main) as the target model, [bloom-560m](https://huggingface.co/bigscience/bloom-560m/tree/main) as the approximation model.

```bash
python main.py \
    --input "The quick brown fox jumps over the lazy " \
    --target_model_name bigscience/bloomz-7b1 \
    --approx_model_name bigscience/bloom-560m
```

You can also use `--v` args to see a token is generated by which model.

![example image](./imgs/sps.jpg "console output")

### Serving
Start an inference server.
```bash
python serving.py
```

Test the serving with curl:
```bash
curl -X POST -H "Content-Type: application/json" -d '{"prompt": "Who is the president of the USA"}' http://127.0.0.1:5000/predict
```
## References
```
@inproceedings{leviathan2023fast,
  title={Fast inference from transformers via speculative decoding},
  author={Leviathan, Yaniv and Kalman, Matan and Matias, Yossi},
  booktitle={International Conference on Machine Learning},
  pages={19274--19286},
  year={2023},
  organization={PMLR}
}

@article{chen2023accelerating,
  title={Accelerating large language model decoding with speculative sampling},
  author={Chen, Charlie and Borgeaud, Sebastian and Irving, Geoffrey and Lespiau, Jean-Baptiste and Sifre, Laurent and Jumper, John},
  journal={arXiv preprint arXiv:2302.01318},
  year={2023}
}
```

## Notations
This repo is built for demostration purpose. Other optimizations, such as batching and parallelism, are not included which are essential for efficiency.