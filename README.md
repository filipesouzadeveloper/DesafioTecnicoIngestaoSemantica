# Ingestão e Busca Semântica com LangChain e Postgres

**Aluno:** Filipe Souza — MBA em Engenharia de Software com IA

Projeto de RAG (Retrieval-Augmented Generation) desenvolvido como entregável do MBA. Permite ingerir um PDF em um banco de dados vetorial (PostgreSQL + pgVector) e responder perguntas via CLI com base exclusivamente no conteúdo do documento.

---

## Tecnologias

- Python 3.10+
- LangChain
- PostgreSQL + pgVector
- Docker & Docker Compose
- Google Gemini (LLM + embeddings) — tier gratuita disponível

---

## Estrutura do Projeto

```
├── docker-compose.yml
├── requirements.txt
├── .env
├── .gitignore
├── src/
│   ├── ingest.py    # Ingestão do PDF no banco vetorial
│   ├── search.py    # Lógica de busca semântica e chamada à LLM
│   └── chat.py      # Interface CLI de perguntas e respostas
├── document.pdf     # PDF a ser ingerido (adicione o seu)
└── README.md
```

---

## Pré-requisitos

- [Docker](https://www.docker.com/) instalado e em execução
- [Python 3.10+](https://www.python.org/) instalado
- Chave de API do Google AI (gratuita em [aistudio.google.com](https://aistudio.google.com))

---

## Configuração

### 1. Clone o repositório

```bash
git clone <url-do-repositorio>
cd <nome-do-repositorio>
```

### 2. Adicione o PDF

Coloque o arquivo PDF que deseja ingerir na raiz do projeto com o nome `document.pdf`.

### 3. Obtenha a chave do Google AI (gratuita)

1. Acesse [aistudio.google.com](https://aistudio.google.com)
2. Clique em **Get API key** → **Create API key**
3. Copie a chave gerada

### 4. Configure as variáveis de ambiente

Edite o arquivo `.env` com suas credenciais:

```env
GOOGLE_API_KEY=AIza...          # Sua chave do Google AI Studio
GOOGLE_EMBEDDING_MODEL=models/embedding-001
GOOGLE_LLM_MODEL=gemini-2.5-flash

DATABASE_URL=postgresql+psycopg://postgres:postgres@localhost:5432/rag
PG_VECTOR_COLLECTION_NAME=documents

PDF_PATH=document.pdf
```

---

## Ordem de Execução

### 1. Suba o banco de dados

```bash
docker compose up -d
```

Aguarde o container ficar saudável (alguns segundos).

### 2. Instale as dependências Python

```bash
pip install -r requirements.txt
```

### 3. Execute a ingestão do PDF

```bash
python src/ingest.py
```

Saída esperada:
```
Carregando PDF: document.pdf
12 página(s) carregada(s).
48 chunk(s) gerado(s).
Armazenando vetores no banco de dados...
Ingestão concluída: 48 chunk(s) armazenado(s) na coleção 'documents'.
```

### 4. Inicie o chat

```bash
python src/chat.py
```

---

## Exemplo de Uso

```
Chat iniciado. Digite 'sair' para encerrar.

PERGUNTA: Qual o faturamento da Empresa SuperTechIABrazil?
RESPOSTA: O faturamento foi de 10 milhões de reais.

PERGUNTA: Quantos clientes temos em 2024?
RESPOSTA: Não tenho informações necessárias para responder sua pergunta.

PERGUNTA: sair
Encerrando chat.
```

---

## Como Funciona

1. **Ingestão** (`ingest.py`): O PDF é carregado, dividido em chunks de 1000 caracteres com overlap de 150, convertido em embeddings via Google AI (`embedding-001`) e armazenado no PostgreSQL com pgVector.

2. **Busca** (`search.py`): A pergunta do usuário é vetorizada via Google AI, os 10 chunks mais relevantes são recuperados por similaridade e enviados como contexto para o Gemini junto com um prompt estrito que proíbe respostas fora do documento.

3. **Chat** (`chat.py`): Loop interativo no terminal que recebe perguntas e exibe respostas.
