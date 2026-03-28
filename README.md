# 📊 Classificação de Sentimento em Reclamações de Consumidores (CFPB)

## 1. 📌 Introdução

Este projeto trata-se da entrega do Tech Challenge da Fase 5 do Grupo 15, composto por Jarbas Ten Caten e Paulo Sergio Xavier Santos.

Tem como objetivo analisar reclamações de consumidores utilizando técnicas de Processamento de Linguagem Natural (NLP) e aprendizado de máquina.

Os dados são obtidos por meio da API pública do Consumer Financial Protection Bureau (CFPB), contendo descrições textuais das reclamações registradas por consumidores.

O projeto realiza:

* Coleta de dados via API
* Rotulagem automática de sentimento (positivo ou negativo)
* Pré-processamento de texto
* Treinamento de um modelo de Deep Learning (LSTM)
* Avaliação de desempenho
* Análise exploratória das principais dores dos clientes por categoria de produto

---

## 2. 📥 Coleta de Dados

A coleta é realizada diretamente da API do CFPB, filtrando reclamações recentes.

Características da coleta:

* Dados dos últimos **90 dias**
* Paginação em múltiplas requisições
* Tamanho máximo de coleta limitado para evitar sobrecarga
* Uso de `User-Agent` para evitar bloqueios (erro 403)

Os dados coletados incluem:

* Produto
* Data da reclamação
* Descrição do consumidor (`Consumer complaint narrative`)

---

## 3. 🏷️ Rotulagem de Sentimento (Heurística)

Como o dataset não possui rótulos de sentimento, foi utilizada uma abordagem baseada em regras.

A classificação é feita com base na presença e **frequência relativa** de palavras-chave associadas a sentimentos positivos e negativos.

* **Negativo:** termos como *error*, *fraud*, *problem*, *unauthorized*, *charge*, *fee*, *denied*, *delay*, entre outros relacionados a falhas, cobranças indevidas e insatisfação.
* **Positivo:** termos como *thank*, *helpful*, *resolved*, *great*, *satisfied*, *efficient*, entre outros relacionados a experiências bem-sucedidas.

A decisão final considera a quantidade de ocorrências de termos positivos e negativos no texto:

* se houver predominância de termos negativos → **negativo**
* caso contrário → **positivo**

⚠️ Observação:
Essa abordagem permite viabilizar o treinamento do modelo, porém:

* pode introduzir ruído nos rótulos;
* não captura contexto semântico (ex: negações ou ironia);
* depende diretamente da cobertura das listas de palavras utilizadas.

---

## 4. 🧹 Pré-processamento de Texto

O texto das reclamações passa pelas seguintes etapas:

* Conversão para minúsculas
* Remoção de pontuação e caracteres especiais
* Remoção de stopwords (NLTK)
* Lematização
* Tokenização
* Padding para tamanho fixo

O texto processado é armazenado na coluna `clean_text`, que é utilizada tanto no treinamento do modelo quanto nas análises exploratórias.

Dependências adicionais do NLTK são baixadas automaticamente:

* `stopwords`
* `wordnet`
* `omw-1.4`

---

## 5. 🤖 Modelagem com Deep Learning

O modelo utilizado é baseado em redes neurais recorrentes:

**Arquitetura:**

* Embedding Layer (dimensão = 100)
* Bidirectional LSTM (64 unidades)
* GlobalMaxPooling
* Dense + Dropout
* Camada de saída com softmax

**Parâmetros de treino:**

* Epochs: 5
* Batch size: 128
* Validation split: 0.1

O modelo é treinado para classificar o sentimento das reclamações.

---

## 6. 📈 Avaliação do Modelo

A avaliação inclui:

* Acurácia
* Relatório de classificação (precision, recall, f1-score)
* Curvas de treino (loss e accuracy)
* Matriz de confusão

⚠️ Importante — Limitação Metodológica:

Os rótulos utilizados no treinamento são gerados por regras heurísticas (Seção 3).
Como consequência, a avaliação do modelo mede, em grande parte, **o quanto o modelo aprende a reproduzir essas regras**, e não necessariamente o sentimento real expresso pelos consumidores.

Portanto:

* o requisito técnico do desafio é atendido;
* porém, os resultados devem ser interpretados com cautela do ponto de vista analítico.

Além disso, os resultados podem variar entre execuções, pois:

* os dados são coletados dinamicamente da API;
* a base pode ser desbalanceada.

---

## 7. 🔍 Análise das Dores dos Clientes

Para compreender as principais dores dos clientes, o notebook implementa análises exploratórias a partir das reclamações classificadas como negativas.

As análises utilizam o texto pré-processado (`clean_text`), garantindo consistência com a etapa de modelagem.

As visualizações são realizadas principalmente sobre os produtos com maior volume de reclamações negativas, permitindo uma análise mais clara e interpretável.

### 7.1 Distribuição de Sentimentos por Produto

Análise da quantidade de reclamações positivas e negativas por tipo de produto.

Essa visualização permite identificar quais categorias concentram maior volume de insatisfação.

### 7.2 Termos Mais Frequentes em Reclamações Negativas por Produto

Para os principais produtos, são identificados os termos mais frequentes nas reclamações negativas, a partir da contagem de palavras no texto limpo.

Essa análise evidencia padrões recorrentes de problemas enfrentados pelos consumidores.

### 7.3 Identificação de Temas de Insatisfação

Os principais temas de insatisfação são identificados a partir de agrupamentos de palavras-chave, representando categorias como:

* Cobrança indevida
* Fraude ou transação não autorizada
* Atendimento ao cliente
* Problemas com crédito ou empréstimo
* Reembolso ou estorno
* Erros operacionais

Essa abordagem permite uma leitura mais estruturada dos problemas relatados.

### 7.4 Nuvens de Palavras por Produto

São geradas nuvens de palavras para os principais produtos com maior volume de reclamações negativas.

Isso facilita a visualização dos termos mais recorrentes em cada categoria.

⚠️ Observação:
As análises são exploratórias e baseadas em frequência de termos e regras simples, não substituindo técnicas mais avançadas como topic modeling.

---

## 8. ⚠️ Limitações

* Rotulagem baseada em regras pode gerar inconsistências
* A avaliação reflete a regra de rotulagem, não necessariamente o sentimento real
* Dataset potencialmente desbalanceado
* Dados variam conforme o período coletado
* Análises de texto são baseadas em frequência, não em contexto semântico profundo
* Análise focada nos principais produtos, não em todas as categorias

---

## 9. 🚀 Possíveis Melhorias

* Uso de modelos pré-treinados (BERT, RoBERTa)
* Rotulagem supervisionada com dados anotados manualmente
* Balanceamento de classes
* Aplicação de técnicas de topic modeling (LDA, BERTopic)
* Expansão da análise para todas as categorias de produto
* Deploy do modelo como API

---

## 10. 📌 Conclusão

O projeto demonstra um pipeline completo de NLP aplicado a dados reais de reclamações de consumidores.

Apesar da simplicidade da abordagem de rotulagem, o modelo é capaz de capturar padrões relevantes e fornecer insights úteis sobre a experiência dos clientes.

As análises exploratórias complementam o modelo ao evidenciar as principais dores enfrentadas pelos consumidores por categoria de produto, contribuindo para uma melhor compreensão dos problemas relatados.

Ainda assim, os resultados devem ser interpretados considerando as limitações da rotulagem heurística utilizada.
