### البناء
يتكون من الأجزاء التالية:
1. **اكتشاف نشاط الصوت (VAD)**
2. **تحويل الكلام إلى نص (STT)**
3. **نموذج اللغة (LM)**
4. **تحويل النص إلى كلام (TTS)**

### الوحدات النمطية
يركز خط الأنابيب على Transformers على مركز Hugging Face.

**VAD** 
- [Silero VAD v5](https://github.com/snakers4/silero-vad)

**STT**
- Any [Whisper](https://huggingface.co/docs/transformers/en/model_doc/whisper) نموذج على Hugging Face Hub من خلال Transformers 
بما في ذلك [whisper-large-v3](https://huggingface.co/openai/whisper-large-v3) and [distil-large-v3](https://huggingface.co/distil-whisper/distil-large-v3)
- [Lightning Whisper MLX](https://github.com/mustafaaljadery/lightning-whisper-mlx?tab=readme-ov-file#lightning-whisper-mlx)
- [Paraformer - FunASR](https://github.com/modelscope/FunASR)

**LLM**
- Any instruction-following model on the [Hugging Face Hub](https://huggingface.co/models?pipeline_tag=text-generation&sort=trending) via Transformers 🤗
- [mlx-lm](https://github.com/ml-explore/mlx-examples/blob/main/llms/README.md)
- [OpenAI API](https://platform.openai.com/docs/quickstart)

**TTS**
- [Parler-TTS](https://github.com/huggingface/parler-tts) 🤗
- [MeloTTS](https://github.com/myshell-ai/MeloTTS)
- [ChatTTS](https://github.com/2noise/ChatTTS?tab=readme-ov-file)

## الاعداد

استنساخ المستودع:
```bash
git clone https://github.com/huggingface/speech-to-speech.git
cd speech-to-speech
```

تثبيت التبعيات المطلوبة باستخدام [uv](https://github.com/astral-sh/uv):
```bash
uv pip install -r requirements.txt
```

بالنسبة لمستخدمي Mac، استخدم
`requirements_mac.txt`  
```bash
uv pip install -r requirements_mac.txt
```

إذا اردت استخدام Melo TTS، فق بتشغيل:
```bash
python -m unidic download
```


## ## الاستخدام

يمكن تشغيل خط الأنابيب بطريقتين:
- **نهج الخادم/العميل**: يتم تشغيل النماذج على الخادم، ويتم بث الإدخال/الإخراج الصوتي من العميل.
- **النهج المحلي**: يتم التشغيل محليًا.

### الإعداد الموصى به

### نهج الخادم/العميل

1. قم بتشغيل خط الأنابيب على الخادم:
   ```bash
   python s2s_pipeline.py --recv_host 0.0.0.0 --send_host 0.0.0.0
   ```

2. قم بتشغيل العميل محليًا للتعامل مع إدخال الميكروفون واستقبال الصوت الناتج:
   ```bash
   python listen_and_play.py --host <IP address of your server>
   ```

### النهج المحلي (Mac)

1. للحصول على الإعدادات المثالية على Mac:
   ```bash
   python s2s_pipeline.py --local_mac_optimal_settings
   ```

This setting:
   - Adds `--device mps` to use MPS for all models.
     - Sets LightningWhisperMLX for STT
     - Sets MLX LM for language model
     - Sets MeloTTS for TTS

### Docker Server

#### Install the NVIDIA Container Toolkit

https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html

#### Start the docker container
```docker compose up```



### Recommended usage with Cuda

Leverage Torch Compile for Whisper and Parler-TTS. **The usage of Parler-TTS allows for audio output streaming, further reducing the overall latency** 🚀:

```bash
python s2s_pipeline.py \
	--lm_model_name microsoft/Phi-3-mini-4k-instruct \
	--stt_compile_mode reduce-overhead \
	--tts_compile_mode default \
  --recv_host 0.0.0.0 \
	--send_host 0.0.0.0 
```

For the moment, modes capturing CUDA Graphs are not compatible with streaming Parler-TTS (`reduce-overhead`, `max-autotune`).

### Multi-language Support

The pipeline currently supports English, French, Spanish, Chinese, Japanese, and Korean.  
Two use cases are considered:

- **Single-language conversation**: Enforce the language setting using the `--language` flag, specifying the target language code (default is 'en').
- **Language switching**: Set `--language` to 'auto'. In this case, Whisper detects the language for each spoken prompt, and the LLM is prompted with "`Please reply to my message in ...`" to ensure the response is in the detected language.

Please note that you must use STT and LLM checkpoints compatible with the target language(s). For the STT part, Parler-TTS is not yet multilingual (though that feature is coming soon! 🤗). In the meantime, you should use Melo (which supports English, French, Spanish, Chinese, Japanese, and Korean) or Chat-TTS.

#### With the server version:

For automatic language detection:

```bash
python s2s_pipeline.py \
    --stt_model_name large-v3 \
    --language auto \
    --mlx_lm_model_name mlx-community/Meta-Llama-3.1-8B-Instruct \
```

Or for one language in particular, chinese in this example

```bash
python s2s_pipeline.py \
    --stt_model_name large-v3 \
    --language zh \
    --mlx_lm_model_name mlx-community/Meta-Llama-3.1-8B-Instruct \
```

#### Local Mac Setup

For automatic language detection:

```bash
python s2s_pipeline.py \
    --local_mac_optimal_settings \
    --device mps \
    --stt_model_name large-v3 \
    --language auto \
    --mlx_lm_model_name mlx-community/Meta-Llama-3.1-8B-Instruct-4bit \
```

Or for one language in particular, chinese in this example

```bash
python s2s_pipeline.py \
    --local_mac_optimal_settings \
    --device mps \
    --stt_model_name large-v3 \
    --language zh \
    --mlx_lm_model_name mlx-community/Meta-Llama-3.1-8B-Instruct-4bit \
```

## Command-line Usage

> **_NOTE:_** References for all the CLI arguments can be found directly in the [arguments classes](https://github.com/huggingface/speech-to-speech/tree/d5e460721e578fef286c7b64e68ad6a57a25cf1b/arguments_classes) or by running `python s2s_pipeline.py -h`.

### Module level Parameters 
See [ModuleArguments](https://github.com/huggingface/speech-to-speech/blob/d5e460721e578fef286c7b64e68ad6a57a25cf1b/arguments_classes/module_arguments.py) class. Allows to set:
- a common `--device` (if one wants each part to run on the same device)
- `--mode` `local` or `server`
- chosen STT implementation 
- chosen LM implementation
- chose TTS implementation
- logging level

### VAD parameters
See [VADHandlerArguments](https://github.com/huggingface/speech-to-speech/blob/d5e460721e578fef286c7b64e68ad6a57a25cf1b/arguments_classes/vad_arguments.py) class. Notably:
- `--thresh`: Threshold value to trigger voice activity detection.
- `--min_speech_ms`: Minimum duration of detected voice activity to be considered speech.
- `--min_silence_ms`: Minimum length of silence intervals for segmenting speech, balancing sentence cutting and latency reduction.


### STT, LM and TTS parameters

`model_name`, `torch_dtype`, and `device` are exposed for each implementation of the Speech to Text, Language Model, and Text to Speech. Specify the targeted pipeline part with the corresponding prefix (e.g. `stt`, `lm` or `tts`, check the implementations' [arguments classes](https://github.com/huggingface/speech-to-speech/tree/d5e460721e578fef286c7b64e68ad6a57a25cf1b/arguments_classes) for more details).

For example:
```bash
--lm_model_name google/gemma-2b-it
```

### Generation parameters

Other generation parameters of the model's generate method can be set using the part's prefix + `_gen_`, e.g., `--stt_gen_max_new_tokens 128`. These parameters can be added to the pipeline part's arguments class if not already exposed.

## Citations

### Silero VAD
```bibtex
@misc{Silero VAD,
  author = {Silero Team},
  title = {Silero VAD: pre-trained enterprise-grade Voice Activity Detector (VAD), Number Detector and Language Classifier},
  year = {2021},
  publisher = {GitHub},
  journal = {GitHub repository},
  howpublished = {\url{https://github.com/snakers4/silero-vad}},
  commit = {insert_some_commit_here},
  email = {hello@silero.ai}
}
```

### Distil-Whisper
```bibtex
@misc{gandhi2023distilwhisper,
      title={Distil-Whisper: Robust Knowledge Distillation via Large-Scale Pseudo Labelling},
      author={Sanchit Gandhi and Patrick von Platen and Alexander M. Rush},
      year={2023},
      eprint={2311.00430},
      archivePrefix={arXiv},
      primaryClass={cs.CL}
}
```

### Parler-TTS
```bibtex
@misc{lacombe-etal-2024-parler-tts,
  author = {Yoach Lacombe and Vaibhav Srivastav and Sanchit Gandhi},
  title = {Parler-TTS},
  year = {2024},
  publisher = {GitHub},
  journal = {GitHub repository},
  howpublished = {\url{https://github.com/huggingface/parler-tts}}
}
```
