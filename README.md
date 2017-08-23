# Qiwi development kit for PHP (QDK)
Готовый набор для быстрой интеграции возможностей Qiwi в приложения написанные с использованием PHP.

Позаботьтесь о логике ваших скриптов, а не о технических моментах Qiwi.

## Системные требования
- php >=5.4.0
- php-curl или allow_url_file_open + ssl

## Лицензия
GNU GENERAL PUBLIC LICENSE v3

Copyright (c) 2017 - Oleg Budrin (Mofsy)


# Возможности
Список всех возможностей данного набора разработчика.
Каждая возможность имеет подробное описание и относится к какой либо из нижеперечисленных групп:

- [ ] Для пользователей
- [ ] Для магазинов
- [ ] Для агентов (TopUp)
- [ ] Утилиты

## Для пользователей (Qdk\User)
Для автоматизации СВОИХ кошельков через персональные токены. 
Получение токена в разделе на сайте: <https://qiwi.com/api>

Всё что делалось раньше парсингом сайта, теперь можно автоматизировать совершенно легально.

> #### Обращение к компании Qiwi: 1
>
> Получение токена кустарное. Планируется внедрить более улучшенную авторизацию, но она по сути не нужна. Для чего крутая авторизация, если это API не предназначено для использования токенов сторонними людьми.
>
> Нужно лишь улучшить текущий интерфейс управления токенами, с переносом <https://qiwi.com/api> в интерфес настроек.
>
>  Стоит отметить ещё то, что токенов должно быть больше чем один. Сейчас нельзя сделать токен только для просмотра баланса или истории платежей. Создав новый токен, предыдущий удаляется. Можно ещё сделать выбор срока действия каждого токена или дать возможность продлевать срок действия через API.

### Дорожная карта
- [ ] Получение данных профиля
    - [ ] Получение данных в RAW формате
    - [ ] Дата создания кошелька (регистрации)
    - [ ] Статус (заблокирован, активен)
    - [ ] Уровень идентификации
    - [ ] Номер первой транзакции после регистрации
    - [ ] Привязанный email
- [ ] Получение данных о токене
    - [ ] Получение данных в RAW формате
    - [ ] Время создания
    - [ ] Время окончания
    - [ ] Права на операции (получение баланса, осуществление платежей, проверка истории)
    - [ ] Оставшийся лимит запросов к API
- [ ] Получение баланса
    - [ ] Всех кошельков в RAW формате
    - [ ] По типу кошелька
- [ ] Платежи
    - [ ] Осуществление платежей
        - [ ] На кошельки по номеру
        - [ ] В счёт провайдеров (магазинов)
    - [ ] Избранные платежи
        - [ ] Получение данных в RAW формате
        - [ ] Повтор избранного платежа
            - [ ] По идентификатору избранного платежа
            - [ ] По провайдеру (повторить все платежи к провайдеру Ромашка)
    - [ ] Регулярные платежи
        - [ ] Получение данных в RAW формате
        - [ ] Редактирование
            - [ ] Удаление платежа
                - [ ] По названию
                - [ ] По идентификатору платежа
                - [ ] По идентификатору провайдера
            - [ ] Изменение платежа
            - [ ] Создание платежа
    - [ ] История платежей
        - [ ] По типу операции (все операции, пополнение, списание, возврат)
        - [ ] По источнику платежа (тип кошелька, карта)
        - [ ] По диапазону времени (начало, конец)
        - [ ] По статусу платежа (исполнен, ошибка, в процессе, отменен)
    - [ ] Статистика платежей
        - [ ] Получение в RAW формате
        - [ ] По диапазону времени
        - [ ] По типу операции
        - [ ] По источнику платежа
    - [ ] Комиссионные тарифы для определенного провайдера
        - [ ] Получение данных в RAW формате
        - [ ] Минимально возможный платёж
        - [ ] Автоматический рассчёт комиссии по указанной сумме и источнику платежа
    - [ ] Доступные провайдеры
        - [ ] Получение данных в RAW формате
        - [ ] Фильтрация данных по категориям провайдеров

### Инициализация
```PHP
$config =
[
    'api' => 'User', // Тип API
    'token' => 'token_value' // Токен полученный вами для своего кошелька
];

$qdk = new Qdk\Core($config);
```

### Профиль
Получение данных о текущем используемом аккаунте QIWI. Метод спорный, но может пригодится.

##### Пример 1
```PHP
$profile = $qdk->query('getProfile');

$profileData = $profile->getData();
$email = $profile->getEmail();
$ip = $profile->getIp();

```
##### Пример 2
```PHP
$profile = $qdk->getApi()->getProfile();
```

### Баланс
Проверка баланса на всех типах внутренних счётов, будь то рублевый счёт кошелька или счёт в долларах. Баланс на внешних счетах не доступен.
##### Пример 1
```PHP
$balance = $qdk->query('getBalance');

// Получение баланса в Рублях
$balanceRub = $balance->getByCurrency(643);
```
##### Пример 2
```PHP
$balance = $qdk->getApi()->getBalance();

// Все данные
$balanceData = $balance->getData();

// Получение баланса в Рублях
$balanceRub = $balance->getByCurrency(643);
```

### Ваучеры (EGG, они же Яйца)
Подробнее можно почитать по ссылам https://static.qiwi.com/ru/doc/voucher.pdf и https://qiwi.com/payment/form/22496

> #### Обращение к компании Qiwi: 2
> К сожалению на данный момент минимальная комиссия за выпуск предоплаченной карты (ваучера) составляет 50 рублей. Снизить бы эту комиссию в завсимости от суммы и будет норм. Раньше комиссии вообще не было.

Действия связанные с ваучерами:
- Создание
- Удаление
- Получение информации (активиован? удален? существует? сумма?)
- Активация
- Комиссия (проверка в утилитах?)

### Статистика
Получение различной статистики по кошельку, а точнее:
- Операции за определенный период
- Сумма пополнений, списаний в отдельности

> Эти варианты можно использовать совместно. Например получить за последний день операции типа ПОПОЛНЕНИЕ.

##### Пример 1
```PHP
$stats = $qdk->query('getStats');
```
##### Пример 2
```PHP
$stats = $qdk->getApi()->getStats();
```

### Платежи


## Для магазинов (Qdk\Shop)
API предназначено для Юридических лиц и Индивидуальных предпринимателей.

### Дорожная карта
- [ ] Выставление счета
    - [ ] По номеру телефона
    - [ ] По адресу email
- [ ] Возврат платежа
    - [ ] По уникальному номеру платежа

## Для агентов (Qdk\Topup)
К сожалению на PHP не пишут для встраиваемых систем терминалов. В будущем можно будет реализовать.

### Дорожная карта
- [ ] Массовые платежи


## Утилиты (Qdk\Utility)
Доступно несколько полезных утилит, которые могут использоваться отдельно.

### Дорожная карта
- [ ] Получение информации о терминалах
- [ ] Определение мобильного оператора
- [ ] Получение данных о категориях провайдеров

### Карта терминалов
Киви - это прежде всего терминалы. Знать о местоположении терминалов важно. QDK даёт вам возможность получать такую информацию в удобном для вас виде.
Кроме местоположения доступна информация о графике работы того или иного терминала.
- Поиск по адресу
- Поиск по координатам
- Поиск по полигону

Примеры использования в examples/terminals.php

### Определение оператора
Определение сотового оператора по номеру телефона. Требуется для проведения платежей.

Примеры использования в examples/check_operator.php

### Категории провайдеров
Получение информации по доступным категориям провайдеров из каталога Qiwi.