# ai-pack

Repositório com os templates canônicos usados para gerar regras, gates e
guardrails para agentes de IA em projetos novos ou existentes.

A ideia é evitar configurar essas informações de forma solta a cada projeto:
o framework em [`.ai-template/`](./.ai-template) é lido por um agente (skill
`/ai-pack`), que conduz uma descoberta guiada por perguntas e gera, a partir
dele, o diretório `.ai/` já ajustado ao contexto específico do projeto.

Veja [`.ai-template/README.md`](./.ai-template/README.md) para o detalhamento
do fluxo de geração.
