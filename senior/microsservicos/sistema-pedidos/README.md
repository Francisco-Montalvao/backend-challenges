# Sistema de Pedidos — Microsserviços

**Nível:** Sênior | **Tema:** Microsserviços | **Tempo estimado:** 1 a 2 semanas

Monolitos têm limites. Neste desafio você vai projetar e implementar a espinha dorsal de um sistema de pedidos distribuído — com serviços independentes, comunicação resiliente e decisões de design que você precisa justificar.

## O que você vai construir

Um marketplace está migrando seu monolito para microsserviços. Você vai construir os serviços principais e fazer com que eles se comuniquem de forma confiável.

## Arquitetura esperada

```
                        ┌─────────────┐
                        │   API GW    │
                        └──────┬──────┘
               ┌───────────────┼───────────────┐
               ▼               ▼               ▼
      ┌────────────────┐ ┌──────────┐ ┌──────────────┐
      │ Order Service  │ │  User    │ │   Product    │
      │                │ │ Service  │ │   Service    │
      └───────┬────────┘ └──────────┘ └──────────────┘
              │ eventos
              ▼
      ┌───────────────┐    ┌─────────────────┐
      │ Message Broker│───▶│Payment Service  │
      └───────────────┘    └─────────────────┘
```

## Serviços obrigatórios

| Serviço | Responsabilidade |
|---|---|
| `order-service` | Criação e gerenciamento de pedidos |
| `product-service` | Catálogo de produtos e controle de estoque |
| `payment-service` | Processamento de pagamentos (simulado) |
| `user-service` | Cadastro e autenticação de usuários |
| `api-gateway` | Roteamento e autenticação centralizada |

## Fluxo principal

1. Usuário autenticado faz um pedido via API Gateway
2. `order-service` valida o estoque consultando `product-service` (síncrono)
3. `order-service` publica evento `PEDIDO_CRIADO` no broker
4. `payment-service` consome o evento e processa o pagamento (simulado)
5. `payment-service` publica `PAGAMENTO_APROVADO` ou `PAGAMENTO_RECUSADO`
6. `order-service` atualiza o status do pedido

## Requisitos técnicos

- Cada serviço com **banco de dados próprio** — sem banco compartilhado
- Comunicação **síncrona** via HTTP/REST entre serviços quando necessário
- Comunicação **assíncrona** via message broker para eventos críticos
- Padrão **Circuit Breaker** na comunicação entre serviços
- **Health check** em todos os serviços (`GET /health`)
- `docker-compose.yml` orquestrando todos os serviços

## Requisitos de resiliência

- O sistema deve continuar funcionando (de forma degradada) se o `payment-service` estiver offline
- Pedidos devem ser processados eventualmente quando o serviço voltar (eventual consistency)
- Logs estruturados com `correlationId` que atravesse todos os serviços

## Exemplos de requisição

**Criar pedido (via API Gateway)**

```bash
TOKEN="seu_token_aqui"

curl -X POST http://localhost:8000/orders \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"items": [{"productId": 7, "quantity": 2}]}'
```

```json
// 201 Created
{ "orderId": 1, "status": "CREATED", "createdAt": "2026-05-25T10:00:00Z" }
```

**Health check**

```bash
curl http://localhost:8000/health
curl http://localhost:3001/health
```

```json
// 200 OK
{ "status": "ok", "service": "order-service", "uptime": 3600 }
```

## Critérios de avaliação

| Critério | Peso |
|---|---|
| Todos os serviços funcionando e se comunicando | 30% |
| Banco de dados isolado por serviço | 15% |
| Circuit Breaker implementado | 20% |
| Resiliência quando payment-service está offline | 20% |
| README com diagrama da arquitetura + decisões de design | 15% |

## Como entregar

1. Suba sua solução em um repositório público
2. Inclua no README: como rodar (`docker-compose up`), diagrama da arquitetura e decisões principais
3. Escreva um `DECISIONS.md` explicando: divisão de serviços, como lidou com consistência eventual, o que faria diferente em produção
4. Abra uma [Discussion](https://github.com/Francisco-Montalvao/backend-challenges/discussions) com o link

## Infraestrutura

```bash
docker-compose up
```

- API Gateway: http://localhost:8000
- RabbitMQ UI: http://localhost:15672 (guest/guest)
- Jaeger UI (tracing): http://localhost:16686

## Diferenciais

- Distributed tracing com OpenTelemetry + Jaeger
- Saga Pattern para transações distribuídas
- Kubernetes manifests (deployment, service, configmap)
- Testes de contrato entre serviços (Pact)
- Rate limiting no API Gateway

Boa sorte — este desafio exige decisões reais de design, não só código que funciona.
