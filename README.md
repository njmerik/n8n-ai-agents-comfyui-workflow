# 🎬 AI Character Video Generator with Upscaling

Автоматизированная система генерации высококачественных видео с вашим персонажем на основе текстового описания. Использует **n8n** для оркестрации, **ComfyUI** для генерации изображений с апскейлом и анимации.

## ✨ Возможности

- 🎨 **Генерация изображений** с сохранением внешности персонажа (Flux + USO LoRA)
- 🔍 **AI-апскейл изображений** (Flux Upscaler 2x) для повышения детализации перед анимацией
- 🎥 **Автоматическая анимация** сгенерированных сцен (Wan2.1 Image-to-Video)
- 🤖 **Гибкая модель AI**: переключение между бесплатной (Llama/Ollama) и платной (OpenAI/Z-AI)
- 💬 **Простой интерфейс**: достаточно написать сцену в чат
- 🔄 **Полная автоматизация**: от идеи до готового видео за один клик

## 📋 Структура проекта

├── workflows/
│   └── character-video-generator.json    # Основной workflow n8n
├── comfyui-workflows/
│   ├── flux1_dev_uso_reference_image_gen.json  # Генерация изображений
│   ├── upscaler_flux.json                      # Flux Upscaler (2x)
│   └── video_wan2_2_14B_i2v.json              # Анимация видео
├── input/
│   └── girl_ref.png                        # Референс персонажа
└── README.md


## 🚀 Как это работает

### Общая схема
┌─────────────┐
│   Chat      │
│   Trigger   │
└────────────┘
       │
       ▼
┌─────────────┐
│     IF      │──── #free ────→ Llama (Ollama)
│  (Switch)   │
└────────────┘──── иначе ────→ OpenAI/Z-AI
       │
       ▼
┌─────────────┐
│  AI Agent   │ → Генерирует промпт сцены на английском
└──────┬──────
       │
       ▼
┌─────────────┐
│    Code     │ → Подставляет промпт в JSON + описание девушки
└──────┬──────┘
       │
       ▼
┌─────────────────┐
│  ComfyUI #1     │ → Flux + USO LoRA → Генерация изображения
│  (Image Gen)    │   с референсом персонажа
└──────┬──────────┘
       │
       ▼
┌─────────────────┐
│  ComfyUI #2     │ → Flux Upscaler → Увеличение 2x + детализация
│  (Upscaler)     │   (опционально, для лучшего качества)
└────────────────┘
       │
       ▼
┌─────────────────┐
│  ComfyUI #3     │ → Wan2.1 I2V → Анимация видео
│  (Video Gen)    │
└─────────────────┘

## 📦 Требования

### Обязательные:
- **Docker** с запущенным **n8n**
- **ComfyUI** (локальная установка)
- **Ollama** (для бесплатной модели Llama)

### Модели ComfyUI:

**Для генерации изображений:**
- `flux1-dev-fp8.safetensors`
- `uso-flux1-projector-v1.safetensors`
- `uso-flux1-dit-lora-v1.safetensors`
- `sigclip_vision_patch14_384.safetensors`

**Для апскейла (Flux):**
- `flux2/flux-2-klein-9b-fp8.safetensors`
- `flux2/qwen_3_8b_fp8mixed.safetensors`
- `flux2/flux2-vae.safetensors`

**Для анимации видео:**
- `wan_2.1_vae.safetensors`
- `wan2.2_i2v_high_noise_14B_fp8_scaled.safetensors`
- `wan2.2_i2v_low_noise_14B_fp8_scaled.safetensors`
- `Wan2.2-Lightning_I2V-A14B-4steps-lora_HIGH_fp16.safetensors`
- `Wan2.2-Lightning_I2V-A14B-4steps-lora_LOW_fp16.safetensors`
- `umt5_xxl_fp8_e4m3fn_scaled.safetensors`

### API ключи:
- **OpenAI API Key** (для платной модели)
- **Z-AI API Key** (альтернатива OpenAI)

## ⚙️ Установка

### 1. Настройка ComfyUI

1. Установите [ComfyUI](https://github.com/comfyanonymous/ComfyUI)
2. Скачайте необходимые модели в папки:
   - Checkpoints → `ComfyUI/models/checkpoints/`
   - CLIP → `ComfyUI/models/clip/`
   - VAE → `ComfyUI/models/vae/`
   - UNET → `ComfyUI/models/unet/`
   - LoRA → `ComfyUI/models/loras/`

3. Поместите ваш референс персонажа в `ComfyUI/input/girl_ref.png`

4. Импортируйте workflow'ы в ComfyUI:
   - Перетащите каждый JSON файл в интерфейс ComfyUI → Save

### 2. Настройка n8n

1. **Установите n8n** (если еще нет):
   ```bash
   docker run -d --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n n8nio/n8n

   Настройте credentials:

    ComfyUI Account: URL http://host.docker.internal:8188 (или IP вашего компьютера)
    OpenAI Account: Ваш API ключ
    Ollama Account: URL http://host.docker.internal:11434

Установите Ollama и скачайте Llama:
ollama pull llama3.1
# или
ollama pull phi4
    Импортируйте workflow:
        Откройте n8n → Workflows → Import from File
        Выберите character-video-generator.json

3. Настройка узлов
AI Agent (платный)

    Chat Model: OpenAI Chat Model / Z-AI
    System Message:
   Ты помощник для генерации изображений. Пользователь пишет сцену на русском. 
Твоя задача — перевести это в детальное описание сцены на английском языке 
для генератора картинок. Описывай только действие и окружение, персонажа 
описывать не нужно (он уже задан). Не пиши приветствий, отвечай только 
описанием сцены.
AI Agent1 (бесплатный)

    Chat Model: Ollama Chat Model (Llama/phi4)
    System Message: тот же текст

IF Node

    Condition: String
    Value 1: {{ $json.chatInput }}
    Operation: Contains
    Value 2: #free

Code Node
const workflow = { /* ваш JSON для генерации изображений */ };

const agentText = $json.output || "";
const finalPrompt = "A photorealistic young woman with light blonde hair, " + agentText;

workflow["112:6"]["inputs"]["text"] = finalPrompt;

return { json: { workflow: JSON.stringify(workflow), videoPrompt: finalPrompt } };
📖 Использование
Базовое использование

    Откройте chat-интерфейс n8n
    Напишите описание сцены:
    девушка гуляет с собаками по городской улице
    Нажмите Enter
    Подождите 3-7 минут (включая апскейл и анимацию)
    Получите готовое видео! 🎉

Переключение моделей
Бесплатная модель (Llama):
девушка с зонтом под дождем #free
Платная модель (качественнее):
девушка гуляет в парке с собаками
Примеры запросов

    девушка в неоновом кафе
    девушка за рулем машины
    девушка читает книгу в библиотеке
    девушка с цветами на улице #free

🎛️ Настройки
Параметры генерации изображений
В workflow Flux:

    Steps: 20 (качество/скорость)
    CFG: 1
    Sampler: euler
    Размер: 1024x1024

Параметры апскейла
В workflow Flux Upscaler:

    Scale: 2x
    Denoise (Creativity): 0.17-0.3 (меньше = ближе к оригиналу)
    Steps: 3
    Tile Size: 1024x1024

Параметры генерации видео
В workflow Wan2.1:

    Length: 81 (количество кадров, уменьшите до 49 для скорости)
    FPS: 16
    Width/Height: 640x640
    Steps: 4 (Lightning LoRA)

⚡ Оптимизация производительности
Для быстрой генерации:

    Отключите узел Flux Upscaler (правой кнопкой → Disable)
    Уменьшите length в Wan2.1 до 49
    Используйте Llama вместо OpenAI
    Уменьшите разрешение

Для лучшего качества:

    Оставьте Flux Upscaler включенным
    Увеличьте steps в Flux до 30-40
    Установите denoise в апскейлере на 0.3-0.45
    Используйте OpenAI/GPT-4 для промптов
    Увеличьте length в Wan2.1 до 81

🐛 Решение проблем
Ошибка: "cannot identify image file"

    Проверьте, что girl_ref.png существует в ComfyUI/input/
    Убедитесь, что файл не поврежден

Ошибка: "Video generation timeout"

    Увеличьте Timeout в узле ComfyUI (до 60-90 минут)
    Уменьшите length в Wan2.1 workflow
    Проверьте, хватает ли VRAM на GPU

Ошибка: "registry.ollama.ai/library/phi4:latest does not support tools"

    Отключите Tool (Think) от AI Agent1
    Или используйте llama3.1 вместо phi4

Ошибка: "No binary data found in property"

    Проверьте настройки Binary Property в узле ComfyUI
    Убедитесь, что выбрано поле data или binary
    Проверьте, что предыдущий узел успешно сгенерировал файл

Генерируется не та девушка

    Проверьте путь к girl_ref.png
    Убедитесь, что USO LoRA правильно подключена
    В System Message укажите не описывать внешность

Видео не анимируется

    Проверьте, что картинка сгенерировалась успешно
    Убедитесь, что в узле #3 правильно подключено Input Image
    Проверьте консоль ComfyUI на ошибки
    
🤝 Вклад
Pull requests приветствуются! Для серьезных изменений сначала откройте issue.
📄 Лицензия
MIT License — используйте на здоровье!
👥 Авторы
Наталия Жмерик/ Njmerik
🙏 Благодарности

    ComfyUI
     — мощный инструмент для генерации
    n8n
     — отличная платформа для автоматизации
    Flux
     — потрясающая модель генерации
    Wan2.1
     — качественная анимация
    Ollama
     — локальные LLM
    TimeSaver Nodes
     — удобные утилиты для ComfyUI

📞 Контакты

    Telegram: @Njmerik
    Email: natali@njmerik.ru
    GitHub: https://github.com/njmerik


    
