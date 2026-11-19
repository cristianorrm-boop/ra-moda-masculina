# Migração do RA Moda Masculina para a arquitetura loja-online — Design

## Intenção

Trocar a base de código do RA Moda Masculina (hoje na arquitetura antiga, mesma família do RA Suplementos antes da migração dele) pela arquitetura `loja-online` V2, igual já feito no RA Modas, no Manejo Mix e no RA Suplementos. Diferente do RA Suplementos (catálogo de demonstração reduzido), a usuária pediu que este projeto saia **igual ao RA Modas**: com o conjunto completo de módulos, incluindo Financeiro (Contas a pagar/receber) e Impressão.

## Por quê agora

Depois de migrar o RA Suplementos nesta mesma conversa, a usuária trouxe a pasta `ra-moda-masculina` para o workspace e pediu para seguir com este projeto, depois o COMANDO STORE.

## Estado atual (levantado em 2026-11-18)

- Projeto Firebase a reaproveitar: `salao-beleza-69163` (nome de origem não tem relação com o nicho; é só o projeto Firebase já configurado para este site).
- `firebase-config.js` no formato antigo (`const firebaseConfig`), precisa virar `window.firebaseConfig`.
- `regras.md` atual não tem `emailsAdmin()` — qualquer usuário autenticado pode escrever. Será substituído pelo `regras.md` do `loja-online`/RA Modas (idêntico), que já tem `emailsAdmin() → ['admin@gmail.com']`.
- Dados no Firestore:
  - **6 produtos**: Camiseta Polo Levi's, Camiseta Soldier Preta, Calça Jeans, Shorts Masculino, Relógio Dourado, Sapato Casual. Fotos reais em `assets/` (preservadas).
  - **5 categorias**, nomes já corretos (sem bug de nomes trocados como houve no RA Suplementos): `calças`(ordem 1), `blusas`(ordem 2), `shorts`(ordem 3), `calçados`(ordem 4), `acessórios`(ordem 5).
  - **1 cupom**: `DESCONTO10` (10%, sem mínimo), campo `valorMinimo` → renomeia para `minimo`.
  - **2 tipos de variação globais** (`tiposVariacao`: cor, tamanho) — modelo antigo, não existe mais no `loja-online` (variações passam a ser por produto, campo `variantes`). Coleção removida depois de migrada.
  - `configuracoes/geral`: **não existe ainda** no Firestore. Os valores reais estão hardcoded em `CONFIG_PADRAO` no `script.js` antigo: `nomeLoja: "RA Moda Masculina"`, `whatsapp: "558182362638"`, `corPrimaria: "#0EA5A6"`, `corSecundaria: "#0F172A"`, `corDestaque: "#C5A049"`, `retiradaDias` todos os 7 dias, `retiradaHoraInicio: "08:00"`, `retiradaHoraFim: "18:00"`, `retiradaIntervalo: 60`. A migração semeia `configuracoes/geral` com esses valores reais (não com o padrão genérico do `loja-online`).
- **Produto com variação dupla**: "Camiseta Polo Levi's" tem hoje duas dimensões simultâneas (Cor: Azul/Preto × Tamanho: P/M/G). O `loja-online` só suporta uma lista plana de opções por produto (campo `variantes: [{id, nome, estoque}]`). **Decisão da usuária:** combinar as duas dimensões em uma lista só de 6 opções (`Azul - P`, `Azul - M`, `Azul - G`, `Preto - P`, `Preto - M`, `Preto - G`), cada uma com `estoque: 0` (ela ajusta depois pela aba Estoque).

## Decisões

1. **Reaproveitar fotos e dados reais** — este catálogo tem produtos e fotos de verdade (não é mostruário genérico como o RA Suplementos), então a migração deve preservar nome/categoria/preço/descrição/fotos exatamente como estão, só adaptando o formato.
2. **Projeto completo, igual ao RA Modas** — copiar a árvore de arquivos do `loja-online` **inteira**, sem excluir `admin-financeiro.js` nem `admin-impressao.js` (diferente do RA Suplementos). PDV com caixa obrigatório (padrão, igual RA Modas).
3. **Categorias com campo `ordem` editável** — os dados atuais já têm uma ordem deliberada (calças, blusas, shorts, calçados, acessórios) definida pela usuária no sistema antigo. Sem a customização de `ordem` (feita no RA Suplementos/Manejo Mix), a vitrine nova mostraria as categorias em ordem arbitrária do Firestore, perdendo essa curadoria. Decisão: portar a mesma customização (campo `ordem` no formulário de categoria em `admin.js`, comparador `porOrdem`/`porOrdemCat` em `admin.js`/`store.js`) para preservar a ordem já definida. Produtos continuam sem `ordem` (mesma regra do RA Suplementos).
4. **Variações do produto duplo** — combinar Cor × Tamanho em uma lista só de 6 opções, conforme decisão da usuária (ver acima). Os outros 5 produtos não têm variação (campo `variantes: []`).
5. **Campos novos dos produtos** — `estoque: 0`, `codigo: null`, `estoqueMinimo: null`, `custo: null`, `precoAntigo: null`, `variantes` (conforme item 4). Campo `imagem2` (não usado em nenhum lugar do `loja-online`) é descartado.
6. **Cupom** — `DESCONTO10` migra com `minimo: valorMinimo ?? 0`.
7. **`tiposVariacao`** — coleção do modelo antigo, removida após a migração (nada mais a lê).
8. **`configuracoes/geral`** — semeada com os valores reais hoje hardcoded no `script.js` antigo (nome da loja, WhatsApp, cores, horário/dias de retirada), não com o padrão genérico do `loja-online`.
9. **E-mail administrador** — `admin@gmail.com`, mesmo padrão já usado no RA Suplementos. A usuária precisa cadastrar esse e-mail em Authentication → Users do projeto `salao-beleza-69163` antes de publicar as regras novas.
10. **Ferramenta de importação em massa** (`importar-produtos.html`) — criada com lista vazia, pronta para a usuária pedir produtos novos depois, igual ao padrão do RA Suplementos/RA Modas.
11. **Sem corte cuidadoso de horário** — é site ainda não publicado oficialmente (sem cliente real dependendo de uptime agora), substituição direta como nos projetos anteriores.

## Não-objetivos

- Não alterar a pasta `loja-online` (só copiar/adaptar).
- Não criar projeto Firebase novo.
- Não redesenhar o catálogo (cores/paleta da loja ficam como já estão definidas: teal/azul-marinho/dourado).

## O que fica a cargo de quem for implementar

- Nomes exatos de variáveis/arquivos seguem o padrão já estabelecido no RA Suplementos (mesmo cabeçalho de script, mesmo estilo de log no `migrar-dados.html`).

## Critério de sucesso

- Painel admin abre, Dashboard mostra 6 produtos, Estoque mostra os 6 com estoque 0 (exceto variantes da camiseta, também 0), Categorias mostra as 5 com a ordem preservada (calças, blusas, shorts, calçados, acessórios) e campo de ordem editável, Cupons mostra `DESCONTO10` com mínimo 0, Financeiro (Contas a pagar/receber) acessível e funcional, Configurações abre sem erro e salva sem travar o botão.
- Vitrine (`index.html`) abre com as 5 categorias na ordem certa, 6 produtos com fotos reais, cores teal/azul-marinho/dourado aplicadas, nome da loja e WhatsApp corretos, sem erro no console.
