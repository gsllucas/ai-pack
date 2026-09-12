# Condução de perguntas ao usuário

Roteiro de descoberta do framework. O agente gerador executa este documento **antes**
de gerar qualquer arquivo de `.ai/`. O protocolo de geração está em `@../README.md`.

<critical>DIVIDIR OS TRECHOS DE PERGUNTAS EM SESSÕES</critical>
<critical>TODAS AS PERGUNTAS DEVEM OFERECER ESCOLHAS RECOMENDADAS AO USUÁRIO</critical>
<critical>AS PERGUNTAS DEVEM OFERECER OPÇÕES EM MÚLTIPLA ESCOLHA, QUANDO POSSÍVEL DE ACORDO COM O ESCOPO DA PERGUNTA, AO USUÁRIO.</critical>

---

## Protocolo de execução

1. **Uma sessão por vez.** Os blocos A → E são apresentados em sequência. O agente só
   avança para o próximo bloco após confirmar o resumo do bloco atual com o usuário.
2. **No máximo 4 perguntas por rodada.** Perguntas longas cansam e produzem respostas
   piores que defaults bem escolhidos.
3. **Toda pergunta oferece opções** com uma marcada como _recomendada_, e a recomendação
   é justificada em uma linha. Perguntas abertas só para nome, descrição e domínio.
4. **Nunca perguntar o que já é observável.** Ver _Regras de inferência_ abaixo.
5. **Nunca perguntar o que já foi respondido.** Antes de cada bloco, reler as respostas
   acumuladas; se uma resposta anterior já determina a próxima pergunta, registrar a
   dedução e apenas confirmar.
6. **Registrar incrementalmente.** Ao fim de cada bloco, gravar as respostas em
   `memory.json` com a origem (`interview`, `inferred`, `default`).
7. **Não decidir pelo usuário em ponto de impacto.** Escolha de arquitetura, banco,
   política de testes e política de aprovação humana são do usuário; o agente recomenda.
8. **Não sei / depois** é uma resposta válida: vai para `memory.json` (campo
   `open_questions`) e vira `TODO(descoberta)` no documento gerado. Nunca preencher com
   suposição.
9. **Linguagem simples, sem jargão.** Agentes de IA tendem a usar termos técnicos que o
   usuário pode não conhecer, o que dificulta o entendimento. Explicar o termo necessário
   em vez de presumir conhecimento prévio; preferir a palavra comum quando ela diz a
   mesma coisa.

### Regras de inferência

Antes do bloco A, inspecionar o repositório e **pré-preencher** respostas. Apresentar
como confirmação (“detectei X, confirma?”), nunca como pergunta em branco.

| Sinal no repositório                                 | Inferir                                            |
| ---------------------------------------------------- | -------------------------------------------------- |
| Arquivos de manifesto de dependências                | linguagem, runtime, gerenciador de pacotes         |
| Dependências declaradas                              | frameworks, ORM/driver, libs de teste, libs de log |
| Scripts de build/test/lint declarados                | comandos dos gates                                 |
| Configuração de CI                                   | gates obrigatórios e ordem de execução             |
| Estrutura de diretórios existente                    | arquitetura e estrutura de pastas atual            |
| Arquivos de lint/format/tipagem                      | convenções de código                               |
| Migrations, schema, docker-compose                   | banco de dados e serviços de apoio                 |
| Histórico de commits                                 | convenção de commit e branch em uso                |
| `CLAUDE.md`, `AGENTS.md`, `.cursorrules` e similares | regras já estabelecidas — importar, não recriar    |

<critical>REGRA EXISTENTE NO REPOSITÓRIO TEM PRECEDÊNCIA SOBRE SUGESTÃO DO FRAMEWORK. EM CONFLITO, PERGUNTAR QUAL PREVALECE.</critical>

<critical>AO FINAL DA GERAÇÃO, IDENTIFICAR O ARQUIVO RAIZ DE INSTRUÇÃO DO AGENTE DO PROJETO (CLAUDE.MD, AGENTS.MD, .CURSORRULES OU EQUIVALENTE) E GARANTIR QUE ELE REFERENCIA `.ai/RULES.md` COMO PONTO DE ENTRADA, SEM SOBRESCREVER O CONTEÚDO EXISTENTE. SE NENHUM EXISTIR, PERGUNTAR AO USUÁRIO SE DEVE SER CRIADO UM ARQUIVO MÍNIMO SÓ COM ESSA REFERÊNCIA.</critical>

<critical>DOCUMENTO DE SPEC, PRD OU TECH-SPEC DE FEATURE (EX.: PASTA `specs/`) NÃO É FONTE DE REGRA, ARQUITETURA OU GUARDRAIL. SPEC TRATA DE FUNCIONALIDADE, EM ESCOPO ISOLADO; ESTE FRAMEWORK GERA PADRÃO DE PROJETO, NÃO REQUISITO DE FEATURE.</critical>

---

## A. Perguntas iniciais e idiomas

Obs.: Serve para criação do arquivo de `.ai/memory.json`.

| ID     | Pergunta                                                                                   | Opções sugeridas                                                      | Recomendado                              | Destino                               |
| ------ | ------------------------------------------------------------------------------------------ | --------------------------------------------------------------------- | ---------------------------------------- | ------------------------------------- |
| `Q-A1` | Em qual idioma escrever os documentos que serão gerados?                                   | pt-BR · en-US · outro                                                 | idioma em que o usuário está conversando | `languages.docs`                      |
| `Q-A2` | Em qual idioma o agente de IA deve conversar com o usuário?                                | pt-BR · en-US · outro                                                 | mesmo de `Q-A1`                          | `languages.chat`                      |
| `Q-A3` | Em qual idioma o agente de IA deve escrever código (nomes de variáveis, funções, classes)? | en-US · igual ao domínio do negócio                                   | en-US                                    | `languages.code`                      |
| `Q-A4` | Em qual idioma escrever mensagens de commit e descrições de PR?                            | igual a `Q-A1` · en-US                                                | igual a `Q-A1`                           | `languages.vcs`                       |
| `Q-A5` | Nome e descrição do projeto em uma frase.                                                  | aberta                                                                | —                                        | `project.name`, `project.description` |
| `Q-A6` | Qual o domínio de negócio e quem são os usuários finais?                                   | aberta                                                                | —                                        | `project.domain`, `project.users`     |
| `Q-A7` | Qual o estágio do projeto?                                                                 | greenfield · em desenvolvimento · em produção · legado em manutenção  | detectar pelo histórico                  | `project.maturity`                    |
| `Q-A8` | Quais superfícies o projeto possui? (múltipla escolha)                                     | API/backend · frontend web · mobile · CLI · worker/batch · biblioteca | detectar pelo repositório                | `project.surfaces`                    |

A resposta de `Q-A8` determina **quais arquivos de guardrail serão gerados** e quais
blocos de perguntas de D são aplicáveis. Superfície não marcada não gera arquivo e não
gera pergunta.

<critical>ESCREVER TODA A GENERAÇÃO DOS DOCUMENTOS DE ACORDO COM A RESPOSTA DO USUÁRIO: "EM QUAL IDIOMA ESCREVER OS DOCUMENTOS QUE SERÃO GERADOS?"</critical>
<critical>O AGENTE DE IA DEVE SE COMUNICAR NA SESSÃO DE CHAT COM BASE NA RESPOSTA ESCOLHIDA PELO USUÁRIO NA PERGUNTA: "EM QUAL IDIOMA O AGENTE DE IA DEVE CONVERSAR COM O USUÁRIO?"</critical>

---

## B. Perguntas sobre regras e princípios gerais

Obs.: Serve para criação do arquivo de `.ai/RULES.md`.

<critical>PERGUNTAS NECESSÁRIAS PARA CRIAÇÃO DO `RULES.MD`</critical>
<critical>CADA BULLET LIST É UM TÓPICO QUE DEVE SER CRIADO NO ARQUIVO DE RULES.MD</critical>

- **Acompanhamento e andamento do trabalho da sessão**
  - `Q-B1` O agente deve manter checklist de progresso visível no chat? _(sim, sempre — recomendado · só em tarefas multi-etapa · não)_ → `session.progress_tracking`
  - `Q-B2` O agente deve apresentar plano antes de implementar? _(sempre para mudanças multi-arquivo — recomendado · sempre · nunca)_ → `session.plan_before_code`

- **Segurança com Git e versionamento**
  - `Q-B3` O agente pode executar `commit` sem pedir? _(não, sempre confirmar — recomendado · sim em branch de feature · sim sempre)_ → `vcs.commit_policy`
  - `Q-B4` O agente pode executar `push`, `merge`, `rebase`, reescrita de histórico? _(nunca sem autorização explícita — recomendado · apenas push em branch própria)_ → `vcs.remote_policy`
  - `Q-B5` Quais são as branches protegidas? _(detectar default do repositório; ex.: `main`, `master`, `develop`)_ → `vcs.protected_branches`
  - `Q-B6` Trabalho deve ocorrer sempre em branch específica da feature? _(sim — recomendado · não)_ → `vcs.feature_branch_required`

- **Convenções de commit e branch (conventional commits e conventional branches)**
  - `Q-B7` Padrão de commit _(Conventional Commits — recomendado · padrão próprio · livre)_ → `vcs.commit_convention`
  - `Q-B8` Padrão de nome de branch _(`tipo/descricao-curta` — recomendado · com ID de ticket · livre)_ → `vcs.branch_convention`
  - `Q-B9` Assinaturas/footers de agente são permitidos no commit? _(não — recomendado · sim)_ → `vcs.allow_agent_signature`

- **Operações destrutivas**
  - `Q-B10` Quais operações exigem confirmação explícita, mesmo em ambiente local? _(múltipla escolha: DDL destrutivo · `DELETE`/`UPDATE` sem `WHERE` · reset/squash de migrations · remoção de arquivos em massa · remoção de volumes e imagens · sobrescrita de `.env`, secrets e dumps · reset de estado remoto)_ → `safety.destructive_ops`
  - `Q-B11` Modo automático (sem prompts de permissão) altera essa política? _(não, nunca — recomendado · sim)_ → `safety.automode_exception`

- **Pergunta sobre decisões arquiteturais e importantes**
  - `Q-B12` Quais decisões exigem aprovação humana? _(múltipla escolha: mudança de arquitetura ou de camada · alteração de contrato público/API · schema e migrations · autenticação, autorização e criptografia · nova dependência · troca de infraestrutura · mudança de requisito · alteração de regra de negócio)_ → `decision_policy.requires_approval`

- **Qualidade de código**
  - `Q-B15` Princípios obrigatórios _(múltipla escolha: Clean Code · responsabilidade única · evitar comentários que descrevem o “o quê” · sem código morto · sem `console.log`/`print` de depuração · sem over-engineering · mudanças cirúrgicas)_ → `quality.principles`
  - `Q-B16` Política de comentários _(só explicar “porquê” não óbvio — recomendado · livre · documentação formal obrigatória em API pública)_ → `quality.comments_policy`
  - `Q-B17` Limites objetivos de complexidade? _(sem limite numérico — recomendado · definir limite de linhas/complexidade por função)_ → `quality.limits`

- **Dependências e pacotes**
  - `Q-B18` O agente pode adicionar dependências? _(não sem aprovação — recomendado · sim se já estiver no lockfile · sim)_ → `dependencies.policy`
  - `Q-B19` Restrições de licença ou de fornecedor? _(nenhuma · lista de licenças permitidas · aberta)_ → `dependencies.constraints`

- **Chaves, segredos e segurança**
  - `Q-B20` Como segredos são gerenciados no projeto? _(variáveis de ambiente · cofre de segredos · outro)_ → `security.secrets_management`
  - `Q-B21` Há dados sensíveis/regulados no domínio? _(múltipla escolha: dados pessoais · dados financeiros · dados de saúde · credenciais de terceiros · nenhum)_ → `security.sensitive_data`
  - `Q-B22` Regras de log: o que nunca pode ser logado? _(segredos, tokens, PII — recomendado · aberta)_ → `security.log_restrictions`

- **Verificação antes de concluir**
  - `Q-B23` O que o agente deve executar antes de declarar uma tarefa concluída? _(múltipla escolha: build · lint · testes · type-check · execução manual do fluxo)_ → `verification.required`
  - `Q-B24` Como reportar quando não for possível validar? _(declarar explicitamente o que ficou não testado — recomendado · outro)_ → `verification.on_failure`

---

## C. Arquitetura, estrutura de pastas e tecnologias stack principal

Obs.: Serve para criação do arquivo de `.ai/skeletons/architecture.md` e `.ai/skeletons/folder-structure.md`.

<critical>SEMPRE CONSIDERAR A ARQUITETURA SUGERIDA COM DIVISÃO DE RESPONSABILIDADEs E SEM ABSTRAÇÕES DESNECESSÁRIAs</critical>
<critical>ABSTRAÇCÃO SÓ DEVE SER CONSIDERADA QUANDO VÁRIAS FONTES EXTERNAS MUTÁVEIS PRECISAM SER AGREGADAS EM ÚNICO PONTO PADRONIZADO, COMO INTERFACES, SCHEMAS E ETC</critical>

### Arquitetura sugerida, perguntar ao usuário nível de complexidade da aplicação. E o que será necessário para construir a aplicação.

`Q-C1` **Nível de complexidade da aplicação** → `architecture.complexity`

| Nível    | Quando                                                            | Consequência na geração                                                                 |
| -------- | ----------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| simples  | CRUD, poucas regras, um consumidor                                | camadas mínimas: rota → serviço → persistência. Sem interface por dependência.          |
| moderado | regras de negócio relevantes, integrações externas                | domínio isolado, repositórios, workers quando houver trabalho assíncrono                |
| complexo | múltiplos contextos, alto acoplamento externo, requisitos rígidos | fronteiras explícitas entre contextos, contratos versionados, abstração nas integrações |

<critical>NÃO GERAR CAMADA QUE O NÍVEL ESCOLHIDO NÃO JUSTIFICA. CAMADA SEM RESPONSABILIDADE PRÓPRIA É ACOPLAMENTO, NÃO ORGANIZAÇÃO.</critical>

`Q-C2` **O que será necessário para construir a aplicação?** Múltipla escolha sobre a
lista abaixo; para cada item marcado, fazer as perguntas de detalhe indicadas.

- web server
  - rate limiter — _precisa? limite por quê (IP, usuário, chave)?_
  - cors — _quais origens permitidas?_
  - middlewares — _quais transversais: request-id, auth, validação, tratamento de erro?_
  - auth (packages, hand-made) + cookies + sessions — _biblioteca ou implementação própria? sessão em cookie, token ou servidor? onde a sessão é armazenada?_
  - rotas — _versionamento de rota? padrão de resposta de erro?_
- database
  - orms/query builders/sql — _qual? SQL puro é permitido? onde?_
  - configurações de pools (performance) — _limites de conexão, timeout_
  - relational databases, nosql databases — _qual engine, qual versão, mais de um?_
- persistence layer (repository) — _repositório por agregado ou por tabela? transações controladas por quem?_
- controller (routes) — _controller pode conter regra de negócio? (recomendado: não)_
- loggers (observability) — _biblioteca, formato, níveis, correlação de requisição, métricas e tracing_
- workers (jobs) — _fila, agendamento, política de retry e idempotência_
- domínio (regras de negócio) — _o domínio pode depender de framework/ORM? (recomendado: não a partir de "moderado")_
- tests (gates e validade) — _níveis exigidos: unidade, integração, contrato, e2e_
- [sugerir conforme boas práticas]
  - configuração e variáveis de ambiente — _validação de config no boot?_
  - tratamento de erros — _erros de domínio vs erros de infraestrutura_
  - cache — _precisa? onde, com qual invalidação?_
  - mensageria/eventos — _síncrono ou assíncrono entre módulos?_
  - migrations — _ferramenta e política de reversão_

`Q-C3` **Direção de dependência permitida** _(domínio no centro, sem depender de
infraestrutura — recomendado a partir do nível moderado · dependência livre entre camadas)_
→ `architecture.dependency_direction`

`Q-C4` **Requisitos não funcionais com número** — latência alvo, volume esperado,
disponibilidade, janela de manutenção, limites de custo. Sem número, registrar como
ausente em vez de inventar. → `architecture.nfrs`

`Q-C5` **Integrações externas** — quais sistemas de terceiros, qual o contrato, o que
acontece quando ficam indisponíveis. → `architecture.integrations`

`Q-C6` **Stacks do projeto** — uma entrada por tecnologia principal (ex.: `node`,
`react-native`, `postgres`). Cada entrada gera `.ai/skeletons/<stack>.md` a partir de
`@../skeletons/stack-template.md`. Perguntar por stack: versão do runtime, gerenciador
de pacotes, frameworks e libs principais, ferramenta de teste, comandos de build/run,
convenção de nomenclatura (camelCase, PascalCase, snake_case, kebab-case) por tipo de
símbolo. → `stacks[]`

### Estrutura de pastas

Estrutura de pastas sugerida. Fazer perguntas ao usuário:

- camada de domínio e regras de negócio
- camada de persistência de dados
- camada de rotas
- camada de códigos úteis e reusáveis (utils)
- camada de rotinas (workers, jobs)

`Q-C7` Manter a estrutura já existente no repositório ou adotar a sugerida? _(manter a
existente quando já houver código — recomendado · adotar a sugerida)_ → `folders.strategy`

`Q-C8` Organização por camada técnica ou por funcionalidade/módulo? _(por camada em
projetos simples · por funcionalidade a partir de moderado — recomendado)_ → `folders.organization`

`Q-C9` Convenção de nomenclatura de arquivos e diretórios _(kebab-case — recomendado ·
snake_case · camelCase · PascalCase para componentes)_ → `folders.naming`

`Q-C10` Onde ficam os testes? _(ao lado do código — recomendado · diretório separado)_ → `folders.tests_location`

### Escrita de código

- Pergunta sobre convenção de escrita: camel, pascal, etc.
  - `Q-C11` Convenção por tipo de símbolo (variável, função, classe, constante, tipo,
    arquivo, tabela/coluna). Default por stack, confirmado pelo usuário. → `stacks[].naming`
- Clean code, convenções e etc
  - `Q-C12` Formatador e linter em uso, com configuração existente? → `stacks[].tooling`
  - `Q-C13` Tipagem estática obrigatória? Modo estrito? → `stacks[].typing`

---

## D. Perguntas sobre guardrails

Obs.: Serve para criação do arquivo de `.ai/guardrails/**/.md`.

Perguntar apenas os blocos correspondentes às superfícies marcadas em `Q-A8`.

- **Guardrails gerais (compartilhados e do escopo comum)**
  - `Q-D1` O agente pode alterar arquivos fora do escopo pedido? _(não, exige nova aprovação — recomendado · sim para correções triviais)_ → `guardrails.general.scope`
  - `Q-D2` O que é proibido tocar sem aprovação? _(múltipla escolha: migrations aplicadas · contratos públicos · configuração de infraestrutura · pipeline de CI · arquivos gerados · dependências)_ → `guardrails.general.frozen`
  - `Q-D3` Como o agente deve lidar com informação que não conseguiu verificar? _(declarar incerteza e perguntar — recomendado · assumir o caso mais provável)_ → `guardrails.general.uncertainty`
- **API**
  - `Q-D4` Estilo e contrato _(REST · GraphQL · gRPC · outro)_; há especificação versionada? → `guardrails.api.style`
  - `Q-D5` Mudança compatível vs quebra: o que caracteriza quebra neste projeto e qual o processo? → `guardrails.api.breaking_change`
  - `Q-D6` Padrões obrigatórios _(múltipla escolha: validação de entrada na borda · formato único de erro · paginação · autenticação e autorização por rota · idempotência em escrita · limites de payload)_ → `guardrails.api.required`
- **Frontend**
  - `Q-D7` Padrão de estado, de dados remotos e de composição de componentes → `guardrails.frontend.patterns`
  - `Q-D8` Regras inegociáveis _(múltipla escolha: sem chamada de rede fora da camada de dados · sem segredo no bundle · acessibilidade mínima · tratamento obrigatório de loading e erro)_ → `guardrails.frontend.required`
- **Mobile**
  - `Q-D9` Plataformas alvo, versões mínimas e política de release → `guardrails.mobile.targets`
  - `Q-D10` Restrições _(múltipla escolha: permissões justificadas · funcionamento offline · armazenamento seguro de credenciais · compatibilidade de versão mínima)_ → `guardrails.mobile.required`
- **UX-UI**
  - `Q-D11` Existe design system, tokens ou biblioteca de componentes? → `guardrails.ux.design_system`
  - `Q-D12` Requisitos de consistência e acessibilidade _(nível de contraste, navegação por teclado, textos de erro, estados vazios)_ → `guardrails.ux.required`

<critical>GUARDRAIL QUE NÃO PODE SER VERIFICADO NÃO DEVE SER GERADO. REESCREVER ATÉ FICAR VERIFICÁVEL OU DESCARTAR.</critical>

---

## E. Perguntas sobre gates de validação

Obs.: Serve para criação do arquivo de `.ai/gates/gates.md`.

<critical>PERGUNTAR COMO O USUÁRIO QUER VALIDAR AS SAÍDAS DE CÓDIGO GERADO PELO AGENTE DE IA</critical>

Gates sugeridos:

- **Build**
  - `Q-E1` Comando exato de build e o que caracteriza sucesso → `gates.build`
- **Lint**
  - `Q-E2` Comando de lint/format/type-check; warnings bloqueiam? _(erros bloqueiam, warnings não — recomendado)_ → `gates.lint`
- **Testes**
  - `Q-E3` Comando de teste e níveis exigidos por tipo de mudança → `gates.tests.command`
  - `Q-E4` Toda mudança de comportamento exige teste? _(sim — recomendado · só em regra de negócio · não)_ → `gates.tests.policy`
  - `Q-E5` Teste antes da implementação (TDD)? _(sim para correção de bug — recomendado · sempre · não)_ → `gates.tests.tdd`
  - `Q-E6` Cobertura mínima, se houver _(sem meta numérica — recomendado quando não existe medição hoje · percentual)_ → `gates.tests.coverage`
  - `Q-E7` O agente pode alterar ou desabilitar teste existente para fazer a suíte passar? _(nunca sem aprovação — recomendado)_ → `gates.tests.mutation_policy`

Perguntas adicionais de fechamento:

- `Q-E8` Ordem de execução dos gates e quais são bloqueantes → `gates.order`
- `Q-E9` Como o agente deve reportar a evidência de execução? _(relatório direto na conclusão da tarefa — recomendado · resumo livre)_ → `gates.evidence`
- `Q-E10` O que fazer quando um gate falha por causa pré-existente, não introduzida pela mudança? _(reportar e não corrigir fora de escopo — recomendado · corrigir)_ → `gates.preexisting_failures`

---

## F. Confirmação e geração

1. Apresentar ao usuário um **resumo consolidado** de todas as respostas, agrupado por
   bloco, marcando o que veio de inferência e o que ficou em aberto.
2. Pedir confirmação explícita. Correções voltam ao bloco de origem.
3. Gravar `.ai/memory.json`.
4. Executar o protocolo de geração descrito em `@../README.md`, seção 5.
5. Apresentar a checklist de validação (`@../README.md`, seção 8) preenchida, listando
   os `TODO(descoberta)` remanescentes.

<critical>NÃO GERAR NENHUM ARQUIVO DE `.ai/` ANTES DA CONFIRMAÇÃO DO RESUMO PELO USUÁRIO.</critical>
