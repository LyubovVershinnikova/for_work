Для проверки текста внутри элемента `<p>` с классом `text-header` на открывшейся странице с использованием Puppeteer можно сделать следующее:

### Пример кода:

```javascript
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch({ headless: true }); // Открываем браузер
  const page = await browser.newPage();

  // Открываем целевую страницу
  await page.goto('https://example.com');

  // Ждём, пока элемент с классом `text-header` появится на странице
  await page.waitForSelector('p.text-header');

  // Извлекаем текст из элемента
  const text = await page.$eval('p.text-header', el => el.textContent);

  // Проверяем текст
  if (text.includes('Искомый текст')) {
    console.log('Текст найден!');
  } else {
    console.log('Текст не найден.');
  }

  // Закрываем браузер
  await browser.close();
})();
```

### Разбор:

1. **`await page.goto(url)`** — Переходим на целевую страницу.
2. **`await page.waitForSelector('p.text-header')`** — Ждём появления элемента `<p>` с классом `text-header`.
3. **`await page.$eval(selector, callback)`** — Извлекаем содержимое элемента с помощью переданного селектора.
4. **Проверка текста:** Сравниваем извлечённый текст с искомым.

### Альтернатива: Проверка текста напрямую через XPath

Если требуется точное соответствие текста, можно использовать XPath:

```javascript
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch({ headless: true });
  const page = await browser.newPage();

  await page.goto('https://example.com');

  // Ищем элемент через XPath
  const [element] = await page.$x("//p[@class='text-header' and contains(text(), 'Искомый текст')]");

  if (element) {
    console.log('Текст найден!');
  } else {
    console.log('Текст не найден.');
  }

  await browser.close();
})();
```

### Полезные советы:

- Если текст должен совпадать полностью, используйте `textContent.trim() === 'Искомый текст'` для сравнения.
- Для сложных динамических страниц добавьте ожидания, например, через `waitForTimeout` или другие триггеры.




Проблема связана с тем, что Puppeteer может начать рендеринг PDF до полного обновления содержимого страницы. Чтобы избежать этого, можно использовать дополнительные проверки перед созданием PDF, чтобы убедиться, что данные на странице действительно обновились.

Вот несколько подходов для решения проблемы:

---

### 1. **Использовать `waitForNavigation`**

Если вы переходите на новую страницу с помощью ссылки или редиректа, добавьте ожидание загрузки новой страницы:

```javascript
await page.goto(url, { waitUntil: 'networkidle2' }); // Ждем, пока сетевые запросы завершатся
await page.waitForSelector(selector); // Ждем появления элемента
await page.pdf({ path: 'page.pdf', format: 'A4' }); // Сохраняем PDF
```

Здесь параметр `waitUntil: 'networkidle2'` заставляет Puppeteer ждать, пока все сетевые запросы завершатся.

---

### 2. **Ждать изменения содержимого страницы**

Если данные обновляются динамически (например, через AJAX), добавьте проверку на изменение текста или атрибутов. Например:

```javascript
const previousContent = await page.$eval(selector, el => el.textContent);

// Ждем, пока содержимое элемента изменится
await page.waitForFunction(
  (selector, previousContent) => {
    const element = document.querySelector(selector);
    return element && element.textContent !== previousContent;
  },
  {}, // Передаем настройки
  selector,
  previousContent
);

// Сохраняем PDF после изменения содержимого
await page.pdf({ path: 'page.pdf', format: 'A4' });
```

---

### 3. **Использовать тайм-ауты (как запасной вариант)**

Если обновление страницы занимает фиксированное время, можно добавить искусственную задержку:

```javascript
await page.goto(url, { waitUntil: 'networkidle2' });
await page.waitForTimeout(2000); // Ждем 2 секунды
await page.pdf({ path: 'page.pdf', format: 'A4' });
```

Однако это менее надежный способ и подходит только для предсказуемых сценариев.

---

### 4. **Убедиться, что все запросы завершены**

Если есть подозрения, что AJAX-запросы еще продолжаются, можно дождаться их завершения:

```javascript
await page.goto(url, { waitUntil: 'networkidle2' });
await page.waitForSelector(selector); // Убедиться, что элемент появился

// Ждем завершения всех сетевых запросов
await page.evaluate(() => {
  return new Promise(resolve => {
    const checkRequests = () => {
      if (window.performance.getEntriesByType('resource').filter(r => !r.responseEnd).length === 0) {
        resolve();
      } else {
        setTimeout(checkRequests, 100);
      }
    };
    checkRequests();
  });
});

// Сохраняем PDF после завершения запросов
await page.pdf({ path: 'page.pdf', format: 'A4' });
```

---

### Общий подход (универсальный пример):

Объединим все рекомендации:

```javascript
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch({ headless: true });
  const page = await browser.newPage();

  const urls = ['https://example.com/page1', 'https://example.com/page2']; // Список страниц

  for (const [index, url] of urls.entries()) {
    await page.goto(url, { waitUntil: 'networkidle2' }); // Открываем страницу и ждем загрузки

    // Ждем, пока элемент с нужным содержимым появится
    await page.waitForSelector('p.text-header');
    
    // Дополнительно ждем изменения содержимого (опционально)
    const previousContent = await page.$eval('p.text-header', el => el.textContent);
    await page.waitForFunction(
      (selector, previousContent) => {
        const element = document.querySelector(selector);
        return element && element.textContent !== previousContent;
      },
      {},
      'p.text-header',
      previousContent
    );

    // Сохраняем страницу в PDF
    await page.pdf({ path: `page${index + 1}.pdf`, format: 'A4' });
  }

  await browser.close();
})();
```

---

### Итог

Используйте комбинацию:

1. `waitUntil: 'networkidle2'` — чтобы дождаться окончания загрузки страницы.
2. `waitForSelector` — чтобы убедиться, что элемент появился.
3. `waitForFunction` — чтобы проверить обновление содержимого элемента.

Эти шаги гарантируют, что PDF сохранится с актуальными данными.