# AI settings

Для работы с моделями будем использовать расширение Cline.

1. Установить расширение **Cline** ([saoudrizwan.claude-dev](https://marketplace.visualstudio.com/items?itemName=saoudrizwan.claude-dev)). После установки на левой боковой панели VS Code появится новая иконка.
1.

## Облачные модели
1. Google Gemini 2.5 Flash / Pro (через Google AI Studio)
    - до 1–2 млн токенов
    - для поиск багов по всему проекту, написание сложных фич
1. Mistral Large 4 / Mistral Codestral (через Mistral la Plateforme)
    - отличный бесплатный уровень
1. Встроенные бесплатные модели Cline (Cline Free Provider)

## Локальные модели
### До 8 ГБ
- Для автодополнения
    1. qwen2.5-coder:7b-instruct (синтаксис, автодополнение, очень быстрая)
    1. deepseek-coder:6.7b (Оптимизация под старые и редкие языки)
- Для автономной работы
    1. qwen3:8b (Интеграция инструментов, агенты)
    1. qwen3.5:9b (Мультимодальность, чтение скриншотов)
- Отладка
    1. deepseek-r1:8b (Сложная логика, поиск архитектурных багов, медленная)
### До 16 ГБ
1. gpt-oss:20b
1. qwen2.5-coder:14b
1. mistral-nemo:12b
