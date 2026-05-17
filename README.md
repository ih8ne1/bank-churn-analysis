# Customer Churn Prediction & Causal Analysis of Retention Triggers

End-to-end ML-проект для мобильного сервиса: предсказание оттока пользователей и **каузальная** оценка эффективности бонусных кампаний через Propensity Score Matching.

## Результаты

- **AUC 0.92** на out-of-time валидации (CatBoost + Optuna)
- **Recall 97%, Precision 85%** — для приоритизации retention-кампаний
- **+14.4% retention** у одного из бонусных триггеров (p = 0.027) после контроля selection bias через PSM
- **−17.5% бюджета на бонусы** высвобождено за счёт обнаружения триггера с обратным эффектом

## Что внутри

### 1. Данные и постановка задачи
~30 000 событий, ~1400 пользователей, 67 дней истории. Сырые event-level логи в PostgreSQL с вложенным JSON в поле `properties`. Таргет оттока — отсутствие активности 21+ день, **порог обоснован через анализ распределения пауз между визитами** (см. график ниже).

![Survival analysis](reports/survival_analysis.png)

### 2. Feature Engineering
28 признаков в четырёх группах:
- **Активность:** дни, события, сессии, возраст аккаунта
- **Обмены:** completed/started/failed/cancelled, completion rate, views without trade
- **Суммы:** средняя, общая, максимальная, число валютных пар
- **Бонусы и реклама:** просмотры, клики, CTR, рефералы

### 3. Сравнение моделей
Пять моделей с честной **time-based валидацией** (нельзя случайно бить временные данные — будет утечка из будущего):

| Модель | AUC | Recall | Precision | F1 |
|---|---|---|---|---|
| Logistic Regression | 0.87 | 0.91 | 0.81 | 0.86 |
| Random Forest | 0.89 | 0.93 | 0.83 | 0.88 |
| XGBoost | 0.91 | 0.95 | 0.84 | 0.89 |
| LightGBM | 0.92 | 0.96 | 0.85 | 0.90 |
| **CatBoost + Optuna** | **0.92** | **0.97** | **0.85** | **0.91** |

Препроцессинг через `ColumnTransformer`, гиперпараметры подобраны через **Optuna (100 trials)** с **TimeSeriesSplit CV**.

![Сравнение моделей](reports/final_comparison.png)

#### Поймали data leakage
Первый замер показал подозрительно высокие AUC (~0.97). Корреляционный анализ показал, что `days_since_last_seen` и `account_age_days` арифметически связаны с таргетом (таргет = `days_since_last_seen ≥ 21`). После удаления — честные метрики, см. таблицу выше.

#### Поймали leakage в CV
Первая попытка тюнинга Optuna запускалась на всём датасете — это привело к подсматриванию в test через одну из train-fold-граней. Правильный подход: тюнинг только на train-части, финальный замер на holdout test.

### 4. Интерпретация через SHAP
![SHAP feature importance](reports/shap_bar.png)

![SHAP beeswarm](reports/shap_beeswarm.png)

### 5. Каузальный анализ через PSM

Это **главная часть проекта**. Наивный анализ показывал, что бонус снижает отток на 11.4 п.п. — но это **selection bias**: бонус получали более активные пользователи, у которых отток ниже изначально.

**Шаги PSM:**
1. Считаем propensity score (вероятность получить бонус) логистической регрессией
2. Проверяем перекрытие распределений (common support)
3. Nearest-neighbor matching с caliper = 0.1
4. Проверяем баланс признаков через SMD < 0.1
5. Считаем честный эффект как разницу средних на сматченных парах + t-test

![PSM overlap](reports/psm_overlap.png)
![PSM balance](reports/psm_balance.png)

**Результаты:**
- Наивная разница в оттоке: **−11.4 п.п.**
- Реальный эффект после контроля selection bias: **−3.6 п.п.**
- **Один триггер** даёт значимый положительный эффект: **+14.4 п.п. retention, p = 0.027**
- **Один триггер** даёт отрицательный эффект **−4.6 п.п.** — отключение высвобождает 17.5% бюджета

## Структура проекта

```
bank-churn-analysis/
├── notebooks/
│   └── churn_analysis.ipynb     # весь анализ end-to-end
├── data/
│   └── README.md                 # описание схемы данных
├── reports/                      # ключевые графики
│   ├── survival_analysis.png
│   ├── final_comparison.png
│   ├── shap_bar.png
│   ├── psm_overlap.png
│   └── psm_balance.png
├── .gitignore
├── README.md
└── requirements.txt
```

## Запуск

```bash
git clone https://github.com/<your-username>/bank-churn-analysis.git
cd bank-churn-analysis

python -m venv .venv
source .venv/bin/activate          # Linux/Mac
.venv\Scripts\activate              # Windows

pip install -r requirements.txt

export DB_PASSWORD=your_password    # или DB_PASSWORD=... в .env
jupyter notebook notebooks/churn_analysis.ipynb
```

> **Note:** Сырые данные коммерческого проекта не публикуются. Структура таблицы и описание схемы — в `data/README.md`. Ноутбук содержит весь код с почищенными выводами; основные графики и таблицы метрик сохранены в `reports/`.

## Стек

**ML / DS:** Python, pandas, NumPy, scikit-learn, LightGBM, CatBoost, XGBoost, Optuna, SHAP

**Causal:** Propensity Score Matching (logistic regression + nearest neighbors), scipy.stats (t-test), баланс через SMD

**Data:** PostgreSQL, SQLAlchemy

**Виз:** matplotlib, seaborn

## Что бы я добавил дальше

- FastAPI-обёртка для модели + Docker для деплоя
- MLflow для трекинга экспериментов
- Двойной DiD или synthetic control для усиления каузального вывода
- Uplift-моделирование (`scikit-uplift`) — оценка эффекта для каждого пользователя
