# Sorterama - Plano de Testes do MVP

Ultima revisao: 09/07/2026.

Este documento define o roteiro minimo de testes manuais e automatizados para homologacao do MVP.

## Objetivo

Validar os fluxos criticos da plataforma:

- onboarding e autenticacao;
- carrinho e checkout Pix;
- webhooks e idempotencia;
- publicacao de concursos, produtos e boloes;
- cadastro/importacao de jogos/participacoes em loterias oficiais;
- resultado oficial, conferencia, premiacao e liberacao para resgate;
- area logada do cliente;
- backoffice financeiro/fiscal;
- comunicacoes transacionais;
- linguagem de compliance.

## Testes Automatizados

Comandos recomendados:

```powershell
dotnet build Sorterama.sln
dotnet test Sorterama.sln --no-build
```

Coberturas minimas atuais esperadas:

- validacao de jogos por modalidade;
- rejeicao de dezenas repetidas;
- conferencia de jogos contra resultado oficial;
- importacao atomica de jogos por arquivo;
- registro financeiro de premiacao por resultado;
- calculo por cotas pagas/elegiveis;
- liberacao administrativa de premiacao para resgate;
- bloqueio de liberacao sem cotas elegiveis.

Novas funcionalidades devem ter pelo menos:

- um teste positivo;
- um teste negativo ou nulo;
- validacao de idempotencia quando houver escrita sensivel.

## Roteiro Manual - Publicacao de Boloes

### Geracao em lote

1. Acessar Backoffice > Produtos e boloes > Geracao.
2. Revisar modelos ativos de Mega-Sena, Lotofacil e Quina.
3. Executar "Gerar Boloes do Proximo Mes".
4. Confirmar que concursos e boloes futuros foram criados.
5. Reexecutar a acao e confirmar que nao houve duplicidade.
6. Validar que os itens seguem em fluxo operacional interno ate cadastro/importacao dos jogos.

Resultado esperado:

- concursos futuros criados conforme calendario configurado;
- bolao mensal da Mega-Sena criado quando modelo estiver ativo;
- reexecucao idempotente;
- valores, cotas e observacoes editaveis no backoffice.

### Cadastro manual de jogos

1. Abrir o bolao no backoffice.
2. Cadastrar dezenas validas conforme modalidade.
3. Salvar.
4. Repetir com dezenas fora da faixa.
5. Repetir com dezenas repetidas.

Resultado esperado:

- jogo valido salvo;
- jogo invalido rejeitado com mensagem clara;
- nenhuma conferencia deve depender de texto descritivo do bolao.

### Importacao de jogos

1. Baixar o modelo de importacao.
2. Preencher arquivo com linhas validas.
3. Importar.
4. Confirmar total de linhas, validas e importadas.
5. Importar arquivo com ao menos uma linha invalida.

Resultado esperado:

- arquivo valido importado;
- arquivo invalido rejeitado integralmente;
- erros exibidos por linha/campo;
- nenhuma linha gravada quando houver erro.

## Roteiro Manual - Resultado e Premiacao

### Cadastro de resultado oficial

1. Acessar Backoffice > Contabil/Fiscal ou Produtos e boloes > Resultados, conforme menu vigente.
2. Registrar resultado oficial de concurso vencido.
3. Confirmar que a lista de pendencias deixa de exibir o concurso quando resultado for registrado.
4. Conferir que datas sao exibidas em `dd/MM/yyyy`, sem hora, nos filtros de periodo.

Resultado esperado:

- resultado oficial registrado;
- numeros validados conforme modalidade;
- concurso disponivel para conferencia dos boloes vinculados.

### Conferencia de jogos

1. Abrir detalhe do resultado.
2. Executar "Conferir boloes vinculados".
3. Confirmar status por bolao:
   - sem jogos;
   - nao premiado;
   - premiado.
4. Reexecutar a conferencia.

Resultado esperado:

- bolao sem jogos aparece como pendencia operacional;
- conferencia por jogo atualiza sem duplicar registros;
- melhor faixa, quantidade de jogos conferidos e dezenas acertadas aparecem no relatorio.

### Registro de premiacao financeira

1. Em bolao premiado, informar faixas oficiais e valores.
2. Registrar premiacao.
3. Confirmar valor total, valor por cota e cotas pagas/elegiveis.
4. Reabrir a tela e confirmar persistencia.

Resultado esperado:

- `PoolResult` contem relatorio de conferencia;
- `PoolPrize` contem valor financeiro;
- `PoolPrizeShare` contem valor por cliente/cota;
- premio fica identificado, mas ainda indisponivel para saque.

### Liberacao para resgate

1. Abrir bolao ou resultado com premiacao registrada.
2. Confirmar status "Identificado, aguardando liberacao".
3. Acionar "Liberar premiacao para resgate".
4. Acessar area logada do cliente em `Account/Prizes`.

Resultado esperado:

- status muda para "Disponivel para resgate";
- saldo entra em "Disponivel para saque";
- texto informa prazo de 5 a 10 dias uteis apos liberacao;
- reexecutar liberacao nao duplica saldo.

## Roteiro Manual - Area Logada

1. Criar usuario completo.
2. Realizar compra Pix em ambiente mock ou homologacao.
3. Confirmar pagamento por webhook.
4. Validar dashboard, pedidos e detalhes.
5. Validar `Account/Prizes` antes e depois da liberacao administrativa.
6. Validar dados bancarios/Pix antes de solicitar resgate.

Resultado esperado:

- valores em `R$` e formato brasileiro;
- horarios em BRT;
- CPF, telefone e e-mail mascarados onde aplicavel;
- saque bloqueado sem dados bancarios/Pix validos;
- mensagem de prazo exibida antes da solicitacao.

## Roteiro Manual - Compliance de Linguagem

1. Buscar pelos termos sensiveis definidos pelo compliance.
2. Validar paginas publicas, backoffice, area logada, e-mails, SMS/WhatsApp, seeds e documentos legais.
3. Confirmar que textos exibidos usam "jogos/participacoes em loterias oficiais".

Comando auxiliar:

```powershell
rg -n -i "<termos-sensiveis-definidos-pelo-compliance>" . -S -g "!**/bin/**" -g "!**/obj/**" -g "!**/wwwroot/lib/**" -g "!**/wwwroot/images/**"
```

Resultado esperado:

- nenhuma ocorrencia em texto exibido ao usuario;
- eventuais nomes tecnicos internos devem ser avaliados antes de renomear, para evitar quebra de contrato ou migracao desnecessaria.

## Roteiro Manual - Pagamento Pix e Webhook

1. Gerar pedido Pix.
2. Confirmar QR Code e copia e cola.
3. Confirmar pagamento via webhook OpenPix/Woovi.
4. Reenviar o mesmo webhook.
5. Validar status do pedido, ledger e comunicacao.

Resultado esperado:

- webhook validado por assinatura;
- processamento idempotente;
- pedido pago uma unica vez;
- falha de e-mail nao impede confirmacao de pagamento.

## Criterios de Saida da Homologacao

- build verde;
- testes automatizados verdes;
- checkout Pix funcional;
- geracao/importacao/publicacao de boloes validada;
- resultado/conferencia/premiacao/liberacao validada;
- resgate bloqueado antes da liberacao e liberado depois;
- textos sensiveis revisados;
- logs sem secrets e sem payloads sensiveis.
