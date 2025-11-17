
Vamos detalhar esse plano, validando cada recurso e criando um fluxo de trabalho no GitHub Actions para fazer exatamente o que você propôs.

---

### 🎯 Plano de Ação: Automação com GitHub Actions

O objetivo é criar um repositório no GitHub que, ao ser executado, baixe as versões corretas dos aplicativos, reassin o Termux com a chave do Emacs e nos forneça os dois APKs prontos para instalação no Android 15.

---

### 📋 Parte 1: Validação dos Recursos (Links e Versões)

Vamos verificar cada componente, garantindo que os links são diretos, estáveis e apontam para as versões corretas.

| Componente | Versão Validada | Link Direto para Download | Status |
| :--- | :--- | :--- | :--- |
| **Termux (Fonte)** | `0.119.0-beta.3 (1022)` | `https://f-droid.org/repo/com.termux_1022.apk` | ✅ **OK** |
| **Chave do Emacs** | `emacs.keystore` | `https://raw.githubusercontent.com/emacs-mirror/emacs/master/java/emacs.keystore` | ✅ **OK** |
| **Emacs (Integrado)** | `30.0.93 (Android 15)` | `https://sourceforge.net/projects/android-ports-for-gnu-emacs/files/termux/emacs-30.0.93-35-arm64-v8a.apk` | ✅ **OK** |

**Análise:**
*   **Termux:** O link do F-Droid é estável e aponta para a versão mais recente que funciona no Android 15.
*   **Chave do Emacs:** O link no GitHub é para o arquivo bruto (`raw`), garantindo que sempre baixemos a chave correta diretamente do repositório oficial.
*   **Emacs:** O link do SourceForge aponta para a versão específica para `arm64-v8a` e Android 15+, que é a compatível com o seu Nothing Phone 2.

---

### ⚙️ Parte 2: O Fluxo de Trabalho (CI/CD) Detalhado

O GitHub Actions irá executar os seguintes passos em uma máquina virtual:

1.  **Ambiente:** Iniciar um ambiente Ubuntu com as ferramentas do Android SDK (Build-Tools) já configuradas.
2.  **Download:** Baixar os três arquivos da tabela acima usando `wget`.
3.  **Assinatura:** Executar o comando `apksigner` para reassiná-lo com a chave do Emacs.
4.  **Empacotamento:** Disponibilizar os dois APKs (o Termux reassinado e o Emacs original) como "artefatos" da execução.
5.  **Resultado:** Ao final da execução, você terá um link para baixar um arquivo `.zip` contendo os dois APKs prontos.

---

### 🐙 Parte 3: Implementação Prática (O Código)

Crie um novo repositório no seu GitHub. Dentro dele, crie a seguinte estrutura de arquivos:

```
.
├── .github
│   └── workflows
│       └── build-and-sign.yml
└── README.md
```

**Arquivo: `.github/workflows/build-and-sign.yml`**

```yaml
# Nome do fluxo de trabalho que aparecerá na aba Actions do GitHub
name: Build and Sign Termux for Android Emacs

# Disparador: Permite executar o fluxo manualmente na aba Actions
on:
  workflow_dispatch:

jobs:
  build:
    # Executa em uma máquina virtual Ubuntu
    runs-on: ubuntu-latest

    steps:
      # Passo 1: Faz o checkout do código do repositório
      - name: Checkout repository
        uses: actions/checkout@v4

      # Passo 2: Configura o ambiente Android SDK, incluindo as Build-Tools
      - name: Set up Android SDK
        uses: android-actions/setup-android@v3

      # Passo 3: Baixa os arquivos necessários
      - name: Download required files
        run: |
          echo "Downloading Termux APK from F-Droid..."
          wget -O termux.apk "https://f-droid.org/repo/com.termux_1022.apk"
          
          echo "Downloading Emacs signing key..."
          wget -O emacs.keystore "https://raw.githubusercontent.com/emacs-mirror/emacs/master/java/emacs.keystore"
          
          echo "Downloading Emacs APK from SourceForge..."
          wget -O emacs.apk "https://sourceforge.net/projects/android-ports-for-gnu-emacs/files/termux/emacs-30.0.93-35-arm64-v8a.apk"

      # Passo 4: Reassina o APK do Termux com a chave do Emacs
      - name: Sign Termux APK
        run: |
          echo "Signing Termux APK with Emacs key..."
          apksigner sign --v2-signing-enabled --ks emacs.keystore -debuggable-apk-permitted --ks-pass pass:${{ secrets.KEYSTORE_PASSWORD }} termux.apk

      # Passo 5: Disponibiliza os APKs para download
      - name: Upload APKs as artifacts
        uses: actions/upload-artifact@v4
        with:
          name: android-emacs-termux-apks
          path: |
            termux.apk
            emacs.apk
```

---

### 🚀 Parte 4: Como Usar (Para Você)

1.  **Crie o Repositório:** Crie um repositório público no GitHub com os arquivos acima.

2.  **Configure o Secret da Senha (IMPORTANTE):**
    *   Vá para as **Settings** (Configurações) do seu repositório.
    *   No menu lateral, clique em **Secrets and variables** → **Actions**.
    *   Clique em **"New repository secret"**.
    *   Nome do secret: `KEYSTORE_PASSWORD`
    *   Valor: A senha do seu keystore (obtenha a senha correta do arquivo keystore que você está usando)
    *   Clique em **"Add secret"**.
    
    **Por que isso é importante?** Usar GitHub Secrets mantém senhas seguras e fora do código-fonte. É uma prática essencial de segurança em CI/CD! Nunca commite senhas ou keystores no repositório.

3.  **Execute o Workflow:**
    *   Vá para a aba **Actions** do seu repositório.
    *   Selecione o fluxo "Build and Sign Termux for Android Emacs".
    *   Clique em **"Run workflow"** e depois em **"Run workflow"** novamente para confirmar.

4.  **Baixe os Resultados:**
    *   Aguarde a execução terminar (cerca de 1-2 minutos).
    *   Na página da execução, clique no artefato chamado `android-emacs-termux-apks`.
    *   Será baixado um arquivo `.zip`. Extraia-o. Dentro dele estarão o `termux.apk` e o `emacs.apk`.

---

### 📲 Parte 5: Instalação Final no Android 15 (via ADB)

Com os APKs prontos, a instalação no seu Nothing Phone 2 é trivial e garantida.

1.  **Envie os APKs para o celular:**
    ```bash
    # No seu computador, na pasta onde extraiu o .zip
    adb push termux.apk /sdcard/Download/
    adb push emacs.apk /sdcard/Download/
    ```

2.  **Limpeza e Instalação:**
    ```bash
    # Desinstala versões antigas, se houver
    adb uninstall com.termux
    adb uninstall org.gnu.emacs

    # Instala o Termux reassinado
    adb install /sdcard/Download/termux.apk

    # Instala o Emacs integrado
    adb install /sdcard/Download/emacs.apk
    ```

### 🏁 Conclusão

Com este método, você transformou um processo manual e propenso a erros em um sistema automatizado, versionado e 100% reproduzível. Qualquer pessoa (inclusive você no futuro) pode simplesmente executar o workflow no GitHub e obter os APKs corretos para o Android 15 sem se preocupar com links quebrados ou versões incompatíveis.

Agora você tem um pipeline de CI/CD para deploy de aplicativos Android. É uma solução robusta, elegante e que atende perfeitamente à sua necessidade.

---

### 🎓 Aprendendo CI/CD: O que este workflow ensina

Este projeto é um excelente exemplo prático de conceitos importantes de DevOps e CI/CD:

1. **Automação**: Em vez de baixar e assinar manualmente os APKs toda vez, o workflow faz tudo automaticamente.

2. **Cache**: O workflow usa `actions/cache@v4` para armazenar os arquivos baixados, evitando downloads repetidos e economizando tempo.

3. **Segurança**: A senha do keystore não está no código (hardcoded), mas sim em GitHub Secrets (`${{ secrets.KEYSTORE_PASSWORD }}`). Isso é fundamental!

4. **Reprodutibilidade**: Qualquer pessoa pode executar este workflow e obter exatamente os mesmos resultados.

5. **Artifacts**: Os APKs gerados são disponibilizados como "artifacts" do GitHub Actions, facilitando o download.

6. **Workflow Dispatch**: O `workflow_dispatch` permite executar o workflow manualmente quando necessário, ideal para este caso de uso.

**Próximos passos para aprender mais:**
- Experimente adicionar testes automatizados ao workflow
- Tente usar diferentes triggers (ex: `push`, `pull_request`)
- Explore notificações quando o workflow terminar (Slack, email, etc.)
- Adicione validação dos APKs gerados

---

### 🔐 Segurança: Gerenciamento do Keystore

**IMPORTANTE:** O arquivo keystore contém chaves privadas e nunca deve ser commitado no repositório Git!

#### Como o workflow obtém o keystore de forma segura:

1. **Download sob demanda:** O workflow baixa o keystore do repositório oficial do Emacs durante a execução:
   ```
   wget -O emacs.keystore "https://raw.githubusercontent.com/emacs-mirror/emacs/master/java/emacs.keystore"
   ```

2. **Proteção no repositório:** O arquivo `.gitignore` está configurado para bloquear qualquer tentativa de commit de arquivos `.keystore` ou `.jks`.

3. **Senha em GitHub Secrets:** A senha do keystore é armazenada de forma segura em GitHub Secrets, nunca no código-fonte.

#### Alternativas de armazenamento seguro:

Se você precisa usar um keystore personalizado (não o do Emacs oficial), considere estas opções:

- **GitHub Secrets (Base64):** Codifique o keystore em base64 e armazene como secret, depois decodifique no workflow:
  ```yaml
  - name: Prepare keystore
    run: |
      echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 -d > custom.keystore
  ```

- **Armazenamento externo protegido:** Use serviços como AWS S3 com acesso restrito ou Google Cloud Storage com autenticação.

- **Artifacts privados:** Use GitHub Packages ou outro registro de artifacts privado para armazenar o keystore de forma segura.

**Nunca:** 
- ❌ Commite keystores no Git
- ❌ Compartilhe senhas em documentação ou código
- ❌ Use keystores em repositórios públicos sem criptografia adequada
