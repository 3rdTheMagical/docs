<!--
.md → .html → .pdf
Редактировать .md файл можно в любом редакторе
Автоматически сгенерировать содержание — VS Code, плагин Markdown All in One
Перед русскоязычными заголовками и заголовками с пробелами добавить <a id="header-name"></a>, заменить id в содержании
Сгенерировать HTML — VS Code, плагин Instant Markdown. Открывает в браузере html странцу
Добавить разрыв страницы в html — <div style="page-break-after: always;"></div>
-->

**Contents**

-   [Callback API of the TravelLine Reservation Form](#callback-api-of-the-travelline-reservation-form)
    -   [onBookingSuccess](#onbookingsuccess)
        -   [Payment Method Types (PaymentSystemType)](#payment-method-types-paymentsystemtype)
    -   [onBookingFailure](#onbookingfailure)
    -   [onBookingEdit](#onbookingedit)
    -   [onBookingStepChanged](#onbookingstepchanged)
        -   [Names of the Reservation Form Pages](#names-of-the-reservation-form-pages)
    -   [onNoAvailableRooms](#onnoavailablerooms)
    -   [onConnect](#onconnect)
        -   [updateGuestToken](#update-guest-token)
    -   [onSearchRooms ^extendedEvents^](#onsearchrooms-extendedevents)
    -   [onLanguageChanged ^extendedEvents^](#onlanguagechanged-extendedevents)
    -   [onCurrencyChanged ^extendedEvents^](#oncurrencychanged-extendedevents)
    -   [onTrackUserAction ^trackEvents^](#ontrackuseraction-trackevents)
        -   [Possible `action` values include](#possible-action-values-include)
        -   [Reservation Form Elements for Which Clicking Can Be Tracked](#reservation-form-elements-for-which-clicking-can-be-tracked)
    -   [Events processing on example of `onBookingStepChanged`](#events-processing-on-example-of-onbookingstepchanged)
-   [POST request](#post-request)

## Callback API of the TravelLine Reservation Form

Callback functions that will be called upon certain events on the form can be
passed to the form setup code. An object containing information about the event
will be passed into the callback function.

```javascript
var q = [
    ['setContext', 'TL-INT-travelline', 'en'],
    ['embed', 'booking-form', {
        container: 'tl-booking-form',
        onBookingSuccess: <callback function name>,
        onLanguageChanged: <callback function name>,
        ...
    }]
];
```

When the reservation form is opened in a modal window, you need to subscribe to
its events differently. To use callback functions, you need to subscribe to the
form events via window.addEventListener.

```javascript
window.addEventListener('tlBookingForm./* event name */', function(e) {
    // Code
});
```

The standard dataset is passed to the `e.detail` field. Example:

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

The event is called when a reservation is successfully completed. By default
`onBookingSuccess` method returns these properties: `bookingNumber`,
`providerName`, `price`, `currency`. To make other properties available, please
contact [TravelLine Support](https://www.travelline.pro/contacts/).

Here is the list of all possible properties of the object that is passed to the
callback function:

-   `bookingNumber` (String) — reservation number
-   `providerId` (String) — hotel ID in TravelLine
-   `providerName` (String) — hotel name
-   `arrivalDate` (String) — arrival date, YYYY-MM-DD
-   `departureDate` (String) — departure date, YYYY-MM-DD
-   `nights` (Number) — period of stay
-   `price` (Number) — total price including taxes
-   `priceAfterTax` (Number) — total price including taxes
-   `priceBeforeTax` (Number) — total price without taxes
-   `currency` (String) — ISO 4217 alpha-3 reservation currency code. Example:
    `"RUB"`, `"GBP"`, `"USD"`
-   `paymentSystemType` (Enum PaymentSystemType) — type of the payment system
-   `customerFullName` (String) — customer's full name separated by spaces
-   `customerEmail` (String) — customer's e-mail to which the reservation
    confirmation will be sent
-   `customerPhone` (String) — customer's phone number
-   `customerComment` (String) — customer's comment
-   `customerShareMarketing` (Boolean) — customer's consent to marketing letters
-   `guests` (Number) — number of guests whose details were provided during the
    reservation
-   `rooms` (Number) — number of rooms booked
-   `roomTypes` (Array of Objects) — booked room categories
    -   `id` (String) — room category ID in TravelLine
    -   `name` (String) — room category name
    -   `price` (Number) — room price including taxes
    -   `priceAfterTax` (Number) — room price including taxes
    -   `priceBeforeTax` (Number) — room price without taxes
    -   `rateName` (String) — names of rates at which the room was booked
        separated by commas with a space. Example: `"17800, 17801"`
    -   `guests` (Array of Objects) — information about the guests staying in
        the room
        -   `name` (String) — guest's full name separated by spaces
        -   `citizenship` (String, Optional) — citizenship in ISO 3166-1 alpha-3
            format. Example: `"RUS"`, `"FRA"`, `"GBR"`
-   `services` (Array of Objects) — services
    -   `id` (String) — service ID in TravelLine
    -   `name` (String) — service name
    -   `quantity` (Number) — number of services ordered
    -   `inclusive` (Boolean) — `true` if the cost of the service is included in
        the reservation price
    -   `price` (Number) — services price including taxes
    -   `priceAfterTax` (Number) — services price including taxes
    -   `priceBeforeTax` (Number) — services price without taxes
    -   `pricePerUnit` (Number) — price of one service, including taxes. 0 if
        the service is included in the price
    -   `pricePerUnitAfterTax` (Number) — price of one service, including taxes.
        `0` if the service is included in the price
    -   `pricePerUnitBeforeTax` (Number) — price of one service, excluding
        taxes. `0` if the service is included in the price
-   `prepayment` (Number, Optional) — amount of prepayment made during
    reservation. Exists if booking has prepayment
-   `prepaymentCurrency` (String) — ISO 4217 alpha-3 prepayment currency code.
    Example: `"RUB"`, `"GBP"`, `"USD"`. Exists if booking has prepayment

#### <a id="payment-method-types-paymentsystemtype"></a>Payment Method Types (PaymentSystemType)

-   `cash` — upon check-in, payment at the office
-   `guarantee` — credit card guarantee
-   `pre_pay` — credit card guarantee (prepayment)
-   `guarantee_noprepay` — payment upon check-in (credit card guarantee)
-   `credit_card` — credit card
-   `payment_service` — electronic money
-   `money_order` — cashless, bank transfer
-   `credit_payment` — credit
-   `direct_bill`
-   `unknown`

### onBookingFailure

The event is called in case of error during the payment process. Properties of
the object passed to the callback function:

-   `bookingNumber` (String) — reservation number
-   `providerId` (String) — hotel ID in TravelLine
-   `providerName` (String) — hotel name

### onBookingEdit

The event is called in case of reservation cancellation. Properties of the
object passed to the callback function:

-   `kind` (String) — `"cancel"` when reservation is cancelled
-   `success` (Boolean) — `true` if reservation is successfully cancelled
-   `status` (String) — new status of the reservation
-   `bookingNumber` (String) — reservation number
-   `providerId` (String) — hotel ID in TravelLine
-   `providerName` (String) — hotel name

### onBookingStepChanged

The event is called in case of moving between the reservation form pages.
Properties of the object passed to the callback function:

-   `step` (String) — name of the new reservation form page

#### Names of the Reservation Form Pages

-   `search` — rooms page
-   `preview` — page for selecting services, transfers, room upgrades
-   `payment` — page for selecting a payment method
-   `complete` — page where the guest lands after completing the reservation
-   `room` — page in the mobile version of the form with a detailed room
    description and rates selection

### onNoAvailableRooms

If the hotel does not have rooms available on the selected dates, the user will
see the "No available rooms found" page. The event is called when this page is
loaded. Properties of the object passed to the callback function:

-   `now` (String) — search date, YYYY-MM-DD
-   `arrivalDate` (String) — arrival date, YYYY-MM-DD
-   `departureDate` (String) — departure date, YYYY-MM-DD
-   `guests` (Number) — number of guests
-   `language` (Sting) — selected language code per ISO 3166-1 alpha-2. Example:
    `"ru"`, `"en"`, `"en"`
-   `currency` (String) — selected currency code per ISO 4217 alpha-3. Example:
    `"RUB"`, `"GBP"`, `"USD"`

### onConnect

#### <a id="update-guest-token"></a>updateGuestToken

The user can be automatically authorized in the personal account of the
reservation form if the pass-through authorization api is used on the hotel
website. To do this, you must subscribe to the event `onConnect`. In the handler
function, add a call `app.updateGuestToken(token);` where `token` is the guest's
jwt-token obtained as a result of the pass-through authorization api call. See
example below:

```javascript
(function(w) {
    // When initializing the reservation form,
    // the onConnectHandler function will be called
    function onConnectHandler(app) {
        // Need to get a guest token.
        // For example, from the local storage of the browser.
        var token = window.localStorage.getItem('guest-token');
        // Update token.
        // If there is no token on the hotel site,
        // then it will be deleted on the reservation form.
        app.updateGuestToken(token);
    }

    var q = [
        ['setContext', 'TL-INT-travelline', 'en'],
        [
            'embed',
            'booking-form',
            {
                container: 'tl-booking-form',
                // Subscribe to the event onConnect
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

The event is called upon transit to the rooms page. The object passed to the
callback function contains the search parameters. To make this event available
for processing, please contact
[TravelLine Support](https://www.travelline.pro/contacts/).

-   `arrivalDate` (String) — arrival date, YYYY-MM-DD
-   `departureDate` (String) — departure date, YYYY-MM-DD
-   `guests` (Array of Objects) — rooms search criteria. Each criterion
    corresponds to accommodation in the room.
    -   `adults` (Number) — number of adults in the room
    -   `children` (Array of Numbers) — children in the room. Each child is
        presented by the age.
-   `language` (String) — selected language code per ISO 3166-1 alpha-2.
    Example: `"ru"`, `"en"`, `"en"`
-   `currency` (String) — selected currency code per ISO 4217 alpha-3. Example:
    `"RUB"`, `"GBP"`, `"USD"`

Example of search parameters for two rooms. One room for one adult guest with
two children aged 4 and 6, the other room is for two adult guests:

```javascript
var data = {
    arrivalDate: '2020-01-15',
    departureDate: '2020-01-19',
    guests: [
        { adults: 1, children: [4, 6] },
        { adults: 2, children: [] }
    ],
    language: 'en',
    currency: 'USD'
};
```

### <a id="onlanguagechanged-extendedevents"></a>onLanguageChanged ^extendedEvents^

The event is called when language is changed on the reservation form. To make
this event available for processing, please contact
[TravelLine Support](https://www.travelline.pro/contacts/). Properties of the
object passed to the callback function:

-   `language`(String) — selected language code per ISO 3166-1 alpha-3. Example:
    `"RUS"`, `"GBR"`, `"USA"`
-   `code` (String) — selected language code. Example: `"ru"`, `"en"`, `"en"`
-   `region` (String) — selected language region: `"ru"`, `"gb"`, `"us"`

### <a id="oncurrencychanged-extendedevents"></a>onCurrencyChanged ^extendedEvents^

The event is called when currency is changed on the reservation form. To make
this event available for processing, please contact
[TravelLine Support](https://www.travelline.pro/contacts/). Properties of the
object passed to the callback function:

-   `currency` (String) — selected currency code per ISO 4217 alpha-3. Example:
    `"RUB"`, `"GBP"`, `"USD"`

### <a id="ontrackuseraction-trackevents"></a>onTrackUserAction ^trackEvents^

The event is called upon various actions made by the user on the reservation
form. To make this event available for processing, please contact
[TravelLine Support](https://www.travelline.pro/contacts/).

-   `action` (String) — name of action or event on the reservation form.
    _Properties of the object passed to the callback function depend on the
    action value._

#### <a id="possible-action-values-include"></a>Possible `action` values include

-   **`change-order-price`** — total price of the reservation has changed room,
    service or transfer added or removed; paid early check-in or late check-out
    selected or canceled. Currency changed during the reservation process.
    Additional properties include:

    -   `amount` (Number) — total price including taxes
    -   `currency` (String) — reservation currency code per ISO 4217 alpha-3.
        Example: `"RUB"`, `"GBP"`, `"USD"`

-   **`select-room`** — room added or removed from the reservation. Indices in
    the arrays of the rooms and prices coincide. Additional properties include:

    -   `ids` (Array of Strings) — rooms ID in the reservation. TravelLine ID
    -   `rooms` (Array of Strings) — names of rooms in the reservation
    -   `prices` (Array of Objects) — price of each room in the reservation
        -   `amount` (Number) — room price including taxes
        -   `currency` (String) — reservation currency code per ISO 4217
            alpha-3. Example: `"RUB"`, `"GBP"`, `"USD"`

-   **`upgrade-room`** — room category upgrade. Additional properties include:

    -   `upgrades` (Array of Objects) — if the user cancels the room category
        upgrade, the array will be empty
        -   `previousRoom` (Object) — room category before upgrade
            -   `id` (String) — room category ID in TravelLine
            -   `room` (String) — room category name
            -   `price` (Object) — room price including taxes
                -   `amount` (Number) — price in numbers
                -   `currency` (String) — reservation currency code per ISO 4217
                    alpha-3. Example: `"RUB"`, `"GBP"`, `"USD"`
        -   `currentRoom` (Object) — room category after upgrade
            -   `id` (String) — room category ID in TravelLine
            -   `room` (String) — room category name
            -   `price` (Object) — room price including taxes
                -   `amount` (Number) — price in numbers
                -   `currency` (String) — reservation currency code per ISO 4217
                    alpha-3. Example: `"RUB"`, `"GBP"`, `"USD"`

-   **`update-options`** — upon transfer from services selection step.
    Additional properties include:

    -   `options` (Array of Objects) — services in the reservation
        -   `id` (String) — service ID in TravelLine
        -   `name` (String) — service name
        -   `quantity` (Number) — number of services ordered
        -   `price` (Object) — services price including taxes
            -   `amount` (Number) — price in numbers
            -   `currency` (String) — reservation currency code per ISO 4217
                alpha-3. Example: `"RUB"`, `"GBP"`, `"USD"`
        -   `rate` (String) — name of rate for which the service is selected
        -   `room` (String) — name of room for which the service is selected
    -   `transfers` (Array of Objects) — transfers in the reservation
        -   `id` (String) — transfer ID in TravelLine
        -   `name` (String) — transfer name
        -   `endPoint` (String) — rail station or airport selected
        -   `vehicles` (Array of Objects) — vehicle
            -   `name` (String) — vehicle name
            -   `quantity` (Number) — number of vehicles
            -   `capacity` (Number) — capacity of the vehicle
            -   `price` (Object) — price of the vehicles selected
                -   `amount` (Number) — price in numbers
                -   `currency` (String) — reservation currency code per ISO 4217
                    alpha-3. Example: `"RUB"`, `"GBP"`, `"USD"`

-   **`change-dates`** — reservation dates changed. Additional properties
    include:

    -   `arrival` (String) — arrival date, YYYY-MM-DD HH:mm
    -   `departure` (String) — departure date, YYYY-MM-DD HH:mm

-   **`search-rooms`** — rooms available for selected dates and selected number
    of guests. Additional properties include:

    -   `arrival` (String) — arrival date, YYYY-MM-DD
    -   `departure` (String) — departure date, YYYY-MM-DD
    -   `adults` (Number) — total number of adults in search request
    -   `children` (Number) — total number of children in search request
    -   `language` (String) — language code per ISO 3166-1 alpha-2. Example:
        `"ru"`, `"en"`, `"en"`
    -   `rooms` (Array of Objects) — room categories available for selection
        -   `id` (String) — room category ID in TravelLine
        -   `room` (String) — room name
        -   `adults` (Number) — number of adults in the room
        -   `children` (Number) — number of children in the room
        -   `price` (Number) — room price including taxes
        -   `currency` (String) — reservation currency code per ISO 4217
            alpha-3. Example: `"RUB"`, `"GBP"`, `"USD"`

-   **`click`** — clicks on specific reservation form elements. Additional
    properties include:

    -   `target` (String) — reservation form element name
    -   `value` (Enum PaymentSystemType) — payment system type is transferred
        upon clicking the reservation payment button.

-   **`type`** — entry of data into the form fields. Events are sent only after
    the guest agrees to personal data processing. Additional properties include:

    -   `field` (String) — entry field name. It can be `"name"`, `"email"`,
        `"phone"`
    -   `value` (String) — value entered

-   **`interact`** — the event indicates the user's activity on the form. No
    additional properties.

#### <a id="reservation-form-elements-for-which-clicking-can-be-tracked"></a>Reservation Form Elements for Which Clicking Can Be Tracked

-   `payment-pay-button` — payment method selection button
-   `rooms-change-search-button` — "Change dates" button in the reservation form
    header
-   `rooms-book-button` — "Book" button next to the room
-   `rooms-cart-book-button` — "Book" button in the basket at the bottom of the
    reservation form. Rooms selection page
-   `services-cart-book-button` — "Continue" button in the basket at the bottom
    of the reservation form. Services selection page
-   `services-book-link` — "Continue booking" link in the header of the
    reservation form. Services selection page

### <a id="events-processing-on-example-of-onbookingstepchanged"></a>Events Processing on Example of `onBookingStepChanged`

Html:

```html
<div id="tl-booking-form"></div>
```

Javascript:

```javascript
(function(w) {
    // doBookingStepChanged is called upon transition between pages
    function doBookingStepChanged(data) {
        console.log('transition to page ', data.step);
    }
    var q = [
        ['setContext', 'TL-INT-travelline', 'en'],
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

## <a id="post-request"></a>POST request

Sometimes it is not enough to handle events in the browser only. TravelLine is
able to send a server POST request after completion or when editing the
reservation. The address to which the data is sent is provided by the hotel. The
data request format is `application/x-www-form-urlencoded`. The following data
is sent:

-   `ProviderId` (String) — hotel ID in TravelLine
-   `ProviderName` (String) — hotel name in TravelLine
-   `ProviderCity` (String) — hotel city
-   `ProviderLogo` (String) — hotel logo url
-   `Status` (String) — reservation status. Possible values:
    -   `"Unconfirmed"` — unfinished reservation. The guest chose the room,
        agreed to the processing of personal data, but did not choose the
        payment method
    -   `"Confirmed"` — reservation is confirmed
    -   `"Cancelled"` — reservation is cancelled
    -   `"Released"` — the guest checked in, but checked out earlier than
        planned
    -   `"Pending"` — reservation pending payment
-   `ChangeDate` (String) — time of the last reservation change in the format
    `yyyy-MM-dd HH:mm:ss`
-   `BookingNumber` (String) — reservation number
-   `ArrivalDate` (String) — arrival date, `YYYY-MM-DD`
-   `DepartureDate` (String) — departure date, `YYYY-MM-DD`
-   `TotalPriceAfterTax` (Number) — total reservation price including taxes
-   `TotalPriceBeforeTax` (Number) — total reservation price without taxes
-   `Currency` (String) — reservation currency code per ISO 4217 alpha-3.
    Example: `"RUB"`, `"GBP"`, `"USD"`
-   `GuaranteeCode` (String) — payment method code. Example: `"AT_ARRIVAL"`,
    `"SBERBANK"`, `"ALPHA_BANK_RBS"`
-   `GuaranteeId` (String) — payment method ID in TravelLine
-   `GuaranteeName` (String) — payment method name. Example:
    `"Payment at check-in"`, `"Credit Card"`,
    `"Credit Card (deferred electronic payment)"`
-   `ReceivedPaymentAmount` (String) — prepayment amount
-   `ReceivedPaymentCurrency` (String) — prepayment currency code per ISO 4217
    alpha-3. Example: `"RUB"`, `"GBP"`, `"USD"`
-   `CustomerName` (String) — customer's full name. Example: `"John Smith"`
-   `CustomerPhone` (String) — customer's phone number
-   `CustomerEmail` (String) — customer's email
-   `CustomerComment` (String) — customer's comment
-   `CustomerShareMarketing` (Boolean) — `true` if the customer has agreed to
    receive hotel news via email or sms
-   `GuestsCount` (Number) — number of guests in the reservation

POST request can send additional line fields with values that you define by
yourself where the TravelLine Reservation Form on the page of the site is set.
For example, you can associate your Roistat customer ID with reservation. To do
this, you must subscribe to the event `onConnect`, obtain an object with the
Reservation Form interface and use the function `pushArtifact` inform the form
of additional line fields:

```javascript
(function(w) {
    // Get customer ID Roistat
    var roistatId = (document.cookie.match('roistat_visit=?=[0-9]+') || [
        '='
    ])[0].split('=')[1];

    // When the Form is initialized, onConnectHandler will be called
    function onConnectHandler(app) {
        // Send customer’s ID Roistat to the Reservation Form
        // In POST-request stands the line field with name RoistatId
        app.pushArtifact('RoistatId', roistatId);
        // Also you can create different other line fields
        app.pushArtifact('Referrer', document.referrer);
        app.pushArtifact('Location', location.href);
    }

    var q = [
        ['setContext', 'TL-INT-travelline', 'en'],
        [
            'embed',
            'booking-form',
            {
                container: 'tl-booking-form',
                // Subscribe to the event onConnect
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

If you want to make sure and no doubt, that the request was sent from our
server, we can assign a unique identifier and send it along with all our
requests with the heading `X-TL-Token`. You will be the one who knows the
identifier, and you will be able to confirm the source of the request by its
presence and correctness.

Please pay attention, that implementation of the POST request mechanism does not
guarantee 100% of all reservations delivery. If the address to which we send the
request is unavailable, no retries can be made and the request will be lost.
POST requests are meant for marketing and other purposes where high accuracy is
not required. To fully integrate CRM with TravelLine services, you can connect a
specialized system interfacing.
[See this link](https://www.travelline.ru/products/crm/tl-integration/) for a
list of supported CRM and a description of our offering.

To setup the server data transfer, please contact
[TravelLine Support](https://www.travelline.pro/contacts/).
