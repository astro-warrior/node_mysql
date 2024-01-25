# Instale as dependências
```
  npm install express mysql2 typescript @types/node @types/express @types/mysql2 ts-node --save
```

# Passo 2: Configuração do TypeScript

```
npx tsc --init
```

Abra o arquivo tsconfig.json e configure-o para incluir os seguintes parâmetros:
```
{
  "compilerOptions": {
    "target": "es6",
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist"
  }
}
```
# Passo 3: Configuração do Banco de Dados
Crie um banco de dados MySQL e uma tabela para armazenar os dados. Por exemplo:
```
CREATE DATABASE IF NOT EXISTS mydatabase;
USE mydatabase;

CREATE TABLE IF NOT EXISTS users (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) NOT NULL
);
```
# Passo 4: Estrutura da Aplicação
```
- src
  - controllers
    - UserController.ts
  - routes
    - index.ts
  - db
    - connection.ts
  - app.ts
```
# Passo 5: Configuração do connection.ts

```
// src/db/connection.ts
import { createConnection } from 'mysql2/promise';

export const connection = createConnection({
  host: 'localhost',
  user: 'root',
  password: 'password',
  database: 'mydatabase',
});
```
# Passo 6: Implementação do UserController.ts
```
// src/controllers/UserController.ts
import { Request, Response } from 'express';
import { connection } from '../db/connection';

class UserController {
  async getAllUsers(req: Request, res: Response): Promise<void> {
    try {
      const [rows] = await connection.execute('SELECT * FROM users');
      res.json(rows);
    } catch (error) {
      console.error(error);
      res.status(500).json({ error: 'Internal Server Error' });
    }
  }

  async getUserById(req: Request, res: Response): Promise<void> {
    const userId = req.params.id;

    try {
      const [rows] = await connection.execute('SELECT * FROM users WHERE id = ?', [userId]);
      res.json(rows[0]);
    } catch (error) {
      console.error(error);
      res.status(500).json({ error: 'Internal Server Error' });
    }
  }

  async createUser(req: Request, res: Response): Promise<void> {
    const { name, email } = req.body;

    try {
      await connection.execute('INSERT INTO users (name, email) VALUES (?, ?)', [name, email]);
      res.json({ message: 'User created successfully' });
    } catch (error) {
      console.error(error);
      res.status(500).json({ error: 'Internal Server Error' });
    }
  }

  async updateUser(req: Request, res: Response): Promise<void> {
    const userId = req.params.id;
    const { name, email } = req.body;

    try {
      await connection.execute('UPDATE users SET name = ?, email = ? WHERE id = ?', [name, email, userId]);
      res.json({ message: 'User updated successfully' });
    } catch (error) {
      console.error(error);
      res.status(500).json({ error: 'Internal Server Error' });
    }
  }

  async deleteUser(req: Request, res: Response): Promise<void> {
    const userId = req.params.id;

    try {
      await connection.execute('DELETE FROM users WHERE id = ?', [userId]);
      res.json({ message: 'User deleted successfully' });
    } catch (error) {
      console.error(error);
      res.status(500).json({ error: 'Internal Server Error' });
    }
  }
}
export default new UserController();
```
# Passo 7: Implementação do index.ts (Rotas)
```
// src/routes/index.ts
import express from 'express';
import UserController from '../controllers/UserController';

const router = express.Router();

router.get('/users', UserController.getAllUsers);
router.get('/users/:id', UserController.getUserById);
router.post('/users', UserController.createUser);
router.put('/users/:id', UserController.updateUser);
router.delete('/users/:id', UserController.deleteUser);

export default router;
```
# Passo 8: Implementação do app.ts (Inicialização do Express)

```
// src/app.ts
import express from 'express';
import bodyParser from 'body-parser';
import cors from 'cors';
import routes from './routes';

const app = express();

// Middlewares
app.use(bodyParser.json());
app.use(cors());

// Rotas
app.use('/api', routes);

// Porta do servidor
const port = process.env.PORT || 3000;
app.listen(port, () => {
  console.log(`Server is running on port ${port}`);
});
```
# Passo 9: Compilação e Execução

Compile TypeScript para JavaScript
```
  npx tsc
```

Execute a aplicação
```
node dist/app.js
