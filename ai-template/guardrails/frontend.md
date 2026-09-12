## Guardrails de Frontend para agentes de IA

[Gerar a partir de `Q-D7` e `Q-D8`. Padrões do projeto: {{guardrails.frontend.patterns}}.
Ajustar ao framework em uso; remover o que não se aplica. Guardrails comuns estão em
`@./guardrails.md` e não devem ser repetidos aqui. Regras de aparência e consistência
visual ficam em `@./ux-ui.md`.]

### Estrutura e composição

- `GR-FE-01` **MUST** — separar componente de apresentação de lógica de dados: componente
  que renderiza não busca nem grava dado diretamente.
- `GR-FE-02` **MUST** — usar o padrão de estado adotado pelo projeto; **MUST NOT** introduzir
  uma segunda biblioteca ou um segundo padrão de gerenciamento de estado.
- `GR-FE-03` **MUST** — manter estado no menor escopo que resolve o problema; estado global
  novo é decisão de nível `CONFIRMAR`.
- `GR-FE-04` **MUST NOT** — duplicar no cliente regra de negócio que já existe no servidor;
  validação no cliente é experiência, não autoridade.

### Dados e comunicação

- `GR-FE-05` **MUST** — toda chamada de rede passa pela camada de dados do projeto, nunca
  direto do componente. {{guardrails.frontend.required}}
- `GR-FE-06` **MUST** — tratar explicitamente os quatro estados de dado remoto: carregando,
  sucesso, vazio e erro. Nenhum deles pode ficar sem interface.
- `GR-FE-07` **MUST** — cancelar ou ignorar resposta de requisição obsoleta ao desmontar ou
  ao mudar de parâmetro.
- `GR-FE-08` **MUST NOT** — assumir formato de resposta não confirmado no contrato da API.

### Segurança

- `GR-FE-09` **MUST NOT** — incluir segredo, chave privada ou credencial no bundle, no
  código ou em variável exposta ao cliente.
- `GR-FE-10` **MUST NOT** — injetar HTML ou conteúdo não sanitizado vindo do usuário ou da API.
- `GR-FE-11` **MUST** — armazenar token conforme a política do projeto; **MUST NOT** mudar o
  mecanismo de armazenamento de sessão sem aprovação.

### Qualidade percebida

- `GR-FE-12` **MUST** — toda ação do usuário dá retorno visível: progresso, sucesso ou erro
  acionável.
- `GR-FE-13` **MUST NOT** — bloquear a interface durante operação assíncrona sem indicação.
- `GR-FE-14` **MUST NOT** — introduzir regressão de acessibilidade: elemento interativo
  precisa ser alcançável por teclado e ter nome acessível. Requisitos completos em `@./ux-ui.md`.
- `GR-FE-15` **MUST NOT** — degradar a performance de carregamento sem medição que justifique
  a troca (peso de bundle, renderização desnecessária, imagem sem dimensão definida).
