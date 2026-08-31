# Snake de Contribuições no Perfil GitHub Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Adicionar a animação de "cobrinha" comendo o gráfico de contribuições ao README do perfil GitHub `ribeirou/ribeirou`, gerada automaticamente via GitHub Actions.

**Architecture:** Uma GitHub Action (`Platane/snk`) roda no push, agendada e sob demanda, gera 2 SVGs (tema claro e escuro) e os publica numa branch órfã `output` do próprio repo. O `README.md` referencia essas URLs raw via um bloco `<picture>` que troca automaticamente conforme o tema claro/escuro do visitante, inserido logo após o banner/typing existentes e antes dos cards de stats.

**Tech Stack:** GitHub Actions, [Platane/snk](https://github.com/Platane/snk) (gera o SVG), `crazy-max/ghaction-github-pages` (publica na branch `output`) — ambas actions públicas de terceiros amplamente usadas, sem custo, sem dependência nova no projeto (não é um app Node, é um repo de perfil só com README + Action).

**Spec:** nenhuma — tarefa bounded, design aprovado em chat (sem spec formal, per superpowers:brainstorming).

## Global Constraints

- Repo: `ribeirou/ribeirou` (público, é o repo especial de perfil do GitHub), local em `C:\Users\ribass\projects\ribeirou-profile`, branch `master`.
- Sem bio pessoal / links no README ainda (usuário vai passar depois) — não adicionar texto pessoal nesta tarefa.
- A cobrinha entra JUNTO com os cards de stats/streak/linguagens já existentes no README — não remove nada que já está lá.
- Nenhuma dependência nova no projeto em si (o projeto não tem `package.json`, é só README + Actions).
- `gh` CLI já autenticado (conta ribeirou, escopo repo) — usar pra disparar/checar a Action.

---

### Task 1: Workflow da GitHub Action

**Files:**
- Create: `.github/workflows/snake.yml`

**Interfaces:**
- Produces: ao rodar, a Action publica 2 arquivos SVG (`github-contribution-grid-snake.svg` e `github-contribution-grid-snake-dark.svg`) na branch `output` do repo, acessíveis via `https://raw.githubusercontent.com/ribeirou/ribeirou/output/<nome-do-arquivo>`. Consumido pela Task 2 (README) e verificado na Task 3.

- [ ] **Step 1: Criar o workflow**

```yaml
name: generate snake animation

on:
  schedule:
    - cron: "0 0 * * *"
  push:
    branches:
      - master
  workflow_dispatch:

jobs:
  generate:
    permissions:
      contents: write
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: generate github-contribution-grid-snake.svg
        uses: Platane/snk@v3
        with:
          github_user_name: ${{ github.repository_owner }}
          outputs: |
            dist/github-contribution-grid-snake.svg
            dist/github-contribution-grid-snake-dark.svg?palette=github-dark

      - name: push github-contribution-grid-snake.svg to the output branch
        uses: crazy-max/ghaction-github-pages@v4
        with:
          target_branch: output
          build_dir: dist
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

- [ ] **Step 2: Commit e push**

```bash
git add .github/workflows/snake.yml
git commit -m "ci: gera animacao de cobrinha do grafico de contribuicoes"
git push
```

- [ ] **Step 3: Disparar a Action manualmente e conferir que rodou**

```bash
gh workflow run "generate snake animation" --repo ribeirou/ribeirou
```

Esperar ~30-60s, depois:

```bash
gh run list --repo ribeirou/ribeirou --workflow "generate snake animation" --limit 1
```

Esperado: status `completed` / conclusion `success`. Se `in_progress`, esperar mais e checar de novo (`gh run watch --repo ribeirou/ribeirou` também funciona pra acompanhar ao vivo).

- [ ] **Step 4: Confirmar que a branch `output` existe com os 2 SVGs**

```bash
gh api repos/ribeirou/ribeirou/contents/github-contribution-grid-snake.svg?ref=output --jq '.name, .size'
gh api repos/ribeirou/ribeirou/contents/github-contribution-grid-snake-dark.svg?ref=output --jq '.name, .size'
```

Esperado: os dois comandos retornam o nome do arquivo e um tamanho em bytes maior que 0 (não erro 404).

---

### Task 2: Inserir a cobrinha no README

**Files:**
- Modify: `README.md`

**Interfaces:**
- Consumes: as duas URLs SVG publicadas pela Task 1 (`https://raw.githubusercontent.com/ribeirou/ribeirou/output/github-contribution-grid-snake.svg` e a variante `-dark.svg`).

- [ ] **Step 1: Ler o `README.md` atual e localizar o ponto de inserção**

O bloco a inserir vai logo depois desta linha (o typing SVG), antes do `<br />` que precede os cards de stats:

```html
<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=22&pause=1000&color=2563EB&center=true&vCenter=true&width=460&lines=building+small%2C+useful+things;React+%7C+Vite+%7C+Tailwind;always+shipping+something" />

<br />
```

- [ ] **Step 2: Inserir o bloco da cobrinha**

Trocar o trecho acima por:

```html
<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=22&pause=1000&color=2563EB&center=true&vCenter=true&width=460&lines=building+small%2C+useful+things;React+%7C+Vite+%7C+Tailwind;always+shipping+something" />

<br /><br />

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/ribeirou/ribeirou/output/github-contribution-grid-snake-dark.svg" />
  <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/ribeirou/ribeirou/output/github-contribution-grid-snake.svg" />
  <img alt="snake eating my contribution graph" src="https://raw.githubusercontent.com/ribeirou/ribeirou/output/github-contribution-grid-snake.svg" />
</picture>

<br />
```

(mantém tudo que já existe depois disso — cards de stats, streak, linguagens, footer — sem tocar naquilo)

- [ ] **Step 3: Commit e push**

```bash
git add README.md
git commit -m "docs: adiciona animacao de cobrinha do grafico de contribuicoes"
git push
```

- [ ] **Step 4: Verificar que as URLs respondem**

```bash
curl -s -o /dev/null -w "%{http_code}\n" https://raw.githubusercontent.com/ribeirou/ribeirou/output/github-contribution-grid-snake.svg
curl -s -o /dev/null -w "%{http_code}\n" https://raw.githubusercontent.com/ribeirou/ribeirou/output/github-contribution-grid-snake-dark.svg
```

Esperado: `200` nas duas.

- [ ] **Step 5: Conferir visualmente**

Abrir `https://github.com/ribeirou` no navegador (via Claude in Chrome, já logado) e confirmar que a cobrinha aparece renderizada logo abaixo do banner/typing, acima dos cards de stats, sem quebrar o layout.

---

## Verificação final

Depois das 2 tasks: `https://github.com/ribeirou` mostra banner → typing → cobrinha animada → cards de stats/streak/linguagens → footer, nessa ordem, sem nada quebrado. A Action já está agendada pra rodar 1x/dia sozinha daqui pra frente, então a cobrinha se mantém atualizada sem precisar disparar manualmente de novo.
