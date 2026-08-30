# Documentação do Agente — FeedbackFlow

# 1. Visão Geral

O FeedbackFlow é um agente de análise de pesquisas de satisfação acadêmica.

Seu objetivo é permitir que gestores e responsáveis pelos projetos consultem informações sobre os feedbacks dos alunos utilizando linguagem natural.

O sistema combina:

- Python;
- Google Sheets;
- FastAPI;
- Ollama;
- Lovable.

---

# 2. Problema

As pesquisas de satisfação são coletadas em diferentes planilhas.

Isso pode gerar:

- dados dispersos;
- estruturas diferentes;
- nomes de campos inconsistentes;
- dificuldade de consolidação;
- dificuldade de comparação entre ciclos;
- dificuldade de identificação de padrões;
- análises manuais demoradas.

---

# 3. Solução

O sistema utiliza um pipeline Python para transformar as três planilhas de pesquisa em uma única base consolidada.

O fluxo é:

```text
5 Planilhas de Pesquisa
          ↓
      Pipeline Python
          ↓
Tratamento e Normalização
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

---

# 4. Arquitetura

```text
┌───────────────────────┐
│  PLANILHA DE PESQUISA │
│          1            │
└───────────┬───────────┘
            │
┌───────────────────────┐
│  PLANILHA DE PESQUISA │
│          2            │
└───────────┬───────────┘
            │
┌───────────────────────┐
│  PLANILHA DE PESQUISA │
│          3            │
└───────────┬───────────┘
            │
┌───────────────────────┐
│  PLANILHA DE PESQUISA │
│          4            │
└───────────┬───────────┘
            │
┌───────────────────────┐
│  PLANILHA DE PESQUISA │
│          5            │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│     PIPELINE PYTHON   │
│                       │
│ • Tratamento          │
│ • Normalização        │
│ • Padronização        │
│ • Anonimização        │
│ • Consolidação        │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│ GOOGLE SHEETS         │
│ CONSOLIDADA           │
│                       │
│ Fonte de dados        │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│    FUNÇÕES PYTHON     │
│                       │
│ • Métricas            │
│ • Comparações         │
│ • Cruzamentos         │
│ • Correlações         │
│ • Reclamações         │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│       FASTAPI         │
│                       │
│ • Autenticação        │
│ • Validação           │
│ • Controle            │
│ • Comunicação         │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│        OLLAMA         │
│                       │
│ • Interpretação       │
│ • Tool Calling        │
│ • Geração de resposta │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│       FASTAPI         │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│        LOVABLE        │
│                       │
│ • Chat                │
│ • Dashboard           │
│ • Gráficos            │
│ • Filtros             │
└───────────┬───────────┘
            │
            ▼
         USUÁRIO
```

---

# 5. Responsabilidade de Cada Camada

## Python — Pipeline

Responsável por:

- ler as três planilhas;
- tratar os dados;
- normalizar;
- padronizar;
- anonimizar;
- consolidar;
- enviar os dados para a Google Sheets consolidada.

---

## Google Sheets

Responsável por:

- armazenar os dados consolidados;
- servir como fonte para as consultas das funções Python.

A Google Sheets não deve ser exposta diretamente ao usuário.

---

## Python — Ferramentas

Responsável por:

- consultar os dados;
- realizar cálculos;
- gerar indicadores;
- executar cruzamentos;
- realizar análises autorizadas.

---

## FastAPI

Responsável por:

- receber solicitações do Lovable;
- controlar autenticação;
- validar requisições;
- controlar ferramentas;
- conversar com o Ollama;
- executar funções Python;
- retornar respostas.

---

## Ollama

Responsável por:

- interpretar a pergunta;
- identificar a intenção;
- selecionar ferramentas;
- solicitar os parâmetros;
- interpretar os resultados;
- gerar a resposta final.

---

## Lovable

Responsável pela interface.

Pode possuir:

- login;
- chat;
- histórico.

---

# 6. Fluxo Completo de uma Pergunta

Usuário:

> Qual foi o principal problema do Projeto A no Ciclo 2?

### Etapa 1

O usuário envia a pergunta pelo Lovable.

### Etapa 2

O Lovable envia a pergunta para:

```http
POST /chat
```

### Etapa 3

A FastAPI recebe e valida a solicitação.

### Etapa 4

A pergunta é enviada ao Ollama.

### Etapa 5

O Ollama identifica a ferramenta necessária.

Exemplo:

```text
calcular_metricas(
    projeto="Projeto A",
    ciclo=2
)
```

### Etapa 6

A FastAPI valida os parâmetros.

### Etapa 7

A função Python consulta a Google Sheets consolidada.

### Etapa 8

A função retorna o resultado.

Exemplo:

```json
{
  "projeto": "Projeto A",
  "ciclo": 2,
  "total_respostas": 115,
  "categoria_principal": "Plataforma utilizada",
  "percentual": 38
}
```

### Etapa 9

O resultado retorna para o Ollama.

### Etapa 10

O Ollama gera:

> No Projeto A, Ciclo 2, o principal problema identificado foi a Plataforma utilizada, representando 38% dos registros analisados.

### Etapa 11

A FastAPI envia a resposta ao Lovable.

### Etapa 12

O Lovable apresenta a resposta.

---

# 7. Endpoint Principal

O endpoint inicial pode ser:

```http
POST /chat
```

Requisição:

```json
{
  "mensagem": "Qual foi o principal problema do Projeto A no Ciclo 2?"
}
```

Resposta:

```json
{
  "resposta": "No Projeto A, Ciclo 2, o principal problema identificado foi a Plataforma utilizada, representando 38% dos registros analisados.",
  "projeto": "Projeto A",
  "ciclo": 2,
  "ferramentas_utilizadas": [
    "calcular_metricas"
  ]
}
```

---

# 8. Segurança

## Lovable

O Lovable não deve possuir:

- credenciais do Google;
- tokens do Ollama;
- credenciais da API;
- credenciais da Google Sheets.

---

## Ollama

O Ollama não deve:

- acessar diretamente a Google Sheets;
- acessar diretamente credenciais;
- executar código arbitrário;
- alterar dados;
- excluir dados.

---

## FastAPI

A FastAPI deve:

- validar usuários;
- validar requisições;
- controlar permissões;
- controlar ferramentas;
- validar argumentos;
- proteger credenciais;
- controlar acesso aos dados.

---

# 9. Privacidade

O pipeline deve anonimizar ou remover informações pessoais antes que os dados sejam disponibilizados às ferramentas.

Sempre que possível, o agente deve trabalhar com:

- totais;
- médias;
- percentuais;
- categorias;
- distribuições;
- comparações agregadas.

---

# 10. Anti-Alucinação

O sistema deve seguir:

```text
LLM NÃO CALCULA
        ↓
Python CALCULA
        ↓
Resultado retorna
        ↓
LLM EXPLICA
```

Regras:

1. Não inventar números.
2. Não estimar valores.
3. Não criar dados inexistentes.
4. Não inventar categorias.
5. Não inventar respostas de alunos.
6. Não responder indicadores sem consultar a ferramenta.
7. Não executar código arbitrário.
8. Não acessar dados diretamente.
9. Informar quando não houver dados.
10. Respeitar o escopo das ferramentas.

---

# 15. Princípio Central

A arquitetura deve manter uma separação clara:

```text
PYTHON
Tratamento + Normalização + Cálculos
              ↓
GOOGLE SHEETS
Dados Consolidados
              ↓
FASTAPI
Controle + Segurança + Orquestração
              ↓
OLLAMA
Interpretação + Tool Calling
              ↓
FASTAPI
Resposta Controlada
              ↓
LOVABLE
Interface
              ↓
USUÁRIO
Consulta + Decisão
```

O objetivo é utilizar o LLM como uma camada de interpretação e interação, mantendo o processamento dos dados e os cálculos sob controle do código Python.
