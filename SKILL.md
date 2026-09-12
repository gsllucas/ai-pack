---
name: ai-pack
description: "Use when a repository needs AI agent guidance files — creating, updating or auditing a .ai/ directory with agent rules, guardrails, architecture docs and validation gates. Triggers: /ai-pack, \"configurar regras de IA\", \"gerar .ai\", \"criar guardrails\", \"regras para o agente neste projeto\", \"setup ai rules\", \"agent instructions for this repo\"."
---

# /ai-pack

Conduz uma descoberta guiada por perguntas sobre o projeto e gera `.ai/`: regras de sessão, matriz de autoridade de decisão, arquitetura, guardrails por superfície e gates de validação.

## Arquivos da skill

| Arquivo | Papel |
|---|---|
| `ai-template/README.md` | Protocolo de geração, convenções de placeholder, mapa template→saída, checklist de validação |
| `ai-template/ask/questions.md` | Roteiro de perguntas (blocos A–E), regras de inferência, IDs `Q-XX` |
| `ai-template/**` (demais) | Templates das saídas de `.ai/` |

`README.md` e `ask/questions.md` **nunca** são copiados para `.ai/`.

## Fluxo

1. **Ler** `ai-template/README.md` e `ai-template/ask/questions.md` por inteiro. Sem isso, parar.
2. **Inspecionar o repositório** e pré-preencher o que for observável (linguagens, deps, scripts, CI, estrutura, `CLAUDE.md`/`AGENTS.md` existentes). Apresentar como confirmação, nunca como pergunta em branco.
3. **Entrevistar** seguindo `ask/questions.md`: um bloco por vez, no máximo 4 perguntas por rodada, sempre com opções e uma recomendação justificada.
4. **Resumir e confirmar** todas as respostas com o usuário.
5. **Gravar `.ai/memory.json`**, depois gerar na ordem: `RULES.md` → `skeletons/` → `guardrails/` → `gates/`.
6. **Validar** com a checklist da seção 8 de `ai-template/README.md` e reportar o resultado.

<critical>NENHUM ARQUIVO DE `.ai/` É ESCRITO ANTES DA CONFIRMAÇÃO DO RESUMO NA ETAPA 4.</critical>

## Modos

- **Bootstrap** — não existe `.ai/`: fluxo completo.
- **Atualização** — `.ai/` existe: ler tudo antes, preservar edições manuais e IDs; apresentar diff e pedir aprovação. Remover arquivo exige confirmação explícita.
- **Auditoria** — comparar `.ai/` com o repositório real e listar divergências, sem escrever.

## Red flags — pare e volte ao fluxo

- "O repositório já diz tudo, não preciso perguntar" → inspeção substitui pergunta redundante, não a entrevista.
- "Vou gerar tudo e o usuário revisa depois" → viola a etapa 4.
- "Falta a versão/comando, coloco o mais provável" → use `TODO(descoberta)` e registre em `memory.json` (campo `open_questions`).
- "Gero todos os guardrails para o caso de precisar" → superfície não marcada não gera arquivo.
- "Acrescento boas práticas gerais de LLM" → documento inflado é ignorado; só entra o que veio de resposta ou evidência.

| Racionalização | Realidade |
|---|---|
| "Placeholder `[...]` fica como guia para o usuário" | Placeholder em `.ai/` é template não terminado. Resolver ou virar `TODO(descoberta)`. |
| "Sobrescrevo `.ai/` porque está desatualizado" | Regra editada à mão é decisão do usuário. Ler, diferenciar, perguntar. |
| "Regra genérica não faz mal" | Faz: dilui as regras aplicáveis e reduz a adesão do agente. |
| "Guardrail vago é melhor que nenhum" | Guardrail não verificável não é acionável. Reescrever ou descartar. |

## Verificação antes de concluir

Nenhum `[`, `{{`, `<critical>` ou `TODO(descoberta)` não intencional em `.ai/`; todo `@` resolve; todo ID é único; todo gate tem comando executável; `memory.json` é JSON válido. Reportar os `TODO(descoberta)` remanescentes.

## Manutenção

`ai-template/` é um **symlink** para `~/Desktop/ai-pack/ai-template/` (repositório `ai-pack`, remote `speckit-shared-principles`) — fonte canônica dos templates. Editar os arquivos direto no repositório; não há cópia para ressincronizar. Ao evoluir o framework, incrementar `framework.template_version` em `ai-template/memory.json` e commitar no repositório `ai-pack`.
