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



```
for (const [index, row] of rows.entries()) {
  const rowStartTime = Date.now(); // Начало обработки строки

  try {
    const [word1, word2] = Object.values(row).slice(0, 2); // Берем первые два слова
    const url = `https://example.com/${word1}/${word2}`; // Собираем URL
    logger.info(`Обрабатываем строку ${index + 1}: ${JSON.stringify(row)}`);
    logger.info(`[Строка ${index + 1}] Открываем URL: ${url}`);
    await page.goto(url);

    // Проверяем наличие элемента <app-payslip-v2>
    try {
      logger.info(`[Строка ${index + 1}] Ожидаем элемент <app-payslip-v2> на странице...`);
      await page.waitForSelector('app-payslip-v2', { timeout: 10000 }); // Таймаут 10 секунд
      logger.info(`[Строка ${index + 1}] Элемент <app-payslip-v2> найден на странице.`);
    } catch (error) {
      logger.warn(`[Строка ${index + 1}] Элемент <app-payslip-v2> не найден или истекло время ожидания.`);
      continue; // Пропустить текущую строку, если элемент не найден
    }

    const fileName = `${word1}_${word2}`; // Ожидаемое имя файла
    logger.info(`[Строка ${index + 1}] Ожидаем файл с именем: ${fileName}`);
    await waitForFile(downloadFolder, fileName);

    logger.info(`[Строка ${index + 1}] Файл найден: ${fileName}.`);
  } catch (error) {
    logger.error(`[Строка ${index + 1}] Ошибка при обработке строки: ${error.message}`);
  }

  const rowEndTime = Date.now(); // Конец обработки строки
  logger.info(`[Строка ${index + 1}] Обработка заняла ${(rowEndTime - rowStartTime) / 1000} секунд.`);
}

```