# Por onde começar?

Mensageria tem muitas partes novas de uma vez. Este arquivo explica na ordem certa para você não se perder.

## 1. Entenda o problema antes do código

Antes de tocar no broker, tenha claro na cabeça:

- **Quem publica?** A API, no momento em que cria ou atualiza um pedido
- **Quem consome?** Um processo separado (consumer/worker) que roda em paralelo
- **O que acontece se o consumer falhar?** A mensagem não pode ser perdida — é aí que entram retry e DLQ

O fluxo simplificado:
```
POST /pedidos → service publica mensagem → retorna 201
                     ↓ (assíncrono)
              consumer lê a mensagem
              tenta enviar notificação
              persiste no banco
```

## 2. Comece pelo producer

Antes de pensar em consumer, retry ou DLQ: faça uma mensagem chegar na fila.

```bash
# Sobe o RabbitMQ
docker-compose up rabbitmq

# Acesse a UI em http://localhost:15672 (guest/guest)
# Crie uma fila manualmente e publique uma mensagem pela UI para testar
```

Depois conecte seu código ao broker e publique a mensagem no `POST /pedidos`.

## 3. Consumer e Ack/Nack

O consumer precisa confirmar que processou a mensagem — isso se chama `Ack`. Se falhar, faz `Nack`:

```
consumer recebe mensagem
  → processa com sucesso → Ack (mensagem removida da fila)
  → falha              → Nack (mensagem volta para a fila ou vai para DLQ)
```

Nunca deixe o auto-ack habilitado em produção — você perde a mensagem se o consumer travar.

## 4. Retry e Dead Letter Queue

Configure a fila principal para reenviar para a DLQ após N tentativas fracassadas:

- **RabbitMQ:** use `x-dead-letter-exchange` e `x-message-ttl` nas propriedades da fila
- **Kafka:** use um tópico separado de retry + outro de dead letter

A DLQ é só uma fila normal — você pode inspecioná-la, reprocessar ou alertar quando mensagens chegarem lá.

## 5. Idempotência

O que acontece se a mesma mensagem for processada duas vezes? (Rede instável, consumer reinicia, etc.)

Salve o ID da mensagem processada no banco. Antes de processar, verifique se já existe:

```sql
-- Se já existe, ignora (não duplica a notificação)
SELECT id FROM notificacoes WHERE mensagem_id = :id
```

---

Links úteis:
- [RabbitMQ — Tutoriais oficiais](https://www.rabbitmq.com/tutorials)
- [Dead Letter Exchanges no RabbitMQ](https://www.rabbitmq.com/docs/dlx)
- [RabbitMQ vs Kafka — quando usar cada um](https://www.cloudamqp.com/blog/when-to-use-rabbitmq-or-apache-kafka.html)

Libs por linguagem: `amqplib` (Node.js), `pika` (Python), Spring AMQP (Java), `amqp091-go` (Go)
