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
- Any instruction-following model on the [Hugging Face Hub](https://huggingface.co/models?pipeline_tag=text-generation&sort=trending) via Transformers 
- [mlx-lm](https://github.com/ml-explore/mlx-examples/blob/main/llms/README.md)
- [OpenAI API](https://platform.openai.com/docs/quickstart)

**TTS**
- [Parler-TTS](https://github.com/huggingface/parler-tts) 
- [MeloTTS](https://github.com/myshell-ai/MeloTTS)
- [ChatTTS](https://github.com/2noise/ChatTTS?tab=readme-ov-file)

## الاعداد

استنساخ المستودع:
```bash
git clone (https://github.com/art539/speech-to-speech.git)
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


### دعم اللغة

يدعم خط الأنابيب حاليًا اللغات الإنجليزية والفرنسية والإسبانية والصينية واليابانية والكورية.
