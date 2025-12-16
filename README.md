# 🏋️‍♂️ AI Fitness Coach — Agent + RAG + PDF (n8n + Gemini + Supabase)

Este repositório documenta um **workflow avançado no n8n** que implementa um **Agente de IA Fitness**, combinando:

* 🤖 **AI Agent (Gemini Chat)**
* 📚 **RAG (Retrieval-Augmented Generation)** com Supabase Vector Store
* 📂 **Ingestão automática de documentos** (Google Drive)
* 📄 **Extração e chunking de PDFs / Docs / Sheets**
* 🧠 **Embeddings com Google Gemini**
* 💬 **Chat público via Webhook**
* 🗂️ **Geração de planos de treino personalizados (12 treinos)**

O fluxo foi desenhado para **produção**, com foco em escalabilidade, reutilização e documentação técnica clara.

---

## 📌 Visão Geral da Arquitetura

### 🔷 Diagrama visual do fluxo (Mermaid — GitHub)

> O diagrama abaixo pode ser colado diretamente no **README.md** do GitHub.
>
> O GitHub renderiza automaticamente blocos `mermaid`.

````mermaid
graph TD

    %% Triggers
    A[Schedule Trigger<br/>Indexação RAG] --> B[Google Drive<br/>Search Files]
    Z[Chat Trigger<br/>Webhook Público] --> Y[AI Agent]

    %% Ingestão de documentos
    B --> C[Loop Over Items]
    C --> D[Set File Metadata]
    D --> E[Download File<br/>PDF / Docs / Sheets]

    %% Roteamento por tipo
    E --> F{Switch<br/>MIME Type}
    F -->|PDF| G[Extract from File]
    F -->|Docs| G
    F -->|Sheets| H[Extract from File]

    %% Processamento de texto
    G --> I[Recursive Text Splitter]
    I --> J[Default Data Loader]

    %% Embeddings
    J --> K[Gemini Embeddings]

    %% Vector Store
    K --> L[Supabase Vector Store<br/>Insert Documents]

    %% RAG como ferramenta
    L --> M[Supabase Vector Store<br/>Retrieve as Tool]
    M --> Y

    %% Agent
    Y --> N[Google Gemini Chat Model]
    Y --> O[Memory Buffer]

    %% Saída
    Y --> P[Plano de Treino<br/>HTML Semântico → PDF]
```text
[Schedule Trigger]
        ↓
[Google Drive Folder]
        ↓
[Download + Conversão PDF]
        ↓
[Extract Text]
        ↓
[Text Splitter]
        ↓
[Embeddings Gemini]
        ↓
[Supabase Vector Store]
        ↓
[RAG disponível como Tool]
        ↓
[AI Agent Gemini]
        ↓
[Resposta em HTML → PDF]
````

Além disso, o fluxo possui um **Chat Trigger público**, permitindo que usuários finais interajam com o agente em tempo real.

---

## 🧩 Componentes do Workflow

### 1️⃣ Triggers

#### ⏰ Schedule Trigger

* Responsável por **indexar periodicamente** os documentos do Google Drive.
* Ideal para manter a base RAG sempre atualizada.

#### 💬 Chat Trigger (Webhook)

* Permite expor o agente como **chat público**.
* Recebe mensagens e dados do usuário (idade, peso, objetivo, frequência de treino).

---

### 2️⃣ Ingestão de Conhecimento (RAG)

#### 📂 Google Drive — Search files and folders

* Varre uma pasta específica no Google Drive.
* Suporta:

  * PDFs
  * Google Docs
  * Google Sheets

#### 🔁 Loop Over Items

* Processa cada arquivo individualmente.
* Evita sobrecarga e permite controle fino.

#### 📥 Download file

* Faz download dos arquivos.
* Converte automaticamente Google Docs em PDF.

#### 🔀 Switch (por tipo de arquivo)

* Direciona o fluxo conforme o MIME Type:

  * `application/pdf`
  * `application/vnd.google-apps.document`
  * `application/vnd.google-apps.spreadsheet`

---

### 3️⃣ Extração e Processamento de Texto

#### 📄 Extract from File

* Extrai texto bruto dos arquivos.

#### ✂️ Recursive Character Text Splitter

* Divide o texto em chunks menores.
* Configurado com **overlap** para preservar contexto semântico.

#### 📦 Default Data Loader

* Prepara os documentos no formato aceito pelo LangChain.

---

### 4️⃣ Embeddings e Vector Store

#### 🧠 Embeddings Google Gemini

* Gera embeddings vetoriais de alta qualidade.

#### 🗄️ Supabase Vector Store (Insert)

* Armazena documentos vetorizados na tabela `documents`.
* Usa a função SQL `match_documents` para similaridade semântica.

#### 🔍 Supabase Vector Store (Retrieve as Tool)

* Expõe o RAG como uma **ferramenta** utilizável pelo agente.

---

### 5️⃣ AI Agent — Fitness Coach

#### 🤖 AI Agent (LangChain)

**Papel:** Treinador físico profissional, especialista em periodização.

**Responsabilidades:**

* Interpretar dados do usuário
* Consultar o RAG quando necessário
* Gerar um plano de treino seguro e progressivo

#### 📜 Prompt do Sistema (resumo)

* Gera **12 treinos personalizados**
* Considera:

  * Idade
  * Peso
  * Dias de treino/semana
  * Objetivo (hipertrofia, emagrecimento, força, condicionamento)
* Saída:

  * Markdown estruturado
  * HTML semântico
  * Pronto para conversão em PDF

---

### 6️⃣ Memória e Modelo

#### 🧠 Simple Memory

* Mantém contexto da conversa
* Melhora continuidade e experiência do usuário

#### 💬 Google Gemini Chat Model

* Modelo principal de linguagem
* Configurado com limite de tokens para segurança

---

## 📄 Saída do Sistema

O agente retorna:

* 📌 Título do plano
* 👤 Dados do aluno
* 📅 Estrutura semanal
* 🏋️ Detalhamento dos 12 treinos
* ⚠️ Observações e recomendações

Formato final:

* HTML semântico
* Pronto para geração de PDF

---

## 🛠️ Pré-requisitos

* n8n (self-hosted ou cloud)
* Conta Google (Drive + Gemini API)
* Supabase com:

  * Extensão `vector`
  * Tabela `documents`
  * Função `match_documents`

---

## 🚀 Como usar

1. Importe o workflow JSON no n8n
2. Configure as credenciais:

   * Google Drive OAuth
   * Google Gemini API
   * Supabase
3. Ative o workflow
4. Aguarde a indexação dos documentos
5. Use o Chat Trigger para interagir com o agente

---

## 📚 Casos de Uso

* Personal Trainer digital
* Healthtechs e Wellness Apps
* MVPs de IA aplicada à saúde
* Estudos de RAG com n8n

---

## 📜 Licença

Este projeto pode ser adaptado para uso educacional, experimental ou comercial.

---

✨ *Workflow criado para demonstrar o uso profissional de IA, RAG e automação no n8n.*
