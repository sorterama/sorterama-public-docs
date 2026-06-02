# Massa de Testes - Publicacao de Boloes no Backoffice

Data: abril de 2026.
Escopo: cadastro e publicacao de `BolaoProduct`, `LotteryContest` e `Pool`.

Este roteiro complementa `software-test-plan.md` com uma massa controlada, reproduzivel e limitada aos tipos:

- Mega-Sena
- Lotofacil
- Quina

## Visao Funcional

Este documento tambem funciona como uma primeira aproximacao de especificacao funcional da publicacao de boloes. A ideia e servir como base compartilhada para:

- time de desenvolvimento;
- time de testes;
- operacao de backoffice;
- refinamentos futuros de epic, feature e regras de negocio.

## Epic

**Epic:** Publicacao de ofertas de bolao na loja

**Objetivo de negocio:** permitir que a operacao interna cadastre, revise, aprove e publique ofertas vendaveis de bolao na loja publica, garantindo coerencia entre produto, concurso e pool antes da venda ao cliente final.

**Resultado esperado da epic:**

- produtos podem ser criados e preparados para venda;
- concursos podem ser cadastrados e abertos para vendas;
- pools podem ser vinculados a produto e concurso;
- o fluxo publisher/approver controla a publicacao;
- a loja publica exibe apenas ofertas coerentes e vendaveis.

## Features

### Feature 1 - Cadastro de produto de bolao

Permite criar e editar `BolaoProduct` com dados comerciais, composicao de preco e configuracao da oferta.

**Inclui:**

- produto avulso (`Single`);
- produto mensal (`RecurringMonthly`);
- textos comerciais;
- configuracao de destaque de premio;
- selecao de pools incluidos para pacote mensal.

### Feature 2 - Cadastro de concurso

Permite criar e manter `LotteryContest`, com dados oficiais e operacionais do sorteio.

**Inclui:**

- tipo de loteria;
- numero do concurso;
- data do sorteio;
- premio estimado;
- indicacao de acumulado;
- abertura do concurso para vendas.

### Feature 3 - Cadastro de bolao

Permite criar `Pool` vinculado a um produto e a um concurso, definindo a oferta operacional que sera publicada.

**Inclui:**

- titulo operacional;
- valor da cota;
- taxa administrativa;
- total de cotas;
- janela de vendas;
- competencia quando aplicavel.

### Feature 4 - Aprovacao e publicacao

Permite que usuarios com perfil apropriado aprovem e publiquem produto e bolao, respeitando a coerencia do fluxo.

**Inclui:**

- envio para aprovacao;
- aprovacao ou rejeicao com motivo;
- publicacao;
- bloqueio de transicoes invalidas.

### Feature 5 - Exibicao da oferta na loja

A loja publica passa a exibir a oferta apenas quando o conjunto produto + concurso + bolao esta em estado vendavel.

**Inclui:**

- produto publicado;
- concurso aberto para vendas;
- bolao publicado;
- janela de venda ativa;
- cotas disponiveis.

## Objetivo

Validar o fluxo operacional do backoffice para publicar ofertas vendaveis na loja publica, cobrindo dominios validos, bordas e falhas esperadas dos campos principais.

O ciclo minimo de publicacao e:

1. Publisher cria produto.
2. Publisher cria concurso.
3. Approver abre o concurso para vendas.
4. Publisher cria bolao associando produto e concurso.
5. Publisher envia produto e bolao para aprovacao.
6. Approver aprova produto e bolao.
7. Approver publica produto e bolao.
8. Loja publica lista a oferta quando produto, concurso e bolao estao coerentes.

## Atores

| Ator | Responsabilidade principal |
| --- | --- |
| `Publisher` | cadastrar produto, concurso e bolao; enviar produto e bolao para aprovacao |
| `Approver` | revisar, aprovar, rejeitar e publicar produto e bolao; abrir concurso para vendas |
| `Administrator` | acompanhar operacao, destravar inconsistencias e auditar status |
| `Cliente final` | visualizar e comprar apenas ofertas publicadas e vendaveis |

## Fluxograma da Publicacao

<pre class="mermaid">
flowchart TD
    A["Inicio"] --> B["Publisher cria produto"]
    B --> C{"Tipo de oferta"}
    C -->|Single| D["Configura produto avulso"]
    C -->|RecurringMonthly| E["Configura produto mensal"]
    D --> F["Salvar produto em Draft"]
    E --> F
    F --> G["Publisher cria concurso"]
    G --> H["Salvar concurso em Draft"]
    H --> I["Approver abre concurso para vendas"]
    I --> J["Publisher cria bolao"]
    J --> K["Vincular produto e concurso"]
    K --> L["Preencher preco, cotas e janela de vendas"]
    L --> M["Salvar bolao em Draft"]
    M --> N{"Produto mensal?"}
    N -->|Sim| O["Selecionar pools incluidos no produto"]
    N -->|Nao| P["Seguir fluxo"]
    O --> P
    P --> Q["Publisher envia produto para aprovacao"]
    Q --> R["Publisher envia bolao para aprovacao"]
    R --> S["Approver revisa produto"]
    S --> T{"Produto aprovado?"}
    T -->|Nao| U["Rejeitar produto com motivo"]
    T -->|Sim| V["Aprovar produto"]
    V --> W["Publicar produto"]
    W --> X["Approver revisa bolao"]
    X --> Y{"Bolao aprovado?"}
    Y -->|Nao| Z["Rejeitar bolao com motivo"]
    Y -->|Sim| AA["Aprovar bolao"]
    AA --> AB["Publicar bolao"]
    AB --> AC{"Checklist final OK?"}
    AC -->|Nao| AD["Corrigir cadastro ou status"]
    AC -->|Sim| AE["Oferta exibida na loja"]
    AD --> X
</pre>

<script type="module">
  import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";

  mermaid.initialize({
    startOnLoad: true,
    securityLevel: "strict",
    theme: "default"
  });
</script>

## Regras de Negocio da Publicacao

Para a oferta aparecer corretamente na loja, o conjunto precisa estar coerente:

- `BolaoProduct` em `Published`;
- `LotteryContest` em `OpenForSales`;
- `Pool` em `Published`;
- janela de vendas ativa;
- cotas disponiveis;
- relacionamento valido entre produto, concurso e bolao.

No caso de pacote mensal:

- o produto deve estar em `RecurringMonthly`;
- os pools precisam estar explicitamente associados ao produto;
- a cobranca e unica para o pacote;
- a distribuicao operacional depende dos pools incluidos e vendaveis.

## Criterios Funcionais para Desenvolvimento e Teste

Ao evoluir esta funcionalidade, desenvolvimento e QA devem validar pelo menos:

1. transicoes de status permitidas e bloqueadas;
2. coerencia entre entidades vinculadas;
3. exibicao correta na loja publica;
4. comportamento de produto mensal versus avulso;
5. mensagens de erro e bloqueios operacionais;
6. preservacao da rastreabilidade entre produto, concurso e bolao.

## Estado Inicial Recomendado

Antes de cada rodada completa:

1. Resetar a base local/homologacao controlada.
2. Subir API para aplicar migrations e seeding.
3. Confirmar usuarios seed:
   - `admin@sorterama.com`
   - `sorteramapublisher@mailinator.com`
   - `sorteramaaprover@mailinator.com`
   - `sorteramaclient@mailinator.com`
4. Registrar a data/hora da rodada.

Para ambiente local com Docker, use:

```powershell
.\scripts\reset-test-database.ps1 -ConfirmReset -RestartApi
```

Em homologacao, prefira novo deploy com banco dedicado limpo ou execute o reset apenas em banco explicitamente separado de producao.

## Convencoes da Massa

Use um identificador de rodada no formato `QA-YYYYMMDD-HHMM`.

Exemplo:

```text
QA-20260426-1030
```

Todos os campos textuais devem incluir o identificador da rodada para facilitar busca e limpeza logica:

```text
QA-20260426-1030 Mega-Sena Avulso
```

Datas abaixo assumem que a rodada sera executada com sorteios no futuro. Se a data ja passou, manter a distancia relativa:

- venda inicia: agora menos 1 hora;
- venda encerra: sorteio menos 2 horas;
- sorteio curto: agora mais 2 dias;
- sorteio medio: agora mais 7 dias;
- sorteio longo: agora mais 15 dias.

## Dominios dos Campos

### BolaoProduct

| Campo | Dominio valido para a massa | Bordas/falhas esperadas |
| --- | --- | --- |
| `Title` | texto unico por rodada, por tipo de loteria e oferta | vazio deve falhar |
| `ShortDescription` | vazio, texto curto, texto com acento | deve aceitar nulo/vazio |
| `Description` | texto operacional com regras do bolao | deve aceitar nulo/vazio |
| `QuotaAmount` | `0.00`, valor baixo, valor usual, valor alto | negativo deve falhar |
| `AdministrationFeeAmount` | `0.00`, valor usual, valor alto | negativo deve falhar |
| `Price` | calculado pela tela como cota + taxa | nao testar manualmente como fonte de verdade |
| `OfferType` | `Single`, `RecurringMonthly` | mensal precisa de pools incluidos |
| `PrizeDisplayMode` | `UseContestOfficialData`, `UseMarketingOverride` | premio custom negativo deve falhar |
| `CustomPrizeHighlightValue` | nulo, `0.00`, valor positivo | ignorado quando modo nao e marketing |
| `CustomPrizeHighlightText` | nulo ou texto comercial curto | ignorado quando modo nao e marketing |
| `PrizeDisplayLabel` | nulo, "Premio estimado", "Premio previsto" | deve aceitar nulo |
| `ContestSummaryText` | nulo ou resumo dos concursos | deve aceitar nulo |
| `ShowAccumulatedBadge` | true/false | sem regra bloqueante |
| `DefaultQuotaCount` | nulo, `1`, `100`, `500` | zero/negativo deve falhar |
| `DefaultSalesWindowDays` | nulo, `1`, `7`, `15` | zero/negativo deve falhar |
| `PoolGenerationDay` | nulo, `1`, `15`, `28` para mensal | menor que 1 ou maior que 28 deve falhar |

### LotteryContest

| Campo | Dominio valido para a massa | Bordas/falhas esperadas |
| --- | --- | --- |
| `LotteryType` | Mega-Sena, Lotofacil, Quina | outros tipos ficam fora desta massa |
| `ContestNumber` | numero positivo e unico por loteria | zero/negativo deve falhar; duplicado deve falhar |
| `DrawDate` | futuro curto, medio e longo | default/vazio deve falhar; passado bloqueia edicao |
| `EstimatedPrize` | nulo, `0.00`, valor positivo | negativo deve falhar |
| `IsAccumulated` | true/false | sem regra bloqueante |
| `OfficialPrizeNotes` | nulo, texto curto | deve aceitar nulo |
| `ContestDisplayName` | nulo ou nome amigavel | deve aceitar nulo |
| `Notes` | nulo ou observacao interna | deve aceitar nulo |
| `ResultNumbers` | formato separado por espaco, virgula, hifen ou barra | vazio deve falhar ao publicar resultado |

### Pool

| Campo | Dominio valido para a massa | Bordas/falhas esperadas |
| --- | --- | --- |
| `LotteryContestId` | concurso Draft ou OpenForSales | zero/inexistente deve falhar; cancelado/resultado publicado deve falhar |
| `BolaoProductId` | produto Draft, PendingApproval, Approved ou Published | zero/inexistente deve falhar; arquivado/rejeitado deve falhar |
| `Title` | texto unico por rodada | vazio deve falhar |
| `QuotaPrice` | valor de cota sem taxa, `0.00` ou positivo | negativo deve falhar |
| `AdministrationFeeAmount` | `0.00` ou positivo | negativo deve falhar |
| `TotalQuotas` | `1`, `10`, `100`, `500` | zero/negativo deve falhar |
| `SalesStartAtUtc` | agora - 1 hora, agora + 1 dia | default deve falhar |
| `SalesEndAtUtc` | posterior ao inicio e anterior ao sorteio | menor/igual ao inicio deve falhar |
| `Competence` | nulo para avulso, `YYYY-MM` para mensal | formato nao validado pelo dominio atual |

## Massa Base Valida

Esta massa cobre os tres tipos permitidos e cria ofertas avulsas publicaveis.

| Caso | Tipo | Produto | Concurso | Sorteio | Cota | Taxa | Cotas | Premio | Acumulado |
| --- | --- | --- | --- | --- | ---: | ---: | ---: | ---: | --- |
| BP-001 | Mega-Sena | `{RUN} Mega-Sena Avulso` | 910001 | +2 dias 20:00 | 17.90 | 2.00 | 100 | 35000000.00 | true |
| BP-002 | Lotofacil | `{RUN} Lotofacil Avulso` | 910002 | +7 dias 20:00 | 13.90 | 2.00 | 120 | 1700000.00 | false |
| BP-003 | Quina | `{RUN} Quina Avulso` | 910003 | +15 dias 20:00 | 15.90 | 2.00 | 80 | 8000000.00 | true |

Resultado esperado:

- produto salvo em `Draft`;
- concurso salvo em `Draft`;
- concurso aberto para vendas;
- pool salvo em `Draft`;
- produto e pool passam por `PendingApproval`, `Approved` e `Published`;
- catalogo publico exibe os tres produtos;
- detalhe do produto mostra o pool publicado e disponivel.

## Massa de Pacote Mensal

Cria um produto mensal com pools incluidos explicitamente. Use os concursos da massa base ou concursos adicionais da mesma loteria.

| Caso | Produto mensal | Pools incluidos | Preco pacote | Cota contabil | Taxa adm | Competence |
| --- | --- | --- | ---: | ---: | ---: | --- |
| BP-010 | `{RUN} Clube Mensal Mega-Sena` | Mega-Sena + Lotofacil + Quina | 49.90 | 43.90 | 6.00 | ano-mes atual |

Resultado esperado:

- o pacote mensal e publicado somente depois de selecionar pools;
- na compra/processamento mensal, o cliente recebe uma participacao em cada pool incluido;
- a cobranca e unica, no valor do pacote;
- a taxa de administracao fiscal e calculada uma vez por pool conforme composicao dos itens.

## Casos de Validacao Negativa

### Produto

| Caso | Acao | Dado | Resultado esperado |
| --- | --- | --- | --- |
| BP-N001 | salvar produto | `Title` vazio | erro de titulo obrigatorio |
| BP-N002 | salvar produto | `QuotaAmount = -1.00` | erro de valor de cota |
| BP-N003 | salvar produto | `AdministrationFeeAmount = -1.00` | erro de taxa administrativa |
| BP-N004 | salvar produto | `DefaultQuotaCount = 0` | erro de quantidade padrao |
| BP-N005 | salvar produto mensal | `PoolGenerationDay = 29` | erro de dia de geracao |
| BP-N006 | editar produto publicado | alterar campos principais | acesso bloqueado ou edicao indisponivel |

### Concurso

| Caso | Acao | Dado | Resultado esperado |
| --- | --- | --- | --- |
| CT-N001 | salvar concurso | `LotteryType = Lotomania` | fora do escopo da massa; nao executar nesta rodada |
| CT-N002 | salvar concurso | `ContestNumber = 0` | erro de numero positivo |
| CT-N003 | salvar concurso | mesmo tipo e numero ja existente | erro de duplicidade |
| CT-N004 | salvar concurso | `EstimatedPrize = -1.00` | erro de premio estimado |
| CT-N005 | editar concurso | sorteio no passado | erro de edicao apos sorteio |
| CT-N006 | publicar resultado | `ResultNumbers` vazio | erro de resultado obrigatorio |

### Pool

| Caso | Acao | Dado | Resultado esperado |
| --- | --- | --- | --- |
| PL-N001 | salvar pool | sem concurso | erro de concurso obrigatorio |
| PL-N002 | salvar pool | sem produto | erro de produto obrigatorio |
| PL-N003 | salvar pool | `Title` vazio | erro de titulo obrigatorio |
| PL-N004 | salvar pool | `TotalQuotas = 0` | erro de cotas positivas |
| PL-N005 | salvar pool | fim de vendas igual/antes do inicio | erro de janela de venda |
| PL-N006 | salvar pool | mesmo produto e mesmo concurso | erro de duplicidade |
| PL-N007 | salvar pool | produto rejeitado ou arquivado | erro de associacao indisponivel |
| PL-N008 | publicar pool | sem aprovar antes | transicao bloqueada |

## Numeros de Jogos para Teste Operacional

Use estes numeros ao cadastrar jogos manuais no pool, quando o teste incluir conferencia.

| Tipo | Jogo 1 | Jogo 2 | Resultado manual |
| --- | --- | --- | --- |
| Mega-Sena | `04 12 23 33 45 56` | `01 08 19 27 39 60` | `04 12 23 33 45 56` |
| Lotofacil | `01 02 03 04 05 06 07 08 09 10 11 12 13 14 15` | `02 04 06 08 10 12 14 16 18 20 22 24 25 03 05` | `01 02 03 04 05 06 07 08 09 10 11 12 13 14 15` |
| Quina | `05 17 29 43 71` | `01 12 25 50 80` | `05 17 29 43 71` |

## Checklist de Execucao por Rodada

```text
Run ID:
Ambiente:
Data/hora:
Commit/branch:
Reset executado: sim/nao

Produto Mega-Sena ID:
Concurso Mega-Sena ID:
Pool Mega-Sena ID:

Produto Lotofacil ID:
Concurso Lotofacil ID:
Pool Lotofacil ID:

Produto Quina ID:
Concurso Quina ID:
Pool Quina ID:

Produto mensal ID:
Pools incluidos:

Resultado catalogo publico:
Resultado checkout:
Resultado publicacao:
Observacoes:
```

## Criterio de Saida

A massa e considerada aprovada quando:

- os tres produtos avulsos sao publicados;
- os tres pools ficam publicados e dentro da janela de venda;
- os concursos associados ficam `OpenForSales`;
- a loja publica exibe Mega-Sena, Lotofacil e Quina;
- os casos negativos bloqueiam os dados invalidos sem criar registros inconsistentes;
- IDs e evidencias da rodada foram registrados.
