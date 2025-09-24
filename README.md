# Next.js-TS-Prisma-MongoDB REST API starter
https://www.prisma.io/docs/guides/nextjs

## 1. Projekt inicializálása a reate-next-app sablonnal
>npx create-next-app@latest --api --eslint<br>

Majd interaktív lépések

> What is your project named? **next_api_starter**<br>
> Would you like to use TypeScript? No / **Yes**<br>
> Would you like your code inside a `src/` directory? **No** / Yes<br>
> Would you like to use Turbopack for `next dev`?  **No** / Yes<br>
> Would you like to customize the import alias (`@/*` by default)? **No** / Yes<br>
> What import alias would you like configured? @/*<br>


## 2. További külső csomagok telepítése, Prisma inicializálása
> npm i -D prisma typescript-eslint<br>
> npm i -g tsx<br>
> npm i @prisma/client<br>
> npx prisma init --datasource-provider mongodb<br>


## 3. Konfigurációs állományok létrehozása, vagy másolása
.vscode/extensions.json (majd a VS Code indításakor a felajánlott bővítmények telepítése)
```
{
    "recommendations": [
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode",
        "prisma.prisma"
    ]
}
```
.vscode/launch.json 
```
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Next.js: debug full stack",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/node_modules/next/dist/bin/next",
      "runtimeArgs": ["--inspect"],
      "skipFiles": ["<node_internals>/**"],
    }
  ]
}
```
.vscode/settings.json 
```
{
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.mouseWheelZoom": true,
    "editor.wordWrap": "on",
    "editor.minimap.enabled": false,
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
        "source.fixAll.eslint": "always"
    },
    "eslint.validate": [
        "typescript",
        "react",
        "typescriptreact",
        "javascript",
        "javascriptreact",
    ],
    "files.autoSave": "afterDelay",
    "files.autoSaveDelay": 1000,
    "git.enableSmartCommit": true,
    "git.confirmSync": false,
    "git.pruneOnFetch": true,
    "git.autofetch": true,
    "git.autofetchPeriod": 60,
    "[prisma]": {
        "editor.defaultFormatter": "Prisma.prisma"
    },
    "[jsonc]": {
        "editor.defaultFormatter": "vscode.json-language-features"
    },
    "typescript.preferences.importModuleSpecifier": "non-relative",
    "javascript.preferences.importModuleSpecifier": "non-relative"
}
```
.vscode/tasks.json 
```
{
  // See https://go.microsoft.com/fwlink/?LinkId=733558
  // for the documentation about the tasks.json format
  "version": "2.0.0",
  "tasks": [
    {
      "type": "npm",
      "script": "dev",
      "group": {
        "kind": "build",
        "isDefault": true
      }
    },
    {
      "type": "npm",
      "script": "test",
      "group": {
        "kind": "test",
        "isDefault": true
      }
    }
  ]
}
```
eslint.config.mjs
```
import js from '@eslint/js';
import tseslint from 'typescript-eslint';

export default [
  js.configs.recommended, // Alap JavaScript ajánlott szabályok
  ...tseslint.configs.recommended, // TypeScript ajánlott szabályok

  {
    ignores: ['node_modules/**', 'dist/**', 'build/**', '*.config.js', '*.config.cjs', '*.config.mjs'],
  },

  {
    files: ['**/*.ts'],
    languageOptions: {
      parser: tseslint.parser,
      parserOptions: {
        project: './tsconfig.json',
      },
    },
    rules: {
      // TypeScript best practices
      '@typescript-eslint/explicit-function-return-type': 'off',
      '@typescript-eslint/no-unused-vars': ['warn', { argsIgnorePattern: '^_' }],
      '@typescript-eslint/no-misused-promises': 'error',
      '@typescript-eslint/no-explicit-any': 'off',

      // Kód tisztaság
      'no-console': 'warn',
      'no-var': 'error',
      'prefer-const': 'error',
      eqeqeq: 'error',
    },
  },
];
```

.prettierrc
```
{
  "singleQuote": true,
  "semi": true,
  "trailingComma": "all",
  "tabWidth": 2,
  "printWidth": 120
}
```

## 4. data és lib mappák másolása, adatbázis fileok kibontása


## 4. A .env állományban a connection string beállítása (local MongoDB szerverhez)
```
DATABASE_URL="mongodb://localhost:27017/sampleDB"
```
ha Mongo Atlas-t használsz:
```
DATABASE_URL="mongodb+srv://user_name:user_password@sandbox.abcdef.mongodb.net/sampleDB?retryWrites=true&w=majority&authSource=admin"
```

## 5. Prisma Schema létrehozása (minta Film modell) ./prisma/schema.prisma
```
generator client {
  provider = "prisma-client-js"
  output   = "../app/generated/prisma"
}

datasource db {
  provider = "mongodb"
  url      = env("DATABASE_URL")
}

model Film {
  // id        String  @id @default(auto()) @map("_id") @db.ObjectId
  id        Int @id @map("_id")
  title     String   @unique
  content   String
  createdAt DateTime? @default(now())
  updatedAt DateTime? @updatedAt

  @@map("filmek")
}
```
majd:
> npx prisma db push<br>
> npx prisma generate<br>


## 6. A Prisma Schema-t a feltöltött (forrás) adatbázistáblákból is létrehozhatjuk
> npx prisma db pull --force<br>

majd a Prisma Schema finomítása után:

> npx prisma db push<br>
> npx prisma generate<br>

## 7. Minden változás után szinkronizálás az adatbázissal és a Prisma Client frissítése:
> npx prisma db push<br>
> npx prisma generate<br>

### 8. Local MongoDB indítása replica set-el

> data/startMongoDB.bat

### 1.8 Replica set inicializálása (csak egyszer kell az adatbázis tároló ("c:\data\db) létrehozásakor, megőrzésre kerül a beállítás)
#### 1. mongo shell indítása
> mongosh
#### 2. replica set inicializálása
> rs.initiate()

### 1.9 Több Prisma Client futtatásának megakadályozása
```
// lib/prisma.ts
// https://www.prisma.io/docs/orm/more/help-and-troubleshooting/nextjs-help

import { PrismaClient } from "@/app/generated/prisma-client";

const globalForPrisma = global as unknown as { prisma: PrismaClient };

const prisma = globalForPrisma.prisma || new PrismaClient();

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;

export default prisma;
```
### 1.10 API útvonalak létrehozása (GET all, POST), hibakezelés nélkül
```
// @/app/posts/route.ts

import { NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";

// GET all posts
export async function GET() {
  const posts = await prisma.post.findMany();
  return NextResponse.json(posts);
}

// CREATE a post with POST method
export async function POST(req: Request) {
  const { title, content } = await req.json();
  const newPost = await prisma.post.create({
    data: { title, content },
  });
  return NextResponse.json(newPost);
}
```
### 1.11 Dinamikus ([id]) API útvonalak létrehozása (GET one, UPDATE, DELETE, PATCH), hibakezelés nélkül
```
// @/app/api/posts/[id]/route.ts

import { NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";

// GET one post
export async function GET(
  req: Request,
  { params }: { params: { id: string } }
) {
  const post = await prisma.post.findUnique({
    where: { id: params.id },
  });
  return NextResponse.json(post);
}

// UPDATE one post
export async function PATCH(
  req: Request,
  { params }: { params: { id: string } }
) {
  const { title, content } = await req.json();
  const updated = await prisma.post.update({
    where: { id: params.id },
    data: { title, content },
  });
  return NextResponse.json(updated);
}

// DELETE one post
export async function DELETE(
  req: Request,
  { params }: { params: { id: string } }
) {
  await prisma.post.delete({
    where: { id: params.id },
  });
  return NextResponse.json({ message: "Deleted successfully" });
}
```
### 1.14 Egyszerű hibakezelés minta
```
// CREATE "ingatlan" with POST method
export async function POST(req: Request) {
  try {
    const data = await req.json();
    const newIngatlan = await prisma.ingatlanok.create({
      data: data,
    });
    return NextResponse.json(newIngatlan, { status: 201 });
  } catch (error: Error | unknown) {
    if (error instanceof Error) {
      if (error.message.includes('Unique constraint failed')) {
        return NextResponse.json({ message: 'Az azonosító már létezik!' }, { status: 409 });
      }
      return NextResponse.json({ message: error.message }, { status: 500 });
    } else {
      return NextResponse.json({ message: 'Unknown error' }, { status: 500 });
    }
  }
}
```
### 1.13 Prisma Studio indítása
> npx prisma studio