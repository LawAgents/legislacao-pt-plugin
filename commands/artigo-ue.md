---
description: Mostra o texto de um artigo de um ato da legislação da União Europeia (regulamento ou diretiva), com a fonte
argument-hint: "[nome, sigla ou número do ato] [número do artigo]"
allowed-tools: Bash(curl:*), Bash(awk:*), Bash(grep:*), Bash(sed:*), Bash(head:*), Bash(tail:*)
disallowed-tools: WebFetch, WebSearch, Write, Edit, NotebookEdit, PowerShell, Agent
---

You answer `/artigo-ue $ARGUMENTS` using only the public repo LawAgents/ue-pt (EU regulations and directives). Reply in European Portuguese, no emojis, no long dashes. Never answer from memory.

## Hard rules

- Base URL: `https://raw.githubusercontent.com/LawAgents/ue-pt/main/`. The only way to read the repo is the shell pipelines below. No other host, no web-fetch or web-search tool, no file writes, no redirection (`>`, `>>`, `tee`), no `curl -o`, no `awk` `system()` or `getline`, no `-f` script files, no other commands.
- Paths come only from `nomes-comuns.md`, the index files, an overview file, `revogados.md` or the fixed names `nomes-comuns.md`, `indice.md`, `revogados.md`. Never build or guess a path.
- The user's text never goes into the URL. The only user-derived values allowed inside a pipeline are (a) the article heading passed to awk with `-v h='...'` and (b) search words for `grep -i -F -e`. The heading is `Artigo ` plus the number; sanitise it to digits, letters, `º`, `ª`, `-`, `.` and refuse anything else. For the words keep only letters, digits, `/`, `.`, `-`, `º`, `ª`; drop any word with another character (quotes, `$`, backticks, `;`, `|`, `&`, `<`, `>`, parentheses, backslash, `*`).
- Headings in the files are markdown headings `#` to `######`. An article heading is `Artigo N.º`, with an optional letter (`Artigo 3.º-A`), followed by an optional epígrafe. Considerandos live in a `## Considerandos` section, annexes in `## Anexo X` sections; chapters and sections are plain headings. Only `Artigo` headings are articles: never answer an article request from the considerandos or the annexes.
- Output the strings below exactly as written. Reproduce article text verbatim; never paraphrase, shorten, interpret or add legal commentary.

## Pipelines

One pipeline per shell call, always starting with `curl -fsS --max-time 20 "https://raw.githubusercontent.com/LawAgents/ue-pt/main/<path>"`. Never print a whole big file: a call's output must stay under 20000 characters. Use `awk 'NR<=N'` as the final limiter (never `head` on a long stream, it breaks the pipe). Do not follow redirects (no `-L`).

- WHOLE (small files only: `nomes-comuns.md`, `indice.md`): the curl command alone.
- SHARD (an `indice-regulamentos-*.md` / `indice-diretivas-*.md` file or `revogados.md`), one per call: `| grep -i -F -e 'word1' | grep -i -F -e 'word2' | awk 'NR<=25'`. If no row matches, retry once with a single `grep -i -F -e 'word1' -e 'word2'` (any word).
- FRONT (front matter): `| awk 'NR==1&&$0=="---"{f=1;next} f&&/^---$/{f=0} f'`
- TABLE (parts table of an overview): `| awk '/^[|]/' | awk 'NR<=200'`
- ARTICLE (replace the heading in `h`): `| awk -v h='Artigo 17.º' '/^#+ /{t=$0;sub(/^#+ +/,"",t);if(p){p=0;d=1} if(!d&&index(t,h)==1){c=substr(t,length(h)+1,1);if(c==""||c==" "||c=="(")p=1}} p' | awk 'NR<=300'`
- NEIGHBOURS (when ARTICLE printed nothing; `n` is the numeric part of the number, digits only): `| awk -v n='17' '/^#+ +Artigo /{t=$0;sub(/^#+ +Artigo +/,"",t);k=t+0;if(k>=n-3&&k<=n+3)print}' | awk 'NR<=40'`. Use the two nearest smaller and the two nearest larger numbers among the headings returned.

A call failed if its output contains a line that starts with `curl:`. Empty output with no `curl:` line means no match.

## Strings (verbatim)

- USAGE: `Uso: /artigo-ue [nome, sigla ou número do ato] [número do artigo]. Exemplo: /artigo-ue RGPD 17`
- LOADING: `A consultar o repo de legislação da UE...`
- NOT_FOUND: `Não encontrado no repo.`
- GITHUB_DOWN: `Não consegui aceder ao repo no GitHub. Não respondo de memória. Tenta outra vez daqui a pouco.`
- PICK_ONE: `Encontrei mais de um diploma. Qual queres abrir? Indica o número da lista.`
- REVOKED_DIPLOMA: `Este diploma está revogado desde {data}.`
- ARTICLE_REVOKED: `Este artigo está revogado. Marca oficial no texto: {marca}.`
- ARTICLE_MISSING: `O artigo {número} não existe neste diploma. Artigos vizinhos: {lista}.`
- SOURCES_HEADER: `Fontes:`
- NOTICE: `Texto sem valor legal: só o Jornal Oficial da União Europeia faz fé. Confirma no EUR-Lex antes de o usares. Não substitui aconselhamento jurídico.`
- ORIGINAL_NOTICE: `Texto original, sem consolidação: pode ter sido alterado depois.`

## Steps

1. Parse the argument. The article number is the last token; the act name is everything before it. Accept `17`, `17.º`, `17º`, `17-A`, `17.º-A`, with an optional leading `art.` or `artigo`. Normalise to the heading form `Artigo 17.º` or `Artigo 17.º-A` (if the file's headings use a different form, follow the file). If there is no name or no number, print USAGE and stop. Run nothing.
2. Print LOADING on its own line, then resolve the act (below).
3. Any failed call (a `curl:` line, including HTTP 404 on a listed path): print GITHUB_DOWN, then NOTICE, nothing else. No sources, no legal content.
4. Resolve the name in layers, never loading more than three index shards:
   1. WHOLE `nomes-comuns.md`. Match the name against the common name, sigla and id columns, ignoring case and accents. Several rows can match.
   2. No match: the act may be named by number. Build the id from digits only: `regulamento-ue-AAAA-N` or `diretiva-ue-AAAA-N`, the four-digit group being the year (`2016/679` and `679/2016` are both `2016`/`679`; a CELEX such as `32016R0679` is year 2016, number 679, `R` = regulamento, `L` = diretiva). If the kind or the year cannot be told, ask the user in one line and stop. WHOLE `indice.md` and pick the single `indice-regulamentos-*.md` or `indice-diretivas-*.md` shard whose year range holds the year; run SHARD on it with the id (or, for a name, with the name's words) and match id, title, common name, siglas and summary on the returned rows.
   3. Still no match: SHARD on `revogados.md` with the same value. If found: REVOKED_DIPLOMA with the date from the list, the title, a Fontes block with one line (title, id, ficheiro: `revogados.md`, estado: revogado desde {data}, oficial: the link from the list), then NOTICE. Stop.
   4. Still nothing: print NOT_FOUND, then NOTICE. No Fontes lines. Stop.
5. More than one plausible act: print a numbered list of at most five (id and title), then PICK_ONE, and stop. Never choose alone. No Fontes block in this state.
6. One act: take its path from the row.
   - Ordinary act: run ARTICLE on it, then FRONT (two calls).
   - Split act: the row path is the overview. Run TABLE on it; the table is `parte | caminho | primeiro artigo | último artigo`, in order. Choose the part whose range (first to last article) contains the number (compare the number first, then the letter suffix). If the ranges do not settle it, try the candidate parts one at a time. Run ARTICLE on that part's `caminho`, then FRONT on the same file (two calls).
7. Read the ARTICLE result:
   - Heading present and the first line after it does not start with `*(Revogado)*`: print the heading line and the body text verbatim. Trailing amendment lines of the section (lines that start with `- *` and end with `*`) are not part of the text: print them after it, verbatim, under the plain label `Alterações:`. If `text_version` is `original sem consolidação`, ORIGINAL_NOTICE on its own line before the Fontes block. Then the Fontes block (with `artigo`) and NOTICE.
   - The body starts with `*(Revogado)*`: print ARTICLE_REVOKED with `{marca}` = the whole marker line that follows the heading, exactly as in the file, not re-cut or reworded. If the marker line already ends with a full stop, do not add the final full stop of the template, so the sentence ends with exactly one. Any trailing amendment lines go under `Alterações:`. Then ORIGINAL_NOTICE if it applies, the Fontes block (with `artigo`) and NOTICE. Do not look for the text in any other version.
   - ARTICLE printed nothing: run NEIGHBOURS on the same file. Print ARTICLE_MISSING with `{número}` = the number as the user wrote it normalised (`17.º`) and `{lista}` = the article numbers from the headings returned, separated by commas. Use only numbers that were returned. Then NOTICE. No Fontes lines in this case.

## Fontes line

One line per act:

`- {title} | id: {id} | ficheiro: {path} | parte: {part} | artigo: {n} | estado: {status} | versão consolidada: {last_amended} | oficial: {source}`

- For a split act the act is the whole, not the part: `id` is the `part_of` field of the part file (the act's id), and `title` is the act's title from the index row or the overview. For any other act, `id` is the `id` field and `title` the `title` field.
- `path` is the repo path of the file you read (for a split act, the part file that holds the article). `parte` is the `part` field of that file; leave it out when `part` is empty.
- `estado`, `versão consolidada` and `oficial` come from the front matter of the file you read. `oficial` is always the `source` field (EUR-Lex link), never a GitHub URL. Leave `artigo` out when the answer does not rest on an article. For an original without consolidation write `versão: original sem consolidação, de {date}` in place of the `versão consolidada` field.
- NOTICE goes on the line right below the last Fontes line.
