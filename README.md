# ProgramaOT

Cliente de Tibia para OTServer (Open Tibia), com binários e bibliotecas do Qt 6 incluídos. Este projeto contém um cliente pronto para uso, acompanhado de recursos, minimapa, traduções e arquivos auxiliares.

## Visão geral

- Binários do cliente e dependências estão em `ProgramaOT/bin/` (inclui `client.exe`, DLLs do Qt, recursos `.rcc`, traduções `.qm`, etc.).
- Arquivos de dados do cliente de Tibia (`Tibia.dat` e `Tibia.spr`) estão presentes tanto na raiz quanto em `bin/`.
- Recursos e catálogos: `assets/`, `sounds/`, `storeimages/` e `assets.json`.
- Dados locais e cache:
  - `cache/` (ex.: boostedcreature.json, compendium.json)
  - `characterdata/` (dados por personagem/ID)
  - `minimap/` (tiles do minimapa em PNG e marcadores em `minimapmarkers.bin`)
  - `conf/` (configurações do cliente)
- Licenças de terceiros em `3rdpartylicences/`.

## Como executar

1. Abra a pasta `ProgramaOT/bin/`.
2. Execute `client.exe`.
3. Certifique-se de que os arquivos `Tibia.dat` e `Tibia.spr` estejam presentes (já fornecidos). O cliente utilizará os recursos e configurações distribuídos junto ao projeto.

Observações:
- Este pacote foi montado para Windows e já inclui a maioria das dependências do Qt necessárias para execução.
- Logs do cliente podem estar em `ProgramaOT/log/client.log`.
- O minimapa e dados de personagem são persistidos localmente nas pastas correspondentes.

## Estrutura do projeto (resumo)

- `bin/`: executável (`client.exe`), DLLs do Qt6 e recursos necessários para rodar o cliente.
- `assets/`, `sounds/`, `storeimages/`: catálogos e mídias usados pelo cliente.
- `minimap/`: imagens do minimapa e marcadores.
- `characterdata/`: dados locais por personagem.
- `cache/`: arquivos de cache (números online, eventos, compêndio, etc.).
- `conf/`: opções do cliente e configurações de GPU.
- `3rdpartylicences/`: textos das licenças de terceiros.
- `README.md`: este documento.

## Publicação no GitHub e arquivos grandes (Git LFS)

Este projeto contém muitos binários e mídias grandes (DLLs, EXEs, PNGs, DAT/SPR). Para versionar e fazer pull/push de forma confiável, recomenda-se usar o Git LFS.

Passos sugeridos:

1. Instale o Git LFS (https://git-lfs.com/).
2. No seu ambiente local, rode:
   - `git lfs install`
3. Utilize o `.gitattributes` (já incluído neste repositório) para rastrear automaticamente os formatos grandes via LFS.
4. Commit e push normalmente. O LFS cuidará do armazenamento dos binários de forma eficiente.

Padrões rastreados pelo LFS neste projeto incluem: `*.dll`, `*.exe`, `*.dat`, `*.spr`, `*.rcc`, `*.qm`, `*.bin`, `*.png`, `*.jpg`, `*.jpeg`, `*.webp`, `*.wav`, `*.ogg`, `*.mp3`.

## Licenças

Consulte a pasta `3rdpartylicences/` para informações sobre as licenças de componentes de terceiros (Qt, Battleye, EasyLogging, Google Protobuf, SFML, etc.).

---

Se precisar, ajuste os padrões de LFS no arquivo `.gitattributes` antes de subir para o GitHub, conforme o tamanho e a natureza dos arquivos adicionais que você pretende incluir.

Trigger workflow test at 2025-11-09 18:14:48

Trigger workflow test


Veja também: README-WORKFLOW.md (guia do workflow de release).
