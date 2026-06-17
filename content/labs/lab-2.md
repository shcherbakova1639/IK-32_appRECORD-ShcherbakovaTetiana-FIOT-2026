## Тема, Мета, Місце розташування

**Тема:** «СТВОРЕННЯ БАЗИ ДАНИХ У MYSQL. ПІДКЛЮЧЕННЯ NODE.JS ДО MYSQL. РОБОТА З ORM SEQUELIZE.

**Мета:** 
- Навчитися створювати базу даних у MySQL
- Освоїти виконання SQL-запитів (SELECT, INSERT, UPDATE, DELETE)
- Підключати серверну програму на Node.js до бази даних
- Використовувати ORM Sequelize для роботи з БД
- Реалізувати зв’язок One-to-Many між таблицями

---

## Хід виконання
1) Створюємо базу даних, таблиці користувачів і постів 

```sql
-- 1. Створення бази даних
CREATE DATABASE web_backend_lab;
GO
USE web_backend_lab;
GO

-- 2. Створення таблиці користувачів (users)
CREATE TABLE users (
    id INT IDENTITY(1,1) PRIMARY KEY, 
    name NVARCHAR(255) NOT NULL,
    group_name NVARCHAR(50) NOT NULL,
    created_at DATETIME DEFAULT GETDATE()
);

-- 2. Створення таблиці постів (posts) з прив'язкою One-to-Many
CREATE TABLE posts (
    id INT IDENTITY(1,1) PRIMARY KEY,
    title NVARCHAR(255) NOT NULL,
    content NTEXT NOT NULL,
    user_id INT,
    created_at DATETIME DEFAULT GETDATE(),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

2) Виконуємо запити
INSERT
```sql
INSERT INTO users (name, group_name) VALUES (N'Щербакова Тетяна', 'ІК-32');
INSERT INTO users (name, group_name) VALUES (N'Іван Іванов', 'ІК-31');

INSERT INTO posts (title, content, user_id) VALUES (N'Перше замовлення', N'Кава супер, круасани гарячі!', 1);
INSERT INTO posts (title, content, user_id) VALUES (N'Чудовий заклад', N'Швидке обслуговування', 2);
```

SELECT
```sql
-- Вибірка всіх постів із відображенням імені користувача, який їх написав (Об'єднання таблиць)
SELECT 
    posts.id AS PostID, 
    posts.title AS PostTitle, 
    posts.content AS PostContent, 
    users.name AS AuthorName,
    users.group_name AS AuthorGroup
FROM posts
INNER JOIN users ON posts.user_id = users.id;
```

![Результат](/assets/labs/lab-2/screen-1.png)

UPDATE
```sql
-- Зміна назви групи для конкретного користувача за його унікальним ID
UPDATE users 
SET group_name = 'ІК-32 (Магістри)' 
WHERE id = 2;

-- Модифікація тексту конкретного поста
UPDATE posts 
SET content = N'Кава просто супер, а круасани завжди гарячі! Рекомендую какао.' 
WHERE id = 1;
```

DELETE
```sql
-- Видалення конкретного поста за його ідентифікатором
DELETE FROM posts 
WHERE id = 2;
```

3) Підключаємо Node.js, використовуємо ORM Sequelize
```js
const { Sequelize, DataTypes } = require("sequelize");
const sequelize = new Sequelize("web_backend_lab", "sa", "1111", { 
    host: "localhost",
    port: 1433,
    dialect: "mssql",
    dialectOptions: {
        options: {
            encrypt: true,
            trustServerCertificate: true
        }
    },
    logging: false
});
```

4) Виконуємо SQL-запити з Node
```js
// 1. GET /users — Отримати всіх користувачів разом з їхніми постами (SELECT + JOIN)
app.get("/users", async (req, res) => {
    try {
        const users = await User.findAll({
            include: [{ model: Post, as: "posts" }]
        });
        res.json(users);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// 2. POST /users — Додати нового користувача (INSERT)
app.post("/users", async (req, res) => {
    try {
        const { name, group } = req.body;
        const newUser = await User.create({ name, group });
        res.status(201).json(newUser);
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
});

// 3. POST /users/:id/posts — Додати новий пост для конкретного користувача (INSERT)
app.post("/users/:id/posts", async (req, res) => {
    try {
        const userId = req.params.id;
        const { title, content } = req.body;
        const newPost = await Post.create({ title, content, user_id: userId });
        res.status(201).json(newPost);
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
});
```
5) Моделі User та Post

```js
// Модель Студента/Користувача
const User = sequelize.define("User", {
    id: {
        type: DataTypes.INTEGER,
        primaryKey: true,
        autoIncrement: true
    },
    name: {
        type: DataTypes.STRING,
        allowNull: false
    },
    group: {
        type: DataTypes.STRING,
        allowNull: false,
        field: "group_name" // мапінг на колонку group_name в базі
    }
}, { timestamps: false, tableName: "users" });

// Модель Поста/Відгука
const Post = sequelize.define("Post", {
    id: {
        type: DataTypes.INTEGER,
        primaryKey: true,
        autoIncrement: true
    },
    title: {
        type: DataTypes.STRING,
        allowNull: false
    },
    content: {
        type: DataTypes.TEXT,
        allowNull: false
    }
}, { timestamps: false, tableName: "posts" });
```

6) Реалізація зв’язку One-to-Many
```js
User.hasMany(Post, { foreignKey: "user_id", as: "posts" });
Post.belongsTo(User, { foreignKey: "user_id", as: "user" });
```
![Результат](/assets/labs/lab-2/screen-2.png)

---

## Висновки
Під час виконання лабораторної роботи №2 було поглиблено та закріплено практичні навички роботи з реляційними базами даних, мовою структурованих запитів SQL та сучасними технологіями об'єктно-реляційного відображення (ORM) в середовищі розробки Node.js.