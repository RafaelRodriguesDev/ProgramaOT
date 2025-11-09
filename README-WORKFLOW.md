# Guia de Workflow de Release do Cliente (ProgramaOT) para o Canary Launcher

Este documento explica, de ponta a ponta, como automatizar a geração do ZIP do cliente (pasta ProgramaOT) e a publicação/atualização da Release no GitHub. O Canary Launcher usa um link estável para baixar sempre o último ZIP da Release marcada como “latest”.

Resumo do fluxo:
- Cada push na branch main do repositório ProgramaOT dispara um GitHub Action.
- O Action compacta a pasta ProgramaOT em client-to-update.zip.
- Remove o asset client-to-update.zip anterior (se existir) na Release v1.0.0.
- Cria/atualiza a Release v1.0.0 e marca como latest.
- O Canary Launcher baixa pelo link estável: https://github.com/RafaelRodriguesDev/ProgramaOT/releases/latest/download/client-to-update.zip

Requisitos:
- Repositório do cliente: https://github.com/RafaelRodriguesDev/ProgramaOT (público para download sem autenticação).
- Branch principal: main.
- GitHub Actions habilitado e permissões de workflow com “Read and write permissions”.
- Pasta ProgramaOT na raiz do repositório com o conteúdo do cliente (pré-compilado).

Conteúdo do workflow (release.yml):

```yaml
name: Package and Release ProgramaOT
on:
  push:
    branches: [ "main" ]
  workflow_dispatch: {}

permissions:
  contents: write

jobs:
  release:
    runs-on: windows-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Create ZIP of ProgramaOT folder
        shell: pwsh
        run: |
          if (Test-Path client-to-update.zip) { Remove-Item client-to-update.zip -Force }
          Compress-Archive -Path ProgramaOT\* -DestinationPath client-to-update.zip -Force

      - name: Delete existing asset named client-to-update.zip (if any)
        uses: actions/github-script@v7
        with:
          script: |
            const owner = context.repo.owner;
            const repo = context.repo.repo;
            const tag = 'v1.0.0';
            try {
              const release = await github.rest.repos.getReleaseByTag({ owner, repo, tag });
              for (const asset of release.data.assets) {
                if (asset.name === 'client-to-update.zip') {
                  await github.rest.repos.deleteReleaseAsset({ owner, repo, asset_id: asset.id });
                  core.info(`Deleted asset ${asset.name}`);
                }
              }
            } catch (e) {
              core.info(`Release ${tag} not found, will create it.`);
            }

      - name: Create/Update Release v1.0.0 and upload asset
        uses: softprops/action-gh-release@v2
        with:
          files: client-to-update.zip
          name: Client v1.0.0
          tag_name: v1.0.0
          draft: false
          prerelease: false
          make_latest: true
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Passo a passo para configurar (Windows/PowerShell):
1) Abrir terminal na pasta local do repositório ProgramaOT (ex.: j:\Projeto\Ot\ProgramaOT).
2) Criar o diretório de workflows e o arquivo release.yml.
3) Comitar e dar push para main.

Comandos (exatamente o que digitar):

```powershell
$ErrorActionPreference = 'Stop'
Set-Location 'j:\Projeto\Ot\ProgramaOT'

# Garantir diretório do workflow
New-Item -ItemType Directory -Force -Path .github\workflows | Out-Null

# Criar conteúdo do workflow
$workflowContent = @'
name: Package and Release ProgramaOT
on:
  push:
    branches: [ "main" ]
  workflow_dispatch: {}

permissions:
  contents: write

jobs:
  release:
    runs-on: windows-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Create ZIP of ProgramaOT folder
        shell: pwsh
        run: |
          if (Test-Path client-to-update.zip) { Remove-Item client-to-update.zip -Force }
          Compress-Archive -Path ProgramaOT\* -DestinationPath client-to-update.zip -Force

      - name: Delete existing asset named client-to-update.zip (if any)
        uses: actions/github-script@v7
        with:
          script: |
            const owner = context.repo.owner;
            const repo = context.repo.repo;
            const tag = 'v1.0.0';
            try {
              const release = await github.rest.repos.getReleaseByTag({ owner, repo, tag });
              for (const asset of release.data.assets) {
                if (asset.name === 'client-to-update.zip') {
                  await github.rest.repos.deleteReleaseAsset({ owner, repo, asset_id: asset.id });
                  core.info(`Deleted asset ${asset.name}`);
                }
              }
            } catch (e) {
              core.info(`Release ${tag} not found, will create it.`);
            }

      - name: Create/Update Release v1.0.0 and upload asset
        uses: softprops/action-gh-release@v2
        with:
          files: client-to-update.zip
          name: Client v1.0.0
          tag_name: v1.0.0
          draft: false
          prerelease: false
          make_latest: true
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
'@

Set-Content -Path .github\workflows\release.yml -Value $workflowContent -Encoding UTF8

# Commit e push
git add .github\workflows\release.yml
git commit -m "Add workflow to package ProgramaOT and publish/update Release v1.0.0"
git push origin HEAD
```

Como testar o workflow:
- Faça uma alteração simples (ex.: adicionar uma linha ao README) e dê push:

```powershell
Set-Location 'j:\Projeto\Ot\ProgramaOT'
Add-Content -Path README.md -Value "`nTrigger workflow test" 

git add README.md
git commit -m "chore: trigger workflow test run"
git push origin HEAD
```

- Acompanhe a execução em: https://github.com/RafaelRodriguesDev/ProgramaOT/actions
- Ao finalizar, valide o link de download:
  - https://github.com/RafaelRodriguesDev/ProgramaOT/releases/latest/download/client-to-update.zip
- Se aparecer 404, verifique se a Release v1.0.0 existe e se o asset foi enviado:
  - https://github.com/RafaelRodriguesDev/ProgramaOT/releases/tag/v1.0.0

Como configurar o Canary Launcher para usar o “latest”:
- newClientUrl no launcher_config.json deve apontar para:
  - https://github.com/RafaelRodriguesDev/ProgramaOT/releases/latest/download/client-to-update.zip
- Detecção de versão:
  - Opção simples: atualizar clientVersion no JSON remoto a cada release.
  - Opção avançada: usar a GitHub API (releases/latest) para ler tag_name e comparar com a versão local.

Personalizações comuns:
- Trocar a branch de gatilho:
  - No YAML, ajuste on.push.branches para outra branch ou múltiplas.
- Mudar o nome do asset (e do link):
  - Altere "client-to-update.zip" em Compress-Archive e em softprops/action-gh-release.
  - O link ficaria: releases/latest/download/<novo-nome>.zip.
- Zipping seletivo (para acelerar):
  - Em vez de ProgramaOT\*, zipar apenas pastas essenciais (ex.: assets, conf, bin, sounds). Exemplo:

```powershell
Compress-Archive -Path assets, conf, bin, sounds -DestinationPath client-to-update.zip -Force
```

- Excluir pastas pesadas (logs, cache, screenshots, crashdump, minimap, storeimages) do pacote.

Solução de problemas:
- Permissões de workflow:
  - No repositório GitHub, vá em Settings > Actions > General > Workflow permissions e selecione "Read and write permissions".
- Link de download dá 404:
  - Verifique o run do Action, se houve erro no upload ou o asset não foi criado.
  - Confirme que a Release v1.0.0 foi marcada como latest (make_latest: true).
- Erro de asset duplicado:
  - O passo "Delete existing asset..." remove o asset anterior antes do upload.
- Repositório privado:
  - O download sem autenticação pelo launcher não funciona. Para privados, seria necessário token e código no launcher para autenticação (não recomendado para clientes finais).

Boas práticas:
- Padronize o nome do ZIP (client-to-update.zip) para manter o link estável.
- Evite incluir arquivos desnecessários no pacote para reduzir tempo de build/upload e melhorar a experiência do jogador.
- Documente no README do ProgramaOT o que está incluído no pacote e o processo de atualização.

Links úteis:
- Actions do repositório: https://github.com/RafaelRodriguesDev/ProgramaOT/actions
- Release específica v1.0.0: https://github.com/RafaelRodriguesDev/ProgramaOT/releases/tag/v1.0.0
- Download “latest”: https://github.com/RafaelRodriguesDev/ProgramaOT/releases/latest/download/client-to-update.zip

