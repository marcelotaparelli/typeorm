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


## Tipos de relacionamentos suportados
- **One-to-One (1:1)** → um registro está ligado a apenas um outro.
- **One-to-Many (1:N)** → um registro pode estar ligado a vários outros.
- **Many-to-Many (N:M)** → vários registros podem estar ligados a vários outros.

---

## Exemplo 1: One-to-Many (Usuário e Posts)
```ts
// entity/User.ts
import { Entity, PrimaryGeneratedColumn, Column, OneToMany } from "typeorm";
import { Post } from "./Post";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  nome: string;

  @OneToMany(() => Post, (post) => post.user)
  posts: Post[];
}

// entity/Post.ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from "typeorm";
import { User } from "./User";

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  titulo: string;

  @ManyToOne(() => User, (user) => user.posts)
  user: User;
}
```

### Uso
```ts
const user = new User();
user.nome = "Marcelo";
await AppDataSource.manager.save(user);

const post = new Post();
post.titulo = "Meu primeiro post";
post.user = user;
await AppDataSource.manager.save(post);
```


## Exemplo 2: Many-to-Many (Estudantes e Cursos)
```ts
// entity/Student.ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany, JoinTable } from "typeorm";
import { Course } from "./Course";

@Entity()
export class Student {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  nome: string;

  @ManyToMany(() => Course, (course) => course.students)
  @JoinTable()
  courses: Course[];
}

// entity/Course.ts
import { Entity, PrimaryGeneratedColumn, Column, ManyToMany } from "typeorm";
import { Student } from "./Student";

@Entity()
export class Course {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  titulo: string;

  @ManyToMany(() => Student, (student) => student.courses)
  students: Student[];
}
```

### Uso
```ts
const student = new Student();
student.nome = "Ana";

const course = new Course();
course.titulo = "Matemática";

student.courses = [course];

await AppDataSource.manager.save(student);
```


## Exemplo 3: One-to-One (Perfil e Usuário)
```ts
// entity/Profile.ts
import { Entity, PrimaryGeneratedColumn, Column, OneToOne } from "typeorm";
import { User } from "./User";

@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  bio: string;

  @OneToOne(() => User, (user) => user.profile)
  user: User;
}

// entity/User.ts
import { Entity, PrimaryGeneratedColumn, Column, OneToOne, JoinColumn } from "typeorm";
import { Profile } from "./Profile";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  nome: string;

  @OneToOne(() => Profile, (profile) => profile.user)
  @JoinColumn()
  profile: Profile;
}
```


## Resumo
- Use **decorators** (`@OneToMany`, `@ManyToOne`, `@ManyToMany`, `@OneToOne`) para definir relacionamentos.  
- Sempre defina o lado inverso da relação para manter consistência.  
- Use `@JoinColumn` em **One-to-One** e `@JoinTable` em **Many-to-Many**.  
- O TypeORM cuida das chaves estrangeiras e tabelas intermediárias automaticamente.


