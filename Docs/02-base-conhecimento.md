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

- **Volume**: cada aba recebeu entre 105 e 130 registros (mínimo de 100 por planilha).
- **Campos adicionais**: `Turma/Módulo`, `Modalidade` (EAD, Presencial, Híbrido) e `% do curso concluído`, incluídos para enriquecer o contexto de análise de evasão e servir de variáveis para os cruzamentos feitos pelas ferramentas do agente.
- **Ruído proposital entre rótulo e texto livre**: nos campos abertos, cerca de 50% do texto é coerente com a categoria marcada, ~20% reflete uma categoria diferente e ~30% é um comentário genérico/vago — usado como conjunto de teste do classificador Python.
- **Correlação fraca entre CSAT e NPS**: em 20% dos registros o NPS é sorteado de forma independente do CSAT.
- **Anonimização antes da categorização**: a coluna `Nome do Aluno` é removida (ou substituída por um ID interno) antes de os textos passarem pelo classificador e antes de a planilha consolidada ficar disponível para as ferramentas do agente — nenhuma ferramenta exposta ao LLM retorna nome de aluno.

---

## Estratégia de Integração

### Como os dados são carregados?

Os dados chegam por meio de uma integração via API (gspread) com as planilhas-fonte, passando por um pipeline Python que extrai, trata, anonimiza e consolida as respostas em uma única planilha. A categorização por tema é feita inteiramente em Python (classificador treinado ou regras), sem uso de LLM.

**A planilha consolidada não é entregue diretamente ao LLM.** Em vez disso, ela é carregada em memória (pandas DataFrame) por um módulo de **ferramentas** (`ferramentas.py`), que expõe um conjunto fechado de funções para consultar e cruzar os dados:

| Ferramenta | O que faz |
|---|---|
| `calcular_metricas(projeto, ciclo)` | CSAT médio, NPS médio, total de registros e distribuição de categorias de um projeto/ciclo |
| `comparar_ciclos(projeto)` | Compara as métricas entre todos os ciclos de um mesmo projeto |
| `cruzar_variaveis(coluna_a, coluna_b, projeto)` | Tabela de contingência entre duas colunas categóricas (ex.: Modalidade x Categoria) |
| `correlacionar_numericas(coluna_a, coluna_b, projeto)` | Correlação entre duas colunas numéricas (CSAT, NPS, % do curso concluído) |
| `top_reclamacoes_categoria(categoria, projeto, limite)` | Retorna exemplos reais de reclamações de uma categoria específica |

O Ollama recebe o schema dessas ferramentas junto com a pergunta do usuário. Ele decide qual ferramenta chamar e com quais argumentos; a função roda de verdade sobre o DataFrame consolidado; o resultado numérico real volta para o modelo, que formula a resposta final. Esse ciclo (pergunta → escolha da ferramenta → execução real → resposta) é repetido até um limite de rodadas (ex.: 3) para permitir perguntas que exigem mais de uma consulta (como comparar dois ciclos), sem correr risco de loop infinito.

### Como os dados são usados no prompt?

O prompt enviado ao Ollama nunca contém a planilha ou um recorte bruto dela. Ele contém:

1. O **system prompt** com as regras do agente e a lista de ferramentas disponíveis (schema JSON).
2. A **pergunta do usuário**.
3. Após a primeira resposta do modelo (que pede uma chamada de ferramenta), o **resultado real** dessa chamada é anexado à conversa como uma mensagem do tipo `tool`, e uma nova chamada ao modelo é feita para ele formular a resposta final com base nesse resultado.

Isso significa que o LLM nunca "vê" os 580 registros de uma vez — ele só enxerga o resultado já processado e filtrado da ferramenta que ele mesmo escolheu chamar.

---

## Exemplo de Contexto Montado

Fluxo real de uma pergunta, mostrando as mensagens trocadas com o Ollama:

```
[system] Você é um agente de análise de satisfação acadêmica. Nunca calcule
números sozinho — sempre use as ferramentas disponíveis.

[user] Como está o Projeto 2 no Ciclo 1?

[assistant → tool_call] calcular_metricas(projeto="Projeto 2", ciclo=1)

[tool → resultado real, calculado em pandas]
{
  "registros": 120,
  "csat_medio": 3.4,
  "nps_medio": 4.1,
  "categoria_mais_frequente": "Qualidade do conteúdo",
  "distribuicao_categorias": {
    "Qualidade do conteúdo": 34,
    "Plataforma utilizada": 28,
    "Suporte": 25,
    "Didática de ensino": 20,
    "Acesso a equipamentos": 13
  }
}

[assistant → resposta final]
No Projeto 2 - Ciclo 1 (120 registros), o CSAT médio foi 3,4 e o NPS médio
foi 4,1. A categoria mais citada como principal problema foi "Qualidade do
conteúdo" (34 registros), seguida de "Plataforma utilizada" (28).
```

> Observação: em nenhum momento o nome de um aluno, ou a planilha bruta, passa pelo modelo — apenas o resultado agregado e já anonimizado da ferramenta chamada.
