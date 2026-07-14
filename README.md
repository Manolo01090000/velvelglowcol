[package.json](https://github.com/user-attachments/files/29991333/package.json)# velvelglowcol
tienda virtual que se expecialisa en vender experiensia 

# Base de datos (SQLite funciona sin configuración; para producción usa PostgreSQL)
DATABASE_URL="file:./dev.db"
# Para producción con PostgreSQL, comenta la línea de arriba y usa:
# DATABASE_URL="postgresql://usuario:password@host:5432/velvet_glowcol"

# NextAuth
NEXTAUTH_SECRET="genera-un-secreto-con-openssl-rand-base64-32"
NEXTAUTH_URL="http://localhost:3000"

# Login con Google (opcional — crea credenciales en https://console.cloud.google.com)
GOOGLE_CLIENT_ID=""
GOOGLE_CLIENT_SECRET=""

# Stripe (crea tu cuenta gratis en https://dashboard.stripe.com/register, usa las llaves de PRUEBA)
STRIPE_SECRET_KEY="sk_test_xxxxxxxxxxxx"
STRIPE_PUBLISHABLE_KEY="pk_test_xxxxxxxxxxxx"
STRIPE_WEBHOOK_SECRET="whsec_xxxxxxxxxxxx"

# Cloudinary (opcional, para subir imágenes reales de productos desde el panel admin)
# Crea una cuenta gratis en https://cloudinary.com
CLOUDINARY_CLOUD_NAME=""
CLOUDINARY_API_KEY=""
CLOUDINARY_API_SECRET=""

# Correo (opcional, para enviar confirmación de pedidos — usa https://resend.com, tiene plan gratis)
RESEND_API_KEY=""
EMAIL_FROM="pedidos@velvetglowcol.com"

# Chat e[postcss.config.js](https://github.com/user-attachments/files/29991340/postcss.config.js)n vivo (opcional)
NEXT_PUBLIC_TAWKTO_ID=""
[Uploading [Uploading[package-lock.json](https://github.com/user-attachments/files/29991335/package-lock.json) package.json…]()next.config.mjs…]()[README.md](https://github.com/user-attachments/files/29991346/README.md)

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite" // Cambia a "postgresql" en producción
  url      = env("DATABASE_URL")
}

enum Role {
  ADMIN
  EMPLOYEE
  CUSTOMER
}

enum OrderStatus {
  NUEVO
  PAGADO
  ENVIADO
  ENTREGADO
  CANCELADO
}

enum PaymentMethod {
  STRIPE
  PSE
  TRANSFERENCIA
}

enum PaymentStatus {
  PENDIENTE
  EXITOSO
  RECHAZADO
}

model User {
  id            String    @id @default(cuid())
  name          String?
  email         String    @unique
  emailVerified DateTime?
  password      String? // null si inició sesión con Google
  image         String?
  role          Role      @default(CUSTOMER)
  createdAt     DateTime  @default(now())

  accounts  Account[]
  sessions  Session[]
  orders    Order[]
  addresses Address[]
  favorites Favorite[]
  reviews   Review[]
}

model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String?
  access_token      String?
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String?
  session_state     String?
  user              User    @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model Address {
  id         String  @id @default(cuid())
  userId     String
  fullName   String
  documento  String
  telefono   String
  direccion  String
  ciudad     String
  departamento String
  pais       String  @default("Colombia")
  esPrincipal Boolean @default(false)
  user       User    @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model Category {
  id       String    @id @default(cuid())
  name     String    @unique
  slug     String    @unique
  products Product[]
}

model Product {
  id            String   @id @default(cuid())
  name          String
  slug          String   @unique
  description   String
  price         Int // en pesos COP, sin decimales
  oldPrice      Int?
  categoryId    String
  images        ProductImage[]
  variants      ProductVariant[]
  isNew         Boolean  @default(false)
  isFeatured    Boolean  @default(false)
  isBestseller  Boolean  @default(false)
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt

  category   Category    @relation(fields: [categoryId], references: [id])
  reviews    Review[]
  favorites  Favorite[]
  orderItems OrderItem[]

  @@index([categoryId])
}

model ProductImage {
  id        String  @id @default(cuid())
  url       String
  productId String
  order     Int     @default(0)
  product   Product @relation(fields: [productId], references: [id], onDelete: Cascade)
}

// Una variante = combinación talla + color, con su propio stock
model ProductVariant {
  id        String @id @default(cuid())
  productId String
  size      String
  color     String
  colorHex  String
  stock     Int    @default(0)
  product   Product @relation(fields: [productId], references: [id], onDelete: Cascade)
  orderItems OrderItem[]

  @@unique([productId, size, color])
}

model Favorite {
  id        String  @id @default(cuid())
  userId    String
  productId String
  user      User    @relation(fields: [userId], references: [id], onDelete: Cascade)
  product   Product @relation(fields: [productId], references: [id], onDelete: Cascade)

  @@unique([userId, productId])
}

model Review {
  id        String   @id @default(cuid())
  productId String
  userId    String
  rating    Int
  comment   String
  createdAt DateTime @default(now())
  product   Product  @relation(fields: [productId], references: [id], onDelete: Cascade)
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model Order {
  id            String        @id @default(cuid())
  orderNumber   String        @unique
  userId        String?
  user          User?         @relation(fields: [userId], references: [id])

  // snapshot de datos del comprador (por si no tiene cuenta)
  nombre        String
  apellidos     String
  documento     String
  telefono      String
  email         String
  direccion     String
  ciudad        String
  departamento  String
  pais          String        @default("Colombia")

  subtotal      Int
  envio         Int
  descuento     Int           @default(0)
  total         Int

  paymentMethod PaymentMethod
  paymentStatus PaymentStatus @default(PENDIENTE)
  status        OrderStatus   @default(NUEVO)
  stripeSessionId String?
  numeroGuia    String?

  createdAt     DateTime      @default(now())
  updatedAt     DateTime      @updatedAt

  items         OrderItem[]
}

model OrderItem {
  id        String  @id @default(cuid())
  orderId   String
  productId String
  variantId String?
  name      String // snapshot del nombre por si el producto cambia luego
  price     Int    // snapshot del precio
  quantity  Int
  size      String?
  color     String?

  order   Order           @relation(fields: [orderId], references: [id], onDelete: Cascade)
  product Product         @relation(fields: [productId], references: [id])
  variant ProductVariant? @relation(fields: [variantId], references: [id])
}

model NewsletterSubscriber {
  id        String   @id @default(cuid())
  email     String   @unique
  createdAt DateTime @default(now())
}

import { PrismaClient } from "@prisma/client";
import bcrypt from "bcryptjs";

const prisma = new PrismaClient();

const CATEGORIES = ["Vestidos", "Blusas", "Jeans", "Pantalones", "Faldas", "Conjuntos", "Chaquetas", "Accesorios"];
const SIZES = ["XS", "S", "M", "L", "XL"];
const COLORS = [
  { name: "Beige", hex: "#D8C6AE" },
  { name: "Negro", hex: "#1C1917" },
  { name: "Blanco", hex: "#F7F4EE" },
  { name: "Rosa palo", hex: "#E7C9C4" },
  { name: "Dorado", hex: "#B9945B" },
  { name: "Verde oliva", hex: "#7C7C52" }
];
const NAMES: Record<string, string[]> = {
  Vestidos: ["Vestido Resolana", "Vestido Alba Midi", "Vestido Sena Satinado", "Vestido Cala Lino"],
  Blusas: ["Blusa Aurora", "Blusa Marfil Seda", "Blusa Nube Popelín", "Blusa Ocre Anudada"],
  Jeans: ["Jean Recto Camila", "Jean Mom Valentina", "Jean Wide Leg Sol", "Jean Tiro Alto Río"],
  Pantalones: ["Pantalón Palazzo Ines", "Pantalón Sastre Nora", "Pantalón Culotte Bruma"],
  Faldas: ["Falda Midi Ámbar", "Falda Plisada Luna", "Falda Lápiz Terra"],
  Conjuntos: ["Conjunto Duna Lino", "Conjunto Costa Dos Piezas", "Conjunto Noche Satín"],
  Chaquetas: ["Chaqueta Blazer Sena", "Chaqueta Denim Cala", "Chaqueta Trench Alba"],
  Accesorios: ["Cinturón Dorado Oria", "Pañoleta Seda Marfil", "Bolso Estructurado Nora"]
};

function slugify(text: string) {
  return text
    .toLowerCase()
    .normalize("NFD")
    .replace(/[\u0300-\u036f]/g, "")
    .replace(/[^a-z0-9]+/g, "-")
    .replace(/(^-|-$)/g, "");
}

async function main() {
  console.log("Sembrando base de datos...");

  // Usuario administrador por defecto
  const adminPassword = await bcrypt.hash("VelvetAdmin2026!", 10);
  await prisma.user.upsert({
    where: { email: "admin@velvetglowcol.com" },
    update: {},
    create: {
      name: "Administrador Velvet",
      email: "admin@velvetglowcol.com",
      password: adminPassword,
      role: "ADMIN"
    }
  });

  // Categorías
  const categoryMap: Record<string, string> = {};
  for (const name of CATEGORIES) {
    const cat = await prisma.category.upsert({
      where: { slug: slugify(name) },
      update: {},
      create: { name, slug: slugify(name) }
    });
    categoryMap[name] = cat.id;
  }

  // Productos
  let count = 0;
  for (const cat of CATEGORIES) {
    for (let i = 0; i < NAMES[cat].length; i++) {
      const name = NAMES[cat][i];
      const price = 89000 + Math.floor((Math.random() * 260000) / 1000) * 1000;
      const hasDiscount = Math.random() > 0.6;
      const discountPct = hasDiscount ? [10, 15, 20, 25][Math.floor(Math.random() * 4)] : 0;
      const oldPrice = hasDiscount ? Math.round(price / (1 - discountPct / 100) / 1000) * 1000 : null;
      const slug = slugify(name) + "-" + (++count);
      const seed = slugify(`velvet-${cat}-${i}`);

      const product = await prisma.product.create({
        data: {
          name,
          slug,
          description: `Pieza confeccionada en tejido de alta calidad, pensada para acompañar tanto el día como la noche. Corte favorecedor, terminaciones cuidadas y una paleta atemporal. Parte de la colección ${cat} de Velvet GlowCol.`,
          price,
          oldPrice,
          categoryId: categoryMap[cat],
          isNew: i === 0,
          isFeatured: Math.random() > 0.5,
          isBestseller: Math.random() > 0.7,
          images: {
            create: [1, 2, 3, 4].map((n) => ({
              url: `https://picsum.photos/seed/${seed}-${n}/700/900`,
              order: n
            }))
          },
          variants: {
            create: SIZES.filter(() => Math.random() > 0.25).flatMap((size) =>
              COLORS.filter(() => Math.random() > 0.5)
                .slice(0, 3)
                .map((c) => ({
                  size,
                  color: c.name,
                  colorHex: c.hex,
                  stock: Math.random() > 0.15 ? Math.floor(Math.random() * 20) + 1 : 0
                }))
            )
          }
        }
      });
      void product;
    }
  }

  console.log(`Listo: ${count} productos creados en ${CATEGORIES.length} categorías.`);
  console.log("Usuario admin: admin@velvetglowcol.com / VelvetAdmin2026!");
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
