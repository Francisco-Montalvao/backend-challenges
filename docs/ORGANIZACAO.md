# 🗂️ Organização de Pastas - Backend Challenges

## Estrutura Atual

```
backend-challenges/
│
├── README.md
├── CONTRIBUTING.md
├── LICENSE
├── desafio.md
├── docs/
│   ├── README.md
│   ├── INDEX.md
│   ├── ORGANIZACAO.md
│   ├── STRUCTURE.md
│   ├── ROADMAP.md
│   ├── TESTING.md
│   └── ...
│
├── junior/
│   ├── crud/
│   │   ├── todo-list/
│   │   └── gerenciador-produtos/
│   ├── autenticacao/
│   │   └── login-jwt/
│   └── relatorios/
│       └── vendas/
│
├── pleno/
│   └── mensageria/
│       └── fila-notificacoes/
│
├── senior/
│   └── microsservicos/
│       └── sistema-pedidos/
│
└── solucoes/
    └── README.md (galeria em breve)
```

---

## 📊 Quantificação

| Categoria | Quantidade |
|-----------|-----------|
| Desafios total | 6 |
| Quick Starts | 6 |
| Docker Compose | 6 |
| SOLUTIONS.md (por desafio) | 6 |
| Soluções publicadas em `/solucoes` | 0 (em breve) |

---

## 🎯 Padrão de Organização

### Por Desafio

Cada desafio segue este padrão mínimo:

```
<desafio>/
├── README.md          ← Enunciado completo
├── QUICKSTART.md      ← Setup em 2 minutos
├── SOLUTIONS.md       ← Links da comunidade
├── docker-compose.yml ← Infraestrutura base
└── dicas.md           ← (quando disponível)
```

### Por Solução

Quando a comunidade enviar soluções, cada pasta deve seguir:

```
solucoes/<desafio>/<linguagem>/
├── README.md          ← Como rodar + explicação
├── Dockerfile         ← (opcional)
├── docker-compose.yml ← (opcional)
├── src/               ← Código-fonte
└── tests/             ← Testes (se houver)
```

---

## 🚀 Fluxo de Navegação

### Usuário Novo
```
1. README.md (raiz)
2. docs/INDEX.md
3. Escolher um desafio
4. Ler QUICKSTART.md
5. Consultar SOLUTIONS.md do desafio
```

### Contribuidor
```
1. CONTRIBUTING.md
2. Escolher um desafio
3. Implementar solução
4. Criar pasta: solucoes/<desafio>/<linguagem>/
5. Atualizar SOLUTIONS.md e abrir PR
```

---

**Última atualização:** 21 de maio de 2026
