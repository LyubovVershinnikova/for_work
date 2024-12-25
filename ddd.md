```
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();

  await page.goto('https://example.com');

  // Проверяем содержимое body
  const isPageNotEmpty = await page.evaluate(() => {
    return document.body && document.body.innerText.trim().length > 0;
  });

  console.log(isPageNotEmpty ? 'Страница не пустая' : 'Страница пустая');

  await browser.close();
})();

```