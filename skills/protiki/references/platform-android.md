# Android / Material 3 — платформенный скелет протика

**Серый скелет главнее этого файла:** из M3 берём только **структуру, поведение, размеры и словарь**; цвета, тени, блюры, анимации остаются запрещены (правило 2 SKILL.md). Правильная высота navigation bar — платформенная верность; фирменный ripple-градиент — переполиш. Счётчик шагов, лог кликов и панель модератора из probes.md сохраняются поверх любого каркаса.

Дистиллят m3.material.io (состояние 2026, включая M3 Expressive) для генерации серых протиков.

Вьюпорт протика: **360×800, 1dp = 1px** (compact-класс, <600dp: https://m3.material.io/foundations/layout/applying-layout/compact). Шрифт: `font-family: Roboto, system-ui, sans-serif`.

---

## 1. Каркас экрана

**Edge-to-edge — дефолт Android 15+.** Контент рисуется под системными зонами; в протике зоны обозначаем плейсхолдерами: status bar **24px** сверху, жестовая зона **24px** снизу с pill-ручкой **108×4px**. (https://m3.material.io/foundations/layout/understanding-layout/parts-of-layout)

**Top app bar** (https://m3.material.io/components/app-bars/specs):
- **small — 64dp** — дефолт для всех экранов флоу. Слева leading icon (← назад или ☰) 24dp в цели 48×48, заголовок title-large 22sp **слева** (не по центру — центр есть только у варианта center-aligned), справа до 3 action-иконок.
- **medium — 112dp**, **large — 152dp** — заголовок второй строкой под иконками; только для корневых/контентных экранов. При скролле схлопываются в small.
- В Expressive появились flexible-варианты — структурно это те же small/medium/large, полиш не берём.

**Navigation bar внизу** (https://m3.material.io/components/navigation-bar/specs):
- Высота **80dp** (+ жестовый инсет снизу). Expressive-вариант «flexible/short» — **64dp**; для протика бери 80dp — оба верны, 80 читается однозначнее.
- **3–5 пунктов**, всегда icon 24dp + подпись label-medium 12sp. Активный пункт — **pill-индикатор 64×32dp** за иконкой (в сером — плашка #d0d0d0 со скруглением 16px).
- Только для корневых разделов. Внутри флоу (checkout, шаги формы) navigation bar **скрывается**.

**Navigation drawer / rail** (https://m3.material.io/components/navigation-rail/guidelines): drawer **deprecated в Expressive**; на замену — navigation rail (collapsed 80dp / expanded ≥220dp), но rail — паттерн ширины ≥600dp. **В мобильном протике 360dp: navigation bar, точка.** Drawer рисуем только если протикуем существующий продукт, где он уже есть.

**FAB** (https://m3.material.io/components/floating-action-button/specs): **56×56dp**, скругление 16dp, отступ **16dp** от правого и нижнего края, висит НАД navigation bar. Один на экран, только для главного созидательного действия раздела («Новый перевод», «+»). **FAB — не submit**: «Оплатить»/«Продолжить» — это полноширинная кнопка внизу, не FAB.

### CSS-рецепт каркаса

```html
<div class="screen">
  <div class="sys-status">9:30</div>
  <header class="topbar"><button class="icon-btn">←</button><h1>Оплата</h1></header>
  <main class="content"><!-- контент --></main>
  <nav class="navbar"><!-- 3–5 .nav-item --></nav>
  <div class="sys-gesture"><i></i></div>
</div>
```
```css
.screen{width:360px;height:800px;margin:0 auto;display:flex;flex-direction:column;
  background:#fafafa;font-family:Roboto,system-ui,sans-serif;color:#333}
.sys-status{height:24px;font-size:12px;padding:0 16px;display:flex;align-items:center;color:#999}
.topbar{height:64px;display:flex;align-items:center;gap:4px;padding:0 4px}
.topbar h1{font-size:22px;font-weight:400;margin:0 0 0 4px}
.icon-btn{width:48px;height:48px;border:0;background:none;font-size:24px;color:#555}
.icon-btn:active{background:#e0e0e0;border-radius:50%}
.content{flex:1;overflow-y:auto;padding:0 16px}
.navbar{height:80px;background:#eee;display:flex}
.nav-item{flex:1;display:flex;flex-direction:column;align-items:center;justify-content:center;
  gap:4px;font-size:12px;color:#777}
.nav-item .ind{width:64px;height:32px;border-radius:16px;display:flex;
  align-items:center;justify-content:center;font-size:20px}
.nav-item.active .ind{background:#d0d0d0}
.nav-item.active{color:#444;font-weight:500}
.sys-gesture{height:24px;display:flex;align-items:center;justify-content:center}
.sys-gesture i{width:108px;height:4px;border-radius:2px;background:#ccc}
```

---

## 2. Навигационные паттерны

**Назад** (https://developer.android.com/guide/navigation/custom-back/predictive-back-gesture): на Android назад — это системный жест от края (predictive back) ПЛЮС стрелка **←** в leading-позиции top app bar. В протике: стрелка ← на каждом не-корневом экране; в сценарии юзертеста словами предупредить «системный жест назад работает как стрелка». Никогда не «< Назад» текстом — это iOS.

**Bottom sheet** (https://m3.material.io/components/bottom-sheets/guidelines) — главный контейнер мобильных под-флоу на Android (выбор способа оплаты, подтверждение, опции):
- **Modal** — со скримом, блокирует экран; для выбора/подтверждения внутри флоу. **Standard** — без скрима, живёт вместе с экраном (корзина-мини, плеер); в протиках нужен редко.
- Структура: скругление верхних углов **28dp**, **drag handle 32×4dp** по центру сверху (обязателен, если шит закрывается свайпом), контент, отступы 16dp/24dp.
- Список действий на Android — это modal bottom sheet со строками list item **56dp**, НЕ iOS action sheet с кнопками по центру.

```css
.scrim{position:absolute;inset:0;background:rgba(0,0,0,.3)}
.sheet{position:absolute;left:0;right:0;bottom:0;background:#f2f2f2;
  border-radius:28px 28px 0 0;padding:8px 16px 24px}
.sheet .handle{width:32px;height:4px;border-radius:2px;background:#bbb;margin:8px auto 16px}
.sheet .row{height:56px;display:flex;align-items:center;gap:16px;font-size:16px}
```

**Dialog vs bottom sheet** (https://m3.material.io/components/dialogs/guidelines): dialog — **одно решение, максимум 2 действия** («Удалить карту? Отмена / Удалить»), прерывает; всё, что список опций или дополнительный контент, — bottom sheet. Basic dialog: ширина **280–560dp** (в 360-вьюпорте — 328dp, отступы 16), скругление 28dp, padding 24dp, заголовок headline-small 24sp, действия — text buttons **справа**, подтверждающее — последним справа.

**Full-screen dialog** — многошаговая форма/создание объекта: top app bar с **✕ (закрыть) слева**, заголовком и **text-кнопкой действия справа** («Сохранить»). Это и есть паттерн «экран-шаг формы» на Android.

**Snackbar** (https://m3.material.io/components/snackbar/guidelines): подтверждение процесса, о котором пользователь мог забыть, или сетевая ошибка с retry. Высота **48dp** (одна строка), отступы 16dp по бокам и снизу, поверх контента НАД navigation bar/FAB, ≤1 действия текстом, сам исчезает (4–10 с). **Ошибки формы — не snackbar, а inline в поле** (см. §3). Snackbar живёт снизу — не рисуй iOS/веб-тост сверху.

```css
.snackbar{position:absolute;left:16px;right:16px;bottom:96px;min-height:48px;
  background:#555;color:#fff;border-radius:4px;padding:8px 16px;
  display:flex;align-items:center;justify-content:space-between;font-size:14px}
```

---

## 3. Контролы для флоу

**Кнопки** (https://m3.material.io/components/buttons/specs). Иерархия по убыванию веса: **filled → tonal → elevated → outlined → text**. Один filled на экран (главное действие). Дефолтная высота **40dp**, форма — полный pill (radius = высота/2), padding 24dp по бокам, label-large 14sp/500. Expressive добавил размеры XS 32 / S 40 / **M 56** / L 96 / XL 136 — для главного CTA флоу оплаты бери **56dp**. В сером иерархию передаём плотностью заливки:

```css
.btn{height:40px;border-radius:20px;padding:0 24px;font-size:14px;font-weight:500;border:0}
.btn:active{filter:brightness(.9)}          /* суррогат ripple — обязателен */
.btn-filled{background:#555;color:#fff}     /* главное действие */
.btn-tonal{background:#ccc;color:#333}      /* второе по весу */
.btn-outlined{background:none;border:1px solid #999;color:#555}
.btn-text{background:none;color:#555}       /* в диалогах, top bar */
.btn-cta{height:56px;border-radius:28px;width:100%;font-size:16px} /* + .btn-filled */
```

**Цели касания ≥48×48dp**, между целями ≥8dp (https://m3.material.io/foundations/designing/structure). Иконка может быть 24dp, но её хитбокс — 48.

**Text fields** (https://m3.material.io/components/text-fields/specs): контейнер **56dp**; два стиля — **filled** (заливка, скругление 4dp сверху, линия снизу) и **outlined** (рамка, radius 4dp); **один стиль на всю форму**. Label плавающий: пустое поле — 16sp внутри, заполненное/в фокусе — 12sp вверху. Helper/error text — **под полем**, 4dp отступ, 12sp; ошибка = текст под полем + маркер у поля, поле не очищается.

```css
.field{position:relative;margin:12px 0}
.field input{width:100%;height:56px;background:#e8e8e8;border:0;border-bottom:1px solid #999;
  border-radius:4px 4px 0 0;padding:20px 16px 6px;font-size:16px}
.field label{position:absolute;left:16px;top:8px;font-size:12px;color:#777}
.field .support{font-size:12px;color:#777;margin:4px 16px 0}
.field.error input{border-bottom:2px solid #333}
.field.error .support{color:#333;font-weight:500}
```

**Chips** (https://m3.material.io/components/chips/specs): высота **32dp**, скругление **8dp** (не pill — это отличие от кнопок), рамка 1px; filter chip при выборе — заливка + галочка слева. Для выбора суммы/тегов/фильтров.

**Switch** (https://m3.material.io/components/switch/specs): трек **52×32dp**, бегунок 16dp (выкл) → 24dp (вкл). Не iOS 51×31 с белым шариком во всю высоту — у M3 бегунок меняет размер.

**Radio 20dp / Checkbox 18dp** (https://m3.material.io/components/radio-button/specs, https://m3.material.io/components/checkbox/specs) — в цели 48dp, подпись справа, вся строка кликабельна (list item 56dp). Radio — выбор способа оплаты; checkbox — согласия.

**Date/time pickers** (https://m3.material.io/components/date-pickers/guidelines, https://m3.material.io/components/time-pickers/guidelines): дата — **модальный календарь-сетка** (или docked у поля), с переключателем на ручной ввод; время — **циферблат** или input. Никаких iOS-барабанов. В протике достаточно серого календаря-сетки в modal dialog.

---

## 4. Паттерны флоу оформления/платежа

**Многошаговая форма.** M3-паттерн: каждый шаг — экран (или full-screen dialog), прогресс — **linear progress indicator 4dp сразу под top app bar** (https://m3.material.io/components/progress-indicators/guidelines) плюс текст «Шаг 2 из 3» (label-medium) в заголовке или под ним. Отдельного компонента «степпер» в M3 нет — не рисуй нумерованные кружки из M1/веба. **CTA «Продолжить» закреплена внизу** (`position:sticky;bottom:0`, padding 16dp, .btn-cta), назад — стрелка ← в top bar.

```css
.progress{height:4px;background:#ddd}
.progress i{display:block;height:4px;background:#777;width:66%}
.bottom-cta{position:sticky;bottom:0;padding:16px;background:#fafafa}
```

**Ошибки**: валидация — inline в поле (см. §3), при submit скроллим к первому ошибочному полю. Snackbar — только «Нет сети» + действие «Повторить». Никогда alert-диалог на ошибку валидации.

**SMS-код** (https://developers.google.com/identity/sms-retriever/overview): Android подставляет код сам (SMS Retriever / Autofill). Структурно в протике: **одно** поле (не 6 клеточек — они допустимы, но одно поле честнее к автозаполнению), цифровая клавиатура, подпись «Код из SMS подставится автоматически», кнопка активируется при 6 цифрах; в кликабельном протике код «подставляется» сам через 1.5 с — это платформенное поведение, не полиш.

**Google Pay** (https://developers.google.com/pay/api/android/guides/brand-guidelines): кнопка «G Pay» высотой **≥48dp**, pill, на всю ширину, выше/вместо ручного ввода карты; тап открывает **payment sheet — bottom sheet поверх приложения**: строка карты, строка адреса, кнопка оплаты. В сером: кнопка-плейсхолдер с текстом «G Pay» + modal sheet из §2 с двумя строками и CTA. Считать шаги до Aha с учётом шита.

---

## 5. Типографика и сетка

Роли, не размеры (https://m3.material.io/styles/typography/type-scale-tokens). Рабочий набор протика:

| Роль | Размер/вес | Где |
|---|---|---|
| headline-small | 24sp/400 | заголовок контента, диалога |
| title-large | 22sp/400 | top app bar |
| title-medium | 16sp/500 | заголовки карточек, строк |
| body-large | **16sp**/400, line-height 24 | основной текст, ввод в полях |
| body-medium | 14sp/400 | вторичный текст |
| label-large | 14sp/500 | кнопки |
| label-medium | 12sp/500 | подписи nav bar, helper |

Сетка: базовый шаг **4dp**; отступы экрана **16dp** по краям; между связанными элементами 8dp, между блоками 16–24dp (https://m3.material.io/foundations/layout/understanding-layout/spacing). Высоты list items: **56dp** (1 строка) / 72dp (2) / 88dp (3) (https://m3.material.io/components/lists/specs).

---

## 6. Анти-ошибки не-нативности — самопроверка перед выдачей, до вето-десятиминутки

Прогони по каждому экрану перед выдачей протика (шаг 5 порядка работы в probes.md). Каждый пункт = «усреднённый мобайл» врёт про платформу так же, как «рыба» в текстах:

- [ ] **iOS segmented control** вместо M3 tabs (primary tabs: высота 48dp, подчёркивание-индикатор 3dp — https://m3.material.io/components/tabs/guidelines) или connected button group.
- [ ] **«< Назад» текстом** вместо стрелки ← в leading-позиции.
- [ ] **iOS action sheet** (стопка кнопок по центру + Cancel) вместо modal bottom sheet со списком строк.
- [ ] **Заголовок top bar по центру** — у small top app bar заголовок слева.
- [ ] **Нет active-состояния** — ripple в сером не воспроизводим, но `:active{filter:brightness(.9)}` обязателен на всём кликабельном, иначе протик «мёртвый».
- [ ] **Цели <48dp** — иконки 24dp без 48dp-хитбокса.
- [ ] **iOS-switch / пикер-барабан** вместо M3 switch (52×32, бегунок меняет размер) и календаря-сетки.
- [ ] **FAB как submit** («Оплатить» кружком) вместо полноширинной нижней CTA.
- [ ] **Toast/баннер сверху** вместо snackbar снизу; alert на ошибку валидации вместо inline-текста под полем.
- [ ] **Nav bar без подписей** (icon-only — iOS tab bar) или >5 пунктов.
- [ ] **Кнопки с radius 8dp** — M3-кнопки полностью скруглённые (pill); 8dp — это chips и text fields.
- [ ] **Bottom sheet без drag handle** при закрытии свайпом; **navigation drawer в новом флоу** (deprecated, Expressive).

---

## 7. Expo-заметки (ступень «Expo-билд»)

- **react-native-paper v5+** — реализует M3 (MD3): `Appbar.Header` (small/medium/large), `BottomNavigation`, `Button` (mode: contained/contained-tonal/outlined/text), `TextInput` (mode: flat/outlined + `HelperText`), `Chip`, `Switch`, `RadioButton.Item`, `Checkbox.Item`, `Snackbar`, `FAB`, `Dialog`, `ProgressBar`. Серость — через кастомную MD3-тему: все `colors.*` в градации серого, `roundness` по дефолту.
- **Навигация**: `@react-navigation/native` + `@react-navigation/bottom-tabs` (таб-бар под M3 — высота, подписи; активный pill даёт `tabBarActiveBackgroundColor` либо Paper `BottomNavigation.Bar`); стек — `@react-navigation/native-stack` (нативный predictive back бесплатно).
- **Bottom sheet**: `@gorhom/bottom-sheet` — drag handle, snap points, backdrop из коробки.
- **Edge-to-edge**: в новых Expo SDK включён для Android по умолчанию (`react-native-edge-to-edge`); инсеты — `useSafeAreaInsets()` из `react-native-safe-area-context`, нижнюю CTA поднимать на `insets.bottom`.
- **Пикеры**: `@react-native-community/datetimepicker` — системные нативные, ничего не рисовать самим.
- **SMS-код**: `TextInput` c `autoComplete="sms-otp"` и `keyboardType="number-pad"` — реальное автозаполнение на устройстве.
- Клавиатура: `KeyboardAvoidingView behavior="height"` (Android), чтобы CTA не пряталась под клавиатурой.

---

*Источник: m3.material.io, состояние на июль 2026. M3 Expressive учтён структурно (deprecated drawer и bottom app bar, flexible nav bar 64dp, размеры кнопок XS–XL, toolbars как замена bottom app bar); его полиш — shape morphing, физика движения, wavy-индикаторы — в серый скелет не берём.*
