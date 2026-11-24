# niri (личный форк `demoj1/niri`)

## Remotes и структура каталогов

- `origin` — `https://github.com/niri-wm/niri` (апстрим)
- `fork` — `git@github.com:demoj1/niri.git` (личный форк на GitHub)

Рабочие папки в `~/:pet/language/rust/`:
- `niri/` — основное дерево, ветка `main` (`origin/main` + наши коммиты всегда сверху, трекает `fork/main`).
- `niri-bp/` — worktree ветки `backport-26.04` (те же патчи поверх релизного тега `v26.04` для снятия `000*.patch`).
- `niri-arch/` — worktree ветки `arch` (трекает `fork/arch`: Arch `PKGBUILD`, `.SRCINFO`, `README.md`, `0001..0004.patch`).
- `niri-pkg/` — рабочая папка сборки и установки Arch-пакета (`makepkg -si`).

## Что поддерживается в форке поверх апстрима

Актуальный лог отличий ведется наверху `README.md`:
1. **`layout: Draw active decorations only on the focused workspace`** — в overview активный focus-ring рисуется только на фокусном воркспейсе.
2. **`overview: Add focus-ring and shadow overrides`** — секция `overview { focus-ring {…} shadow {…} }` поверх `layout`.
3. **`Add block-out-blur window rule`** — правило `block-out-blur { passes … offset … }` (размытие заблокированных окон в скринкастах/скриншотах вместо черного прямоугольника).

*(Фикс #3295 `Don't send frame callbacks while monitors are powered off` уже попал в апстрим в коммите `78b7adfe` и снят с `main`, но по-прежнему бэкпортирован в `niri-bp`/`niri-arch` для `v26.04` до следующего релиза; путь физического выключения кнопкой #4461/#4550 остаётся открытым.)*

Новые фичи делаем строго опциональными (без новых опций в конфиге поведение совпадает с апстримом). На каждый новый или убранный патч обновляем таблицу `Fork changelog` в `README.md`.

## Синк с апстримом и обновление форка на GitHub

Делать периодически (раз в 1–2 недели или по запросу пользователя):

1. Синкануть `main` форка поверх свежего апстрима:
   ```sh
   git fetch origin
   git checkout main
   git rebase origin/main
   ```
   При конфликтах или если какой-то фикс уже попал в апстрим — адаптировать или дропнуть соответствующий коммит (и обновить таблицу в `README.md`).
2. Проверить сборку и тесты:
   ```sh
   cargo test --all --exclude niri-visual-tests
   ```
3. **Обязательно обновить личный форк на GitHub** (`fork`):
   - после `rebase` на `origin/main`: `git push --force-with-lease fork main`
   - после обычных локальных коммитов: `git push fork main`
4. При необходимости обновить релизный Arch-пакет:
   - перенести актуальные патчи в `niri-bp` (`backport-26.04`) и снять патчи (`git format-patch`) в `niri-arch`;
   - обновить `pkgrel`, контрольные суммы (`updpkgsums`) и `.SRCINFO` в `niri-arch`, закоммитить и пушнуть в `fork/arch` (`git push fork arch`);
   - собрать и поставить пакет из `niri-pkg` через `makepkg -si`.
   *(В `/etc/pacman.conf` пакет `niri` добавлен в `IgnorePkg` — убрать оттуда, когда фикс #3295 попадет в релиз апстрима.)*
