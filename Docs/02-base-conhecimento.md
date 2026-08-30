# Base de Conhecimento

## Dados Utilizados

O agente utiliza como base de conhecimento um único arquivo Excel (`pesquisas_satisfacao_curso.xlsx`), contendo 5 planilhas (abas) de pesquisas de satisfação de curso — sendo 2 abas do mesmo projeto em ciclos diferentes e 3 abas de projetos distintos, totalizando 580 registros de feedback de alunos.

| Arquivo | Formato | Utilização no Agente |
|---------|---------|---------------------|
| `Projeto 1 - Ciclo 1` | XLSX | Contextualizar o histórico de satisfação do Projeto 1 no primeiro ciclo |
| `Projeto 1 - Ciclo 2` | XLSX | Comparar a evolução da satisfação do Projeto 1 entre ciclos |
| `Projeto 2 - Ciclo 1` | XLSX | Contextualizar o histórico de satisfação do Projeto 2 |
| `Projeto 3 - Ciclo 1` | XLSX | Contextualizar o histórico de satisfação do Projeto 3 |
| `Projeto 4 - Ciclo 1` | XLSX | Contextualizar o histórico de satisfação do Projeto 4 |

Cada registro (linha) representa a resposta de um aluno a uma pesquisa de satisfação, contendo os seguintes campos:

- **Projeto** e **Ciclo** — identificam a turma/edição do curso avaliada
- **Nome do Aluno**, **Turma/Módulo**, **Modalidade** e **% do curso concluído** — dados de contexto do respondente
- **Data/Hora do Registro** — quando a resposta foi enviada
- **CSAT (1 a 5)** e **NPS (0 a 10)** — métricas quantitativas de satisfação
- **Principal problema para permanência** — categoria marcada pelo aluno (Acesso a equipamentos, Plataforma utilizada, Qualidade do conteúdo, Didática de ensino ou Suporte)
- **Sugestão de melhoria** e **Problema/Reclamação relatado** — campos abertos de texto livre

---

## Adaptações nos Dados

Os dados são **sintéticos**, gerados programaticamente para simular pesquisas de satisfação reais, e passaram pelas seguintes adaptações intencionais:

- **Volume**: cada aba recebeu entre 105 e 130 registros (mínimo de 100 por planilha), com nomes de alunos, datas e distribuições geradas aleatoriamente.
- **Campos adicionais** além dos exigidos originalmente: `Turma/Módulo`, `Modalidade` (EAD, Presencial, Híbrido) e `% do curso concluído`, incluídos para enriquecer o contexto de análise de evasão.
- **Ruído proposital entre rótulo e texto livre**: nos campos abertos (sugestão e reclamação), cerca de 50% do texto é coerente com a categoria de problema marcada, ~20% reflete uma categoria diferente da marcada e ~30% é um comentário genérico/vago sem pista clara de categoria — simulando a subjetividade real de respostas humanas e servindo como conjunto de teste para o classificador Python (não para o LLM).
- **Correlação fraca entre CSAT e NPS**: em 20% dos registros o NPS é sorteado de forma independente do CSAT, para não gerar uma relação artificialmente perfeita entre as duas métricas.
- **Anonimização antes da categorização**: a coluna `Nome do Aluno` é removida (ou substituída por um ID interno) antes de os textos passarem pelo classificador e antes de qualquer dado ser exposto ao LLM de interação — nenhum nome de aluno é processado pelo modelo de linguagem.

---

## Estratégia de Integração

### Como os dados são carregados?

Os dados chegam por meio de uma integração via API (gspread) com as planilhas-fonte das pesquisas de satisfação, passando por um pipeline Python responsável por extrair, tratar, anonimizar e consolidar as respostas dos diferentes projetos e ciclos em uma única planilha.

**A categorização por tema é feita inteiramente em Python, sem uso de LLM**: um classificador treinado ou um conjunto de regras de palavras-chave atribui a categoria (Acesso a equipamentos, Plataforma utilizada, Qualidade do conteúdo, Didática de ensino, Suporte ou Sem categoria clara) a cada comentário aberto. Essa etapa roda de forma determinística, rápida e sem custo de inferência de modelo generativo.

O **Ollama entra só depois**, na camada de interação: quando o usuário faz uma pergunta na interface Streamlit, o modelo local recebe o recorte de dados já tratado e categorizado (filtrado por Python conforme a pergunta) e sua única função é interpretar a pergunta e formular a resposta em linguagem natural — ele nunca vê o dado bruto nem participa da categorização.

Esse pipeline é o que garante que a planilha final chegue já consolidada (uma aba por projeto/ciclo, com colunas padronizadas e categoria já preenchida pelo classificador). É esse arquivo consolidado — e não os dados brutos das planilhas-fonte — que funciona como a base de conhecimento do agente. No início da sessão da interface Streamlit, o agente lê todas as abas do arquivo consolidado e carrega os registros em memória (ex.: um DataFrame por aba ou um único DataFrame com a coluna `Projeto`/`Ciclo` já identificando a origem).

### Como os dados são usados no prompt?

Os dados não são inseridos integralmente no prompt do LLM (o volume de 580 linhas inviabilizaria isso, e modelos locais tendem a ser mais sensíveis a prompts longos do que LLMs de fronteira). A divisão de trabalho é:

- **Python calcula, o LLM explica.** Agregações numéricas (médias de CSAT/NPS, contagens por categoria, comparação entre ciclos) são sempre calculadas em código antes de chegar ao modelo — o Ollama nunca é solicitado a somar, contar ou calcular médias sozinho, apenas a interpretar números já prontos.
- **Filtragem por Python, não pelo LLM.** Quando o usuário pergunta sobre um projeto/ciclo/categoria específico, o filtro nos dados é feito em pandas antes de montar o prompt — apenas o recorte relevante entra no contexto enviado ao Ollama.
- O prompt final enviado ao modelo local combina: (1) o resumo agregado relevante à pergunta, já calculado, e (2) alguns exemplos de comentários daquele recorte (já categorizados pelo classificador Python), servindo apenas como ilustração qualitativa — nunca como fonte de novos rótulos.

---

## Exemplo de Contexto Montado

```
Registro de Pesquisa de Satisfação (já categorizado pelo pipeline Python):
- Projeto: Projeto 2 | Ciclo: 1
- Modalidade: EAD | Turma: C3
- % do curso concluído: 40%
- Data do registro: 01/03/2025 03:45
- CSAT: 3/5 | NPS: 3/10
- Categoria atribuída pelo classificador: Didática de ensino (confiança: 0.78)
- Sugestão de melhoria: "Pra mim, seria legal ter algum tipo de auxílio pra comprar um equipamento melhor."
- Reclamação relatada: "Nada que me faça pensar em desistir, só um cansaço geral."
```

> Observações:
> - O campo "Nome do Aluno" foi removido do contexto — a identificação do respondente não é enviada ao LLM.
> - A categoria já vem atribuída pelo classificador Python (com um score de confiança), o LLM não reclassifica — ele apenas usa essa informação para responder à pergunta do usuário.
