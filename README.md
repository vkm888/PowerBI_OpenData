# PowerBI_OpenData
PowerBI звіти налаштовані на отримання даних з OpenData

## НБУ курс валют.pbix
📊 **звіт Power BI для моніторингу курсів валют НБУ**
аналіз валютних трендів (USD/EUR) у стилі Google Finance. Дашборд автоматизує збір даних та візуалізує динаміку за різні періоди.

[НБУ курс валют.pdf](/img/НБУ%20курс%20валют.pdf)

<p align="center">
  <img src="/img/Screenshot 2026-04-11.png" width="600" title="НБУ курс валют.pbix">
  <br>
</p>

### Ключовий функціонал:
* **Динамічна фільтрація:** Перемикання періодів (1д, 5д, 1міс, ... 2 р., 5 р., MAX) через параметри полів.
* **Фінансовий дизайн:** Контрастні мітки даних, синхронізовані осі та умовне форматування змін курсу.
* **Автоматизація:** Повний цикл від запиту до API до розрахунку денної волатильності в DAX.

API для розробників - https://bank.gov.ua/ua/open-data/api-dev

link на дані в форматі JSON - https://bank.gov.ua/NBU_Exchange/exchange_site?valcode=usd&start=20260407&end=20260411&json

### Power Query (M Language):
Для отримання даних використовується пряме підключення до API НБУ. Приклад коду для завантаження історії:

```powerquery
let
    // 1. Формуємо дати у форматі yyyyMMdd
    CurrentDate = DateTime.ToText(DateTime.LocalNow(), "yyyyMMdd"),
    StartOfYear = Date.ToText(#date(2021, 1, 1), "yyyyMMdd"),
    
    // 2. Функція для отримання даних
    GetCurrencyData = (vCode as text) =>
        let
            Url = "https://bank.gov.ua/NBU_Exchange/exchange_site?valcode=" & vCode & "&start=" & StartOfYear & "&end=" & CurrentDate & "&json",
            Source = Json.Document(Web.Contents(Url))
        in
            Source,
            
    // 3. Отримуємо дані для USD та EUR
    USD = GetCurrencyData("usd"),
    EUR = GetCurrencyData("eur"),
    
    // 4. Об'єднуємо результати
    CombinedList = List.Combine({USD, EUR}),
    
    // 5. Перетворюємо список записів на таблицю
    Table = Table.FromList(CombinedList, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    
    // 6. Розгортаємо колонки
    ExpandedTable = Table.ExpandRecordColumn(Table, "Column1", 
        {"exchangedate", "rate", "cc", "txt"}, 
        {"Дата", "Курс", "Код", "Назва"}),

    // 7. Налаштовуємо типи даних
    FinalTable = Table.TransformColumnTypes(ExpandedTable, {
        {"Дата", type date}, 
        {"Курс", type number}, 
        {"Код", type text}, 
        {"Назва", type text}
    }, "uk-UA")
in
    FinalTable
```

Автор: Віктор Калюта | https://vkm.pp.ua
