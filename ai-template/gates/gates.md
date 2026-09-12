## Gates de validacão para agentes de IA

[Gerar a partir do bloco E de `@../ask/questions.md`. Todo gate precisa de **comando executável**
e de **critério objetivo de aprovação**. Gate sem comando é intenção, não gate: se o
projeto ainda não tem o comando, registrar `TODO(descoberta)` em vez de inventar.]

[Este arquivo responde **como provar que a mudança está pronta**. As regras de quando
validar estão em `@../rules/RULES.md`, seção 12; o que o código não pode violar está em
`@../guardrails/`.]

---

### Quando cada gate é obrigatório

| Tipo de mudança                          | Gates obrigatórios                                     |
| ---------------------------------------- | ------------------------------------------------------ |
| Documentação apenas                      | Lint (se aplicável a documentação)                     |
| Correção de bug                          | Testes (com teste que reproduz o bug) · Lint · Build   |
| Funcionalidade nova                      | Testes · Lint · Build                                  |
| Refatoração sem mudança de comportamento | Testes existentes sem alteração · Lint · Build         |
| Mudança de schema ou migration           | Todos os gates · verificação de migração e de reversão |
| Mudança de dependência                   | Todos os gates · instalação limpa                      |

- `GATE-RULE-01` **MUST** — executar os gates aplicáveis **antes** de declarar a tarefa
  concluída, e ler a saída de cada um.
- `GATE-RULE-02` **MUST NOT** — declarar sucesso com base em inspeção visual do código.
- `GATE-RULE-03` **MUST NOT** — alterar, marcar como ignorado ou remover teste, regra de
  lint ou etapa de build para obter aprovação. {{gates.tests.mutation_policy}}
- `GATE-RULE-04` **MUST** — ao não conseguir executar um gate, dizer **qual**, **por quê**
  e **o que ficou sem verificação**. {{verification.on_failure}}

---

### Testes

**Comando:** `{{gates.tests.command}}`
**Aprovação:** suíte completa sem falha e sem teste ignorado novo.

- `GATE-01` **MUST** — toda mudança de comportamento tem teste que falha sem a
  implementação e passa com ela. {{gates.tests.policy}}
- `GATE-02` **MUST** — correção de bug começa por um teste que reproduz o bug.
  {{gates.tests.tdd}}
- `GATE-03` **MUST** — o teste verifica o comportamento observável, não a implementação
  interna; renomear um método privado não deve quebrar teste.
- `GATE-04` **MUST NOT** — teste depender de rede, relógio real, ordem de execução ou
  estado deixado por outro teste.
- `GATE-05` **MUST** — ao corrigir uma falha de teste, identificar a causa antes de mudar
  o código; ajustar a expectativa do teste só quando a expectativa é comprovadamente errada,
  e declarar isso.
- `GATE-06` **SHOULD** — cobertura mínima: {{gates.tests.coverage}}.
- Níveis exigidos por tipo de mudança: [unidade · integração · contrato · e2e — preencher]

### Build

**Comando:** `{{gates.build}}`
**Aprovação:** build concluído sem erro e sem aviso novo.

- `GATE-07` **MUST** — o build roda a partir de estado limpo quando a mudança toca
  dependências ou configuração.
- `GATE-08` **MUST NOT** — considerar aprovado um build que emite aviso novo introduzido
  pela mudança.
- `GATE-09` **MUST** — artefato gerado não entra no controle de versão, salvo onde o
  projeto já o faz deliberadamente.

### Lint

**Comando:** `{{gates.lint}}`
**Aprovação:** nenhum erro; avisos conforme a política do projeto.

- `GATE-10` **MUST** — código formatado pela ferramenta do projeto, sem formatação manual
  divergente.
- `GATE-11` **MUST NOT** — suprimir regra com comentário inline sem justificativa escrita e
  aprovada.
- `GATE-12` **MUST** — verificação de tipos, quando o projeto a tem, passa no modo
  configurado. {{stacks.typing}}

### Gates adicionais

[Gerar apenas os que o projeto tiver. Remover esta seção inteira quando não houver nenhum.]

- **Migrations:** comando de aplicação e de reversão; aprovação exige as duas direções.
- **Contrato/API:** validação da especificação contra a implementação.
- **Segurança:** auditoria de dependências e verificação de segredos.
- **Execução manual:** fluxo a exercitar quando o teste automatizado não cobre.

---

### Ordem de execução

{{gates.order}}

Ordem padrão, do mais barato ao mais caro: lint → build → testes → gates adicionais.
Parar na primeira falha bloqueante, corrigir e reexecutar **desde o início**.

---

### Evidência

{{gates.evidence}}

- `GATE-RULE-05` **MUST** — reportar, para cada gate: comando executado, resultado e o
  trecho relevante da saída em caso de falha.
- `GATE-RULE-06` **MUST NOT** — transcrever saída que não foi realmente produzida.
- Relatar isso diretamente na conclusão da tarefa (ver `@../rules/RULES.md`, seção 12).

---

### Definition of Done

Uma tarefa só está concluída quando **todos** os itens abaixo são verdadeiros:

- [ ] O escopo pedido está implementado por inteiro, ou o que faltou está declarado.
- [ ] Todos os gates aplicáveis foram executados e aprovados.
- [ ] Nenhum guardrail de `@../guardrails/` foi violado.
- [ ] Nenhuma decisão de nível `CONFIRMAR` foi tomada sem aprovação.
- [ ] Documentação afetada foi atualizada no mesmo trabalho.
- [ ] O diff não contém mudança fora do escopo.
- [ ] Não há código morto, log de depuração ou `TODO` não declarado.
- [ ] O que ficou sem verificação está explicitamente listado.

---

### Falhas pré-existentes

{{gates.preexisting_failures}}

- `GATE-RULE-07` **MUST** — distinguir falha introduzida pela mudança de falha já existente
  na base, comparando com o estado anterior.
- `GATE-RULE-08` **MUST** — reportar a falha pré-existente e seguir com a tarefa; corrigi-la
  é escopo novo e exige aprovação (`RULE-SCOPE-02`).
