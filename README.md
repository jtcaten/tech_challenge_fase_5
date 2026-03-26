# Classificação de Reclamações Financeiras e Análise das Dores dos Clientes

Projeto desenvolvido como parte do Tech Challenge (Fase 5) da Pós-Tech FIAP, com foco em **classificação automática do sentimento** de reclamações de clientes sobre produtos e serviços financeiros e na **identificação das principais dores por categoria de produto**.

---

## 1. Objetivo do Projeto

O objetivo é construir um pipeline completo de NLP e Deep Learning capaz de:

- Classificar o **sentimento** das reclamações de clientes (positivo ou negativo) a partir do texto da reclamação.
- Identificar as **principais dores** relatadas em cada categoria de produto financeiro (ex.: cartão de crédito, empréstimo, hipoteca).
- Gerar **visualizações** que ajudem a entender o sentimento dos clientes e os temas mais recorrentes.

---

## 2. Fonte de Dados

Os dados utilizados são provenientes da **Consumer Complaint Database** do CFPB (Consumer Financial Protection Bureau), acessados via API pública:

- Documentação da API:  
  https://cfpb.github.io/api/ccdb/

- Endpoint utilizado para coleta de dados (formato CSV):  
  `https://www.consumerfinance.gov/data-research/consumer-complaints/search/api/v1/`

Principais características:

- Reclamações de consumidores sobre produtos/serviços financeiros nos EUA.
- Campos importantes utilizados:
  - `Consumer complaint narrative` – texto livre da reclamação.
  - `Product` – categoria do produto financeiro.
  - `Company` – instituição financeira associada.
  - Metadados adicionais (data, estado, etc.).

O projeto coleta **apenas reclamações com narrativa disponível** e utiliza, por padrão, os **últimos N dias** (configurável via parâmetro no código).

---

## 3. Definição da Variável Alvo (Sentimento)

Como o dataset original não traz um rótulo explícito de sentimento, foi adotada uma estratégia de **rotulagem heurística** (*weak supervision*):

- A coluna `sentiment` é construída a partir de uma **regra baseada em palavras** presentes no texto da reclamação:
  - Lista de palavras associadas a feedbacks mais positivos (ex.: `resolved`, `thank`, `helpful`, `satisfied`, `good`, `great`, `appreciate`, etc.).
  - Lista de palavras associadas a experiências negativas (ex.: `error`, `fraud`, `complaint`, `unauthorized`, `charged`, `problem`, `issue`, `bad`, `poor`, `delay`, `denied`, etc.).
- Para cada reclamação, o texto é analisado e rotulado como:
  - `positive`: quando a contagem de termos positivos é maior que a de termos negativos (eventualmente com ajustes na lógica).
  - `negative`: caso contrário.

Observações importantes:

- Trata-se de uma aproximação, não de um “ground truth” perfeito.
- A base de reclamações tende naturalmente a ser **fortemente enviesada para o negativo**, o que gera desbalanceamento entre as classes.
- Essa abordagem é adequada ao contexto do desafio, que permite o uso de regras ou anotações manuais para construção da variável alvo.

---

## 4. Pipeline de Dados e NLP

Etapas principais do pipeline:

1. **Coleta de dados via API**
   - Parâmetros configuráveis:
     - `date_received_min`, `date_received_max` (janela de datas).
     - `has_narrative=true` (garante existência de texto).
     - `format=csv`, `size` e paginação (`page`) para lidar com volumes maiores.
   - Uso de cabeçalho `User-Agent` explícito para evitar erros 403 da API.

2. **Criação da variável alvo**
   - Aplicação da função `rule_based_sentiment` para criar a coluna `sentiment` (`positive` / `negative`).
   - Filtragem de linhas sem texto ou sem rótulo.

3. **Pré-processamento de texto**
   - Conversão para minúsculas.
   - Remoção de URLs, caracteres especiais e números.
   - Tokenização simples por palavras.
   - Remoção de *stopwords* em inglês.
   - Lematização usando `WordNetLemmatizer` (NLTK).
   - Criação de uma coluna `clean_text` com o texto limpo.

4. **Preparação para modelagem**
   - Conversão do `sentiment` em rótulo numérico (`negative = 0`, `positive = 1`).
   - Split treino/teste com estratificação.
   - Tokenização (`Tokenizer` do Keras) e conversão do texto em sequências numéricas.
   - *Padding* das sequências para comprimento fixo.

---

## 5. Modelagem (Deep Learning)

Modelo utilizado:

- Arquitetura em Keras/TensorFlow:
  - Camada `Embedding` para representar palavras em vetor denso.
  - Camada `Bidirectional LSTM` para capturar dependências no texto.
  - `GlobalMaxPool1D` para agregação.
  - Camadas densas com `ReLU` e `Dropout`.
  - Saída sigmoidal para classificação binária (positivo/negativo).

- Função de perda e otimização:
  - `binary_crossentropy`.
  - Otimizador `adam`.
  - Métrica principal: `accuracy` (com avaliação adicional via `precision`, `recall`, `f1-score`).

- Treinamento:
  - Divisão treino/validação.
  - Manejo de desbalanceamento via `class_weight` (opcional).
  - Número de épocas e batch size configuráveis.

---

## 6. Desempenho do Modelo

Exemplo de resultado (janela de 90 dias, rótulo heurístico baseado em dicionário de palavras):

- **Acurácia** em teste: ~0.99  
- Métricas por classe:
  - `negative`: precisão ~0.99, recall ~1.00, F1 ~1.00  
  - `positive`: precisão ~0.99, recall ~0.82, F1 ~0.90  

Embora os rótulos sejam heurísticos e a base desbalanceada, o modelo é capaz de:

- Capturar bem os padrões textuais associados aos poucos casos rotulados como positivos.
- Manter ótimo desempenho global, o que é coerente com a literatura de sentimento em bases desbalanceadas (uso de F1 e métricas por classe, em vez de só acurácia).[web:94][web:87][web:96]

---

## 7. Análise das Dores dos Clientes

Com o modelo treinado e o target disponível, foram realizadas as seguintes análises:

- **Distribuição de sentimentos por produto**
  - Contagem de reclamações positivas/negativas por categoria de produto (`Product`).
  - Gráficos de barras evidenciando quais produtos concentram mais insatisfações.

- **Frequência de termos em reclamações negativas**
  - Cálculo das palavras mais frequentes em `clean_text` para a classe `negative`.
  - Gráfico de barras com as 20 palavras mais comuns.

- **Nuvens de palavras por produto**
  - Geração de word clouds para reclamações negativas em cada um dos principais produtos.
  - Identificação visual de temas recorrentes (ex.: cobrança indevida, atraso, fraude, etc.).

Essas visualizações ajudam a responder:  
> “Quais são as principais dores dos clientes em cada categoria de produto financeiro?”

---

## 8. Como Reproduzir o Projeto

### 8.1. Pré-requisitos

- Python 3.9+  
- Bibliotecas principais:
  - `requests`, `pandas`, `numpy`
  - `nltk`
  - `tensorflow` / `keras`
  - `scikit-learn`
  - `matplotlib`, `seaborn`, `wordcloud`

Exemplo de instalação básica:

```bash
pip install -r requirements.txt
# ou, em ambiente controlado:
# pip install requests pandas numpy nltk tensorflow scikit-learn matplotlib seaborn wordcloud
