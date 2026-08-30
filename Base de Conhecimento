# Base de Conhecimento

## Dados Utilizados

O agente utiliza como base de conhecimento um único arquivo Excel (`pesquisas_satisfacao_curso.xlsx`), contendo 5 planilhas (abas) de pesquisas de satisfação de curso — sendo 2 abas do mesmo projeto em ciclos diferentes e 3 abas de projetos distintos, totalizando 580 registros de feedback de alunos.

| Arquivo | Formato | Utilização no Agente |
|---------|---------|---------------------|
| `pesquisas_satisfacao_curso.xlsx` — aba `Projeto 1 - Ciclo 1` | XLSX | Contextualizar o histórico de satisfação do Projeto 1 no primeiro ciclo |
| `pesquisas_satisfacao_curso.xlsx` — aba `Projeto 1 - Ciclo 2` | XLSX | Comparar a evolução da satisfação do Projeto 1 entre ciclos |
| `pesquisas_satisfacao_curso.xlsx` — aba `Projeto 2 - Ciclo 1` | XLSX | Contextualizar o histórico de satisfação do Projeto 2 |
| `pesquisas_satisfacao_curso.xlsx` — aba `Projeto 3 - Ciclo 1` | XLSX | Contextualizar o histórico de satisfação do Projeto 3 |
| `pesquisas_satisfacao_curso.xlsx` — aba `Projeto 4 - Ciclo 1` | XLSX | Contextualizar o histórico de satisfação do Projeto 4 |

Cada registro (linha) representa a resposta de um aluno a uma pesquisa de satisfação, contendo os seguintes campos:

- **Projeto** e **Ciclo** — identificam a turma/edição do curso avaliada
- **Nome do Aluno**, **Turma/Módulo**, **Modalidade** e **% do curso concluído** — dados de contexto do respondente
- **Data/Hora do Registro** — quando a resposta foi enviada
- **CSAT (1 a 5)** e **NPS (0 a 10)** — métricas quantitativas de satisfação
- **Principal problema para permanência** — categoria marcada pelo aluno (Acesso a equipamentos, Plataforma utilizada, Qualidade do conteúdo, Didática de ensino ou Suporte)
- **Sugestão de melhoria** e **Problema/Reclamação relatado** — campos abertos de texto livre

> [!TIP]
> **Quer um dataset mais robusto?** Você pode utilizar datasets públicos do [Hugging Face](https://huggingface.co/datasets) relacionados a finanças, desde que sejam adequados ao contexto do desafio.

---

## Adaptações nos Dados

Os dados são **sintéticos**, gerados programaticamente para simular pesquisas de satisfação reais, e passaram pelas seguintes adaptações intencionais:

- **Volume**: cada aba recebeu entre 105 e 130 registros (mínimo de 100 por planilha), com nomes de alunos, datas e distribuições geradas aleatoriamente.
- **Campos adicionais** além dos exigidos originalmente: `Turma/Módulo`, `Modalidade` (EAD, Presencial, Híbrido) e `% do curso concluído`, incluídos para enriquecer o contexto de análise de evasão.
- **Ruído proposital entre rótulo e texto livre**: nos campos abertos (sugestão e reclamação), cerca de 50% do texto é coerente com a categoria de problema marcada, ~20% reflete uma categoria diferente da marcada e ~30% é um comentário genérico/vago sem pista clara de categoria — simulando a subjetividade real de respostas humanas e criando um desafio mais próximo do real para um futuro classificador de texto.
- **Correlação fraca entre CSAT e NPS**: em 20% dos registros o NPS é sorteado de forma independente do CSAT, para não gerar uma relação artificialmente perfeita entre as duas métricas.

---

## Estratégia de Integração

### Como os dados são carregados?

O arquivo `.xlsx` é carregado no início da sessão do agente, lendo todas as 5 abas e consolidando os registros em memória (ex.: um DataFrame por aba ou um único DataFrame com a coluna `Projeto`/`Ciclo` já identificando a origem). Não há consulta em banco de dados externo — a base de conhecimento é estática e lida diretamente do arquivo local.

### Como os dados são usados no prompt?

Os dados não são inseridos integralmente no system prompt (o volume de 580 linhas inviabilizaria isso). Em vez disso:

- Um **resumo agregado** (médias de CSAT/NPS por projeto/ciclo, distribuição das categorias de problema) pode compor o contexto fixo do system prompt.
- Os **registros individuais** são consultados dinamicamente conforme a pergunta do usuário (ex.: filtrando por projeto, ciclo, categoria de problema ou faixa de data) e inseridos no prompt apenas os trechos relevantes à consulta.
- Os campos de texto livre (sugestão/reclamação) são a principal entrada para tarefas de classificação e sumarização, sendo passados ao agente junto com o rótulo original quando o objetivo é avaliar ou treinar a categorização.

---

## Exemplo de Contexto Montado

```
Registro de Pesquisa de Satisfação:
- Projeto: Projeto 2 | Ciclo: 1
- Aluno: Ana Carolina Leão | Turma: C3 | Modalidade: EAD
- % do curso concluído: 40%
- Data do registro: 01/03/2025 03:45
- CSAT: 3/5 | NPS: 3/10
- Principal problema marcado: Didática de ensino
- Sugestão de melhoria: "Pra mim, seria legal ter algum tipo de auxílio pra comprar um equipamento melhor."
- Reclamação relatada: "Nada que me faça pensar em desistir, só um cansaço geral."
```
