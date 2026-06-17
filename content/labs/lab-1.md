## Тема, Мета, Місце розташування

**Тема:** Веб-орієнтований застосунок кав'ярні “CoffeeShop”.

**Мета:** Створити сучасний та інтуїтивно зрозумілий веб-сервіс для кав'ярні “CoffeeShop”, який надає клієнтам можливість комфортно ознайомитися з картою напоїв та випічки, дистанційно оформити предзамовлення та налаштувати рецептуру кави під власні смаки. Застосунок має запропонувати естетичний та адаптивний дизайн, детально продемонструвати позиції меню за допомогою якісних фото і описів, а також мінімізувати час очікування клієнта в самому закладі.

**Місце розташування:**
- GitHub: [Посилання](https://github.com/shcherbakova1639/IK-32_appRECORD-ShcherbakovaTetiana-FIOT-2026)
- Власний веб-застосунок(GitHub): [Посилання](https://github.com/shcherbakova1639/CoffeeShop)

---

## Опис предметного середовища

Предметна галузь:
Кав'ярня “CoffeeShop”. Сайт надає актуальне меню напоїв та десертів, інформацію про заклад, контактні дані, графік роботи та можливість оформити попереднє замовлення онлайн для самовивозу.

Структура веб-застосунку
- Головна сторінка — вітальний блок, акційні пропозиції та коротка інформація про кав'ярню.
- Меню — перелік кавових напоїв, чаю та десертів з назвою, зображенням, ціною й описом.
- Про нас — відомості про концепцію закладу, якість зерен та команду бариста.
- Відгуки — перегляд та додавання відгуків від відвідувачів.
- Контакти — адреса кав'ярні, інтерактивна карта, графік роботи, телефон та соціальні мережі.

Сценарій взаємодії (бізнес-логіка)
- Користувач відкриває сайт і переходить у розділ Меню.
- Ознайомлюється з асортиментом кави та обирає потрібний напій або десерт.
- Заповнює просту форму попереднього замовлення (Ім'я, телефон, бажаний час, коли він забере замовлення, примітки щодо цукру чи молока).
- Кав'ярня отримує замовлення на екран бариста, який одразу починає його готувати до вказаного часу.
- Користувач приходить у заклад, забирає готову гарячу каву без очікування в черзі та оплачує її.
- Після візиту користувач може залишити відгук про сервіс на сайті.

Функціональні вимоги
- Відображення меню товарів з цінами, фото та описами складових.
- Перегляд детальної інформації про кожен напій (об'єм, калорійність).
- Форма оформлення попереднього замовлення на певний час.
- Форма додавання відгуку про роботу кав'ярні.
- Розділи «Про нас» і «Контакти» доступні з головного навігаційного меню.
- Адаптивна верстка для зручного користування з мобільних телефонів на ходу.

Нефункціональні вимоги
- Простий, затишний та інтуїтивно зрозумілий інтерфейс.
- Швидке завантаження сторінок — орієнтовно до 2–3 секунд.
- Коректна передача та валідація даних із форм (наприклад, перевірка формату телефону).
- Стабільна робота у всіх сучасних браузерах (Chrome, Safari, Edge, Firefox).
- Використання оптимізованих, якісних зображень кави та інтер'єру.

Стек технологій
- HTML5 — семантична розмітка сторінок (header, main, footer) і контент-блоків.
- CSS3 — стилізація, використання сіток Flexbox/Grid для карток меню, адаптивність через медіа-запити (media queries).
- JavaScript (Vanilla) — логіка роботи адаптивного бургер-меню, плавний скрол, валідація форм замовлення.
- Git + GitHub — контроль версій для командної або індивідуальної розробки.
- GitHub Pages / Vercel — публікація звітного документа та самого сайту в мережі.
- Assets — зображення товарів, фони та іконки (папки зі статичними ресурсами).

---

## Хід виконання

### Частина 1

1) Головна сторінка index.html
```html
<!DOCTYPE html>
<html lang="uk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CoffeeShop — Головна</title>
    <link rel="stylesheet" href="../css/style.css">
</head>
<body>

    <header class="header">
        <div class="header-container">
            <a href="index.html" class="logo">Coffee<span>Shop</span></a>
            
            <button class="burger-btn" id="burgerBtn" aria-label="Відкрити меню">
                <span class="burger-bar bar1"></span>
                <span class="burger-bar bar2"></span>
                <span class="burger-bar bar3"></span>
            </button>

            <ul class="nav-menu" id="navMenu">
                <li><a href="index.html" class="nav-link active">Головна</a></li>
                <li><a href="menu.html" class="nav-link">Меню</a></li>
                <li><a href="#order" class="nav-link">Замовлення</a></li>
                <li><a href="#contacts" class="nav-link">Контакти</a></li>
            </ul>
        </div>
    </header>

    <main>
        <div class="container">
            <section class="hero">
                <h1>Твоя ідеальна кава чекає на тебе</h1>
                <p>Замовляй онлайн заздалегідь та забирай без черги дорогою на навчання чи роботу.</p>
                <a href="menu.html" class="btn">Перейти до Меню</a>
            </section>

            <section id="order" class="order-section">
                <h2 class="section-title" style="margin-bottom: 1.5rem;">Швидке передзамовлення</h2>
                <form id="coffeeForm">
                    <div class="form-group">
                        <label for="name">Ваше ім'я</label>
                        <input type="text" id="name" class="form-control" placeholder="Введіть ім'я" required>
                    </div>
                    <div class="form-group">
                        <label for="phone">Номер телефону</label>
                        <input type="tel" id="phone" class="form-control" placeholder="+380" required>
                    </div>
                    <div class="form-group">
                        <label for="time">Бажаний час отримання</label>
                        <input type="time" id="time" class="form-control" required>
                    </div>
                    <div class="form-group">
                        <label for="note">Примітки до замовлення</label>
                        <textarea id="note" class="form-control" rows="3" placeholder="Наприклад: без цукру, на вівсяному молоці..."></textarea>
                    </div>
                    <button type="submit" class="btn" style="width: 100%;">Надіслати замовлення</button>
                </form>
            </section>
        </div>
    </main>

    <footer class="footer" id="contacts">
        <div class="footer-grid">
            <div class="footer-col">
                <h3>CoffeeShop</h3>
                <p>Твій затишний простір якісної кави та свіжої випічки щодня.</p>
            </div>
            <div class="footer-col">
                <h3>Контакти</h3>
                <p>📍 вул. Університетська, 15</p>
                <p>📞 +38 (099) 123-45-67</p>
                <p>✉️ info@coffeeshop.ua</p>
            </div>
            <div class="footer-col">
                <h3>Графік роботи</h3>
                <p>Пн-Пт: 07:30 – 21:00</p>
                <p>Сб-Нд: 09:00 – 22:00</p>
            </div>
        </div>
        <div class="footer-bottom">
            <p>&copy; 2026 CoffeeShop. Усі права захищено. Студентський проєкт KPI.</p>
        </div>
    </footer>

    <script src="../js/script.js"></script>
</body>
</html>
```

2) Сторінка меню menu.html
```html
<!DOCTYPE html>
<html lang="uk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CoffeeShop — Меню</title>
    <link rel="stylesheet" href="../css/style.css">
</head>
<body>

    <header class="header">
        <div class="header-container">
            <a href="index.html" class="logo">Coffee<span>Shop</span></a>
            
            <button class="burger-btn" id="burgerBtn" aria-label="Відкрити меню">
                <span class="burger-bar bar1"></span>
                <span class="burger-bar bar2"></span>
                <span class="burger-bar bar3"></span>
            </button>

            <ul class="nav-menu" id="navMenu">
                <li><a href="index.html" class="nav-link">Головна</a></li>
                <li><a href="menu.html" class="nav-link active">Меню</a></li>
                <li><a href="index.html#order" class="nav-link">Замовлення</a></li>
                <li><a href="index.html#contacts" class="nav-link">Контакти</a></li>
            </ul>
        </div>
    </header>

    <main style="padding-top: 4rem;">
        <div class="container">
            <h2 class="section-title">Наше Меню</h2>
            
            <div class="menu-grid">
                <article class="menu-card">
                    <img src="https://images.unsplash.com/photo-1509042239860-f550ce710b93?q=80&w=400" alt="Капучино" class="card-img">
                    <div class="card-content">
                        <div class="card-title">
                            <span>Капучино</span>
                            <span class="card-price">55 ₴</span>
                        </div>
                        <p class="card-desc">Класичний кавовий напій з пишною молочною пінкою на основі подвійного еспресо.</p>
                        <button class="btn" style="padding: 0.5rem 1rem; font-size: 0.9rem;">В кошик</button>
                    </div>
                </article>

                <article class="menu-card">
                    <img src="https://images.unsplash.com/photo-1514432324607-a09d9b4aefdd?q=80&w=400" alt="Лате" class="card-img">
                    <div class="card-content">
                        <div class="card-title">
                            <span>Лате Макіато</span>
                            <span class="card-price">60 ₴</span>
                        </div>
                        <p class="card-desc">Ніжний молочний смак з легким відтінком добірного кавового зерна арабіки.</p>
                        <button class="btn" style="padding: 0.5rem 1rem; font-size: 0.9rem;">В кошик</button>
                    </div>
                </article>

                <article class="menu-card">
                    <img src="https://images.unsplash.com/photo-1607958996333-41aef7caefaa?q=80&w=400" alt="Круасан" class="card-img">
                    <div class="card-content">
                        <div class="card-title">
                            <span>Круасан з мигдалем</span>
                            <span class="card-price">65 ₴</span>
                        </div>
                        <p class="card-desc">Хрустка французька випічка з ніжною мигдалевою начинкою всередині.</p>
                        <button class="btn" style="padding: 0.5rem 1rem; font-size: 0.9rem;">В кошик</button>
                    </div>
                </article>
            </div>
        </div>
    </main>

    <footer class="footer">
        <div class="footer-bottom" style="border: none; padding: 0;">
            <p>&copy; 2026 CoffeeShop</p>
        </div>
    </footer>

    <script src="../js/script.js"></script>
</body>
</html>
```

3) Cтиль
```css
/* --- Базові та скидаючі стилі --- */
* {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}

:root {
    --primary-color: #4a2c11; /* Кавовий */
    --accent-color: #d4a373;  /* Кремовий/золотавий */
    --text-dark: #2b1a08;
    --bg-light: #fefae0;
    --white: #ffffff;
    --font-main: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

body {
    font-family: var(--font-main);
    color: var(--text-dark);
    background-color: var(--bg-light);
    line-height: 1.6;
}

a {
    text-decoration: none;
    color: inherit;
    transition: color 0.3s ease;
}

img {
    max-width: 100%;
    height: auto;
    display: block;
}

/* --- Шапка (Header) --- */
.header {
    background-color: var(--primary-color);
    color: var(--white);
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    z-index: 1000;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.header-container {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem 2rem;
    max-width: 1200px;
    margin: 0 auto;
}

.logo {
    font-size: 1.5rem;
    font-weight: bold;
    letter-spacing: 1px;
}

.logo span {
    color: var(--accent-color);
}

/* Навігація для ПК */
.nav-menu {
    display: flex;
    list-style: none;
    gap: 2rem;
}

.nav-link:hover, .nav-link.active {
    color: var(--accent-color);
}

/* Кнопка бургер-меню (прихована на ПК) */
.burger-btn {
    display: none;
    background: none;
    border: none;
    cursor: pointer;
    flex-direction: column;
    gap: 6px;
    z-index: 1001;
}

.burger-bar {
    width: 25px;
    height: 3px;
    background-color: var(--white);
    transition: all 0.3s ease;
}

/* --- Основний блок (Main) --- */
main {
    margin-top: 60px; /* Відступ під фіксовану шапку */
}

.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 2rem 1rem;
}

/* Hero секція */
.hero {
    background: linear-gradient(rgba(0,0,0,0.5), rgba(0,0,0,0.6)), url('https://images.unsplash.com/photo-1501339847302-ac426a4a7cbb?q=80&w=1200') no-repeat center center/cover;
    color: var(--white);
    text-align: center;
    padding: 5rem 1rem;
    border-radius: 8px;
    margin-bottom: 3rem;
}

.hero h1 {
    font-size: 2.5rem;
    margin-bottom: 1rem;
}

.hero p {
    font-size: 1.2rem;
    margin-bottom: 1.5rem;
    color: #f1f1f1;
}

.btn {
    display: inline-block;
    background-color: var(--accent-color);
    color: var(--primary-color);
    padding: 0.8rem 2rem;
    border-radius: 4px;
    font-weight: bold;
    border: none;
    cursor: pointer;
    transition: transform 0.2s ease, background-color 0.3s ease;
}

.btn:hover {
    transform: translateY(-2px);
    background-color: #e9c46a;
}

/* Секція Меню (Grid-сітка) */
.section-title {
    text-align: center;
    font-size: 2rem;
    margin-bottom: 2rem;
    position: relative;
    color: var(--primary-color);
}

.menu-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 2rem;
    margin-bottom: 4rem;
}

.menu-card {
    background-color: var(--white);
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 4px 6px rgba(0,0,0,0.05);
    transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.menu-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 8px 15px rgba(0,0,0,0.1);
}

.card-img {
    height: 200px;
    object-fit: cover;
    width: 100%;
}

.card-content {
    padding: 1.5rem;
}

.card-title {
    font-size: 1.3rem;
    margin-bottom: 0.5rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.card-price {
    color: var(--primary-color);
    font-weight: bold;
}

.card-desc {
    font-size: 0.9rem;
    color: #666;
    margin-bottom: 1rem;
}

/* --- Форма замовлення --- */
.order-section {
    background-color: var(--white);
    padding: 2rem;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0,0,0,0.05);
    max-width: 600px;
    margin: 0 auto 4rem auto;
}

.form-group {
    margin-bottom: 1.2rem;
}

.form-group label {
    display: block;
    margin-bottom: 0.5rem;
    font-weight: 600;
}

.form-control {
    width: 100%;
    padding: 0.8rem;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-family: inherit;
}

.form-control:focus {
    outline: none;
    border-color: var(--primary-color);
}

/* --- Нижній колонтитул (Footer) --- */
.footer {
    background-color: var(--primary-color);
    color: var(--white);
    padding: 3rem 1rem 1.5rem 1rem;
    margin-top: auto;
}

.footer-grid {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    max-width: 1200px;
    margin: 0 auto;
    gap: 2rem;
    border-bottom: 1px solid #613c19;
    padding-bottom: 2rem;
}

.footer-col {
    flex: 1;
    min-width: 250px;
}

.footer-col h3 {
    color: var(--accent-color);
    margin-bottom: 1rem;
}

.footer-col p {
    font-size: 0.9rem;
    margin-bottom: 0.5rem;
}

.footer-bottom {
    text-align: center;
    padding-top: 1.5rem;
    font-size: 0.85rem;
    color: #bbb;
}

/* --- Медіа-запити (Media Queries) для адаптивності --- */
@media (max-width: 768px) {
    .burger-btn {
        display: flex; /* Показуємо бургер на мобільних */
    }

    /* Трансформація меню у випадаючу панель */
    .nav-menu {
        position: fixed;
        top: 0;
        right: -100%;
        width: 70%;
        height: 100vh;
        background-color: var(--primary-color);
        flex-direction: column;
        align-items: center;
        justify-content: center;
        gap: 2.5rem;
        transition: right 0.4s ease;
        box-shadow: -5px 0 15px rgba(0,0,0,0.2);
    }

    .nav-menu.active {
        right: 0; /* Виїжджає при кліку */
    }

    .hero h1 {
        font-size: 2rem;
    }

    /* Анімація іконки бургера при відкритті */
    .burger-btn.open .bar1 {
        transform: rotate(-45deg) translate(-5px, 6px);
    }
    .burger-btn.open .bar2 {
        opacity: 0;
    }
    .burger-btn.open .bar3 {
        transform: rotate(45deg) translate(-5px, -6px);
    }
}
```
4) Скрипти
```js
const burgerBtn = document.getElementById('burgerBtn');
const navMenu = document.getElementById('navMenu');

// Відкриття/Закриття мобільного меню за кліком на бургер
burgerBtn.addEventListener('click', () => {
    navMenu.classList.toggle('active');
    burgerBtn.classList.toggle('open');
});

// Закриття меню при кліку на будь-яке посилання
document.querySelectorAll('.nav-link').forEach(link => {
    link.addEventListener('click', () => {
        navMenu.classList.remove('active');
        burgerBtn.classList.remove('open');
    });
});

// Проста обробка відправки форми замовлення
document.getElementById('coffeeForm').addEventListener('submit', (e) => {
    e.preventDefault();
    alert('Дякуємо! Бариста вже отримав замовлення і починає його готувати.');
    e.target.reset();
});
```

### Частина 2

1. HTTP-сервер, який повертає повідомлення
![](/assets/labs/lab-1/screen-3.jpg)
2. Маршрут, який повертає список студентів у форматі JSON
![](/assets/labs/lab-1/screen-4.jpg)
3. Маршрут для додавання нового студента
POST /students
![](/assets/labs/lab-1/screen-5.png)
4. Маршрут для оновлення даних
PUT /students/:id
![](/assets/labs/lab-1/screen-6.jpg)
![](/assets/labs/lab-1/screen-7.jpg)
5. Маршрут  для видалення студента
DELETE /students/:id
![](/assets/labs/lab-1/screen-8.jpg)
---

## Скріншоти
1. Головна сторінка
![Головна сторінка](/assets/labs/lab-1/screen-1.png)
2. Меню
![Меню](/assets/labs/lab-1/screen-2.png)
---

## Висновки

У ході оформлення звіту для “CoffeeShop” було вибудовано чітку структуру сторінок, застосовано семантичну HTML5-розмітку та додано приклади коду, таблиць, зображень і форми замовлення, що повністю відтворюють реальний вміст розробленого проєкту кав'ярні. Створений звіт є цілісним, зрозумілим та придатним до публікації на платформі GitHub Pages як статичний сайт.
