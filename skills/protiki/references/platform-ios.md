# iOS-скелет — дистиллят Apple HIG для протиков

**Серый скелет главнее этого файла:** из HIG берём только структуру, поведение, размеры и словарь; цвета, тени, блюр, анимации остаются запрещены (правило 2 SKILL.md). Правильная высота tab bar — платформенная верность; фирменный полиш — переполиш. Счётчик шагов, лог кликов и панель модератора из probes.md сохраняются поверх любого каркаса.

Справочник агента-генератора: как собрать платформенно-верный СЕРЫЙ скелет iOS-экрана (HTML-протик в мобильном вьюпорте или Expo). В HTML-протике 1pt = 1px CSS. Сверено с HIG (актуальная редакция, iOS 26, июль 2026).

## 1. Каркас экрана

HIG: https://developer.apple.com/design/human-interface-guidelines/layout

- Базовый вьюпорт протика: **393×852** (логические pt iPhone 15/16/17). Допустим 390×844.
- Safe areas (iPhone с Dynamic Island): **top inset 59pt**, **bottom inset 34pt** (home indicator). Старые с notch: top 47pt. SE: top 20pt, bottom 0. По умолчанию бери 59/34.
- **Status bar** — всегда есть: время слева, батарея/сеть справа; в протике — серая полоска-плейсхолдер внутри верхних 59pt.
- **Navigation bar**: стандартная высота **44pt** (под статус-баром). Слева — back «‹ Назад» (chevron + подпись предыдущего экрана), по центру — заголовок 17pt semibold, справа — одно действие (текст или иконка). Large title: под 44pt-полосой строка заголовка 34pt bold с выравниванием влево, суммарно ~**96pt**; при скролле large title схлопывается в центр 44pt-полосы (в протике достаточно статичного варианта). HIG: https://developer.apple.com/design/human-interface-guidelines/navigation-bars
- **Tab bar**: высота **49pt** + 34pt home indicator = 83pt снизу. **2–5 вкладок**, каждая = иконка ~25×25 + подпись 10pt под ней, активная выделена (в сером — залито темнее). Tab bar — только верхний уровень навигации; виден на всех корневых экранах, исчезать при push внутрь секции может, но не обязан. iOS 26 структурно: search-вкладка может стоять отдельной «таблеткой» справа от остальных; таб-бар минимизируется при скролле — в протике не имитируй, это полиш. HIG: https://developer.apple.com/design/human-interface-guidelines/tab-bars
- **Edge-to-edge**: контент скроллится ПОД барами до краёв экрана; паддинги скролл-области = высоты баров. Ничего интерактивного в нижних 34pt.

CSS-рецепт каркаса (серый скелет):

```html
<div class="phone"><!-- 393×852 -->
  <div class="statusbar"></div>
  <header class="navbar"><button class="back">‹ Назад</button><h1>Заголовок</h1><button class="action">Готово</button></header>
  <main class="content">…</main>
  <nav class="tabbar">
    <button class="tab active"><span class="ico"></span>Главная</button>
    <button class="tab"><span class="ico"></span>Поиск</button>
  </nav>
  <div class="home-indicator"></div>
</div>
```
```css
.phone{width:393px;height:852px;margin:auto;background:#f2f2f2;position:relative;overflow:hidden;
  font-family:-apple-system,system-ui,'SF Pro Text',sans-serif;display:flex;flex-direction:column}
.statusbar{height:59px;flex:none;display:flex;justify-content:space-between;padding:20px 24px 0;color:#888;font-size:15px}
.navbar{height:44px;flex:none;display:flex;align-items:center;justify-content:space-between;
  padding:0 16px;background:#e8e8e8;border-bottom:1px solid #d0d0d0}
.navbar h1{font-size:17px;font-weight:600;position:absolute;left:50%;transform:translateX(-50%)}
.back{font-size:17px;color:#555;min-height:44px}
.content{flex:1;overflow-y:auto;padding:16px}
.tabbar{height:49px;flex:none;display:flex;background:#e8e8e8;border-top:1px solid #d0d0d0}
.tab{flex:1;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:2px;font-size:10px;color:#999}
.tab.active{color:#444}.ico{width:25px;height:25px;border-radius:6px;background:currentColor;opacity:.5}
.home-indicator{height:34px;flex:none;display:flex;align-items:center;justify-content:center}
.home-indicator::after{content:'';width:134px;height:5px;border-radius:3px;background:#c0c0c0}
```

## 2. Навигационные паттерны

HIG: https://developer.apple.com/design/human-interface-guidelines/modality , …/navigation-bars

- **Push (стек)**: движение вглубь одной задачи. Новый экран въезжает справа, back «‹ Назад» слева, свайп-назад от левого края. В протике свайп можно не делать, но **кнопка back обязана работать** (реально переключать экран, не декорация). Back — всегда chevron ‹, никогда стрелка ←.
- **Modal sheet (.pageSheet)**: карточка снизу, скруглённый верх, **grabber 36×5pt** по центру верха, верхний край ~10pt ниже статус-бара — предыдущий экран виден подложкой (в сером — затемни его). Detents: medium (пол-экрана) / large. Закрытие: свайп вниз или кнопка. В шите свой заголовок: «Отменить» слева ИЛИ «✕» **справа**, действие «Готово» справа.
- **Правило sheet vs push**: push — следующий шаг ТОЙ ЖЕ задачи (шаг 2 оформления); sheet — самостоятельная подзадача/прерывание с явным завершением (фильтры, создание объекта, выбор адреса). Многошаговый флоу целиком может жить внутри sheet со своим push-стеком.
- **Full-screen modal**: только для иммерсивных задач (камера, просмотр фото, видео). Не дефолт для форм.
- **Alert**: центр экрана, заголовок + текст + 1–2 кнопки (вертикально, если 3+; отмена снизу/слева). Только для критичной информации, требующей решения. HIG: …/alerts
- **Action sheet**: список действий снизу, «Отменить» отдельной кнопкой ниже. **Подтверждение разрушающего действия — всегда action sheet снизу** (разрушающий пункт первым, в нативе красный — в сером пометь «(удалить)»), не alert. HIG: …/action-sheets

```css
.sheet{position:absolute;left:0;right:0;bottom:0;top:69px;background:#ececec;border-radius:10px 10px 0 0}
.grabber{width:36px;height:5px;border-radius:3px;background:#c8c8c8;margin:6px auto}
.scrim{position:absolute;inset:0;background:rgba(0,0,0,.2)}
```

## 3. Контролы для флоу

Минимальная цель нажатия — **44×44pt**, всегда и для всего. HIG: …/buttons , …/lists-and-tables , …/text-fields , …/pickers , …/toggles , …/segmented-controls

- **Кнопка основного действия**: во всю ширину минус поля 16pt, высота **50pt**, текст 17pt semibold по центру. Внизу экрана — над home indicator, не под ним. Второстепенная — текстовая (plain), без рамки.
  `​.btn{display:block;width:100%;min-height:50px;border-radius:12px;background:#c9c9c9;font-size:17px;font-weight:600}`
- **Списки/таблицы**: ячейка **min 44pt** (обычно 44–56), текст слева, значение/аксессуар справа, chevron › у переходов. **Формы и настройки — inset grouped list**: скруглённые группы на светлом фоне, заголовок секции UPPERCASE 13pt над группой, пояснение 13pt под ней.
  `​.group{border-radius:10px;background:#e6e6e6;margin:0 16px 24px}
  .cell{min-height:44px;display:flex;align-items:center;justify-content:space-between;padding:0 16px;border-bottom:1px solid #d5d5d5}`
- **Текстовые поля**: высота min 44pt, плейсхолдер + подпись; в группированной форме поле = ячейка списка (label слева, ввод справа), не «висящий» инпут. Клавиатура обязана соответствовать типу — в HTML-протике это атрибуты:
  - телефон: `type="tel" inputmode="tel" autocomplete="tel"`
  - **код из SMS: `inputmode="numeric" autocomplete="one-time-code"`** — iOS сам подставит код, обязательный атрибут
  - сумма: `inputmode="decimal"`; email: `type="email" autocomplete="email"`; карта: `inputmode="numeric" autocomplete="cc-number"`
- **Пикеры**: дефолт для выбора из списка — menu picker (ячейка со значением и значком ▲▼, по тапу меню); wheel picker (барабан) — для дат/времени и в нижней части шитов; календарная сетка — выбор дня. Не рисуй андроидный dropdown.
- **Toggle (switch)**: **51×31pt**, только в ячейке списка справа, для мгновенного вкл/выкл без подтверждения.
  `​.toggle{width:51px;height:31px;border-radius:16px;background:#bbb;position:relative}.toggle::after{content:'';width:27px;height:27px;border-radius:50%;background:#f5f5f5;position:absolute;top:2px;left:2px}.toggle.on::after{left:22px}`
- **Segmented control**: высота **32pt**, 2–5 сегментов равной ширины, скруглённая подложка, активный сегмент — «приподнятая» плашка. Только для переключения видов одного контента, НЕ для действий и НЕ для навигации.
  `​.seg{display:flex;height:32px;border-radius:9px;background:#dcdcdc;padding:2px;margin:0 16px}.seg>button{flex:1;border-radius:7px;font-size:13px;color:#777}.seg>.active{background:#f5f5f5;color:#333;font-weight:600}`

## 4. Флоу оформления / платежа

HIG: https://developer.apple.com/design/human-interface-guidelines/entering-data , …/progress-indicators , …/privacy , …/apple-pay

- **Каркас флоу — без tab bar.** Оформление/платёж — не корневой раздел: флоу открывается sheet'ом (.pageSheet с grabber, свой push-стек внутри) или push-ом вглубь раздела со скрытым tab bar. Снизу экрана шага — только CTA 50pt над home indicator 34pt; каркас из §1 без блока `.tabbar`.
- **Шаги**: HIG не требует «один вопрос на экран», требование — минимум ввода и короткие шаги. Правило протика: связанные поля группируй в один экран-форму (inset grouped), смену контекста (доставка → оплата → проверка) — отдельными push-шагами. Один вопрос на экран — только для развилок, определяющих следующий шаг.
- **Progress indication**: в заголовке nav bar — «Доставка» + строка «Шаг 2 из 4» (13pt под заголовком) или тонкий progress bar под nav bar. Не рисуй материальный stepper с кружками-галками.
- **Ошибки — inline, не alert**: подпись 13pt под полем + пометка самого поля; валидация при уходе с поля или сабмите, не на каждый символ. Alert — только когда шаг невозможно продолжить (нет сети, платёж отклонён целиком).
- **Permission-запросы**: системный диалог — alert по центру, текст не кастомизируется, появляться должен в момент нужды, не пачкой на старте. Паттерн **pre-prompt**: перед системным — свой экран/шит «зачем нам пуши» с кнопками «Разрешить» / «Позже»; системный alert показывай после согласия. В протике системный диалог — серый alert с подписью «(системный)».
- **Apple Pay**: не кнопка «оплатить картой», а **payment sheet** — шит снизу: строки (карта, адрес, доставка) + итог + подтверждение Face ID (side button). В сером скелете: sheet со списком строк, суммой и зоной «☐ Подтвердите — Face ID». Кнопка вызова —  Pay, отдельно от обычной кнопки оплаты.
- **Face ID**: системный оверлей поверх текущего экрана (иконка + статус), НЕ отдельный экран логина. В протике — маленькая центральная плашка «Face ID…», исчезающая по тапу.

## 5. Типографика и сетка

HIG: https://developer.apple.com/design/human-interface-guidelines/typography

Шкала SF (размер/интерлиньяж, pt): Large Title **34/41** bold · Title1 28/34 · Title2 22/28 · Title3 20/25 · Headline 17/22 semibold · **Body 17/22** — дефолт всего текста · Callout 16/21 · Subheadline 15/20 · Footnote 13/18 · Caption1 12/16 · Caption2 11/13.

- Минимум читаемого текста — 11pt; основной текст меньше 17pt не делай (Dynamic Type в протике не имитируем, но база должна быть честной).
- Font stack: `-apple-system, system-ui, 'SF Pro Text', sans-serif`. Не подключай кастомные шрифты — скелет.
- Поля контента: **16pt** слева/справа. Вертикальный ритм — кратно 8pt. Межсекционный отступ в формах — 24–35pt.
- Иерархия серым: вес и размер, не цвет. Вторичный текст — 15/13pt #8e8e93-серый.

## 6. Анти-ошибки не-нативности — самопроверка перед выдачей, до вето-десятиминутки

Прогони по каждому экрану перед выдачей протика (шаг 5 порядка работы в probes.md); любой пункт = экран не iOS:

1. ☐ Гамбургер-меню / drawer вместо tab bar (у iOS нет навигационного drawer).
2. ☐ Стрелка ← вместо chevron «‹ Назад» — это андроидный back.
3. ☐ Крестик закрытия СЛЕВА (место iOS: «Отменить» слева текстом, «✕»/«Готово» справа).
4. ☐ Snackbar/toast — на iOS их нет; статус — inline, изменением UI или alert.
5. ☐ Кнопка «Назад» внутри контента страницы — назад живёт только в nav bar.
6. ☐ Интерактивная цель меньше 44×44pt (иконки 24px без паддинга — ошибка).
7. ☐ FAB (плавающая круглая кнопка) — материальный паттерн; действие — в nav bar справа или кнопкой в контенте.
8. ☐ Чекбоксы-квадратики в формах — iOS использует toggle или галочку в ячейке списка.
9. ☐ Больше 5 вкладок в tab bar / вкладки без подписей.
10. ☐ Контент или кнопки в нижних 34pt (зона home indicator).
11. ☐ Alert на каждую ошибку валидации; подтверждение удаления alert'ом вместо action sheet.
12. ☐ Материальный dropdown/outlined-input вместо ячейки-пикера / группированной формы.
13. ☐ Поле кода SMS без `autocomplete="one-time-code"`, телефон без `inputmode="tel"`.
14. ☐ Заголовок экрана выровнен влево в 44pt-баре (влево — только large title 34pt).

## 7. Expo-заметки: нативное поведение бесплатно

Для Expo-протика бери системные компоненты — они сами дают верные размеры/жесты, ничего не имитируй руками:

- **`expo-router` Stack / `@react-navigation/native-stack`** — настоящий push: свайп-назад от края, «‹ Back», `headerLargeTitle: true` — честный large title.
- **`presentation: 'formSheet'` или `'modal'`** в опциях экрана — системный pageSheet; `sheetGrabberVisible: true`, `sheetAllowedDetents` — detents.
- **`Tabs` из expo-router / `@react-navigation/bottom-tabs`** — таб-бар верных размеров; iOS 26-поведение отдаст система.
- **`Alert.alert(...)`** — нативный alert; **`ActionSheetIOS.showActionSheetWithOptions`** (`destructiveButtonIndex`, `cancelButtonIndex`) — нативный action sheet.
- **`Switch`** (RN core) — системный toggle; **`@react-native-segmented-control/segmented-control`** — системный segmented.
- **`TextInput`**: `keyboardType="phone-pad" | "number-pad" | "decimal-pad"`, **`textContentType="oneTimeCode"`** для SMS-кода, `autoComplete`.
- **`@react-native-community/datetimepicker`** — системные wheel/календарь; **`react-native-safe-area-context`** `useSafeAreaInsets()` — реальные 59/34.
- **`expo-local-authentication`** — настоящий Face ID-оверлей для протика платёжного шага.
- Не тяни UI-киты (NativeBase, Paper — материальный) и кастомные таб-бары: ломают нативность, которую стек даёт даром.
