# velvelglowcol

Tienda virtual especializada en vender experiencias.

## Tecnología

- **Framework**: Next.js
- **Base de datos**: SQLite (desarrollo) / PostgreSQL (producción)
- **ORM**: Prisma
- **Autenticación**: NextAuth.js
- **Pagos**: Stripe
- **Imágenes**: Cloudinary (opcional)

## Configuración

Crea un archivo `.env.local` con las variables de entorno necesarias:

### Base de datos

```
DATABASE_URL="file:./dev.db"
```

Para producción con PostgreSQL, comenta la línea de arriba y usa:

```
DATABASE_URL="postgresql://usuario:password@host:5432/velvet_glowcol"
```

### NextAuth

```
NEXTAUTH_SECRET="genera-un-secreto-con-openssl-rand-base64-32"
NEXTAUTH_URL="http://localhost:3000"
```

### Google Login (opcional)

Crea credenciales en [Google Cloud Console](https://console.cloud.google.com)

```
GOOGLE_CLIENT_ID=""
GOOGLE_CLIENT_SECRET=""
```

### Stripe

Crea tu cuenta gratis en [Stripe Dashboard](https://dashboard.stripe.com/register) y usa las llaves de PRUEBA

```
STRIPE_SECRET_KEY="sk_test_xxxxxxxxxxxx"
STRIPE_PUBLISHABLE_KEY="pk_test_xxxxxxxxxxxx"
STRIPE_WEBHOOK_SECRET="whsec_xxxxxxxxxxxx"
```

### Cloudinary (opcional)

Para subir imágenes reales de productos desde el panel admin. Crea una cuenta gratis en [Cloudinary](https://cloudinary.com)

```
CLOUDINARY_CLOUD_NAME=""
CLOUDINARY_API_KEY=""
CLOUDINARY_API_SECRET=""
```

### Email (opcional)

Para enviar confirmación de pedidos. Usa [Resend](https://resend.com) (tiene plan gratis)

```
RESEND_API_KEY=""
EMAIL_FROM="pedidos@velvetglowcol.com"
```

### Chat en vivo (opcional)

```
NEXT_PUBLIC_TAWKTO_ID=""
```

## Instalación

```bash
npm install
npx prisma migrate dev
npm run dev
```

La aplicación estará disponible en `http://localhost:3000`

## Credenciales de desarrollo

- **Email**: admin@velvetglowcol.com
- **Contraseña**: VelvetAdmin2026!

## Base de datos

La base de datos se pre-carga con:
- 8 categorías de productos
- 32 productos de ejemplo (4 por categoría)
- Variantes de talla y color
- 1 usuario administrador
