# Инструкция: как загрузить проект на GitHub

## 1. Подготовка (один раз)

Если ещё не делал:
- зарегистрирован на github.com
- настроил SSH-ключ (`ssh -T git@github.com` отвечает "Hi <username>!")
- настроил `git config --global user.name/user.email`

## 2. Создать репозиторий на GitHub

1. Иди на github.com → справа сверху "+" → New repository
2. Repository name: **`bank-churn-analysis`**
3. Description: `End-to-end churn prediction with causal analysis (PSM) of retention triggers. LightGBM/CatBoost, Optuna, SHAP.`
4. Public
5. **Не ставь** галочку "Add a README" — у нас уже свой
6. **Не ставь** галочку ".gitignore" — у нас уже свой
7. Create repository

## 3. Залить локальную папку

Положи папку проекта куда-нибудь удобно (например, `~/projects/bank-churn-analysis`).

В терминале (Git Bash на Windows):

```bash
cd путь/к/bank-churn-analysis

# Проверь что .gitignore работает — НЕ должно быть лишних файлов
ls -la

# Инициализация
git init
git branch -M main

# Добавляем remote (замени username на свой реальный GitHub username!)
git remote add origin git@github.com:ТВОЙ_USERNAME/bank-churn-analysis.git

# Первый коммит
git add .
git status                    # ← обязательно проверь что попадает в коммит!
                              #    Не должно быть .pkl, .csv, .env, паролей
git commit -m "initial commit: churn prediction with PSM causal analysis"
git push -u origin main
```

## 4. Что проверить после пуша

1. Открой `https://github.com/ТВОЙ_USERNAME/bank-churn-analysis`
2. README отображается со всеми графиками
3. Кликни на `notebooks/churn_analysis.ipynb` — GitHub его отрендерит
4. Графики в `reports/` отображаются

## 5. Косметика на странице репозитория

1. На странице репо справа от "About" → шестерёнка
2. Topics (теги): `machine-learning`, `churn-prediction`, `causal-inference`,
   `propensity-score-matching`, `lightgbm`, `catboost`, `shap`, `optuna`
3. ✓ Releases — выключить (не нужно)
4. ✓ Packages — выключить
5. Description ввести (из шага 2)

## 6. Запинить в профиле

1. Иди на свой профиль (github.com/ТВОЙ_USERNAME)
2. Customize your pins (справа)
3. Выбери `bank-churn-analysis`
4. Save pins

## 7. Добавить в резюме

В контактах добавить ссылку: `github.com/ТВОЙ_USERNAME`

В описание проекта в резюме добавить: `→ github.com/ТВОЙ_USERNAME/bank-churn-analysis`

## Если что-то сломалось

**`fatal: remote origin already exists`** — у тебя уже привязан remote от прошлой попытки. Удали и добавь заново:
```bash
git remote remove origin
git remote add origin git@github.com:ТВОЙ_USERNAME/bank-churn-analysis.git
```

**`Could not read from remote repository`** — проверь:
```bash
git remote -v
```
Должно показать `git@github.com:...`, не `github.io`, не `github.con`.

**В коммит попал файл с паролем** — НЕ паникуй и НЕ пушь:
```bash
git reset HEAD~1            # откатываем коммит, изменения остаются
# почисти файл с паролем
git add .
git commit -m "..."
```
Если уже запушил — нужно `git filter-branch` или просто удали репозиторий на GitHub и создай заново (только если нет других коммитов).

**Ноутбук не рендерится на GitHub** — обычно проблема в JSON-ошибке. Запусти локально `jupyter notebook` и пересохрани файл.
