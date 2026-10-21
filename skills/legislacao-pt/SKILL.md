---
name: legislacao-pt
description: Consulta legislação portuguesa no repo LawAgents/pt e cita sempre a fonte oficial. Usar quando o utilizador pedir uma lei, um artigo ou uma pesquisa de diplomas portugueses.
---

# Legislação portuguesa com fontes

Este plugin lê o repo público LawAgents/pt, que guarda a legislação consolidada do Diário da República em markdown. Responde só com o que está no repo e cita sempre a fonte.

## Regras que nunca se quebram

1. Nunca respondas de memória. Se a informação não está no repo, diz "Não encontrado no repo." e não cites nada.
2. Termina toda a resposta com o bloco `Fontes:`, uma linha por diploma citado: título completo, id, caminho do ficheiro no repo, artigo (quando a resposta se apoia num artigo), estado, data da versão consolidada e ligação oficial do Diário da República. A ligação oficial é a do campo `source` do ficheiro, nunca a do ficheiro no GitHub.
3. Logo abaixo das fontes, escreve sempre a linha: "Texto consolidado sem valor legal. Confirma no Diário da República antes de o usares. Não substitui aconselhamento jurídico."
4. Se o GitHub não responde, diz que não conseguiste aceder ao repo e não respondas de memória.
5. Não escrevas ficheiros, não guardes registos e não envies a pergunta do utilizador a nenhum serviço além do GitHub.

## Como resolver um nome

1. Lê primeiro o ficheiro curto de nomes comuns e siglas.
2. Se não encontrares, lê o índice do tipo de ato certo. Nunca carregues todos os índices de uma vez.
3. Se o nome não aparecer nos diplomas em vigor, consulta o ficheiro de revogados. Se constar de lá, responde que o diploma está revogado desde a data indicada, com título e ligação oficial.
4. Se houver mais de um diploma plausível, lista até cinco, numerados, e pergunta qual abrir. Nunca escolhas sozinho.

## Comandos

### /lei [nome ou sigla do diploma]

Devolve a ficha do diploma: título, estado, data da consolidação, sumário, estrutura por livros, títulos e capítulos, e o bloco `Fontes:`. Nunca devolvas o texto inteiro. Só o dás se o utilizador o pedir expressamente, depois de avisares o tamanho aproximado em artigos e de ele confirmar.

### /artigo [nome ou sigla do diploma] [número do artigo]

Devolve o texto do artigo pedido, com o bloco `Fontes:`. Aceita o número com ou sem sufixo de letra (483 ou 483.º-A). Em códigos divididos em partes, abre só a parte onde o artigo está, escolhida pela visão geral do código. Se o artigo estiver revogado, diz que está revogado, mostra a marca oficial e cita a fonte, sem procurar o texto noutra versão. Se o número não existir no diploma, diz que não existe e lista os números vizinhos.

### /pesquisar-lei [palavras]

Pesquisa no índice (títulos, nomes comuns, siglas, sumários e temas) e devolve até cinco diplomas, cada um com id, título e tipo, mais o bloco `Fontes:`. Não há pesquisa de texto integral em todo o repo. Se nada corresponder, diz que não encontrou diplomas para essas palavras no índice.
