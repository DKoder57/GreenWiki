# Decisões Técnicas

Registro de decisões importantes do projeto, para evitar perda de contexto e facilitar manutenção futura.

---

### Data: 2026-08-27

**Decisão:**
Conteúdo em Markdown/YAML versionado em Git (Camada A), em vez de banco gerenciado.

**Motivo:**
Custo zero e permanência máxima. Portável — se o host sumir, o conteúdo continua acessível e pode ser reimplantado em outro lugar em minutos, sem depender de exportar um banco proprietário.

**Alternativas consideradas:**
- Banco gerenciado (Supabase, PlanetScale) — risco de mudança de política de tier gratuito
- CMS headless tradicional

---

### Data: 2026-08-27

**Decisão:**
Astro como gerador de site, com Content Collections + validação Zod.

**Motivo:**
Feito especificamente pra sites com muito conteúdo estruturado; a validação Zod garante que campos obrigatórios (ex: aviso médico) nunca faltem por esquecimento humano. Zero JS por padrão, com opção de ilhas React quando precisar de interatividade.

**Alternativas consideradas:**
- Next.js com export estático
- MediaWiki (motor de wiki tradicional) — descartado por exigir servidor + banco rodando 24/7, conflita com o requisito de custo zero permanente

---

### Data: 2026-08-27

**Decisão:**
Modelo de conteúdo em duas camadas: Camada A (curada, página estática) e Camada B (registro completo no Cloudflare D1, sem página pré-gerada).

**Motivo:**
Pré-renderizar as ~40-60 mil espécies do escopo estourava o limite de 20.000 arquivos do Cloudflare Pages (free) e o tempo de build. Separar por profundidade de tratamento resolve isso sem abrir mão de completude de busca.

**Alternativas consideradas:**
- Pré-renderizar tudo como página estática (inviável nos limites gratuitos)
- Limitar o catálogo só às espécies populares (perderia a meta de completude nativa + cultivável)

---

### Data: 2026-08-27

**Decisão:**
Hospedagem no Cloudflare Pages, com GitHub Pages como fallback.

**Motivo:**
Free tier mais generoso (sem limite de banda listado nos termos oficiais) e integração nativa com D1, R2 e Workers, todos do mesmo provedor.

**Alternativas consideradas:**
- Netlify (tier gratuito historicamente mais restrito)
- GitHub Pages como opção única

---

### Data: 2026-08-27

**Decisão:**
Imagens hospedadas no Cloudflare R2, fora do deploy do site.

**Motivo:**
Evita que o Cloudflare Pages estoure o limite de 20.000 arquivos por causa de fotos versionadas junto com o conteúdo.

**Alternativas consideradas:**
- Versionar imagem direto no repositório Git
- CDN de imagem de terceiro

---

### Data: 2026-08-27

**Decisão:**
Comentários via Giscus (GitHub Discussions).

**Motivo:**
Zero custo, zero servidor próprio pra manter — a permanência fica atrelada à infraestrutura do GitHub.

**Alternativas consideradas:**
- Cloudflare D1 + Worker (permitiria comentário anônimo, mas exige mais manutenção própria)

---

### Data: 2026-08-27

**Decisão:**
Escopo de conteúdo: espécies nativas do Brasil (vasculares) + espécies cultiváveis/ornamentais populares globalmente, incluindo cultivares das espécies de maior interesse agrícola/ornamental.

**Motivo:**
Cobre ao mesmo tempo o valor de wiki de flora brasileira e a utilidade real como fonte de dados pro GreenKeeper, cujo público não se limita a plantas nativas.

**Alternativas consideradas:**
- Só espécies nativas do Brasil
- Catálogo mundial completo, incluindo fungos/algas (~350-400 mil espécies — inviável pra um curador só)

---

### Data: 2026-08-27

**Decisão:**
Uso medicinal tradicional é documentado como fato histórico/cultural (não como instrução de tratamento), com aviso fixo injetado automaticamente pelo template sempre que o bloco `uso_medicinal` existir.

**Motivo:**
Mantém valor enciclopédico real sem virar aconselhamento médico, e elimina o risco de o aviso ser esquecido por edição manual.

**Alternativas consideradas:**
- Não incluir categoria medicinal no catálogo
- Aviso como texto livre por espécie, digitado manualmente (risco de omissão)

---

### Data: 2026-08-27

**Decisão:**
Nenhuma API de terceiro (Perenual, GBIF, etc.) é consultada ao vivo em produção — todo dado externo é importado e versionado como cópia própria.

**Motivo:**
Evita que o site fique refém de uma API gratuita mudar de política, preço ou sair do ar.

**Alternativas consideradas:**
- Consultar as APIs de terceiro em tempo real a cada requisição do usuário

---

<!-- Copie o bloco acima para cada nova decisão -->

## Pendências (ainda não fechadas)

- [ ] Astro vs. Next.js — confirmação definitiva (Astro está sendo usado como padrão, mas não foi formalmente "fechado")
- [ ] Domínio próprio (custo ~US$10-15/ano) vs. subdomínio gratuito (`.pages.dev`)
- [ ] Cultivares populares (banana prata, laranja bahia) — página própria (Camada A) ou subseção da espécie?
- [ ] Incluir fungos/algas no escopo (Flora e Funga do Brasil cobre) ou manter só vasculares?
- [ ] Licença do *conteúdo* das espécies (texto/taxonomia), separada da licença do código (MIT) — candidata: CC BY-SA
- [ ] Confirmar texto exato da licença de reuso de dados da Flora e Funga do Brasil e da Embrapa antes de importação em massa
- [ ] `cha` herda o aviso médico obrigatório sempre, ou só quando descrito com finalidade terapêutica?
