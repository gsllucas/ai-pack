# `ai-template` — Framework de geração de orientações para agentes de IA

Este diretório é a **fonte** do framework. Ele não é lido durante o desenvolvimento
do dia a dia: ele é lido **uma vez**, por um agente de IA, que conduz uma descoberta
guiada por perguntas com o usuário e, a partir das respostas, gera o diretório `.ai/`
**específico do projeto**.

```
ai-template/   (genérico, versionado no framework)   →   .ai/   (específico do projeto)
```

- `ai-template/` **nunca** contém informação de um projeto real.
- `.ai/` **nunca** contém instrução para o agente gerador, placeholder ou pergunta.
- O framework gera **regras, diretrizes e padrões de arquitetura**, nunca specs de
  funcionalidade: specs (PRD, tech-spec, requisito de feature) são tratadas em um fluxo
  isolado, no sentido de funcionalidade, e não podem influenciar o conteúdo gerado aqui.

---

## 1. Fluxo de uso

| Fase              | Quem executa               | Entrada                   | Saída                     |
| ----------------- | -------------------------- | ------------------------- | ------------------------- |
| **1. Descoberta** | agente gerador + usuário   | `ask/questions.md`        | respostas confirmadas     |
| **2. Registro**   | agente gerador             | respostas                 | `.ai/memory.json`         |
| **3. Geração**    | agente gerador             | templates + `memory.json` | arquivos de `.ai/`        |
| **4. Validação**  | agente gerador             | `.ai/`                    | relatório de consistência |
| **5. Uso**        | agentes de desenvolvimento | `.ai/RULES.md`            | código, evidências        |

---

## 2. Mapa de arquivos

| Template                           | Gera                                    | Cardinalidade                                   |
| ---------------------------------- | --------------------------------------- | ----------------------------------------------- |
| `ask/questions.md`                 | — (roteiro de perguntas, não é copiado) | —                                               |
| `README.md`                        | — (manual do framework, não é copiado)  | —                                               |
| `memory.json`                      | `.ai/memory.json`                       | 1                                               |
| `RULES.md`                         | `.ai/RULES.md`                          | 1                                               |
| `skeletons/architecture.md`        | `.ai/skeletons/architecture.md`         | 1                                               |
| `skeletons/folder-structure.md`    | `.ai/skeletons/folder-structure.md`     | 1                                               |
| `skeletons/stack-template.md`      | `.ai/skeletons/<stack>.md`              | 1 por stack (ex.: `node.md`, `react-native.md`) |
| `guardrails/guardrails.md`         | `.ai/guardrails/guardrails.md`          | 1                                               |
| `guardrails/api.md`                | `.ai/guardrails/api.md`                 | 0 ou 1 (só se o projeto tiver API)              |
| `guardrails/frontend.md`           | `.ai/guardrails/frontend.md`            | 0 ou 1                                          |
| `guardrails/mobile.md`             | `.ai/guardrails/mobile.md`              | 0 ou 1                                          |
| `guardrails/ux-ui.md`              | `.ai/guardrails/ux-ui.md`               | 0 ou 1                                          |
| `gates/gates.md`                   | `.ai/gates/gates.md`                    | 1                                               |

Arquivos de superfície inexistente **não são gerados vazios**: se o projeto não tem
mobile, `.ai/guardrails/mobile.md` não existe e a referência a ele é removida de
`.ai/guardrails/guardrails.md`.

---

## 3. Responsabilidades e limites de cada arquivo

Esta tabela existe para evitar duplicação de regra. Se uma informação cabe em dois
arquivos, ela vai no dono e o outro **referencia por ID**.

| Arquivo                            | Responde a                                                                              | Não contém                                                |
| ---------------------------------- | --------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| `RULES.md`                         | Como o agente se comporta na sessão, o que pode decidir sozinho e o que exige aprovação | Detalhe de tecnologia, comandos de validação              |
| `skeletons/architecture.md`        | Como o sistema é organizado: camadas, responsabilidades, direção de dependência         | Regra de comportamento do agente, estrutura de diretórios |
| `skeletons/folder-structure.md`    | Onde cada tipo de código mora e onde colocar código novo                                | Descrição conceitual das camadas                          |
| `skeletons/<stack>.md`             | Como escrever código naquela tecnologia: versões, convenções, idioms, comandos          | Arquitetura geral, regras de sessão                       |
| `guardrails/*.md`                  | O que o código produzido **não pode** violar                                            | Comandos executáveis de validação                         |
| `gates/gates.md`                   | Como provar objetivamente que uma mudança está pronta                                   | Regra de estilo ou de arquitetura                         |
| `memory.json`                      | Respostas da descoberta em formato legível por máquina                                  | Prosa normativa                                           |

---

## 4. Convenções de escrita dos templates

| Notação                    | Significado                                            | Sobrevive no `.ai/`?            |
| -------------------------- | ------------------------------------------------------ | ------------------------------- |
| `[texto entre colchetes]`  | Instrução para o agente gerador                        | **Não**                         |
| `{{campo.do.memory}}`      | Valor vindo de `memory.json`                           | **Não** (é substituído)         |
| `<critical>...</critical>` | Regra inegociável do framework, vale durante a geração | **Não**                         |
| `MUST` / `SHOULD` / `MAY`  | Nível normativo da regra gerada                        | **Sim** (literal, não traduzir) |
| `@./<caminho>.md`          | Referência a outro documento de `.ai/`                 | **Sim**                         |
| `RULE-*`, `GR-*`, `GATE-*` | Identificador estável para rastreabilidade             | **Sim**                         |

### Níveis normativos

- `MUST` — obrigatório. Violação bloqueia a conclusão da tarefa.
- `SHOULD` — recomendado. Desvio é permitido, mas **deve** ser declarado no plano de mudança.
- `MAY` — preferência/opcional. Não gera bloqueio nem justificativa.

Os três tokens são mantidos **em inglês e literais** em qualquer idioma de geração,
porque são marcadores de severidade e não prosa.

### Identificadores

| Prefixo | Onde             | Formato             | Exemplo       |
| ------- | ---------------- | ------------------- | ------------- |
| `RULE`  | `RULES.md`       | `RULE-<GRUPO>-<NN>` | `RULE-GIT-03` |
| `GR`    | `guardrails/**`  | `GR-<ESCOPO>-<NN>`  | `GR-API-05`   |
| `GATE`  | `gates/gates.md` | `GATE-<NN>`         | `GATE-02`     |

IDs são **estáveis**: ao regenerar, um ID existente nunca muda de significado. Regra
removida tem seu ID aposentado, não reaproveitado.

---

## 5. Protocolo de geração

<critical>O AGENTE GERADOR DEVE EXECUTAR ESTAS ETAPAS NA ORDEM, SEM PULAR ETAPAS.</critical>

1. **Ler todo o `ai-template/`** antes de qualquer pergunta, incluindo este README.
2. **Inspecionar o repositório** para inferir o que já é observável (linguagens,
   gerenciador de pacotes, frameworks, scripts de build/test/lint, CI, estrutura atual).
   O resultado da inspeção vira _resposta pré-preenchida_, apresentada ao usuário para
   confirmação — nunca uma pergunta do zero.
3. **Conduzir a descoberta** seguindo `ask/questions.md`, bloco por bloco (A → E), respeitando
   o protocolo de perguntas descrito lá.
4. **Escrever `.ai/memory.json`** com as respostas confirmadas antes de gerar qualquer
   documento em prosa. `memory.json` é a única fonte de verdade da geração.
5. **Gerar os arquivos** nesta ordem, porque cada um referencia os anteriores:
   `RULES.md` → `skeletons/` → `guardrails/` → `gates/`.
6. **Resolver todos os placeholders.** Nenhum `[...]`, `{{...}}` ou `<critical>` pode
   permanecer em `.ai/`.
7. **Validar** com a checklist da seção 8 e apresentar o resultado ao usuário.
8. **Referenciar `.ai/` no arquivo raiz de instrução do agente do projeto** (`CLAUDE.md`,
   `AGENTS.md`, `.cursorrules`, `.windsurfrules` ou equivalente): identificar qual desses
   arquivos existe no repositório e adicionar uma linha apontando para `.ai/RULES.md`
   como ponto de entrada, sem sobrescrever o conteúdo existente. Se nenhum existir,
   perguntar ao usuário se deseja criar um arquivo mínimo contendo apenas essa referência.

---

## 6. Invariantes do agente gerador

- `MUST` gerar todo o conteúdo no idioma respondido em `Q-A1`.
- `MUST` registrar apenas o que foi respondido, confirmado ou verificado no repositório.
- `MUST NOT` inventar tecnologia, versão, comando, convenção, requisito ou restrição
  que não tenha origem em uma resposta ou em evidência do repositório.
- Quando uma informação necessária não existir: registrar em `memory.json` (campo
  `open_questions`) e escrever no documento gerado o marcador `TODO(descoberta): <pergunta>`
  — nunca um valor plausível.
- `MUST NOT` alterar código-fonte, configuração ou dependências do projeto durante a
  geração. O framework produz apenas o diretório `.ai/`.
- `MUST NOT` copiar `ask/questions.md` ou `README.md` para `.ai/`.
- Regras genéricas de LLM que não têm relação com o projeto não entram em `.ai/`:
  um documento inflado é ignorado por agentes. Preferir poucas regras aplicáveis a
  muitas regras decorativas.
- `MUST NOT` basear qualquer regra, arquitetura ou guardrail em documento de spec, PRD
  ou tech-spec de funcionalidade. Specs pertencem a um escopo isolado, tratado no sentido
  de funcionalidade; este framework gera padrão de projeto, não requisito de feature.

---

## 7. Regeneração e evolução

- Se `.ai/` já existe, o agente `MUST` lê-lo antes de sobrescrever e `MUST` preservar:
  regras editadas manualmente pelo usuário e IDs existentes.
- Mudanças na descoberta são aplicadas como **diff apresentado ao usuário**, não como
  substituição silenciosa do diretório.
- `memory.json` (campo `framework.template_version`) registra a versão do template
  usada; ao regenerar com uma versão diferente, o agente `MUST` relatar o que mudou no
  framework.
- Remoção de arquivo em `.ai/` exige confirmação explícita do usuário.

---

## 8. Checklist de validação final

Executar antes de declarar a geração concluída:

- [ ] Nenhum `[`, `{{`, `<critical>` ou `TODO(descoberta)` não intencional em `.ai/`.
- [ ] Todo arquivo referenciado com `@` existe; todo arquivo existente é alcançável a
      partir de `.ai/RULES.md`.
- [ ] Nenhuma regra aparece com o mesmo conteúdo em dois arquivos.
- [ ] Todo ID é único dentro do seu prefixo.
- [ ] Todo gate em `gates/gates.md` tem comando executável e critério de aprovação.
- [ ] Todo guardrail é verificável (é possível dizer se foi violado ou não).
- [ ] Tecnologias citadas em `skeletons/` têm arquivo de stack correspondente.
- [ ] Idioma dos documentos corresponde à resposta `Q-A1`.
- [ ] `memory.json` é JSON válido e coerente com os documentos gerados.
- [ ] `.ai/RULES.md` está referenciado no arquivo raiz de instrução do agente do
      projeto (`CLAUDE.md`, `AGENTS.md` ou equivalente).

---

## 9. Decisões de arquitetura do framework

Ambiguidades da estrutura original, resolvidas e registradas aqui para que futuras
gerações não as reabram.

| #   | Ambiguidade                                                                       | Decisão                                                                                                                                                                                             |
| --- | --------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | `ask/questions.md` seria o índice do `.ai/` ou o roteiro de perguntas?            | É **exclusivamente o roteiro de descoberta** e não é copiado. O índice do `.ai/` é a seção _Mapa de contexto_ de `RULES.md`.                                                                        |
| 2   | Qual arquivo é o ponto de entrada do `.ai/`?                                      | `.ai/RULES.md`. Todo o resto é alcançável por referência `@` a partir dele.                                                                                                                         |
| 3   | `stack-a.md` / `stack-b.md` duplicavam o mesmo placeholder                        | Unificados em `skeletons/stack-template.md`, que gera **1 arquivo por stack**, nomeado pela stack.                                                                                                  |
| 4   | `guardrails.md` apontava todas as referências para `api.md`                       | Corrigido; a lista de referências passa a ser gerada a partir das superfícies reais do projeto.                                                                                                     |
| 5   | `memory.json` era citado em `ask/questions.md` mas não existia                    | Criado como template com esquema explícito.                                                                                                                                                         |
| 6   | Onde ficam convenções de nomenclatura de código?                                  | Nomenclatura de **código** fica no arquivo da stack (é específica da linguagem); nomenclatura de **arquivos e diretórios** fica em `folder-structure.md`.                                           |
| 7   | `templates/` (ADR, plano de mudança, pedido de decisão, relatório de verificação) | Removido: os artefatos eram desnecessários para o framework. Plano, pedido de decisão e evidência de verificação passam a ser apresentados diretamente na sessão (`@./RULES.md`, seções 2, 3 e 12). |
| 8   | Layout do `ai-template/` misturava arquivos soltos na raiz (`RULES.md`, `memory.json`, `questions.md`) com pastas (`gates/`, `guardrails/`, `skeletons/`) | Padronizado: cada saída nomeada ganhou pasta própria (`rules/`, `memory/`, `ask/`); só `README.md` fica solto na raiz. `.ai/` espelha o mesmo padrão, exceto `ask/`, que nunca é copiado. **Substituída pela decisão 9.** |
| 9   | `RULES.md` e `memory.json` ficavam um nível abaixo, em `rules/` e `memory/`, pastas com um único arquivo cada | Decisão do mantenedor (2026-09-12, template 1.5.0): `RULES.md` (ponto de entrada) e `memory.json` (fonte de verdade da geração) ficam na **raiz** do `ai-template/` e do `.ai/`; pastas só agrupam conjuntos (`ask/`, `skeletons/`, `guardrails/`, `gates/`). `.ai/` espelha o layout, exceto `ask/` e `README.md`, que nunca são copiados. Referências: de `RULES.md` para os demais, `@./<pasta>/<arquivo>`; das subpastas para eles, `@../RULES.md` e `@../memory.json`. |
