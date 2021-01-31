---
layout: base.njk
title: Вставляем картинку правильно
---

Эта картинка вставлена стандартно через элементы маркдауна `![]()`

![Ubuntu logo](./img/a086c775-285.webp)

А вот ниже вставлена как `<picture>`

{% image "./src/imgs/circle-of-friends-web/JPEG/cof_orange_hex.jpg", "Ubuntu logo" %}

