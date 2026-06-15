# UML-диаграммы для TicketFlow

Ниже - набор UML-диаграмм в формате PlantUML.

---

## 1. Диаграмма вариантов использования

```plantuml
@startuml
left to right direction

actor "Покупатель" as Buyer
actor "Организатор" as Organizer
actor "Администратор" as Admin
actor "Платёжный сервис" as PaymentGateway
actor "SMS-сервис" as SmsGateway

rectangle "TicketFlow" {
  usecase "Зарегистрироваться\nпо телефону" as UC_Register
  usecase "Получить SMS-код" as UC_SMS
  usecase "Найти мероприятие" as UC_Search
  usecase "Выбрать билет" as UC_SelectTicket
  usecase "Добавить билет\nв корзину" as UC_AddCart
  usecase "Удалить билет\nиз корзины" as UC_RemoveCart
  usecase "Оформить заказ" as UC_Checkout
  usecase "Оплатить заказ" as UC_Pay
  usecase "Получить электронный\nбилет" as UC_ETicket
  usecase "Просмотреть мои\nбилеты" as UC_MyTickets
  usecase "Отменить купленный\nбилет" as UC_CancelTicket
  usecase "Запросить возврат\nсредств" as UC_Refund
  usecase "Создать мероприятие" as UC_CreateEvent
  usecase "Управлять продажами" as UC_ManageSales
  usecase "Модерировать сервис" as UC_Moderate
}

Buyer --> UC_Register
Buyer --> UC_Search
Buyer --> UC_SelectTicket
Buyer --> UC_AddCart
Buyer --> UC_RemoveCart
Buyer --> UC_Checkout
Buyer --> UC_MyTickets
Buyer --> UC_CancelTicket

UC_Register --> UC_SMS : <<include>>
UC_Checkout --> UC_Pay : <<include>>
UC_Checkout --> UC_ETicket : <<include>>
UC_CancelTicket --> UC_Refund : <<include>>

SmsGateway --> UC_SMS
PaymentGateway --> UC_Pay
PaymentGateway --> UC_Refund

Organizer --> UC_CreateEvent
Organizer --> UC_ManageSales
Admin --> UC_Moderate
@enduml
```

---

## 2. Диаграмма классов

```plantuml
@startuml
class User {
  +id: string
  +name: string
  +phone: string
  +email: string
  +registerByPhone()
}

class Event {
  +id: string
  +title: string
  +category: string
  +date: Date
  +time: string
  +place: string
  +description: string
}

class TicketType {
  +id: string
  +name: string
  +price: number
  +totalLimit: number
  +availableCount: number
}

class Cart {
  +id: string
  +addTicket()
  +removeTicket()
  +changeQuantity()
  +calculateTotal()
}

class CartItem {
  +quantity: number
}

class Order {
  +id: string
  +createdAt: DateTime
  +status: OrderStatus
  +total: number
  +create()
}

class Ticket {
  +id: string
  +status: TicketStatus
  +qrCode: string
  +cancel()
}

class Payment {
  +id: string
  +amount: number
  +method: string
  +status: PaymentStatus
  +pay()
}

class RefundRequest {
  +id: string
  +reason: string
  +comment: string
  +status: RefundStatus
  +createdAt: DateTime
  +create()
}

enum OrderStatus {
  Created
  Paid
  Cancelled
}

enum TicketStatus {
  Active
  RefundRequested
  Cancelled
}

enum PaymentStatus {
  Pending
  Paid
  Failed
  Refunded
}

enum RefundStatus {
  Requested
  Approved
  Rejected
  Completed
}

User "1" -- "0..1" Cart
User "1" -- "0..*" Order
Event "1" -- "1..*" TicketType
Cart "1" -- "0..*" CartItem
CartItem "*" -- "1" Event
CartItem "*" -- "1" TicketType
Order "1" -- "1..*" Ticket
Order "1" -- "1" Payment
Ticket "*" -- "1" Event
Ticket "*" -- "1" TicketType
Ticket "1" -- "0..1" RefundRequest
@enduml
```

---

## 3. Диаграмма последовательности: покупка билета

```plantuml
@startuml
actor "Покупатель" as Buyer
participant "Интерфейс TicketFlow" as UI
participant "Каталог мероприятий" as Catalog
participant "Корзина" as Cart
participant "Заказ" as Order
participant "Платёжный сервис" as Payment
participant "Электронный билет" as ETicket

Buyer -> UI: Открывает каталог
UI -> Catalog: Запросить список мероприятий
Catalog --> UI: Список мероприятий

Buyer -> UI: Выбирает билет
UI -> Cart: Добавить билет
Cart --> UI: Корзина обновлена

Buyer -> UI: Вводит данные заказа
Buyer -> UI: Нажимает "Оплатить"
UI -> Order: Создать заказ
Order -> Payment: Провести оплату
Payment --> Order: Оплата успешна
Order -> ETicket: Создать электронный билет
ETicket --> UI: Номер заказа и QR-код
UI --> Buyer: Показать билет
@enduml
```

---

## 4. Диаграмма последовательности: отмена билета и возврат

```plantuml
@startuml
actor "Покупатель" as Buyer
participant "Интерфейс TicketFlow" as UI
participant "Мои билеты" as Tickets
participant "Сервис возвратов" as RefundService
participant "Платёжный сервис" as Payment
participant "Каталог мероприятий" as Catalog

Buyer -> UI: Открывает вкладку "Возвраты"
UI -> Tickets: Получить активные билеты
Tickets --> UI: Список активных билетов

Buyer -> UI: Выбирает билет
Buyer -> UI: Указывает причину возврата
Buyer -> UI: Нажимает "Отменить билет и запросить возврат"

UI -> RefundService: Создать запрос возврата
RefundService -> Tickets: Изменить статус билета
Tickets --> RefundService: Статус "Возврат запрошен"
RefundService -> Payment: Запросить возврат средств
Payment --> RefundService: Возврат создан
RefundService -> Catalog: Вернуть место в остатки
RefundService --> UI: Возврат запрошен
UI --> Buyer: Показать новый статус
@enduml
```

---

## 5. Диаграмма активности: оформление заказа

```plantuml
@startuml
start

:Открыть каталог мероприятий;
:Найти мероприятие;
:Выбрать тип билета;

if (Билеты доступны?) then (да)
  :Добавить билет в корзину;
else (нет)
  :Показать сообщение о недоступности;
  stop
endif

:Проверить корзину;

if (Нужно удалить билет?) then (да)
  :Удалить билет из корзины;
  :Пересчитать сумму;
endif

:Ввести имя, телефон и email;
:Выбрать способ оплаты;
:Нажать "Оплатить";

if (Оплата успешна?) then (да)
  :Создать заказ;
  :Создать электронный билет;
  :Показать билет в разделе "Мои билеты";
else (нет)
  :Показать ошибку оплаты;
endif

stop
@enduml
```

---

## 6. Диаграмма активности: возврат билета

```plantuml
@startuml
start

:Открыть вкладку "Возвраты";
:Получить список активных билетов;

if (Есть активные билеты?) then (да)
  :Выбрать билет;
  :Указать причину возврата;
  :Отправить запрос;
  :Изменить статус билета на "Возврат запрошен";
  :Создать запрос возврата средств;
  :Вернуть место в доступные остатки;
  :Показать пользователю результат;
else (нет)
  :Показать сообщение "Нет активных билетов для возврата";
endif

stop
@enduml
```
