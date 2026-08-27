# Arquitetura — GreenKeeper Flora

Este documento existe pra permitir retomar o projeto em qualquer ponto sem precisar reconstruir o raciocínio do zero. Cobre visão geral, stack, modelo de dados, fontes, pipeline e riscos conhecidos.

---

## 1. Visão geral e propósito

O projeto tem dois papéis simultâneos, pelo mesmo conteúdo:

1. **Wiki pública** — enciclopédia de plantas pesquisável por qualquer visitante, com página rica por espécie (estilo dinâmico, cards, não um artigo denso tradicional).
2. **Fonte de dados do GreenKeeper** — o app (React Native/Expo, TypeScript, SQLite local, React Query) consome os dados de cuidado/cultivo de cada espécie via API gerada a partir do mesmo catálogo.

Escopo de conteúdo: espécies nativas do Brasil (plantas vasculares) + espécies cultiváveis/ornamentais populares globalmente, incluindo hortaliças/aromáticas, frutíferas, trepadeiras, arbóreas, flores, plantas medicinais tradicionais e carnívoras ornamentais, além de cultivares/variedades das espécies de maior interesse agrícola/ornamental (banana, citros, mirtilo etc.).

Estimativa de escopo: ~32-33 mil nativas vasculares + ~35 mil cultiváveis/ornamentais globais, com sobreposição — ordem de grandeza de **40-60 mil espécies** no registro completo, e uma fração disso (grandeza de milhares, crescendo ao longo de anos) com tratamento editorial completo.

---

## 2. Modelo de conteúdo: duas camadas

Pré-renderizar página estática pra cada uma das 40-60 mil espécies não cabe nos limites gratuitos de hospedagem (Cloudflare Pages: 20.000 arquivos). A solução é separar por profundidade de tratamento:

- **Camada A — Wiki curada.** Espécies com artigo completo, escrito à mão, imagem tratada, visual rico. Pré-renderizada como página estática pelo Astro. Cresce organicamente — comparável a projetos reais do gênero (ver seção 10), a expectativa realista é casa dos milhares ao longo de anos, não uma meta de curto prazo.
- **Camada B — Registro completo.** Todas as demais espécies do escopo (nativas + cultiváveis globais), como dado estruturado no Cloudflare D1, sem página estática pré-gerada. Renderizada sob demanda (template simples/stub) por um Cloudflare Worker quando acessada.

Uma espécie pode "subir" da Camada B pra Camada A a qualquer momento, sem mudar de schema — só passa a ganhar corpo de artigo e imagem.

---

## 3. Stack técnica e justificativa

| Decisão | Escolha | Por quê |
| --- | --- | --- |
| Conteúdo (Camada A) | Markdown/MDX + YAML front matter, versionado em Git | Portável, sem lock-in em banco proprietário; histórico de mudanças de graça via Git |
| Gerador de site | Astro (Content Collections + validação Zod) | Feito pra sites com muito conteúdo estruturado; zero JS por padrão; permite ilhas React se precisar de interatividade |
| Busca (Camada A) | Pagefind | Busca full-text estática, zero servidor, zero custo |
| Busca (A + B) | Índice JSON leve gerado no build + Fuse.js no client | Pagefind só indexa HTML gerado (só Camada A); esse índice cobre também a Camada B |
| Hospedagem | Cloudflare Pages | Free: 20.000 arquivos, 500 builds/mês, 100 domínios customizados, sem limite de banda listado nos termos oficiais |
| Fallback de hospedagem | GitHub Pages | Free: soft limit de 100 GB/mês, site até 1 GB |
| Banco (Camada B) | Cloudflare D1 | Free: 5 GB de storage, 5M leituras/escritas por mês — suficiente pra 40-60 mil registros estruturados |
| API dinâmica (Camada B) | Cloudflare Worker | Serve stub/consulta sob demanda sem manter servidor rodando |
| Imagens | Cloudflare R2 (10 GB grátis) | Mantém o Pages abaixo do limite de 20.000 arquivos; só Camada A recebe imagem tratada |
| API pro GreenKeeper | JSON estático gerado no build | App consome via fetch, sem servidor ao vivo, cacheável por CDN |
| Comentários | Giscus (GitHub Discussions) | Zero servidor pra manter, zero custo, infraestrutura do GitHub |
| CI/CD | GitHub Actions (herdado do template) | TypeScript check, ESLint, security audit, Dependabot |

**Alternativa considerada e descartada:** MediaWiki (motor de wiki tradicional, usado por projetos como Wikispecies/Practical Plants). Exige servidor + banco rodando 24/7, o que conflita com o requisito de custo zero permanente. A abordagem estática + serverless cobre o mesmo caso de uso sem esse custo fixo.

---

## 4. Modelo de dados

Schema completo de uma entrada de espécie (`src/content/especies/*.md`), validado via Zod no `src/content/config.ts`:

```yaml
---
slug: "alecrim"
nomes_populares: ["Alecrim"]
nome_cientifico: "Rosmarinus officinalis"
familia: "Lamiaceae"
grupo_botanico: "eudicotiledoneas"   # facilita filtro; Domínio/Reino são constantes e não entram como faceta de busca
categorias: ["hortalica-aromatica", "medicinal-tradicional", "cha", "aroma-perfumado"]

# Camada 1 — dados acionáveis pro GreenKeeper
app_data:
  intervalo_rega_dias: 7
  intervalo_adubacao_dias: 45
  epoca_poda: ["primavera"]
  dificuldade: "facil"
  luz: "sol-pleno"
  estagios_crescimento: ["muda", "vegetativo", "floracao"]

# Camada 2 — atributos específicos por categoria
atributos:
  hortalica_aromatica:
    parte_usada_culinaria: "folhas"
    epoca_colheita: ["ano-todo"]

# Camada 3 — uso medicinal (obrigatório quando a categoria existe; aviso é injetado pelo template, não editável)
uso_medicinal:
  usos_tradicionais: ["digestivo", "estimulante circulatorio leve"]
  parte_utilizada: "folhas"
  fonte_referencia: "RENISUS / Ministério da Saúde"

# Camada 4 — cultivares (ICNCP: abaixo da espécie, ex: Citrus sinensis 'Bahia')
cultivares: []
---
Corpo do artigo em Markdown (só na Camada A)...
```

**Regra de validação:** se `categorias` contém `medicinal-tradicional`, o Zod exige `uso_medicinal` preenchido (build falha sem isso). O aviso fixo de "não substitui orientação médica, consulte um profissional" é responsabilidade do **template**, não do front matter — garante que nunca falta por esquecimento humano.

**Taxonomia completa armazenada, mas não toda usada como filtro:** a escada Domínio → Reino → Filo/Clado → Ordem → Família → Gênero → Espécie → Cultivar fica guardada pra exibição (breadcrumb) e rigor científico. As facetas de busca reais são: grupo botânico amplo, família e categoria de uso — Domínio e Reino são praticamente constantes em todo o catálogo (Eukaryota/Plantae) e não discriminam nada.

---

## 5. Categorias de uso (tags multi-valor)

`ornamental`, `frutifera`, `hortalica-aromatica`, `trepadeira`, `arborea`, `flor`, `cha`, `medicinal-tradicional`, `aroma-perfumado`, `carnivora-ornamental`.

Cada categoria pode ter um bloco de `atributos` próprio (ex: `trepadeira.mecanismo`, `carnivora.tipo_armadilha`) — mas só ganha esse bloco se tiver campo com substância real; caso contrário fica só como tag simples de filtro, sem inventar dado pra preencher.

---

## 6. Fontes de dados

| Finalidade | Fonte | Observação |
| --- | --- | --- |
| Taxonomia global | GBIF (API + bulk download) | Backbone geralmente CC0/CC-BY; checar licença por registro de ocorrência |
| Nomes aceitos globais | Plants of the World Online / WCVP (Kew) | Download em lote |
| Taxonomia global | Catalogue of Life | CC BY |
| Dados estruturados gerais | Wikidata | CC0 |
| Nativas do Brasil | Flora e Funga do Brasil (JBRJ) | Bulk Darwin Core Archive, atualizado semanalmente — checar termos de uso oficiais antes de importar em massa |
| Cuidado/cultivo | Perenual API | Free tier com chave de API — checar limites atuais antes de rodar em lote |
| Cultivares brasileiras | Embrapa — RNC + RENASEM | Dados abertos federais |
| Cultivares internacionais | USDA GRIN-Global | Domínio público (governo dos EUA) |
| Medicinal tradicional | RENISUS (Ministério da Saúde) | 71 espécies oficiais |
| Medicinal tradicional | ANVISA (RDC nº 10/2010) | Monografias oficiais |
| Medicinal (complementar) | Kew — *State of the World's Plants* (2017) | CC BY 4.0, ~28.187 espécies com uso medicinal registrado |
| Texto de referência (não copiar) | Wikipedia / Wikispecies | CC BY-SA — só consultar, escrever com palavras próprias |
| Imagens | Wikimedia Commons | Checar licença por arquivo individual; só usado pra Camada A |

**Princípio geral:** nunca consultar API de terceiro ao vivo em produção. Toda fonte externa é importada, normalizada e guardada como cópia própria (arquivo `.md` ou linha no D1) — se a API de origem mudar de política ou sair do ar, o site continua funcionando.

---

## 7. Pipeline de dados e build

```
Fontes externas (bulk download)
        │
        ▼
Scripts de importação/normalização (Node/Python, rodam local)
        │
        ├──> Camada A: gera .md em src/content/especies/ (com corpo de artigo, quando existir)
        │        │
        │        ▼
        │    Build Astro (Content Collections + validação Zod)
        │        │
        │        ▼
        │    Deploy Cloudflare Pages ──> gera plants.json estático (API pro GreenKeeper)
        │
        └──> Camada B: carga em lote no Cloudflare D1
                 │
                 ▼
             Cloudflare Worker (consulta sob demanda + stub de página)
```

Índice de busca unificado (Fuse.js) é gerado a partir de ambas as camadas no momento do build, cobrindo nome popular/científico/família de todo o catálogo.

---

## 8. Integração com o GreenKeeper

- GreenKeeper já usa SQLite local + React Query.
- Fluxo recomendado: o app faz fetch periódico do `plants.json` (ou consulta o Worker pra Camada B) e **cacheia no SQLite que já existe**, em vez de depender de rede a cada tela.
- Isso garante uso offline e evita acoplar o app diretamente ao D1/Worker em tempo real.

---

## 9. Aspectos legais e licenciamento

- **Fatos e dados não são protegidos por direito autoral** — taxonomia, dias de rega, época de colheita podem vir de qualquer fonte listada na seção 6.
- O que exige cuidado é **texto/prosa** de terceiros — nunca copiar da Wikipedia ou de outra fonte, sempre reescrever com palavras próprias.
- CC BY-SA (Wikipedia/Commons) exige atribuição + compartilhamento igual só pro conteúdo derivado específico, não contamina o projeto inteiro.
- **Pendente:** confirmar o texto exato da licença de reuso de dados da Flora e Funga do Brasil e da Embrapa antes de importação em massa.
- **Pendente:** decidir a licença do *conteúdo* (texto/dados das espécies) separada da licença do *código* (MIT) — ver `docs/DECISIONS.md`.

---

## 10. Comparação com projetos semelhantes (viabilidade)

- **PFAF (Plants For A Future)** — banco de comestíveis/medicinais/úteis. Começou com 1.500 plantas do terreno do fundador, hoje tem mais de 8.000 entradas (7.000 temperadas + 1.000 tropicais), mantido desde 2008 por um administrador de banco de dados empregado pela instituição. Levou décadas pra sair de 1.500 pra 8.000 entradas *ricas* — mesmo com equipe dedicada. Isso calibra a expectativa de crescimento da Camada A: casa dos milhares, ao longo de anos, não uma meta de curto prazo. A arquitetura de duas camadas é o que torna isso viável desde o primeiro dia (Camada B garante completude de busca enquanto a Camada A cresce organicamente).

---

## 11. Riscos conhecidos e mitigação

| Risco | Mitigação |
| --- | --- |
| Busca não cobre a Camada B (Pagefind só indexa HTML estático) | Índice JSON leve cobrindo as duas camadas (seção 7) |
| Sinonímia — nomes científicos mudam com revisões taxonômicas | Processo de re-sync periódico contra GBIF/POWO (trimestral), sinaliza divergência sem sobrescrever sozinho |
| Idioma das fontes (inglês/latim vs. português) | Normalização de nomes populares/textos no script de importação |
| Direito de imagem em escala (checar licença por arquivo é inviável em massa) | Imagem tratada só na Camada A |
| Dependência de API de terceiro gratuita mudar de política | Nunca consultar ao vivo em produção — sempre importar e guardar cópia própria (seção 6) |
| Estouro do limite de arquivos do Cloudflare Pages (20.000) | Imagens fora do deploy (R2), Camada B fora do build estático (D1 + Worker) |
| Estouro do limite de storage do R2 (10 GB) | Compressão/WebP obrigatório, imagem só pra Camada A |

---

## 12. Decisões em aberto

Ver `docs/DECISIONS.md` para o registro vivo de decisões pendentes (ex: Astro vs. Next.js definitivo, Giscus vs. comentário anônimo, domínio próprio vs. subdomínio gratuito, cultivares populares com página própria ou subseção, escopo incluir ou não fungos/algas, licença do conteúdo).
