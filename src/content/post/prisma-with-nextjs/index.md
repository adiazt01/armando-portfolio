---
title: "Prisma con Next.js 14"
description: "Cómo integrar Prisma con Next.js 14 y solucionar el error de múltiples instancias de PrismaClient"
publishDate: "12 Oct 2024"
coverImage: 
    src: "./cover.png"
    alt: "Banner de Prisma"
tags: ["Prisma", "Next.js"]
---

## Prisma con Next.js 14

Prisma es un ORM (Object Relational Mapping) para bases de datos SQL y NoSQL. Prisma es una herramienta que nos permite interactuar con nuestra base de datos de una manera más sencilla y segura. Prisma nos permite escribir consultas en un lenguaje de programación seguro y tipado, y nos ayuda a evitar errores comunes en la escritura de consultas SQL.

### Inicializar un proyecto de Next.js con Prisma

Para integar Prisma con Next.js 14[^1], primero necesitamos inicializar un proyecto de Next.js. Para ello, podemos utilizar el siguiente comando:

[^1]: [Next.js 14](https://nextjs.org/blog/next-14)

```bash
npx npx create-next-app@latest
```

Una vez que hemos inicializado nuestro proyecto de Next.js, necesitamos instalar Prisma[^2] en nuestro proyecto como dependencia de desarrollo. Para ello, podemos utilizar el siguiente comando:

[^2]: [Prisma](https://www.prisma.io/)

```bash
npm install prisma --save-dev
```

### Configurar Prisma en nuestro proyecto y definir el modelo de datos para migraciones

Una vez que hemos instalado Prisma, procederemos a usar el Prisma CLI con el prefijo de `npx`, este es un comando que nos permite interactuar con Prisma desde la línea de comandos. Para inicializar Prisma en nuestro proyecto, podemos utilizar el siguiente comando:

```bash
npx prisma init
```

Si todo ha ido bien, Prisma habrá creado un directorio llamado `prisma` en la raíz de nuestro proyecto. Dentro de este directorio, encontraremos un archivo llamado `schema.prisma`, que es donde definiremos nuestro modelo de datos tal como este:

```prisma title="schema.prisma"
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

// Este es un modelo de datos de ejemplo
model User {
    id Int @id @default(autoincrement())
    name String
    email String @unique
    posts Post[]
}
```

### Configurar variables de entorno con la URL de la base de datos (Opcional)

Por motivos de simplicidad mediante un `docker-compose.yml`, podemos crear un contenedor de base de datos PostgreSQL y configurar las variables de entorno en un archivo `.env` en la raíz de nuestro proyecto.

```yml title="docker-compose.yml"
// docker-compose.yml
version: '3.1'

services:
  db:
    image: postgres
    restart: always
    environment:
      POSTGRES_USER: prisma # Usuario de la base de datos
      POSTGRES_PASSWORD: prisma # Contraseña de la base de datos
      POSTGRES_DB: prisma # Nombre de la base de datos
    ports:
      - "5432:5432"
```

Ahora montaremos el contenedor de la base de datos PostgreSQL con el siguiente comando:

```bash
docker-compose up -d
```

Una vez que el contenedor de la base de datos PostgreSQL esté en funcionamiento, podemos configurar las variables de entorno en un archivo `.env` en la raíz de nuestro proyecto. Por ejemplo, podemos hacer lo siguiente:

```env title=".env"
DATABASE_URL="postgresql://prisma:prisma@localhost:5432/prisma"
```

### Aplicar migraciones y generar el cliente de Prisma

Una vez que hemos definido nuestro modelo de datos en el archivo `schema.prisma`, necesitamos aplicar las migraciones a nuestra base de datos. Para ello, podemos utilizar el siguiente comando:

```bash
npx prisma migrate dev --name init
```

Este comando creará una migración con el nombre `init` y la aplicará a nuestra base de datos. Una vez que la migración se haya aplicado correctamente, podemos generar el cliente de Prisma ejecutando el siguiente comando:

```bash
npx prisma generate
```

### Consultas en componentes de servidor en Next.js

Con el cliente de Prisma generado, podemos comenzar a interactuar con nuestra base de datos desde nuestro proyecto de Next.js. Para ello, podemos importar el cliente de Prisma en nuestro archivo de entrada de la aplicación y utilizarlo para realizar consultas a nuestra base de datos. Por ejemplo, podemos hacer lo siguiente:

```javascript title="ListOfUsers.tsx"
// Componente del servidor
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

export default async function ListOfUsers() {
  const users = await prisma.user.findMany()
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}
```

### Error de multiples instancias de PrismaClient en Next.js 14

Si estamos utilizando Prisma con Next.js 14, es posible que nos encontremos con un error que nos indica que estamos creando múltiples instancias de `PrismaClient`[^3]. Para solucionar este error, nos ubicaremos en la carpeta `lib` de nuestro proyecto y crearemos un archivo llamado `db.ts` (Nombre a elección). Dentro de este archivo, desarrollaremos las logica para crear una única instancia de `PrismaClient` mediante el uso de un patrón singleton, de tal manera que podamos importar esta instancia en cualquier parte de nuestro proyecto.

[^3]: [Error de multiples instancias de PrismaClient en Next.js 14](
  https://www.prisma.io/docs/orm/more/help-and-troubleshooting/help-articles/nextjs-prisma-client-dev-practices
)

```javascript title="db.ts"

import { PrismaClient } from "@prisma/client";

const prismaClientSingleton = () => {
  return new PrismaClient();
};

declare const globalThis: {
  prismaGlobal: ReturnType<typeof prismaClientSingleton>;
} & typeof global;

const db = globalThis.prismaGlobal ?? prismaClientSingleton();

export default db;

if (process.env.NODE_ENV !== "production") globalThis.prismaGlobal = db;
```

Una vez que hemos creado el archivo `db.ts`, podemos importar la instancia de `PrismaClient` en cualquier parte de nuestro proyecto de Next.js. Por ejemplo, podemos hacer lo siguiente:

```javascript title="ListOfUsers.tsx"
import db from '@/lib/db'

export default async function ListOfUsers() {
  const users = await db.user.findMany()
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}
```

Con esto, hemos solucionado el error de múltiples instancias de `PrismaClient` en Next.js 14 y podemos continuar trabajando con Prisma en nuestro proyecto de Next.js.

## Conclusión

En este post hemos visto cómo integrar Prisma con Next.js 14. Hemos inicializado un proyecto de Next.js, configurado Prisma en nuestro proyecto, definido el modelo de datos para migraciones, y realizado consultas a nuestra base de datos desde componentes de servidor en Next.js. También hemos solucionado el error de múltiples instancias de `PrismaClient` en Next.js 14. Espero que este post te haya sido de utilidad y te haya ayudado a integrar Prisma con Next.js 14 en tu proyecto. ¡Gracias por leer! 🚀