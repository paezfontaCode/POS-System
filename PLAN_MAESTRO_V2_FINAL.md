# Plan Maestro Unificado

> \*\*Ultima actualizacion:\*\* 11-Jul-2026
> \*\*Proposito:\*\* Documento unico de referencia para todo el desarrollo del sistema POS + Taller.
> \*\*Enfoque:\*\* MVP primero, expansión según necesidades reales. Vibe Coding con IA.

\---

## 1\. Resumen Ejecutivo

|Campo|Detalle|
|-|-|
|**Proyecto**|Sistema POS + Gestión de Taller |
|**Usuario**|Dueño del negocio (no programador profesional, usa IA para desarrollar)|
|**Objetivo**|Sistema funcional que cubra ventas, inventario y reparaciones|
|**Enfoque**|MVP primero, luego expansión según necesidades reales|
|**Plazo MVP**|Fases 1-5 (sin offline) — hitos por funcionalidad, no por dias calendario|

### Perfiles del Sistema

* **Administrador (Dueño/Esposa):** Control total, finanzas, inventario, reportes, gestión de abonos/créditos.
* **Vendedor:** Punto de venta (POS) rápido, búsqueda por IMEI, registro de clientes.
* **Tecnico:** Gestion de ordenes de reparación, actualización de estados, registro de repuestos usados.

> \*\*Regla de Oro:\*\* No soy un programador profesional, uso "Vibe Coding" (programación asistida por IA). El código debe ser extremadamente limpio, modular, bien comentado y facil de mantener.

\---

## 2\. Stack Tecnologico (Definitivo)

> \*\*NO uses otras tecnologias a menos que se justifique explicitamente.\*\*

### Frontend (UI/UX)

|Tecnologia|Version / Nota|
|-|-|
|React + Vite|React 18, Vite 5|
|TypeScript|**Relajado para Vibe Coding** (ver seccion 2.4)|
|Tailwind CSS|Utilidades CSS|
|shadcn/ui|Componentes minimalistas (OBLIGATORIO)|
|Zustand|Estado global simple|
|TanStack Query|Datos del servidor (cache, revalidacion)|
|React Router DOM|v6|

~~Offline/PWA (Dexie.js + Service Workers)~~ — **ELIMINADO del MVP.** Ver seccion 2.5.

### Backend (API)

|Tecnologia|Version / Nota|
|-|-|
|Node.js + Express.js|Runtime + Framework|
|TypeScript|**Relajado para Vibe Coding** (ver seccion 2.4)|
|Prisma ORM|Manejo de base de datos (OBLIGATORIO)|
|Zod|Validacion de inputs en API y formularios|
|bcrypt|Hash de contraseñas|
|JWT|Autenticacion (tokens)|

### Base de Datos e Infraestructura

|Tecnologia|Nota|
|-|-|
|PostgreSQL 16|Motor de base de datos|
|Docker Compose|**SOLO para levantar PostgreSQL** (ver seccion 2.3)|
|Node.js nativo|Backend corre con `npm run dev` (hot-reload directo)|
|Vite nativo|Frontend corre con `npm run dev` (HMR directo)|

\---

## 3\. Ajustes Criticos Aplicados (vs. Version Anterior)

Estos ajustes se aplicaron para reducir la friccion durante el desarrollo con IA y evitar bloqueos innecesarios.

### 3A. Offline/PWA — Eliminado del MVP

**Problema original:** Implementar Dexie.js + Service Workers para sincronizacion offline es extremadamente complejo de depurar con IA. Si se vende el ultimo celular con IMEI X offline, y al mismo tiempo se busca online, se genera un conflicto de sincronizacion (doble venta). El negocio es local — no justifica esta complejidad en el MVP.

**Solucion aplicada:** Eliminar completamente el modo offline del MVP. La red LAN estable (router WiFi de calidad, PC del mostrador y del taller conectadas por cable o WiFi 5GHz) es suficiente. Si se cae internet, el sistema local sigue funcionando porque todo corre en la misma red. El modo PWA se evaluara post-produccion, cuando el sistema ya tenga meses de estabilidad.

> \*\*Nota futura:\*\* Cuando se decida implementar offline, la arquitectura deberia ser "conflict-free" — es mas facil bloquear operaciones que requieran consistencia estricta (ventas de celulares por IMEI) que intentar resolver conflictos de sincronizacion.

### 3B. Docker — Solo para PostgreSQL

**Problema original:** Meter el Backend y Frontend en Docker complica el debugging. No se ven los errores en consola nativa, se pierde el hot-reload de React, y los logs de error quedan encapsulados en contenedores — critico cuando la IA genera un bug.

**Solucion aplicada:** Docker Compose se usa UNICAMENTE para levantar PostgreSQL. Node.js (Backend) y Vite (Frontend) corren de forma nativa en la maquina con `npm run dev`. Esto da acceso directo a los logs de error, hot-reload instantaneo, y depuracion natural.

**docker-compose.yml definitivo (solo DB):**

```yaml
# docker-compose.yml — SOLO PostgreSQL
version: "3.8"
services:
  db:
    image: postgres:16-alpine
    container\_name: SamanDigital-db
    restart: unless-stopped
    ports:
      - "5432:5432"
    environment:
      POSTGRES\_USER: saman
      POSTGRES\_PASSWORD: saman123
      POSTGRES\_DB: saman\_digital
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

**Flujo de trabajo diario:**

```bash
# 1. Levantar la base de datos (una sola vez)
docker compose up -d

# 2. Backend (en terminal separada)
cd backend \&\& npm run dev
# → http://localhost:3001

# 3. Frontend (en otra terminal)
cd frontend \&\& npm run dev
# → http://localhost:5173
```

### 3C. TypeScript — Relajado para Vibe Coding

**Problema original:** TypeScript estricto es genial en equipos profesionales, pero con Vibe Coding la IA a veces se pierde en bucles de errores de tipos (`Type 'string' is not assignable to type...`), gastando tokens y tiempo sin avanzar en la funcionalidad.

**Solucion aplicada:** Mantener TypeScript, pero con estas reglas pragmáticas:

1. **Zod como fuente de verdad:** Validar TODO en los bordes (API endpoints y formularios). El tipado de Zod se infiere automaticamente con `z.infer<>`. Esto evita tener que tipificar dos veces.
2. **`any` como escape hatch:** Si la IA se atasca en un error de tipos complejo, se permite usar `any` o `unknown` temporalmente para desbloquear. Se agrega un comentario `// TODO: tipar correctamente` y se avanza.
3. **Prompt de desbloqueo:** Cuando TS bloquee, decirle a la IA: *"Ignora el tipado estricto por un momento, usa `any` o `unknown` para desbloquear esto, y luego lo refinamos."*
4. **tsconfig.json relajado:**

```jsonc
// tsconfig.json (backend y frontend)
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": false,                    // NO activar strict completo
    "noImplicitAny": false,             // Permite any implicito
    "strictNullChecks": true,           // SI mantener este — evita muchos bugs en runtime
    "noUncheckedIndexedAccess": false,  // Desactivado — genera mucho ruido con IA
    "esModuleInterop": true,
    "skipLibCheck": true,               // Evita errores de tipos en node\_modules
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "react-jsx",
    "outDir": "./dist",
    "rootDir": "./src",
    "paths": {
      "@/\*": \["./src/\*"]
    }
  },
  "include": \["src"],
  "exclude": \["node\_modules", "dist"]
}
```

> \*\*Por que `strictNullChecks: true`?\*\* Es la unica regla estricta que se mantiene porque previene el error mas comun en Vibe Coding: intentar acceder a una propiedad de algo que es `null` o `undefined` en runtime. La IA suele asumir que los datos de la API siempre llegan completos, y este chequeo atrapa esos errores en tiempo de compilacion.

\---

## 4\. Reglas de Diseno y UI/UX

El usuario final prefiere un diseño **limpio, minimalista y no sobrecargado**.

1. **Cero Ruido Visual:** Evita sombras excesivas, bordes innecesarios y colores saturados. Usa espacios en blanco generosos.
2. **Paleta de Colores:** Fondo claro (`slate-50` / `white`) o modo oscuro profesional (`slate-900`). Colores de acento sobrios (azul indigo o esmeralda).
3. **Tipografia:** Sans-serif moderna (Inter o similar), jerarquia clara de tamanos.
4. **Layout:** Sidebar lateral oscuro o minimalista para navegacion principal. Header superior limpio con busqueda global y perfil de usuario.
5. **Tablas:** Faciles de leer, con paginacion y filtros claros.
6. **Feedback:** Usa "Toasts" (notificaciones flotantes) para confirmar acciones exitosas o mostrar errores.

\---

## 5\. Estructura de Carpetas del Proyecto

```
Pos-sistem/
├── docker-compose.yml          # Solo PostgreSQL
├── .env                        # Variables de entorno (DB URL, JWT secret)
├── .gitignore
│
├── backend/
│   ├── package.json
│   ├── tsconfig.json
│   ├── prisma/
│   │   ├── schema.prisma       # Modelo de datos (ver seccion 6)
│   │   └── migrations/
│   ├── src/
│   │   ├── index.ts            # Entry point del servidor
│   │   ├── config/
│   │   │   └── db.ts           # Prisma Client singleton
│   │   ├── middleware/
│   │   │   ├── auth.ts         # Verificacion de JWT
│   │   │   └── errorHandler.ts # Manejo centralizado de errores
│   │   ├── routes/
│   │   │   ├── auth.ts
│   │   │   ├── products.ts
│   │   │   ├── customers.ts
│   │   │   ├── sales.ts
│   │   │   ├── repairs.ts
│   │   │   └── payments.ts
│   │   ├── validators/
│   │   │   └── \*.ts            # Schemas Zod para cada endpoint
│   │   └── utils/
│   │       └── helpers.ts      # Funciones utilitarias
│   └── tests/                  # (Opcional, post-MVP)
│
└── frontend/
    ├── package.json
    ├── tsconfig.json
    ├── vite.config.ts
    ├── tailwind.config.ts
    ├── components.json         # Configuracion de shadcn/ui
    ├── public/
    └── src/
        ├── main.tsx
        ├── App.tsx
        ├── index.css
        ├── api/
        │   └── \*.ts            # Llamadas al backend (fetch/axios)
        ├── components/
        │   ├── ui/             # Componentes shadcn/ui
        │   ├── layout/
        │   │   ├── Sidebar.tsx
        │   │   └── Header.tsx
        │   └── shared/         # Componentes reutilizables
        ├── pages/
        │   ├── Login.tsx
        │   ├── Dashboard.tsx
        │   ├── Inventory.tsx
        │   ├── POS.tsx
        │   ├── Repairs.tsx
        │   └── Customers.tsx
        ├── stores/
        │   └── \*.ts            # Zustand stores
        ├── hooks/
        │   └── \*.ts            # Custom hooks (TanStack Query)
        ├── lib/
        │   └── utils.ts        # Utilidades (cn, formatters)
        └── types/
            └── \*.ts            # Tipos TypeScript (inferidos de Zod)
```

\---

## 6\. Schema de Base de Datos (Prisma) — Revisado y Mejorado

> \*\*Cambios vs. version anterior:\*\* Se agregaron indices para consultas frecuentes, se agrego el modelo `CashRegister` para la Fase 4 (Caja Diaria), se agrego `@@index` en claves foraneas para rendimiento, y se incluye `isActive` en User para desactivar sin borrar.

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE\_URL")
}

// ══════════════════════════════════════
// ENUMS
// ══════════════════════════════════════

enum Role {
  ADMIN
  SELLER
  TECH
}

enum ProductType {
  GENERIC   // Accesorios, repuestos (stock general)
  UNIQUE    // Celulares (con IMEI unico)
}

enum ProductStatus {
  AVAILABLE
  SOLD
  IN\_REPAIR
}

enum RepairStatus {
  PENDING
  DIAGNOSING
  WAITING\_PARTS
  IN\_PROGRESS
  READY
  DELIVERED
  CANCELLED
}

enum PaymentMethod {
  CASH
  TRANSFER
  CARD
  MIXED
  CREDIT
}

enum SaleStatus {
  PAID
  PENDING
  PARTIAL
  CANCELLED
}

enum CashRegisterStatus {
  OPEN
  CLOSED
}

// ══════════════════════════════════════
// MODELOS
// ══════════════════════════════════════

model User {
  id           String   @id @default(cuid())
  name         String
  email        String   @unique
  passwordHash String
  role         Role     @default(ADMIN)
  isActive     Boolean  @default(true)   // Permite desactivar sin borrar
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt

  sales   Sale\[]
  repairs RepairTicket\[] @relation("AssignedTechnician")
  cashRegisters CashRegister\[] 
  @@index(\[email])                        // Login lookup rapido
  @@index(\[role])
}

model Customer {
  id        String   @id @default(cuid())
  name      String
  phone     String
  email     String?
  address   String?
  notes     String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  isActive  Boolean  @default(true)
  sales    Sale\[]
  repairs  RepairTicket\[]
  payments Payment\[]

  @@index(\[phone])                       // Busqueda mas frecuente en POS
  @@index(\[name])                        // Busqueda por nombre
}

model Product {
  id          String         @id @default(cuid())
  type        ProductType
  name        String
  category    String
  brand       String?
  priceCost   Float
  priceSale   Float
  stock       Int            @default(0)  // Solo para GENERIC
  minStock    Int            @default(5)

  // Solo UNIQUE (celulares)
  imei        String?        @unique
  model       String?
  status      ProductStatus? @default(AVAILABLE)
  isActive  Boolean  @default(true)	
  createdAt   DateTime       @default(now())
  updatedAt   DateTime       @updatedAt

  saleItems   SaleItem\[]
  repairParts RepairPart\[]

  @@index(\[type])                        // Filtrar genericos vs celulares
  @@index(\[category])                    // Filtrar por categoria
  @@index(\[status])                      // Ver disponibles vs vendidos
}

model Sale {
  id            String        @id @default(cuid())
  invoiceNumber String        @unique
  customerId    String
  userId        String
  subtotal      Float
  tax           Float         @default(0)
  discount      Float         @default(0)
  total         Float
  paymentMethod PaymentMethod
  status        SaleStatus    @default(PAID)
  amountPaid    Float         @default(0)
  notes         String?
  createdAt     DateTime      @default(now())
  updatedAt     DateTime      @updatedAt

  customer Customer @relation(fields: \[customerId], references: \[id])
  user     User     @relation(fields: \[userId], references: \[id])
  items    SaleItem\[]
  payments Payment\[]

  @@index(\[customerId])
  @@index(\[userId])
  @@index(\[createdAt])                   // Reportes por fecha
  @@index(\[status])                      // Ventas pendientes/parciales
}

model SaleItem {
  id        String  @id @default(cuid())
  saleId    String
  productId String
  quantity  Int
  unitPrice Float
  subtotal  Float
  imei      String? // Congela el IMEI al momento de la venta

  sale    Sale    @relation(fields: \[saleId], references: \[id], onDelete: Cascade)
  product Product @relation(fields: \[productId], references: \[id])

  @@index(\[saleId])
  @@index(\[productId])
  @@index(\[imei])                        // Buscar que celular se vendio
}

model Payment {
  id            String        @id @default(cuid())
  saleId        String
  customerId    String
  amount        Float
  paymentMethod PaymentMethod
  notes         String?
  date          DateTime      @default(now())

  sale     Sale     @relation(fields: \[saleId], references: \[id])
  customer Customer @relation(fields: \[customerId], references: \[id])

  @@index(\[saleId])
  @@index(\[customerId])
  @@index(\[date])                         // Reportes de pagos por fecha
}

model RepairTicket {
  id              String       @id @default(cuid())
  ticketNumber    String       @unique
  customerId      String
  technicianId    String?
  deviceType      String
  brand           String
  model           String
  imei            String?
  passwordPattern String?
  reportedIssue   String
  diagnosticNotes String?
  status          RepairStatus @default(PENDING)
  laborCost       Float        @default(0)
  totalCost       Float        @default(0)
  createdAt       DateTime     @default(now())
  updatedAt       DateTime     @updatedAt
  deliveredAt     DateTime?

  customer   Customer       @relation(fields: \[customerId], references: \[id])
  technician User?          @relation("AssignedTechnician", fields: \[technicianId], references: \[id])
  parts      RepairPart\[]

  @@index(\[customerId])
  @@index(\[technicianId])
  @@index(\[status])                       // Filtrar por estado (Kanban)
  @@index(\[imei])                         // Buscar reparaciones por IMEI
  @@index(\[createdAt])                    // Orden cronologico
}

model RepairPart {
  id        String       @id @default(cuid())
  repairId  String
  productId String
  quantity  Int
  unitPrice Float
  subtotal  Float

  repair  RepairTicket @relation(fields: \[repairId], references: \[id], onDelete: Cascade)
  product Product      @relation(fields: \[productId], references: \[id])

  @@index(\[repairId])
  @@index(\[productId])
}

model CashRegister {
  id          String             @id @default(cuid())
  userId      String             // Quien abre/cierra la caja
  status      CashRegisterStatus @default(OPEN)
  openingBalance Float
  closingBalance Float?
  openedAt    DateTime           @default(now())
  closedAt    DateTime?

  user        User               @relation(fields: \[userId], references: \[id])

  @@index(\[userId])
  @@index(\[status])
  @@index(\[openedAt])                     // Caja del dia actual
}
```

### Mejoras al Schema vs. Version Anterior

|Mejora|Por que|
|-|-|
|**`@@index` en campos de busqueda** (`phone`, `name`, `imei`, `createdAt`, `status`)|Sin indices, PostgreSQL hace escaneo completo de tabla. Con cientos de productos y ventas, las consultas se vuelven lentas. Estos indices hacen que las busquedas en el POS sean instantaneas.|
|**Modelo `CashRegister` nuevo**|El PLAN\_MAESTRO original mencionaba "Caja Diaria" en Fase 4 pero no tenia modelo. Este modelo registra quien abrio, saldo inicial, saldo final y cuando cerro. Es la base para el arqueo de caja.|
|**`isActive` en User**|Permite desactivar un usuario sin borrarlo (conserva el historial de ventas y reparaciones que hizo).|
|**`imei` en SaleItem como snapshot**|Congela el IMEI al momento de la venta. Si el producto cambia, el registro de la venta queda intacto.|
|**`CANCELLED` en RepairStatus**|Permite cancelar reparaciones que el cliente no retira. El status original no lo tenia.|

\---

## 7\. Plan de Desarrollo (Fases)

> \*\*NO intentes construir todo a la vez.\*\* Sigue este orden estricto.

### Fase 1: Cimientos (DB + Backend Base)

**Objetivo:** Base de datos corriendo, backend con los endpoints criticos del MVP.

**Tareas:**

* \[ ] Crear estructura de carpetas (ver seccion 5)
* \[ ] Crear `docker-compose.yml` (solo PostgreSQL, ver seccion 3B)
* \[ ] Crear `.env` con `DATABASE\_URL` y `JWT\_SECRET`
* \[ ] Inicializar backend: `npm init -y`, instalar dependencias
* \[ ] Crear `schema.prisma` con todos los modelos (seccion 6)
* \[ ] Ejecutar migracion inicial: `npx prisma migrate dev --name init`
* \[ ] Crear seed script para el primer usuario Admin
* \[ ] Configurar Express + TypeScript + Prisma Client + CORS
* \[ ] Crear middleware de manejo centralizado de errores
* \[ ] Crear endpoints del MVP:

  * `POST /api/auth/login` — Login con email/password, devuelve JWT
  * `GET /api/products` — Listar productos (con filtro por tipo y busqueda)
  * `POST /api/products` — Crear producto (generico o celular con IMEI)
  * `PUT /api/products/:id` — Editar producto
  * `GET /api/products?search=imei123` — Busqueda por IMEI o nombre
  * `GET /api/customers` — Listar clientes
  * `POST /api/customers` — Crear cliente
  * `POST /api/sales` — Crear venta con items
  * `GET /api/repairs` — Listar reparaciones
  * `POST /api/repairs` — Crear reparacion
  * `PATCH /api/repairs/:id/status` — Cambiar estado de reparacion
* \[ ] Validar TODOS los endpoints con Zod (ver seccion 8.1)

**Entregable:** API REST funcionando, probable con Postman o curl. Base de datos con datos de prueba.

**Metodo sugerido:** Pide los endpoints de a uno a la IA. Ejemplo: *"Crea el endpoint POST /api/products que reciba los datos del producto, valide con Zod (diferenciando GENERIC vs UNIQUE), verifique que el IMEI no exista si es celular, y devuelva el producto creado."*

\---

### Fase 2: Frontend — Layout + Autenticacion + Inventario

**Objetivo:** Interfaz base funcional con login, navegacion y gestion de inventario.

**Tareas:**

* \[ ] Inicializar frontend: `npm create vite@latest frontend --template react-ts`
* \[ ] Instalar y configurar Tailwind CSS + shadcn/ui
* \[ ] Crear Layout principal:

  * Sidebar con navegacion (Dashboard, Inventario, POS, Taller, Clientes)
  * Header con busqueda global y perfil de usuario
* \[ ] Implementar autenticacion:

  * Pantalla de Login
  * Guardar token en localStorage
  * Proteccion de rutas (redirect a login si no hay token)
  * Logout
* \[ ] Crear capa de API (funciones `fetch` con token JWT)
* \[ ] Modulo de Inventario:

  * Tabla de productos con filtros (tipo, categoria, estado)
  * Formulario modal para agregar/editar producto
  * Campo IMEI visible solo si tipo = UNIQUE
  * Busqueda por nombre o IMEI en tiempo real
  * Indicador visual de stock bajo (< minStock)

**Entregable:** Sistema con login funcional, navegacion entre paginas, y CRUD completo de inventario.

**Metodo sugerido:** Empieza por el Layout y Login. Cuando funcione, pasa a Inventario. No toques POS ni Taller todavia.

\---

### Fase 3: Frontend — POS + Taller + Clientes

**Objetivo:** Modulos operativos principales listos para usar en la tienda.

**Tareas — POS:**

* \[ ] Busqueda rapida de productos (por nombre o IMEI)
* \[ ] Carrito de compras (agregar, quitar, cambiar cantidad)
* \[ ] Seleccion de cliente (buscar/crear desde el POS)
* \[ ] Metodos de pago (Efectivo, Transferencia, Tarjeta, Mixto)
* \[ ] Calculo automatico de totales (subtotal + descuento)
* \[ ] Boton "Procesar Venta" que:

  * Cree la venta en la API
  * Actualice el stock de productos genericos
  * Cambie el estado del celular a SOLD (si aplica)
  * Muestre confirmacion con numero de factura
* \[ ] Impresion de ticket (80mm) — opcional, post-MVP si es complejo

**Tareas — Taller:**

* \[ ] Lista de reparaciones con filtro por estado
* \[ ] Formulario de ingreso de equipo:

  * Seleccionar/crear cliente
  * Tipo de dispositivo, marca, modelo, IMEI
  * Patron de contrasena del dispositivo
  * Descripcion de la falla
* \[ ] Cambio de estado (Pendiente → Diagnosticando → En Proceso → Listo → Entregado)
* \[ ] Registro de repuestos usados (seleccionar del inventario, descontar stock)
* \[ ] Calculo automatico: total = mano de obra + repuestos

**Tareas — Clientes:**

* \[ ] Tabla de clientes con busqueda
* \[ ] Formulario para agregar/editar
* \[ ] Vista de historial del cliente (sus compras y reparaciones)

**Entregable:** Sistema funcional que puedes usar en tu tienda para vender y recibir equipos.

**Metodo sugerido:** Haz POS primero — es el modulo mas critico. Luego Taller. Luego Clientes (que es el mas simple).

\---

### Fase 4: Estabilizacion + Despliegue LAN

**Objetivo:** Sistema estable corriendo en tu red local, listo para produccion.

**Tareas:**

* \[ ] Probar todos los flujos completos:

  * Crear producto generico → venderlo → verificar stock actualizado
  * Crear celular con IMEI → venderlo → verificar que no se pueda vender de nuevo
  * Crear cliente → venderle → verificar historial
  * Registrar reparacion → cambiar estados → marcar entregada
  * Buscar por IMEI en el POS
* \[ ] Manejar errores comunes:

  * Que pasa si el IMEI ya existe al crear un celular?
  * Que pasa si el stock es 0 y se intenta vender?
  * Que pasa si la red se cae momentaneamente? (debe mostrar error claro, no crashear)
* \[ ] Configurar red LAN:

  * Asegurar que las PCs del mostrador y taller pueden comunicarse
  * El backend debe escuchar en `0.0.0.0` (no `localhost`) para ser accesible desde otros equipos
  * Configurar el firewall de Windows si aplica
* \[ ] (Opcional) Acceder desde otros dispositivos en la misma red via `http://<IP-DEL-SERVIDOR>:5173`
* \[ ] Capacitacion basica (tu y tu esposa): login, vender, registrar reparacion

**Entregable:** Sistema en uso real en Saman Digital (nombre tienda).

**Metodo sugerido:** Esta fase es 80% pruebas y 20% codigo. No pidas funcionalidades nuevas — solo corrige lo que falle. Pide a la IA: *"Al ejecutar \[accion], obtengo este error: \[pegar error]. Analiza la causa raiz y dame la solucion."*

\---

### Fase 5: Expansion — Finanzas y Caja (Post-MVP)

**Objetivo:** Agregar funcionalidades financieras cuando el MVP ya sea estable (1-2 meses de uso real).

**Tareas:**

* \[ ] Modulo de "Cuentas por Cobrar":

  * Ventas a credito (status = PARTIAL)
  * Interfaz para registrar abonos parciales
  * Historial de pagos por cliente
  * Alerta de clientes con saldo pendiente
* \[ ] Caja Diaria:

  * Apertura de caja (saldo inicial)
  * Registro automatico de ingresos (ventas del dia)
  * Registro manual de egresos (gastos)
  * Cierre de caja con arqueo (saldo esperado vs real)
* \[ ] (Opcional) Roles multiples:

  * Crear usuarios VENDEDOR y TECNICO
  * Permisos: solo ADMIN puede borrar productos, solo TECH puede cambiar estado de reparacion
* \[ ] (Opcional) Reportes basicos:

  * Ventas del dia/semana/mes
  * Productos mas vendidos
  * Reparaciones por estado

**Entregable:** Sistema con control financiero basico.

\---

### Fase 6: Mejoras Futuras (Sin Timeline)

Estas funcionalidades se evaluaran cuando el sistema tenga meses de estabilidad en produccion:

* **Modo Offline/PWA:** Solo si la red LAN resulta inestable. Requiere arquitectura de sincronizacion conflict-free.
* **Dashboard con graficos:** Usar una libreria como Recharts o Chart.js.
* **Exportacion PDF/CSV:** Para reportes de ventas y reparaciones.
* **Notificaciones:** Alertas de stock bajo, reparaciones listas para entregar, pagos atrasados.

\---

## 8\. Metodos y Sugerencias para el Desarrollo con IA

### 8.1. Zod como Escudo Principal (Validacion en los Bordes)

La filosofia es simple: **Zod valida en los bordes, TypeScript infiere de Zod, y dentro del codigo usas `any` si hace falta.**

**Ejemplo — Backend (endpoint):**

```typescript
// backend/src/validators/product.ts
import { z } from "zod";

export const createProductSchema = z.discriminatedUnion("type", \[
  z.object({
    type: z.literal("GENERIC"),
    name: z.string().min(1, "Nombre requerido"),
    category: z.string().min(1),
    brand: z.string().optional(),
    priceCost: z.number().positive("El costo debe ser mayor a 0"),
    priceSale: z.number().positive("El precio debe ser mayor a 0"),
    stock: z.number().int().min(0).default(0),
    minStock: z.number().int().min(0).default(5),
  }),
  z.object({
    type: z.literal("UNIQUE"),
    name: z.string().min(1, "Nombre requerido"),
    category: z.string().min(1),
    brand: z.string().min(1, "La marca es requerida para celulares"),
    model: z.string().min(1, "El modelo es requerido para celulares"),
    imei: z.string().min(10, "IMEI muy corto").max(15, "IMEI muy largo"),
    priceCost: z.number().positive(),
    priceSale: z.number().positive(),
  }),
]);

// El tipo se infiere automaticamente — no hay que escribirlo a mano
export type CreateProductInput = z.infer<typeof createProductSchema>;
```

```typescript
// backend/src/routes/products.ts
import { createProductSchema } from "../validators/product";

router.post("/", async (req, res) => {
  // 1. Validar con Zod
  const result = createProductSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(400).json({ errors: result.error.flatten() });
  }

  // 2. result.data ya esta tipado correctamente
  const product = await prisma.product.create({ data: result.data });
  res.json(product);
});
```

**Ejemplo — Frontend (formulario):**

```typescript
// frontend/src/validators/product.ts
// Mismo schema de Zod — se puede compartir o duplicar
import { z } from "zod";

export const productFormSchema = z.object({
  name: z.string().min(1, "Nombre requerido"),
  category: z.string().min(1, "Categoria requerida"),
  priceCost: z.coerce.number().positive("Costo invalido"),
  priceSale: z.coerce.number().positive("Precio invalido"),
});

// Uso con react-hook-form + zodResolver
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";

const form = useForm({
  resolver: zodResolver(productFormSchema),
});
```

### 8.2. Como Pedirle Codigo a la IA (Patrones que Funcionan)

**MAL:**

* *"Hazme el POS"*
* *"Crea todo el backend"*
* *"El sistema no funciona"*

**BIEN:**

* *"Basado en el schema de Prisma, crea el endpoint POST /api/sales que reciba los datos de la venta, valide con Zod, cree la venta y los SaleItems en una transaccion, actualice el stock de los productos genericos, cambie el status del celular a SOLD si aplica, y devuelva la venta creada con su numero de factura."*
* *"En el componente POS.tsx, agrega un campo de busqueda que filtre productos por nombre o IMEI usando TanStack Query. Muestra los resultados en una lista y al hacer clic agregue el producto al carrito (Zustand store)."*

**Regla:** Cada pedido debe tener: **que crear + basado en que + que validar + que devolver**.

### 8.3. Cuando Algo Falla (Protocolo de Debug)

1. **Copia el error EXACTO** de la consola del navegador (F12 → Console) o de la terminal del backend.
2. **Pegalo a la IA** con este formato:

> \*"Al ejecutar \[accion concreta], obtengo este error: \[pegar error completo]. Analiza la causa raiz y dame la solucion."\*

3. **No adivines.** Si la IA te da una solucion que no entiendes, preguntale: *"Explicame que hace esta linea antes de que la implemente."*
4. **Si es un error de TypeScript**, aplica la regla de la seccion 3C: *"Ignora el tipado estricto, usa any para desbloquear, y luego lo refinamos."*

### 8.4. Estrategia de Desarrollo por Iteraciones

```
Iteracion tipica para un endpoint:

1. Pides el endpoint a la IA (prompt especifico)
2. La IA genera el codigo
3. Lo guardas en el archivo correspondiente
4. Ejecutas y pruebas con Postman/curl
5. Si falla → le pasas el error a la IA → corrige
6. Cuando funcione → pasas al siguiente endpoint
7. NO avances al siguiente hasta que el actual funcione
```

### 8.5. Comandos de Referencia Rapida

```bash
# === BASE DE DATOS ===
docker compose up -d                          # Levantar PostgreSQL
docker compose down                           # Detener PostgreSQL
npx prisma migrate dev --name init            # Crear migracion inicial
npx prisma migrate dev --name add\_field\_x     # Nueva migracion
npx prisma studio                             # Visor visual de la DB (http://localhost:5555)
npx prisma generate                           # Regenerar Prisma Client
npx prisma db seed                            # Ejecutar seed script

# === BACKEND ===
cd backend
npm run dev                                   # Iniciar con hot-reload
npm run build                                 # Compilar TypeScript
npm start                                     # Ejecutar compilado

# === FRONTEND ===
cd frontend
npm run dev                                   # Iniciar Vite (HMR)
npm run build                                 # Build de produccion
npx shadcn@latest add button                  # Agregar componente shadcn
npx shadcn@latest add dialog                  # Agregar componente shadcn
npx shadcn@latest add table                   # Agregar componente shadcn
```

### 8.6. Variables de Entorno (.env)

```bash
# .env (en la raiz del proyecto)

# Base de datos
DATABASE\_URL="postgresql://saman:saman123@localhost:5432/saman\_digital"

# JWT
JWT\_SECRET="cambia-esto-por-una-clave-secreta-larga-y-aleatoria"
JWT\_EXPIRES\_IN="8h"

# Servidor
PORT=3001
NODE\_ENV="development"

# Frontend URL (para CORS)
FRONTEND\_URL="http://localhost:5173"



\# Frontend (Vite requiere el prefijo VITE\_ para exponer variables al navegador)

VITE\_API\_URL="http://localhost:3001/api"
```

> \*\*Importante:\*\* El archivo `.env` NUNCA se sube a Git. Agrega `.env` a tu `.gitignore`.

### 8.7. Checklist antes de pasar a la siguiente fase

* \[ ] Todos los endpoints de la fase actual responden correctamente
* \[ ] Los errores se manejan con mensajes claros (no errores crpticos de servidor)
* \[ ] La validacion con Zod rechaza datos incorrectos
* \[ ] El frontend muestra loading states mientras carga datos
* \[ ] Los formularios muestran errores de validacion al usuario
* \[ ] Probaste el flujo completo de punta a punta (no solo partes aisladas)

\---

## 9\. Reglas de Oro para la IA

1. **Paso a Paso:** Si te pido una funcionalidad grande, dividela en pasos logicos. No generes 500 lineas de codigo de una sola vez.
2. **Manejo de Errores:** Si el codigo que generaste falla, no adivines. Pedeme el error exacto de la consola o terminal para analizar la causa raiz.
3. **IMEI como String:** Los IMEIs siempre deben tratarse como `String` en la base de datos y en el frontend, nunca como `Number` (para evitar perdida de precision por ceros a la izquierda o limites de JavaScript).
4. **Seguridad Basica:** Las contrasenas deben hashearse (bcrypt). Los endpoints deben verificar el rol del usuario (ej. solo ADMIN puede borrar productos, solo TECH puede cambiar estado de reparacion).
5. **Comentarios Explicativos:** Comenta el "por que" de la logica de negocio compleja, no el "que" hace el codigo.
6. **No sobrecargues:** Si una pantalla tiene demasiados botones o campos, detente y propon una version mas limpia usando modales o pestanas (Tabs).
7. **Transacciones en Ventas:** Al crear una venta, usar `prisma.$transaction()` para crear la venta + los items + actualizar stock + cambiar status del celular. Si algo falla, todo se revierte.
8. **Numeros de Factura:** Generar con formato secuencial: `FAC-0001`, `FAC-0002`, etc. Buscar el ultimo numero existente y sumar 1.
9. **Soft Deletes:** Nunca borres registros de la base de datos. Agrega un campo `isActive` o `deletedAt` y filtra las consultas.

