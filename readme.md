node-qiwi-api
================
Официальная [документация](https://developer.qiwi.com/qiwiwallet/qiwicom_ru.html) к Qiwi api.

Get started
----------------
Для начала вам необходимо получить токен на сайте [Qiwi](https://qiwi.com/api).
```
npm install node-qiwi-api
```
Инициализируйте новый кошелек с выданным токеном:
```js
var Qiwi = require('node-qiwi-promise-api').Qiwi;
var Wallet = new Qiwi(token);
```
Теперь вы можете получать информацию о кошельке и совершать переводы на другие кошельки, мобильный телефон и карты.

Информация об аккаунте
----------------
```js
Wallet.getAccountInfo()
  .then((info) => {
    console.log(info);
  }
  .catch((error) => {
    console.log(error);
  })
```

Баланс
----------------
```js
Wallet.getBalance()  
  .then((balance) => {
    console.log(balance);
  }
  .catch((error) => {
    console.log(error);
  })
```
История операций
----------------
```js
Wallet.getOperationHistory(requestOptions)
  .then((history) => {
    console.log(history);
  }
  .catch((error) => {
    console.log(error);
  })
```
requestOptions включают в себя: 
* **rows** - Число платежей в ответе, для разбивки отчета на части. Целое число от 1 до 50. Обязательный параметр.
* **operation** - Тип операций в отчете, для отбора. Допустимые значения: ALL - все операции, IN - только пополнения, OUT - только платежи, QIWI_CARD - только платежи по картам QIWI (QVC, QVP). По умолчанию ALL
* **sources** - Источники платежа, для отбора. Каждый источник задается как отдельный параметр и нумеруется элементом массива, начиная с нуля (sources[0], sources[1] и т.д.). Допустимые значения: QW_RUB - рублевый счет кошелька, QW_USD - счет кошелька в долларах, QW_EUR - счет кошелька в евро, CARD - привязанные и непривязанные к кошельку банковские карты, MK - счет мобильного оператора. Если не указаны, учитываются все источники
* **startDate** - Начальная дата поиска платежей (формат ГГГГ-ММ-ДДTчч:мм:ссZ). По умолчанию, равен вчерашней дате. Используется только вместе с endDate
* **endDate** - Конечная дата поиска платежей (формат ГГГГ-ММ-ДДTчч:мм:ссZ). По умолчанию, равен текущей дате. Используется только вместе с startDate
* **nextTxnDate** - Дата транзакции (формат ГГГГ-ММ-ДДTчч:мм:ссZ), для отсчета от предыдущего списка (см. параметр nextTxnDate в ответе). Используется только вместе с nextTxnId
* **nextTxnId** - 	Номер предшествующей транзакции, для отсчета от предыдущего списка (см. параметр nextTxnId в ответе). Используется только вместе с nextTxnDate
Максимальный допустимый интервал между startDate и endDate - 90 календарных дней.

Например информация о 25 исходящих платежах может быть получена следующим образом:
```js
Wallet.getOperationHistory({rows: 25, operation: "OUT"})
```
Статистика по операциям
----------------
Для получения статистики по суммам платежей за заданный период используется подзапрос запроса истории.
```js
Wallet.getOperationStats(requestOptions)
  .then((stats) => {
    console.log(stats);
  }
  .catch((error) => {
    console.log(error);
  })
```
requestOptions: **operation, sources, startDate, endDate** - Параметры аналогичны параметрам в **getOperationHistory**.

Перевод на Qiwi кошелек
----------------
```js
Wallet.toWallet({ amount: '0.01', comment: 'test', account: '+79261234567' })
  .then((status) => {
    console.log(status);
  }
  .catch((error) => {
    console.log(error);
  })
```
* **amount** - Сумма
* **comment** - Комментарий к платежу.
* **account** - Номер телефона получателя (с международным префиксом)

Перевод на мобильный телефон
----------------
Ничем не отличается от перевода на кошелек, за исключением того, что номер указывается без международного префикса:
```js
Wallet.toMobilePhone({ amount: '0.01', comment: 'test', account: '9261234567' })
  .then((status) => {
    console.log(status);
  }
  .catch((error) => {
    console.log(error);
  })
```

Перевод на карту
----------------
Ничем не отличается от других переводов, за исключением того, что в account указывается номер кредитной карты:
```js
Wallet.toCard({ amount: '0.01', comment: 'test', account: '5213********0000' })
  .then((status) => {
    console.log(status);
  }
  .catch((error) => {
    console.log(error);
  })
```

Перевод на банковский счет
----------------
```js
Wallet.toBank({ amount: '0.01', account: '5213********0000', account_type: '1', exp_date: 'MMYY' }, recipient)
  .then((status) => {
    console.log(status);
  }
  .catch((error) => {
    console.log(error);
  })
```
* **ammount** - Сумма
* **account** - Номер карты/счета получателя
* **account_type** - Тип банковского идентификатора. Допустимые значения:
  * для Тинькофф Банк - карта “1”, договор “3”
  * для Альфа-Банка - карта “1”, счет “2”
  * для Промсвязьбанка - карта “7”, счет “9”
  * для банка Русский Стандарт - карта “1”, счет “2”, договор “3”
* **exp_date** - Срок действия карты, в формате ММГГ (например, 0218). Только для перевода на карту.
* **recipient** - Допустимые значения:
  * 466 - Тинькофф Банк
  * 464 - Альфа-Банк
  * 821 - Промсвязьбанк
  * 815 - Русский Стандарт

Уточнение комиссии по операции
----------------
```js
Wallet.checkComission(recipient)
  .then((comissionInfo) => {
    console.log(comissionInfo);
  }
  .catch((error) => {
    console.log(error);
  })
```
data.content.terms.commission.ranges[i]:
* **recipient** - Допустимые значения хранятся в this.recipients. Список на [официальном сайте](https://developer.qiwi.com/qiwiwallet/qiwicom_ru.html#commission).
Ответ содержит:
* **bound** - Сумма платежа, начиная с которой применяется условие
* **rate** - Комиссия (абс.множитель)
* **fixed** - Фиксированная сумма комиссии