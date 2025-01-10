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