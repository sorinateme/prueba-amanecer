# Amanecer Salud Renal — App nativa (Capacitor)

Este proyecto envuelve tu app web (`www/index.html`) en un contenedor nativo con
[Capacitor](https://capacitorjs.com), generando un proyecto **Android** (Android Studio)
y un proyecto **iOS** (Xcode) listos para compilar.

- App ID: `org.amanecer.saludrenal`
- Nombre: **Amanecer Salud Renal**

## Estructura
```
amanecer-capacitor/
├── www/index.html        ← tu app (copia del HTML original)
├── android/               ← proyecto nativo Android (Android Studio)
├── ios/                   ← proyecto nativo iOS (Xcode)
├── capacitor.config.ts
└── package.json
```

## 1. Instalar dependencias
En tu computadora, con Node.js instalado:
```bash
cd amanecer-capacitor
npm install
```

## 2. Compilar para Android
Requisitos: [Android Studio](https://developer.android.com/studio) instalado.
```bash
npx cap open android
```
Esto abre el proyecto en Android Studio. Desde ahí:
- **Run ▶** para probarlo en un emulador o celular conectado.
- **Build > Generate Signed Bundle/APK** para generar el `.apk` o `.aab` para subir a Google Play.

## 3. Compilar para iPhone (iOS)
Requisitos: una **Mac** con **Xcode** instalado (Apple no permite compilar apps de iOS en Windows/Linux).
```bash
npx cap open ios
```
Esto abre el proyecto en Xcode. Desde ahí:
- Selecciona tu equipo de desarrollador (**Signing & Capabilities**).
- **Run ▶** para probar en el simulador o en tu iPhone.
- **Product > Archive** para subirla a App Store Connect (TestFlight / App Store).

> Si no tienes Mac, puedes usar un servicio de compilación en la nube para iOS
> (por ejemplo, Ionic Appflow o un Mac rentado por hora) usando esta misma carpeta `ios/`.

## 4. Cada vez que edites `www/index.html`
Después de modificar la app, vuelve a copiar los cambios a los proyectos nativos:
```bash
npx cap sync
```

## 5. Generar el .aab automáticamente con GitHub Actions (sin instalar nada)
Este proyecto incluye `.github/workflows/build-aab.yml`, que compila el
**Android App Bundle (.aab)** en la nube cada vez que subes cambios.

**Pasos:**
1. Crea un repositorio en GitHub y sube esta carpeta completa (incluyendo `.github/`).
   ```bash
   git init
   git add .
   git commit -m "Amanecer Salud Renal - proyecto Capacitor"
   git branch -M main
   git remote add origin https://github.com/TU_USUARIO/TU_REPO.git
   git push -u origin main
   ```
2. En GitHub, ve a la pestaña **Actions** de tu repositorio: el workflow
   "Build Android App Bundle (.aab)" se ejecuta solo. Si no arranca, puedes
   lanzarlo manualmente con el botón **Run workflow**.
3. Cuando termine (unos minutos), entra al run y descarga el archivo
   `amanecer-salud-renal-aab` en la sección **Artifacts** — ahí está tu `.aab`.

**Importante — esto genera un AAB *sin firmar* salvo que configures un keystore:**
Google Play exige que el AAB esté firmado con tu propia clave (un archivo
`.jks`). Para que el workflow lo firme automáticamente:

1. Genera un keystore (una sola vez, en tu computadora, con Java instalado):
   ```bash
   keytool -genkeypair -v -keystore release-key.jks -alias amanecer \
     -keyalg RSA -keysize 2048 -validity 10000
   ```
   Guarda ese archivo `release-key.jks` en un lugar seguro — si lo pierdes,
   nunca más podrás actualizar la app publicada.
2. Conviértelo a Base64:
   ```bash
   base64 -i release-key.jks -o release-key.b64.txt
   ```
3. En tu repo de GitHub: **Settings → Secrets and variables → Actions → New
   repository secret**, y crea estos 4 secrets:
   - `KEYSTORE_BASE64` → el contenido completo de `release-key.b64.txt`
   - `KEYSTORE_PASSWORD` → la contraseña que pusiste al generar el keystore
   - `KEY_ALIAS` → `amanecer` (o el alias que hayas usado)
   - `KEY_PASSWORD` → la contraseña de la clave (puede ser la misma que la del keystore)
4. Vuelve a correr el workflow (push nuevo o "Run workflow"). El `.aab`
   descargado ya vendrá firmado, listo para subir a la Google Play Console.

Sin estos secrets, el workflow igual funciona y genera un `.aab` sin firmar
(útil para probar que todo compila, pero Google Play no lo aceptará hasta
que lo firmes).

## Notas importantes
- **Ícono y splash screen**: por ahora la app usa los íconos genéricos de Capacitor.
  Te recomiendo generar los definitivos (con el logo de Amanecer) con
  `npx @capacitor/assets generate` (requiere un `icon.png` de 1024×1024 y un
  `splash.png` en la carpeta `assets/`), o reemplazando manualmente los archivos en
  `android/app/src/main/res/` y `ios/App/App/Assets.xcassets/`.
- **Internet requerido**: la app carga Tailwind CSS, Chart.js y el script de
  Google Identity Services desde internet (CDN). El dispositivo necesita conexión
  para que cargue correctamente. Si quieres que funcione sin internet, puedo
  descargar esas librerías y empaquetarlas localmente dentro de `www/`.
- **Inicio de sesión con Google**: sigue sin configurar (falta el `GOOGLE_CLIENT_ID`
  real en el HTML). Además, el SDK web de Google Identity Services no siempre
  funciona bien dentro de un WebView nativo — si lo vas a usar en la app empacada,
  lo ideal es cambiar a un plugin nativo (`@capacitor-community/google-signin` o
  `@capacitor/google-auth`) en vez del script de navegador.
- **Datos guardados en `localStorage`**: seguirán funcionando igual dentro del
  WebView nativo (persisten en el dispositivo).
- **Nombre del paquete**: si más adelante quieres cambiar `org.amanecer.saludrenal`
  por otro identificador, debe hacerse antes de publicar en las tiendas (después
  ya no se puede cambiar sin perder la app publicada).
