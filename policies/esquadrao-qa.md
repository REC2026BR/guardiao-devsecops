# Esquadrao de QA de Negocio — regra de registro

Cobre tanto o pipeline reutilizavel (`scripts/qa_squad/motor.py`, fluxo via
Playwright) quanto simulacoes ad-hoc de regras de negocio (cenarios escritos
sob medida pra aquele projeto, rodando contra um banco de teste isolado, sem
usar o motor generico). As duas formas sao legitimas — mas as duas precisam
seguir a mesma regra de registro abaixo.

## Por que essa regra existe

Em 21/10/2026, uma simulacao de negocio (13 cenarios de negociacao) rodou de
verdade no NuTrocas, achou e corrigiu 5 bugs reais — mas so ficou registrada
na conversa (resumida depois pelo sistema) e nos commits de correcao, sem
nenhum artefato no repositorio confirmando que a simulacao tinha acontecido.
Isso quase causou desconfianca sobre se o trabalho relatado era real. A regra
abaixo fecha essa lacuna.

## Regra inegociavel

Toda execucao do Esquadrao — pipeline ou ad-hoc — so pode ser reportada como
concluida depois de um commit contendo um registro verificavel em
`scripts/qa_squad/historico.jsonl` (fluxos do motor generico) ou
`scripts/qa_squad/simulacoes_negocio.jsonl` (simulacao de negocio ad-hoc, sem
o motor generico). Sem esse commit, a execucao nao existe pra qualquer outra
sessao ou pessoa que vier depois — "eu rodei" vira fato so quando ha commit
provando.

Esquema minimo de uma linha de `simulacoes_negocio.jsonl`:

```json
{
  "gerado_em": "2026-10-21T00:15:28+00:00",
  "projeto": "NuTrocas",
  "tipo": "simulacao_negocio_adhoc",
  "cenarios_testados": 13,
  "ambiente": "postgres_teste_isolado",
  "achados_confirmados": [
    {"descricao": "ValueGapDirection com descricao publica invertida", "commit": "4b5836b"}
  ],
  "achados_refutados": 1,
  "sessao": "https://claude.ai/code/session_..."
}
```

`HISTORICO.md` (fluxos do motor) e `SIMULACOES_NEGOCIO.md` (ad-hoc) devem ser
regenerados a partir desses JSONL, nunca editados a mao — mesmo padrao que
`PROJECT_MAP.md` ja usa.

## Checklist antes de reportar "o Esquadrao terminou"

1. A simulacao rodou contra ambiente isolado (nunca producao com escrita).
2. Cada achado passou pela fase cetica de verificacao antes de virar commit
   de correcao.
3. O commit do fix (se houver) referencia os cenarios/achados que o
   motivaram.
4. Existe um commit — pode ser o mesmo do fix — com o registro em
   `simulacoes_negocio.jsonl` ou `historico.jsonl`.
5. So depois dos 4 itens acima o resultado pode ser comunicado como
   concluido pra fora da sessao que rodou.

## Economia de tokens: PROJECT_MAP.md nao e so um artefato do Esquadrao

O objetivo de "economia" do sistema nao se resume a medicao do
`limpar_codigo` (`economista.py`) sobre o codigo-fonte. O efeito maior e
evitar que qualquer sessao futura do Claude — rodando o Esquadrao ou nao —
gaste tokens remapeando a estrutura do projeto do zero (Glob/Grep
exploratorio amplo) quando ja existe um `PROJECT_MAP.md` gerado
(`scripts/qa_squad/project_map.py`).

**Convencao obrigatoria em todo repositorio conectado ao Guardiao:** o
`CLAUDE.md` da raiz (lido automaticamente por qualquer sessao do Claude Code
no inicio do trabalho) deve conter uma secao curta instruindo a ler
`PROJECT_MAP.md` antes de explorar o codigo com Glob/Grep/Read amplos, e so
cair pra exploracao normal quando o mapa nao tiver o que for preciso. Se o
repo ja usa `@AGENTS.md` ou tem um `CLAUDE.md` extenso de outro assunto
(regras de negocio, arquitetura), a secao entra como adicao, nunca
substituindo o conteudo existente. Ja aplicada nos 11 repositorios conectados
em 21/10/2026 — replicar em qualquer repositorio novo que entrar na familia.
