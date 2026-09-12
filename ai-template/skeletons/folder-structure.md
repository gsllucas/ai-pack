## Estrutura de pastas

[Escrever e completar estrutura com base nas sugestões a seguir]
[Sempre considerar a estrutura de pastas com divisão de responsabilidades]
[Sempre sugerir estrutura de pastas onde não há duplicidade de código]

[Gerar a partir de `Q-C7` a `Q-C10`. Quando o repositório já tem código, a estrutura
real tem precedência sobre a sugerida: documentar o que existe e propor ajustes como
decisão de nível `CONFIRMAR`, não aplicá-los direto.]

[Este arquivo responde **onde o código mora**. A descrição conceitual das camadas está em
`@./architecture.md`. Convenções de nomenclatura de *símbolos de código* estão em
`@./<stack>.md`; aqui ficam apenas nomes de arquivos e diretórios.]

---

### 1. Árvore

```
[Árvore real do projeto, com um comentário de uma linha por diretório de primeiro e
segundo nível. Não incluir diretórios gerados, dependências ou artefatos de build.]
```

[Arquitetura sugerida — usar como ponto de partida quando o projeto for greenfield:]

- domain
- routes/controllers
- database
- repositories
- utils
- workers/jobs

---

### 2. Responsabilidade por diretório

| Diretório                   | Contém                                                      | Não contém                               |
| --------------------------- | ----------------------------------------------------------- | ---------------------------------------- |
| `domain/`                   | entidades, regras de negócio, invariantes, erros de domínio | SQL, HTTP, framework, ORM                |
| `routes/` ou `controllers/` | definição de rota, validação de entrada, serialização       | regra de negócio, query                  |
| `database/`                 | conexão, pool, migrations, schema                           | regra de negócio                         |
| `repositories/`             | acesso a dados e tradução para o domínio                    | regra de negócio, resposta HTTP          |
| `utils/`                    | funções puras, sem estado e sem I/O, usadas em 2+ lugares   | regra de negócio, acesso a banco ou rede |
| `workers/` ou `jobs/`       | rotinas assíncronas e agendadas                             | lógica duplicada dos casos de uso        |

[Ajustar nomes e linhas à estrutura real. Remover diretórios inexistentes; acrescentar
os que existem no projeto e não estão listados.]

---

### 3. Onde colocar código novo

Aplicar na ordem; a primeira condição verdadeira decide:

1. É regra de negócio ou invariante do domínio? → `domain/`
2. É orquestração de um fluxo completo (transação, múltiplos passos)? → camada de
   aplicação/caso de uso, se existir; caso contrário, `domain/`
3. É acesso a banco de dados? → `repositories/`
4. É tradução de protocolo (HTTP, CLI, fila) para chamada interna? → `routes/`
5. É execução assíncrona ou agendada? → `workers/`
6. É função pura, sem estado, já usada em dois lugares? → `utils/`
7. Nenhuma das anteriores → **parar e perguntar**. Criar diretório novo é decisão de
   nível `CONFIRMAR` (`@../RULES.md`, seção 3).

- **MUST NOT** criar arquivo “de apoio” em diretório cuja responsabilidade ele não tem.
- **MUST NOT** criar `helpers/`, `common/`, `shared/` ou `misc/` como destino de código
  sem dono: nomes genéricos acumulam responsabilidades e viram acoplamento.

---

### 4. Nomenclatura de arquivos e diretórios

| Elemento             | Convenção              | Exemplo |
| -------------------- | ---------------------- | ------- |
| Diretório            | {{folders.naming}}     |         |
| Arquivo de código    | {{folders.naming}}     |         |
| Arquivo de teste     | [padrão do projeto]    |         |
| Arquivo de migration | [padrão da ferramenta] |         |

- O nome do arquivo **MUST** descrever a responsabilidade, não o tipo técnico.
- Sufixo de camada **MUST** ser consistente em todo o projeto (usar sempre, ou nunca).

---

### 5. Regras de importação

- Importação permitida segue a direção de dependência de `@./architecture.md`, seção 3.
- **MUST NOT** importar de diretório interno de outro módulo: usar o ponto de entrada
  público do módulo.
- **MUST NOT** criar ciclo de importação entre diretórios.
- [Registrar aqui o mecanismo real de import do projeto: caminho relativo, alias,
  workspace — e qual é o padrão obrigatório.]

---

### 6. Anti-duplicação

- Antes de criar arquivo, função ou tipo, **MUST** procurar no projeto por implementação
  equivalente; reutilizar em vez de recriar.
- Lógica repetida em dois lugares **SHOULD** ser extraída para o dono da responsabilidade
  — não para `utils/` por padrão.
- **MUST NOT** manter duas fontes de verdade para a mesma regra, tipo ou constante.

---

### 7. Testes

- Localização: {{folders.tests_location}}
- Nomenclatura e organização espelham a estrutura do código de produção.
- Fixtures e dados de apoio ficam junto dos testes que os usam; compartilhar apenas
  quando usados por 2+ suítes.
- Comandos e critérios de aprovação estão em `@../gates/gates.md`.
