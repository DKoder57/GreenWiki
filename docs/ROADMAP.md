# Roadmap

Planejamento macro do projeto, por fases. O detalhamento de cada item vive nas Issues (ver `issues.csv` e convenção `[Fase][Categoria] Título`).

- [ ] **F0 — Setup**: adaptar os workflows de CI/CD do template para a stack Astro/TypeScript deste projeto
- [ ] **F1 — Arquitetura**: inicializar o projeto Astro com Content Collections, definir o schema Zod completo (espécie, cultivar, `app_data`, `atributos`, `uso_medicinal`) e configurar os design tokens/paleta de cores
- [ ] **F2 — Dados**: scripts de importação em lote das fontes externas (GBIF, Flora e Funga do Brasil, Perenual, Embrapa RNC/RENASEM, RENISUS/ANVISA)
- [ ] **F3 — Infra**: provisionar Cloudflare D1 (Camada B), Worker de leitura sob demanda, R2 (imagens) e deploy no Cloudflare Pages
- [ ] **F4 — Frontend**: home, listagem por família com busca (Pagefind), índice de busca unificado (Camada A + B) e página de detalhe da espécie
- [ ] **F5 — Integrações**: comentários via Giscus, API JSON estática para o GreenKeeper e cache local no SQLite do app
- [ ] **F6 — Conteúdo e manutenção**: popular as primeiras espécies da Camada A, processo de sincronização periódica de nomenclatura e configuração do Kanban (GitHub Project)

> Sem detalhamento excessivo aqui — cada fase vira Issues específicas conforme o desenvolvimento avança (ver `issues.csv`).
