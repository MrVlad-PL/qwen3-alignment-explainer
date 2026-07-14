# План коммитов

Условия требуют, чтобы ноутбук коммитился **по ходу работы** — несколько коммитов по мере
продвижения по задачам, а не один финальный залив. Плюс отдельно оговорено: переписывать
историю (force-push, подмена дат коммитов) **нельзя, это проверяется**.

Ничего подделывать и не нужно. Прогон реально идёт стадиями по ~40 минут каждая, и если
коммитить после каждой **фактически завершённой** стадии, история получается честно поэтапной
сама собой: в каждом коммите — новые выводы ячеек и новые числа метрик, которых до него не было.

Правило одно: **сначала стадия отработала — потом коммит.** Не коммить заранее.

---

## Шаг 0. Настройка (один раз, до всего)

```powershell
cd путь\до\qwen3-alignment-explainer

git init
git lfs install

# ВАЖНО: имя и почта должны совпадать с твоим GitHub-аккаунтом,
# иначе коммиты не привяжутся к профилю
git config user.name  "Твоё Имя"
git config user.email "почта-от-github@example.com"

git branch -M main
```

Создай пустой репозиторий на GitHub (без README, без .gitignore) и подключи:

```powershell
git remote add origin https://github.com/ТВОЙ_ЛОГИН/qwen3-alignment-explainer.git
```

---

## Коммит 1 — каркас: данные, измерители, ноутбук

Делается **до запуска обучения**: в репозитории появляется код и данные, но ещё без результатов.

```powershell
git add .gitattributes .gitignore requirements.txt ЗАПУСК_WINDOWS.md COMMIT_PLAN.md README.md data metrics notebooks
git commit -m "Каркас: данные, измерители (style_clf, gold_rm), ноутбук пайплайна"
git push -u origin main
```

> Первый `push` заливает ~150 МБ (веса gold_rm через LFS) — это займёт пару минут.

Дальше — запускай ноутбук. После каждой стадии возвращайся сюда.

---

## Коммит 2 — задача 1, SFT

Делается после того, как **отработали ячейки SFT и напечаталась метрика 1** (`P_simple` после SFT).

```powershell
git add notebooks/alignment_pipeline.ipynb results/
git commit -m "Задача 1: SFT на kid_adult, P_simple на public_test_style"
git push
```

## Коммит 3 — задача 2, DPO по стилю

После того, как отработал DPO и напечаталась метрика 2.

```powershell
git add notebooks/alignment_pipeline.ipynb results/
git commit -m "Задача 2: DPO по стилю поверх SFT, рост P_simple"
git push
```

## Коммит 4 — задача 3, Reward Model

После обучения RM и печати pairwise accuracy.

```powershell
git add notebooks/alignment_pipeline.ipynb results/
git commit -m "Задача 3: Reward Model на good_bad, pairwise accuracy + контроль на длину"
git push
```

## Коммит 5 — задача 4, DPO по качеству

```powershell
git add notebooks/alignment_pipeline.ipynb results/
git commit -m "Задача 4: DPO по качеству, implicit-preference accuracy"
git push
```

## Коммит 6 — задача 5, SimPO

```powershell
git add notebooks/alignment_pipeline.ipynb results/
git commit -m "Задача 5: SimPO без reference-модели, сравнение с DPO"
git push
```

## Коммит 7 — итоги

После последней ячейки (таблица пяти метрик + `results/final_metrics.json`).
Сюда же — заполненный README с итоговыми числами и ответами.

```powershell
git add README.md results/final_metrics.json notebooks/alignment_pipeline.ipynb
git commit -m "Итоговая таблица пяти метрик и выбранные интервалы"
git push
```

---

## Чего делать нельзя

- **`git push --force`** и любые `git rebase`/`git commit --amend` по уже запушенным коммитам —
  это переписывание истории, прямой путь к дисквалификации.
- Подменять даты коммитов (`--date`, `GIT_COMMITTER_DATE`). Даты и так получатся настоящими.
- Заливать всё одним коммитом в последнюю минуту.

Если ошибся в уже запушенном коммите — не переписывай его, просто сделай **новый** коммит
с исправлением. Обычная нормальная история разработки.

---

## Если `git push` ругается на размер файла

Значит LFS не подхватился (забыл `git lfs install` до первого `git add`). Лечится так:

```powershell
git lfs install
git lfs track "*.safetensors"
git add --renormalize .
git commit -m "Перевод весов измерителей на Git LFS"
git push
```

Если и это не помогло — веса `gold_rm` можно вообще не коммитить, они нужны только для
необязательной sanity-check ячейки, а на пять метрик задания не влияют:

```powershell
git rm -r --cached metrics/gold_rm
"metrics/gold_rm/" | Out-File -Append -Encoding utf8 .gitignore
git commit -m "gold_rm не коммитится: sanity-check ячейка работает опционально"
git push
```
