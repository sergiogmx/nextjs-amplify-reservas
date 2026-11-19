# Landing Page con Formulario de Reservas

Landing page sencilla con sistema de reservas integrada, desarrollada con **Next.js (App Router)**, desplegada en **AWS Amplify**, con persistencia de datos en **Supabase** y notificaciones por correo mediante **Amazon SES**.

## Stack Tecnológico

* **Framework:** Next.js (App Router + Server Actions)
* **Estilos:** Tailwind CSS
* **Validación:** Zod
* **Base de Datos:** Supabase (PostgreSQL)
* **Correos Transaccionales:** Amazon SES (`@aws-sdk/client-ses`)
* **Hosting & despliegue:** AWS Amplify Hosting
* **Calidad:** ESLint, Prettier, TypeScript (`tsc --noEmit`), Vitest y GitHub Actions

## Estructura del Proyecto

```text
├── src/
│   ├── app/
│   │   ├── actions/
│   │   │   └── reservas.ts      # Server Action (validación, Supabase y SES)
│   │   ├── layout.tsx
│   │   └── page.tsx             # Landing page principal y formulario
│   ├── components/
│   │   └── BookingForm.tsx      # Componente cliente del formulario
│   └── lib/
│       ├── ses.ts               # Configuración cliente de Amazon SES
│       └── supabase.ts          # Configuración cliente de Supabase
├── .github/workflows/ci.yml     # CI: lint, formato, tipos, tests y build
├── .editorconfig
├── .nvmrc                       # Versión de Node (22)
├── .prettierrc.json
├── .env.example                 # Plantilla de variables de entorno
└── package.json
```

## Requisitos Previos

1. **Node.js 22** (ver `.nvmrc`; con nvm basta `nvm use`). Next.js requiere 20.9 como mínimo.
2. Cuenta en **Supabase** con un proyecto creado.
3. Cuenta en **AWS** con acceso a **Amazon SES** (dominio o correo remitente verificado) y **AWS Amplify**.

## Configuración de la Base de Datos (Supabase)

Ejecuta el siguiente script SQL en el **SQL Editor** de tu proyecto en Supabase para crear la tabla de reservas:

```sql
create table reservas (
  id uuid default gen_random_uuid() primary key,
  nombre text not null,
  email text not null,
  telefono text,
  fecha date not null,
  hora text not null,
  notas text,
  estado text default 'pendiente',
  created_at timestamp with time zone default timezone('utc'::text, now()) not null
);

-- Habilitar Row Level Security (RLS)
alter table reservas enable row level security;
```

## Instalación y Desarrollo Local

1. **Clona el repositorio e instala las dependencias:**

```bash
git clone <URL_DE_TU_REPOSITORIO>
cd nextjs-amplify-reservas
nvm use        # Node 22, según .nvmrc
npm install
```

`package.json` ya incluye Supabase, el SDK de SES, Zod, Prettier y Vitest, además de los scripts que usa el CI.

2. **Configura las variables de entorno:**

Copia `.env.example` a `.env.local` (`cp .env.example .env.local`) y llena los valores:

```env
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://tu-proyecto.supabase.co
SUPABASE_SERVICE_ROLE_KEY=tu_service_role_key

# AWS SES (Nota: Amplify no permite variables con prefijo AWS_, usa APP_AWS_)
APP_AWS_REGION=us-east-1
APP_AWS_ACCESS_KEY_ID=tu_access_key_id
APP_AWS_SECRET_ACCESS_KEY=tu_secret_access_key
SES_FROM_EMAIL=reservas@tudominio.com
ADMIN_EMAIL=tu-correo@tudominio.com
```

3. **Inicia el servidor de desarrollo:**

```bash
npm run dev
```

Abre [http://localhost:3000](http://localhost:3000) en tu navegador.

## Calidad de Código y CI

Antes de abrir un PR, corre lo mismo que corre el CI:

```bash
npm run lint && npm run format:check && npm run typecheck && npm test
```

| Script | Qué hace |
|---|---|
| `npm run lint` | ESLint |
| `npm run format` | Formatea con Prettier (`format:check` solo verifica) |
| `npm run typecheck` | Genera los tipos de rutas (`next typegen`) y verifica tipos con `tsc --noEmit` |
| `npm test` | Tests con Vitest |

Los tests cubren la parte con lógica real: el schema de Zod y la Server Action de reservas, con Supabase y SES simulados (`vi.mock`). No hace falta probar la UI de la landing.

El workflow [.github/workflows/ci.yml](.github/workflows/ci.yml) corre en cada PR y en cada push a `main`: lint, formato, tipos, tests y `next build`. El build usa variables de entorno ficticias, así que el CI no necesita secretos. Amplify sigue encargándose del despliegue; el CI solo evita que algo roto llegue a `main`.

## Despliegue en AWS Amplify

1. Sube este repositorio a **GitHub**, **GitLab** o **Bitbucket**.
2. Ingresa a la consola de **AWS Amplify** y selecciona **Create new app**.
3. Conecta tu repositorio y selecciona la rama principal (`main`).
4. En **Build settings**, fija Node 22 agregando `nvm use 22` en la fase `preBuild` del `amplify.yml` (o elige Node 22 en la imagen de build).
5. En la sección **Environment variables**, agrega todas las variables definidas en tu `.env.local` (usando el prefijo `APP_AWS_` para tus credenciales IAM).
6. Haz clic en **Save and deploy**. Amplify detectará Next.js automáticamente, compilará el proyecto y habilitará HTTPS.

## Notas de Producción

* **Amazon SES Sandbox:** Por defecto, las cuentas nuevas de AWS SES solo pueden enviar correos a direcciones verificadas. Solicita el **Production Access** en la consola de SES para enviar confirmaciones a cualquier cliente.
* **Seguridad:** Nunca expongas `SUPABASE_SERVICE_ROLE_KEY` ni las llaves de AWS en componentes cliente (`"use client"`). Úsalas exclusivamente dentro de Server Actions o Route Handlers.
