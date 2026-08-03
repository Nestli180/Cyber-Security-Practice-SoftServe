# WebGoat — Missing Function Level Access Control

## Завдання 1. Пошук прихованих елементів

Відкриваємо урок **Missing Function Level Access Control**.

Для пошуку прихованих пунктів меню відкриваємо інструменти розробника браузера (**F12**) або натискаємо правою кнопкою миші та обираємо **Переглянути код сторінки**.

У HTML-коді знаходимо приховане меню **Admin**, яке містить два невидимі для звичайного користувача пункти:

- Users
- Config

![alt text](<1 finding_hidden_items-1.png>)

Отримані назви вводимо у поле відповіді WebGoat.

![alt text](<2 result_finding_hidden_items-1.png>)

---

## Завдання 2. Gathering User Info

Запускаємо **Burp Suite** та переходимо до вкладки **Proxy → Intercept**.

Повертаємося до WebGoat і натискаємо **Submit**. У Burp Suite натискаємо **Forward**, поки не буде перехоплено запит до:

```text
/WebGoat/access-control/user-hash
```

Після цього відправляємо запит до **Repeater**.

![alt text](<3 finding_request-1.png>)

---

У Repeater змінюємо шлях запиту на

```text
/WebGoat/access-control/users
```

та виконуємо його.

Спочатку сервер повертає статус

```text
HTTP 415 Unsupported Media Type
```

![alt text](<4 unsupported_media_type-1.png>)

Ця помилка означає, що сервер очікує інший тип даних.

Змінюємо заголовок

```text
Content-Type
```

на

```text
application/json
```

та повторно виконуємо запит.

Після цього отримуємо відповідь

```text
HTTP 400 Bad Request
```

![alt text](<5 bad_request-1.png>)

Причина полягає в тому, що використовується метод **POST**, хоча для цього endpoint необхідний **GET**.

Змінюємо метод запиту з

```text
POST
```

на

```text
GET
```

та повторно виконуємо запит.

У відповідь сервер повертає список користувачів.

![alt text](<6 list_of_users-1.png>)

Знаходимо користувача **Jerry**, копіюємо його **User Hash** та вставляємо у поле відповіді WebGoat.

![alt text](<7 result_hash-1.png>)

---

## Завдання 3. The company fixed the problem, right?

Для перевірки виправлення виконуємо GET-запит до endpoint

```text
/access-control/users-admin-fix
```

У відповідь сервер повідомляє про відсутність прав доступу.

---

Для отримання адміністративних прав використовуємо endpoint

```text
/ access-control/users
```

та змінюємо метод запиту на

```text
POST
```

У тілі запиту передаємо JSON:

```json
{
  "username": "your_username",
  "password": "your_password",
  "admin": true
}
```

де `your_username` та `your_password` необхідно замінити на облікові дані поточного користувача WebGoat.

![alt text](<8 making_admin-2.png>)

Після успішного виконання POST-запиту повторно надсилаємо GET-запит до

```text
/access-control/users-admin-fix
```

У відповідь сервер повертає потрібний **Hash** користувача **Jerry**.

![alt text](<9 result_getting_jerry_hash-2.png>)

Копіюємо отриманий хеш та вводимо його у поле відповіді WebGoat.

![alt text](<10 result_hash-1.png>)

---

## Висновок

Під час виконання лабораторної роботи було досліджено вразливість **Missing Function Level Access Control**. Було знайдено приховані елементи інтерфейсу, проаналізовано HTTP-запити за допомогою Burp Suite, отримано доступ до службових кінцевих точок та продемонстровано можливість обходу механізму контролю доступу шляхом модифікації HTTP-запитів. Це підтверджує, що приховування функціоналу лише на стороні клієнта не забезпечує належного рівня безпеки без перевірки прав доступу на сервері.