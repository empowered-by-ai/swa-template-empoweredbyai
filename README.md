# swa-template-empoweredbyai

Plantilla reutilizable para publicar sitios estáticos (HTML/CSS/JS plano, o un
proyecto Node.js que compile a estático) en **S3 + CloudFront**, sin escribir
infraestructura ni workflows nuevos cada vez.

Basada en el patrón de deploy de [`empowered-by-ai-web`](https://github.com/empowered-by-ai/empowered-by-ai-web):
`aws s3 sync` + invalidación de CloudFront **buscando la distribución solo por
el dominio (alias)**, nunca por un Distribution ID hardcodeado.

## Cómo usar esta plantilla

1. Crea un repo nuevo a partir de esta plantilla (o cópialo).
2. Copia tu proyecto dentro de `src/`:
   - **HTML/CSS/JS simple**: copia tus archivos tal cual (`index.html`,
     `styles.css`, `js/`, imágenes, etc.) directamente en `src/`.
   - **Proyecto Node.js** (Next.js con export estático, Vite, Astro,
     cualquier cosa que compile a una carpeta estática): copia el proyecto
     completo (incluido su `package.json`) dentro de `src/`.
3. Configura las **Variables** y **Secrets** del Environment de GitHub
   (`dev`, `test`, `main`) que vayas a usar — ver tabla abajo.
4. Push a `main` → deploy automático a `dev`. Para promover a `test` o
   `main`, ejecuta el workflow "Deploy Static Site to S3 + CloudFront" a mano
   (Actions → Run workflow) eligiendo el environment.

El workflow detecta solo si hay un `src/package.json`:

- **Si existe** → instala dependencias (`npm ci`), corre el build
  (`BUILD_COMMAND`, por defecto `npm run build`) y despliega la carpeta de
  salida (`BUILD_OUTPUT_DIR` si la defines; si no, autodetecta `dist/`,
  `build/`, `out/` o `public/`, en ese orden).
- **Si no existe** → despliega `src/` tal cual (sitio estático plano).

## Variables por Environment (Settings → Environments → `<env>` → Variables)

| Variable            | Obligatoria | Descripción                                                                 |
| ------------------- | ----------- | ---------------------------------------------------------------------------- |
| `AWS_REGION`        | Sí          | Región de AWS, ej. `us-east-1`.                                              |
| `S3_BUCKET`         | Sí          | Nombre del bucket S3 destino.                                                |
| `S3_PREFIX`         | No          | Path/prefijo dentro del bucket, ej. `mi-sitio`. Vacío = raíz del bucket.     |
| `SITE_DOMAIN`       | Sí          | Dominio (alias) del sitio, ej. `mi-sitio.empoweredbyai.org`. Se usa **solo esto** para encontrar la distribución de CloudFront a invalidar — no hace falta el Distribution ID. |
| `BUILD_COMMAND`     | No          | Comando de build si tu proyecto es Node.js. Por defecto `npm run build`.    |
| `BUILD_OUTPUT_DIR`  | No          | Carpeta de salida del build, relativa a `src/`. Si no se define, se autodetecta (`dist`, `build`, `out`, `public`). |
| `NODE_VERSION`      | No          | Versión de Node para el build. Por defecto `20`.                            |

## Secrets por Environment (Settings → Environments → `<env>` → Secrets)

| Secret                  | Obligatorio | Descripción                          |
| ------------------------ | ----------- | ------------------------------------- |
| `AWS_ACCESS_KEY`         | Sí          | Access Key ID con permisos de S3/CloudFront. |
| `AWS_ACCESS_KEY_SECRET`  | Sí          | Secret Access Key correspondiente.    |

## Requisitos de infraestructura (una sola vez, fuera de este repo)

Este template **no crea** el bucket S3 ni la distribución de CloudFront, solo
las usa. Antes de tu primer deploy necesitas tener ya provisionado, por
environment:

- Un bucket S3 (puede ser compartido entre varios sitios usando distintos
  `S3_PREFIX`).
- Una distribución de CloudFront cuyo **alias (CNAME)** sea exactamente el
  valor de `SITE_DOMAIN`, apuntando al bucket (o a ese prefijo) como origen.
- Un usuario/rol de IAM con permisos `s3:PutObject`, `s3:DeleteObject`,
  `s3:ListBucket` sobre el bucket, y `cloudfront:ListDistributions` +
  `cloudfront:CreateInvalidation` sobre la distribución.

## Desarrollo local

```bash
npx --yes serve src        # sirve src/ tal cual (sitio estático)
# o, si src/ es un proyecto Node.js:
cd src && npm install && npm run dev
```

## Estructura

```
.
├── .github/workflows/deploy.yml   # workflow de deploy (S3 + CloudFront)
├── src/                           # ← aquí copias tu proyecto
└── README.md
```
