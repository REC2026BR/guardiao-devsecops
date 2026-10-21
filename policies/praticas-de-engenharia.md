# Práticas de engenharia — regras gerais de sessão

Diferente de `policies/esquadrao-qa.md` (específica do Esquadrão de QA),
este arquivo cobre práticas gerais de como uma sessão do Claude Code conduz
qualquer trabalho de engenharia nos repositórios conectados ao Guardião.
Nasceu de uma avaliação do projeto externo
[obra/superpowers](https://github.com/obra/superpowers) (21/10/2026) — não
importamos o framework inteiro (ele assume um estilo de trabalho mais formal,
com portão de brainstorming antes de qualquer código, que não combina com
como este time delega), só as quatro práticas abaixo, que já tinham se
mostrado necessárias por incidentes reais desta mesma sessão.

## 1. Nenhuma alegação de sucesso sem evidência fresca

**Regra:** nunca reportar algo como "funciona", "verificado" ou "corrigido"
sem ter rodado o comando de verificação **nesta mesma resposta**, e nunca
confiar no relatório de um subagente sem checar o estado real depois
(`git log`/`git diff`, ambiente de verdade) — um subagente pode reportar
"concluído" sem ter commitado nada.

**Por que essa regra existe:** dois incidentes reais em 21/10/2026, na
mesma sessão:

- Um subagente despachado para corrigir achados de UX (NuTrocas) reportou
  "completed" sem ter feito nenhum commit — só foi descoberto porque a
  sessão coordenadora checou `git status`/`git log` do clone antes de
  confiar no relatório, em vez de repassar "terminado" adiante.
- Um PR (achado SEO-01, reescrita de `_redirects` no Cloudflare Pages) foi
  testado localmente (`wrangler pages dev`) e pareceu correto — mas o
  emulador local tem um comportamento diferente do Cloudflare real. O PR
  foi mesclado, foi ao ar, e quebrou rotas reais em produção (`/app/itens`
  e outras davam 404). Só foi detectado porque a sessão testou direto
  contra o domínio de produção depois do deploy — não porque o teste local
  tivesse "passado".

**Na prática:**
- Rodar a suíte de teste/comando de verificação de novo antes de qualquer
  frase de conclusão — não confiar em uma rodada anterior na mesma sessão
  se algo mudou desde então.
- Mudança que afeta produção (config de deploy, rota, infraestrutura): a
  verificação final é contra o ambiente real, não só local/emulado, quando
  isso for possível de fazer com segurança (sem escrever dado real).
- Relatório de subagente ("terminei", "corrigido") é hipótese, não fato —
  confirmar com `git diff`/estado real antes de repassar como concluído
  pra fora daquela sub-tarefa.

## 2. Revisão de código independente antes de mesclar

**Regra:** antes de mesclar um PR que a própria sessão escreveu, despachar
uma revisão de código independente (um agente ou skill de revisão sem o
histórico da sessão de implementação) — não só autorrevisão de quem
escreveu o código.

**Por quê:** nesta sessão, PRs escritos pela própria sessão coordenadora
foram mesclados só com autorrevisão; o único PR que recebeu revisão
cética de verdade (testar contra produção, questionar as alegações do
subagente que o escreveu) foi o do SEO-01 — justamente por desconfiança
prévia, não por hábito. Revisão independente por padrão pega mais cedo o
que a autorrevisão tende a não ver (o autor já decidiu que o código está
certo antes de reler).

**Na prática:** usar a skill de revisão de código disponível (`/code-review`
ou equivalente) antes de mesclar PRs autorais, não só quando já há suspeita
de problema.

## 3. Isolamento de trabalho paralelo: convenção de worktree

**Regra:** antes de criar isolamento manual pra trabalho paralelo (vários
agentes mexendo no mesmo repositório ao mesmo tempo), checar se o harness
já oferece uma ferramenta nativa de worktree/isolamento. Só cair pra
`git worktree` manual se não houver. Quando for `git worktree` manual,
usar um diretório `.worktrees/` dentro do próprio projeto (git-ignorado),
não clones completos separados em outro caminho.

**Por quê:** nesta sessão, 4 agentes trabalharam em paralelo em achados
independentes da auditoria do NuTrocas — funcionou, mas cada um ganhou um
clone completo do repositório num diretório próprio
(`Nutrocas-priv`, `Nutrocas-ux`, `Nutrocas-a11y`, `Nutrocas-api3perf`), mais
pesado e lento de criar do que seria necessário. `git worktree` (ou uma
ferramenta nativa equivalente) dá o mesmo isolamento (branch própria,
sem conflito de arquivo) compartilhando o `.git` do repositório, sem
precisar clonar tudo de novo a cada agente.

## 4. Descrição de skill: só "quando usar", nunca "o que faz"

**Regra:** a descrição (frontmatter) de uma skill deve dizer só em que
situação usá-la — sintomas, contexto, gatilho — nunca resumir o processo
ou o que ela faz por dentro. É a descrição que outra sessão lê pra decidir
se aquela skill é relevante agora; resumir o "como" nela não ajuda a
decisão de "quando" e ainda ocupa espaço do limite de caracteres.

**Na prática:** ao escrever ou revisar uma skill (ex.: `/esquadrao`),
começar a descrição com "Use quando..." e cortar qualquer trecho que
explique o funcionamento interno — isso vai no corpo do arquivo, não no
frontmatter.
