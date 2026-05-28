# Fila de Notificações

**Nível:** Pleno | **Tema:** Mensageria | **Tempo estimado:** 4 a 6 dias

Chamadas HTTP síncronas têm limites. Neste desafio você vai implementar um sistema de notificações assíncrono com fila de mensagens — um padrão presente em praticamente todo backend que escala.

## O que você vai construir

Uma plataforma de e-commerce precisa enviar notificações após eventos importantes (compra confirmada, pagamento aprovado, pedido enviado). O volume é alto e as notificações não podem bloquear o fluxo principal. Você vai construir o sistema de fila que resolve isso.

## Fluxo esperado

```
API → publica evento na fila → consumidor processa → notificação enviada (log estruturado)
```

## Tipos de notificação

| Tipo | Gatilho |
|---|---|
| `COMPRA_CONFIRMADA` | Pedido criado com sucesso |
| `PAGAMENTO_APROVADO` | Status do pedido atualizado |
| `PEDIDO_ENVIADO` | Pedido marcado como enviado |

## Endpoints

| Método | Rota | Descrição | Sucesso | Erro |
|---|---|---|---|---|
| POST | `/pedidos` | Criar pedido (publica evento na fila) | 201 | 400 |
| PUT | `/pedidos/{id}/status` | Atualizar status do pedido | 200 | 400, 404 |
| GET | `/notificacoes` | Listar histórico de notificações | 200 | — |

## Exemplos de requisição

**Criar pedido**

```bash
curl -X POST http://localhost:3000/pedidos \
  -H "Content-Type: application/json" \
  -d '{"usuarioId": 42, "itens": [{"produtoId": 7, "quantidade": 2}], "total": 149.90}'
```

```json
// 201 Created — resposta imediata, sem aguardar a notificação
{ "id": 1, "status": "CRIADO", "criadoEm": "2026-05-25T10:00:00Z" }

// Log assíncrono (alguns ms depois, no consumer):
// [NOTIFICAÇÃO] Tipo: COMPRA_CONFIRMADA | Usuário: 42 | Pedido: 1 | Status: ENVIADA
```

**Listar notificações**

```bash
curl http://localhost:3000/notificacoes
```

```json
// 200 OK
[
  {
    "id": 1,
    "tipo": "COMPRA_CONFIRMADA",
    "usuarioId": 42,
    "pedidoId": 1,
    "status": "ENVIADA",
    "criadaEm": "2026-05-25T10:00:01Z"
  }
]
```

## Requisitos técnicos

- Use RabbitMQ ou Kafka como message broker
- O envio é assíncrono — a API responde antes do consumer terminar
- Retry com no mínimo 3 tentativas em caso de falha no consumer
- Dead Letter Queue (DLQ) para mensagens que falharam após todas as tentativas
- Histórico de notificações persistido no banco de dados
- Cada evento deve gerar exatamente uma notificação (idempotência)
- Status do pedido segue a ordem: `CRIADO → PAGAMENTO_APROVADO → ENVIADO`

## Qualidade do código

```
controller  →  valida e responde HTTP
service     →  publica o evento na fila
consumer    →  processa a mensagem e persiste a notificação
```

## Critérios de avaliação

| Critério | Peso |
|---|---|
| Publicação e consumo de eventos funcionando | 35% |
| Retry e DLQ configurados corretamente | 25% |
| Histórico de notificações persistido | 20% |
| README com arquitetura e instruções de execução | 20% |

## Como entregar

1. Suba sua solução em um repositório público
2. Inclua no README: como rodar, variáveis de ambiente e um diagrama/descrição da arquitetura
3. Abra uma [Discussion](https://github.com/Francisco-Montalvao/backend-challenges/discussions) com o link

## Infraestrutura

O `docker-compose.yml` na pasta sobe PostgreSQL + RabbitMQ para você focar na aplicação:

```bash
docker-compose up
```

- API: http://localhost:3000
- RabbitMQ UI: http://localhost:15672 (guest/guest)

> Se sua app ainda não está containerizada, rode o compose para a infra e a sua API localmente apontando para esses serviços.

## Dicas finais

- Comece com RabbitMQ se for sua primeira vez com mensageria — curva de aprendizado menor que Kafka
- Suba o broker com Docker primeiro, teste manualmente, depois integre à API
- Implemente retry e DLQ por último, quando o fluxo feliz já estiver funcionando
- Testes de integração com o broker e idempotência no consumer são os diferenciais que separam uma solução boa de uma excelente

Boa sorte — mensageria está em todo backend que cresce!
