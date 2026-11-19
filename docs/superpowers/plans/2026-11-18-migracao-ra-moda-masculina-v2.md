# Migração do RA Moda Masculina para a arquitetura loja-online — Plano de Execução

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Trocar a base de código do RA Moda Masculina (arquitetura antiga) pela arquitetura completa do loja-online — igual ao RA Modas, com Financeiro e Impressão incluídos — reaproveitando o projeto Firebase `salao-beleza-69163` e os 6 produtos/5 categorias/1 cupom já cadastrados (fotos e dados reais preservados).

**Architecture:** Site estático sem build (HTML/CSS/JS puro + Firebase Firestore/Auth), igual ao padrão do workspace. A pasta `ra-moda-masculina` recebe a árvore de arquivos completa do `loja-online` (não alterado, só copiado), com os valores de Firebase do próprio projeto. Substituição direta, sem cliente real dependendo de uptime.

**Tech Stack:** HTML/CSS/JS (sem framework, sem build), Firebase Firestore + Authentication (SDK compat 10.12.2).

**Spec:** `docs/superpowers/specs/2026-11-18-migracao-ra-moda-masculina-v2-design.md`

## Global Constraints

- Projeto Firebase a reaproveitar: `salao-beleza-69163` (não criar projeto novo).
- `loja-online` (pasta irmã) não deve ser alterado — é só fonte para copiar.
- E-mail administrador único: `admin@gmail.com` — cadastrar em **Authentication → Users** do projeto `salao-beleza-69163` antes de publicar as regras novas.
- Dados atuais a reaproveitar (não apagar): 6 `produtos`, 5 `categorias`, 1 `cupom` (`DESCONTO10`). `tiposVariacao` (2 docs: cor, tamanho) é removida após a migração.
- Copiar a árvore de arquivos **completa** do `loja-online`, SEM excluir `admin-financeiro.js` nem `admin-impressao.js` (diferente do RA Suplementos — aqui é "igual ao RA Modas").
- Repositório Git do RA Moda Masculina (`origin` → `github.com/cristianorrm-boop/ra-moda-masculina`, branch `main`) continua o mesmo.
- `configuracoes/geral` não existe ainda — semear com os valores reais hoje hardcoded em `CONFIG_PADRAO` do `script.js` antigo (não com o padrão genérico do loja-online).

## Review Focus

- **`A.configMudou` não definido** → como esta migração NÃO exclui `admin-impressao.js` (ele define `A.configMudou`), esse bug específico do RA Suplementos não se repete aqui — mas confirmar depois de copiar que `admin-financeiro.js`/`admin-impressao.js` estão mesmo incluídos em `admin.html`, senão o mesmo bug volta.
- **Produto com duas dimensões de variação** (Camiseta Polo Levi's: Cor × Tamanho) → sem achatar em uma lista só, o produto perde metade das opções ou quebra o formulário de variantes do admin. Task 3 cobre isso explicitamente com a lista das 6 combinações.
- **Ordem das categorias perdida** → sem portar o campo `ordem` (customização feita no RA Suplementos), a vitrine mostraria as categorias em ordem arbitrária do Firestore, perdendo a curadoria já definida (calças, blusas, shorts, calçados, acessórios). Task 1b porta a mesma customização de `admin.js`/`store.js` feita no RA Suplementos.
- **`configuracoes/geral` inexistente** → sem semear esse documento, a loja abriria com nome/cor/WhatsApp genéricos do `loja-online` em vez dos reais da RA Moda Masculina. Task 3 cria esse documento com os valores reais.
- **Cupom com campo `valorMinimo` não renomeado para `minimo`** → cupom trata todo pedido como sem mínimo (não é bug grave aqui, já que o valor é 0, mas o campo precisa existir com o nome certo). Task 3 cobre o mapeamento.
- **`firebase-config.js` no formato antigo** (`const firebaseConfig`) → `js/db.js` não encontra a configuração. Task 2 cobre com verificação explícita.

---

## Task 1: Substituir a árvore de arquivos pela do loja-online (completa)

**Files:**
- Remove: `index.html`, `admin.html`, `script.js`, `style.css`, `manifest.json`, `sw.js`, `regras.md` (versões antigas)
- Copy (de `../loja-online/` para `./`): `index.html`, `admin.html`, `acompanhar.html`, `manifest.webmanifest`, `manifest-admin.webmanifest`, `sw.js`, `regras.md`, `css/style.css`, `css/admin.css`, `css/splash.css`, `js/db.js`, `js/store.js`, `js/acompanhar.js`, `js/admin.js`, `js/admin-config.js`, `js/admin-estoque.js`, `js/admin-clientes.js`, `js/admin-pdv.js`, `js/admin-financeiro.js`, `js/admin-impressao.js`, `js/admin-relatorios.js`, `js/admin-camera.js`, `js/pwa.js`, `js/splash.js`, `js/vendor/qrcode.js`, `icons/icon-180.png`, `icons/icon-192.png`, `icons/icon-512.png`, `icons/icon-maskable-512.png`
- Keep untouched: `.git/`, `.gitignore`, `.vscode/`, `assets/` (fotos reais dos 6 produtos + logo), `docs/superpowers/`

**Interfaces:**
- Produces: árvore de arquivos do RA Moda Masculina idêntica à do loja-online (completa), pronta para a Task 2 (config Firebase) e Task 1b (customização de ordem).

- [ ] **Passo 1: Listar o conteúdo atual de `ra-moda-masculina/` (raiz) para confirmar os nomes exatos dos arquivos antigos antes de remover**

- [ ] **Passo 2: Remover os arquivos antigos da vitrine/admin listados acima**

- [ ] **Passo 3: Copiar a árvore de arquivos completa do loja-online listada acima para dentro de `ra-moda-masculina/`, preservando a estrutura de pastas**

- [ ] **Passo 4: Conferir que `admin.html` (copiado) inclui as tags `<script>` de `admin-financeiro.js` e `admin-impressao.js`, e que nenhum botão/aba foi removido**

- [ ] **Passo 5: Conferir que todo caminho de `imagem`/`src` referenciado nos arquivos HTML copiados aponta pra um arquivo que existe de fato em `ra-moda-masculina/`**

Rodar um script Node que extrai todo `href=`/`src=` relativo dos HTMLs e confere com `fs.existsSync`.
Esperado: 0 arquivos faltando.

- [ ] **Passo 6: Commit**

```bash
git add -A
git commit -m "Troca a base de código pela arquitetura loja-online completa"
```

---

## Task 1b: Portar a customização de ordem manual de categorias (RA Suplementos → aqui)

**Files:**
- Modify: `js/admin.js` (comparador `porOrdem`, coluna/campo "Ordem" no formulário de categoria)
- Modify: `js/store.js` (comparador `porOrdemCat`, aplicado em `raizes()`/`filhosDe()`)

**Interfaces:**
- Consumes: campo `ordem` já presente nos 5 documentos de `categorias` (Task 1 copiou o código; os dados já têm o campo).
- Produces: categorias ordenadas por `ordem` (crescente) tanto no admin quanto na vitrine, com campo editável no formulário de categoria.

- [ ] **Passo 1: Em `js/admin.js`, adicionar o comparador `porOrdem` e aplicá-lo na listagem de categorias (`telas.categorias`), e adicionar o campo numérico "Ordem de exibição" no formulário de categoria (`formCategoria`), igual ao feito no RA Suplementos**

- [ ] **Passo 2: Em `js/store.js`, adicionar o comparador `porOrdemCat` e aplicá-lo em `raizes()`/`filhosDe()`**

- [ ] **Passo 3: Verificar a sintaxe dos dois arquivos**

Rodar `node --check js/admin.js` e `node --check js/store.js`.
Esperado: sem erro.

- [ ] **Passo 4: Commit**

```bash
git add js/admin.js js/store.js
git commit -m "Porta ordenação manual de categorias (campo ordem) do RA Suplementos"
```

---

## Task 2: Configurar o Firebase do RA Moda Masculina no formato novo

**Files:**
- Modify: `firebase-config.js`

**Interfaces:**
- Produces: `window.firebaseConfig` populado.

- [ ] **Passo 1: Reescrever `firebase-config.js` no formato que `js/db.js` espera**

```js
window.firebaseConfig = {
  apiKey: "AIzaSyC77__9NFsHR9g4ADHCym-3rtzPgj0ejjQ",
  authDomain: "salao-beleza-69163.firebaseapp.com",
  projectId: "salao-beleza-69163",
  storageBucket: "salao-beleza-69163.firebasestorage.app",
  messagingSenderId: "291853463898",
  appId: "1:291853463898:web:7af92b2e07fbe6d69f0363",
};
```

(valores copiados do `firebase-config.js` antigo do próprio RA Moda Masculina)

- [ ] **Passo 2: Verificar que nenhum outro arquivo do projeto ainda referencia `const firebaseConfig` (formato antigo)**

Rodar `grep -rn "const firebaseConfig" .` a partir da raiz do projeto.
Esperado: nenhuma ocorrência.

- [ ] **Passo 3: Commit**

```bash
git add firebase-config.js
git commit -m "Configura o Firebase do RA Moda Masculina no formato do loja-online"
```

---

## Task 3: Utilitário único de migração de dados

**Files:**
- Create: `migrar-dados.html` (mesmo padrão já usado no RA Suplementos: login por e-mail/senha, botão único, log na tela)

**Interfaces:**
- Consumes: `window.firebaseConfig` (Task 2), SDKs Firebase (app-compat, firestore-compat, auth-compat) via CDN, versão 10.12.2.
- Produces: `produtos` (6, com campos novos e variantes da camiseta combinadas), `categorias` (5, ordem preservada), `cupons` (1, campo `minimo`), `configuracoes/geral` (criada com os valores reais do `script.js` antigo). `tiposVariacao` removida.

- [ ] **Passo 1: Criar `migrar-dados.html` com login (e-mail/senha) e um botão único "Migrar dados"**

Ao clicar, em sequência:

1. **Categorias** — reescrever cada uma mantendo `nome`, `ordem`, `icone` como já estão (sem correção de nome necessária aqui, diferente do RA Suplementos).
2. **Produtos** — para cada um dos 6 produtos atuais, reescrever mantendo `nome`, `categoria`, `preco`, `imagem`, `descricao`, `destaque`, `ativo`, descartando `imagem2`, e adicionando: `estoque: 0`, `codigo: null`, `estoqueMinimo: null`, `custo: null`, `precoAntigo: null`, `variantes`:
   - Para o produto "Camiseta Polo Levi's" (id `1`): `variantes: [{id:"v1",nome:"Azul - P",estoque:0},{id:"v2",nome:"Azul - M",estoque:0},{id:"v3",nome:"Azul - G",estoque:0},{id:"v4",nome:"Preto - P",estoque:0},{id:"v5",nome:"Preto - M",estoque:0},{id:"v6",nome:"Preto - G",estoque:0}]`
   - Para os outros 5 produtos: `variantes: []`
3. **Cupons** — reescrever `DESCONTO10` com `{ codigo, tipo, valor, minimo: valorMinimo ?? 0, ativo }`.
4. **Configurações** — criar `configuracoes/geral` com `{ ...mesclarConfig({}), nomeLoja: "RA Moda Masculina", whatsapp: "558182362638", corPrimaria: "#0EA5A6", corSecundaria: "#0F172A", corDestaque: "#C5A049", retiradaDias: [0,1,2,3,4,5,6], retiradaHoraInicio: "08:00", retiradaHoraFim: "18:00", retiradaIntervalo: 60 }` — só se o documento ainda não existir (não sobrescrever se a usuária já tiver criado/editado algo).
5. **tiposVariacao** — apagar todos os documentos da coleção.
6. Mostrar no log quantos itens foram migrados em cada etapa.

- [ ] **Passo 2: Verificar a sintaxe do arquivo**

Rodar o script Node de checagem de sintaxe (`new Function` sobre o conteúdo de cada `<script>`).
Esperado: sem erro.

- [ ] **Passo 3: Commit**

```bash
git add migrar-dados.html
git commit -m "Adiciona utilitário de migração de produtos/categorias/cupons/config"
```

(A execução de fato — login e clique no botão — é feita pela usuária, autenticada. Conferir o resultado via leitura pública do Firestore na Task 5.)

---

## Task 4: Conferir regras do Firestore, e-mail administrador e ferramenta de importação

**Files:**
- Modify: `regras.md` (copiado na Task 1 — já vem com `emailsAdmin() → ['admin@gmail.com']`, só confirmar)
- Create: `importar-produtos.html` (mesmo padrão do RA Suplementos, lista `PRODUTOS` vazia)

**Interfaces:**
- Nenhuma — conferência/ajuste de configuração + ferramenta auxiliar.

- [ ] **Passo 1: Conferir que `regras.md` (copiado) já tem `emailsAdmin()` retornando `['admin@gmail.com']`**

- [ ] **Passo 2: Lembrete para a usuária (fora do código): cadastrar `admin@gmail.com` em Authentication → Users do projeto `salao-beleza-69163`, antes de publicar estas regras**

- [ ] **Passo 3: Criar `importar-produtos.html` com login, campo para colar produtos novos e botão "Importar", lista `PRODUTOS` vazia**

- [ ] **Passo 4: Verificar a sintaxe do arquivo**

Esperado: sem erro.

- [ ] **Passo 5: Commit**

```bash
git add importar-produtos.html
git commit -m "Adiciona ferramenta de importação em massa de produtos"
```

---

## Task 5: Publicar regras, migrar dados e verificar localmente

**Files:** nenhum arquivo novo — tarefa de verificação ponta a ponta.

**Interfaces:**
- Consumes: tudo das Tasks 1-4.

- [ ] **Passo 1: Pedir para a usuária cadastrar `admin@gmail.com` em Authentication → Users do projeto `salao-beleza-69163`, e publicar o conteúdo de `regras.md` no Console do Firebase**

- [ ] **Passo 2: Pedir para a usuária rodar `migrar-dados.html` localmente (logada), clicando no botão único**

- [ ] **Passo 3: Conferir via leitura pública do Firestore (REST) que `categorias` tem 5 itens com `ordem` preservada, `cupons` tem 1 item com campo `minimo`, `produtos` tem 6 itens com os campos novos (a camiseta com 6 variantes), `configuracoes/geral` existe com os valores reais, e `tiposVariacao` está vazia/removida**

- [ ] **Passo 4: Servir a pasta localmente e abrir `admin.html`; fazer login com `admin@gmail.com`**

Esperado: entra no painel, Dashboard mostra os 6 produtos, Estoque mostra os 6 produtos (camiseta com 6 variantes) com estoque 0, Categorias mostra as 5 na ordem certa com campo "Ordem" editável, Cupons mostra `DESCONTO10` com mínimo 0, PDV exige abertura de caixa, Financeiro (Contas a pagar/receber) abre sem erro, Configurações abre e salva sem travar o botão.

- [ ] **Passo 5: Abrir `index.html` localmente**

Esperado: carrega a vitrine com as 5 categorias na ordem certa, os 6 produtos com fotos reais, cores teal/azul-marinho/dourado aplicadas, nome "RA Moda Masculina" e WhatsApp corretos, sem erro no console (F12).

---

## Task 6: Deploy final

**Files:**
- Modify: `sw.js` (bump de `VERSAO`)

**Interfaces:**
- Consumes: sistema já funcionando localmente (Task 5).

- [ ] **Passo 1: Aumentar a constante `VERSAO` em `sw.js`**

- [ ] **Passo 2: Commit e push (confirmar com a usuária antes de dar push, mesmo padrão já seguido no RA Suplementos)**

```bash
git add sw.js
git commit -m "Prepara deploy: bump de versão do service worker"
git push
```
