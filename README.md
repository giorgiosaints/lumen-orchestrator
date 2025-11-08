# Lumen Orchestrator

Orquestrador construído com Lumen para coordenar chamadas entre microserviços, gerenciar rotas de requisição, políticas de retry e expor endpoints de health/monitoring. Este repositório fornece a base para construir um gateway leve e confiável para arquiteturas distribuídas.

> Observação: este README é um ponto de partida — ajuste os comandos, variáveis de ambiente e exemplos conforme a implementação real do repositório.

## Principais funcionalidades

- Roteamento e proxy reverso para microserviços
- Retry configurável e políticas de timeout
- Health checks e endpoints de monitoramento
- Logging estruturado e métricas básicas
- Suporte para execução via Docker / docker-compose
- Estrutura de configuração via `.env`

## Tecnologias

- PHP (Lumen)
- Composer
- Docker (opcional)
- Redis / RabbitMQ (opcional, para filas)
- MySQL / Postgres (opcional, para persistência de estado)

## Requisitos

- PHP 8.0+ (verifique a versão exigida no composer.json)
- Composer
- Docker & Docker Compose (opcional, recomendado para ambiente isolado)

## Instalação (local)

1. Clone o repositório:
   ```bash
   git clone https://github.com/giorgiosaints/lumen-orchestrator.git
   cd lumen-orchestrator
   ```

2. Instale dependências PHP:
   ```bash
   composer install --no-dev --optimize-autoloader
   ```

3. Copie o arquivo de ambiente e ajuste as variáveis:
   ```bash
   cp .env.example .env
   # edite .env com seus valores (DB, serviços downstream, chaves, etc.)
   ```

4. Execute migrações (se aplicável):
   ```bash
   php artisan migrate
   ```

5. Inicie o servidor Lumen:
   ```bash
   php -S localhost:8000 -t public
   ```
   Ou use o comando artisan se disponível:
   ```bash
   php artisan serve --host=0.0.0.0 --port=8000
   ```

## Uso com Docker

Exemplo mínimo de execução via docker-compose:

```yaml
version: "3.8"
services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - APP_ENV=local
      - APP_DEBUG=true
    volumes:
      - ./:/var/www/html
    depends_on:
      - redis
  redis:
    image: redis:7
    ports:
      - "6379:6379"
```

Inicie:
```bash
docker-compose up -d --build
```

## Arquitetura e comportamento

- O Orchestrator expõe endpoints REST que recebem requisições de entrada e roteiam para os microserviços configurados.
- Políticas de retry e timeout podem ser aplicadas por rota. Verifique o arquivo de configuração (ex.: config/orchestrator.php ou .env) para ajustar comportamentos.
- Health check: existe um endpoint `/health` ou `/status` que avalia dependências (banco, redis, serviços downstream).
- Logging: use channels Monolog configurados em `config/logging.php`.

## Configuração (exemplo .env)

```env
APP_NAME="Lumen Orchestrator"
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8000

# Database
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=orchestrator
DB_USERNAME=root
DB_PASSWORD=secret

# Redis
REDIS_HOST=127.0.0.1
REDIS_PORT=6379

# Orchestration
ORCHESTRATOR_RETRY_COUNT=3
ORCHESTRATOR_TIMEOUT_MS=2000

# Downstream service example
SERVICE_USER_BASE_URL=http://users-service:8080
SERVICE_ORDER_BASE_URL=http://orders-service:8080
```

Ajuste as chaves acima conforme sua infraestrutura.

## Endpoints (exemplos)

- GET /health — retorna status do serviço e dependências
- POST /orchestrate — endpoint genérico que recebe payload e executa fluxo orquestrado
- GET /metrics — métricas básicas (se implementadas)

Exemplo de requisição para orquestrar:
```bash
curl -X POST http://localhost:8000/orchestrate \
  -H "Content-Type: application/json" \
  -d '{"action":"create-order","payload":{...}}'
```

## Desenvolvimento

- Habilite o modo dev no `.env`:
  ```
  APP_ENV=local
  APP_DEBUG=true
  ```
- Executar testes (se houver):
  ```bash
  ./vendor/bin/phpunit
  ```
- Formatação e lint:
  ```bash
  composer run-script phpcbf   # ou php-cs-fixer / phpcs conforme configurado
  ```

## Observabilidade e produção

- Utilize um gerenciador de processos (supervisord / systemd / Docker) para garantir reinício automático.
- Configure logs rotacionados e envie logs para central de observabilidade (ELK, Loki).
- Proteja endpoints com autenticação (API keys / JWT / OAuth) conforme necessidade.
- Em produção, ajuste:
  - APP_DEBUG=false
  - Timeouts e retries adequados
  - Políticas de circuit breaker para chamadas a serviços lentos/indisponíveis

## Contribuição

Contribuições são bem-vindas. Sugestões:

- Abra uma issue descrevendo o problema ou a feature desejada.
- Para PRs:
  - Fork o repositório
  - Crie uma branch com nome descritivo (ex.: feature/retry-policy)
  - Adicione testes
  - Abra um Pull Request explicando as mudanças

## Licença

Este projeto está sob a licença MIT — veja o arquivo LICENSE para detalhes.

## Contato

Para dúvidas ou suporte, abra uma issue no repositório ou contate o mantenedor: @giorgiosaints
