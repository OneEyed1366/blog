+++
date = "2022-10-10T21:12:57+03:00"
title = "Мозги на прокачку: асинхронная асинхронность и немного race-condition'ов"
description = "Немного 'брюзжания' на тему не всегда очевидного поведения асинхронности в JS"
tags = ["study", "JavaScript", "async", "web", "event-loop"]
keywords = ["async-await", "race-condition", "engines"]
showFullContent = false
readingTime = true 
hideComments = true 
draft = true
+++

## Что?

> `Асинхронность` в JavaScript - это
> `способность исполнять команды, не блокируя возможность взаимодействия`
> с приложением

## Как?

Это возможно благодаря тому, что JavaScript, хоть он и "как-бы-однопоточный"
([Web-Worker'ы](https://developer.mozilla.org/ru/docs/Web/API/Web_Workers_API/Using_web_workers) - привет!),
имеет `под капотом не только себя` любимого, `но и`, например, `Web API`.

## А поточнее?
