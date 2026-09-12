## Guardrails de UX/UI para agentes de IA

[Gerar a partir de `Q-D11` e `Q-D12`. Design system do projeto:
{{guardrails.ux.design_system}}. Aplica-se às superfícies com interface (web e/ou mobile).
Regras de implementação ficam em `@./frontend.md` e `@./mobile.md`; aqui ficam as regras de
consistência e experiência.]

### Consistência

- `GR-UX-01` **MUST** — usar os componentes e tokens existentes do projeto (cor, espaçamento,
  tipografia, raio, sombra). **MUST NOT** introduzir valor arbitrário fora do sistema.
- `GR-UX-02` **MUST NOT** — criar componente novo quando existe um equivalente; estender o
  existente ou pedir aprovação para o novo.
- `GR-UX-03` **MUST** — manter padrão de linguagem: mesmos termos para os mesmos conceitos,
  mesmo tom, mesma capitalização, no idioma da interface.
- `GR-UX-04` **MUST** — posicionar ações primárias e secundárias como no restante do produto.

### Estados obrigatórios

- `GR-UX-05` **MUST** — toda tela ou bloco de dado define: carregando, vazio, erro, sem
  permissão e sucesso. {{guardrails.ux.required}}
- `GR-UX-06` **MUST** — estado vazio explica o que aconteceu e qual é a próxima ação possível.
- `GR-UX-07` **MUST** — mensagem de erro diz o que aconteceu e o que o usuário pode fazer;
  **MUST NOT** expor código interno, stack trace ou jargão técnico.

### Interação

- `GR-UX-08` **MUST** — ação destrutiva ou irreversível exige confirmação que nomeia o objeto
  afetado e a consequência.
- `GR-UX-09` **MUST** — impedir envio duplicado de formulário e ação repetida durante
  processamento.
- `GR-UX-10` **MUST** — preservar o que o usuário digitou após erro de validação ou de rede.
- `GR-UX-11` **MUST** — validar formulário no momento adequado e apontar o campo com problema
  junto do campo.

### Acessibilidade

- `GR-UX-12` **MUST** — todo elemento interativo é operável por teclado, com foco visível e
  ordem de navegação previsível.
- `GR-UX-13` **MUST** — todo controle e imagem com significado tem nome ou texto alternativo.
- `GR-UX-14` **MUST NOT** — comunicar informação apenas por cor, posição ou movimento.
- `GR-UX-15` **MUST** — respeitar o contraste mínimo e o tamanho mínimo de alvo de toque
  definidos pelo projeto.
- `GR-UX-16` **MUST** — respeitar preferências do sistema quando o projeto as suporta
  (tema, redução de movimento, tamanho de fonte).

### Responsividade

- `GR-UX-17` **MUST** — funcionar nos tamanhos de tela suportados, sem rolagem horizontal
  indevida e sem conteúdo cortado.
- `GR-UX-18` **MUST NOT** — assumir dimensão fixa de conteúdo dinâmico: texto traduzido,
  nome longo e lista vazia precisam continuar legíveis.
