# Sorterama - Fluxos de Publicacao no Backoffice

Ultima revisao: 09/07/2026.

Este documento descreve o fluxo operacional para criar, revisar, publicar e conferir boloes no backoffice.

## Principios

- A geracao em lote cria estrutura operacional, nao publica automaticamente tudo na loja.
- Um bolao deve ter jogos/participacoes em loterias oficiais cadastrados ou importados antes da publicacao.
- A importacao de jogos deve ser atomica: se houver erro, nada e gravado.
- Resultado, conferencia, premiacao financeira e liberacao de resgate sao etapas separadas.
- Textos exibidos devem usar "jogos/participacoes em loterias oficiais".

## Fluxo 1 - Publicacao Manual de Bolao por Concurso

1. Criar ou revisar o concurso.
2. Abrir o concurso para vendas, quando aplicavel.
3. Criar ou revisar o produto comercial.
4. Criar bolao vinculado ao produto e ao concurso.
5. Cadastrar jogos manualmente ou importar arquivo.
6. Revisar:
   - titulo;
   - modalidade;
   - concurso;
   - data do sorteio;
   - valor da cota;
   - taxa de administracao;
   - quantidade de cotas;
   - janela de vendas;
   - observacoes.
7. Enviar para aprovacao.
8. Approver aprova.
9. Publicar.
10. Validar a oferta na loja publica.

## Fluxo 2 - Geracao em Lote do Proximo Mes

1. Acessar a area de geracao de boloes.
2. Revisar modelos:
   - Mega-Sena: terca, quinta e sabado;
   - Lotofacil: segunda a sabado;
   - Quina: segunda a sabado;
   - mensal Mega-Sena, quando ativo.
3. Executar "Gerar Boloes do Proximo Mes".
4. Conferir resumo da geracao.
5. Reexecutar a geracao para validar idempotencia.
6. Abrir cada bolao gerado e revisar os dados.
7. Cadastrar/importar jogos.
8. Enviar para aprovacao e publicar apenas quando operacionalmente completo.

## Fluxo 3 - Importacao de Jogos

### Modelo CSV

```text
LotteryType;GameNumber;Numbers;Source;ProviderBetId;ReceiptUrl;Notes
Mega-Sena;1;"01,05,12,23,44,60";Manual;;;
Mega-Sena;2;"02,08,19,31,45,59";Manual;;;
Lotofacil;;"01,02,03,04,05,06,07,08,09,10,11,12,13,14,15";Manual;;;
Quina;;"01,05,12,23,44";Manual;;;
```

### Regras

- `LotteryType`: obrigatorio.
- `LotteryType`: deve bater com enum/modalidade existente.
- `Numbers`: obrigatorio.
- `Numbers`: sempre separado por virgula.
- Mega-Sena: minimo 6 dezenas, faixa 01-60.
- Quina: minimo 5 dezenas, faixa 01-80.
- Lotofacil: minimo 15 dezenas, faixa 01-25.
- Nao permitir dezenas repetidas no mesmo jogo.
- `ReceiptUrl`: opcional.
- `GameNumber`: pode vir vazio; se informado, nao pode duplicar.

### Resultado esperado

- arquivo valido importa todos os jogos;
- arquivo com erro rejeita todas as linhas;
- tela exibe total de linhas, total valido e erros por linha/campo.

## Fluxo 4 - Resultado Oficial e Conferencia

1. Acessar a lista de concursos vencidos pendentes.
2. Selecionar o concurso.
3. Informar dezenas oficiais conforme modalidade.
4. Salvar resultado oficial.
5. Executar conferencia dos boloes vinculados.
6. Revisar relatorio:
   - jogos conferidos;
   - melhor faixa;
   - premiado/nao premiado/sem jogos;
   - dezenas acertadas;
   - pendencias operacionais.
7. Reexecutar conferencia para validar idempotencia.

## Fluxo 5 - Registro de Premiacao

1. Em bolao premiado, informar faixas oficiais.
2. Informar quantidade de jogos premiados por faixa.
3. Informar valor por jogo premiado.
4. Registrar premiacao.
5. Confirmar:
   - valor total;
   - valor por cota;
   - cotas pagas/elegiveis;
   - compartilhamentos por cliente/cota.

Separacao conceitual:

- `PoolResult`: relatorio/conferencia.
- `PoolPrize`: valor financeiro do premio.
- `PoolPrizeShare`: valor por cliente/cota.

## Fluxo 6 - Liberacao para Resgate

1. Confirmar que a premiacao foi registrada.
2. Validar que o premio foi recebido ou esta apto para liberar operacionalmente.
3. Acionar "Liberar premiacao para resgate".
4. Confirmar status "Disponivel para resgate".
5. Validar que o cliente visualiza saldo em `Account/Prizes`.

Antes da liberacao:

- premio aparece como identificado/aguardando liberacao;
- saldo nao entra como disponivel para saque.

Depois da liberacao:

- saldo entra em disponivel para saque;
- cliente ve texto de prazo de 5 a 10 dias uteis apos liberacao.

## Fluxo 7 - Publicacao Publica de Resultados

1. Abrir detalhe do resultado oficial.
2. Revisar conferencias dos boloes.
3. Revisar premiacoes registradas.
4. Informar observacao publica, se necessario.
5. Marcar resultado como publico.
6. Validar loja publica:
   - pagina de resultados;
   - resumo na home;
   - filtros de modalidade, concurso e periodo;
   - status em portugues.

Regra:

- dados de clientes, pedidos, documentos e pagamentos nao sao expostos publicamente.

## Dados Sugeridos para Homologacao

Mega-Sena:

```text
LotteryType;GameNumber;Numbers;Source;ProviderBetId;ReceiptUrl;Notes
Mega-Sena;1;"01,05,12,23,44,60";Manual;;;
Mega-Sena;2;"02,08,19,31,45,59";Manual;;;
```

Quina:

```text
LotteryType;GameNumber;Numbers;Source;ProviderBetId;ReceiptUrl;Notes
Quina;1;"01,05,12,23,44";Manual;;;
Quina;2;"02,08,19,31,45";Manual;;;
```

Lotofacil:

```text
LotteryType;GameNumber;Numbers;Source;ProviderBetId;ReceiptUrl;Notes
Lotofacil;1;"01,02,03,04,05,06,07,08,09,10,11,12,13,14,15";Manual;;;
Lotofacil;2;"02,03,04,05,06,07,08,09,10,11,12,13,14,15,16";Manual;;;
```
