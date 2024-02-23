# Nest + Docker + PGSQL + TypeOrm

- [x] Init Nest project
- [x] Create Dockerfile
- [x] Create docker-compose.yml
- [x] Create .dockerignore
- [x] Build the containers
- [x] Setting up the PG database
- [x] Install the TypeORM dependencies
- [x] Import TypeORM at the app.module.ts
- [x] Create a new REST module
- [x] Create the user entity
- [x] Inject the user entity on the TypeORMModule (users.module.ts)
- [x] Create the request and response DTOs
- [x] Apply the DTOs on the controllers
- [x] Apply the DTOs on the services

## Initiating a new Nest project

```bash
nest new project-name
```

## Creating the Dockerfile

```Dockerfile
FROM node:18

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

CMD [ "npm", "run", "start:dev"
```

## Create docker-compose.yml

```docker-compose.yml
version: '3.5'

services:
  db:
    image: postgres
    restart: always
    environment:
      - POSTGRES_PASSWORD=postgres
    container_name: postgres
    volumes:
      - ./pgdata:/var/lib/postgresql/data
    ports:
      - '7654:7654'

  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: nest-docker-postgres
    environment:
      - PORT=${PORT}
    ports:
      - '3000:3000'
    depends_on:
      - db
    volumes:
      - ./src:/app/src

  pgadmin:
    image: dpage/pgadmin4
    restart: always
    container_name: nest-pgadmin4
    environment:
      - PGADMIN_DEFAULT_EMAIL=admin@admin.com
      - PGADMIN_DEFAULT_PASSWORD=pgadmin4
    ports:
      - '5050:80'
    depends_on:
      - db
```

## Creating .dockerignore

```dockerignore
Dockerfile
.dockerignore
node_modules
npm-debug.log
dist
```


## Building the containers

```bash
docker compose up --build
```

## Setting up the PG database

To set up your Postgres database you must access `http://localhost:5050` (pgadmin) and log in using the credentials you have specified at the `docker-compose.yml` file.

In this case, we are going to use the following credentials:

```.env
EMAIL=admin@admin.com
PASSWORD=pgadmin4
```

After logging in, you must create a new server (Right click Servers > Register > Server), go to the Connection tab and fill in the following fields:

```
Host name/address: db
Port: 5432
Maintenance database: postgres
Username: postgres
Password: postgres
```

## Installing the TypeORM dependencies

```bash
npm install @nestjs/typeorm typeorm pg class-validators
```

## Importing TypeORM at the app.module.ts

```typescript app.module.ts
imports: [
  TypeOrmModule.forRoot({
    type: 'postgres',
    host: 'db',
    port: 5432,
    username: 'postgres',
    password: 'postgres',
    database: 'postgres',
    entities: [],
    synchronize: true,
    autoLoadEntities: true,
  }),
],
```

## Creating a new REST module for Users

```bash
nest g rest users
```

## Creating the user entity

```typescript user.entity.ts
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class UserEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column({ default: true })
  isActive: boolean;
}
```

## Injecting the user entity on the TypeORMModule (users.module.ts)

```typescript users.module.ts
imports: [TypeOrmModule.forFeature([UserEntity])],
```

## Creating the request DTO

```typescript request-user.dto.ts
import { IsBoolean, IsNotEmpty, IsString } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @IsNotEmpty()
  firstName: string;

  @IsString()
  @IsNotEmpty()
  lastName: string;

  @IsBoolean()
  isActive: boolean;
}
```

## Creating the response DTO

```typescript response-user.dto.ts
import { UserEntity } from './../entities/user.entity';

export class ResponseUserDTO {
  id: number;
  firstName: string;
  lastName: string;
  isActive: boolean;

  constructor(user: Partial<UserEntity>) {
    this.id = user.id;
    this.firstName = user.firstName;
    this.lastName = user.lastName;
    this.isActive = user.isActive;
  }
}
```
