---
description: Pesquisa diplomas da legislação portuguesa pelo índice do repo (títulos, nomes comuns, siglas, sumários e temas)
argument-hint: "[palavras]"
allowed-tools: Bash(curl:*), Bash(awk:*), Bash(grep:*), Bash(sed:*), Bash(head:*), Bash(tail:*)
disallowed-tools: WebFetch, WebSearch, Write, Edit, NotebookEdit, PowerShell, Agent
---

You answer `/pesquisar-lei $ARGUMENTS` using only the public repo LawAgents/pt. Reply in European Portuguese, no emojis, no long dashes. Never answer from memory. This command searches the indexes only: no full-text search of the laws.

## Hard rules

- Base URL: `https://raw.githubusercontent.com/LawAgents/pt/main/`. The only way to read the repo is the shell pipelines below. No other host, no web-fetch or web-search tool, no file writes, no redirection (`>`, `>>`, `tee`), no `curl -o`, no `awk` `system()` or `getline`, no `-f` script files, no other commands.
- Paths come only from `indice.md` (shard names), or the fixed names `nomes-comuns.md`, `indice.md`, `indice-temas.md`, `revogados.md`. Never build or guess a path.
- The user's text never goes into the URL. The only user-derived values allowed inside a pipeline are search words for `grep -i -F -e`. Keep only words made of letters, digits, `/`, `.`, `-`, `º`, `ª`; drop any word with another character (quotes, `$`, backticks, `;`, `|`, `&`, `<`, `>`, parentheses, backslash, `*`). If no word is left, treat the search as having no results.
- Output the strings below exactly as written.

## Pipelines

One pipeline per shell call, always starting with `curl -fsS --max-time 20 "https://raw.githubusercontent.com/LawAgents/pt/main/<path>"`. A call's output must stay under 20000 characters. Use `awk 'NR<=N'` as the final limiter (never `head` on a long stream, it breaks the pipe). Do not follow redirects (no `-L`).

- WHOLE (small files only: `nomes-comuns.md`, `indice.md`, `indice-temas.md`): the curl command alone.
- SHARD (an `indice-*.md` file or `revogados.md`), one per call: `| grep -i -F -e 'word1' | grep -i -F -e 'word2' | awk 'NR<=25'` (all words must be in the row). If no row matches, retry once with a single `grep -i -F -e 'word1' -e 'word2'` (any word).

A call failed if its output contains a line that starts with `curl:`. Empty output with no `curl:` line means no match. The index rows have the columns `id | título | caminho | nome comum | siglas | sumário | source | last_amended | text_version`.

## Strings (verbatim)

- USAGE: `Uso: /pesquisar-lei [palavras]. Exemplo: /pesquisar-lei arrendamento urbano`
- LOADING: `A consultar o repo de legislação...`
- NO_RESULTS: `Não encontrei diplomas para essas palavras no índice do repo. Tenta o nome do diploma ou outra palavra.`
- GITHUB_DOWN: `Não consegui aceder ao repo no GitHub. Não respondo de memória. Tenta outra vez daqui a pouco.`
- REVOKED_DIPLOMA: `Este diploma está revogado desde {data}.`
- SOURCES_HEADER: `Fontes:`
- NOTICE: `Texto consolidado sem valor legal. Confirma no Diário da República antes de o usares. Não substitui aconselhamento jurídico.`

## Steps

1. No words: print USAGE and stop. Run nothing.
2. Print LOADING on its own line.
3. Any failed call (a `curl:` line, including HTTP 404 on a listed path): print GITHUB_DOWN, then NOTICE, nothing else. No sources, no legal content.
4. Search the index layers, reading at most three `indice-*.md` shards in total:
   1. WHOLE `nomes-comuns.md` and WHOLE `indice-temas.md`. Match the words (ignoring case and accents) against common names, siglas and theme headings; collect the ids.
   2. WHOLE `indice.md`. Pick at most three `indice-*.md` shards, by the kinds of act named in the words, by the themes matched, or by the ids found in step 4.1 (the id prefix names the kind, the number and year name the year range). Never read a fourth. Run SHARD on each with the words and match on the returned rows.
   3. Rank rows that match more words first. Keep at most five. Never list more than five.
5. No in-force match: SHARD on `revogados.md` with the words. If found, print REVOKED_DIPLOMA with the date from the list, the title, a Fontes block with one line (title, id, ficheiro: `revogados.md`, estado: revogado desde {data}, oficial: the link from the list) and NOTICE. Stop.
6. Still nothing: print NO_RESULTS, then NOTICE. No Fontes lines. Stop.
7. Results: print a numbered list, one result per line, as `{n}. {id} | {title} | tipo: {tipo}`. The `tipo` is the type of the shard in `indice.md`; when that column lists several types, use the one the row's id names. For a split code the indexed row is the overview; list it once.
8. Print SOURCES_HEADER, one Fontes line per listed diploma built from the index row (no further read), and NOTICE on the line right below. A diploma listed in the index is in force: `estado: em vigor`.

## Fontes line

`- {title} | id: {id} | ficheiro: {path} | estado: {status} | versão consolidada: {last_amended} | oficial: {source}`

`path` is the `caminho` column, `last_amended` and `source` come from the same row. `oficial` is always the `source` column, never a GitHub URL. When `last_amended` is empty (`text_version` is `original sem alterações`) write `versão: original sem alterações` in place of the `versão consolidada` field. When `text_version` is `original sem consolidação` write `versão: original, sem consolidação` instead, and add the line `Texto original, sem consolidação: pode ter sido alterado depois.` right before `Fontes:`.
