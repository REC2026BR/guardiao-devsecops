# Guardiao DevSecOps

Kit inicial de seguranca continua para repositorios GitHub e ambientes VDS.

O projeto comeca em modo seguro: analisa Pull Requests, detecta segredos,
dependencias vulneraveis e problemas conhecidos, mas nao faz deploy, altera
firewall ou modifica banco de dados automaticamente.

## Agentes da v1

- **Code Agent:** CodeQL para analise semantica do codigo.
- **Secret Agent:** Gitleaks para detectar credenciais no codigo e no historico.
- **Dependency Agent:** Dependency Review e Dependabot.
- **Infrastructure Agent:** Trivy/Checkov, habilitados conforme a infraestrutura.

## Esquadrao de QA de Negocio

Complementar aos agentes de CI acima (que olham codigo estatico), o
Esquadrao simula uso real do produto — cliques reais via Playwright
(`scripts/qa_squad/`, pilot em NuTrocas/Dr-Apto/Dr-Hiscon) e, quando
necessario, simulacoes de negocio ad-hoc contra um banco de teste isolado,
com fase de verificacao cetica antes de qualquer achado virar correcao.
Regra de registro obrigatoria — nenhuma execucao conta como concluida sem
deixar um commit verificavel: ver `policies/esquadrao-qa.md`.

## Como integrar um projeto

Copie este workflow para `.github/workflows/security.yml` do repositorio:

```yaml
name: Guardiao DevSecOps

on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read
  security-events: write
  pull-requests: write
  actions: read

jobs:
  security:
    uses: REC2026BR/guardiao-devsecops/.github/workflows/security-reusable.yml@main
    with:
      languages: '["javascript"]'
```

Para repositorio privado, o repositorio central e o repositorio que chama
precisam ser da mesma conta/organizacao do GitHub (aqui, `REC2026BR`) — ja
e o caso de toda a familia de projetos.

Para mais de uma linguagem, use por exemplo
`languages: '["javascript", "python"]'`.

Referenciamos `@main` (nao uma tag) de proposito: ainda nao existe um
processo de release aqui, entao fixar numa tag inexistente quebraria a
integracao. Quando `v1` for taggeada de verdade, os repositorios conectados
podem migrar pra ela.

Nota: a permissao `actions: read` no bloco `permissions:` acima e exigida
tanto pelo workflow chamador quanto pelo workflow reutilizavel central —
sem ela em ambos, a chamada falha com `startup_failure` antes de qualquer
job rodar (GitHub nao deixa um reusable workflow pedir mais permissao do
que o chamador concede).

## Regras de operacao

1. A branch `main` e protegida.
2. Todo trabalho acontece em branch e Pull Request.
3. O agente pode comentar e abrir issues, mas nao faz merge sozinho.
4. Nenhum agente recebe segredo de producao.
5. Acoes de infraestrutura exigem aprovacao humana.
6. Alertas criticos bloqueiam o merge ate serem tratados ou formalmente aceitos.
7. Nenhuma execucao do Esquadrao de QA de Negocio e reportada como concluida
   sem um commit de registro verificavel (ver `policies/esquadrao-qa.md`).

## Proximos passos

- Ativar CodeQL, Dependabot e Secret Scanning nas configuracoes da organizacao.
- Configurar `CODEOWNERS` e revisores obrigatorios.
- Adicionar o ambiente `staging` antes de qualquer automacao de producao.
- Integrar Trivy/Checkov quando o repositorio tiver Docker ou IaC.
- Conectar alertas do VDS a um canal de incidentes.
- Taggear `v1` quando o workflow reutilizavel estabilizar, pra repositorios
  conectados fixarem numa versao em vez de seguir `main` direto.
