## Arquitetura

[Descrição factual de como **este** sistema é organizado. Gerar a partir do bloco C de
`@../ask/questions.md`. Só entra aqui o que foi respondido pelo usuário ou verificado no
repositório. Camada ou componente não usado pelo projeto é removido, não descrito como
“não se aplica”.]

[Este arquivo descreve *o sistema*. Regras de comportamento do agente ficam em
`@../RULES.md`; restrições sobre o código produzido ficam em `@../guardrails/`; onde os
arquivos moram fica em `@./folder-structure.md`; como escrever em cada tecnologia fica
em `@./<stack>.md`.]

[Sempre considerar a arquitetura sugerida com divisão de responsabilidade e sem abstrações desnecessária]
[Abstração só deve ser considerada como necessidade quando várias fontes externas mutáveis precisam ser agregadas em único ponto padronizado, como interfaces, schemas e etc]

---

### 1. Visão geral

- **Sistema:** {{project.name}} — {{project.description}}
- **Domínio:** {{project.domain}}
- **Superfícies:** {{project.surfaces}}
- **Nível de complexidade adotado:** {{architecture.complexity}}
- **Estilo arquitetural:** [ex.: monólito em camadas, monólito modular, serviços
  independentes. Registrar o que existe hoje, não o que seria ideal.]

---

### 2. Camadas e responsabilidades

[Uma linha por camada realmente existente. “Pode depender de” é a regra que os agentes
vão verificar; preencher com nomes de camadas desta tabela.]

| Camada                      | Responsabilidade                                                                              | Pode depender de                   | Nunca contém                            |
| --------------------------- | --------------------------------------------------------------------------------------------- | ---------------------------------- | --------------------------------------- |
| Entrada (rotas/controllers) | Traduzir protocolo em chamada de caso de uso: validar entrada, autorizar, serializar resposta | domínio, aplicação                 | regra de negócio, acesso direto a banco |
| Domínio                     | Regras de negócio e invariantes do domínio                                                    | nada externo                       | SQL, HTTP, framework, ORM               |
| Aplicação / casos de uso    | Orquestrar domínio, transações e efeitos colaterais                                           | domínio, contratos de persistência | detalhe de protocolo                    |
| Persistência (repositórios) | Traduzir entre modelo de domínio e armazenamento                                              | domínio, driver/ORM                | regra de negócio                        |
| Rotinas (workers/jobs)      | Trabalho assíncrono, agendado ou em lote                                                      | aplicação, domínio                 | lógica duplicada da camada de entrada   |
| Infraestrutura              | Clientes de serviços externos, log, configuração, cache                                       | bibliotecas externas               | regra de negócio                        |
| Utilitários                 | Funções puras e reutilizáveis, sem estado                                                     | nada do projeto                    | acesso a I/O, regra de negócio          |

[Em projeto de complexidade `simples`, fundir Aplicação em Entrada e remover a linha —
camada sem responsabilidade própria é acoplamento, não organização.]

---

### 3. Direção de dependência

- {{architecture.dependency_direction}}
- Dependência permitida flui **para dentro**: entrada → aplicação → domínio. O domínio
  não conhece quem o chama nem onde os dados são gravados.
- Dependência proibida (violação bloqueia a entrega):
  - [listar as proibições concretas do projeto, ex.: “domínio importando o ORM”,
    “controller executando query”, “worker duplicando regra do caso de uso”]
- Comunicação entre módulos de mesmo nível: [direta · por contrato · por evento]

---

### 4. Política de abstração

- Criar interface/adaptador **MUST** ter justificativa: várias fontes externas mutáveis
  agregadas em um ponto padronizado (integrações, schemas, contratos).
- Uma única implementação sem previsão concreta de segunda **MUST NOT** virar interface.
- Abstração introduzida sem esse critério é dívida: aumenta indireção e esconde o fluxo.
- [Listar as abstrações que existem hoje e o motivo de cada uma.]

---

### 5. Componentes

[Para cada item marcado em `Q-C2`, preencher com a decisão real do projeto. Remover os
itens não utilizados.]

- **web server** — [framework e versão; onde o servidor é montado]
  - **rate limiter** — [estratégia, limite, chave de contagem, resposta ao exceder]
  - **cors** — [origens, métodos e cabeçalhos permitidos]
  - **middlewares** — [lista ordenada e o que cada um garante; ex.: request-id → log →
    auth → validação → handler → tratamento de erro]
  - **auth** — [biblioteca ou implementação própria; cookies, token ou sessão; onde a
    sessão é armazenada; tempo de expiração; como a autorização é verificada por rota]
  - **rotas** — [prefixo e versionamento; formato padrão de sucesso e de erro]
- **database** — [engine e versão; mais de um banco?]
  - **orm/query builder/sql** — [qual; quando SQL puro é aceito e onde ele pode morar]
  - **pool de conexões** — [tamanho, timeout, comportamento sob saturação]
  - **transações** — [quem abre e fecha transação; política de retry]
  - **migrations** — [ferramenta, nomenclatura, política de reversão]
- **persistence layer (repository)** — [granularidade: por agregado ou por tabela; o que
  o repositório retorna: entidade de domínio ou registro bruto]
- **controller (routes)** — [o que o controller pode fazer e o que é proibido]
- **loggers (observability)** — [biblioteca, formato, níveis, correlação de requisição;
  métricas e tracing quando existirem; ver seção 9]
- **workers (jobs)** — [fila ou agendador, concorrência, retry, idempotência, dead letter]
- **domínio (regras de negócio)** — [principais entidades, agregados e invariantes]
- **tests (gates e validade)** — [níveis existentes; comandos em `@../gates/gates.md`]
- **configuração** — [origem das variáveis, validação no boot, o que é obrigatório]
- **cache** — [onde, chave, TTL, política de invalidação]
- **mensageria/eventos** — [broker, contratos, garantia de entrega]

---

### 6. Fluxo fim a fim

[Descrever o caminho de uma operação representativa do sistema, nomeando os arquivos
reais envolvidos. Serve para o agente localizar onde intervir sem adivinhar.]

```
[ex.: requisição → middleware de auth → controller → caso de uso → domínio →
repositório → banco → resposta]
```

---

### 7. Integrações externas

| Sistema | Uso        | Contrato                                   | Falha / indisponibilidade              |
| ------- | ---------- | ------------------------------------------ | -------------------------------------- |
| [nome]  | [para quê] | [REST/gRPC/SDK, onde está a especificação] | [timeout, retry, fallback, degradação] |

---

### 8. Requisitos não funcionais

[Somente números fornecidos pelo usuário. Sem resposta, escrever `TODO(descoberta)` —
nunca estimar.]

| Requisito           | Alvo                               | Como é medido |
| ------------------- | ---------------------------------- | ------------- |
| Latência            | {{architecture.nfrs.latency}}      |               |
| Volume / throughput | {{architecture.nfrs.throughput}}   |               |
| Disponibilidade     | {{architecture.nfrs.availability}} |               |
| Limites de custo    | {{architecture.nfrs.cost}}         |               |

---

### 9. Observabilidade

- **Logs:** [formato, níveis, campos obrigatórios, correlação entre camadas]
- **Métricas:** [o que é medido e onde é exposto]
- **Tracing:** [se existe, qual propagação de contexto]
- **Erros:** [para onde vão; o que caracteriza erro de domínio vs de infraestrutura]
- Toda operação nova **MUST** ser observável no mesmo padrão das existentes.
- O que nunca pode ser logado está em `@../RULES.md` (`RULE-SEC-03`).

---

### 10. Decisões vigentes

[Registro direto das decisões de nível `CONFIRMAR` aprovadas, sem arquivo separado nem
numeração própria: cada decisão vira uma linha nesta tabela, editada no próprio documento.]

| Decisão   | Data         | Status                | Contexto              |
| --------- | ------------ | --------------------- | --------------------- |
| [decisão] | [AAAA-MM-DD] | vigente / substituída | [motivo em uma linha] |

- Decisão registrada é o estado atual do projeto: **MUST NOT** ser contrariada sem nova
  aprovação que atualize esta tabela.

---

### 11. Como estender

[Playbooks curtos, com a sequência exata de passos e os arquivos tocados. Gerar apenas
os playbooks aplicáveis ao projeto.]

- **Nova rota/endpoint:** [passos]
- **Novo caso de uso:** [passos]
- **Nova entidade ou tabela:** [passos, incluindo migration e repositório]
- **Nova integração externa:** [passos, incluindo tratamento de falha]
- **Novo worker/job:** [passos, incluindo idempotência]

---

### 12. Invariantes arquiteturais

[Cada item é verificável e bloqueia a conclusão quando violado. Não repetir guardrails
gerais — referenciar por ID.]

- `MUST` [ex.: o domínio não importa nada de infraestrutura]
- `MUST` [ex.: toda escrita passa por repositório]
- `MUST` [ex.: todo endpoint valida entrada antes de chamar o caso de uso]
- `MUST NOT` [ex.: lógica de negócio em controller ou em migration]

Violação de invariante exige `CONFIRMAR` e atualização da tabela de decisões (seção 10)
— ver `@../RULES.md`, seção 3.
