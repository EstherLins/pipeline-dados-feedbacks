# Base de Conhecimento — FeedbackFlow

# 1. Objetivo

A base de conhecimento define a estrutura dos dados utilizados pelo FeedbackFlow e como esses dados são disponibilizados ao agente.

O sistema foi desenvolvido para analisar pesquisas de satisfação de alunos de diferentes projetos e ciclos.

---

# 2. Fontes de Dados

O sistema possui inicialmente cinco planilhas de pesquisa de satisfação.

Cada planilha representa uma fonte de respostas.

As planilhas podem representar:

- diferentes projetos;
- diferentes ciclos;
- diferentes turmas;
- diferentes períodos.

A estrutura das planilhas pode variar entre as fontes.

Por isso, o pipeline Python realiza a padronização antes da consolidação.

---

# 3. Pipeline de Tratamento

O Python será responsável por:

```text
Planilha 1
     ↓
Planilha 2
     ↓
Planilha 3
     ↓
Planilha 4
     ↓
Planilha 5
     ↓
  Python
     ↓
Leitura dos dados
     ↓
Padronização dos nomes das colunas
     ↓
Tratamento dos tipos
     ↓
Tratamento de valores ausentes
     ↓
Normalização
     ↓
Padronização das categorias
     ↓
Tratamento das datas
     ↓
Anonimização
     ↓
Consolidação
     ↓
Google Sheets Consolidada
```

---

# 4. Google Sheets Consolidada

Após o tratamento, o Python envia os dados para uma Google Sheets consolidada.

Essa planilha passa a ser a principal fonte de dados utilizada pelas ferramentas analíticas do agente.

Estrutura:

```text
Planilhas de Pesquisa
        ↓
      Python
        ↓
Google Sheets Consolidada
        ↓
     Funções Python
        ↓
      FastAPI
        ↓
      Ollama
```

---

# 5. Campos dos Dados

A base consolidada deve possuir campos padronizados como:

- `projeto`
- `ciclo`
- `nome_aluno`
- `turma`
- `modalidade`
- `datahora_registro`
- `csat`
- `nps`
- `principal_problema_permanencia`
- `sugestao_melhoria`
- `problema_reclamacao`

Outros campos podem ser adicionados futuramente.

---

# 6. Padronização

O pipeline deve garantir que as cinco fontes utilizem o mesmo padrão.

Exemplo:

```text
Fonte 1:
Projeto

Fonte 2:
Nome do Projeto

Fonte 3:
Projeto/Curso
```

Após o tratamento:

```text
projeto
```

O mesmo princípio deve ser aplicado aos demais campos.

---

# 7. Tratamento de Categorias

As categorias devem possuir valores padronizados.

Exemplo:

```text
Acesso a equipamentos
Plataforma utilizada
Qualidade do conteúdo
Didática de ensino
Suporte
```

Variações de escrita devem ser normalizadas antes da consolidação.

---

# 8. Indicadores

O agente pode trabalhar com indicadores como:

## CSAT

Avaliação de satisfação do aluno.

Escala:

```text
1 a 5
```

---

## NPS

Indicador de recomendação.

Escala:

```text
0 a 10
```

---

# 9. Ferramentas do Agente

## calcular_metricas()

Retorna indicadores agregados.

Exemplo:

```json
{
  "projeto": "Projeto A",
  "ciclo": 2,
  "total_respostas": 120,
  "csat_medio": 4.1,
  "nps": 52
}
```

---

## comparar_ciclos()

Compara ciclos de um mesmo projeto.

Exemplo:

```json
{
  "projeto": "Projeto A",
  "ciclo_1": {
    "csat": 3.8,
    "nps": 41
  },
  "ciclo_2": {
    "csat": 4.1,
    "nps": 52
  }
}
```

---

## cruzar_variaveis()

Realiza cruzamentos entre variáveis.

Exemplo:

```text
Modalidade
×
Principal problema para permanência
```

---

## correlacionar_numericas()

Analisa a relação entre variáveis numéricas.

Exemplo:

```text
CSAT × NPS
```

---

## top_reclamacoes_categoria()

Identifica reclamações relacionadas a uma categoria.

Exemplo:

```text
Categoria:
Plataforma utilizada
```

---

# 10. Fluxo de Consulta

Quando o usuário faz uma pergunta:

```text
Usuário
   ↓
Lovable
   ↓
FastAPI
   ↓
Ollama
   ↓
Tool Calling
   ↓
FastAPI
   ↓
Função Python
   ↓
Google Sheets Consolidada
   ↓
Resultado
   ↓
Ollama
   ↓
FastAPI
   ↓
Lovable
   ↓
Usuário
```

---

# 11. Papel do LLM

O LLM não é responsável por:

- armazenar dados;
- limpar dados;
- normalizar dados;
- calcular métricas;
- alterar a planilha;
- excluir registros.

O LLM é responsável por:

- interpretar perguntas;
- identificar a intenção;
- escolher ferramentas;
- fornecer argumentos;
- interpretar resultados;
- gerar respostas em linguagem natural.

---

# 12. Fonte de Verdade

A fonte de dados do agente é:

```text
Google Sheets Consolidada
```

Porém, o Ollama não acessa essa planilha diretamente.

A consulta ocorre através das funções Python controladas pela API.

---

# 13. Segurança dos Dados

A Google Sheets Consolidada não deve ser exposta diretamente ao usuário final.

As credenciais utilizadas para acesso à planilha devem permanecer no backend.

O Lovable não deve receber:

- credenciais;
- tokens;
- chaves de API;
- credenciais do Google;
- informações internas da infraestrutura.

---

# 14. Princípio Central

```text
Planilhas
    ↓
Python
    ↓
Tratamento
    ↓
Google Sheets Consolidada
    ↓
Funções Python
    ↓
FastAPI
    ↓
Ollama
    ↓
FastAPI
    ↓
Lovable
    ↓
Usuário
```

A separação de responsabilidades deve ser preservada durante toda a evolução do projeto.
