# 💡 Por onde começar?

Relatórios com agregações parecem complexos, mas ficam simples quando você entende o padrão. Este guia explica do zero — do banco até a resposta HTTP.

---

## 1. Comece pelo banco, não pelo código

Antes de escrever qualquer endpoint, rode as queries no banco e veja os resultados. Só depois encapsule no repositório.

```sql
-- Total de vendas em um período
SELECT
    SUM(valor_total) AS total_vendido,
    COUNT(*)         AS quantidade_vendas
FROM vendas
WHERE data BETWEEN '2026-01-01' AND '2026-01-31';

-- Ranking de vendedores (JOIN + GROUP BY + ORDER BY)
SELECT
    v.id,
    v.nome,
    SUM(ve.valor_total) AS total_vendido,
    COUNT(*)            AS quantidade_vendas
FROM vendas ve
JOIN vendedores v ON v.id = ve.vendedor_id
WHERE ve.data BETWEEN '2026-01-01' AND '2026-01-31'
GROUP BY v.id, v.nome
ORDER BY total_vendido DESC;

-- Total agrupado por dia
SELECT
    data,
    SUM(valor_total) AS total_vendido,
    COUNT(*)         AS quantidade_vendas
FROM vendas
WHERE data BETWEEN '2026-01-01' AND '2026-01-31'
GROUP BY data
ORDER BY data ASC;
```

> **Regra de ouro:** deixe o banco calcular com `SUM`, `COUNT` e `GROUP BY`. Nunca busque todos os registros e some no código.

---

## 2. Implemente o CRUD antes dos relatórios

Os relatórios dependem de dados no banco. A ordem certa é:

```
1. Criar tabelas (migration ou import.sql)
2. CRUD de vendedores
3. CRUD de vendas
4. Endpoints de relatório
```

Seguindo essa ordem você consegue testar cada etapa antes de avançar.

---

## 3. Validação de datas em Java

Query params chegam como `String` — converta e valide antes de usar na query.

```java
// Converter e validar o formato
LocalDate parseDate(String valor, String nomeCampo) {
    try {
        return LocalDate.parse(valor, DateTimeFormatter.ISO_LOCAL_DATE);
    } catch (DateTimeParseException e) {
        throw new ValidacaoException(nomeCampo + " está em formato inválido. Use YYYY-MM-DD.");
    }
}

// Validar se os dois parâmetros foram enviados
if (dataInicio == null || dataInicio.isBlank()) {
    throw new ValidacaoException("data_inicio é obrigatório. Use o formato YYYY-MM-DD.");
}

// Validar ordem do intervalo
if (fim.isBefore(inicio)) {
    throw new ValidacaoException("data_fim não pode ser anterior a data_inicio.");
}
```

Valide nesta ordem: **ausente → formato inválido → intervalo invertido**.

> Crie um método reutilizável para essa validação — os três endpoints de relatório vão precisar dela.

---

## 4. Validações do CRUD

Além das datas, fique atento às validações das entidades:

```java
// Vendedor — email único no banco
// Antes de inserir, verifique:
SELECT COUNT(*) FROM vendedores WHERE email = :email;

// Venda — vendedor_id deve existir
// Antes de inserir, verifique:
SELECT COUNT(*) FROM vendedores WHERE id = :vendedor_id;

// Venda — valor_total maior que zero
if (valorTotal <= 0) {
    throw new ValidacaoException("valor_total deve ser maior que zero.");
}

// Venda — data não pode ser futura
if (data.isAfter(LocalDate.now())) {
    throw new ValidacaoException("data não pode ser uma data futura.");
}
```

---

## 5. Status HTTP neste desafio

| Situação | Status |
|---|---|
| Recurso criado com sucesso | 201 |
| Consulta ou atualização bem-sucedida | 200 |
| Remoção bem-sucedida | 204 |
| Parâmetro ausente, formato inválido ou regra de negócio violada | 400 |
| Recurso não encontrado (id inexistente) | 404 |

---

## 6. Cuidado com o tipo numérico

Alguns drivers JDBC retornam `NUMERIC` como `BigDecimal`. Na resposta JSON, serialize como número — não como string.

```java
// Use BigDecimal nos seus DTOs
private BigDecimal totalVendido;
private Long quantidadeVendas;
```

---

## 7. Estrutura em camadas

Organize o código assim — cada camada tem uma responsabilidade:

```
Controller   → recebe a requisição, valida os parâmetros, retorna a resposta HTTP
Service      → aplica as regras de negócio (validações, lógica)
Repository   → executa as queries no banco
```

Não deixe SQL no controller, e não deixe lógica de negócio no repository.

---

## 📚 Links úteis

### SQL e banco de dados
- [PostgreSQL — Funções de agregação](https://www.postgresql.org/docs/current/functions-aggregate.html) — documentação oficial do `SUM`, `COUNT`, `GROUP BY`
- [PostgreSQL — Tipos de data e hora](https://www.postgresql.org/docs/current/datatype-datetime.html) — como o banco lida com o tipo `DATE`
- [SQLZoo — Exercícios de GROUP BY](https://sqlzoo.net/wiki/SUM_and_COUNT) — pratique agregações com exercícios interativos

### Java
- [LocalDate — Documentação oficial (Java 17)](https://docs.oracle.com/en/java/docs/api/java.base/java/time/LocalDate.html) — parse e validação de datas
- [Spring Boot — Building a REST Service](https://spring.io/guides/gs/rest-service) — guia oficial para criar APIs REST com Spring
- [Baeldung — Spring REST Error Handling](https://www.baeldung.com/exception-handling-for-rest-with-spring) — como retornar erros 400 e 404 de forma centralizada
- [Baeldung — Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) — acesso ao banco com Spring

### HTTP
- [MDN — HTTP Status Codes](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Status) — referência completa dos códigos de status
- [REST API Design — Best Practices](https://restfulapi.net/http-methods/) — convenções de métodos e rotas REST

### Dados de teste
- [JavaFaker](https://github.com/DiUS/java-faker) — biblioteca para gerar nomes, e-mails e telefones fictícios no seed
- [Mockaroo](https://mockaroo.com) — gere arquivos CSV ou SQL com dados realistas pelo navegador, sem código

---

Boa sorte — e lembre-se: rode a query no banco antes de implementar o endpoint. 🚀
