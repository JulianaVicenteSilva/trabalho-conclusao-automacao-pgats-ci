# Trabalho de Conclusão — Automação de Testes com CI/CD

> Pós-Graduação em Automação de Testes de Software (PGATS)  
> Disciplina: Integração Contínua com GitHub Actions

---

## Sobre o Projeto

Este repositório implementa uma pipeline de **Integração Contínua (CI)** completa usando GitHub Actions, aplicada sobre um projeto de testes automatizados com **Mocha** e **Mochawesome**.

O projeto contém uma classe JavaScript (`ServicoDePagamento`) com lógica de categorização de pagamentos, coberta por 5 testes unitários que validam os comportamentos esperados.

---

## Estrutura do Projeto

```
.
├── .github/
│   └── workflows/
│       └── ci.yml              # Pipeline de CI (arquivo único com todos os gatilhos)
├── src/
│   └── servicoDePagamento.js   # Classe com a lógica de negócio
├── test/
│   └── servicoDePagamento.test.js  # Testes unitários com Mocha
├── package.json
└── README.md
```

---

## Tecnologias Utilizadas

| Ferramenta | Versão | Função |
|---|---|---|
| Node.js | 22.x | Ambiente de execução JavaScript |
| Mocha | ^11.7.6 | Framework de testes |
| Mochawesome | ^7.1.4 | Gerador de relatório HTML/JSON |
| GitHub Actions | — | Plataforma de CI/CD |

---

## A Pipeline de CI

O arquivo `.github/workflows/ci.yml` implementa **um único workflow** com **três gatilhos combinados**, contemplando os quatro níveis de CI estudados na disciplina:

### Gatilhos configurados

```yaml
on:
  push:
    branches: ["**"]       # Nível 4 — Dispara em qualquer push
  workflow_dispatch:        # Nível 1 — Disparo manual pela interface do GitHub
  schedule:
    - cron: "0 11 * * 1"   # Nível 2 — Toda segunda-feira às 08h BRT
```

### Fluxo de execução (jobs e steps)

```
push / manual / agendado
        │
        ▼
  [Checkout do código]
        │
        ▼
  [Configurar Node.js 22 + cache npm]
        │
        ▼
  [Instalar dependências — npm ci]
        │
        ▼
  [Executar testes — Mocha + Mochawesome]
        │          (continue-on-error: true)
        ▼
  [Publicar relatório — upload-artifact]
        │          (if: always())
        ▼
  [Exibir resumo na aba Summary]
        │          (if: always())
        ▼
  [Verificar falhas e finalizar job]
```

---

## Conceitos Aplicados

### Nível 1 — Disparo Manual (`workflow_dispatch`)
Permite acionar a pipeline manualmente pelo botão **"Run workflow"** na aba **Actions** do GitHub, com campo opcional para informar o motivo da execução.

### Nível 2 — Execução Agendada (`schedule`)
A pipeline executa automaticamente em três agendamentos usando sintaxe cron:
- `0 11 * * 1` → toda segunda-feira às 11h UTC (08h BRT)
- `*/30 * * * *` → a cada 30 minutos
- `0 3 17 6 *` → dia 17/06 à meia noite BRT (03h UTC)

### Nível 3 — Após outra pipeline (`workflow_run`)
Este nível pode ser implementado adicionando um segundo workflow que use `workflow_run: workflows: ['CI - Testes Automatizados']` para disparar após a conclusão desta pipeline.

### Nível 4 — Integrada ao build (`push`)
A pipeline dispara automaticamente a cada `git push` em qualquer branch, simulando um ambiente de integração contínua real onde cada alteração no código aciona a bateria de testes.

---

## Relatório de Testes

A pipeline gera automaticamente um **relatório visual HTML** com o Mochawesome:

- **Artefato publicado:** disponível na aba **Actions → [execução] → Artifacts**
- **Nome do artefato:** `relatorio-mochawesome-{número_da_execução}`
- **Retenção:** 30 dias
- **Conteúdo:** relatório HTML interativo + arquivo JSON com os dados dos testes

Além disso, a pipeline exibe um **resumo inline** na aba **Summary** com a contagem de testes passados, falhos e total.

---

## Como Executar Localmente

```bash
# Instalar dependências
npm install

# Executar os testes com relatório
npx mocha --reporter mochawesome --reporter-options reportDir=mochawesome-report,reportFilename=mochawesome,html=true,json=true "test/**/*.test.js"
```

O relatório será gerado em `mochawesome-report/mochawesome.html`.

---

## Testes Implementados

O arquivo `test/servicoDePagamento.test.js` contém **5 testes** que cobrem:

| Teste | Descrição |
|---|---|
| 1 | Realizar pagamento com código de barras, empresa e valor |
| 2 | Retornar categoria `"cara"` quando valor > 100 |
| 3 | Retornar categoria `"padrão"` quando valor = 100 |
| 4 | Retornar categoria `"padrão"` quando valor < 100 |
| 5 | Consultar o último pagamento realizado de uma lista |

---

## Evidência de Execução

Pipeline executada com sucesso — 5 testes passando, 0 falhas, relatório gerado.

A pipeline pode ser verificada acessando:  
https://github.com/JulianaVicenteSilva/trabalho-conclusao-automacao-pgats-ci/actions
