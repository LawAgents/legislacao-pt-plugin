# legislacao-pt

Plugin do Claude Code que consulta a legislação portuguesa consolidada guardada no repo público [LawAgents/pt](https://github.com/LawAgents/pt) e cita sempre a fonte oficial. Responde só com o que está no repo e nunca de memória.

## O que faz

Três comandos:

| Comando | O que devolve |
|---|---|
| `/lei [nome ou sigla do diploma]` | A ficha do diploma: título, estado, data da versão consolidada, sumário, estrutura por livros, títulos e capítulos, e as fontes. O texto inteiro só se o pedires, depois de um aviso do tamanho. |
| `/artigo [nome ou sigla do diploma] [número do artigo]` | O texto do artigo (`483`, `483.º`, `483.º-A`), com as fontes. Trata artigos revogados e números que não existem. |
| `/pesquisar-lei [palavras]` | Até cinco diplomas encontrados no índice (títulos, nomes comuns, siglas, sumários e temas), com as fontes. Não pesquisa o texto das leis. |

Exemplos: `/lei Código Civil`, `/artigo Código Civil 483`, `/pesquisar-lei arrendamento urbano`.

Cada resposta com legislação termina com o bloco `Fontes:` (título, id, ficheiro no repo, artigo, estado, data da versão consolidada e ligação oficial do Diário da República) e a linha: "Texto consolidado sem valor legal. Confirma no Diário da República antes de o usares. Não substitui aconselhamento jurídico."

## Como instalar no Claude Code

1. No Claude Code, escreve `/plugin marketplace add LawAgents/legislacao-pt-plugin`.
2. Escreve `/plugin install legislacao-pt@legislacao-pt`.
3. Reinicia a sessão e experimenta `/lei Código Civil`.

## Privacidade

- O plugin não escreve ficheiros no teu computador, não guarda registos e não tem contas.
- Só lê ficheiros públicos em `raw.githubusercontent.com/LawAgents/pt`, com `curl` em modo de leitura (e `awk`, `grep` e `sed` para escolher o trecho). Não contacta mais nenhum serviço.
- A tua pergunta não vai no endereço: o plugin só pede ficheiros do repo por caminhos que vêm dos índices. Do que escreves, só o número do artigo e as palavras da pesquisa são usados, localmente, para escolher o trecho do ficheiro.
- Os comandos pedem permissão só para `curl`, `awk`, `grep`, `sed`, `head` e `tail`; as ferramentas de escrita e de pesquisa na web ficam desligadas nos comandos.

## Aviso

O texto é consolidado e não tem valor legal. Confirma sempre no Diário da República. Não substitui aconselhamento jurídico.

## Licença

MIT (ver `LICENSE`). Os textos legislativos no repo LawAgents/pt são atos oficiais do Diário da República e não recebem licença própria.
