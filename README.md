# 11ty-picture
По умолчанию, eleventy умеет работать с маркдауном и вставлять картинки как `![alt](./url)`. В этом случае картинка вставляется в код вовнутрь параграфа:

```html
<p>
  <img src="./img/a086c775-285.webp" alt="Ubuntu logo">
</p>
```

Кроме того, вам надо либо отдельно настроить работу движка с картинками, либо использовать картинки, уже лежащие на сервере и брать пути к ним.

## Вариант решения проблемы
Разработчики движка предлагают [следующее решение](https://www.11ty.dev/docs/plugins/image/): вы можете вставить изображение в качестве непосредственно картинки без обёрток, либо более классного элемента `<picture>`, ну или `<figure>`, чтобы дополнять ваши изображения подписями, различными форматами и размерами. 

Плюс вы получите возможность размещать свои изображения сразу на сервер.

[Live](https://furtivite-11ty-pictures.netlify.app)

### Шаг 1. Установите зависимости
Вам потребуется 

```bash
npm install @11ty/eleventy-img
```

### Шаг 2. Создайте файл .eleventy.js
Если у вас его ещё нет. 

```javascript
// Используем установленную зависимость
const Image = require("@11ty/eleventy-img");

// Добавляем асинхронную функцию
async function imageShortcode(src, alt, sizes = "100vw") {
  if(alt === undefined) {
    // Если забыли alt, получите ошибку
    throw new Error(`Missing \`alt\` on responsiveimage from: ${src}`);
  }

  let metadata = await Image(src, {
    widths: [300, 600],
    // Указываем форматы, которые хотим использовать
    formats: ['webp', 'jpeg'],
    // Указываем, куда складывать готовые картинки 
    // По умолчанию они складываются в обычный /img, а не в собранные файлы
    outputDir: "./build/img",
  });

  let lowsrc = metadata.jpeg[0];

  // Формируем свой правильный html-тег
  return `<picture>
    ${Object.values(metadata).map(imageFormat => {
      return `  <source type="${imageFormat[0].sourceType}" srcset="${imageFormat.map(entry => entry.srcset).join(", ")}" sizes="${sizes}">`;
    }).join("\n")}
      <img
        src="${lowsrc.url}"
        width="${lowsrc.width}"
        height="${lowsrc.height}"
        alt="${alt}"
        loading="lazy"
        decoding="async">
    </picture>`;
}


module.exports = eleventyConfig => {
  // Указываем в каком виде вы хотите использовать шорткод
  eleventyConfig.addNunjucksAsyncShortcode("image", imageShortcode);
  eleventyConfig.addLiquidShortcode("image", imageShortcode);
  eleventyConfig.addJavaScriptFunction("image", imageShortcode);
  
  // Прочие настройки по сборке проекта
  return {
    dir: {
      input: 'src',
      output: 'build',
      includes: "includes",
      layouts: "layouts",
      data: "data",
    }
  };
};
```

### Шаг третий. Пользуемся
```markdown
Пишем текст, а ниже помещаем картинку с нужными параметрами 

{% image "./src/images/cat.jpg", "photo of my cat" %}
{% image "./src/images/cat.jpg", "photo of my cat", "(min-width: 30em) 50vw, 100vw" %}
```

**Обратите внимание**
Для Nunjucks-шаблона _обязательно_ ставить запятые между аргументами, для Liquid-шаблонов — это опционально.

## Что вы получите в итоге
В итоге вы получите в `/build/` директории ваш html-файл, вашу директорию `/img/` с картинками, которые вы использовали уже в нужных форматах. Названия файлов изменятся на кастомные, но это можно поменять. Информация так же есть [в документации](https://www.11ty.dev/docs/plugins/image/).

**Обратите внимание**
[![Netlify Status](https://api.netlify.com/api/v1/badges/9ebc05e5-4028-4887-86d3-590f4ecb091d/deploy-status)](https://app.netlify.com/sites/furtivite-11ty-pictures/deploys)

Если вы будете пользоваться [Netlify](https://www.netlify.com) для сборки и публикации проекта, не забудьте заменить директорию сборки с `_site` на `build`, если она отличается от стандартой. У меня в примере она написана ниже надписи «Прочие настройки по сборке проекта»
