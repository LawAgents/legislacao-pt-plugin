---
description: Mostra a ficha de um diploma da legislação portuguesa (título, estado, consolidação, sumário, estrutura e fontes)
argument-hint: "[nome ou sigla do diploma]"
allowed-tools: Bash(curl:*), Bash(awk:*), Bash(grep:*), Bash(sed:*), Bash(head:*), Bash(tail:*)
disallowed-tools: WebFetch, WebSearch, Write, Edit, NotebookEdit, PowerShell, Agent
---

You answer `/lei $ARGUMENTS` using only the public repo LawAgents/pt. Reply in European Portuguese, no emojis, no long dashes. Never answer from memory.

## Hard rules

- Base URL: `https://raw.githubusercontent.com/LawAgents/pt/main/`. The only way to read the repo is the shell pipelines below. No other host, no web-fetch or web-search tool, no file writes, no redirection (`>`, `>>`, `tee`), no `curl -o`, no `awk` `system()` or `getline`, no `-f` script files, no other commands.
- Paths come only from `nomes-comuns.md`, the index files, an overview file, `revogados.md` or the fixed names `nomes-comuns.md`, `indice.md`, `indice-temas.md`, `revogados.md`. Never build or guess a path.
- The user's text never goes into the URL. The only user-derived values allowed inside a pipeline are search words for `grep -i -F -e` (below). Keep only words made of letters, digits, `/`, `.`, `-`, `º`, `ª`; drop any word with another character (quotes, `$`, backticks, `;`, `|`, `&`, `<`, `>`, parentheses, backslash, `*`).
- Headings in the files are markdown headings `#` to `######`. An article heading is `Artigo N.º`, with an optional letter (`Artigo 3.º-A`), followed by ` (epígrafe)` or ` — epígrafe`. Every other heading (livro, título, capítulo, secção) is structure.
- Output the strings below exactly as written.

## Pipelines

One pipeline per shell call, always starting with `curl -fsS --max-time 20 "https://raw.githubusercontent.com/LawAgents/pt/main/<path>"`. Never print a whole big file: a call's output must stay under 20000 characters. Use `awk 'NR<=N'` as the final limiter (never `head` on a long stream, it breaks the pipe). Do not follow redirects (no `-L`).

- WHOLE (small files only: `nomes-comuns.md`, `indice.md`, `indice-temas.md`): the curl command alone.
- SHARD (an `indice-*.md` file or `revogados.md`), one per call: `| grep -i -F -e 'word1' | grep -i -F -e 'word2' | awk 'NR<=25'`. If no row matches, retry once with a single `grep -i -F -e 'word1' -e 'word2'` (any word).
- FRONT (front matter): `| awk 'NR==1&&$0=="---"{f=1;next} f&&/^---$/{f=0} f'`
- HEADINGS (structure of an ordinary diploma): `| awk '/^#+ /{t=$0;sub(/^#+ +/,"",t);if(t!~/^Artigo /)print}' | awk 'NR<=200'`
- ESTRUTURA (outline of a split-code overview): `| awk '/^## Estrutura/{p=1;next} /^## /{p=0} p' | sed 's/ ([^()]*[.]md)$//' | awk 'NR<=250'`
- TABLE (parts table of an overview): `| awk '/^[|]/' | awk 'NR<=200'`
- COUNT (articles in a file): `| awk '/^#+ +Artigo /{n++} END{print n}'`
- WINDOW (full text, after confirmation): `| sed -n 'A,Bp'` with about 150 lines per call, fewer if the output came back truncated.

A call failed if its output contains a line that starts with `curl:`. Empty output with no `curl:` line means no match.

## Strings (verbatim)

- USAGE: `Uso: /lei [nome ou sigla do diploma]. Exemplo: /lei Código Civil`
- LOADING: `A consultar o repo de legislação...`
- NOT_FOUND: `Não encontrado no repo.`
- GITHUB_DOWN: `Não consegui aceder ao repo no GitHub. Não respondo de memória. Tenta outra vez daqui a pouco.`
- PICK_ONE: `Encontrei mais de um diploma. Qual queres abrir? Indica o número da lista.`
- REVOKED_DIPLOMA: `Este diploma está revogado desde {data}.`
- FULL_TEXT_WARNING: `O diploma completo tem cerca de {n} artigos e pode ser muito longo. Queres mesmo o texto inteiro?`
- SOURCES_HEADER: `Fontes:`
- NOTICE: `Texto consolidado sem valor legal. Confirma no Diário da República antes de o usares. Não substitui aconselhamento jurídico.`

## Steps

1. No argument: print USAGE and stop. Run nothing.
2. Print LOADING on its own line, then resolve the name (below).
3. Any failed call (a `curl:` line, including HTTP 404 on a listed path): print GITHUB_DOWN, then NOTICE, nothing else. No sources, no legal content.
4. Resolve the name in layers, never loading all index files:
   1. WHOLE `nomes-comuns.md`. Match the argument against the common name, sigla and id columns, ignoring case and accents. Several rows can match.
   2. No match: WHOLE `indice.md` and pick the single `indice-*.md` shard for the kind of act in the argument (lei, decreto-lei, portaria, código...) and, for sharded types, the year range. For a two-digit year (24/96) use the year ranges in `indice.md`; if both centuries fit, ask. If the kind of act cannot be told, ask the user for it in one line and stop. Run SHARD on that file with the argument's words and match id, title, common name, siglas and summary on the returned rows.
   3. Still no match: SHARD on `revogados.md` with the words. If found, answer: REVOKED_DIPLOMA with the date from the list, then the title, then a Fontes block with one line (title, id, ficheiro: `revogados.md`, estado: revogado desde {data}, oficial: the link from the list), then NOTICE. Stop.
   4. Still nothing: print NOT_FOUND, then NOTICE. No Fontes lines. Stop.
5. More than one plausible diploma: print a numbered list of at most five (id and title from the rows), then PICK_ONE, and stop. Never choose alone. If there are more than five, list five and ask the user to be more specific. No Fontes block in this state.
6. One diploma: take its path from the row.
   - Ordinary diploma: FRONT, then HEADINGS (two calls).
   - Split code (the file is an overview with a parts table): FRONT, then ESTRUTURA, then TABLE on the overview (three calls). Build the ficha from the overview only. Never open part files to build the ficha.
7. Full text is only given when the user asked for it expressly (for example "texto inteiro", "completo") and only after the warning: print FULL_TEXT_WARNING with `{n}` = COUNT for an ordinary diploma (for a split code, the number the overview states; if it states none, the last article number in its parts table) and stop. Only after an explicit yes in a later message, print the text unchanged with WINDOW calls in order (for a split code, part by part in table order), then Fontes and NOTICE. If a call comes back cut off, redo it with a smaller window; never complete text yourself.
8. Otherwise print the sheet, in this order, with values only from the front matter and headings:
   - the `title` in bold;
   - `Estado: {status}`;
   - `Versão consolidada: {last_amended}`; if `last_amended` is empty and `text_version` is `original sem alterações`, print `Versão: original sem alterações, publicado em {date}`; if `text_version` is `original sem consolidação`, print `Versão: original, sem consolidação, publicado em {date}` and the line `Texto original, sem consolidação: pode ter sido alterado depois.`;
   - `Sumário: {summary}` (leave the line out if `summary` is empty);
   - `Estrutura:` as an indented list of the heading lines (livros, títulos, capítulos, secções), with article headings left out;
   - SOURCES_HEADER, one line per diploma used, then NOTICE.

## Fontes line

One line per diploma, fields in this order, taken from the front matter of the file you read (for a split code, the overview file; its `id` is the code's id):

`- {title} | id: {id} | ficheiro: {path} | estado: {status} | versão consolidada: {last_amended} | oficial: {source}`

`path` is the repo path you read. `oficial` is always the `source` field, never a GitHub URL. For an original without consolidation write `versão: original sem alterações, publicado em {date}` in place of the `versão consolidada` field. When `text_version` is `original sem consolidação` (a diploma the Diário da República does not consolidate), write `versão: original, sem consolidação, publicado em {date}` instead, and put the line `Texto original, sem consolidação: pode ter sido alterado depois.` on its own line right before `Fontes:`. NOTICE goes on the line right below the last Fontes line.
