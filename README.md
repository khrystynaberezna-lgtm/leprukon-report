# ЛЕПРЕКОН — автоматичний звіт Bolt Food UA

Пакет для автооновлюваного звіту партнера **ЛЕПРЕКОН** (заклади «Лепрекон»).
Побудований за методичкою [Stores-internal-weekly-report](https://github.com/khrystynaberezna-lgtm/Stores-internal-weekly-report/tree/main/docs) (шаблон Hop Hey: вкладки Monthly + Weekly).

## Параметри звіту

| Параметр | Значення | Де |
|----------|----------|-----|
| `PARTNER_NAME` (group_name у Databricks) | `LEPRUKON` ⚠️ **потребує підтвердження** | `generate_report.py` |
| `PARTNER_DISPLAY` (заголовок) | `ЛЕПРЕКОН` | `generate_report.py`, `template.html` |
| `DATA_START` | `2025-01-01` | `generate_report.py` |
| Initials (лого) | `LP` | `template.html` |
| Slug публічного репо / Pages | `leprukon-report` | `publish.sh`, workflow |
| Live URL (після публікації) | `https://khrystynaberezna-lgtm.github.io/leprukon-report/` | — |

## Файли

```
LEPRUKON/
├── generate_report.py                 # тягне дані з Databricks → index.html
├── template.html                      # HTML-шаблон (брендинг Bolt, Chart.js)
├── publish.sh                         # генерація + git push у публічний репо
├── requirements.txt                   # databricks-sql-connector
├── .env.example                       # шаблон конфігу (скопіювати в .env, додати PAT)
├── .gitignore                         # .env та report_data.json не комітяться
├── cursor-rule.mdc                    # правило Cursor для цього звіту
├── README.md                          # цей файл
└── .github/workflows/update-report.yml  # автооновлення щопонеділка 05:00 UTC
```

## Що ще потрібно зробити, щоб звіт «ожив» (3 кроки)

### 1. Підтвердити `group_name` у Databricks
Коли SQL Warehouse доступний, виконай:
```sql
SELECT group_name, COUNT(*) AS stores
FROM main.ng_delivery.dim_provider_v2
WHERE country_code='ua' AND (LOWER(group_name) LIKE '%lepr%' OR group_name LIKE '%Лепр%')
GROUP BY group_name ORDER BY stores DESC LIMIT 50;
```
Якщо точне значення ≠ `LEPRUKON` — заміни `PARTNER_NAME` у `generate_report.py`.

### 2. Додати Databricks PAT
```bash
cd LEPRUKON
cp .env.example .env
# у .env встав DATABRICKS_TOKEN=dapi... (Databricks → User Settings → Developer → Access tokens)
```

### 3. Перша генерація + публікація
```bash
cd LEPRUKON
./publish.sh
```
`publish.sh` встановить залежності, згенерує `index.html` зі свіжими даними і запушить його у публічний репо `leprukon-report` (GitHub Pages).

> Для публікації потрібні: встановлений GitHub CLI (`brew install gh` + `gh auth login`),
> створений публічний репо `leprukon-report` з увімкненими Pages (main /root),
> та 3 секрети у репо: `DATABRICKS_HOST`, `DATABRICKS_WAREHOUSE_ID`, `DATABRICKS_TOKEN`
> (див. крок 5–6 методички `02-partner-report-guide.md`).

## Автооновлення

`.github/workflows/update-report.yml` запускається **щопонеділка о 05:00 UTC (08:00 Київ)**
або вручну (Actions → Run workflow) і перегенеровує `index.html` зі свіжими даними Databricks.
