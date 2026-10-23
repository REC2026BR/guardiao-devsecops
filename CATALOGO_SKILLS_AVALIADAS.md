# Catálogo de skills e repositórios externos avaliados

Registro de toda avaliação de skill/framework externo feita por uma sessão
do Claude Code nesta família de projetos — adotada ou rejeitada. Ver
`policies/avaliacao-skills-externas.md` pro processo usado em cada
avaliação. Nunca editar o veredito de uma entrada antiga sem registrar a
reavaliação como nova entrada com data — isto é histórico, não um
documento vivo reescrito por cima.

Mantido a partir de 21/10/2026. Antes de avaliar algo novo, procurar aqui
primeiro — pode já ter sido avaliado.

---

## obra/superpowers

- **O que é**: framework de metodologia de desenvolvimento assistido por
  agente — 15 skills de processo (brainstorming, planos, TDD, debug
  sistemático, revisão de código, git worktree, etc.).
- **Licença**: não verificada formalmente (framework de processo, não
  código vendorizado).
- **Veredito**: **adotado parcialmente** — não o framework inteiro (assume
  um portão de brainstorming formal antes de qualquer código, que não
  combina com como este time delega), só 4 práticas específicas, escritas
  com nossas próprias palavras (não cópia de arquivo) em
  `policies/praticas-de-engenharia.md`: nenhuma alegação de sucesso sem
  evidência fresca, revisão de código independente antes de mesclar,
  convenção de worktree pra trabalho paralelo, descrição de skill só
  "quando usar".
- **Data**: 21/10/2026.

## NousResearch/hermes-agent

- **O que é**: agente autônomo sempre-ligado, com memória persistente em
  disco por usuário, controlado via Telegram/Discord/Slack, com acesso a
  terminal e browser. Bundla uma biblioteca grande de skills, incluindo
  3 de design visual (`claude-design`, `popular-web-designs`,
  `design-md`).
- **Licença**: MIT.
- **Veredito do agente em si**: **rejeitado** — arquitetura de agente
  sempre-ligado com memória persistente por usuário final é risco de
  LGPD e superfície de ataque desnecessária (terminal/browser) pra um
  caso de uso que é só "ajudar no cadastro de um item". Projeto também
  muito novo (fev/2026) pra ancorar um fluxo central do produto.
- **Veredito das skills de design (`claude-design`, `popular-web-designs`)**:
  **rejeitado** — `popular-web-designs` é um catálogo de 54 sistemas de
  design de marcas conhecidas (Stripe, Linear, Vercel etc.), exatamente o
  tipo de "cópia visual de outra empresa" que o usuário não quer. A
  tentativa de vendorizar os templates (mesmo filtrados) foi bloqueada
  duas vezes pelo classificador de segurança do harness (conteúdo com
  instrução endereçada a ferramenta de outro agente — `write_file`,
  `browser_vision`). Substituído por uma skill própria,
  `design-visual`, escrita do zero, no NuTrocas.
- **Data**: 21/10/2026.

## anthropics/skills — skill `frontend-design`

- **O que é**: skill oficial da Anthropic contra o "jeito genérico de IA"
  em design — fundamenta cada design no assunto real do produto, lista
  defaults específicos a evitar (paleta creme+terracota, preto+verde-ácido,
  "kit de card SaaS", tipografia batida), sem citar marca de terceiro pra
  copiar. Mesma skill replicada (confirmado byte a byte idêntico) em
  `anthropics/claude-code` (`plugins/frontend-design/`).
- **Licença**: Apache 2.0.
- **Veredito**: **adotado, vendorizado sem modificação** — primeira
  parte, método (não catálogo de marca), conteúdo limpo. Em
  `.claude/skills/frontend-design/` no NuTrocas, PR #24.
- **Data**: 21/10/2026.

## anthropics/skills — demais skills do repositório

- **`brand-guidelines`**: rejeitado — é a identidade visual da própria
  Anthropic (cores/fontes deles), pensada pra estilizar documento/slide
  com a marca da Anthropic. Traria marca de outra empresa pro produto.
- **`theme-factory`**: rejeitado — 10 temas prontos de cor/fonte num
  catálogo pra escolher, voltado a slides. Mesmo padrão de "escolher de
  uma prateleira" rejeitado em outros casos.
- **`canvas-design`**: fora de escopo — gera arte estática (pôster
  PNG/PDF), não interface de app.
- **`webapp-testing`**: fora de escopo por redundância — testa app local
  com Playwright, mas já temos isso coberto pelo Esquadrão
  (`scripts/qa_squad/`).
- Demais (`docx`, `pdf`, `pptx`, `xlsx`, `mcp-builder`, `claude-api`,
  `slack-gif-creator`, `academy-guide`, `algorithmic-art`,
  `discernment-nudge`, `doc-coauthoring`, `internal-comms`,
  `skill-creator`, `web-artifacts-builder`): sem relação com design de
  UI, não avaliados em profundidade.
- **Data**: 21/10/2026.

## vercel-labs/web-interface-guidelines (+ vercel-labs/agent-skills)

- **O que é**: mais de 100 regras concretas de acessibilidade,
  formulários, animação, performance, i18n e antipadrões de UX — um
  linter de boas práticas, não estilo visual.
- **Licença**: MIT (ambos os repositórios).
- **Veredito**: **adotado, com adaptação** — trazido como retrato fixo
  traduzido (não busca ao vivo via `WebFetch`, que não é confiável neste
  tipo de ambiente de sessão; e preferimos auditar conteúdo externo uma
  vez a confiar nele de novo a cada execução). Em
  `.claude/skills/web-design-guidelines/` no NuTrocas, PR #25.
- **Data**: 21/10/2026.

## nextlevelbuilder/ui-ux-pro-max-skill

- **O que é**: banco de dados pesquisável de design (79 estilos, 192
  paletas, 74 combinações de fonte, 119 diretrizes de UX, 22 stacks),
  com uma ferramenta CLI em Python (`search.py`) que gera um "sistema de
  design" a partir de palavras-chave.
- **Licença**: MIT.
- **Legitimidade**: confirmada — 138.414 estrelas reais (API do GitHub,
  não o número de 29.636 da descrição colada), código Python auditado
  sem chamada de rede/eval/exec suspeita, projeto ativo e testado.
- **Veredito**: **rejeitado**, apesar de legítimo — três razões: (1)
  contraria a decisão de identidade própria do time — o núcleo da
  ferramenta é "escolher de um catálogo pronto por palavra-chave", o
  oposto de construir identidade; (2) cobre 22 stacks, a maioria
  irrelevante (SwiftUI, Flutter, WPF etc. — o NuTrocas é só React web);
  (3) redundante com o que já foi adotado (`frontend-design` já cobre
  "fundamentar no assunto real", `web-design-guidelines` já cobre
  correção técnica) — essa ferramenta seria mais pesada (código Python
  executável, não só markdown) pro mesmo resultado.
- **Data**: 21/10/2026.

---

## Como adicionar uma entrada nova

Uma seção por repositório avaliado, nesta ordem: o que é, licença,
legitimidade (se houve dúvida a verificar), veredito (adotado/rejeitado/
adotado parcialmente, com razão), onde foi vendorizado se aplicável
(caminho + PR), e data. Rejeição também é registro válido — não pular
registrar só porque a resposta foi "não".
