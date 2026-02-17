# Aventura 3D — Guia para Gerar o APK

## Visão Geral da Estrutura

```
aventura3d/
├── www/                          ← Código-fonte do jogo (HTML/JS)
│   └── index.html                ← Jogo completo
├── android/                      ← Projeto Android (Capacitor)
│   ├── app/
│   │   ├── build.gradle
│   │   ├── proguard-rules.pro
│   │   └── src/main/
│   │       ├── AndroidManifest.xml
│   │       ├── assets/public/    ← Cópia dos arquivos web (gerada pelo cap sync)
│   │       ├── java/com/aventura3d/
│   │       │   └── MainActivity.java
│   │       └── res/              ← Ícones, splash, temas
│   ├── build.gradle
│   ├── settings.gradle
│   ├── variables.gradle
│   └── gradle.properties
├── capacitor.config.ts           ← Configuração do Capacitor
└── package.json
```

---

## Pré-requisitos

Antes de começar, instale as seguintes ferramentas:

| Ferramenta | Versão mínima | Download |
|---|---|---|
| Node.js | 18+ | https://nodejs.org |
| JDK (Java) | 17 | https://adoptium.net |
| Android Studio | Hedgehog+ | https://developer.android.com/studio |
| Android SDK | API 34 | via Android Studio SDK Manager |

No Android Studio, instale via **SDK Manager**:
- Android SDK Platform 34
- Android SDK Build-Tools 34
- Android SDK Platform-Tools

---

## Passo 1 — Instalar dependências Node

```bash
# Dentro da pasta do projeto
cd aventura3d
npm install
```

---

## Passo 2 — Sincronizar o Capacitor com o Android

Este comando copia os arquivos de `www/` para `android/app/src/main/assets/public/` e atualiza os plugins.

```bash
npx cap sync android
```

---

## Passo 3 — Gerar ícones (recomendado)

Substitua os arquivos de ícone nas pastas abaixo com sua arte (PNG, fundo transparente):

```
android/app/src/main/res/mipmap-mdpi/ic_launcher.png       — 48×48 px
android/app/src/main/res/mipmap-hdpi/ic_launcher.png       — 72×72 px
android/app/src/main/res/mipmap-xhdpi/ic_launcher.png      — 96×96 px
android/app/src/main/res/mipmap-xxhdpi/ic_launcher.png     — 144×144 px
android/app/src/main/res/mipmap-xxxhdpi/ic_launcher.png    — 192×192 px
```

Ferramenta online gratuita para gerar todos os tamanhos: https://easyappicon.com

---

## Passo 4 — Abrir no Android Studio

```bash
npx cap open android
```

O Android Studio abrirá automaticamente com o projeto configurado.

---

## Passo 5a — Gerar APK de Debug (para testes)

No Android Studio, vá em **Build → Build Bundle(s) / APK(s) → Build APK(s)**.

O APK será gerado em:
```
android/app/build/outputs/apk/debug/app-debug.apk
```

Para instalar direto em um dispositivo conectado via USB (com depuração USB ativada):
```bash
# Via linha de comando no Android Studio Terminal
./gradlew assembleDebug
adb install app/build/outputs/apk/debug/app-debug.apk
```

---

## Passo 5b — Gerar APK de Release (para publicar)

### 5b.1 — Criar keystore (apenas uma vez)

```bash
keytool -genkey -v \
  -keystore aventura3d-release.jks \
  -alias aventura3d \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000
```

Guarde o arquivo `.jks` e as senhas em local seguro — sem eles é impossível atualizar o app na loja.

### 5b.2 — Configurar assinatura no build.gradle

Adicione dentro do bloco `android {}` do arquivo `android/app/build.gradle`:

```groovy
signingConfigs {
    release {
        storeFile     file("../../aventura3d-release.jks")
        storePassword "SUA_SENHA_KEYSTORE"
        keyAlias      "aventura3d"
        keyPassword   "SUA_SENHA_KEY"
    }
}
buildTypes {
    release {
        signingConfig signingConfigs.release
        minifyEnabled false
    }
}
```

### 5b.3 — Compilar

```bash
cd android
./gradlew assembleRelease
```

APK final:
```
android/app/build/outputs/apk/release/app-release.apk
```

---

## Publicar na Google Play Store

1. Acesse https://play.google.com/console e crie uma conta de desenvolvedor (taxa única de U$25).
2. Crie um novo aplicativo e preencha os metadados (descrição, screenshots, classificação etária).
3. Para a Play Store moderna, gere um **AAB** (Android App Bundle) em vez de APK:
   ```bash
   ./gradlew bundleRelease
   # Saída: android/app/build/outputs/bundle/release/app-release.aab
   ```
4. Envie o `.aab` pela aba **Produção → Criar nova versão**.

---

## Publicar fora da Play Store (sideload)

Para distribuir o `.apk` diretamente (WhatsApp, link de download etc.), o usuário final precisa:
1. Ir em **Configurações → Segurança → Instalar apps desconhecidos** e habilitar para o navegador/gerenciador de arquivos.
2. Abrir o arquivo `.apk` no dispositivo para instalar.

---

## Solução de Problemas Comuns

**`SDK location not found`** → Abra o Android Studio, vá em *SDK Manager* e copie o caminho do SDK. Crie o arquivo `android/local.properties` com: `sdk.dir=/caminho/para/seu/Android/Sdk`

**`Minimum supported Gradle version is X`** → Atualize o Android Studio ou edite `gradle-wrapper.properties` com uma versão compatível.

**WebGL não funciona no dispositivo** → Certifique-se que o Android 5.0+ e que o WebView do sistema está atualizado (via Play Store, buscar "Android System WebView").

**Tela em branco no app** → Execute `npx cap sync android` novamente para garantir que os assets foram copiados corretamente.
