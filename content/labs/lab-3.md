## Тема, Мета, Місце розташування

**Тема:** Розробка функціонального REST API. Реєстрація та авторизація користувачів. Валідація даних і обробка помилок

**Мета:** 
- вивчення принципів побудови REST API;
- набуття практичних навичок розробки серверного застосунку з використанням платформи Node.js і фреймворку Express;
- реалізувати механізми реєстрації та авторизації користувачів;
- забезпечити валідацію вхідних даних;
- забезпечити обробку помилок;
- організувати захищений доступ до ресурсів із використанням JWT-токенів і системи ролей користувачів.

---

## Хід виконання

1. Встановити необхідні бібліотеки
```bash
npm install bcryptjs jsonwebtoken express-rate-limit
```
2. Реалізувати реєстрацію та авторизацію користувача
```js
app.post("/api/auth/register", async (req, res) => {
    try {
        const { email, password } = req.body;

        // Перевіряємо, чи передані всі дані
        if (!email || !password) {
            return res.status(400).json({ message: "Будь ласка, заповніть всі поля." });
        }

        // Перевіряємо, чи існує вже користувач з такою поштою
        const userExists = await User.findOne({ where: { email } });
        if (userExists) {
            return res.status(400).json({ message: "Користувач з таким email вже існує." });
        }

        // Надійно хешуємо пароль перед збереженням
        const hashedPassword = await bcrypt.hash(password, 10);

        // Записуємо нового користувача в SQL Server
        const newUser = await User.create({
            email,
            passwordHash: hashedPassword
        });

        res.status(201).json({ 
            message: "Користувача успішно зареєстровано!", 
            userId: newUser.id 
        });

    } catch (error) {
        console.error("Помилка реєстрації:", error.message);
        res.status(500).json({ message: "Помилка сервера при реєстрації.", error: error.message });
    }
});

//  МАРШРУТ: АВТОРИЗАЦІЯ (ВХІД) 
app.post("/api/auth/login", async (req, res) => {
    try {
        const { email, password } = req.body;

        if (!email || !password) {
            return res.status(400).json({ message: "Будь ласка, вкажіть email та пароль." });
        }

        // Шукаємо користувача за email
        const user = await User.findOne({ where: { email } });
        if (!user) {
            return res.status(400).json({ message: "Невірний email або пароль." });
        }

        // Порівнюємо надісланий пароль із хешем у базі
        const isMatch = await bcrypt.compare(password, user.passwordHash);
        if (!isMatch) {
            return res.status(400).json({ message: "Невірний email або пароль." });
        }

        // Генеруємо JWT токен терміном на 1 годину
        const token = jwt.sign(
            { id: user.id, email: user.email, role: user.role },
            SECRET_KEY,
            { expiresIn: "1h" }
        );

        // Повертаємо успіх та токен клієнту
        res.json({ 
            message: "Вхід успішний!", 
            token 
        });

    } catch (error) {
        console.error("Помилка входу:", error.message);
        res.status(500).json({ message: "Помилка сервера при вході.", error: error.message });
    }
});
```
![Результат](/assets/labs/lab-3/screen-1.png)
3. Додати валідацію даних, обробку помилок
```js
// === МАРШРУТ: РЕЄСТРАЦІЯ З ВАЛІДАЦІЄЮ ТА ОБРОБКОЮ ПОМИЛОК ===
app.post("/api/auth/register", async (req, res) => {
    try {
        const { email, password } = req.body;

        // 1. ВАЛІДАЦІЯ: Перевірка пустих полів
        if (!email || !password) {
            return res.status(400).json({ message: "Всі поля є обов'язковими для заповнення." });
        }

        // 2. ВАЛІДАЦІЯ: Перевірка формату email (Регулярний вираз)
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        if (!emailRegex.test(email)) {
            return res.status(400).json({ message: "Некоректний формат email адреси." });
        }

        // 3. ВАЛІДАЦІЯ: Перевірка довжини пароля
        if (password.length < 6) {
            return res.status(400).json({ message: "Пароль має бути не менше 6 символів." });
        }

        // 4. ОБРОБКА ПОМИЛОК БІЗНЕС-ЛОГІКИ: Перевірка дублікатів у базі
        const userExists = await User.findOne({ where: { email } });
        if (userExists) {
            return res.status(400).json({ message: "Користувач з таким email вже існує." });
        }

        // Хешування пароля
        const hashedPassword = await bcrypt.hash(password, 10);

        // Збереження в базу даних SQL Server
        const newUser = await User.create({
            email,
            passwordHash: hashedPassword
        });

        // Успішна відповідь (Код 201 Created)
        res.status(201).json({ 
            message: "Користувача успішно зареєстровано!", 
            userId: newUser.id 
        });

    } catch (error) {
        // 5. ГЛОБАЛЬНА ОБРОБКА ПОМИЛОК СЕРВЕРА (Блок catch)
        console.error("Критична помилка реєстрації:", error.message);
        
        res.status(500).json({ 
            message: "Внутрішня помилка сервера при реєстрації.", 
            error: error.message 
        });
    }
});
```
4. Реалізувати захищений маршрут
```js
const authMiddleware = (req, res, next) => {
    // Отримуємо заголовок Authorization із запиту
    const authHeader = req.headers["authorization"];
    
    // Токен зазвичай передається у форматі "Bearer <TOKEN>", тому розділяємо рядок
    const token = authHeader && authHeader.split(" ")[1];

    // Якщо токена взагалі немає у запиті
    if (!token) {
        return res.status(401).json({ message: "Доступ заборонено. Токен відсутній." });
    }

    try {
        // Перевіряємо токен за допомогою нашого секретного ключа
        const decoded = jwt.verify(token, SECRET_KEY);
        
        // Записуємо дані користувача з токена в об'єкт req, щоб наступний маршрут міг їх бачити
        req.user = decoded;
        
        // Пропускаємо запит далі до самого маршруту
        next();
    } catch (error) {
        // Якщо токен підроблений, застарів або зламаний
        return res.status(403).json({ message: "Невалідний або прострочений токен." });
    }
};
```
5. Протестувати API через Postman або curl (тестуємо через Thunder Client)
1) Тест валідації (Спроба реєстрації з помилками)
![Результат](/assets/labs/lab-3/screen-2.png)
2) Успішна реєстрація нового користувача
![Результат](/assets/labs/lab-3/screen-3.png)
3) Успішна авторизація (Вхід) та отримання токена
![Результат](/assets/labs/lab-3/screen-4.png)
4) Перевірка захисту (Спроба доступу без токена)
![Результат](/assets/labs/lab-3/screen-5.png)
5) Авторизований доступ до профілю (З токеном)
![Результат](/assets/labs/lab-3/screen-6.png)
6. Проаналізувати отримані результати
Практична перевірка розробленого REST API через Thunder Client підтвердила повну працездатність системи безпеки та її коректну інтеграцію з базою даних Microsoft SQL Server.

Тестування вхідного контролю показало, що при спробах надсилання некоректного email або занадто короткого пароля сервер успішно блокує запити на ранньому етапі, повертаючи статус 400 Bad Request. Це доводить ефективність налаштованої валідації, яка захищає базу даних від сміттєвих записів.

Сценарії успішної реєстрації та входу підтвердили правильність роботи криптографічного ядра: сервер коректно хешує паролі, звіряє їх при авторизації та успішно генерує захищений JWT-токен (статус 200 OK).

Перевірка захищеного маршруту профілю підтвердила надійність ізоляції даних. Спроба доступу без токена була миттєво відхилена мідлварою зі статусом 401 Unauthorized. Після додавання валідного токена в заголовок Authorization: Bearer <token> сервер успішно верифікував підпис, розпізнав користувача та надав конфіденційні дані профілю. Додаток працює стабільно і повністю захищає ресурси від несанкціонованого доступу.

7. Додати підтвердження пароля при реєстрації
```js
 const { email, password, confirmPassword } = req.body;
        if (password !== confirmPassword) {
            return res.status(400).json({ message: "Пароль та підтвердження пароля не збігаються." });
        }
```
![Результат](/assets/labs/lab-3/screen-7.png)
![Результат](/assets/labs/lab-3/screen-8.png)
8. Додати роль користувача (admin/user)
```js
const User = sequelize.define("User", {
role: { 
        type: DataTypes.STRING, 
        defaultValue: "user" 
    }
})

const adminMiddleware = (req, res, next) => {
    // req.user заповнюється попередньою мідлварою authMiddleware після дешифрації токена
    if (!req.user || req.user.role !== "admin") {
        return res.status(403).json({ message: "Доступ заборонено. Потрібні права адміністратора." });
    }
    
    // Якщо роль "admin" — пропускаємо запит далі
    next();
};

app.get("/api/admin/dashboard", authMiddleware, adminMiddleware, (req, res) => {
    res.json({
        message: "Успішно! Вітаємо в адмін-панелі сервера.",
        secretAdminData: "Ці дані бачить лише адмін."
    });
});
![Результат](/assets/labs/lab-3/screen-9.png)
```
9. Реалізувати logout
```js
app.post("/api/auth/logout", authMiddleware, async (req, res) => {
    try {
        // req.user.id ми дістаємо з Access токена, який прийшов у заголовку Authorization
        const user = await User.findByPk(req.user.id);
        
        if (!user) {
            return res.status(404).json({ message: "Користувача не знайдено." });
        }

        // Видаляємо токен оновлення з бази даних (занулюємо його)
        await user.update({ refreshToken: null });

        // Повертаємо успішну відповідь
        res.json({ 
            message: "Успішний вихід із системи. Токен анульовано в базі даних." 
        });

    } catch (error) {
        console.error("Помилка при виході:", error.message);
        res.status(500).json({ 
            message: "Внутрішня помилка сервера при спробі виходу." 
        });
    }
});
```
![Результат](/assets/labs/lab-3/screen-10.png)
10. Додати оновлення профілю
```js
// === МАРШРУТ: ОНОВЛЕННЯ ДАНИХ ПРОФІЛЮ (PUT) ===
app.put("/api/profile/update", authMiddleware, async (req, res) => {
    try {
        const { email } = req.body;
        const userId = req.user.id; // Безпечно дістаємо ID з токена

        // 1. ВАЛІДАЦІЯ: Якщо користувач прислав email, перевіряємо його формат
        if (email) {
            const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
            if (!emailRegex.test(email)) {
                return res.status(400).json({ message: "Некоректний формат email." });
            }

            // Перевіряємо, чи не зайнятий цей email кимось іншим
            const emailExists = await User.findOne({ where: { email } });
            if (emailExists && emailExists.id !== userId) {
                return res.status(400).json({ message: "Цей email вже зайнятий іншим користувачем." });
            }
        }

        // 2. ОНОВЛЕННЯ В БАЗІ ДАНИХ
        const user = await User.findByPk(userId);
        if (!user) {
            return res.status(404).json({ message: "Користувача не знайдено." });
        }

        // Оновлюємо поле, якщо воно було передано у запиті
        if (email) user.email = email;
        
        await user.save(); // Зберігаємо зміни в SQL Server

        res.json({ 
            message: "Профіль успішно оновлено!",
            profile: {
                id: user.id,
                email: user.email,
                role: user.role
            }
        });

    } catch (error) {
        console.error("Помилка оновлення профілю:", error.message);
        res.status(500).json({ message: "Помилка сервера при оновленні профілю." });
    }
});
```
11. Зберігати користувачів у базі

В межах виконання цього пункту було реалізовано персистентне зберігання облікових записів користувачів у реляційній системі керування базами даних Microsoft SQL Server за допомогою технології ORM Sequelize. Модель User абстрагує прямі SQL-запити, забезпечуючи мапінг полів об'єкта (таких як email, passwordHash, role) на відповідні типи даних у таблиці СУБД (VARCHAR, NVARCHAR, INT). Збереження записів відбувається за допомогою вбудованих асинхронних методів ORM, зокрема User.create() при реєстрації нового клієнта та user.update() чи user.save() при модифікації полів сесії чи профілю. Синхронізація схеми даних між сервером додатків та сервером баз даних автоматизована засобами Sequelize через виклик методу sequelize.sync().
```js
// === ОПИС МОДЕЛІ ДЛЯ ЗБЕРЕЖЕННЯ КОРИСТУВАЧІВ В БД ===
const User = sequelize.define("User", {
    id: { 
        type: DataTypes.INTEGER, 
        primaryKey: true, 
        autoIncrement: true 
    },
    email: { 
        type: DataTypes.STRING, 
        allowNull: false, 
        unique: true 
    },
    passwordHash: { 
        type: DataTypes.STRING, 
        allowNull: false, 
        field: "password_hash" // мапінг на колонку password_hash в базі
    },
    role: { 
        type: DataTypes.STRING, 
        defaultValue: "user" 
    },
    refreshToken: {
        type: DataTypes.STRING(500),
        allowNull: true,
        field: "refresh_token" // мапінг на колонку refresh_token в базі
    }
}, { 
    timestamps: false, // вимикаємо автоматичні колонки createdAt/updatedAt
    tableName: "users" // явна назва таблиці в MS SQL Server
});
```
12. Реалізувати refresh token
```js
// МАРШРУТ: ОНОВЛЕННЯ ACCESS ТОКЕНА ЗА ДОПОМОГОЮ REFRESH ТОКЕНА
app.post("/api/auth/refresh", async (req, res) => {
    try {
        const { token } = req.body; // Отримуємо refresh токен із тіла запиту

        // 1. ВАЛІДАЦІЯ: Перевірка наявності токена у запиті
        if (!token) {
            return res.status(401).json({ message: "Refresh токен відсутній." });
        }

        // 2. ПЕРЕВІРКА В БАЗІ ДАНИХ: Шукаємо користувача з таким токеном в SQL Server
        const user = await User.findOne({ where: { refreshToken: token } });
        if (!user) {
            return res.status(403).json({ message: "Невалідний refresh токен (відсутній у базі даних)." });
        }

        // 3. КРИПТОГРАФІЧНА ВЕРИФІКАЦІЯ: Перевіряємо підпис та термін дії токена
        jwt.verify(token, REFRESH_SECRET, (err, decoded) => {
            if (err) {
                return res.status(403).json({ message: "Термін дії токена закінчився або він пошкоджений." });
            }

            // 4. ГЕНЕРАЦІЯ НОВОГО ACCESS ТОКЕНА
            const newAccessToken = jwt.sign(
                { id: user.id, email: user.email, role: user.role },
                ACCESS_SECRET,
                { expiresIn: "15m" } // Нова перепустка знову діє 15 хвилин
            );

            // Повертаємо новий токен клієнту
            res.json({ 
                accessToken: newAccessToken 
            });
        });

    } catch (error) {
        console.error("Помилка при оновленні токена:", error.message);
        res.status(500).json({ message: "Внутрішня помилка сервера при оновленні токена." });
    }
});
```
13. Додати логування помилок

Створено допоміжну функцію logError, яка автоматично перехоплює контекст виникнення виключної ситуації: фіксує точний час події у форматі ISO, HTTP-метод запиту, цільовий URL-ендпоінт та системне повідомлення про помилку. Функцію інтегровано в усі блоки catch маршрутизатора Express. Це забезпечує безпеку інфраструктури, оскільки деталі внутрішніх збоїв (наприклад, помилки підключення до MSSQL) логуються суворо всередині консолі сервера для аудиту адміністратором, тоді як зовнішньому клієнту повертається узагальнений і безпечний статус 500 Internal Server Error без витоку технічних даних додатка.
```js
// === ФУНКЦІЯ СТРУКТУРОВАНОГО ЛОГУВАННЯ ПОМИЛОК ===
const logError = (method, path, error) => {
    const timestamp = new Date().toISOString();
    // Виводимо в консоль красивий структурований лог (можна також налаштувати запис у файл)
    console.error(`[ERROR] [${timestamp}] [${method}] ${path} -> ${error.message || error}`);
};
```
14. Обмежити кількість спроб входу

За допомогою пакетного менеджера npm (Node Package Manager) до складу залежностей було додано модуль express-rate-limit, який відповідає за підрахунок та лімітування інтенсивності HTTP-запитів з однієї IP-адреси.
```bash
npm install express-rate-limit
```
Налаштовуємо лімітер:
```js
const rateLimit = require("express-rate-limit");

// === НАЛАШТУВАННЯ ОБМЕЖЕННЯ СПРОБ ВХОДУ (RATE LIMIT) ===
const loginLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // Вікно блокування: 15 хвилин
    max: 5, // Максимальна кількість спроб з однієї IP за цей час
    message: { 
        message: "Забагато спроб входу з цієї IP-адреси. Спробуйте знову через 15 хвилин." 
    },
    standardHeaders: true, // Повертає стандартну інформацію про ліміти у заголовках RateLimit-*
    legacyHeaders: false, // Вимикає застарілі заголовки X-RateLimit-*
});
```
15. Додати middleware для перевірки токена
```js
const authMiddleware = (req, res, next) => {
    const authHeader = req.headers["authorization"];
    const token = authHeader && authHeader.split(" ")[1];

    if (!token) {
        return res.status(401).json({ message: "Доступ заборонено. Токен відсутній." });
    }

    try {
        // Верифікація за допомогою ACCESS ключа
        const decoded = jwt.verify(token, ACCESS_SECRET);
        req.user = decoded;
        next();
    } catch (error) {
        return res.status(403).json({ message: "Невалідний або прострочений access токен." });
    }
};
```
16. Реалізувати зміну пароля
```js
// === МАРШРУТ: ЗМІНА ПАРОЛЯ (PUT) ===
app.put("/api/profile/change-password", authMiddleware, async (req, res) => {
    try {
        const { oldPassword, newPassword } = req.body;
        const userId = req.user.id; // Надійно отримуємо ID користувача з токена

        // 1. ВАЛІДАЦІЯ: Перевірка пустих полів
        if (!oldPassword || !newPassword) {
            return res.status(400).json({ message: "Будь ласка, вкажіть старий та новий паролі." });
        }

        // 2. ВАЛІДАЦІЯ: Перевірка довжини нового пароля
        if (newPassword.length < 6) {
            return res.status(400).json({ message: "Новий пароль має бути не менше 6 символів." });
        }

        // 3. Шукаємо користувача в базі даних SQL Server
        const user = await User.findByPk(userId);
        if (!user) {
            return res.status(404).json({ message: "Користувача не знайдено." });
        }

        // 4. ПЕРЕВІРКА: Чи збігається старий пароль із хешем у базі
        const isMatch = await bcrypt.compare(oldPassword, user.passwordHash);
        if (!isMatch) {
            return res.status(400).json({ message: "Поточний (старий) пароль вказано невірно." });
        }

        // 5. БЕЗПЕКА: Перевірка, щоб новий пароль не збігався зі старим
        if (oldPassword === newPassword) {
            return res.status(400).json({ message: "Новий пароль не може збігатися зі старим." });
        }

        // 6. ХЕШУВАННЯ ТА ЗБЕРЕЖЕННЯ
        user.passwordHash = await bcrypt.hash(newPassword, 10);
        await user.save(); // Оновлюємо рядок в базі даних

        res.json({ 
            message: "Пароль успішно змінено!" 
        });

    } catch (error) {
        console.error("Помилка зміни пароля:", error.message);
        res.status(500).json({ message: "Помилка сервера при зміні пароля." });
    }
});
```
![Результат](/assets/labs/lab-3/screen-11.png)
17. Реалізувати видалення користувача
```js
app.delete("/api/profile/delete", authMiddleware, async (req, res) => {
    try {
        const { password } = req.body;
        const userId = req.user.id; // Безпечно витягуємо ID з токена

        // 1. ВАЛІДАЦІЯ: Перевірка, чи передано пароль
        if (!password) {
            return res.status(400).json({ message: "Будь ласка, вкажіть ваш пароль для підтвердження видалення." });
        }

        // 2. Пошук користувача в базі даних
        const user = await User.findByPk(userId);
        if (!user) {
            return res.status(404).json({ message: "Користувача не знайдено." });
        }

        // 3. ПЕРЕВІРКА: Чи збігається пароль із хешем у базі
        const isMatch = await bcrypt.compare(password, user.passwordHash);
        if (!isMatch) {
            return res.status(400).json({ message: "Невірний пароль. Видалення скасовано." });
        }

        // 4. ВИДАЛЕННЯ З БАЗИ ДАНИХ
        await user.destroy(); // Метод Sequelize для виконання SQL-запиту DELETE

        res.json({ 
            message: "Ваш обліковий запис було успішно видалено з бази даних." 
        });

    } catch (error) {
        // Використовуємо уніфіковане логування помилок
        logError("DELETE", "/api/profile/delete", error);
        res.status(500).json({ message: "Помилка сервера при спробі видалення акаунта." });
    }
});
```
18. Реалізувати відновлення пароля
```js
resetToken: {
        type: DataTypes.STRING(100),
        allowNull: true,
        field: "reset_token"
    },
    resetTokenExpires: {
        type: DataTypes.DATE,
        allowNull: true,
        field: "reset_token_expires"
    }
```
```js
app.post("/api/auth/forgot-password", async (req, res) => {
    try {
        const { email } = req.body;
        if (!email) return res.status(400).json({ message: "Будь ласка, вкажіть ваш email." });

        const user = await User.findOne({ where: { email } });
        // З міркувань безпеки, якщо email не знайдено, краще повертати успіх, 
        -- але для лабораторної та тестів повернемо 404, щоб бачити логіку
        if (!user) return res.status(404).json({ message: "Користувача з таким email не знайдено." });

        // Генеруємо випадковий унікальний токен (скидання сесії) за допомогою вбудованого модуля crypto або jwt
        const crypto = require("crypto");
        const token = crypto.randomBytes(20).toString("hex");

        // Встановлюємо термін дії токена: поточний час + 1 година
        const expires = new Date();
        expires.setHours(expires.getHours() + 1);

        // Записуємо дані в базу
        await user.update({
            resetToken: token,
            resetTokenExpires: expires
        });

        // Повертаємо токен у відповіді (імітація відправки на email)
        res.json({
            message: "Токен для відновлення пароля успішно згенеровано.",
            resetToken: token,
            info: "У реальній системі цей токен було б надіслано на вказану електронну пошту."
        });

    } catch (error) {
        logError("POST", "/api/auth/forgot-password", error);
        res.status(500).json({ message: "Помилка сервера при запиті відновлення пароля." });
    }
});

// === МАРШРУТ 2: СКИДАННЯ ПАРОЛЯ (Застосування нового пароля за токеном) ===
app.post("/api/auth/reset-password", async (req, res) => {
    try {
        const { token, newPassword } = req.body;

        if (!token || !newPassword) {
            return res.status(400).json({ message: "Необхідно надати токен та новий пароль." });
        }

        if (newPassword.length < 6) {
            return res.status(400).json({ message: "Новий пароль має бути не менше 6 символів." });
        }

        // Шукаємо користувача, у якого збігається токен І термін дії токена більший за поточний час
        const { Op } = require("sequelize");
        const user = await User.findOne({
            where: {
                resetToken: token,
                resetTokenExpires: { [Op.gt]: new Date() } // Op.gt означає "більше ніж" (Greater Than)
            }
        });

        if (!user) {
            return res.status(400).json({ message: "Токен відновлення невалідний або його термін дії закінчився." });
        }

        // Хешуємо новий пароль
        const hashedPassword = await bcrypt.hash(newPassword, 10);

        // Оновлюємо пароль та очищаємо поля токена скидання, щоб його не можна було використати вдруге
        await user.update({
            passwordHash: hashedPassword,
            resetToken: null,
            resetTokenExpires: null
        });

        res.json({ message: "Пароль успішно оновлено! Тепер ви можете увійти з новим паролем." });

    } catch (error) {
        logError("POST", "/api/auth/reset-password", error);
        res.status(500).json({ message: "Помилка сервера при скиданні пароля." });
    }
});
```
19. Додати підтвердження email

Додаємо до моделі User два нових поля: isVerified та verificationToken
```js
isVerified: {
        type: DataTypes.BOOLEAN,
        defaultValue: false,
        field: "is_verified"
    },
    verificationToken: {
        type: DataTypes.STRING(100),
        allowNull: true,
        field: "verification_token"
    }
```
Оновлення Реєстрації (app.post("/api/auth/register"))
```js
const hashedPassword = await bcrypt.hash(password, 10);
        
        // Генеруємо токен верифікації
        const crypto = require("crypto");
        const vToken = crypto.randomBytes(20).toString("hex");

        const newUser = await User.create({
            email,
            passwordHash: hashedPassword,
            isVerified: false, // Користувач спочатку не підтверджений
            verificationToken: vToken
        });

        res.status(201).json({ 
            message: "Користувача успішно зареєстровано! Будь ласка, підтвердіть ваш email.", 
            userId: newUser.id,
            verificationToken: vToken // Повертаємо токен для тестів у Thunder Client
        });
```
Додавання нового маршруту підтвердження
```js
// === МАРШРУТ: ВЕРИФІКАЦІЯ EMAIL (GET) ===
app.get("/api/auth/verify-email", async (req, res) => {
    try {
        const { token } = req.query; // Отримуємо токен з параметрів URL (?token=...)

        if (!token) {
            return res.status(400).json({ message: "Токен верифікації відсутній." });
        }

        // Шукаємо користувача з таким токеном
        const user = await User.findOne({ where: { verificationToken: token } });
        if (!user) {
            return res.status(400).json({ message: "Невалідний або вже використаний токен підтвердження." });
        }

        // Оновлюємо статус акаунта
        await user.update({
            isVerified: true,
            verificationToken: null // Очищаємо токен
        });

        res.json({ message: "Email успішно підтверджено! Тепер ви можете увійти в систему." });

    } catch (error) {
        logError("GET", "/api/auth/verify-email", error);
        res.status(500).json({ message: "Помилка сервера при підтвердженні email." });
    }
});
```
Оновлення Авторизації (app.post("/api/auth/login"))
```js
// Порівнюємо надісланий пароль із хешем у базі
        const isMatch = await bcrypt.compare(password, user.passwordHash);
        if (!isMatch) {
            return res.status(400).json({ message: "Невірний email або пароль." });
        }

        // НОВА ПЕРЕВІРКА БЕЗПЕКИ: Чи підтверджено email
        if (!user.isVerified) {
            return res.status(403).json({ message: "Доступ заборонено. Будь ласка, підтвердіть ваш email перед входом." });
        }

```
20. Реалізувати OAuth (Google login)
Встановлюємо офіційної бібліотеки Google
```bash
npm install google-auth-library
```
Код маршруту Google-авторизації в server.js
```js
const { OAuth2Client } = require("google-auth-library");
// Для лабораторної можна використати будь-який набір символів як GOOGLE_CLIENT_ID, 
// але у реальному додатку цей ID береться з Google Cloud Console
const GOOGLE_CLIENT_ID = "your-google-client-id.apps.googleusercontent.com";
const googleClient = new OAuth2Client(GOOGLE_CLIENT_ID);

// === МАРШРУТ: АВТЕНТИФІКАЦІЯ ЧЕРЕЗ OAUTH 2.0 (GOOGLE LOGIN) ===
app.post("/api/auth/google", async (req, res) => {
    try {
        const { idToken } = req.body; // Отримуємо id_token, який фронтенд забрав у Google

        if (!idToken) {
            return res.status(400).json({ message: "id_token від Google відсутній у запиті." });
        }

        // 1. ВЕРИФІКАЦІЯ: Перевіряємо токен через сервери Google
        const ticket = await googleClient.verifyIdToken({
            idToken: idToken,
            audience: GOOGLE_CLIENT_ID,
        });

        const payload = ticket.getPayload();
        const googleEmail = payload.email; // Отримуємо підтверджений email від Google

        // 2. БІЗНЕС-ЛОГІКА: Шукаємо користувача в нашій базі SQL Server
        let user = await User.findOne({ where: { email: googleEmail } });

        if (!user) {
            // Якщо користувача з таким email немає, реєструємо його автоматично.
            // Пароль генеруємо випадковий (або пустий), бо вхід буде йти тільки через Google.
            const crypto = require("crypto");
            const randomPassword = crypto.randomBytes(16).toString("hex");
            const hashedPassword = await bcrypt.hash(randomPassword, 10);

            user = await User.create({
                email: googleEmail,
                passwordHash: hashedPassword,
                isVerified: true, // Google вже верифікував цю пошту, додаткове підтвердження не потрібне
                role: "user"
            });
        }

        // 3. ГЕНЕРАЦІЯ СЕСІЇ: Виписуємо наші локальні JWT токени додатка
        const accessToken = jwt.sign(
            { id: user.id, email: user.email, role: user.role },
            ACCESS_SECRET,
            { expiresIn: "15m" }
        );

        const refreshToken = jwt.sign(
            { id: user.id },
            REFRESH_SECRET,
            { expiresIn: "7d" }
        );

        // Оновлюємо refresh_token в базі даних
        await user.update({ refreshToken });

        res.json({
            message: "Вхід через Google успішний!",
            user: { id: user.id, email: user.email, role: user.role },
            accessToken,
            refreshToken
        });

    } catch (error) {
        // Логуємо помилку криптографічного підпису Google
        logError("POST", "/api/auth/google", error);
        res.status(400).json({ message: "Невалідний id_token від Google. Авторизацію відхилено." });
    }
});
```
![Результат](/assets/labs/lab-3/screen-12.png)

---

## Висновки
У процесі виконання лабораторної роботи було спроєктовано та успішно реалізовано комплексну серверу систему автентифікації та авторизації для вебдодатка кав'ярні на базі платформи Node.js, фреймворку Express та СУБД Microsoft SQL Server (через ORM Sequelize).