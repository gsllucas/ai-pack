## Guardrails de API para agentes de IA

[Gerar a partir de `Q-D4`, `Q-D5` e `Q-D6`. Estilo do projeto: {{guardrails.api.style}}.
Ajustar cada item ao contrato real; remover o que não se aplica. Guardrails comuns a todas
as superfícies estão em `@./guardrails.md` e não devem ser repetidos aqui.]

### Contrato

- `GR-API-01` **MUST NOT** — alterar contrato público existente (rota, método, formato de
  entrada ou saída, código de status, semântica de campo) sem aprovação: é decisão de nível
  `CONFIRMAR` em `@../RULES.md`, seção 3.
- `GR-API-02` **MUST** — mudança compatível é aditiva: campo novo opcional, endpoint novo.
  Remoção, renomeação, mudança de tipo ou de obrigatoriedade é quebra.
  Processo de quebra neste projeto: {{guardrails.api.breaking_change}}
- `GR-API-03` **MUST** — manter a especificação do contrato sincronizada com a implementação
  no mesmo commit. [Indicar onde a especificação vive.]
- `GR-API-04` **MUST** — seguir o padrão de versionamento do projeto ao introduzir versão nova.

### Entrada e saída

- `GR-API-05` **MUST** — validar toda entrada na borda, incluindo tipo, faixa, tamanho e
  campos desconhecidos, antes de alcançar a camada de aplicação.
- `GR-API-06` **MUST** — usar o formato único de erro do projeto: mesma estrutura, mesmos
  códigos, mesma semântica de status. [Descrever o formato.]
- `GR-API-07` **MUST NOT** — retornar campo não declarado no contrato, nem vazar entidade
  interna ou coluna de banco diretamente na resposta.
- `GR-API-08` **MUST** — paginar toda listagem que pode crescer, com limite máximo imposto
  pelo servidor. {{guardrails.api.required}}
- `GR-API-09` **MUST** — impor limite de tamanho de payload e de profundidade de estrutura.

### Autenticação e autorização

- `GR-API-10` **MUST** — toda rota declara explicitamente se é pública ou protegida; rota
  nova sem declaração é tratada como protegida.
- `GR-API-11` **MUST** — verificar autorização sobre o recurso solicitado, não apenas a
  autenticação do chamador.
- `GR-API-12` **MUST NOT** — aceitar identificador de usuário, papel ou permissão vindo do
  corpo, da query ou de cabeçalho não verificado.

### Comportamento

- `GR-API-13` **MUST** — operação de escrita repetível é idempotente ou expõe mecanismo de
  idempotência.
- `GR-API-14` **MUST** — escrita que envolve mais de um efeito ocorre em transação ou tem
  compensação explícita.
- `GR-API-15` **MUST** — toda chamada a sistema externo tem timeout e comportamento definido
  para falha.
- `GR-API-16` **MUST** — propagar identificador de correlação da requisição até os logs.
- `GR-API-17` **MUST NOT** — executar trabalho longo de forma síncrona na requisição; usar
  worker quando exceder o limite definido em `@../skeletons/architecture.md`.
