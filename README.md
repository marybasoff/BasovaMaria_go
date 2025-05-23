
Система извлечения и распознавания именованных сущностей (NER)
=============

Содержание
-----------------
-   [Обзор проекта](#обзор-проекта)
-   [Архитектура решения](#архитектура-решения)
-   [Выбор моделей](#выбор-моделей)
-   [Подробное описание компонентов](#подробное-описание-компонентов)
-   [Streamlit интерфейс](#streamlit-интерфейс)
-   [Запуск проекта](#запуск-проекта)
-   [Сравнение моделей](#сравнение-моделей)
-   [Post Scriptum](#post-scriptum)

Обзор проекта
-----------------
Проект представляет собой комплексное решение для:
1. Извлечения текстового контента с веб-страниц
2. Автоматической разметки именованных сущностей (NER)
3. Адаптации языковых моделей для задач NER
4. Обучения и оценки моделей на извлеченных данных

Архитектура решения
-----------------
```
[Веб-страница] → [Извлечение текста] → [Предразметка BERT] → [Обучение] → [Прогнозирование]
                     ↑                      ↑                      ↑
                [Очистка HTML]      [Мультиязычная NER]    [Кастомизированная NER]
```

Выбор моделей
-----------------
Основная модель: Sberbank AI (ruBert-large)
**Обоснование выбора:**
1. **Оптимизация для русского языка**: Специально обучена на русскоязычных данных
2. **Эффективность**: Оптимальное соотношение качества и производительности
3. **Поддержка**: Активно развивается российским разработчиком

**Кастомизация для NER:**
1. Добавлен классификационный head для 9 классов сущностей
2. Настроены соответствия меток:
   ```python
   {
       0: 'O', 1: 'B-PER', 2: 'I-PER',
       3: 'B-ORG', 4: 'I-ORG',
       5: 'B-LOC', 6: 'I-LOC',
       7: 'B-MISC', 8: 'I-MISC'
   }
   ```
3. Заморожены базовые слои для сохранения языковых знаний

Почему не выбрали Qwen:
1. **Избыточность**: Qwen слишком большая для задачи NER
2. **Ресурсоемкость**: Требует значительных вычислительных мощностей
3. **Специализация**: ruBert лучше адаптирована для русского языка

BERT модель (Davlan/bert-base-multilingual-cased-ner-hrl)
**Роль в проекте:**
- Предварительная разметка данных (teacher model)
- Мультиязычная поддержка
- Быстрое прототипирование

Подробное описание компонентов
-----------------
1. Извлечение текста (`extract_text_from_url`)
**Функционал:**
- Имитация браузера через User-Agent
- Удаление некритичных HTML-элементов
- Интеллектуальное извлечение контента
- Очистка и нормализация текста

**Алгоритм:**
1) Запрос страницы с таймаутом
2) Парсинг BeautifulSoup
3) Удаление скриптов, стилей и др.
4) Извлечение из содержательных тегов (article, main и др.)
5) Объединение и очистка текста

2. Аннотация BERT (`annotate_with_bert`)
**Особенности:**
- Использует предобученную NER-модель
- Возвращает сущности с позициями и типами
- Ограничение в 512 токенов

3. Адаптация модели (`adapt_qwen_for_ner`)
**Ключевые изменения:**
1) Добавлен TokenClassification head
2) Настроены метки сущностей
3) Заморозка базовых слоев

4. NERDataset
**Особенности:**
- Преобразование текста и аннотаций в формат для обучения
- Выравнивание меток с токенизированным текстом
- Поддержка динамической длины текста

Streamlit интерфейс
-----------------
Ключевые компоненты:
```python
# Настройки страницы
st.set_page_config(
    page_title="NER со SBER AI",
    layout="wide"
)

# Основные функции
@st.cache_data  # Кэширование данных
@st.cache_resource  # Кэширование моделей
```

Функционал:
1. Ввод URL через сайдбар
2. Настройка параметров анализа
3. Визуализация результатов:
   - Очищенный текст
   - Токены и предсказания
   - Технические метрики

Запуск проекта
-----------------
1. Установите зависимости:
```bash
pip install torch transformers beautifulsoup4 requests streamlit
```

2. Запустите приложение (если через google colab):
```bash
!streamlit run /usr/local/lib/python3.11/dist-packages/colab_kernel_launcher.py [ARGUMENTS]
```

3. Откройте в браузере:
```
http://localhost:8501
```

Сравнение моделей
-----------------
| Модель                   | Преимущества            | Недостатки                      |
|--------------------------|-------------------------|---------------------------------|
| **Sberbank ruBert**      | Лучшее качество для RU  | Только русский язык             |
| **Facebook XLM-RoBERTa** | Мультиязычность         | Требует индивидуальной настройки|
| **Babelscape WikiNEuRal**| Специализирована для NER| Ограниченный набор сущностей    |
| **Qwen**                 | Мощная языковая модель  | Избыточна для NER               |

Post Scriptum
-----------------
На самом деле, была большая надежда на Qwen модель, код был кастомизирован специально под нее, но он считался очень долго и показывал неадекватные предикты. Проблема, к сожалению, не была решена, поэтому было принято решение использовать в качестве основной модели sberbank ai.
