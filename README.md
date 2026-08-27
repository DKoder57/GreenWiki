# GreenKeeper Flora

**Wiki de plantas nativas e cultiváveis que funciona como enciclopédia pública de pesquisa e, ao mesmo tempo, como fonte de dados estruturados para o app GreenKeeper.**

[![Build](https://img.shields.io/github/actions/workflow/status/DKoder57/greenkeeper-flora/typescript-check.yml?label=build)](https://github.com/DKoder57/greenkeeper-flora/actions)
[![License](https://img.shields.io/github/license/DKoder57/greenkeeper-flora)](https://github.com/DKoder57/greenkeeper-flora/blob/main/LICENSE)
[![Last Commit](https://img.shields.io/github/last-commit/DKoder57/greenkeeper-flora)](https://github.com/DKoder57/greenkeeper-flora/commits/main)

> Nome do repositório e badges acima são sugestão — ajuste para o nome real assim que criar o repo a partir do template.

## 📑 Sumário

- [Sobre](#-sobre)
- [Features](#-features)
- [Tecnologias](#️-tecnologias)
- [Como rodar](#️-como-rodar)
- [Estrutura do projeto](#-estrutura-do-projeto)
- [Roadmap](#️-roadmap)
- [Contribuindo](#-contribuindo)
- [Licença](#-licença)

## 📖 Sobre

Cataloga espécies nativas do Brasil e as plantas cultiváveis/ornamentais mais populares globalmente — hortaliças e aromáticas, frutíferas, trepadeiras, arbóreas, flores, plantas medicinais tradicionais e carnívoras ornamentais — em formato de wiki pública e pesquisável. O mesmo conteúdo alimenta o app GreenKeeper via uma API estática/dinâmica, sem duplicar dado nenhum entre as duas pontas.

Documentação técnica completa (arquitetura, fluxo de dados, decisões) vive em [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) e `docs/DECISIONS.md` — aqui fica só o essencial pra começar.

## ✨ Features

- [ ] Wiki pública com página por espécie (taxonomia completa, categorias de uso, cultivares)
- [ ] Busca unificada por nome popular, nome científico e família
- [ ] API estática consumida pelo app GreenKeeper (dados de cuidado/cultivo)
- [ ] Registro completo (Camada B) de todas as espécies nativas + cultiváveis populares, mesmo sem artigo rico
- [ ] Comentários por espécie via GitHub Discussions
- [ ] Aviso automático em todo conteúdo de uso medicinal tradicional (ver [Roadmap](#️-roadmap))

## 🛠️ Tecnologias

| Camada | Tecnologia |
| --- | --- |
| Frontend | Astro + Tailwind CSS |
| Busca | Pagefind (Camada A) + índice JSON próprio (Camada A + B) |
| Banco de dados | Markdown/YAML versionado em Git (Camada A) + Cloudflare D1 (Camada B) |
| Armazenamento de imagem | Cloudflare R2 |
| Infra / Deploy | Cloudflare Pages + Cloudflare Workers |
| Comentários | Giscus (GitHub Discussions) |
| CI/CD | GitHub Actions ([workflows](.github/workflows)) |

## ▶️ Como rodar

```
# Clonar
git clone https://github.com/DKoder57/greenkeeper-flora.git
cd greenkeeper-flora

# Instalar dependências
npm install

# Rodar em desenvolvimento
npm run dev
```

## 📂 Estrutura do projeto

```
.
├── .github/              # CI (GitHub Actions) e Dependabot
├── docs/                 # Arquitetura, roadmap e decisões técnicas
│   ├── ARCHITECTURE.md
│   └── DECISIONS.md
├── issues.csv            # Backlog inicial (F0-F6), importado como GitHub Issues
├── src/
│   ├── content/
│   │   └── especies/     # Entradas de espécie (Markdown + front matter)
│   └── pages/
└── public/
```

## 🗺️ Roadmap

Planejamento macro por fases (F0 a F6) já detalhado em `issues.csv` e importado como Issues do GitHub. O raciocínio técnico de cada fase está em [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

## 🤝 Contribuindo

Projeto pessoal/solo — sugestões e issues são bem-vindas, mas não há processo formal de contribuição externa no momento.

## 📄 Licença

Código sob licença MIT (ver `LICENSE`). **O conteúdo/dados das espécies (texto, taxonomia) tem licenciamento próprio a definir** — considerar CC BY-SA para compatibilidade com fontes como Wikipedia/Wikidata, já que código e conteúdo são ativos diferentes com necessidades de licença diferentes (ver `docs/DECISIONS.md`).
