# COVID-19 Data Analysis

## Описание на проекта

Това хранилище съдържа документация за анализ на информационната система за случаи, свързани с COVID-19 от WHO (Световната Здравна Организация) и CDC (Център за контрол и превенция на заболяванията), налична в маркетплейса на Snowflake. Целта на проекта е да се опишат събраните данни и тяхната структура.

---

## Описание на таблиците

### **WHO_DAILY_REPORT**
- `CASES` – Брой нови случаи за деня
- `CASES_TOTAL` – Общо натрупани случаи
- `CASES_TOTAL_PER_100000` – Общо случаи на 100 000 население
- `COUNTRY_REGION` – Държава/регион
- `DATE` – Дата на записване
- `DEATHS` – Брой нови смъртни случаи
- `DEATHS_TOTAL` – Общо смъртни случаи
- `DEATHS_TOTAL_PER_100000` – Смъртни случаи на 100 000 население
- `ISO3166_1` – Код на държавата (ISO 3166-1)
- `TRANSMISSION_CLASSIFICATION` – Класификация на предаването на вируса

### **WHO_SITUATION_REPORTS**
- `CASES_NEW` – Нови случаи от последния доклад
- `COUNTRY` – Име на държавата
- `COUNTRY_REGION` – Държава/регион според ISO 3166-1
- `DATE` – Дата на доклада
- `DAYS_SINCE_LAST_REPORTED_CASE` – Брой дни от последния регистриран случай
- `DEATHS` – Общ брой починали за съответната география
- `DEATHS_NEW` – Нови смъртни случаи от последния доклад
- `ISO3166_1` – ISO код на държавата
- `LAST_REPORTED_FLAG` – Флаг, показващ дали има нови данни
- `LAST_UPDATE_DATE` – Дата на последна актуализация
- `SITUATION_REPORT_NAME` – Име на доклада на СЗО
- `SITUATION_REPORT_URL` – URL към доклада на СЗО
- `TOTAL_CASES` – Общо регистрирани случаи

### **CDC_TESTING**
- `DATE` – Дата на тестване
- `INCONCLUSIVE` – Брой неубедителни тестове (неопределени резултати)
- `ISO3166_1` – Код на държавата (ISO 3166-1)
- `ISO3166_2` – Код на региона (ISO 3166-2, ако е наличен)
- `NEGATIVE` – Брой отрицателни тестове
- `POSITIVE` – Брой положителни тестове

---

## Use Cases (Приложения)

### **1. Анализ на смъртността спрямо броя на случаите по държави**  
```sql
-- Изчислява общия брой случаи и смъртност по държави
SELECT 
    COUNTRY_REGION, 
    SUM(DEATHS_TOTAL) AS TOTAL_DEATHS, 
    SUM(CASES_TOTAL) AS TOTAL_CASES
FROM WHO_DAILY_REPORT
GROUP BY COUNTRY_REGION
ORDER BY TOTAL_DEATHS DESC;
```

### **2. Разпределение на случаите по държавни региони**  
```sql
-- Групиране на общия брой случаи по държави подредени по общ брой случаи 
SELECT 
    COUNTRY_REGION, 
    SUM(TOTAL_CASES) AS TOTAL_CASES
FROM WHO_SITUATION_REPORTS
GROUP BY COUNTRY_REGION
ORDER BY TOTAL_CASES DESC;

```

### **3. Положителни тестове спрямо отрицателни в USA**  
```sql
-- Анализ на процента положителни тестове спрям отрицателните в Щатите
SELECT 
    CT.ISO3166_1 AS COUNTRY_CODE, 
    SUM(CT.POSITIVE) AS TOTAL_POSITIVE_TESTS, 
    SUM(CT.NEGATIVE) AS TOTAL_NEGATIVE_TESTS, 
    (SUM(CT.POSITIVE) * 100.0 / (SUM(CT.POSITIVE) + SUM(CT.NEGATIVE))) AS POSITIVE_TEST_RATE
FROM CDC_TESTING CT
GROUP BY CT.ISO3166_1
ORDER BY POSITIVE_TEST_RATE DESC;

```

### **4. Тенденции в броя на новите случаи и ефекта на локдаун мерките**  
```sql
-- Анализ на броя на новите случаи преди и след първите локдаун мерки
SELECT 
    COUNTRY_REGION, 
    DATE, 
    SUM(CASES_NEW) AS DAILY_NEW_CASES
FROM WHO_SITUATION_REPORTS
WHERE DATE BETWEEN '2020-03-01' AND '2020-05-31'  -- Период на първия локдаун (Ориентировъчно държавите обявиха такъв през март месец)
AND TRANSMISSION_CLASSIFICATION IN ('Community transmission', 'Clusters of cases', 'Sporadic cases') 
GROUP BY COUNTRY_REGION, DATE
ORDER BY COUNTRY_REGION, DATE;

```
