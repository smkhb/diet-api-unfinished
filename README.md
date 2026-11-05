# 🥗 Daily Diet API

![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![Fastify](https://img.shields.io/badge/Fastify-000000?style=for-the-badge&logo=fastify&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-003B57?style=for-the-badge&logo=sqlite&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)

> A REST API to track your daily meals, mark which ones are within your diet, and check your progress with metrics — including your best streak of on-diet days. 🏆

---

## 📑 Table of Contents

- [✨ Features](#-features)
- [🧰 Tech Stack](#-tech-stack)
- [📁 Project Structure](#-project-structure)
- [🚀 Getting Started](#-getting-started)
- [⚙️ Environment Variables](#️-environment-variables)
- [🗄️ Database](#️-database)
- [📜 Scripts](#-scripts)
- [📡 API Reference](#-api-reference)
- [⚠️ Known Limitations](#️-known-limitations)
- [🤝 Contributing](#-contributing)
- [📄 License](#-license)
- [👨‍💻 Author](#-author)
- [📚 References](#-references)

---

## ✨ Features

- 👤 **Users** — create a user and list all users
- 🍽️ **Meals** — register a meal with a name, description, date/time (set automatically) and whether it is within the diet
- ✏️ **Edit** — update any of a meal's fields
- 🗑️ **Delete** — remove a meal or a user
- 🔍 **Query** — list all meals of a user, or fetch a single meal
- 📊 **Metrics** — per user:
  - 🔢 Total number of meals
  - ✅ Meals within the diet
  - ❌ Meals outside the diet
  - 🔥 Best sequence of consecutive days with on-diet meals

---

## 🧰 Tech Stack

| Layer      | Technology                                                    |
| ---------- | ------------------------------------------------------------- |
| Runtime    | [Node.js](https://nodejs.org/)                                |
| Language   | [TypeScript](https://www.typescriptlang.org/)                 |
| Framework  | [Fastify](https://fastify.dev/)                               |
| Database   | [SQLite](https://www.sqlite.org/) via [Knex.js](https://knexjs.org/) |
| Validation | [Zod](https://zod.dev/)                                       |
| Linting    | [ESLint](https://eslint.org/) with the Rocketseat config      |

---

## 📁 Project Structure

```text
diet-api/
├── db/
│   └── app.db              # SQLite database (created locally, git-ignored)
├── src/
│   ├── @types/
│   │   └── knex.d.ts       # Knex table typings
│   ├── env/
│   │   └── index.ts        # Environment variable validation (Zod)
│   ├── routes/
│   │   ├── meal.ts         # /meals endpoints
│   │   ├── metric.ts       # /metric endpoints
│   │   └── user.ts         # /users endpoints
│   ├── databse.ts          # Knex configuration and instance
│   └── server.ts           # Fastify app entry point
├── knexfile.ts             # Knex CLI configuration
├── package.json
└── tsconfig.json
```

---

## 🚀 Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) 18 or later (LTS recommended)
- npm (comes with Node.js)

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/smkhb/diet-api.git
cd diet-api

# 2. Install dependencies
npm install

# 3. Create the database folder
mkdir -p db

# 4. Create the .env file (see "Environment Variables" below)
echo "DATABASE_URL=./db/app.db" > .env
echo "NODE_ENV=development" >> .env

# 5. Create the tables (see "Database" below)

# 6. Start the development server
npm run dev
```

The server will be available at **http://localhost:3333** 🎉

---

## ⚙️ Environment Variables

Create a `.env` file in the project root:

| Variable       | Required | Default      | Description                                                                 |
| -------------- | :------: | ------------ | --------------------------------------------------------------------------- |
| `DATABASE_URL` |    ✅    | —            | Path to the SQLite database file, e.g. `./db/app.db`                        |
| `NODE_ENV`     |    ❌    | `production` | One of `development`, `production` or `test`                                |

Example `.env`:

```env
DATABASE_URL=./db/app.db
NODE_ENV=development
```

> 🔒 Never commit your `.env` file. It is already listed in `.gitignore`.

---

## 🗄️ Database

The project uses SQLite, so no database server is needed. The database is a single file, set by `DATABASE_URL`.

Create the tables with any SQLite client, for example:

```bash
sqlite3 db/app.db
```

<details>
<summary>📋 Show the schema</summary>

```sql
CREATE TABLE users (
  user_id    CHAR(36)  PRIMARY KEY,
  name       TEXT      NOT NULL,
  created_at DATETIME  NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE meals (
  meal_id     CHAR(36)  PRIMARY KEY,
  user_id     CHAR(36)  REFERENCES users(user_id) ON DELETE CASCADE,
  name        TEXT      NOT NULL,
  description TEXT      NOT NULL,
  is_in_diet  BOOLEAN   NOT NULL,
  created_at  DATETIME  NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at  DATETIME  NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

</details>

IDs are UUID v4 strings generated by the API.

---

## 📜 Scripts

| Command                 | Description                                              |
| ----------------------- | -------------------------------------------------------- |
| `npm run dev`           | 🔄 Starts the server with hot reload (`tsx watch`)       |
| `npm run knex -- <cmd>` | 🛠️ Runs the Knex CLI, e.g. `npm run knex -- migrate:latest` |
| `npm run lint`          | 🧹 Lints `src/` with ESLint and auto-fixes issues        |

To run without watch mode:

```bash
npx tsx src/server.ts
```

---

## 📡 API Reference

Base URL: `http://localhost:3333`

All request and response bodies are JSON. Successful creations return `201 Created` with an empty body.

### 👤 Users — `/users`

| Method   | Endpoint      | Description                 | Body            |
| -------- | ------------- | --------------------------- | --------------- |
| `POST`   | `/users`      | ➕ Create a user            | `{ "name": "" }` |
| `GET`    | `/users`      | 📋 List all users           | —               |
| `DELETE` | `/users/:id`  | 🗑️ Delete a user by ID      | —               |

<details>
<summary>📦 Example</summary>

```bash
curl -X POST http://localhost:3333/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Samuel"}'
```

```json
{
  "users": [
    {
      "user_id": "3f2c9d1e-8b7a-4c6d-9e0f-1a2b3c4d5e6f",
      "name": "Samuel",
      "created_at": "2024-08-07 14:30:00"
    }
  ]
}
```

</details>

### 🍽️ Meals — `/meals`

| Method   | Endpoint        | Description                                                         | Body                                                                 |
| -------- | --------------- | ------------------------------------------------------------------- | -------------------------------------------------------------------- |
| `POST`   | `/meals`        | ➕ Register a meal                                                  | `{ "user_id", "name", "description", "is_in_diet" }`                 |
| `GET`    | `/meals/:id`    | 🔍 List a user's meals, **or** fetch one meal by its ID             | —                                                                    |
| `PATCH`  | `/meals/:id`    | ✏️ Update a meal (any field is optional)                            | `{ "name"?, "description"?, "is_in_diet"? }`                         |
| `DELETE` | `/meals/:id`    | 🗑️ Delete a meal                                                    | —                                                                    |

> 💡 `GET /meals/:id` first looks for meals with `user_id = :id`. If none are found, it looks for a meal with `meal_id = :id`.

<details>
<summary>📦 Example</summary>

```bash
# Register a meal
curl -X POST http://localhost:3333/meals \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "3f2c9d1e-8b7a-4c6d-9e0f-1a2b3c4d5e6f",
    "name": "Breakfast",
    "description": "Oatmeal with banana",
    "is_in_diet": true
  }'

# Mark a meal as off-diet
curl -X PATCH http://localhost:3333/meals/<meal_id> \
  -H "Content-Type: application/json" \
  -d '{"is_in_diet": false}'
```

Response of `GET /meals/:id`:

```json
{
  "meals": [
    {
      "meal_id": "a1b2c3d4-0000-4000-8000-000000000001",
      "user_id": "3f2c9d1e-8b7a-4c6d-9e0f-1a2b3c4d5e6f",
      "name": "Breakfast",
      "description": "Oatmeal with banana",
      "is_in_diet": 1,
      "created_at": "2024-08-07 08:00:00",
      "updated_at": "2024-08-07 08:00:00"
    }
  ]
}
```

> ℹ️ SQLite stores booleans as integers, so `is_in_diet` is returned as `1` (true) or `0` (false).

</details>

### 📊 Metrics — `/metric`

All metric endpoints take a `user_id` and return a number.

| Method | Endpoint                             | Description                                  |
| ------ | ------------------------------------ | -------------------------------------------- |
| `GET`  | `/metric/totalmeals/:id`             | 🔢 Total number of meals                     |
| `GET`  | `/metric/totalmealsond/:id`          | ✅ Meals within the diet                     |
| `GET`  | `/metric/totalmealsoffd/:id`         | ❌ Meals outside the diet                    |
| `GET`  | `/metric/bestmealsodsequence/:id`    | 🔥 Best sequence of consecutive on-diet days |

<details>
<summary>📦 Example</summary>

```bash
curl http://localhost:3333/metric/totalmeals/3f2c9d1e-8b7a-4c6d-9e0f-1a2b3c4d5e6f
# 12
```

</details>

---

## ⚠️ Known Limitations

This project was built as a learning exercise. The following are known gaps:

- 🔐 **No authentication.** Users are identified by the `user_id` sent in the request. Anyone who knows a user ID can read, edit or delete that user's data.
- 🚧 **Ownership is not enforced on meals.** The requirement says a user may only view, edit and delete their own meals, but `GET`, `PATCH` and `DELETE /meals/:id` do not check the owner.
- 🐛 **`bestmealsodsequence` is incorrect.** It compares day-of-month strings with string concatenation (`"1" + 1` becomes `"11"`) and ignores month boundaries, so it returns `1` for consecutive days (e.g. days 1–12) and when a user has no meals.
- 🔌 **`PORT` is not used.** The server always listens on `3333`. The `PORT` variable is declared in `src/env/index.ts` as a number, but values from `.env` are strings, so setting `PORT` currently makes the app fail at startup.
- 🔗 **Deleting a user does not delete their meals.** The schema declares `ON DELETE CASCADE`, but SQLite only enforces foreign keys after `PRAGMA foreign_keys = ON`, which the app does not set.
- ❗ **Invalid input returns `500`** instead of `400`, because Zod validation errors are not mapped to HTTP responses.
- 🗂️ **Migrations are not versioned.** The Knex migrations folder is not in the repository, so the schema above is the source of truth for now.

---

## 🤝 Contributing

Contributions are welcome! 🎉

1. Fork the project
2. Create your branch: `git checkout -b feature/my-feature`
3. Commit your changes: `git commit -m "feat: add my feature"`
4. Push to the branch: `git push origin feature/my-feature`
5. Open a Pull Request

Please run `npm run lint` before opening a PR.

---

## 📄 License

Distributed under the MIT License. See [`LICENSE`](./LICENSE) for more information.

---

## 👨‍💻 Author

**Samuel Bernardo**

[![GitHub](https://img.shields.io/badge/GitHub-smkhb-181717?style=flat-square&logo=github)](https://github.com/smkhb)

---

## 📚 References

- 📝 [Node.js Desafio 02](https://efficient-sloth-d85.notion.site/Desafio-02-be7cdb37aaf74ba898bc6336427fa410) — the challenge this project is based on

---

<p align="center">Made with ☕ and 🥗 by <a href="https://github.com/smkhb">Samuel Bernardo</a></p>
