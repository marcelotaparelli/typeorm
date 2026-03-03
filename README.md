# typeorm


## Instalação
```bash
npm install typeorm reflect-metadata
npm install pg # exemplo: PostgreSQL
```

> Obs.: instale o driver correspondente ao banco de dados que você vai usar (`mysql2`, `sqlite3`, `pg`, etc.).

## Configuração inicial
Crie um arquivo `data-source.ts` para configurar a conexão:

```ts
import "reflect-metadata";
import { DataSource } from "typeorm";
import { User } from "./entity/User";

export const AppDataSource = new DataSource({
  type: "postgres", // ou mysql, sqlite, etc.
  host: "localhost",
  port: 5432,
  username: "seu_usuario",
  password: "sua_senha",
  database: "seu_banco",
  synchronize: true, // cria tabelas automaticamente (não recomendado em produção)
  logging: true,
  entities: [User],
  migrations: [],
  subscribers: [],
});
```

## Definindo entidades
As entidades representam tabelas no banco. Exemplo `User`:

```ts
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  nome: string;

  @Column()
  email: string;
}
```


## Inicializando a conexão
No seu `index.ts` ou `app.ts`:

```ts
import { AppDataSource } from "./data-source";
import { User } from "./entity/User";

AppDataSource.initialize()
  .then(async () => {
    console.log("Conexão estabelecida!");

    // Criando um novo usuário
    const user = new User();
    user.nome = "Marcelo";
    user.email = "marcelo@example.com";
    await AppDataSource.manager.save(user);

    console.log("Usuário salvo:", user);

    // Consultando usuários
    const users = await AppDataSource.manager.find(User);
    console.log("Todos os usuários:", users);
  })
  .catch((error) => console.log(error));
```


## Migrations
Para versionar o banco de dados:

```bash
npx typeorm migration:create src/migration/NomeMigration
npx typeorm migration:run
```


## Resumo
- **Entities** → representam tabelas.  
- **Repositories/Manager** → manipulam dados (CRUD).  
- **Migrations** → controlam evolução do schema.  
- **Decorators** → definem colunas, chaves, relacionamentos.  


## Boas práticas
- Use `synchronize: false` em produção e confie em **migrations**.  
- Centralize a configuração no `DataSource`.  
- Separe entidades em uma pasta `entity/`.  
- Use `Repository` para consultas complexas em vez de `manager`.
