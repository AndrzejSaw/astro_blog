---
title: "Создаём форму запроса в Astro"
pubDate: 2025-02-15
description: "Пошагово разбираем, как сверстать и обработать форму обратной связи в проекте на Astro."
author: "Мария Пименова"
image:
  url: "https://images.unsplash.com/photo-1556740749-887f6717d7e4?auto=format&fit=crop&w=1200&q=80"
  alt: "Рабочий стол с блокнотом и формой обратной связи"
tags: ["astro", "forms", "frontend"]
---
# Создаём форму запроса в Astro

Дата публикации: 15 февраля 2025 года

Формы остаются ключевым способом общения с посетителями: через них собираем заявки, вопросы и отзывы. В Astro можно быстро сверстать форму на чистом HTML и дополнить её интерактивом при необходимости. Разберёмся, как реализовать форму запроса с валидацией и отправкой данных на серверless-функцию.

## Шаг 1. Планируем структуру

Для формы запроса нам понадобятся поля имени, email, темы обращения и текст сообщения. Полезно сразу продумать, какие поля обязательные, а какие опциональные. Минимальный набор:

- Имя — текстовое поле с автозаполнением;
- Email — обязательный атрибут `type="email"` для базовой валидации;
- Тема — селект с предустановленными категориями;
- Сообщение — многострочная `textarea`;
- Чекбокс согласия с политикой обработки данных.

## Шаг 2. Вёрстка компонента

Создадим страницу `src/pages/contact.astro` с формой. Astro позволяет использовать обычный HTML, поэтому фокусируемся на доступности и семантике.

```astro
---
import Layout from "../layouts/BaseLayout.astro";
---

<Layout title="Связаться со мной">
  <section class="mx-auto max-w-2xl space-y-6">
    <header class="space-y-2">
      <h1 class="text-3xl font-semibold">Запрос на сотрудничество</h1>
      <p class="text-slate-500">
        Заполните форму, чтобы обсудить проект, задать вопрос или предложить тему для статьи.
      </p>
    </header>

    <form
      class="space-y-5 rounded-2xl border border-slate-200 bg-white p-8 shadow-sm"
      method="post"
      action="/api/contact"
    >
      <div class="grid gap-5 md:grid-cols-2">
        <label class="flex flex-col gap-2">
          <span class="text-sm font-medium text-slate-700">Имя</span>
          <input
            class="rounded-lg border border-slate-300 px-3 py-2 text-base focus:border-indigo-500 focus:outline-none focus:ring-2 focus:ring-indigo-200"
            type="text"
            name="name"
            autocomplete="name"
            required
          />
        </label>

        <label class="flex flex-col gap-2">
          <span class="text-sm font-medium text-slate-700">Email</span>
          <input
            class="rounded-lg border border-slate-300 px-3 py-2 text-base focus:border-indigo-500 focus:outline-none focus:ring-2 focus:ring-indigo-200"
            type="email"
            name="email"
            autocomplete="email"
            required
          />
        </label>
      </div>

      <label class="flex flex-col gap-2">
        <span class="text-sm font-medium text-slate-700">Тема обращения</span>
        <select
          class="rounded-lg border border-slate-300 px-3 py-2 text-base focus:border-indigo-500 focus:outline-none focus:ring-2 focus:ring-indigo-200"
          name="subject"
          required
        >
          <option value="">Выберите тему</option>
          <option value="consulting">Консультация по фронтенду</option>
          <option value="workshop">Проведение воркшопа</option>
          <option value="partnership">Партнёрство или реклама</option>
          <option value="other">Другое</option>
        </select>
      </label>

      <label class="flex flex-col gap-2">
        <span class="text-sm font-medium text-slate-700">Сообщение</span>
        <textarea
          class="h-32 rounded-lg border border-slate-300 px-3 py-2 text-base focus:border-indigo-500 focus:outline-none focus:ring-2 focus:ring-indigo-200"
          name="message"
          required
        ></textarea>
      </label>

      <label class="flex items-start gap-3 text-sm text-slate-600">
        <input class="mt-1" type="checkbox" name="consent" required />
        <span>
          Соглашаюсь с <a class="text-indigo-600 underline" href="/privacy">политикой обработки данных</a>.
        </span>
      </label>

      <button
        class="inline-flex items-center justify-center gap-2 rounded-lg bg-indigo-600 px-5 py-2.5 text-base font-medium text-white transition hover:bg-indigo-500 focus:outline-none focus:ring-2 focus:ring-indigo-200"
        type="submit"
      >
        Отправить запрос
      </button>
    </form>
  </section>
</Layout>
```

## Шаг 3. Настраиваем обработчик

Astro позволяет создавать эндпоинты в каталоге `src/pages/api`. Добавим файл `contact.ts`:

```ts
import type { APIRoute } from "astro";

export const prerender = false;

export const POST: APIRoute = async ({ request }) => {
  const formData = await request.formData();
  const name = formData.get("name");
  const email = formData.get("email");
  const subject = formData.get("subject");
  const message = formData.get("message");

  if (!name || !email || !subject || !message) {
    return new Response(JSON.stringify({ error: "Не все поля заполнены" }), {
      status: 400,
      headers: { "Content-Type": "application/json" },
    });
  }

  // Здесь можно подключить email-сервис, Notion, Supabase или другую систему хранения
  console.log({ name, email, subject, message });

  return new Response(JSON.stringify({ success: true }), {
    status: 200,
    headers: { "Content-Type": "application/json" },
  });
};
```

В обработчике мы отключаем предварительный рендеринг (`prerender = false`), чтобы функция выполнялась на сервере. После валидации формы можно вызывать сторонние API: отправлять письма, сохранять заявки в базе данных или Slack.

## Шаг 4. Улучшаем UX

Чтобы пользователям было легче заполнять форму, добавляем небольшие улучшения:

- Состояния загрузки и успеха на кнопке отправки (можно подключить небольшой island-компонент на React/Svelte);
- Валидацию на клиенте для подсветки ошибок до отправки (HTML5 или Zod + Astro Island);
- Сообщения об ошибках рядом с полями;
- Логику повторной отправки при сетевых ошибках.

## Шаг 5. Проверяем работу формы

Перед публикацией протестируйте форму локально:

1. Запустите `npm run dev` и откройте страницу `/contact`.
2. Отправьте тестовое сообщение, чтобы убедиться, что API-обработчик возвращает статус `200`.
3. Проверьте браузерную консоль и лог сервера: там должны появиться данные из `formData`.
4. Включите режим «плохой сети» в DevTools, чтобы увидеть, как интерфейс реагирует на задержки.

## Что дальше

Форма — отправная точка. После базовой реализации стоит подключить:

1. Honeypot-поле или CAPTCHA для защиты от ботов;
2. Логику рассылки уведомлений через Resend, Postmark или SendGrid;
3. Интеграцию с CRM, чтобы заявки не терялись;
4. Аналитику по конверсии формы и тепловые карты для поиска проблем.

Теперь у вас есть готовый шаблон формы запроса в Astro: копируйте, адаптируйте под свой стиль и расширяйте, исходя из потребностей проекта.
