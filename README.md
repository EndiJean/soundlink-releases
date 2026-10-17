# SoundLink — Releases

Este repositório não contém código-fonte. Ele existe só para hospedar as
**releases** (builds publicados) do SoundLink: o instalador do servidor
Windows (`.exe`) e o app Android (`.apk`), além de servir como fonte de
dados para a checagem automática de atualização embutida nos dois.

O código-fonte do projeto mora em outro lugar (repositórios `PCAudioServer`
e `SoundLink`, locais/privados).

## O que é o SoundLink

SoundLink transforma o celular Android em uma caixa de som sem fio para o
PC. É composto por duas partes que conversam entre si pela rede Wi-Fi:

- **PCAudioServer** (Windows, Python) — roda na bandeja do sistema, captura
  o áudio que o Windows está reproduzindo (via WASAPI loopback) e
  transmite em PCM bruto por TCP (porta `38473`) para o celular. Permite
  ajustar o volume digital da transmissão direto pelo menu da bandeja, e
  avisa quando há uma atualização disponível.
- **SoundLink** (Android, Kotlin) — conecta no servidor, recebe o áudio e
  toca no celular através de um serviço em primeiro plano (continua
  funcionando com a tela bloqueada). Também consegue compartilhar o
  instalador do servidor Windows via um link local, pra facilitar a
  instalação em outro PC sem precisar de internet.

### Checagem de atualização

Tanto o app Android quanto o servidor Windows consultam a API do GitHub
(`/repos/EndiJean/soundlink-releases/releases/latest`) na inicialização:

- Se a versão publicada aqui for mais nova que a instalada, o **app
  Android** mostra um aviso na tela com um botão para baixar e instalar
  (usa o `.apk` anexado à release).
- O **servidor Windows** mostra a opção "Baixar atualização" no menu da
  bandeja, que abre a página da release no navegador (usa o `.exe`
  anexado à release).

A checagem é silenciosa: sem internet ou com o GitHub fora do ar, ela
simplesmente não encontra nada, sem travar o uso normal de nenhum dos dois.

## Como publicar uma nova versão

1. No código-fonte, atualize a versão nos dois lados para o mesmo número:
   - PC: `VERSION` em `pc_audio_server/config.py`
   - Android: `versionName` (e incremente `versionCode`) em
     `app/build.gradle.kts`
2. Gere os dois builds:
   - PC: `pyinstaller SoundLinkServer.spec` → gera
     `dist/SoundLinkServer-<versao>.exe`
   - Android: copie o `.exe` gerado para
     `app/src/main/assets/SoundLinkServer.exe` (sobrescrevendo) e rode
     `./gradlew assembleDebug` → gera
     `app/build/outputs/apk/debug/SoundLink-<versao>-debug.apk`

   O `.exe` precisa estar sempre atualizado dentro do `.apk` porque o app
   Android o usa para o compartilhamento local do instalador.

3. Publique a release aqui no GitHub:
   - [Releases → "Draft a new release"](../../releases/new)
   - **Tag**: `v<versao>` (ex: `v1.2.0`) — crie a tag na hora, com target
     `main`
   - **Title**: `v<versao>`
   - Anexe os dois arquivos gerados no passo 2, **sem renomear**
   - **Release label**: `None` (nunca marcar como "Pre-release" — releases
     marcadas assim não aparecem no endpoint `/releases/latest` que o app e
     o servidor consultam)
   - **Publish release**

Depois de publicada, o app Android e o servidor Windows detectam a nova
versão automaticamente na próxima vez que forem abertos.

### Formato de nomes esperado

A checagem de atualização procura, entre os arquivos anexados à release:

- O primeiro arquivo terminado em `.apk` → usado pelo app Android
- O primeiro arquivo terminado em `.exe` → usado pelo servidor Windows

O nome exato do arquivo não importa para a lógica (só a extensão), mas
manter o padrão `SoundLink-<versao>-debug.apk` /
`SoundLinkServer-<versao>.exe` ajuda a identificar de qual build cada
arquivo veio.
