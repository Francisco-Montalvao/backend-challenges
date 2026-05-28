# 🧪 Guia de Testes — Backend Challenges

Cada nível tem estratégias de teste diferentes. Aqui está como testar seus desafios.

---

## 🟢 Nível Júnior — Testes Básicos

### O que testar

1. **Testes Unitários** — Funções isoladas
2. **Testes de Integração** — API completa
3. **Testes de Banco de Dados** — Persistência

### Ferramentas Recomendadas

| Linguagem | Framework        | Assertions | Mocking     |
|-----------|------------------|-----------|------------|
| Java      | JUnit 5 + Mockito | AssertJ   | Mockito    |
| Python    | pytest           | pytest    | pytest-mock |
| Node.js   | Jest             | expect()  | Jest       |
| Go        | testing + testify| assert    | testify    |

### Exemplo: To-Do List API

```bash
# 1. Instale dependências de teste
npm install --save-dev jest supertest

# 2. Estruture os testes
tests/
├── unit/
│   ├── service.test.js
│   └── repository.test.js
└── integration/
    └── api.test.js

# 3. Rode os testes
npm test

# 4. Verifique cobertura
npm test -- --coverage
```

### ✅ Critério Mínimo de Teste

- Teste **cada endpoint** (GET, POST, PUT, DELETE)
- Teste **casos de sucesso** (2xx)
- Teste **casos de erro** (4xx, 5xx)
- Teste **validações** (campos obrigatórios, tipos, limites)

**Exemplo de teste para To-Do List:**

```javascript
describe('POST /tarefas', () => {
  it('deve criar uma tarefa com sucesso', async () => {
    const res = await request(app).post('/tarefas').send({
      titulo: 'Estudar Jest',
      descricao: 'Aprender testes em JavaScript'
    });
    expect(res.status).toBe(201);
    expect(res.body.id).toBeDefined();
    expect(res.body.concluida).toBe(false);
  });

  it('deve retornar 400 com título vazio', async () => {
    const res = await request(app).post('/tarefas').send({
      descricao: 'Sem título'
    });
    expect(res.status).toBe(400);
  });
});
```

---

## 🟡 Nível Pleno — Testes Avançados

### O que testar

1. **Testes de Integração** com message broker
2. **Testes de Cache**
3. **Testes de Concorrência**
4. **Testes de Performance**

### Exemplo: Fila de Notificações

```bash
# Use testcontainers para subir RabbitMQ/Kafka automaticamente
npm install --save-dev testcontainers

# Teste o fluxo completo
npm test -- --testPathPattern=integration
```

**Exemplo de teste para fila:**

```javascript
describe('Fila de Notificações', () => {
  let rabbitMQ;

  beforeAll(async () => {
    // Suba um container do RabbitMQ para teste
    rabbitMQ = await new RabbitMQContainer().start();
  });

  it('deve publicar e consumir mensagem com sucesso', async () => {
    const notificacao = {
      tipo: 'COMPRA_CONFIRMADA',
      usuarioId: 42
    };

    await publisher.publish(notificacao);

    // Aguarde o consumidor processar
    await new Promise(resolve => setTimeout(resolve, 500));

    const registroDb = await db.notificacoes.findOne(notificacao.type);
    expect(registroDb.status).toBe('ENVIADA');
  });

  it('deve fazer retry após falha', async () => {
    // Simule falha no consumidor
    consumer.simulateError = true;

    await publisher.publish({ tipo: 'TESTE' });
    
    // Aguarde retries
    await sleep(2000);

    const tentativas = await db.logs.countBy({ tipo: 'TESTE' });
    expect(tentativas).toBeGreaterThanOrEqual(3); // Mínimo 3 tentativas
  });
});
```

### 🎯 Critério Mínimo

- Teste o **fluxo completo** (publicação → consumo)
- Teste **retries e DLQ**
- Teste **idempotência** (mesma mensagem 2x = resultado igual)
- Teste **timeout e erro** de conexão

---

## 🔴 Nível Sênior — Testes Distribuídos

### O que testar

1. **Testes de Contrato** (entre serviços)
2. **Testes de Chaos** (resiliência)
3. **Testes de Integração de Ponta a Ponta**
4. **Testes de Segurança**

### Ferramentas

| Ferramenta | Uso |
|-----------|-----|
| Pact | Testes de contrato |
| Testcontainers | Containers para testes |
| Resilience4j | Teste de Circuit Breaker |
| JMeter / k6 | Testes de carga |
| OWASP ZAP | Testes de segurança |

### Exemplo: Sistema de Pedidos (Microsserviços)

```bash
# 1. Teste de contrato entre order-service e product-service
npm install --save-dev @pact-foundation/pact

# 2. Structure dos testes
tests/
├── pact/
│   └── order-product.pact.js
├── e2e/
│   └── order-flow.test.js
└── chaos/
    └── circuit-breaker.test.js
```

**Exemplo de teste de contrato Pact:**

```javascript
describe('Order Service > Product Service', () => {
  const provider = new Pact({ consumer: 'OrderService', provider: 'ProductService' });

  it('deve consultar estoque de produto', () => {
    return provider
      .addInteraction({
        state: 'produto existe',
        uponReceiving: 'uma requisição de estoque',
        withRequest: { method: 'GET', path: '/products/7/stock' },
        willRespondWith: {
          status: 200,
          body: { productId: 7, available: 10 }
        }
      })
      .then(() => consultoProductService(7))
      .then(result => expect(result.available).toBe(10));
  });
});
```

**Exemplo de teste de Circuit Breaker:**

```javascript
describe('Resiliência - Circuit Breaker', () => {
  it('deve falhar rápido após 3 erros', async () => {
    // Simule 3 falhas no payment-service
    paymentServiceMock.returnError(3);

    const start = Date.now();
    try {
      await createOrder({ userId: 1, total: 100 });
    } catch (error) {
      expect(error.message).toContain('Circuit breaker is OPEN');
      expect(Date.now() - start).toBeLessThan(1000); // Falha rápido
    }
  });

  it('deve tentar recuperar após tempo de espera', async () => {
    // Aguarde o timeout do circuit breaker
    await sleep(30000);

    paymentServiceMock.returnSuccess();
    const order = await createOrder({ userId: 1, total: 100 });
    expect(order.status).toBe('APPROVED');
  });
});
```

### 🎯 Critério Mínimo

- Teste **comunicação entre serviços** com Pact
- Teste **Circuit Breaker** (open/half-open/closed)
- Teste **eventual consistency** (dados sincronizam)
- Teste **fallback** (continua funcionando com degradação)

---

## 🧪 Configuração Geral

### Cobertura de Código

```bash
# Verifique cobertura
npm test -- --coverage

# Objetivo mínimo: 70% de linha coverage
# Objetivo ideal: 85%+ de linha coverage
```

### CI/CD

Adicione GitHub Actions para rodar testes automaticamente:

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm install
      - run: npm test
      - run: npm run test:coverage
```

---

## 📊 Checklist de Testes por Nível

### 🟢 Júnior
- [ ] Testes unitários para serviços
- [ ] Testes de integração para endpoints
- [ ] Testes de validação (entrada/saída)
- [ ] Cobertura mínima: 70%

### 🟡 Pleno
- [ ] Tudo do Júnior +
- [ ] Testes de integração com broker
- [ ] Testes de retry e DLQ
- [ ] Testes de timeout e erro
- [ ] Cobertura mínima: 80%

### 🔴 Sênior
- [ ] Tudo do Pleno +
- [ ] Testes de contrato (Pact)
- [ ] Testes de chaos/resiliência
- [ ] Testes de segurança (OWASP)
- [ ] Testes de performance
- [ ] Cobertura mínima: 85%

---

## 🚀 Template de Test Suite

```javascript
describe('API - [Funcionalidade]', () => {
  let app;
  let db;

  beforeAll(async () => {
    // Setup
    app = await createApp();
    db = await setupTestDatabase();
  });

  afterEach(async () => {
    // Limpe dados entre testes
    await db.clear();
  });

  afterAll(async () => {
    // Teardown
    await app.close();
    await db.close();
  });

  describe('Casos de Sucesso', () => {
    it('deve fazer X', async () => {
      // Arrange
      const input = { /* ... */ };

      // Act
      const result = await api.call(input);

      // Assert
      expect(result.status).toBe(200);
      expect(result.body).toEqual(expectedOutput);
    });
  });

  describe('Casos de Erro', () => {
    it('deve retornar 400 para entrada inválida', async () => {
      const res = await api.call({ /* invalid */ });
      expect(res.status).toBe(400);
    });
  });
});
```

---

**Quer ajuda com testes em uma linguagem específica? Abra uma Issue!**

