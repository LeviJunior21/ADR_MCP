# ADR: Suporte a Requisições em Lote com JSON-RPC 2.0 (Batch Requests)

## Status

**Decidido e Implementado**  
PR [#416](https://github.com/modelcontextprotocol/modelcontextprotocol/pull/416)  
**Data**: 2025-03-26

---

## Contexto

Para sistemas que utilizam JSON-RPC 2.0 (como o Model Context Protocol - MCP), é comum a necessidade de realizar múltiplas chamadas para diferentes métodos RPC em sequência.  
Realizar essas chamadas individualmente introduz latência e sobrecarga de rede.

A especificação JSON-RPC 2.0 define um mecanismo oficial de **requisições em lote (batch)**, que permite ao cliente enviar várias chamadas RPC em um único payload JSON.

---

## Decisão

Adotou-se o suporte à funcionalidade de **batch requests**, conforme descrito na [especificação oficial JSON-RPC 2.0](https://www.jsonrpc.org/specification#batch).

### Exemplo de requisição em lote:

```json
[
    { "jsonrpc": "2.0", "method": "sum", "params": [1, 2, 3], "id": 1 },
    { "jsonrpc": "2.0", "method": "notify_hello", "params": [7] },
    { "jsonrpc": "2.0", "method": "subt", "params": [42, 23], "id": 2 }
]
```

### Exemplo de resposta:

```json
[
    { "jsonrpc": "2.0", "result": 6, "id": 1 },
    { "jsonrpc": "2.0", "result": 19, "id": 2 }
]
```

> Notificações (sem `id`) não recebem resposta, conforme a especificação.

---

## Consequências

### Positivas

- Redução de latência em cenários com múltiplas chamadas RPC.
- Menos overhead de conexão (principalmente em ambientes HTTP/1.1).
- Compatível com implementações padronizadas de JSON-RPC 2.0.
- Suporte a paralelismo e agregação de chamadas assíncronas.

### Negativas / Cuidados

- A ordem das respostas **não é garantida**. Clientes devem usar `id` para correlacionar.
- Requisições batch **não podem ser arrays vazios (`[]`)** — isso é inválido.
- O servidor precisa distinguir entre **notificações** e **chamadas normais**.
- Deve-se limitar o tamanho de batches para evitar abuso  
  (ex: `maxBatchSize = 50`).

---

## Alternativas Consideradas Rejeitadas

- **Realizar chamadas JSON-RPC individualmente**  
  Rejeitado por causa de latência acumulada e overhead de rede.

- **Implementar agregação personalizada fora do protocolo**  
  Rejeitado por romper compatibilidade com clientes JSON-RPC existentes.

---

## Ações Adicionais

- Implementar **testes automatizados** para validar o parsing e roteamento de batches.
- Incluir suporte a **tool calls dentro de lotes**, respeitando a semântica do protocolo.
- Expor erro padronizado para lote inválido:

```json
{
    "jsonrpc": "2.0",
    "error": {
        "code": -32600,
        "message": "Invalid Request"
    },
    "id": null
}
```

---

## Referências

- [JSON-RPC 2.0 Specification](https://www.jsonrpc.org/specification#batch)
