### البنية
ينفذ هذا المستودع خط أنابيب متتابع من الكلام إلى الكلام يتكون من الأجزاء التالية:
1. **اكتشاف نشاط الصوت (VAD)**
2. **تحويل الكلام إلى نص (STT)**
3. **نموذج اللغة (LM)**
4. **تحويل النص إلى كلام (TTS)**

### الوحدات النمطية
يوفر خط الأنابيب نهجًا مفتوحًا وموحدًا بالكامل، مع التركيز على الاستفادة من النماذج المتاحة من خلال مكتبة Transformers على مركز Hugging Face. تم تصميم الكود لسهولة التعديل، ونحن ندعم بالفعل تنفيذات خاصة بالجهاز ومكتبات خارجية:

**VAD**
- [Silero VAD v5](https://github.com/snakers4/silero-vad)

**STT**
- أي نقطة تفتيش لنموذج [Whisper](https://huggingface.co/docs/transformers/en/model_doc/whisper) على Hugging Face Hub من خلال Transformers ، بما في ذلك [whisper-large-v3](https://huggingface.co/openai/whisper-large-v3) و[distil-large-v3](https://huggingface.co/distil-whisper/distil-large-v3)
- [Lightning Whisper MLX](https://github.com/mustafaaljadery/lightning-whisper-mlx?tab=readme-ov-file#lightning-whisper-mlx)
- [Paraformer - FunASR](https://github.com/modelscope/FunASR)

**LLM**
- أي نموذج يتبع التعليمات على [Hugging Face Hub](https://huggingface.co/models?pipeline_tag=text-generation&sort=trending) عبر Transformers 
- [mlx-lm](https://github.com/ml-explore/mlx-examples/blob/main/llms/README.md)
- [OpenAI API](https://platform.openai.com/docs/quickstart)

**TTS**
- [Parler-TTS](https://github.com/huggingface/parler-tts) 
- [MeloTTS](https://github.com/myshell-ai/MeloTTS)
- [ChatTTS](https://github.com/2noise/ChatTTS?tab=readme-ov-file)

## الإعداد

استنساخ المستودع:
```bash
git clone https://github.com/art539/speech-to-speech.git
cd speech-to-speech
```

قم بتثبيت التبعيات المطلوبة باستخدام [uv](https://github.com/astral-sh/uv):
```bash
uv pip install -r requirements.txt
```

بالنسبة لمستخدمي Mac، استخدم ملف `requirements_mac.txt` بدلاً من ذلك:
```bash
uv pip install -r requirements_mac.txt
```

إذا كنت تريد استخدام Melo TTS، فأنت بحاجة أيضًا إلى تشغيل:
```bash
python -m unidic download
```

## الاستخدام

يمكن تشغيل خط الأنابيب بطريقتين:
- **نهج الخادم/العميل**: يتم تشغيل النماذج على الخادم، ويتم بث إدخال/إخراج الصوت من العميل.
- **نهج محلي**: يتم تشغيله محليًا.

### الإعداد الموصى به

### نهج الخادم/العميل

1. قم بتشغيل خط الأنابيب على الخادم:
```bash
python s2s_pipeline.py --recv_host 0.0.0.0 --send_host 0.0.0.0
```

2. قم بتشغيل العميل محليًا للتعامل مع إدخال الميكروفون واستقبال الصوت الناتج:
```bash
python listen_and_play.py --host <عنوان IP الخاص بخادمك>
```

### النهج المحلي (Mac)

1. لإعدادات مثالية على Mac:
```bash
python s2s_pipeline.py --local_mac_optimal_settings
```

هذا الإعداد:
- يضيف `--device mps` لاستخدام MPS لجميع الطرز.
- تعيين LightningWhisperMLX لـ STT
- تعيين MLX LM لنموذج اللغة
- تعيين MeloTTS لـ TTS

### Docker Server

#### تثبيت NVIDIA Container Toolkit

https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html

#### بدء تشغيل حاوية docker
```docker compose up```

### الاستخدام الموصى به مع Cuda

الاستفادة من Torch Compile لـ Whisper وParler-TTS. **يتيح استخدام Parler-TTS بث إخراج الصوت، مما يقلل بشكل أكبر من زمن الوصول الإجمالي** 🚀:

```bash
python s2s_pipeline.py \
--lm_model_name microsoft/Phi-3-mini-4k-instruct \
--stt_compile_mode reduce-overhead \
--tts_compile_mode default \
--recv_host 0.0.0.0 \
--send_host 0.0.0.0
```

في الوقت الحالي، لا تتوافق أوضاع التقاط الرسوم البيانية CUDA مع بث Parler-TTS (`reduce-overhead`، `max-autotune`).

### دعم متعدد اللغات

يدعم خط الأنابيب حاليًا الإنجليزية والفرنسية والإسبانية والصينية واليابانية والكورية.
يتم النظر في حالتين للاستخدام:

- **محادثة أحادية اللغة**: فرض إعداد اللغة باستخدام علامة `--language`، وتحديد رمز اللغة المستهدفة (الإعداد الافتراضي هو 'en').
- **تبديل اللغة**: اضبط `--language` على 'تلقائي'. في هذه الحالة، يكتشف Whisper اللغة لكل مطالبة منطوقة، ويتم مطالبة LLM بـ "`الرجاء الرد على رسالتي في ...`" لضمان أن تكون الاستجابة باللغة المكتشفة.

يرجى ملاحظة أنه يجب عليك استخدام نقاط تفتيش STT وLLM المتوافقة مع اللغة (اللغات) المستهدفة. بالنسبة لجزء STT، فإن Parler-TTS ليس متعدد اللغات بعد. وفي غضون ذلك، يجب عليك استخدام Melo (الذي يدعم اللغة الإنجليزية والفرنسية والإسبانية والصينية واليابانية والكورية) أو Chat-TTS.

#### مع إصدار الخادم:

للكشف التلقائي عن اللغة:

```bash
python s2s_pipeline.py \
--stt_model_name large-v3 \
--language auto \
--mlx_lm_model_name mlx-community/Meta-Llama-3.1-8B-Instruct \
```

أو للغة واحدة على وجه الخصوص، الصينية في هذا المثال

```bash
python s2s_pipeline.py \
--stt_model_name large-v3 \
--language zh \
--mlx_lm_model_name mlx-community/Meta-Llama-3.1-8B-Instruct \
```

#### إعداد Mac المحلي

للكشف التلقائي عن اللغة:

```bash
python s2s_pipeline.py \
--l

### البنية
ينفذ هذا المستودع خط أنابيب متتابع من الكلام إلى الكلام يتكون من الأجزاء التالية:
1. **اكتشاف نشاط الصوت (VAD)**
2. **تحويل الكلام إلى نص (STT)**
3. **نموذج اللغة (LM)**
4. **تحويل النص إلى كلام (TTS)**

### الوحدات النمطية
يوفر خط الأنابيب نهجًا مفتوحًا وموحدًا بالكامل، مع التركيز على الاستفادة من النماذج المتاحة من خلال مكتبة Transformers على مركز Hugging Face. تم تصميم الكود لسهولة التعديل، ونحن ندعم بالفعل تنفيذات خاصة بالجهاز ومكتبات خارجية:

**VAD**
- [Silero VAD v5](https://github.com/snakers4/silero-vad)

**STT**
- أي نقطة تفتيش لنموذج [Whisper](https://huggingface.co/docs/transformers/en/model_doc/whisper) على Hugging Face Hub من خلال Transformers ، بما في ذلك [whisper-large-v3](https://huggingface.co/openai/whisper-large-v3) و[distil-large-v3](https://huggingface.co/distil-whisper/distil-large-v3)
- [Lightning Whisper MLX](https://github.com/mustafaaljadery/lightning-whisper-mlx?tab=readme-ov-file#lightning-whisper-mlx)
- [Paraformer - FunASR](https://github.com/modelscope/FunASR)

**LLM**
- أي نموذج يتبع التعليمات على [Hugging Face Hub](https://huggingface.co/models?pipeline_tag=text-generation&sort=trending) عبر Transformers 
- [mlx-lm](https://github.com/ml-explore/mlx-examples/blob/main/llms/README.md)
- [OpenAI API](https://platform.openai.com/docs/quickstart)

**TTS**
- [Parler-TTS](https://github.com/huggingface/parler-tts) 
- [MeloTTS](https://github.com/myshell-ai/MeloTTS)
- [ChatTTS](https://github.com/2noise/ChatTTS?tab=readme-ov-file)

## الإعداد

استنساخ المستودع:
```bash
git clone https://github.com/art539/speech-to-speech.git
cd speech-to-speech
```

قم بتثبيت التبعيات المطلوبة باستخدام [uv](https://github.com/astral-sh/uv):
```bash
uv pip install -r requirements.txt
```

بالنسبة لمستخدمي Mac، استخدم ملف `requirements_mac.txt` بدلاً من ذلك:
```bash
uv pip install -r requirements_mac.txt
```

إذا كنت تريد استخدام Melo TTS، فأنت بحاجة أيضًا إلى تشغيل:
```bash
python -m unidic download
```

## الاستخدام

يمكن تشغيل خط الأنابيب بطريقتين:
- **نهج الخادم/العميل**: يتم تشغيل النماذج على الخادم، ويتم بث إدخال/إخراج الصوت من العميل.
- **نهج محلي**: يتم تشغيله محليًا.

### الإعداد الموصى به

### نهج الخادم/العميل

1. قم بتشغيل خط الأنابيب على الخادم:
```bash
python s2s_pipeline.py --recv_host 0.0.0.0 --send_host 0.0.0.0
```

2. قم بتشغيل العميل محليًا للتعامل مع إدخال الميكروفون واستقبال الصوت الناتج:
```bash
python listen_and_play.py --host <عنوان IP الخاص بخادمك>
```

### النهج المحلي (Mac)

1. لإعدادات مثالية على Mac:
```bash
python s2s_pipeline.py --local_mac_optimal_settings
```

هذا الإعداد:
- يضيف `--device mps` لاستخدام MPS لجميع الطرز.
- تعيين LightningWhisperMLX لـ STT
- تعيين MLX LM لنموذج اللغة
- تعيين MeloTTS لـ TTS

### Docker Server

#### تثبيت NVIDIA Container Toolkit

https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html

#### بدء تشغيل حاوية docker
```docker compose up```

### الاستخدام الموصى به مع Cuda

الاستفادة من Torch Compile لـ Whisper وParler-TTS. **يتيح استخدام Parler-TTS بث إخراج الصوت، مما يقلل بشكل أكبر من زمن الوصول الإجمالي** 🚀:

```bash
python s2s_pipeline.py \
--lm_model_name microsoft/Phi-3-mini-4k-instruct \
--stt_compile_mode reduce-overhead \
--tts_compile_mode default \
--recv_host 0.0.0.0 \
--send_host 0.0.0.0
```

في الوقت الحالي، لا تتوافق أوضاع التقاط الرسوم البيانية CUDA مع بث Parler-TTS (`reduce-overhead`، `max-autotune`).

### دعم متعدد اللغات

يدعم خط الأنابيب حاليًا الإنجليزية والفرنسية والإسبانية والصينية واليابانية والكورية.
يتم النظر في حالتين للاستخدام:

- **محادثة أحادية اللغة**: فرض إعداد اللغة باستخدام علامة `--language`، وتحديد رمز اللغة المستهدفة (الإعداد الافتراضي هو 'en').
- **تبديل اللغة**: اضبط `--language` على 'تلقائي'. في هذه الحالة، يكتشف Whisper اللغة لكل مطالبة منطوقة، ويتم مطالبة LLM بـ "`الرجاء الرد على رسالتي في ...`" لضمان أن تكون الاستجابة باللغة المكتشفة.


#### مع إصدار الخادم:

للكشف التلقائي عن اللغة:

```bash
python s2s_pipeline.py \
--stt_model_name large-v3 \
--language auto \
--mlx_lm_model_name mlx-community/Meta-Llama-3.1-8B-Instruct \
```

أو للغة واحدة على وجه الخصوص، الصينية في هذا المثال

```bash
python s2s_pipeline.py \
--stt_model_name large-v3 \
--language zh \
--mlx_lm_model_name mlx-community/Meta-Llama-3.1-8B-Instruct \
```

#### إعداد Mac المحلي

للكشف التلقائي عن اللغة:

```bash
python s2s_pipeline.py \
--l
