# Report schema

The deterministic aggregation stage produces a human-readable text dataset before the AI stage.

```text
SEO-АУДИТ САЙТА
Всего страниц: <N>

--- Страница 1 ---
Title: <value or [ОТСУТСТВУЕТ]>
Title длина: <N> символов
Description: <value or [ОТСУТСТВУЕТ]>
Description длина: <N> символов
H1: <value or [ОТСУТСТВУЕТ]>
```

Claude then converts this dataset into a structured narrative with overall statistics, findings for Title / Description / H1 and recommended corrective actions.
