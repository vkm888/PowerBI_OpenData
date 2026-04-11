# PowerBI_OpenData
PowerBI звіти налаштовані на отримання даних з OpenData

## NBU_exchange_rate.pbix
📊 **звіт Power BI для моніторингу курсів валют НБУ**
аналіз валютних трендів (USD/EUR) у стилі Google Finance. Дашборд автоматизує збір даних та візуалізує динаміку за різні періоди.


<p align="center">
  <img src="/img/nbu_exchange_rate.png" width="600" title="NBU_exchange_rate.pbix">
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
    // 1. СТАТИЧНИЙ КРОК ДЛЯ POWER BI SERVICE
    // Ми просто оголошуємо базовий URL як константу
    BaseUrl = "https://bank.gov.ua/NBU_Exchange/exchange_site",

    // 2. Формуємо дати
    CurrentDate = DateTime.ToText(DateTime.LocalNow(), "yyyyMMdd"),
    StartOfYear = Date.ToText(#date(2021, 1, 1), "yyyyMMdd"),
    
    // 3. Функція з використанням RelativePath (тепер вона посилається на статичний BaseUrl)
    GetCurrencyData = (vCode as text) =>
        let
            Source = Json.Document(Web.Contents(
                BaseUrl, 
                [
                    Query = [
                        valcode = vCode,
                        start = StartOfYear,
                        end = CurrentDate,
                        json = ""
                    ]
                ]
            ))
        in
            Source,

    // 4. Отримуємо дані
    USD = GetCurrencyData("usd"),
    EUR = GetCurrencyData("eur"),
    
    // 5. Об'єднуємо та обробляємо
    CombinedList = List.Combine({USD, EUR}),
    Table = Table.FromList(CombinedList, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    ExpandedTable = Table.ExpandRecordColumn(Table, "Column1", 
        {"exchangedate", "rate", "cc", "txt"}, 
        {"Дата", "Курс", "Код", "Назва"}),

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
