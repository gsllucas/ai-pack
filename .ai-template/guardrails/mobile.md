## Guardrails de Mobile para agentes de IA

[Gerar a partir de `Q-D9` e `Q-D10`. Plataformas e versões alvo: {{guardrails.mobile.targets}}.
Ajustar à tecnologia em uso (nativo ou multiplataforma); remover o que não se aplica.
Guardrails comuns estão em `@./guardrails.md`; regras visuais estão em `@./ux-ui.md`.]

### Compatibilidade

- `GR-MOB-01` **MUST** — usar apenas APIs disponíveis nas versões mínimas suportadas
  declaradas; recurso mais novo exige verificação de disponibilidade em tempo de execução.
- `GR-MOB-02` **MUST NOT** — alterar versão mínima de plataforma, SDK ou dependência nativa
  sem aprovação: decisão de nível `CONFIRMAR`.
- `GR-MOB-03` **MUST** — manter paridade de comportamento entre plataformas ou declarar
  explicitamente a diferença. {{guardrails.mobile.required}}

### Permissões e privacidade

- `GR-MOB-04` **MUST NOT** — solicitar permissão que a funcionalidade não usa.
- `GR-MOB-05` **MUST** — pedir permissão no momento do uso, com justificativa, e tratar a
  recusa com caminho alternativo funcional.
- `GR-MOB-06` **MUST NOT** — coletar, armazenar ou enviar identificador de dispositivo ou
  dado pessoal sem necessidade declarada.

### Armazenamento e segurança

- `GR-MOB-07` **MUST** — guardar credencial e token no armazenamento seguro da plataforma,
  nunca em preferências simples, arquivo ou log.
- `GR-MOB-08` **MUST** — tratar o dispositivo como ambiente não confiável: nenhuma decisão
  de autorização depende apenas do cliente.

### Rede e ciclo de vida

- `GR-MOB-09` **MUST** — tratar ausência de conectividade e conexão instável como caso
  normal, não como erro inesperado.
- `GR-MOB-10` **MUST** — preservar e restaurar estado ao voltar de segundo plano; operação
  interrompida não pode deixar dado inconsistente.
- `GR-MOB-11` **MUST NOT** — executar trabalho pesado na thread de interface.
- `GR-MOB-12` **MUST** — considerar consumo de bateria e de dados em rotina periódica,
  sincronização e uso de sensores.

### Entrega

- `GR-MOB-13` **MUST NOT** — alterar configuração de build, assinatura, identificador de
  aplicativo ou canal de release sem aprovação.
- `GR-MOB-14` **MUST** — considerar que versões antigas continuam instaladas: mudança de
  contrato precisa ser compatível com clientes não atualizados.
