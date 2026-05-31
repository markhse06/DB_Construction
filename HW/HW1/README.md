# Проектирование базы данных для сервиса доставки еды

## 1. Роли пользователей и их действия

| Роль | Описание | Действия |
|------|----------|----------|
| **Клиент** | Пользователь, заказывающий еду | • Просмотр меню<br>• Создание заказа<br>• Изменение заказа (до подтверждения)<br>• Оплата заказа<br>• Отслеживание статуса заказа<br>• Просмотр истории заказов |
| **Официант/Менеджер** | Сотрудник, принимающий/обрабатывающий заказы | • Просмотр новых заказов<br>• Подтверждение заказа<br>• Изменение статуса заказа<br>• Отмена заказа (при необходимости) |
| **Куриер** | Сотрудник, доставляющий заказы | • Просмотр назначенных заказов<br>• Обновление статуса доставки<br>• Подтверждение доставки |
| **Администратор** | Сотрудник с полным доступом | • Управление меню (добавление/редактирование/удаление блюд)<br>• Управление пользователями<br>• Просмотр всех заказов<br>• Генерация отчётов<br>• Настройка системы |

---

## 2. Объекты для хранения данных

### 2.1 Users
| Поле | Тип | Описание |
|------|-----|----------|
| `id` | INT PRIMARY KEY AUTO_INCREMENT | Уникальный идентификатор |
| `username` | VARCHAR(50) UNIQUE | Имя пользователя |
| `email` | VARCHAR(100) UNIQUE | Email |
| `phone` | VARCHAR(20) | Телефон |
| `password_hash` | VARCHAR(255) | Хеш пароля |
| `role` | ENUM('client', 'manager', 'courier', 'admin') | Роль пользователя |
| `created_at` | TIMESTAMP | Дата создания |

### 2.2 Orders
| Поле | Тип | Описание |
|------|-----|----------|
| `id` | INT PRIMARY KEY AUTO_INCREMENT | Уникальный идентификатор |
| `user_id` | INT FOREIGN KEY | Ссылка на пользователя (клиента) |
| `status` | ENUM('pending', 'confirmed', 'preparing', 'delivering', 'delivered', 'cancelled') | Статус заказа |
| `total_amount` | DECIMAL(10,2) | Сумма заказа |
| `payment_status` | ENUM('unpaid', 'paid', 'refunded') | Статус оплаты |
| `delivery_address` | TEXT | Адрес доставки |
| `courier_id` | INT FOREIGN KEY | Ссылка на курира (NULL до назначения) |
| `created_at` | TIMESTAMP | Дата создания |
| `updated_at` | TIMESTAMP | Дата последнего обновления |
| `delivered_at` | TIMESTAMP | Время доставки |

### 2.3 MenuItems
| Поле | Тип | Описание |
|------|-----|----------|
| `id` | INT PRIMARY KEY AUTO_INCREMENT | Уникальный идентификатор |
| `name` | VARCHAR(100) | Название блюда |
| `description` | TEXT | Описание |
| `price` | DECIMAL(10,2) | Цена |
| `category` | VARCHAR(50) | Категория (пицца, суши, салат и т.д.) |
| `is_available` | BOOLEAN | Доступно ли блюдо |
| `created_at` | TIMESTAMP | Дата создания |

### 2.4 OrderItems
| Поле | Тип | Описание |
|------|-----|----------|
| `id` | INT PRIMARY KEY AUTO_INCREMENT | Уникальный идентификатор |
| `order_id` | INT FOREIGN KEY | Ссылка на заказ |
| `menu_item_id` | INT FOREIGN KEY | Ссылка на блюдо |
| `quantity` | INT | Количество |
| `price_at_order` | DECIMAL(10,2) | Цена на момент заказа |

### 2.5 Payments
| Поле | Тип | Описание |
|------|-----|----------|
| `id` | INT PRIMARY KEY AUTO_INCREMENT | Уникальный идентификатор |
| `order_id` | INT FOREIGN KEY | Ссылка на заказ |
| `amount` | DECIMAL(10,2) | Сумма оплаты |
| `payment_method` | ENUM('card', 'cash', 'wallet') | Метод оплаты |
| `transaction_id` | VARCHAR(100) | ID транзакции |
| `paid_at` | TIMESTAMP | Время оплаты |

### 2.6 DeliveryStatuses
| Поле | Тип | Описание |
|------|-----|----------|
| `id` | INT PRIMARY KEY AUTO_INCREMENT | Уникальный идентификатор |
| `order_id` | INT FOREIGN KEY | Ссылка на заказ |
| `status` | ENUM('pending', 'confirmed', 'preparing', 'delivering', 'delivered', 'cancelled') | Статус |
| `changed_by` | INT FOREIGN KEY | Кто изменил (пользователь) |
| `changed_at` | TIMESTAMP | Время изменения |
| `comment` | TEXT | Комментарий (опционально) |

---

## 3. Связи между объектами

### Основные связи:

1. **Users → Orders** (1:многие)
   - Один клиент может иметь много заказов
   - `Orders.user_id` → `Users.id`

2. **Orders → OrderItems** (1:многие)
   - Один заказ содержит много позиций
   - `OrderItems.order_id` → `Orders.id`

3. **MenuItems → OrderItems** (1:многие)
   - Одно блюдо может быть в многих заказах
   - `OrderItems.menu_item_id` → `MenuItems.id`

4. **Users → Orders** (1:многие, для курира)
   - Один куриер может доставлять много заказов
   - `Orders.courier_id` → `Users.id`

5. **Orders → Payments** (1:многие)
   - Один заказ может иметь несколько оплат (предоплата, доплата, возврат)
   - `Payments.order_id` → `Orders.id`

6. **Orders → DeliveryStatuses** (1:многие)
   - Один заказ имеет истории изменения статусов
   - `DeliveryStatuses.order_id` → `Orders.id`

7. **Users → DeliveryStatuses** (1:многие)
   - Один пользователь может менять статусы многих заказов
   - `DeliveryStatuses.changed_by` → `Users.id`

---

## 4. Схема объектной модели (ER-диаграмма)

```
┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│    Users        │       │    Orders       │       │   MenuItems     │
├─────────────────┤       ├─────────────────┤       ├─────────────────┤
│ id (PK)         │◄──────│ user_id (FK)    │       │ id (PK)         │
│ username        │       │ id (PK)         │       │ name            │
│ email           │       │ status          │       │ description     │
│ phone           │       │ total_amount    │       │ price           │
│ password_hash   │       │ payment_status  │       │ category        │
│ role            │       │ delivery_address│       │ is_available    │
│ created_at      │       │ courier_id (FK) │──────►│ created_at      │
└─────────────────┘       │ created_at      │       └─────────────────┘
      ▲                   │ updated_at      │                ▲
      │                   │ delivered_at    │                │
      │                   └─────────────────┘                │
      │                         │      │                     │
      │                         │      │                     │
      │                         ▼      ▼                     │
      │              ┌─────────────────┐      ┌─────────────────┐
      │              │  OrderItems     │      │    Payments     │
      │              ├─────────────────┤      ├─────────────────┤
      │              │ id (PK)         │      │ id (PK)         │
      │              │ order_id (FK)   │──────│ order_id (FK)   │
      │              │ menu_item_id(FK)│──────│ amount          │
      │              │ quantity        │      │ payment_method  │
      │              │ price_at_order  │      │ transaction_id  │
      │              └─────────────────┘      │ paid_at         │
      │                                       └─────────────────┘
      │
      │                   ┌─────────────────┐
      └───────────────────│ DeliveryStatuses│
                          ├─────────────────┤
                          │ id (PK)         │
                          │ order_id (FK)   │──────► Orders.id
                          │ status          │
                          │ changed_by (FK) │──────► Users.id
                          │ changed_at      │
                          │ comment         │
                          └─────────────────┘
```

### Обозначения:
- **(PK)** — Primary Key (первичный ключ)
- **(FK)** — Foreign Key (внешний ключ)
- **►** — связь "многие" (many)
- **◄──────** — связь "один" (one)
- Вертикальная черта у "одной" стороны, горизонтальная у "многих"

---

## 5. Additional Notes

### Индексы для производительности:
```sql
CREATE INDEX idx_orders_user ON Orders(user_id);
CREATE INDEX idx_orders_status ON Orders(status);
CREATE INDEX idx_orders_courier ON Orders(courier_id);
CREATE INDEX idx_orderitems_order ON OrderItems(order_id);
CREATE INDEX idx_orderitems_menu ON OrderItems(menu_item_id);
CREATE INDEX idx_payments_order ON Payments(order_id);
CREATE INDEX idx_deliverystatuses_order ON DeliveryStatuses(order_id);
```

### Триггеры:
- Автоматическое обновление `updated_at` в Orders при изменении
- Автоматическое добавление записи в `DeliveryStatuses` при изменении статуса заказа
- Автоматический пересчёт `total_amount` в Orders при добавлении/изменении OrderItems
