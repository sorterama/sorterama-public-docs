# Sorterama - Produto e Arquitetura

Documento publico para investidores, parceiros e stakeholders.

Ultima revisao: 25/04/2026.

## Visao Geral

Sorterama e uma plataforma digital para organizar, vender e operar boloes de loteria com controle financeiro, rastreabilidade operacional e automacao fiscal.

A proposta e transformar uma operacao tradicionalmente manual em um produto digital escalavel, com experiencia simples para o cliente e ferramentas de controle para a empresa.

## Problema

Boloes costumam depender de processos fragmentados:

- controle manual de participantes;
- baixa rastreabilidade de pagamentos;
- dificuldade para conciliar valores;
- pouca transparencia para o cliente;
- operacao fiscal e contabil separada do fluxo de venda;
- suporte dependente de planilhas e historico informal.

Esses pontos limitam crescimento, confianca e capacidade de auditoria.

## Solucao

Sorterama centraliza o ciclo de vida da venda:

1. publicacao de ofertas;
2. cadastro com validacao de contato;
3. login do cliente com senha ou codigo;
4. compra de cotas ou pacotes;
5. pagamento Pix;
6. confirmacao automatica;
7. registro financeiro;
8. emissao fiscal da taxa de administracao;
9. notificacao ao cliente;
10. acompanhamento pelo backoffice.

O objetivo do MVP e validar o ciclo completo com seguranca operacional antes de ampliar volume, canais comerciais e automacoes.

## Experiencia do Cliente

O cliente acessa a loja, escolhe uma oferta, realiza o pagamento por Pix e acompanha seus pedidos na area logada.

Fluxos principais:

- onboarding guiado com verificacao por telefone e opcao de fallback por e-mail;
- login com senha ou login por codigo;
- compra avulsa de cota;
- pacote mensal com mais de um bolao incluido;
- acompanhamento do pedido;
- acesso a informacoes fiscais quando aplicavel;
- recebimento de comunicacoes transacionais.

## Backoffice

O backoffice foi pensado para dar controle operacional e financeiro para a equipe.

Capacidades previstas no MVP:

- cadastro e publicacao de produtos;
- cadastro de concursos e boloes;
- aprovacao de ofertas;
- consulta de clientes e pedidos;
- acompanhamento de onboarding, autenticacao e status operacionais;
- acompanhamento financeiro;
- conciliacao entre pedido, pagamento, ledger e nota fiscal;
- exportacao CSV para apoio fiscal e contabil.

## Diferenciais Operacionais do MVP

Mesmo em fase inicial, o produto ja foi desenhado para reduzir atrito operacional em pontos que costumam travar o crescimento:

- validacao de contato antes da conclusao do cadastro;
- login por codigo para reduzir barreira de acesso;
- pagamento Pix com confirmacao automatica;
- separacao financeira entre cota e taxa de administracao;
- retaguarda administrativa para publicacao, suporte e conciliacao;
- emissao fiscal desacoplada do momento da compra.

## Modelo Financeiro

A plataforma separa os componentes financeiros da venda:

- valor destinado a cota/participacao;
- taxa de administracao;
- descontos ou isencoes;
- status de pagamento;
- eventos de conciliacao.

Essa separacao permite acompanhar receita operacional, valores de terceiros, pendencias, divergencias e informacoes relevantes para contabilidade.

No MVP, a emissao fiscal esta focada na taxa de administracao.

## Arquitetura em Alto Nivel

Sorterama usa uma arquitetura modular, com separacao entre interface, regras de negocio, integracoes e persistencia.

```mermaid
flowchart LR
    Cliente["Cliente"]
    Loja["Loja Web"]
    Backoffice["Backoffice"]
    Plataforma["API e Regras de Negocio"]
    Dados["Banco de Dados"]
    Fila["Fila Assincrona"]
    Pagamento["Pagamento Pix"]
    Fiscal["Emissao Fiscal"]
    Comunicacao["E-mail e Notificacoes"]
    Relatorios["Relatorios e Conciliacao"]

    Cliente --> Loja
    Loja --> Plataforma
    Backoffice --> Plataforma
    Plataforma --> Dados
    Plataforma --> Fila
    Plataforma --> Pagamento
    Fila --> Fiscal
    Plataforma --> Comunicacao
    Plataforma --> Relatorios
```

## Integracoes Externas

No MVP, as integracoes relevantes sao:

- OpenPix/Woovi para pagamento Pix;
- SendGrid para e-mails transacionais;
- Zenvia para SMS e WhatsApp;
- Focus NFe para emissao de nota fiscal de servico;
- PostgreSQL, Redis e RabbitMQ como base operacional.

Essas integracoes ficam encapsuladas, o que facilita substituicao de provider e reduz acoplamento do produto a fornecedores especificos.

## Principios Tecnicos

- Separacao clara entre produto, dominio, infraestrutura e interfaces.
- Processos criticos assincronos para evitar bloquear o cliente.
- Integracoes externas encapsuladas por adapters.
- Rastreabilidade de pedidos, pagamentos e notas fiscais.
- Base preparada para automacao operacional e analise de dados.
- Deploy conteinerizado para facilitar evolucao e portabilidade.

## Resiliencia Operacional

Fluxos que dependem de provedores externos sao tratados de forma resiliente.

Exemplo: a confirmacao de pagamento nao precisa esperar a emissao fiscal. A nota fiscal pode ser processada em segundo plano, com status acompanhado pelo backoffice.

Isso reduz impacto para o cliente e permite que a operacao trate pendencias sem perder rastreabilidade.

Da mesma forma, fluxos de comunicacao e validacao de contato podem seguir por canais alternativos, reduzindo dependencia de um unico meio.

## Dados e Relatorios

A plataforma registra eventos financeiros em uma base interna de ledger, permitindo:

- visao de vendas confirmadas;
- separacao entre cota e taxa de administracao;
- identificacao de pendencias;
- conciliacao com pagamentos;
- conciliacao com emissao fiscal;
- extracao CSV para analises operacionais.

Com o produto homologado, essa base pode evoluir para relatorios mensais de operacao, indicadores de crescimento e materiais para investidores.

## Observabilidade

O MVP ja considera logs e rastreabilidade como parte da operacao.

Objetivos:

- identificar falhas de pagamento, fila e emissao fiscal;
- localizar eventos por pedido, pagamento ou nota;
- apoiar suporte e investigacao;
- reduzir dependencia de analise manual no banco de dados.

Em fases futuras, a observabilidade pode evoluir para dashboards operacionais, alertas automaticos e analise de funil.

## Estagio Atual do MVP

Na revisao atual, o produto ja demonstra:

- onboarding funcional com verificacao de contato;
- login por codigo em funcionamento;
- checkout Pix com geracao de chave copia e cola;
- backoffice para operacao e publicacao;
- trilha financeira e fiscal desacoplada;
- documentacao tecnica, plano de testes e guia de setup local para acelerar time e homologacao.

## Evolucao Esperada

Apos homologacao, os proximos vetores naturais de evolucao sao:

- melhorar experiencia mobile;
- fortalecer relatorios operacionais e comerciais;
- automatizar mais fluxos de suporte;
- ampliar integracoes fiscais e financeiras;
- criar indicadores mensais para decisao de negocio;
- evoluir controles antifraude e compliance;
- preparar a plataforma para maior volume de usuarios e transacoes.

## Tese de Produto

Sorterama combina tres elementos importantes:

- uma experiencia de compra simples para o consumidor;
- uma operacao interna rastreavel e auditavel;
- uma base tecnica preparada para escalar processos financeiros e fiscais.

O MVP busca provar que o ciclo completo de venda, pagamento, participacao, conciliacao e fiscalizacao pode operar de forma digital e confiavel.
