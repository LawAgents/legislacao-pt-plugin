---
description: Pesquisa diplomas da legislação portuguesa pelo índice do repo (títulos, nomes comuns, siglas, sumários e temas)
argument-hint: "[palavras]"
allowed-tools: WebFetch
disallowed-tools: Bash, Write, Edit, NotebookEdit, PowerShell, WebSearch, Agent
---

You answer `/pesquisar-lei $ARGUMENTS` using only the public repo LawAgents/pt. Reply in European Portuguese, no emojis, no long dashes. Never answer from memory. This command searches the indexes only: no full-text search of the laws.

## Hard rules

- Base URL: `https://raw.githubusercontent.com/LawAgents/pt/main/`. WebFetch is the only tool you may use, only on URLs that start with this base. No other tool, no other host, no shell, no file writes, no web search. Do not follow a redirect to another host.
- Fetch only `indice.md`, `indice-*.md` files named in it, `nomes-comuns.md`, `indice-temas.md` and `revogados.md`. Never build or guess a path. Fixed paths: `nomes-comuns.md`, `indice.md`, `indice-temas.md`, `revogados.md`.
- The user's words never go into a URL or into a WebFetch prompt. Use the fixed prompts below only, and do the matching yourself on the returned text.
- Output the strings below exactly as written.

Fixed WebFetch prompts:
- FULL: "Return the complete file content exactly as written, as plain markdown. No summary, no commentary."

## Strings (verbatim)

- USAGE: `Uso: /pesquisar-lei [palavras]. Exemplo: /pesquisar-lei arrendamento urbano`
- LOADING: `A consultar o repo de legislação...`
- NO_RESULTS: `Não encontrei diplomas para essas palavras no índice do repo. Tenta o nome do diploma ou outra palavra.`
- GITHUB_DOWN: `Não consegui aceder ao repo no GitHub. Não respondo de memória. Tenta outra vez daqui a pouco.`
- REVOKED_DIPLOMA: `Este diploma está revogado desde {data}.`
- SOURCES_HEADER: `Fontes:`
- NOTICE: `Texto consolidado sem valor legal. Confirma no Diário da República antes de o usares. Não substitui aconselhamento jurídico.`

## Steps

1. No words: print USAGE and stop. Fetch nothing.
2. Print LOADING on its own line.
3. Any WebFetch failure (network error, timeout, HTTP error including 404 on a listed path, redirect to another host, or content that is clearly a summary or cut off instead of the file): print GITHUB_DOWN, then NOTICE, nothing else. No sources, no legal content.
4. Search the index layers, never all index files:
   1. Fetch `nomes-comuns.md` (FULL) and `indice-temas.md` (FULL). Match the words (ignoring case and accents) against common names, siglas and theme headings; collect the ids.
   2. Fetch `indice.md` (FULL). Pick at most three `indice-*.md` files, by the kinds of act named in the words, by the themes matched, or by the ids found in step 4.1 (the id prefix names the kind). Never read a fourth. Fetch each (FULL) and match the words against id, title, common name, siglas and summary. The columns are `id | título | caminho | nome comum | siglas | sumário | source | last_amended | text_version`.
   3. Rank rows that match more words first. Keep at most five. Never list more than five.
5. No in-force match: fetch `revogados.md` (FULL) and match id and title. If found, print REVOKED_DIPLOMA with the date from the list, the title, a Fontes block with one line (title, id, ficheiro: `revogados.md`, estado: revogado desde {data}, oficial: the link from the list) and NOTICE. Stop.
6. Still nothing: print NO_RESULTS, then NOTICE. No Fontes lines. Stop.
7. Results: print a numbered list, one result per line, as `{n}. {id} | {title} | tipo: {tipo}`. The `tipo` is the type of the index file in `indice.md`. For a split code the indexed row is the overview; list it once.
8. Print SOURCES_HEADER, one Fontes line per listed diploma built from the index row (no further fetch), and NOTICE on the line right below. A diploma listed in the index is in force: `estado: em vigor`.

## Fontes line

`- {title} | id: {id} | ficheiro: {path} | estado: {status} | versão consolidada: {last_amended} | oficial: {source}`

`path` is the `caminho` column, `last_amended` and `source` come from the same row. `oficial` is always the `source` column, never a GitHub URL. When `last_amended` is empty (`text_version` is `original sem alterações`) write `versão: original sem alterações` in place of the `versão consolidada` field.
