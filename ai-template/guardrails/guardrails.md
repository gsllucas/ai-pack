## Referência de Guardrails

[Índice dos guardrails. Gerar apenas as linhas das superfícies marcadas em `Q-A8`:
arquivo de superfície inexistente não é gerado nem referenciado.]

api: @./api.md
frontend: @./frontend.md
mobile: @./mobile.md
ux-ui: @./ux-ui.md

[Arquivos desta pasta]

---

### O que é um guardrail

Guardrail é uma **restrição verificável sobre o código produzido**: é possível olhar um
diff e dizer se foi violada ou não.

| Documento            | Governa                                              |
| -------------------- | ---------------------------------------------------- |
| `@../RULES.md` | o comportamento do agente na sessão                  |
| `guardrails/**`      | o artefato que o agente produz                       |
| `@../gates/gates.md` | como provar objetivamente que o artefato está pronto |

Regra que não pode ser verificada em um diff não é guardrail — ou vira gate, ou vira
regra de comportamento, ou é descartada.

### Como aplicar

1. Antes de implementar, ler este arquivo e o da superfície envolvida.
2. Ao identificar que a mudança pedida viola um guardrail: **parar antes de escrever o
   código**, explicar qual guardrail e por quê, e propor a alternativa que respeita a regra.
3. Exceção a guardrail é decisão de nível `CONFIRMAR` (`@../RULES.md`, seção 3): exige
   aprovação explícita e registro do motivo no plano de mudança.
4. Violação descoberta em código **já existente** é reportada, não corrigida junto de
   outra tarefa (`RULE-SCOPE-02`).

---

## Guardrails Gerais

[Escreva guardrails de IA gerais que são de uso comum para agentes de LLM de IA aqui]

[Gerar a partir de `Q-D1`, `Q-D2` e `Q-D3`, ajustando cada item à realidade do projeto.
Remover o que não se aplica; não inflar a lista.]

### Consistência com o projeto

- `GR-GEN-01` **MUST** — código novo deve ser indistinguível do existente em estilo,
  estrutura e nomenclatura. O padrão do projeto vence a preferência do agente.
- `GR-GEN-02` **MUST NOT** — introduzir um segundo padrão para um problema que o projeto
  já resolve de uma forma (ex.: segunda biblioteca de validação, segundo formato de erro).
- `GR-GEN-03` **MUST** — respeitar as invariantes de `@../skeletons/architecture.md`,
  seção 12, e a direção de dependência da seção 3.
- `GR-GEN-04` **MUST NOT** — alterar sem aprovação: {{guardrails.general.frozen}}.

### Verificação e suposições

- `GR-GEN-05` **MUST NOT** — chamar função, rota, campo, tabela ou opção de biblioteca
  cuja existência e assinatura não tenham sido verificadas no repositório ou na documentação.
- `GR-GEN-06` **MUST NOT** — inventar comportamento de dependência externa para fazer o
  código “fechar”. Sem confirmação, parar e perguntar. {{guardrails.general.uncertainty}}
- `GR-GEN-07` **MUST** — quando houver mais de uma interpretação razoável do requisito,
  implementar a mais conservadora e declarar a escolha.

### Correção e testabilidade

- `GR-GEN-08` **MUST** — toda mudança de comportamento vem acompanhada de teste que falha
  sem a mudança. Política completa em `@../gates/gates.md`.
- `GR-GEN-09` **MUST** — código novo é testável sem infraestrutura real: dependências
  externas entram por parâmetro ou construtor.
- `GR-GEN-10` **MUST NOT** — introduzir estado global mutável ou singleton novo.
- `GR-GEN-11` **MUST** — tratar explicitamente o caminho de erro; nenhuma falha pode ser
  silenciosamente convertida em sucesso ou em valor vazio.

### Segurança

- `GR-GEN-12` **MUST** — validar e normalizar toda entrada vinda de fora do sistema na
  borda, antes de qualquer uso.
- `GR-GEN-13` **MUST NOT** — concatenar entrada externa em query, comando de shell,
  caminho de arquivo ou template sem o mecanismo de escape/parametrização do projeto.
- `GR-GEN-14` **MUST NOT** — expor dado sensível, segredo, stack trace ou detalhe interno
  em log, mensagem de erro ou resposta.
- `GR-GEN-15` **MUST** — escrever toda mensagem de erro voltada ao usuário em linguagem
  natural, dizendo o que aconteceu e, quando possível, o que fazer. **MUST NOT** repassar
  ao usuário exceção, código interno, query ou jargão técnico sem tradução.
- `GR-GEN-16` **MUST** — verificar autorização no ponto de acesso ao recurso, não apenas
  na interface que o oferece.

### Dados e reversibilidade

- `GR-GEN-17` **MUST NOT** — escrever código que apague ou sobrescreva dado sem filtro
  explícito e sem caminho de reversão.
- `GR-GEN-18` **MUST** — migration destrutiva ou irreversível exige aprovação e plano de
  rollback declarado antes da execução.
- `GR-GEN-19` **MUST** — operação que pode ser reexecutada (retry, worker, webhook) é
  idempotente.

### Performance

- `GR-GEN-20` **MUST NOT** — introduzir trabalho ilimitado: consulta sem limite ou índice,
  iteração sobre coleção sem paginação, chamada externa sem timeout, retry sem teto.
- `GR-GEN-21` **MUST NOT** — consultar dentro de laço o que pode ser resolvido em uma
  operação (N+1).

### Observabilidade e documentação

- `GR-GEN-22` **MUST** — operação nova é observável no mesmo padrão das existentes
  (`@../skeletons/architecture.md`, seção 9).
- `GR-GEN-23` **MUST** — mudança que altera contrato, configuração, comando ou fluxo
  atualiza a documentação correspondente no mesmo trabalho.
- `GR-GEN-24` **MUST** — decisão relevante tomada durante a implementação aparece no
  plano de mudança ou na tabela de decisões de `@../skeletons/architecture.md`,
  seção 10; não fica apenas no chat.

### Escopo

- `GR-GEN-25` **MUST NOT** — incluir no diff mudança não pedida: renomeação, reformatação,
  refatoração oportunista ou atualização de dependência. {{guardrails.general.scope}}
- `GR-GEN-26` **MUST NOT** — deixar `TODO`, código comentado ou implementação parcial sem
  declarar explicitamente no relatório final.
