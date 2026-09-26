# Workflow walkthrough

## Phase 1 — Start and configure

### `▶️ Запуск аудита`
Manual trigger. Starts one audit execution.

### `🌐 Адрес сайта`
Stores the target site in `baseUrl`.

## Phase 2 — Discover internal pages

### `📥 Скачать главную страницу`
Downloads the homepage HTML using an HTTP GET request.

### `🔗 Собрать ссылки`
Extracts `title`, `meta description`, `h1` and all `a[href]` values.

### `📋 Разбить на список`
Converts the link array into individual n8n items so each URL can be processed independently.

### `🧹 Фильтр ссылок`
Keeps relative URLs and URLs belonging to the configured site.

### `🔂 Убрать дубли`
Removes repeated raw URL values.

### `🔧 Сделать полные адреса`
Creates `fullUrl`: absolute URLs are preserved; relative paths are combined with `baseUrl`.

## Phase 3 — Crawl and extract

### `📥 Скачать все страницы`
Requests every discovered `fullUrl`. The current design continues when an individual page request fails.

### `🏷️ Извлечь мета-теги`
Extracts three fields from each returned page:

```text
title
description
h1
```

## Phase 4 — Build the analytical dataset

### `📝 Собрать отчёт`
Converts multiple page items into one text payload. For each page it records the values of `Title`, `Description` and `H1` and calculates character counts for `Title` and `Description`.

Missing values are represented as `[ОТСУТСТВУЕТ]`.

## Phase 5 — AI analysis

### `🤖 Анализ Claude AI`
Receives the aggregated report and asks Claude to produce a structured SEO assessment covering overall statistics, Title, Description, H1 and corrective actions.

### `🧠 Модель Claude`
Provides the LLM used by the chain.

## Phase 6 — Presentation and delivery

### `🎨 Оформить HTML`
Converts selected Markdown constructs from the AI response into simple HTML suitable for an email body.

### `✉️ Отправить на email`
Sends the formatted report through Gmail.

## Design principle

The workflow intentionally uses a deterministic-first approach:

```text
collect facts → normalize data → aggregate → ask AI to interpret
```

This makes the LLM input bounded and traceable instead of sending raw website HTML directly to the model.
