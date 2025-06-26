
# 📄 Ficha Técnica Detalhada do Projeto: Análise de Dados Amazon

## 🚀 Visão Geral do Projeto

Este projeto, intitulado **"Projeto 4 - DataLab Amazon"**, teve como objetivo principal **analisar o impacto do desconto na satisfação do cliente**, identificar padrões nas avaliações de produtos e destacar os fatores que influenciam a satisfação, fornecendo insights acionáveis para otimizar o desempenho na plataforma.

**Equipe:** Gabriela, Socorro, Gemini

## 💾 Conjunto de Dados

* **Fontes:**
    * `amazon_review.xlsx`: Contém dados de avaliações de produtos (reviews).
    * `amazon_product.xlsx`: Contém dados de produtos, incluindo preço, categoria e informações de desconto.
* **Tamanho Inicial:** Ambos os datasets possuíam mais de 1.300 observações antes da combinação e filtragem.
* **Localização (no ambiente Colab):** `/content/drive/MyDrive/Amazon-dataset/`

## 🛠️ Ferramentas e Tecnologias

* **Linguagem de Programação:** Python
* **Ambiente de Desenvolvimento:** Jupyter Notebook ou Google Colab
* **Principais Bibliotecas Python:**
    * `pandas`: Manipulação, leitura e organização de DataFrames.
    * `numpy`: Operações numéricas.
    * `matplotlib.pyplot`: Criação de gráficos e visualizações.
    * `seaborn`: Visualização estatística avançada.
    * `statsmodels.formula.api`: Modelagem estatística (Regressão Linear e Logística).
    * `scipy.stats`: Testes estatísticos (Z-score, Qui-Quadrado).
    * `sklearn.metrics`: Métricas de avaliação de modelos (Matriz de Confusão, Relatório de Classificação).
* **Gerenciamento e Documentação:** Notion (para relatórios e fichas técnicas).

## 📊 Etapas da Análise de Dados

### 1. **Carregamento e Junção de Dados**

* **Carregamento Inicial:** Os datasets `amazon_review.xlsx` e `amazon_product.xlsx` foram carregados no Python usando `pandas.read_excel()`. Uma inspeção inicial (`df.dtypes`, `df.info()`) foi realizada sem forçar tipos para entender a estrutura bruta.
* **Junção dos DataFrames:** Após o processamento individual, `Dadosreview` e `Dadosproduto` foram unidos em `Dadoscombinados` utilizando `pd.merge` na coluna `'id_produto'` com um `how='left'` merge para preservar todas as informações de review.

### 2. **Processos Detalhados de Limpeza e Pré-processamento**

Esta etapa foi fundamental para garantir a qualidade, consistência e adequação dos dados para análises posteriores.

* **Tratamento de Valores Nulos:**
    * Identificação e quantificação de nulos por coluna (`.isnull().sum()`).
    * **Imputação:**
        * Colunas numéricas: Preenchidos com a média da coluna (`.fillna(mean_value)`).
        * Colunas categóricas: Preenchidos com a moda da coluna (`.fillna(mode_value[0])`).
* **Identificação e Tratamento de Valores Duplicados:**
    * Verificação de duplicatas na coluna `'id_produto'` em ambos os datasets.
    * Em `Dadosproduto`: Linhas duplicadas baseadas em `'id_produto'` foram removidas, mantendo a primeira ocorrência (`.drop_duplicates(subset=['id_produto'], keep='first')`).
    * Em `Dadosreview`: Duplicatas também foram verificadas e removidas por `'id_produto'`. (**Nota:** Embora realizada, essa remoção pode levar à perda de múltiplos reviews válidos para um mesmo produto, conforme apontado nas limitações.)
    * Duplicatas de linhas inteiras também foram verificadas e removidas em `Dadosproduto`.
* **Tratamento de Dados Fora do Escopo e Discrepantes (Padronização e Conversão de Tipos):**
    * Colunas que deveriam ser numéricas (`classificacao_produto`, `pessoas_que_votaram`, `produto_preco_real`, `produto_desconto`, `porcentagem_desconto`) mas foram carregadas como strings (devido a símbolos como '₹', '%' ou vírgulas decimais) foram convertidas para `float` utilizando `pd.to_numeric` com `errors='coerce'`.
    * Novos NaNs introduzidos pela conversão foram subsequentemente preenchidos com a média.
    * **Padronização Categórica:** Aplicação de `.str.strip()`, `.str.lower().str.capitalize()` em colunas como `nome_produto`, `marca_produto`. Para colunas com pipes (ex: `categoria_produto`, `lista_variacoes`), foi aplicada uma padronização que capitaliza cada palavra separada por pipe.
* **Identificação e Tratamento de Outliers em Variáveis Numéricas:**
    * Outliers identificados em colunas como `produto_desconto`, `produto_preco_real`, `porcentagem_desconto`, `classificacao_produto`, `pessoas_que_votaram` utilizando o método **Z-Score** (limiar de 3).
    * **Tratamento:** Outliers identificados por Z-Score em `pessoas_que_votaram` foram removidos. Além disso, foi aplicado **Winsorizing (capping)** na coluna `pessoas_que_votarem` (utilizando o método IQR com fator de 1.5) para limitar valores extremos sem remoção total.

### 3. **Técnicas de Análise Aplicadas e Modelagem**

#### **3.1. Análise de Risco Relativo**

* **Objetivo:** Comparar o risco (probabilidade) de categorias específicas (e.g., 'Electronics' vs. 'Home&Kitchen') de terem "Alta Classificação" (acima da média geral do dataset).
* **Metodologia:** Cálculo de Odds Ratios/Risco Relativo a partir de tabelas de contingência (com base em contagens de alta/baixa classificação por categoria), seguido por um **Teste Qui-Quadrado** para verificar a significância estatística das diferenças.
* **Insights:** Foi observado um risco relativo, indicando que categorias como 'Electronics' apresentavam um risco diferenciado de alta classificação.

#### **3.2. Regressão Linear**

* **Objetivo:** Modelar a relação linear entre `classificacao_produto` (dependente) e `pessoas_que_votaram`, `produto_preco_real`, `porcentagem_desconto`, e `categoria_principal` (independentes).
* **Resultados:**
    * **R-squared:** Baixo (~0.056), indicando pouca variância explicada pelo modelo.
    * **Significância Global (F-statistic):** Modelo como um todo foi estatisticamente significativo.
    * **Coeficientes Significativos:** `pessoas_que_votaram` (pequeno efeito positivo), `produto_preco_real` (pequeno efeito positivo) e `porcentagem_desconto` (pequeno efeito negativo, contraintuitivo).
    * **Categorias:** As categorias individuais (`C(categoria_principal)`) não mostraram efeito estatisticamente significativo na média da classificação após controle de outras variáveis.
* **Diagnóstico:** Identificação de alta multicolinearidade e não-normalidade dos resíduos.

#### **3.3. Regressão Logística**

* **Objetivo:** Prever a **probabilidade binária** (`is_alta_classificacao` = 1 ou 0) de um produto ter uma alta classificação (`classificacao_produto >= 4.0`), utilizando as mesmas variáveis independentes da regressão linear.
* **Resolução de Problemas:** Filtramos categorias com separação completa (ex: 'Car&Motorbike', 'Toys&Games' com 0 ocorrências em uma das classes) para permitir a convergência do modelo.
* **Resultados:**
    * **Pseudo R-quadrado:** Baixo (~0.062), indicando que as variáveis explicam uma pequena parte da probabilidade.
    * **Significância Global (LLR p-value):** Modelo como um todo foi estatisticamente significativo.
    * **Coeficientes Significativos (Odds Ratios):**
        * `pessoas_que_votaram` e `produto_preco_real`: Aumentam ligeiramente as chances de alta classificação.
        * `porcentagem_desconto`: **Diminui significativamente** as chances de alta classificação (Odds Ratio < 1).
        * `C(categoria_principal)[T.Electronics]` e `C(categoria_principal)[T.Home&Kitchen]`: **Diminuem significativamente** as chances de alta classificação **comparadas a 'Computers&Accessories'** (categoria base).
* **Desempenho do Modelo (Classificação):**
    * **Acurácia:** ~65%.
    * **Recall para 'Alta Classificação' (Classe 1):** ~88% (modelo é bom em identificar esta classe).
    * **Recall para 'Não Alta Classificação' (Classe 0):** ~24% (modelo tem dificuldade em identificar esta classe, gerando muitos Falsos Positivos).
    * **Conclusão de Desempenho:** O modelo tende a classificar muitos produtos como "Alta Classificação", sendo eficaz para identificar a classe majoritária, mas com limitações na identificação da classe minoritária.

### 4. **Visualizações de Dados**

* **Gráficos de Dispersão com Regressão:** Para visualizar a relação entre `classificacao_produto` e variáveis numéricas (e.g., `pessoas_que_votaram`, `produto_preco_real`, `porcentagem_desconto`).
* **Box Plots:** Para analisar a distribuição da `classificacao_produto` por `categoria_principal`, revelando medianas, dispersões e outliers em cada grupo (e.g., 'Electronics' com distribuição mais ampla, 'Home&Kitchen' com mediana alta mas muitos outliers baixos).
* **Gráfico de Proporção de Alta Classificação:** Gráficos de barras mostrando a proporção de produtos com "Alta Classificação" por `categoria_principal`.
* **Gráficos de Resíduos:** Para diagnóstico do modelo (linearidade, homoscedasticidade).

## 💡 Resultados e Insights Finais

* A **qualidade dos dados foi significativamente melhorada** através de um processo de limpeza detalhado, resultando em datasets mais consistentes para análise.
* A **categoria do produto** é um fator estratégico que influencia tanto o potencial de vendas quanto a satisfação do consumidor.
* **Variáveis numéricas** como número de votos, preço e porcentagem de desconto têm uma **relação estatisticamente significativa** com a classificação, embora a magnitude do impacto individual possa ser pequena. O efeito negativo do desconto é um ponto chave para investigação.
* A **complexidade da satisfação do cliente é elevada**, e o modelo atual, embora estatisticamente válido, explica uma parte limitada da variação nas classificações, indicando a presença de **outros fatores não modelados** que são importantes.

## 🗺️ Recomendações Estratégicas

* Investigar fatores específicos (além dos já analisados) que impulsionam as altas classificações, com foco especial em categorias de sucesso como 'Electronics'.
* Analisar subcategorias em 'Home&Kitchen' (e outras) com bom desempenho para identificar e replicar melhores práticas.
* Integrar a **categoria do produto em estratégias de promoção e destaque** na plataforma Amazon.
* Aprofundar a investigação sobre os múltiplos fatores que contribuem para o sucesso de vendas, considerando a interação entre avaliação, descontos, visibilidade e características do produto.
* Manter o foco na qualidade e no valor percebido, reconhecidos como impulsionadores do feedback positivo do consumidor.

## 🚀 Como Executar o Projeto

1.  **Acesso aos Dados:** Garanta que os arquivos `amazon_review.xlsx` e `amazon_product.xlsx` estejam acessíveis no caminho `/content/drive/MyDrive/Amazon-dataset/` do seu Google Drive, caso esteja utilizando o Google Colab.
2.  **Notebook Colab:** O código-fonte completo está disponível no notebook Google Colab: `projeto-4-datalab-amazon.ipynb`.
3.  **Execução:** Abra o notebook no Google Colab e execute as células sequencialmente para replicar todas as etapas de carregamento de dados, pré-processamento, análises (Risco Relativo, Regressão Linear, Regressão Logística) e visualizações.

---
