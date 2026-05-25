# 🎬 AI Character Video Generator with Upscaling

Автоматизированная система генерации  видео с вашим персонажем на основе текстового описания. Использует **n8n** для оркестрации, **ComfyUI** для генерации изображений с апскейлом и анимации.

## ✨ Возможности

| Функция | Описание |
|---------|----------|
| 🎨 **Генерация изображений** | С сохранением внешности персонажа (Flux + USO LoRA) |
| 🔍 **AI-апскейл изображений** | Flux Upscaler 2x для повышения детализации перед анимацией |
| 🎥 **Автоматическая анимация** | Сгенерированных сцен (Wan2.1 Image-to-Video) |
| 🤖 **Гибкая модель AI** | Переключение между бесплатной (Llama/Ollama) и платной (OpenAI/Z-AI) |
| 💬 **Простой интерфейс** | Достаточно написать сцену в чат |
| 🔄 **Полная автоматизация** | От идеи до готового видео за один клик |

## 📋 Структура проекта

```text
.
├── workflows/
│   └── character-video-generator.json
├── comfyui-workflows/
│   ├── flux1_dev_uso_reference_image_gen.json
│   ├── upscaler_flux.json
│   └── video_wan2_2_14B_i2v.json
├── input/
│   └── girl_ref.png
└── README.md




Детальная архитектура

┌─────────────────────────────────────────────────────────────────┐
│                        CHAT TRIGGER                             │
│                    (When chat message)                          │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
                ┌────────────────┐
                │  IF: Contains  │
                │     #free?     │
                └────────────────┘
                         │
            ┌────────────┴────────────┐
            │                         │
         #free                    иначе
            │                         │
            ▼                         ▼
    ┌──────────────┐          ┌──────────────┐
    │ Llama/Ollama │          │ OpenAI/Z-AI  │
    │   (Free)     │          │   (Paid)     │
    └──────────────┘          └──────────────┘
            │                         │
            └────────────┬────────────┘
                         │
                         ▼
                ┌────────────────┐
                │   AI Agent     │
                │ (Prompt Gen)   │
                └────────────────┘
                         │
                         ▼
                ┌────────────────┐
                │  Code Node     │
                │ (Inject Prompt)│
                └────────────────
                         │
                         ▼
        ╔═══════════════════════════════════╗
        ║     COMFYUI PIPELINE              ║
        ╚═══════════════════════════════════╝
                         │
            ┌────────────┴────────────┐
            ▼                         ▼
    ┌──────────────┐          ┌──────────────┐
    │  ComfyUI #1  │   ────→  │  ComfyUI #2  │
    │ Image Generation        │ Flux Upscaler│
    │ Flux + USO LoRA         │ 2x + Details │
    └──────────────┘          └──────────────┘
            │                         │
            └────────────┬────────────┘
                         │
                         ▼
                ┌──────────────┐
                │  ComfyUI #3  │
                │  Wan2.1 I2V  │
                │  Animation   │
                └──────────────┘
                         │
                         ▼
                ┌──────────────┐
                │ FINAL VIDEO  │
                └──────────────┘

📦 Требования
Обязательные компоненты
Компонент	Назначение
Docker	Запуск n8n
n8n	Оркестрация workflow
ComfyUI	Генерация изображений и видео
Ollama	Бесплатная LLM (Llama)

Модели ComfyUI

Для генерации изображений:

    flux1-dev-fp8.safetensors

    uso-flux1-projector-v1.safetensors

    uso-flux1-dit-lora-v1.safetensors

    sigclip_vision_patch14_384.safetensors

Для апскейла (Flux):

    flux2/flux-2-klein-9b-fp8.safetensors

    flux2/qwen_3_8b_fp8mixed.safetensors

    flux2/flux2-vae.safetensors

Для анимации видео:

    wan_2.1_vae.safetensors

    wan2.2_i2v_high_noise_14B_fp8_scaled.safetensors

    wan2.2_i2v_low_noise_14B_fp8_scaled.safetensors

    Wan2.2-Lightning_I2V-A14B-4steps-lora_HIGH_fp16.safetensors

    Wan2.2-Lightning_I2V-A14B-4steps-lora_LOW_fp16.safetensors

    umt5_xxl_fp8_e4m3fn_scaled.safetensors

API ключи

Ключ	Назначение
OpenAI API Key	Платная модель (качественная)
Z-AI API Key	Альтернатива OpenAI

⚙️ Установка
1. Настройка ComfyUI
Шаг	   Действие
1	    Установите ComfyUI
2	    Скачайте модели в папки (см. список выше)
3	    Поместите girl_ref.png в ComfyUI/input/
4	    Импортируйте JSON workflow'ы в ComfyUI

2. Настройка n8n

Установка Docker:
docker run -d --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n n8nio/n8n

Настройка Credentials:
Сервис	URL
ComfyUI	http://host.docker.internal:8188
Ollama	http://host.docker.internal:11434

Установка Ollama:
ollama pull llama3.1
# или
ollama pull phi4

3. Настройка узлов n8n
AI Agent (платный):

    Chat Model: OpenAI Chat Model / Z-AI

    System Message:
Ты помощник для генерации изображений. Пользователь пишет сцену на русском. 
Твоя задача — перевести это в детальное описание сцены на английском языке 
для генератора картинок. Описывай только действие и окружение, персонажа 
описывать не нужно (он уже задан). Не пиши приветствий, отвечай только 
описанием сцены.

AI Agent1 (бесплатный):

    Chat Model: Ollama Chat Model (Llama/phi4)

    System Message: тот же текст

IF Node:
Поле	          Значение
Condition	      String
Value 1	       {{ $json.chatInput }}
Operation	     Contains
Value 2      	 #free

Code Node:

const workflow = { /* ваш JSON для генерации изображений */ };
const agentText = $json.output || "";
const finalPrompt = "A photorealistic young woman with light blonde hair, " + agentText;
workflow["112:6"]["inputs"]["text"] = finalPrompt;
return { json: { workflow: JSON.stringify(workflow), videoPrompt: finalPrompt } };

📖 Использование

Базовое использование

Шаг	        Действие
1	          Откройте chat-интерфейс n8n
2	          Напишите описание сцены
3	          Нажмите Enter
4	          Подождите 3-7 минут
5	          Получите готовое видео! 🎉

Примеры запросов

Тип	                    Пример
Платная модель	        девушка гуляет в парке с собаками
Бесплатная модель	    девушка с зонтом под дождем #free
Ночная сцена	        девушка в неоновом кафе
Экшн	                девушка за рулем машины
Спокойная	            девушка читает книгу в библиотеке

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


    
