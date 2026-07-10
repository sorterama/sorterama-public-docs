# Sorterama - Guia de Execucao Local

Data: abril de 2026.
Ultima revisao: 09/07/2026.

Este guia foi escrito para permitir que outro desenvolvedor rode a aplicacao localmente, com o menor numero possivel de dependencias externas obrigatorias.

## O que sobe localmente

- API: `Sorterama.Api`
- Loja publica: `Sorterama.Web`
- Backoffice: `Sorterama.Backoffice.Web`
- PostgreSQL 15
- Redis 7
- RabbitMQ 3-management

## Pre-requisitos

- .NET SDK 10
- Docker Desktop
- PowerShell
- porta 5000 livre para a API
- porta 5271 ou 7267 livre para a loja
- porta 56873 ou 56872 livre para o backoffice
- portas 5432, 5672, 6379 e 15672 livres para a infraestrutura local

### Ferramentas opcionais para assets visuais

Necessarias apenas quando for ajustar imagens estaticas do site, e-mails transacionais ou SVGs.

- Python 3
- Pillow
- ImageMagick
- Inkscape

## Estrategia recomendada para ambiente local

Para evitar bloqueios com providers externos, use este perfil:

- Pix em modo mock
- NFS-e em modo mock
- SMS/WhatsApp em modo noop
- SendGrid configurado apenas se voce realmente quiser testar e-mail

Isso e suficiente para desenvolvimento da maior parte dos fluxos de UI, onboarding, backoffice e autenticacao.

## 1. Suba a infraestrutura local

Na raiz do repositorio:

```powershell
docker compose up -d sorterama-db rabbitmq redis
```

Verifique:

```powershell
docker compose ps
```

RabbitMQ management:

- [http://localhost:15672](http://localhost:15672)
- usuario: `guest`
- senha: `guest`

## 2. Configure as variaveis de ambiente

As aplicacoes usam placeholders em `appsettings`, entao as variaveis abaixo precisam existir na sua sessao antes do `dotnet run`.

### API

```powershell
$env:ASPNETCORE_ENVIRONMENT = "Development"
$env:ConnectionStrings__SqlConnection = "Host=localhost;Port=5432;Database=SORTERAMA;Username=postgres;Password=postgres"
$env:Redis__ConnectionString = "localhost:6379"
$env:EventBusConfiguration__HostAddress = "amqp://guest:guest@localhost:5672"
$env:Jwt__Key = "CHANGE-ME-LOCAL-JWT-KEY-WITH-AT-LEAST-32-CHARS"
$env:Jwt__Issuer = "Sorterama"
$env:Jwt__Audience = "SorteramaUsers"
$env:AdminSeed__Password = "CHANGE-ME@123"
$env:Otp__Pepper = "CHANGE-ME-LOCAL-OTP-PEPPER"
$env:WebApp__BaseUrl = "http://localhost:5271"
$env:WebProxyKey__Key = "CHANGE-ME-LOCAL-WEBPROXY-KEY"

$env:OpenPix__Enabled = "false"
$env:ServiceInvoiceProvider__Provider = "Mock"
$env:Zenvia__Enabled = "false"

# Opcional para testar e-mail de verdade:
# $env:SendGrid__ApiKey = "..."
# $env:SendGrid__FromEmail = "no-reply@seudominio.com"
# $env:SendGrid__FromName = "Sorterama"
# $env:SendGrid__AssetsBaseUrl = "http://localhost:5271"
```

### Loja publica

Abra outra sessao do PowerShell:

```powershell
$env:ASPNETCORE_ENVIRONMENT = "Development"
$env:SorteramaApi__BaseUrl = "http://localhost:5000"
```

### Backoffice

Abra outra sessao do PowerShell:

```powershell
$env:ASPNETCORE_ENVIRONMENT = "Development"
$env:SorteramaApi__BaseUrl = "http://localhost:5000"
```

## 3. Restore e build

Na raiz:

```powershell
dotnet restore Sorterama.sln --disable-parallel -v minimal -p:RestoreUseStaticGraphEvaluation=true
dotnet build Sorterama.sln --no-restore -v minimal /m:1 /nodeReuse:false /p:UseSharedCompilation=false
```

## 4. Suba os tres projetos

### Terminal 1 - API

```powershell
dotnet run --project .\Sorterama.Api\Sorterama.Api.csproj --launch-profile LocalHost
```

API:

- [http://localhost:5000/swagger](http://localhost:5000/swagger)
- health: [http://localhost:5000/health](http://localhost:5000/health)

### Terminal 2 - Loja publica

```powershell
dotnet run --project .\Sorterama.Web\Sorterama.Web.csproj --launch-profile http
```

Loja:

- [http://localhost:5271](http://localhost:5271)

### Terminal 3 - Backoffice

```powershell
dotnet run --project .\Sorterama.Backoffice.Web\Sorterama.Backoffice.Web.csproj
```

Backoffice:

- [http://localhost:56873](http://localhost:56873)
- ou [https://localhost:56872](https://localhost:56872)

## 5. O que esperar no primeiro boot

A API aplica migrations e roda o `ApplicationSeeder` automaticamente.

Isso deve:

- criar ou atualizar os bancos
- seedar usuario administrativo
- preparar dados basicos necessarios ao MVP

Se houver erro de migration, valide primeiro:

- conexao com Postgres
- `Jwt__Key`
- `AdminSeed__Password`
- `Otp__Pepper`

## 6. Credenciais e usuarios

O admin seed usa:

- e-mail: valor de `AdminSeed__Email` ou `admin@sorterama.com`
- senha: valor de `AdminSeed__Password`

Se for preciso recriar o ambiente, limpe os volumes do Docker com cuidado e suba novamente a infraestrutura.

## 7. Fluxos locais recomendados

### Smoke da loja

1. abrir a loja
2. abrir onboarding
3. preencher dados
4. solicitar codigo
5. validar o comportamento local do provider configurado
6. concluir cadastro
7. fazer login com senha
8. fazer login por codigo

### Smoke do backoffice

1. entrar com usuario administrativo
2. cadastrar ou gerar produto/concurso/bolao
3. cadastrar jogos manualmente ou importar CSV
4. enviar bolao para aprovacao
5. aprovar e publicar oferta
6. validar oferta na loja publica
7. registrar resultado oficial
8. conferir jogos dos boloes vinculados
9. registrar premiacao quando houver
10. liberar premiacao para resgate
11. validar saldo em `Account/Prizes`

### Smoke de checkout

Com `OpenPix__Enabled=false`, a cobranca Pix usa `MockPixGateway`.

Isso permite:

1. adicionar item ao carrinho
2. abrir checkout
3. gerar copia e cola Pix de teste
4. seguir com validacoes de UI e persistencia

### Smoke de jogos, resultados e resgate

Use os exemplos em `Sorterama.Documentation/backoffice-publication-test-data.md`.

1. baixar o modelo de importacao de jogos no backoffice
2. importar arquivo valido
3. importar arquivo invalido e confirmar rejeicao integral
4. cadastrar resultado oficial
5. conferir o bolao
6. registrar faixas de premiacao
7. confirmar que o premio aparece como identificado/aguardando liberacao
8. liberar premiacao para resgate
9. validar que a area logada mostra prazo de 5 a 10 dias uteis apos liberacao

## 8. Providers externos opcionais

### SendGrid

Necessario apenas para testar e-mails reais.

Sem `SendGrid__ApiKey`, fluxos que disparam e-mail podem falhar.

### Zenvia

Com `Zenvia__Enabled=false`, SMS e WhatsApp usam servicos noop e nao bloqueiam o desenvolvimento local.

### OpenPix/Woovi

Para desenvolvimento do fluxo da loja sem webhook real, prefira `OpenPix__Enabled=false`.

### Focus NFe

Para desenvolvimento local, prefira:

```powershell
$env:ServiceInvoiceProvider__Provider = "Mock"
```

## 9. Troubleshooting

### A loja ou o backoffice nao conseguem chamar a API

Verifique:

- `SorteramaApi__BaseUrl`
- API realmente escutando em `http://localhost:5000`
- se voce nao subiu a API apenas em HTTPS

### Login por codigo volta para a mesma pagina

Consulte primeiro os logs da `Sorterama.Web`.

Os problemas recentes nessa area normalmente apareceram como:

- erro de `ModelState`
- `ChallengeId` ausente
- problema de canal (`Channel`) nao reenviado

### Onboarding nao envia codigo

Verifique:

- `Otp__Pepper`
- `Zenvia__Enabled`
- `SendGrid__ApiKey` se estiver usando fallback por e-mail
- logs da API em `/api/onboarding/phone-verification`, `/api/onboarding/email-verification` e `/api/onboarding/verify-code`

### Erro de sessao ou token invalido

Verifique:

- `SORTERAMA_DP_KEYS_DIR`
- reinicios frequentes apagando chaves locais
- `Jwt__Key`

## 10. Comandos uteis

Parar a infraestrutura:

```powershell
docker compose stop
```

Ver logs do Postgres:

```powershell
docker compose logs -f sorterama-db
```

Ver logs do RabbitMQ:

```powershell
docker compose logs -f rabbitmq
```

Ver logs do Redis:

```powershell
docker compose logs -f redis
```

## 11. Padronizacao dos assets do Muri

As imagens do Muri usadas no site e em mensagens transacionais ficam em:

```text
Sorterama.Web/wwwroot/images/muri
```

Para evitar problemas de exibicao em fundo escuro, clientes de e-mail ou navegadores com renderizacao diferente, o repositorio possui scripts para padronizar as imagens principais do Muri.

Os scripts fazem:

- correcao de transparencia em lote
- aplicacao de fundo `#fefefe`
- padronizacao de canvas para `569x584`
- aplicacao de margem visual
- otimizacao dos PNGs para web/e-mail
- inclusao de fundo nos SVGs correspondentes
- validacao de renderizacao dos SVGs com Inkscape, quando disponivel
- geracao de preview visual em arquivo temporario

### Windows

Instale as dependencias:

```powershell
winget install Python.Python.3.12
winget install ImageMagick.ImageMagick
winget install Inkscape.Inkscape
python -m pip install pillow
```

Se `python`, `magick` ou `inkscape` nao forem reconhecidos no terminal atual, abra um novo PowerShell.

Se ainda assim o Python abrir a Microsoft Store, desative os aliases:

```text
Configuracoes > Apps > Configuracoes avancadas de app > Aliases de execucao de app
```

Desative:

- `python.exe`
- `python3.exe`

Execute na raiz do repositorio:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\standardize-muri-assets.ps1
```

Tambem e possivel informar parametros:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\standardize-muri-assets.ps1 `
  -ImageDir "Sorterama.Web/wwwroot/images/muri" `
  -Background "#fefefe" `
  -CanvasWidth 569 `
  -CanvasHeight 584 `
  -Margin 24
```

### Linux

Instale as dependencias:

```bash
sudo apt update
sudo apt install -y python3 python3-pip imagemagick inkscape
python3 -m pip install pillow
```

Execute na raiz do repositorio:

```bash
chmod +x scripts/standardize-muri-assets.sh
./scripts/standardize-muri-assets.sh
```

Tambem e possivel informar parametros:

```bash
./scripts/standardize-muri-assets.sh \
  --image-dir "Sorterama.Web/wwwroot/images/muri" \
  --background "#fefefe" \
  --canvas-width 569 \
  --canvas-height 584 \
  --margin 24
```

### macOS

Instale as dependencias com Homebrew:

```bash
brew install python imagemagick inkscape
python3 -m pip install pillow
```

Execute na raiz do repositorio:

```bash
chmod +x scripts/standardize-muri-assets.sh
./scripts/standardize-muri-assets.sh
```

### Resultado esperado

Ao final da execucao, o script deve informar:

- relatorio dos PNGs processados
- status dos SVGs
- caminho do preview visual
- resultado da otimizacao com ImageMagick, quando disponivel
- caminho dos SVGs renderizados pelo Inkscape, quando disponivel

No Windows o preview normalmente fica em:

```text
%TEMP%\sorterama_muri_contact_sheet.png
```

No Linux/macOS o preview normalmente fica em:

```text
/tmp/sorterama_muri_contact_sheet.png
```

## 12. Checklist minimo para novo desenvolvedor

1. clonar o repositorio
2. instalar .NET 10 SDK
3. subir `sorterama-db`, `rabbitmq` e `redis`
4. exportar variaveis de ambiente da API
5. exportar `SorteramaApi__BaseUrl` para loja e backoffice
6. rodar restore e build
7. subir API, loja e backoffice
8. abrir swagger, loja e backoffice
9. testar onboarding e login
