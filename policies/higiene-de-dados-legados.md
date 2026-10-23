# Higiene de dados legados — levantamento e correção sem segredo de produção

Cobre o cenário em que um achado de produto (ex.: um valor de texto livre
fora de uma lista fechada, um campo nulo que não deveria estar) exige
consultar — e às vezes corrigir — dado real em produção, sem que a sessão
do Claude Code (ou o Guardião) jamais precise segurar uma credencial de
banco de produção. Isso é trabalho **específico de cada produto da
família** (a lista de categorias do NuTrocas não significa nada pro
Dr-Apto, por exemplo) — não algo que o Guardião execute central, só o
padrão/checklist abaixo, pra qualquer sessão replicar quando precisar.

## Por que essa regra existe

Em 23/10/2026, no NuTrocas, um usuário reportou um bug visual real
("Eletronicos E Games" aparecendo numa tela) que revelou um problema mais
amplo: valores de categoria fora da lista fechada do frontend, legado de
antes dela existir. Pra medir o tamanho real do problema era preciso
consultar a base de produção — mas a sessão não tem (e não deveria ter)
credencial de banco de produção.

A solução que funcionou, documentada abaixo, levou três tentativas até
ficar certa:

1. **Colar o script no Web Shell do Render falhou duas vezes** — o
   terminal web vaza sequências de escape de modo "colar" (bracketed
   paste, `^[[200~`/`~`) como texto literal, corrompendo qualquer coisa
   colada ali (confirmado com heredoc e com um one-liner em base64,
   mesma causa nos dois). A solução foi comitar o script dentro do
   próprio repositório (já presente no container implantado) e pedir só
   um caminho curto pra digitar — nada de colar.
2. **O script comitado não chegava no container** — `render.yaml` usa
   `dockerContext: ./backend`, e o `Dockerfile` só copia os arquivos
   listados explicitamente num `COPY`; um script fora de `backend/` (ou
   dentro dela mas esquecido do `COPY`) nunca chega na imagem implantada,
   mesmo commitado e com o build passando. Esse exato erro foi cometido
   **duas vezes** — na primeira versão do script de levantamento (PR #37
   no NuTrocas) e, reintroduzido, na primeira versão do script de
   correção (PR #39) — só pego pela revisão de código independente antes
   do merge, não por quem escreveu o script.
3. **Script de correção sem dry-run seria arriscado por padrão** — a
   versão que foi pra produção só aplica mudança com uma flag explícita
   (`--aplicar`); sem a flag, só mostra o que mudaria.

## Regra inegociável

1. **Nunca manusear credencial de banco de produção.** O script roda de
   dentro do próprio serviço do produto (ex.: Render Web Shell), onde a
   variável de ambiente (`DATABASE_URL` ou equivalente) já está
   configurada — a sessão nunca pede, vê ou guarda esse valor.
2. **Levantamento (leitura) e correção (escrita) são dois scripts
   separados.** Nunca um script que já escreve de cara — primeiro mede,
   só depois, com o usuário ciente do tamanho do problema, se prepara a
   correção.
3. **Script de correção nunca aplica por padrão.** Precisa de uma flag
   explícita (ex. `--aplicar`); sem ela, só mostra (dry-run) o que
   mudaria, sem escrever nada.
4. **Correção automática só cobre casos inequívocos.** Match exato após
   normalização (sem acento, sem diferença de maiúscula/minúscula, sem
   espaço nas pontas) com um valor canônico já existente — nunca
   inferência de "provavelmente quis dizer X". Qualquer valor que exija
   julgamento de negócio (mapear texto livre pra uma entre várias opções
   plausíveis) fica de fora de propósito, documentado pra decisão
   humana.
5. **Testar localmente nos dois modos antes de indicar pro usuário**
   rodar contra produção — banco de teste descartável, dry-run não altera
   nada, `--aplicar` corrige só os casos esperados e deixa o resto
   intocado.
6. **Confirmar que o script realmente chega no ambiente onde vai
   rodar** antes de instruir o usuário a executá-lo — checar
   explicitamente o contexto de build (Docker `dockerContext`/`COPY`,
   sync remoto, etc.); não presumir que "está commitado" equivale a
   "está no container". Isso é responsabilidade também de quem revisa o
   PR, não só de quem escreve o script (ver incidente #2 acima).
7. **Nunca colar o conteúdo do script num terminal web de terceiro.**
   Comitar o script no repositório (ele já está no container/ambiente
   que vai executá-lo) e passar pro usuário só um caminho de arquivo
   curto pra digitar — nunca um bloco de código pra colar.

## Checklist antes de dizer "pode rodar" pro usuário

- [ ] Em nenhum momento a sessão pediu, viu ou guardou a credencial de
      banco de produção — o script só roda de dentro do próprio serviço,
      onde a variável de ambiente já existe.
- [ ] Script de levantamento só lê (nenhum `UPDATE`/`DELETE`/`INSERT`).
- [ ] Script de correção tem dry-run como padrão e uma flag explícita
      pra aplicar de verdade.
- [ ] Regra de match da correção automática é normalização exata —
      nada de inferência; casos ambíguos ficam documentados, não
      corrigidos sozinhos.
- [ ] Testado localmente (banco descartável) nos dois modos.
- [ ] Confirmado — e coberto pela revisão de código — que o arquivo
      chega de fato no ambiente onde vai rodar.
- [ ] Comando final pro usuário é um caminho de arquivo pra digitar,
      nunca conteúdo pra colar.

## Referência

NuTrocas, 23/10/2026: `backend/levantamento_categorias.py` (PR #37) +
`backend/corrige_categorias_obvias.py` (PR #39).
