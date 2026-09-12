# Stack: {{stack.id}}

[Template de stack. Gera **um arquivo por tecnologia principal** do projeto, salvo em
`.ai/skeletons/<stack>.md`, nomeado com o nome da stack. Exemplo: `node.md`,
`react-native.md`, `postgres.md`.]

[Preencher a partir de `Q-C6`, `Q-C11`, `Q-C12` e `Q-C13`, e do que for verificável no
repositório (manifesto de dependências, configuração de lint, scripts). Não registrar
versão, comando ou convenção que não tenha sido confirmada.]

[Este arquivo responde **como escrever código nesta tecnologia**. Arquitetura geral fica
em `@./architecture.md`; comandos de validação e critérios de aprovação ficam em
`@../gates/gates.md` — aqui só se referencia.]

---

## 1. Identificação

| Item                         | Valor                     |
| ---------------------------- | ------------------------- |
| Papel no projeto             | {{stack.role}}            |
| Linguagem                    | {{stack.language}}        |
| Versão do runtime            | {{stack.runtime_version}} |
| Gerenciador de pacotes       | {{stack.package_manager}} |
| Frameworks e libs principais | {{stack.frameworks}}      |
| Ferramenta de teste          | {{stack.test_tooling}}    |
| Tipagem estática             | {{stack.typing}}          |

- **MUST** usar apenas recursos disponíveis na versão declarada do runtime.
- **MUST** usar o gerenciador de pacotes declarado; não misturar gerenciadores.

---

## 2. Comandos

| Ação                       | Comando                      |
| -------------------------- | ---------------------------- |
| Instalar dependências      | `{{stack.commands.install}}` |
| Executar localmente        | `{{stack.commands.run}}`     |
| Build                      | `{{stack.commands.build}}`   |
| Lint / format / type-check | `{{stack.commands.lint}}`    |
| Testes                     | `{{stack.commands.test}}`    |

---

## 3. Nomenclatura de símbolos

| Tipo de símbolo          | Convenção                         | Exemplo |
| ------------------------ | --------------------------------- | ------- |
| Variável e parâmetro     | [ex.: camelCase]                  |         |
| Função e método          | [ex.: camelCase, verbo no início] |         |
| Classe, tipo e interface | [ex.: PascalCase]                 |         |
| Constante                | [ex.: UPPER_SNAKE_CASE]           |         |
| Booleano                 | [ex.: prefixo `is`/`has`/`can`]   |         |
| Arquivo e diretório      | ver `@./folder-structure.md`      |         |
| Tabela e coluna          | [convenção do banco]              |         |

- Identificadores **MUST** ser escritos em {{languages.code}}.
- Nome **MUST** revelar intenção; abreviação só quando for o termo do domínio.

---

## 4. Estilo e ferramentas

- Formatador: [ferramenta e arquivo de configuração]
- Linter: [ferramenta, arquivo de configuração, regras que bloqueiam]
- Verificador de tipos: [ferramenta, modo estrito?]
- **MUST NOT** desabilitar regra de lint com comentário inline sem justificativa escrita
  na mesma linha e aprovação — ver `@../rules/RULES.md` (`RULE-VER-05`).
- **MUST NOT** alterar a configuração das ferramentas para acomodar código novo.

---

## 5. Idioms obrigatórios

[Padrões reais desta stack neste projeto, com exemplo curto quando o texto não bastar.
Preferir poucas regras aplicadas a muitas regras decorativas.]

- `MUST` [ex.: tratamento de erro assíncrono no padrão adotado pelo projeto]
- `MUST` [ex.: validação de entrada com a biblioteca X na borda]
- `SHOULD` [ex.: preferir composição de funções a herança]
- `MAY` [ex.: preferência de sintaxe sem impacto funcional]

---

## 6. Padrões proibidos

[O que existe na linguagem/framework mas não é aceito neste projeto, com o motivo e a
alternativa correta. Sem alternativa, a regra não é acionável.]

| Proibido | Motivo   | Usar em vez disso |
| -------- | -------- | ----------------- |
| [padrão] | [motivo] | [alternativa]     |

---

## 7. Anatomia de um módulo

```
[Estrutura típica de um arquivo desta stack no projeto: ordem de imports, exports,
declarações. Serve para que o código novo seja indistinguível do existente.]
```

---

## 8. Erros e exceções

- Tipos de erro do projeto e quando usar cada um: [listar]
- Erro de domínio **MUST** ser distinguível de erro de infraestrutura.
- **MUST NOT** capturar exceção sem tratar ou sem repropagar com contexto.
- Mensagem de erro voltada ao usuário **MUST NOT** expor detalhe interno, stack trace,
  query ou dado sensível, e **MUST** ser escrita em linguagem natural, sem jargão técnico.

---

## 9. Testes nesta stack

- Ferramenta e convenção de nomes: [descrever]
- Estrutura do teste: [ex.: arranjo, ação, verificação]
- Dublês (mock/stub/fake): [o que é permitido dublar; **MUST NOT** dublar o que está
  sendo testado]
- Teste **MUST** ser determinístico: sem dependência de relógio real, rede, ordem de
  execução ou estado compartilhado entre testes.
- Política de cobertura e obrigatoriedade: `@../gates/gates.md`.

---

## 10. Configuração e ambiente

- Variáveis obrigatórias: [nome e finalidade — **nunca** o valor]
- Validação de configuração no boot: [como é feita]
- **MUST NOT** ler variável de ambiente fora da camada de configuração.
- Segredos seguem `@../rules/RULES.md`, seção 8.

---

## 11. Performance

- Limites conhecidos e alvos: [do projeto, não genéricos]
- Armadilhas comuns nesta stack: [ex.: consultas N+1, bloqueio do loop de eventos,
  renderização desnecessária]
- Otimização **MUST** ser justificada por medição; **MUST NOT** trocar clareza por
  desempenho sem número que sustente a troca.

---

## 12. Armadilhas conhecidas

[Comportamentos não óbvios desta stack **neste projeto** que já causaram erro. Cada item
evita um retrabalho previsível.]

- [armadilha → como evitar]

---

## 13. Política de versão

- Atualização de runtime, framework ou dependência é decisão de nível `CONFIRMAR`.
- **MUST NOT** usar recurso marcado como deprecado na versão em uso.
- [Registrar restrições de compatibilidade: versão mínima suportada, ambiente de destino.]
