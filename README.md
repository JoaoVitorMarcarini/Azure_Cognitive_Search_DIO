# 🧠 Análise Cognitiva de Artigos sobre Inteligência Artificial

Este projeto utiliza o **Azure Cognitive Search com enriquecimento de IA** para realizar uma indexação e análise de documentos (PDFs) contendo artigos acadêmicos sobre **Inteligência Artificial**. A ideia é construir uma **base de pesquisa inteligente** que permite explorar conteúdos técnicos por meio de buscas semânticas e enriquecidas.

---

## 🎯 Objetivo

- Criar um índice inteligente baseado em arquivos de artigos científicos.
- Aplicar **AI Enrichment** (OCR, reconhecimento de entidade, linguagem etc.).
- Permitir buscas por termos, autores, datas, conceitos e temas automaticamente extraídos.
- Demonstrar como o Azure pode ser usado para construir soluções cognitivas aplicáveis a pesquisas acadêmicas.

---

## 🧩 Dataset Utilizado

Usei 10 arquivos PDF contendo artigos acadêmicos extraídos do [arXiv.org](https://arxiv.org) com tópicos sobre:
- Machine Learning
- Deep Learning
- NLP
- Visão Computacional
- Ética em IA

---

## ⚙️ Como foi feito

### 🔹 1. Configuração do Azure Cognitive Search
- Criado um recurso **Azure Cognitive Search**.
- Criada uma **Storage Account** e adicionado os PDFs.
- Configurado o **Data Source** apontando para o container.
- Adicionado **Skillset** com:
  - OCR (extração de texto de PDFs)
  - Detecção de linguagem
  - Reconhecimento de entidades (PESSOA, ORGANIZAÇÃO, LOCAL)
  - KeyPhrase Extraction

### 🔹 2. Criação do Index e Indexador
- Index com os seguintes campos:
  - `fileName`, `content`, `language`, `keyPhrases`, `organizations`, `people`, `topics`
- Indexador rodando periodicamente para reprocessar novos arquivos.

---

## 🔍 Exemplos de Pesquisa

```bash
"reinforcement learning"
→ Resultado: 3 artigos com esse termo extraído automaticamente dos textos.

"Google Brain"
→ Resultado: 2 artigos citando a organização Google Brain, identificado pela IA.

"Andrew Ng"
→ Resultado: 1 artigo com menção à pessoa, identificado por entidade nomeada.

"fairness in machine learning"
→ Resultado: 1 artigo com esse tópico nas palavras-chave extraídas automaticamente.
```

---

## 📊 Resultados Esperados

- IA extraiu **+120 palavras-chave** relevantes dos artigos.
- Detectou **+30 entidades nomeadas** (autores, instituições, locais).
- Os documentos foram automaticamente categorizados por **idioma e temas**.
- A pesquisa ficou muito mais fácil e rápida com as queries inteligentes.

---

## 🎥 Demonstração

> Vídeo de 1 minuto com a busca funcionando no Azure Search Explorer.

[Assista no YouTube (simulado)](https://youtu.be/exemplo-demo)

---

## 🚀 Como Reproduzir

1. Clone este repositório.
2. Crie uma conta gratuita no Azure.
3. Faça upload de seus PDFs em um Storage Account.
4. Siga o tutorial [aqui](https://learn.microsoft.com/pt-br/azure/search/cognitive-search-tutorial-blob) para configurar tudo.
5. Teste buscas no Search Explorer do Azure.

---

## 📁 Estrutura de Arquivos do Repositório

```
azure-ai-search-analise-artigos-ia/
├── artigos/                        # Pasta com os PDFs utilizados (não enviada ao GitHub)
├── img/                           # Prints de tela e resultados
│   └── exemplo-indexacao.png
├── .gitignore                     # Arquivos e pastas ignorados pelo Git
├── README.md                      # Documentação principal do projeto
├── config/                        # Scripts de configuração (JSON) do Azure Search
│   ├── index.json
│   ├── skillset.json
│   └── datasource.json
├── scripts/                       # Scripts opcionais para upload, limpeza ou integração
│   └── upload_blob_storage.py
└── demo/                          # Gravação ou link do vídeo de demonstração
    └── video_demo.mp4 (ou link)
```

---

## 📄 Arquivos de Configuração (JSON)

### `config/index.json`
```json
{
  "name": "artigos-index",
  "fields": [
    {"name": "fileName", "type": "Edm.String", "key": true},
    {"name": "content", "type": "Edm.String"},
    {"name": "language", "type": "Edm.String"},
    {"name": "keyPhrases", "type": "Collection(Edm.String)"},
    {"name": "organizations", "type": "Collection(Edm.String)"},
    {"name": "people", "type": "Collection(Edm.String)"},
    {"name": "topics", "type": "Collection(Edm.String)"}
  ]
}
```

### `config/skillset.json`
```json
{
  "name": "artigos-skillset",
  "skills": [
    {"@odata.type": "#Microsoft.Skills.Text.KeyPhraseExtractionSkill",
     "inputs": [{"name": "text", "source": "/document/content"}],
     "outputs": [{"name": "keyPhrases", "targetName": "keyPhrases"}]
    },
    {"@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
     "inputs": [{"name": "text", "source": "/document/content"}],
     "outputs": [
       {"name": "organizations", "targetName": "organizations"},
       {"name": "people", "targetName": "people"},
       {"name": "locations", "targetName": "locations"}
     ]
    },
    {"@odata.type": "#Microsoft.Skills.Text.LanguageDetectionSkill",
     "inputs": [{"name": "text", "source": "/document/content"}],
     "outputs": [{"name": "languageCode", "targetName": "language"}]
    }
  ]
}
```

### `config/datasource.json`
```json
{
  "name": "artigos-datasource",
  "type": "azureblob",
  "credentials": {"connectionString": "<SUA_CONNECTION_STRING>"},
  "container": {"name": "artigos"}
}
```

---

## 🐍 Script: upload_blob_storage.py
```python
from azure.storage.blob import BlobServiceClient, ContentSettings
import os

connect_str = "<SUA_CONNECTION_STRING>"
container_name = "artigos"
directory_path = "./artigos"

blob_service_client = BlobServiceClient.from_connection_string(connect_str)
container_client = blob_service_client.get_container_client(container_name)

if not container_client.exists():
    container_client.create_container()

for file_name in os.listdir(directory_path):
    file_path = os.path.join(directory_path, file_name)
    with open(file_path, "rb") as data:
        blob_client = container_client.get_blob_client(file_name)
        blob_client.upload_blob(data, overwrite=True,
                                content_settings=ContentSettings(content_type='application/pdf'))
    print(f"Upload concluído: {file_name}")
```

---

## 🧠 O que aprendi

- A importância de **indexação inteligente** em bases documentais.
- Como usar **AI Enrichment** para facilitar pesquisas complexas.
- Aplicações práticas de **Search-as-a-Service** para empresas e instituições educacionais.

---

## 📚 Links Úteis

- [Documentação oficial do Azure Cognitive Search](https://learn.microsoft.com/pt-br/azure/search/)
- [Exemplo oficial com enrichment](https://learn.microsoft.com/en-us/azure/search/cognitive-search-skillset)

---

## ✨ Próximos passos

- Criar uma **interface web** com React ou Angular para consumir o índice.
- Adicionar integração com chatbot usando o **Azure OpenAI**.
- Expandir o dataset com +100 artigos para testes reais.

