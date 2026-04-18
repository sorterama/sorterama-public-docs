# Sorterama - Plano de Testes do Software

Data: abril de 2026.

Este documento unifica a estrategia de testes, o runbook manual e o roteiro ponta a ponta do MVP.

## Objetivo

Validar o Sorterama antes da homologacao ampliada e criar uma base minima de seguranca para evolucao continua.

O plano cobre:

- onboarding e login;
- loja publica;
- carrinho e checkout Pix;
- confirmacao de pagamento;
- idempotencia;
- assinatura/pacote mensal;
- backoffice operacional;
- dashboard financeiro;
- conciliacao fiscal/NFS-e;
- notificacoes;
- permissao e regressao visual basica.

## Regras de Execucao

- Executar em homologacao isolada sempre que houver validacao por terceiro.
- Nao reutilizar o mesmo e-mail em rodadas completas.
- Registrar ambiente, data/hora, usuario, passo, resultado esperado, resultado obtido e evidencia.
- Registrar IDs gerados durante o teste.
- Reprocessar webhooks somente nos cenarios de idempotencia.
- Parar a rodada se login, checkout, pagamento ou publicacao de oferta ficarem bloqueados.

## Ambientes

### Local

Usado por desenvolvimento para validacao rapida.

- API local
- Web publica local
- Backoffice local
- Banco local ou Docker
- OpenPix sandbox ou simulacao local
- Provider fiscal mock ou Focus NFe homologacao

### Homologacao

Usado para validacao por operador, QA ou stakeholder.

- URL da loja:
- URL do backoffice:
- URL da API:
- Banco separado de producao:
- OpenPix sandbox configurado:
- Focus NFe homologacao configurado:
- Logs acessiveis para suporte:

### Producao

Nao executar roteiro completo em producao. Usar apenas smoke tests controlados, com dados e usuarios previamente autorizados.

## Perfis Necessarios

| Perfil | Uso no teste |
| --- | --- |
| Cliente novo | Criar conta, comprar e acompanhar pedido |
| Cliente existente | Validar login e nova compra |
| Publisher | Criar produtos, concursos e boloes |
| Approver | Aprovar/publicar produtos e boloes |
| Support | Consultar pedidos e processar rotinas |
| Administrator | Acessar financeiro, conciliacao e relatorios |

## Dados de Teste

Registrar antes da rodada:

- Produto avulso:
- Produto mensal:
- Concurso avulso:
- Concursos do pacote mensal:
- Pool avulso:
- Pools do pacote mensal:
- Cliente de teste:
- CPF de teste:
- E-mail de teste:
- Periodo do relatorio financeiro/fiscal:

Registrar durante a rodada:

- Product ID:
- Contest ID:
- Pool ID:
- Customer ID:
- CustomerSubscription ID:
- Order ID:
- Payment ID:
- ServiceInvoice ID:
- Numero da NFS-e:

## Camadas de Teste

### Camada 1 - Checklist Manual de Release

Obrigatoria antes de cada release candidata.

1. Aplicar migrations em banco limpo.
2. Criar usuario backoffice.
3. Criar produto avulso.
4. Criar concurso.
5. Criar pool.
6. Publicar oferta.
7. Comprar pela loja publica.
8. Confirmar pagamento.
9. Validar pedido e participacao.
10. Validar ledger financeiro.
11. Validar NFS-e ou fila fiscal pendente.
12. Rodar pacote mensal.
13. Validar notificacoes criticas.
14. Validar relatorios do backoffice.

### Camada 2 - Testes Automatizados Criticos

Minimo para gate de CI/CD:

- regras de publicacao de `BolaoProduct`;
- status de `LotteryContest`;
- publicacao, cota e snapshot de `Pool`;
- criacao de checkout a partir do carrinho;
- processamento de webhook OpenPix;
- idempotencia de pagamento;
- processamento de assinatura mensal;
- criacao e emissao de `ServiceInvoice`;
- relatorio fiscal cruzando nota, pedido pago e ledger.

### Camada 3 - Integracao

Cenarios prioritarios:

1. Produto + concurso + pool publicado.
2. Carrinho + checkout + pagamento confirmado.
3. Assinatura mensal + processamento + pagamento.
4. Pagamento confirmado + ledger + NFS-e.
5. Relatorio fiscal e financeiro no backoffice.

## Cenarios Ponta a Ponta

### Cenario 1 - Onboarding e Login

Passos:

1. Abrir loja publica.
2. Iniciar cadastro.
3. Informar e-mail novo.
4. Preencher dados obrigatorios.
5. Submeter cadastro.
6. Fazer login.
7. Sair da conta.
8. Fazer login novamente.

Resultado esperado:

- cadastro conclui sem erro;
- campos obrigatorios sao validados;
- opcionais nao bloqueiam conclusao;
- login e logout funcionam.

### Cenario 2 - Publicacao de Oferta Avulsa

Passos:

1. Entrar no backoffice como `Publisher`.
2. Criar produto avulso.
3. Informar valor da cota e taxa administrativa.
4. Criar concurso aberto para vendas.
5. Criar pool vinculado ao produto e concurso.
6. Submeter produto/pool para aprovacao.
7. Entrar como `Approver`.
8. Aprovar/publicar produto e pool.

Resultado esperado:

- produto, concurso e pool sao salvos;
- componentes financeiros ficam separados;
- snapshot de publicacao fica coerente;
- oferta aparece na loja dentro da janela de venda.

### Cenario 3 - Catalogo, Carrinho e Checkout

Passos:

1. Abrir catalogo publico.
2. Localizar produto publicado.
3. Abrir detalhe.
4. Adicionar ao carrinho.
5. Conferir item, quantidade e total.
6. Prosseguir para checkout.
7. Gerar Pix.

Resultado esperado:

- produto aparece no catalogo;
- detalhe mostra preco, premio, concurso e disponibilidade;
- carrinho preserva pool correto;
- pedido fica pendente;
- pagamento Pix e criado com valor correto.

### Cenario 4 - Pagamento, Participacao e Idempotencia

Passos:

1. Confirmar pagamento via sandbox/simulador OpenPix.
2. Aguardar webhook.
3. Abrir area do cliente.
4. Validar pedido pago.
5. Validar participacao criada.
6. Validar reducao de cota.
7. Reprocessar o mesmo webhook.
8. Validar que nada duplicou.

Resultado esperado:

- pedido e pagamento ficam pagos;
- ledger fica confirmado;
- participacao e criada uma vez;
- pool perde exatamente uma cota;
- replay nao duplica ledger, participacao ou baixa de cota.

### Cenario 5 - Pacote Mensal

Regra de negocio:

- cliente compra um pacote mensal uma vez;
- preco cobrado e o preco do produto mensal;
- pacote inclui pools explicitamente selecionados;
- cliente nao paga a soma das cotas avulsas;
- apos pagamento, recebe uma participacao em cada pool incluido;
- cada pool incluido perde uma cota somente apos pagamento.

Passos:

1. Criar pelo menos dois pools publicados.
2. Criar produto `RecurringMonthly`.
3. Definir preco do pacote.
4. Selecionar pools incluidos.
5. Publicar produto.
6. Cliente ativa assinatura.
7. Backoffice executa processamento mensal.
8. Confirmar pedido com um item por pool incluido.
9. Confirmar Pix unico no valor do pacote.
10. Confirmar pagamento.
11. Validar participacoes.
12. Reexecutar processamento e webhook para validar idempotencia.

Resultado esperado:

- cobranca unica no valor do pacote;
- pedido contem os pools incluidos;
- processamento nao duplica cobranca valida;
- pagamento cria participacao em cada pool;
- quotas caem uma vez por pool.

### Cenario 6 - Backoffice de Suporte

Passos:

1. Abrir lista de pedidos.
2. Localizar pedido pendente.
3. Abrir detalhe.
4. Conferir cliente, itens, pagamento e valores.
5. Confirmar pagamento.
6. Atualizar detalhe.
7. Validar status pago e participacoes.

Resultado esperado:

- suporte localiza pedido;
- detalhe mostra composicao financeira;
- status acompanha webhook;
- contexto permite auditar cliente, pedido, pagamento e participacao.

### Cenario 7 - Dashboard Financeiro

Passos:

1. Acessar financeiro com usuario sem `Administrator`.
2. Confirmar bloqueio.
3. Acessar como `Administrator`.
4. Filtrar periodo com pedidos testados.
5. Validar total vendido.
6. Validar valor de cotas.
7. Validar taxa administrativa bruta.
8. Validar descontos/isencoes.
9. Validar taxa administrativa liquida.
10. Validar agrupamento por pool e produto.
11. Validar status de splits e divergencias.

Resultado esperado:

- apenas `Administrator` acessa;
- totais batem com ledger;
- pendentes nao inflam vendas confirmadas;
- descontos reduzem apenas taxa administrativa liquida;
- divergencias de split aparecem quando existirem.

### Cenario 8 - NFS-e e Relatorio Fiscal

Passos:

1. Confirmar pagamento de pedido com taxa administrativa.
2. Verificar criacao/garantia de `ServiceInvoice`.
3. Confirmar publicacao de `service-invoice.requested`.
4. Aguardar handler fiscal ou polling.
5. Validar status da nota.
6. Validar e-mail de nota emitida.
7. Acessar area logada do cliente.
8. Confirmar acesso ao numero/link da nota.
9. Abrir relatorio fiscal no backoffice.
10. Filtrar periodo.
11. Exportar CSV.
12. Comparar nota, pedido pago e ledger da taxa de administracao.

Resultado esperado:

- falha fiscal nao desfaz pagamento;
- nota fica pendente ou emitida com rastreabilidade;
- cliente recebe informacao suficiente por e-mail;
- backoffice mostra emitido, registrado no ledger, diferenca e pendencias;
- CSV contem dados para conferencia fiscal/contabil.

### Cenario 9 - Notificacoes

Passos:

1. Gerar compra pendente.
2. Confirmar pagamento.
3. Emitir ou simular emissao de NFS-e.
4. Validar notificacoes para:
   - compra confirmada;
   - nota fiscal emitida;
   - resultado, se implementado.

Resultado esperado:

- notificacoes sao criadas/enviadas no contexto correto;
- nao ha duplicidade para o mesmo evento;
- SMS nao e obrigatorio para NFS-e no MVP.

### Cenario 10 - Permissoes e Regressao Visual

Passos:

1. Acessar backoffice sem login.
2. Acessar backoffice com perfil cliente.
3. Acessar financeiro sem `Administrator`.
4. Abrir loja em desktop e mobile.
5. Abrir detalhe, carrinho e checkout em desktop/mobile.
6. Abrir backoffice em desktop.

Resultado esperado:

- areas restritas bloqueiam perfis indevidos;
- textos nao quebram layout;
- botoes principais ficam acessiveis;
- fluxo de compra nao depende de tela grande.

## Smoke Test de Homologacao

Antes de convidar outra pessoa para testar:

1. Abrir loja.
2. Fazer login no backoffice.
3. Criar produto/pool de teste.
4. Confirmar produto na loja.
5. Gerar checkout.
6. Confirmar cobranca Pix.
7. Simular webhook pago.
8. Confirmar pedido pago.
9. Confirmar ledger.
10. Confirmar fila fiscal ou NFS-e.
11. Abrir dashboard financeiro.
12. Abrir relatorio fiscal.

## Severidade de Defeitos

### S1 - Bloqueador

- Login nao funciona.
- Checkout nao gera pedido.
- Pagamento nao confirma pedido.
- Webhook duplica cota/participacao/ledger.
- Backoffice nao publica oferta.
- Falha fiscal bloqueia pedido pago.

### S2 - Alto

- Valor financeiro incorreto.
- Split incorreto.
- NFS-e com base fiscal incorreta.
- Relatorio fiscal divergente do ledger.
- Usuario sem permissao acessa area restrita.

### S3 - Medio

- Texto confuso.
- Layout quebrado sem impedir fluxo.
- Mensagem de erro pouco clara.
- Log insuficiente para diagnosticar uma falha nao critica.

### S4 - Baixo

- Ajuste visual pequeno.
- Melhoria de usabilidade sem impacto no fluxo principal.

## Template de Registro

```text
Ambiente:
Data/hora:
Testador:
Modulo:
Cenario:
Passo:
Resultado: Pass / Fail / Blocked
Resultado esperado:
Resultado obtido:
IDs envolvidos:
Evidencia:
Severidade:
Observacoes:
```

## Criterios de Aceite do MVP

- Onboarding de cliente novo passa.
- Produto avulso aparece na loja.
- Checkout gera pedido e Pix.
- Pagamento confirma pedido e cria participacao.
- Replay do webhook nao duplica efeitos.
- Produto mensal gera cobranca unica do pacote.
- Backoffice cria, aprova e publica produto/pool.
- Dashboard financeiro reconcilia valores do ledger.
- NFS-e fica emitida ou pendente com rastreabilidade.
- Relatorio fiscal cruza nota, pedido pago e ledger.
- Notificacoes criticas chegam ou ficam registradas.
- Nenhum defeito S1 aberto.
