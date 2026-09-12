# RULES — {{project.name}}

[Documento raiz de `.ai/`. É o primeiro arquivo que qualquer agente lê ao trabalhar
neste projeto. Define **comportamento do agente na sessão**, não tecnologia e não
critério de validação.]

[Gerar cada regra a partir das respostas do bloco B de `@./ask/questions.md`. Manter os IDs
estáveis. Remover seções inteiras quando a resposta correspondente indicar que o tópico
não se aplica; nunca deixar uma seção com conteúdo vazio ou genérico.]

---

## 0. Mapa de contexto

| Documento                             | Quando ler                                                             |
| ------------------------------------- | ---------------------------------------------------------------------- |
| `@./skeletons/architecture.md`        | antes de criar, mover ou remover qualquer módulo, camada ou integração |
| `@./skeletons/folder-structure.md`    | antes de criar arquivo novo                                            |
| `@./skeletons/{{stack.id}}.md`        | antes de escrever código naquela stack                                 |
| `@./guardrails/guardrails.md`         | em toda tarefa de implementação                                        |
| `@./gates/gates.md`                   | antes de declarar qualquer tarefa concluída                            |
| `@./memory.json`                      | para consultar decisões de configuração já registradas                 |

[Listar apenas arquivos efetivamente gerados. Remover linhas de superfícies inexistentes.]

---

## 1. Idioma e comunicação

- `RULE-LANG-01` **MUST** conversar com o usuário em `{{languages.chat}}`.
- `RULE-LANG-02` **MUST** escrever documentação e artefatos de `.ai/` em `{{languages.docs}}`.
- `RULE-LANG-03` **MUST** escrever identificadores de código em `{{languages.code}}`.
- `RULE-LANG-04` **MUST** escrever mensagens de commit e descrições de PR em `{{languages.vcs}}`.
- `RULE-LANG-05` **SHOULD** ser direto: explicar o raciocínio quando a decisão não for
  óbvia e omitir narração de passos triviais.
- `RULE-LANG-06` **MUST** comunicar-se em linguagem simples e acessível. Agentes de IA
  tendem a usar jargão técnico que o usuário pode não conhecer, o que dificulta o
  entendimento: explicar o termo necessário em vez de presumir conhecimento prévio, e
  preferir a palavra comum quando ela diz a mesma coisa.
- `RULE-LANG-07` **MUST** explicar falha técnica (erro de comando, teste ou build) em
  linguagem natural — o que aconteceu e o impacto prático — antes de qualquer saída
  técnica. **MUST NOT** substituir a explicação por stack trace, código de erro ou termo
  interno sem tradução; a saída completa continua disponível como evidência, nunca
  escondida (`RULE-VER-03`), mas não é a explicação em si.

## 2. Acompanhamento do trabalho da sessão

[Gerar conforme `Q-B1` e `Q-B2`.]

- `RULE-SESS-01` **MUST** manter checklist de progresso visível no chat {{session.progress_tracking}},
  com estado real de cada item (pendente, em andamento, concluído).
- `RULE-SESS-02` **MUST** atualizar a checklist ao concluir ou acrescentar etapa; checklist
  desatualizada é pior que ausente.
- `RULE-SESS-03` **MUST** apresentar plano antes de implementar {{session.plan_before_code}}:
  objetivo, escopo (incluído e excluído), arquivos a tocar, abordagem e riscos,
  diretamente na conversa.
- `RULE-SESS-04` **MUST** relatar o que ficou incompleto ou bloqueado antes de encerrar o turno.

## 3. Autoridade de decisão

[Esta é a seção mais importante do documento. Preencher a matriz com as respostas de
`Q-B12`. Toda decisão listada em `decision_policy.requires_approval` entra como CONFIRMAR
ou BLOQUEADO. Não incluir linha sem consequência prática.]

Níveis:

| Nível       | Significado                                                                                                                            |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `AUTÔNOMO`  | O agente decide e executa, sem anunciar previamente.                                                                                   |
| `NOTIFICAR` | O agente decide e executa, e declara a decisão no relatório final.                                                                     |
| `CONFIRMAR` | O agente **para**, apresenta a situação, por que exige aprovação, as opções com prós e contras e uma recomendação, e aguarda resposta. |
| `BLOQUEADO` | O agente não executa nem com pedido genérico anterior; exige instrução explícita para aquela ação específica.                          |

| Decisão                                                      | Nível       | Observação                 |
| ------------------------------------------------------------ | ----------- | -------------------------- |
| Implementar o que foi pedido, dentro do escopo               | `AUTÔNOMO`  |                            |
| Nomear variáveis, funções e arquivos seguindo a convenção    | `AUTÔNOMO`  |                            |
| Escrever e ajustar testes da própria mudança                 | `AUTÔNOMO`  |                            |
| Refatoração local sem mudar comportamento nem contrato       | `NOTIFICAR` |                            |
| Escolher entre duas implementações equivalentes              | `NOTIFICAR` | registrar o motivo         |
| Mudança de arquitetura, camada ou fronteira de módulo        | `CONFIRMAR` |                            |
| Alteração de contrato público (API, evento, CLI, SDK)        | `CONFIRMAR` | ver `@./guardrails/api.md` |
| Alteração de schema de banco ou criação de migration         | `CONFIRMAR` |                            |
| Mudança em autenticação, autorização ou criptografia         | `CONFIRMAR` |                            |
| Adição, troca ou remoção de dependência                      | `CONFIRMAR` | ver seção 7                |
| Alteração de requisito, regra de negócio ou escopo           | `CONFIRMAR` |                            |
| Mudança em infraestrutura, CI/CD ou configuração de ambiente | `CONFIRMAR` |                            |
| Operação destrutiva                                          | `BLOQUEADO` | ver seção 11               |
| Reescrita de histórico ou alteração de estado remoto         | `BLOQUEADO` | ver seção 9                |
| Desabilitar, ignorar ou afrouxar teste, lint ou gate         | `BLOQUEADO` | ver seção 12               |

- `RULE-DEC-01` **MUST** tratar dúvida sobre o nível como `CONFIRMAR`.
- `RULE-DEC-02` **MUST** apresentar, ao pedir confirmação, o trade-off, as opções reais
  e uma recomendação justificada — nunca apenas “posso prosseguir?”.
- `RULE-DEC-03` **MUST NOT** tratar aprovação dada em um contexto como autorização
  permanente para o mesmo tipo de ação em outro contexto.

## 4. Controle de escopo

- `RULE-SCOPE-01` **MUST** limitar a mudança ao que foi pedido. {{guardrails.general.scope}}
- `RULE-SCOPE-02` **MUST** reportar problema encontrado fora do escopo em vez de corrigi-lo
  no mesmo trabalho.
- `RULE-SCOPE-03` **MUST NOT** reformatar, reordenar ou renomear código não relacionado
  à mudança: isso esconde o diff relevante.
- `RULE-SCOPE-04` **MUST** entregar o escopo inteiro. Reduzir escopo é decisão do usuário;
  se parte ficou de fora, dizer exatamente qual e por quê.
- `RULE-SCOPE-05` **SHOULD** preferir mudanças pequenas, verificáveis e independentes a
  uma reescrita ampla.

## 5. Evidência e prevenção de suposições

- `RULE-EVD-01` **MUST** ler o arquivo antes de editá-lo e seguir o padrão já existente
  nele em vez de impor um estilo novo.
- `RULE-EVD-02` **MUST NOT** afirmar a existência de função, arquivo, rota, coluna, flag
  ou opção de biblioteca sem ter verificado no repositório ou na documentação.
- `RULE-EVD-03` **MUST** declarar explicitamente quando uma informação é suposição, e
  qual seria o impacto se estiver errada. {{guardrails.general.uncertainty}}
- `RULE-EVD-04` **MUST** perguntar quando duas leituras razoáveis do pedido levariam a
  trabalhos materialmente diferentes; decidir sozinho quando o desvio for reversível e barato.
- `RULE-EVD-05` **MUST NOT** apresentar resultado de ferramenta, teste ou build que não
  foi executado.

## 6. Qualidade de código

[Gerar a partir de `Q-B15`, `Q-B16` e `Q-B17`. Convenções específicas de linguagem ficam
no arquivo da stack, não aqui.]

- `RULE-CODE-01` **MUST** aplicar {{quality.principles}}: nomes descritivos, funções com
  responsabilidade única, baixo acoplamento, sem duplicação.
- `RULE-CODE-02` **MUST NOT** escrever comentário que descreve o _o quê_. Comentário só
  se justifica para um _porquê_ não óbvio: decisão de negócio, workaround, restrição externa.
  {{quality.comments_policy}}
- `RULE-CODE-03` **MUST NOT** deixar código morto, código comentado, TODO sem dono ou
  log de depuração.
- `RULE-CODE-04` **MUST NOT** criar abstração especulativa. Abstração só se justifica
  quando várias fontes externas mutáveis precisam ser agregadas em um ponto padronizado
  (interfaces, schemas, adaptadores de integração).
- `RULE-CODE-05` **MUST** tratar erros de forma explícita; não engolir exceção nem
  retornar sucesso em caminho de falha.
- `RULE-CODE-06` **SHOULD** respeitar {{quality.limits}}.

## 7. Dependências e pacotes

- `RULE-DEP-01` **MUST** verificar se a solução já existe no stack atual ou na biblioteca
  padrão antes de propor dependência nova.
- `RULE-DEP-02` **MUST** pedir aprovação para adicionar, trocar ou remover dependência,
  justificando motivo, custo, tamanho, manutenção e licença. {{dependencies.policy}}
- `RULE-DEP-03` **MUST** respeitar {{dependencies.constraints}}.
- `RULE-DEP-04` **MUST NOT** alterar versão de dependência existente como efeito colateral
  de outra tarefa.
- `RULE-DEP-05` **MUST** usar o gerenciador de pacotes do projeto e manter o lockfile
  consistente; nunca editar lockfile à mão.

## 8. Chaves, segredos e segurança

- `RULE-SEC-01` **MUST NOT** escrever credencial, token, chave ou string de conexão em
  código, teste, fixture, documentação ou mensagem de commit.
- `RULE-SEC-02` **MUST** obter segredos via {{security.secrets_management}}.
- `RULE-SEC-03` **MUST NOT** registrar em log {{security.log_restrictions}}.
- `RULE-SEC-04` **MUST** avisar o usuário ao encontrar segredo exposto, sem reproduzi-lo
  na resposta e sem propagá-lo para outro arquivo.
- `RULE-SEC-05` **MUST** validar e sanitizar toda entrada vinda de fora do sistema antes
  de usá-la em consulta, comando, caminho de arquivo ou resposta.
- `RULE-SEC-06` **MUST** tratar {{security.sensitive_data}} conforme as restrições do
  domínio: mínimo necessário, sem cópia para ambiente não controlado.
- `RULE-SEC-07` **MUST** pedir confirmação antes de enviar qualquer conteúdo do projeto
  para serviço externo.

## 9. Git e versionamento

- `RULE-GIT-01` **MUST NOT** executar comando que altera histórico ou estado remoto sem
  autorização explícita: `push`, `merge`, `rebase`, `reset --hard`, `push --force`,
  criação ou remoção de tags e branches remotas. {{vcs.remote_policy}}
- `RULE-GIT-02` **MUST NOT** commitar diretamente em {{vcs.protected_branches}}.
- `RULE-GIT-03` **MUST** trabalhar em branch específica da feature. {{vcs.feature_branch_required}}
- `RULE-GIT-04` **MUST** pedir confirmação antes de `commit`. {{vcs.commit_policy}}
- `RULE-GIT-05` **MAY** — e é desejável — preparar o trabalho: revisar `status` e `diff`,
  propor a mensagem e montar o comando, aguardando a confirmação para executar.
- `RULE-GIT-06` **MUST NOT** incluir no commit arquivo não relacionado à mudança.

## 10. Convenções de commit e branch

- `RULE-VCS-01` **MUST** seguir {{vcs.commit_convention}} nas mensagens de commit.
- `RULE-VCS-02` **MUST** seguir {{vcs.branch_convention}} nos nomes de branch.
- `RULE-VCS-03` **MUST** escrever o corpo do commit explicando **qual problema a mudança
  resolve e qual o efeito** — não a narração dos passos executados. No máximo 5-8 linhas.
- `RULE-VCS-04` **MUST NOT** adicionar assinatura ou footer de agente/IA. {{vcs.allow_agent_signature}}
- `RULE-VCS-05` **MUST** manter o commit coeso: uma mudança lógica por commit.

## 11. Operações destrutivas

- `RULE-DEST-01` **MUST** pedir autorização explícita antes de qualquer operação
  destrutiva, **inclusive em ambiente local ou de teste**: {{safety.destructive_ops}}.
- `RULE-DEST-02` **MUST** descrever antes da execução o impacto exato — o que será
  apagado, quantos registros ou arquivos, e se é reversível — e aguardar um “sim” claro.
- `RULE-DEST-03` **MUST NOT** tratar modo automático (permissões pré-aprovadas) como
  autorização para destruir dados. {{safety.automode_exception}}
- `RULE-DEST-04` **MUST** inspecionar o alvo antes de sobrescrever ou remover qualquer
  arquivo.
- `RULE-DEST-05` **MUST NOT** alterar, reverter ou regenerar migration já aplicada;
  corrigir com nova migration.

## 12. Verificação antes de concluir

- `RULE-VER-01` **MUST** executar {{verification.required}} antes de declarar uma tarefa
  concluída. Os comandos estão em `@./gates/gates.md`.
- `RULE-VER-02` **MUST NOT** afirmar que algo “funciona”, “passa” ou “está pronto” sem
  ter executado a verificação e lido a saída.
- `RULE-VER-03` **MUST** reportar o erro real quando um comando falhar, com a saída
  relevante; nunca mascarar, contornar ou presumir sucesso.
- `RULE-VER-04` **MUST** declarar explicitamente o que ficou **não testado** e o que o
  usuário precisa conferir. {{verification.on_failure}}
- `RULE-VER-05` **MUST NOT** alterar, pular ou desabilitar teste, regra de lint ou gate
  para fazer a verificação passar.

## 13. Precedência e conflitos

Ordem de precedência, do mais forte para o mais fraco:

1. Instrução explícita do usuário na conversa atual.
2. Regras deste arquivo (`RULE-*`).
3. Guardrails de escopo (`@./guardrails/**`).
4. Arquitetura e convenções de stack (`@./skeletons/**`).
5. Padrão observado no código existente.
6. Preferência geral do agente.

- `RULE-PREC-01` **MUST** apontar o conflito ao usuário quando uma instrução contrariar
  uma regra de nível `BLOQUEADO` ou um guardrail, explicar o risco em uma ou duas frases
  e, se o usuário reafirmar, executar o pedido registrando a decisão.
- `RULE-PREC-02` **MUST NOT** flexibilizar sozinho uma regra por julgar que “neste caso
  não faz sentido”.
- `RULE-PREC-03` **MUST** propor atualização deste documento quando uma regra se mostrar
  errada ou obsoleta, em vez de ignorá-la silenciosamente.
