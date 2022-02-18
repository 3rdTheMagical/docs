<!--
.md → .html → .pdf
Редактировать .md файл можно в любом редакторе
Автоматически сгенерировать содержание — VS Code, плагин Markdown All in One
Перед русскоязычными заголовками и заголовками с пробелами добавить <a id="header-name"></a>, заменить id в содержании
Сгенерировать HTML — VS Code, плагин Instant Markdown. Открывает в браузере html странцу
Добавить разрыв страницы в html — <div style="page-break-after: always;"></div>
-->

**Содержание**

-   [Callback API формы бронирования TravelLine](#callback-api-formy-bronirovaniya-travelline)
    -   [onBookingSuccess](#onbookingsuccess)
        -   [Типы способов оплаты (PaymentSystemType)](#tipy-sposobov-oplaty-paymentsystemtype)
    -   [onBookingFailure](#onbookingfailure)
    -   [onBookingEdit](#onbookingedit)
    -   [onBookingStepChanged](#onbookingstepchanged)
        -   [Названия страниц формы бронирования](#nazvaniya-stranic-formy-bronirovaniya)
    -   [onNoAvailableRooms](#onnoavailablerooms)
    -   [onConnect](#onconnect)
        -   [updateGuestToken](#update-guest-token)
    -   [onSearchRooms ^extendedEvents^](#onsearchrooms-extendedevents)
    -   [onLanguageChanged ^extendedEvents^](#onlanguagechanged-extendedevents)
    -   [onCurrencyChanged ^extendedEvents^](#oncurrencychanged-extendedevents)
    -   [onTrackUserAction ^trackEvents^](#ontrackuseraction-trackevents)
        -   [Возможные значения `action`](#vozmozhnye-znacheniya-action)
        -   [Элементы формы бронирования, для которых доступно отслеживание кликов](#elementy-formy-bronirovaniya-dlya-kotoryh-dostupno-otslezhivanie-klikov)
    -   [Обработка событий на примере `onBookingStepChanged`](#obrabotka-sobytiy-na-primere-onbookingstepchanged)
-   [Post-запрос](#post-zapros)

## <a id="callback-api-formy-bronirovaniya-travelline"></a>Callback API формы бронирования TravelLine

В код установки формы бронирования можно передавать callback-функции, которые
будут вызваны при определённых событиях на форме. В callback-функцию будет
передан объект, содержащий информацию о событии.

```javascript
let q = [
    ['setContext', 'TL-INT-travelline', 'ru'],
    ['embed', 'booking-form', {
        container: 'tl-booking-form',
        onBookingSuccess: <имя callback-функции>,
        onLanguageChanged: <имя callback-функции>,
        ...
    }]
];
```

Когда форма бронирования открывается в модальном окне, подписываться на её
события нужно по-другому. Для использования callback-функций нужно подписаться
на события формы через window.addEventListener.

```javascript
window.addEventListener('tlBookingForm./* название события */', function(e) {
    // Обработка
});
```

Стандартный набор данных передаётся в поле `e.detail`. Пример:

```javascript
window.addEventListener('tlBookingForm.bookingStepChanged', function(e) {
    switch (e.detail.step) {
        case 'search':
        case 'preview':
        case 'payment':
        case 'complete':
            break;
        default:
            return;
    }
});
```

### onBookingSuccess

Событие вызывается, когда бронирование успешно завершено. По умолчанию
`onBookingSuccess` возвращает поля `bookingNumber`, `providerName`, `price`,
`currency`. Чтобы включить остальные поля, обратитесь в
[службу техподдержки TravelLine](https://www.travelline.ru/support/).

Список всех возможных свойств объекта, который передаётся в callback-функцию:

-   `bookingNumber` (String) — номер брони
-   `providerId` (String) — id гостиницы в системе TravelLine
-   `providerName` (String) — название гостиницы
-   `arrivalDate` (String) — дата заезда в формате YYYY-MM-DD
-   `departureDate` (String) — дата выезда в формате YYYY-MM-DD
-   `nights` (Number) — срок проживания
-   `price` (Number) — итоговая цена с учётом налогов
-   `priceAfterTax` (Number) — итоговая цена с учётом налогов
-   `priceBeforeTax` (Number) — итоговая цена без учёта налогов
-   `currency` (String) — ISO 4217 alpha-3 код валюты бронирования. Пример:
    `"RUB"`, `"GBP"`, `"USD"`
-   `paymentSystemType` (Enum PaymentSystemType) — тип платёжной системы
-   `customerFullName` (String) — фамилия, имя и отчество заказчика, разделённые
    пробелами
-   `customerEmail` (String) — почта заказчика, на которую будет отправлено
    подтверждение бронирования
-   `customerPhone` (Object) — телефон заказчика, на который он может получить
    смс-подтверждение бронирования и/или нововсти гостиницы
    -   `phone` (String) — номер телефона. Первым символом может быть `+`, если
        заказчик его указал. Пример: `"+79270000000"`, `"89270000000"`
-   `customerComment` (String) — комментарий заказчика
-   `customerShareMarketing` (Boolean) — согласие заказчика на получение
    маркетинговых писем
-   `guests` (Number) — количество гостей, чьи данные были указаны при
    бронировании
-   `rooms` (Number) — количество забронированных номеров
-   `roomTypes` (Array of Objects) — забронированные категории номеров
    -   `id` (String) — id категории номера в TravelLine
    -   `name` (String) — название категории номера
    -   `price` (Number) — цена номера с учётом налогов
    -   `priceAfterTax` (Number) — цена номера с учётом налогов
    -   `priceBeforeTax` (Number) — цена номера без учёта налогов
    -   `rateName` (String) — названия тарифов, по которым был забронирован
        номер, перечисленные через запятую с пробелом. Пример: `"17800, 17801"`
    -   `guests` (Array of Objects) — информация о гостях, проживающих в номере
        -   `name` (String) — фамилия, имя и отчество гостя, разделённые
            пробелами
        -   `citizenship` (String, Optional) — гражданство в формате ISO 3166-1
            alpha-3. Пример: `"RUS"`, `"FRA"`, `"GBR"`
-   `services` (Array of Objects) — услуги
    -   `id` (String) — id услуги в TravelLine
    -   `name` (String) — название услуги
    -   `quantity` (Number) — количество заказанных услуг
    -   `inclusive` (Boolean) — `true`, если стоимость услуги включена в
        стоимость
    -   `price` (Number) — цена услуг с учётом налогов
    -   `priceAfterTax` (Number) — цена услуг с учётом налогов
    -   `priceBeforeTax` (Number) — цена услуг без учёта налогов
    -   `pricePerUnit` (Number) — цена одной услуги с учётом налогов. `0`, если
        услуга включена в стоимость
    -   `pricePerUnitAfterTax` (Number) — цена одной услуги с учётом налогов.
        `0`, если услуга включена в стоимость
    -   `pricePerUnitBeforeTax` (Number) — цена одной услуги без учёта налогов.
        `0`, если услуга включена в стоимость
-   `prepayment` (Number, Optional) — сумма внесённой при бронировании
    предоплаты. Если бронь сделана без предоплаты, этого поля не будет
-   `prepaymentCurrency` (String) — ISO 4217 alpha-3 код валюты предоплаты.
    Пример: `"RUB"`, `"GBP"`, `"USD"`. Если бронь сделана без предоплаты, этого
    поля не будет

#### <a id="tipy-sposobov-oplaty-paymentsystemtype"></a>Типы способов оплаты (PaymentSystemType)

-   `cash` — при заселении, оплата в офисе
-   `guarantee` — гарантия банковской картой
-   `pre_pay` — гарантия банковской картой (предоплата)
-   `guarantee_noprepay` — оплата при заселении (гарантия банковской картой)
-   `credit_card` — кредитная карта
-   `payment_service` — электронные деньги
-   `money_order` — безналичный расчёт, банковский перевод
-   `credit_payment` — кредит
-   `direct_bill`
-   `unknown`

### onBookingFailure

Событие вызывается при ошибке во время оплаты. Свойства объекта, который
передаётся в callback-функцию:

-   `bookingNumber` (String) — номер брони
-   `providerId` (String) — id гостиницы в системе TravelLine
-   `providerName` (String) — название гостиницы

### onBookingEdit

Событие вызывается при отмене брони. Свойства объекта, который передаётся в
callback-функцию:

-   `kind` (String) — `"cancel"`, при отмене брони
-   `success` (Boolean) — `true`, если бронь успешно отменена
-   `status` (String) — новый статус брони
-   `bookingNumber` (String) — номер брони
-   `providerId` (String) — id гостиницы из системы TravelLine
-   `providerName` (String) — название гостиницы

### onBookingStepChanged

Событие вызывается при переходе между страницами формы бронирования. Свойства
объекта, который передаётся в callback-функцию:

-   `step` (String) — название новой страницы формы бронирования

#### <a id="nazvaniya-stranic-formy-bronirovaniya"></a>Названия страниц формы бронирования

-   `search` — страница с номерами
-   `preview` — страница выбора услуг, трансферов, повышения категории номера
-   `payment` — страница с выбором способа оплаты
-   `complete` — страница, на которую гость попадает после завершения
    бронирования
-   `room` — страница в мобильной версии формы с подробным описанием номера и
    выбором тарифов

### onNoAvailableRooms

Если у гостиницы на выбранные даты нет номеров, пользователю будет показана
страница «Не найдено доступных номеров». Событие вызывается при загрузке этой
страницы. Свойства объекта, который передаётся в callback-функцию:

-   `now` (String) — дата поиска в формате YYYY-MM-DD
-   `arrivalDate` (String) — дата заезда в формате YYYY-MM-DD
-   `departureDate` (String) — дата выезда в формате YYYY-MM-DD
-   `guests` (Number) — количество гостей
-   `language` (Sting) — ISO 3166-1 alpha-2 код выбранного языка. Пример:
    `"ru"`, `"en"`, `"en"`
-   `currency` (String) — ISO 4217 alpha-3 код выбранной валюты. Пример:
    `"RUB"`, `"GBP"`, `"USD"`

### onConnect

#### <a id="update-guest-token"></a>updateGuestToken

Пользователя можно автоматически авторизовать в личном кабинете формы
бронирования, если на сайте гостинцы используется api сквозной авторизации. Для
этого нужно подписаться на событие onConnect. В функцию-обработчик нужно
добавить вызов метода `app.updateGuestToken(token);`, где token — это jwt-токен
гостя, полученный в результате вызова api сквозной авторизации. Пример ниже:

```javascript
(function(w) {
    // При инициализации формы бронирования будет
    // вызвана функция onConnectHandler
    function onConnectHandler(app) {
        // Необходимо получить токен гостя.
        // Например, из локального хранилища браузера.
        var token = window.localStorage.getItem('guest-token');
        // Обновляем токен.
        // Если на сайте токена нет, то и на форме он удалится.
        app.updateGuestToken(token);
    }

    var q = [
        ['setContext', 'TL-INT-travelline', 'ru'],
        [
            'embed',
            'booking-form',
            {
                container: 'tl-booking-form',
                // Подписываемся на событие onConnect
                onConnect: onConnectHandler
            }
        ]
    ];
    var t = (w.travelline = w.travelline || {}),
        ti = (t.integration = t.integration || {});
    ti.__cq = ti.__cq ? ti.__cq.concat(q) : q;
    if (!ti.__loader) {
        ti.__loader = true;
        var d = w.document,
            p = d.location.protocol,
            s = d.createElement('script');
        s.type = 'text/javascript';
        s.async = true;
        s.src =
            (p === 'https:' ? p : 'http:') +
            '//ibe.tlintegration.com/integration/loader.js';
        (
            d.getElementsByTagName('head')[0] ||
            d.getElementsByTagName('body')[0]
        ).appendChild(s);
    }
})(window);
```

### <a id="onsearchrooms-extendedevents"></a>onSearchRooms ^extendedEvents^

Событие вызывается при переходе на страницу с номерами. Объект, который
передаётся в callback-функцию, содержит параметры поиска. Чтобы это событие
стало доступно для обработки, обратитесь в
[службу техподдержки TravelLine](https://www.travelline.ru/support/).

-   `arrivalDate` (String) — дата заезда в формате YYYY-MM-DD
-   `departureDate` (String) — дата выезда в формате YYYY-MM-DD
-   `guests` (Array of Objects) — критерии поиска номеров. Каждый критерий
    соответствует размещению в номере
    -   `adults` (Number) — количество взрослых в номере
    -   `children` (Array of Numbers) — дети в номере. Каждый ребёнок
        представлен своим возрастом
-   `language` (Strng) — ISO 3166-1 alpha-2 код выбранного языка. Пример:
    `"ru"`, `"en"`, `"en"`
-   `currency` (String) — ISO 4217 alpha-3 код выбранной валюты. Пример:
    `"RUB"`, `"GBP"`, `"USD"`

Пример параметров поиска для двух номеров: первый — с одним взрослым и двумя
детьми 4 и 6 лет, второй — с двумя взрослыми:

```javascript
var data = {
    arrivalDate: '2020-01-15',
    departureDate: '2020-01-19',
    guests: [
        { adults: 1, children: [4, 6] },
        { adults: 2, children: [] }
    ],
    language: 'ru',
    currency: 'RUB'
};
```

### <a id="onlanguagechanged-extendedevents"></a>onLanguageChanged ^extendedEvents^

Событие вызывается при смене языка на форме бронирования. Чтобы это событие
стало доступно для обработки, обратитесь в
[службу техподдержки TravelLine](https://www.travelline.ru/support/). Свойства
объекта, который передаётся в callback-функцию:

-   `language`(String) — ISO 3166-1 alpha-3 код выбранного языка. Пример:
    `"RUS"`, `"GBR"`, `"USA"`
-   `code` (String) — код выбранного языка. Пример: `"ru"`, `"en"`, `"en"`
-   `region` (String) — регион выбранного языка. Пример: `"ru"`, `"gb"`, `"us"`

### <a id="oncurrencychanged-extendedevents"></a>onCurrencyChanged ^extendedEvents^

Событие вызывается при смене валюты на форме бронирования. Чтобы это событие
стало доступно для обработки, обратитесь в
[службу техподдержки TravelLine](https://www.travelline.ru/support/). Свойства
объекта, который передаётся в callback-функцию:

-   `currency` (String) — ISO 4217 alpha-3 код выбранной валюты. Пример:
    `"RUB"`, `"GBP"`, `"USD"`

### <a id="ontrackuseraction-trackevents"></a>onTrackUserAction ^trackEvents^

Событие вызывается при совершении пользователем различных действий на форме
бронирования. Чтобы это событие стало доступно для обработки, обратитесь в
[службу техподдержки TravelLine](https://www.travelline.ru/support/).

-   `action` (String) — название действия или события на форме бронирования.
    _Свойства объекта, который передаётся в callback-функцию, зависят от
    значения action._

#### <a id="vozmozhnye-znacheniya-action"></a>Возможные значения `action`

-   **`change-order-price`** — изменилась итоговая цена брони: добавлен или
    удалён номер, услуга, трансфер; выбран или отменён платный ранний заезд,
    платный поздний выезд. Или изменилась валюта в процессе бронирования.
    Дополнительные свойства:

    -   `amount` (Number) — итоговая цена с учётом налогов
    -   `currency` (String) — ISO 4217 alpha-3 код валюты бронирования. Пример:
        `"RUB"`, `"GBP"`, `"USD"`

-   **`select-room`** — номер был добавлен или удалён из брони. Индексы в
    массивах у номеров и цен совпадают. Дополнительные свойства:

    -   `ids` (Array of Strings) — id номеров в брони. Id из системы TravelLine
    -   `rooms` (Array of Strings) — названия номеров в брони
    -   `prices` (Array of Objects) — цена каждого номера в брони
        -   `amount` (Number) — цена номера с учётом налогов
        -   `currency` (String) — ISO 4217 alpha-3 код валюты бронирования.
            Пример: `"RUB"`, `"GBP"`, `"USD"`

-   **`upgrade-room`** — повышение категории номера. Дополнительные свойства:

    -   `upgrades` (Array of Objects) — если пользователь отменил повышение
        категории, массив будет пустым
        -   `previousRoom` (Object) — номер до повышения категории
            -   `id` (String) — id категории номера из системы TravelLine
            -   `room` (String) — название категории номера
            -   `price` (Object) — цена номера с учётом налогов
                -   `amount` (Number) — числовое представление цены
                -   `currency` (String) — ISO 4217 alpha-3 код валюты
                    бронирования. Пример: `"RUB"`, `"GBP"`, `"USD"`
        -   `currentRoom` (Object) — номер после повышения категории
            -   `id` (String) — id категории номера из системы TravelLine
            -   `room` (String) — название категории номера
            -   `price` (Object) — цена номера с учётом налогов
                -   `amount` (Number) — числовое представление цены
                -   `currency` (String) — ISO 4217 alpha-3 код валюты
                    бронирования. Пример: `"RUB"`, `"GBP"`, `"USD"`

-   **`update-options`** — при переходе с шага выбора услуг. Дополнительные
    свойства:

    -   `options` (Array of Objects) — услуги в брони
        -   `id` (String) — id услуги из системы TravelLine
        -   `name` (String) — название услуги
        -   `quantity` (Number) — количество заказанных услуг
        -   `price` (Object) — цена услуг с учётом налогов
            -   `amount` (Number) — числовое представление цены
            -   `currency` (String) — ISO 4217 alpha-3 код валюты бронирования.
                Пример: `"RUB"`, `"GBP"`, `"USD"`
        -   `rate` (String) — название тарифа, для которого выбрана услуга
        -   `room` (String) — название номера, для которого выбрана услуга
    -   `transfers` (Array of Objects) — трансферы в брони
        -   `id` (String) — id трансфера из системы TravelLine
        -   `name` (String) — название трансфера
        -   `endPoint` (String) — выбранный вокзал или аэропорт
        -   `vehicles` (Array of Objects) — транспортное средство
            -   `name` (String) — название транспортного средства
            -   `quantity` (Number) — количество транспортных средств
            -   `capacity` (Number) — вместимость транспортного средства
            -   `price` (Object) — цена выбранных транспортных средств
                -   `amount` (Number) — числовое представление цены
                -   `currency` (String) — ISO 4217 alpha-3 код валюты
                    бронирования. Пример: `"RUB"`, `"GBP"`, `"USD"`

-   **`change-dates`** — изменились даты проживания. Дополнительные свойства:

    -   `arrival` (String) — дата заезда в формате YYYY-MM-DD HH:mm
    -   `departure` (String) — дата выезда в формате YYYY-MM-DD HH:mm

-   **`search-rooms`** — доступные номера на заданные даты с выбранным
    количеством гостей. Дополнительные свойства:

    -   `arrival` (String) — дата заезда в формате YYYY-MM-DD
    -   `departure` (String) — дата выезда в формате YYYY-MM-DD
    -   `adults` (Number) — общее количество взрослых в поисковом запросе
    -   `children` (Number) — общее количество детей в поисковом запросе
    -   `language` (String) — ISO 3166-1 alpha-2 код языка. Пример: `"ru"`,
        `"en"`, `"en"`
    -   `rooms` (Array of Objects) — доступные для выбора категории номеров
        -   `id` (String) — id категории номера из системы TravelLine
        -   `room` (String) — название номера
        -   `adults` (Number) — количество взрослых в номере
        -   `children` (Number) — количество детей в номере
        -   `price` (Number) — цена номера с учётом налогов
        -   `currency` (String) — ISO 4217 alpha-3 код валюты бронирования.
            Пример: `"RUB"`, `"GBP"`, `"USD"`

-   **`click`** — клики по определённым элементам формы бронирования.
    Дополнительные свойства:

    -   `target` (String) — название элемента формы бронирования
    -   `value` (Enum PaymentSystemType) — при клике на кнопку оплаты брони
        передаётся тип платёжной системы

-   **`type`** — ввод данных в поля на форме. События отправляются только после
    того, как гость подтвердит своё согласие на обработку персональных данных.
    Дополнительные свойства:

    -   `field` (String) — название поля ввода. Может быть `"name"`, `"email"`,
        `"phone"`
    -   `value` (String) — значение, введённое в поле

-   **`interact`** — событие позволяет понять, как долго пользователь не
    совершал действий на форме. Дополнительных свойств нет

#### <a id="elementy-formy-bronirovaniya-dlya-kotoryh-dostupno-otslezhivanie-klikov"></a>Элементы формы бронирования, для которых доступно отслеживание кликов

-   `payment-pay-button` — кнопка выбора способа оплаты
-   `rooms-change-search-button` — кнопка «Изменить даты» в шапке формы
    бронирования
-   `rooms-book-button` — кнопка «Забронировать» рядом с номером
-   `rooms-cart-book-button` — кнопка «Забронировать» в корзине, в нижней части
    формы бронирования. Страница выбора номеров
-   `services-cart-book-button` — кнопка «Продолжить» в корзине, в нижней части
    формы бронирования. Страница выбора услуг
-   `services-book-link` — ссылка «Продолжить бронирование» в шапке формы
    бронирования. Страница выбора услуг

### <a id="obrabotka-sobytiy-na-primere-onbookingstepchanged"></a>Обработка событий на примере `onBookingStepChanged`

Html:

```html
<div id="tl-booking-form"></div>
```

Javascript:

```javascript
(function(w) {
    // При переходах между страницами будет вызвана doBookingStepChanged
    function doBookingStepChanged(data) {
        console.log('Переход на страницу ', data.step);
    }
    var q = [
        ['setContext', 'TL-INT-travelline', 'ru'],
        [
            'embed',
            'booking-form',
            {
                container: 'tl-booking-form',
                onBookingStepChanged: doBookingStepChanged
            }
        ]
    ];
    var t = (w.travelline = w.travelline || {}),
        ti = (t.integration = t.integration || {});
    ti.__cq = ti.__cq ? ti.__cq.concat(q) : q;
    if (!ti.__loader) {
        ti.__loader = true;
        var d = w.document,
            p = d.location.protocol,
            s = d.createElement('script');
        s.type = 'text/javascript';
        s.async = true;
        s.src =
            (p === 'https:' ? p : 'http:') +
            '//ibe.tlintegration.com/integration/loader.js';
        (
            d.getElementsByTagName('head')[0] ||
            d.getElementsByTagName('body')[0]
        ).appendChild(s);
    }
})(window);
```

## <a id="post-zapros"></a>Post-запрос

Иногда недостаточно обрабатывать события только в браузере. Система TravelLine
умеет отправлять серверный POST-запрос после завершения или при редактировании
бронирования. Адрес, на который будут отправляться данные, предоставляет
гостиница. Формат данных запроса `application/x-www-form-urlencoded`.
Отправляются следующие данные:

-   `ProviderId` (String) — id гостиницы в системе TravelLine
-   `ProviderName` (String) — название гостиницы в системе TravelLine
-   `ProviderCity` (String) — город, в котором расположена гостиница
-   `ProviderLogo` (String) — url-адрес логотипа гостиницы
-   `Status` (String) — статус брони. Возможные значения:
    -   `"Unconfirmed"` — незавершённое бронирование. Гость выбрал номер,
        согласился с обработкой персональных данных, но не выбрал способ оплаты
    -   `"Confirmed"` — бронь подтверждена
    -   `"Cancelled"` — бронь отменена
    -   `"Released"` — гость заселился, но выехал раньше планируемой даты
    -   `"Pending"` — бронь ожидает оплаты
-   `ChangeDate` (String) — время последнего изменения брони в формате
    `yyyy-MM-dd HH:mm:ss`
-   `BookingNumber` (String) — номер брони
-   `ArrivalDate` (String) — дата заезда в формате `YYYY-MM-DD`
-   `DepartureDate` (String) — дата выезда в формате `YYYY-MM-DD`
-   `TotalPriceAfterTax` (Number) — итоговая стоимость брони с учётом налогов
-   `TotalPriceBeforeTax` (Number) — итоговая стоимость брони без учёта налогов
-   `Currency` (String) — ISO 4217 alpha-3 код валюты бронирования. Пример:
    `"RUB"`, `"GBP"`, `"USD"`
-   `GuaranteeCode` (String) — код способа оплаты. Пример: `"AT_ARRIVAL"`,
    `"SBERBANK"`, `"ALPHA_BANK_RBS"`
-   `GuaranteeId` (String) — id способа оплаты в системе TravelLine
-   `GuaranteeName` (String) — название способа оплаты. Пример:
    `"При заселении"`, `"Банковская карта"`,
    `"Банковская карта (отложенная электронная оплата)"`
-   `ReceivedPaymentAmount` (String) — сумма предоплаты
-   `ReceivedPaymentCurrency` (String) — ISO 4217 alpha-3 код валюты суммы
    предоплаты. Пример: `"RUB"`, `"GBP"`, `"USD"`
-   `CustomerName` (String) — ФИО заказчика, например, `"Иванов Иван Иванович"`
-   `CustomerPhone` (String) — номер телефона заказчика
-   `CustomerEmail` (String) — почта заказчика
-   `CustomerComment` (String) — комментарий заказчика, который он оставил на
    форме бронирования
-   `CustomerShareMarketing` (Boolean) — `true`, если заказчик согласился
    получать новости гостиницы по почте или sms
-   `GuestsCount` (Number) — количество гостей в брони

В POST-запросе могут передаваться дополнительные поля со значениями, которые вы
сами определяете на странице сайта, где установлена форма бронирования.
Например, вы можете связать идентификатор гостя в Roistat с бронированием. Для
этого нужно подписаться на событие `onConnect`, получить объект с интерфейсом
формы и с помощью ф-ции `pushArtifact` сообщить форме о дополнительных полях:

```javascript
(function(w) {
    // Получаем идентификатор гостя Roistat
    var roistatId = (document.cookie.match('roistat_visit=?=[0-9]+') || [
        '='
    ])[0].split('=')[1];

    // При инициализации формы бронирования будет вызвана ф-ция onConnectHandler
    function onConnectHandler(app) {
        // Передаём идентификатор гостя Roistat в форму бронирования
        // В POST-запросе будет поле с именем RoistatId
        app.pushArtifact('RoistatId', roistatId);
        // Также можно создавать другие свои поля
        app.pushArtifact('Referrer', document.referrer);
        app.pushArtifact('Location', location.href);
    }

    var q = [
        ['setContext', 'TL-INT-travelline', 'ru'],
        [
            'embed',
            'booking-form',
            {
                container: 'tl-booking-form',
                // Подписываемся на событие onConnect
                onConnect: onConnectHandler
            }
        ]
    ];
    var t = (w.travelline = w.travelline || {}),
        ti = (t.integration = t.integration || {});
    ti.__cq = ti.__cq ? ti.__cq.concat(q) : q;
    if (!ti.__loader) {
        ti.__loader = true;
        var d = w.document,
            p = d.location.protocol,
            s = d.createElement('script');
        s.type = 'text/javascript';
        s.async = true;
        s.src =
            (p === 'https:' ? p : 'http:') +
            '//ibe.tlintegration.com/integration/loader.js';
        (
            d.getElementsByTagName('head')[0] ||
            d.getElementsByTagName('body')[0]
        ).appendChild(s);
    }
})(window);
```

Чтобы вы могли быть уверенными, что запрос был отправлен с нашего сервера, мы
можем назначить уникальный идентификатор и отправлять его вместе со всеми нашими
запросами в заголовке `X-TL-Token`. Этот идентификатор будет известен только вам
и по его наличию и корректности вы сможете подтвердить источник запроса.

Обратите внимание, что реализация механизма отправки POST-запросов не
гарантирует 100% доставки всех бронирований. При недоступности адреса, на
который мы отправляем запрос, повторные попытки не совершаются и запрос будет
потерян. POST-запросы предназначены для маркетинговых и иных целей, где не
требуется высокая точность. Для полноценной интеграции CRM с сервисами
TravelLine вы можете подключить специализированный модуль. Список поддерживаемых
CRM и описание модуля можно найти
[по этой ссылке](https://www.travelline.ru/products/crm/tl-integration/).

Чтобы настроить серверную передачу данных, обратитесь в
[службу техподдержки TravelLine](https://www.travelline.ru/support/).
