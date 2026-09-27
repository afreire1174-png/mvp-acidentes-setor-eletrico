# mvp-acidentes-setor-eletrico

MVP de Engenharia de Dados para análise de acidentes e ocorrências de segurança no setor elétrico, utilizando Databricks e tabelas Delta.

## 1. Contexto do Negócio

O setor elétrico envolve atividades que expõem trabalhadores a diferentes riscos ocupacionais, tornando a segurança do trabalho um tema relevante para a gestão das organizações. Nesse contexto, a análise de registros de acidentes, quase-acidentes e outras ocorrências pode contribuir para compreender as circunstâncias dos eventos e produzir informações que apoiem a discussão de medidas preventivas, especialmente aquelas voltadas à preservação da vida. Vale mencionar que essas ocorrências  podem apresentar diferentes níveis de gravidade, desde eventos sem lesão fatal até acidentes com óbito. Conhecer apenas o total de registros não é suficiente para orientar ações preventivas: é preciso entender como as ocorrências se distribuem e quais características aparecem com maior frequência nos casos fatais. Essa análise pode ajudar a identificar situações que merecem investigação e prioridade nas ações de prevenção.

### 1.1. Objetivo

O objetivo deste MVP é descrever a distribuição das ocorrências registradas na base e comparar os acidentes fatais com as demais ocorrências. A análise buscará identificar diferenças observáveis nas variáveis disponíveis, como período, local, atividade e características do evento, conforme os campos efetivamente presentes no banco de dados. Os resultados terão caráter descritivo e exploratório; associações encontradas não serão tratadas como causas dos acidentes.

### 1.2. Perguntas do Negócio

      1.1 Quantas ocorrências estão registradas na base e como elas se distribuem por gravidade?

      1.2 Como as ocorrências evoluem ao longo dos anos e dos meses?

      1.3 Quais agentes causadores apresentam maior número de ocorrências e quais estão mais associados aos eventos de maior grau de risco, classificados como Alto ou Crítico?

      1.4 Quais características e combinações de fatores aparecem com maior frequência nos eventos de maior grau de risco, classificados como Alto ou Crítico, e nos acidentes fatais,     
          considerando segmento, estado/local e agente causador?
          
      1.5 Síntese das respostas às perguntas de negócio

## 2. Coleta de Dados

A coleta dos dados deste MVP foi realizada por meio da obtenção de uma planilha corporativa contendo registros históricos de acidentes e ocorrências de segurança no setor elétrico. Trata-se de uma fonte de dados secundários, pois as informações já haviam sido registradas pela organização e foram utilizadas no projeto para fins de análise.

O arquivo de origem, denominado “BD_Acidentes_Dados_Brutos.xlsx”, foi obtido em formato Excel e contém uma única aba, chamada “Acidentes”, composta por 1.718 registros e 63 colunas, referentes às ocorrências registradas entre 2020 e 2025. 

O conjunto de dados reúne informações sobre as circunstâncias e as consequências das ocorrências, incluindo data e horário, localização, segmento de atuação, tipo de trabalho, agente causador, gravidade, lesões, afastamentos e investigação. Esse contexto permite analisar a distribuição dos eventos ao longo do tempo e identificar padrões associados às atividades e aos níveis de gravidade, contribuindo para responder às perguntas de negócio definidas no projeto.

Por conter dados pessoais e informações relacionadas a lesões e afastamentos, o arquivo original não foi incluído nos materiais disponibilizados publicamente com o MVP. A documentação da coleta apresenta a origem, o formato, a abrangência temporal e a estrutura geral da base, preservando a confidencialidade dos registros. Os procedimentos de proteção, limpeza e padronização dos dados são descritos nas respectivas etapas de tratamento.

A tabela a seguir apresenta os campos existentes no arquivo de origem, agrupados por assunto para facilitar a compreensão do conteúdo coletado. Esses grupos descrevem a estrutura da fonte e não representam as tabelas do modelo analítico desenvolvido no projeto.

Grupo	                          Campos presentes na base
Identificação e vínculo      	  Código, Empregado, ID, Nome, Empresa, Fornecedor, Nome Subcontratada, Nº do Contrato, Chave
Caracterização da ocorrência	  Classificação, Ocorrência com veículo, Nexo causal, Descrição, Local, Tipo de Trabalho
Perfil e atividade            	Função, Sexo, Idade, Tempo de Empresa, Segmento, Segmento Expansão
Data e horário	                Dia da Semana, Dia, Mês, Ano, Hora, Data completa
Localização	                    Município, Estado, Instalação, Latitude, Longitude
Estrutura organizacional	      Processos e gerências internas das empresas
Afastamento e consequências	    Data Início do Afastamento, Data Fim do Afastamento, Dias Perdidos, Dias Debitados, Dias Perdidos Corrigidos, Reabilitado, Data da CAT, Nº da CAT, Agente Causador, Tipo de Lesão, Parte do Corpo Atingida, Gravidade
Risco e investigação	          Potencial, Grau de Risco, Compromissos (Regra de Ouro), Causa raiz 1 a Causa raiz 8, Plano de ação, Cartão Alerta, Relatório de Investigação

## 3. Modelagem de Dados

A modelagem foi organizada em duas subseções: Estrutura e definição do modelo de dados e Catálogo e dicionário de dados. A primeira
apresenta o modelo flat, sua granularidade, o identificador técnico e os tipos propostos. A segunda documenta o significado dos campos
e suas regras de interpretação.

A estrutura considera a versão preparada para análise, com
substituição de `Chave` por `id_registro`. Essa alteração não
caracteriza, isoladamente, anonimização da base.

### 3.1 Estrutura e definição do modelo de dados

Neste MVP, foi adotado o modelo de tabela única desnormalizada (flat table), tendo como referência a versão
`BD_Acidentes_com_id_sem_chave_v2.xlsx`, resultante da criação do identificador técnico `id_registro` e da exclusão da coluna `Chave`.

Essa versão contém 1.718 registros e 23 colunas, referentes ao
período de 2020 a 2025. Os atributos de caracterização, localização,
tempo, vínculo e consequências das ocorrências estão reunidos
na mesma estrutura, permitindo consultas e agregações sem
necessidade de junções entre tabelas.

A implementação da tabela analítica final na camada Gold permanece
prevista para as etapas seguintes do pipeline.

A granularidade corresponde a uma linha por registro de segurança. O campo `id_registro`, gerado como UUID e persistido na versão
preparada para análise, apresentou 1.718 valores distintos e nenhum valor ausente. A coluna `Chave` foi excluída dessa versão, permanecendo
na entrada preservada na Bronze.

A unicidade do identificador não comprova que cada linha represente um acidente distinto. Portanto, os totais serão interpretados como
quantidades de registros.

A escolha pelo modelo flat considera o volume de dados e os objetivos analíticos do projeto. A estrutura permite avaliar a distribuição dos registros por período, classificação, gravidade, segmento, estado, categoria de local e agente causador, além de comparar as características dos registros fatais com as demais ocorrências. Nesse modelo, não há separação em tabelas fato e dimensão nem relacionamentos por chaves estrangeiras.

Para implementação no Databricks, propõe-se a tabela acidentes analíticos, com tipos de dados definidos conforme o significado dos campos. Ano, Mês e Potencial devem ser representados como números inteiros. Os demais campos devem ser inicialmente armazenados como texto, incluindo Hora, que contém faixas horárias, e Tempo de Empresa, que apresenta intervalos e descrições de duração. A tabela analítica final está prevista como `workspace.mvp_gold.acidentes_analiticos`. Os tipos apresentados no dicionário são propostos para a estrutura analítica. Na Bronze,
todos os campos foram armazenados como STRING; as conversões de Ano, Mês e Potencial para inteiros serão avaliadas na Silver.

A análise temporal deve respeitar o detalhamento disponível na fonte. Como a planilha não contém a data completa das ocorrências, as séries temporais serão organizadas por ano e mês. Os campos Dia da Semana e Hora permitem análises complementares de distribuição, mas não possibilitam reconstruir a data exata dos eventos.

A identificação dos registros fatais será baseada no campo Classificação, considerando as categorias Fatalidade e Fatalidade Trajeto. O campo Gravidade, por representar níveis de gravidade que não correspondem diretamente à fatalidade, não será utilizado isoladamente para essa identificação. Propõe-se a criação de um indicador derivado que diferencie registros fatais, demais classificações reconhecidas e situações com classificação ausente ou não reconhecida.

### 3.2 Catálogo e dicionário de dados

O catálogo de dados documenta a origem, a finalidade e a organização do conjunto utilizado no projeto. O dicionário complementa essa documentação ao apresentar a definição dos atributos, os tipos de dados propostos e as regras de interpretação, contribuindo para a utilização consistente das informações e para a reprodutibilidade das análises.

A versão analisada contém 1.718 registros e 23 colunas. A tabela a seguir apresenta os campos, suas descrições, os tipos propostos para implementação no Databricks e as regras de interpretação. A documentação considera a substituição da coluna Chave pelo identificador técnico id_registro e explicita os atributos que ainda dependem de validação com a fonte. 

Campo	                                      Descrição	                                  Tipo proposto	                            Domínio ou regra
id_registro	                        Identificador técnico de cada registro.	              STRING	                        UUID único, obrigatório e persistente; não identifica necessariamente um acidente distinto.
Empregado	                          Categoria de vínculo do envolvido.	                  STRING	                        Próprio ou Terceiro.
Classificação	                      Categoria do registro de segurança.	                  STRING	                        Acidentes típicos e de trajeto, com ou sem afastamento; Fatalidade; Fatalidade Trajeto; Quase Acidente; Desvio Crítico; Doença Ocupacional.
Empresa	                            Empresa associada ao registro.	                      STRING	                        Relação organizacional a confirmar com a fonte.
Segmento	                          Segmento ou área de atuação.	                        STRING	                        Categorias da fonte, sujeitas à padronização.
Sexo	                              Sexo informado para o envolvido.	                    STRING	                        Masculino, Feminino e marcador NA, cujo significado requer validação.
Tempo de Empresa	                  Duração do vínculo com a empresa.	                    STRING	                        Faixas e descrições de tempo; não corresponde a uma medida numérica padronizada.
Dia da Semana	                      Dia da semana da ocorrência.	                        STRING	                        Segunda-feira a domingo, após padronização.
Mês	                                Mês de referência do registro.	                      INT	                            Valores de 1 a 12.
Ano	                                Ano de referência do registro.	                      INT	                            Período observado: 2020 a 2025.
Hora	                              Faixa horária da ocorrência.	                        STRING	                        Intervalos horários, como 07:00 - 08:00.
Descrição	                          Relato das circunstâncias da ocorrência.	            STRING	                        Texto livre; requer avaliação de informações identificáveis antes da divulgação.
Local	                              Categoria do local da ocorrência.	                    STRING	                        Categorias como Empresa, Via pública e Área rural; códigos específicos requerem validação.
Organização do trabalho            	Forma de organização da atividade.	                  STRING	                        Individual, Equipe, Dupla e Empresa; esta última requer esclarecimento.
Estado	                            Unidade federativa associada à ocorrência.	          STRING	                        Valores sujeitos à validação e padronização.
Diretoria	                          Diretoria associada ao registro.	                    STRING	                        Categorias organizacionais definidas pela fonte.
Agente Causador	                    Agente apontado como causador da ocorrência.	        STRING	                        Não equivale necessariamente à causa raiz investigada.
Tipo de Lesão	                      Natureza da lesão registrada.	                        STRING                        	Distinguir informação ausente de não aplicabilidade.
Parte do Corpo Atingida	            Região corporal afetada.	                            STRING                        	Categorias anatômicas; distinguir informação ausente de não aplicabilidade.
Gravidade	                          Nível de gravidade atribuído ao registro.            	STRING	                        Baixo, Médio e Alto; valores - e 16 requerem validação. Não identifica isoladamente fatalidades.
Potencial	                          Valor numérico de potencial registrado.	              INT                            	Valores observados entre 1 e 25; significado da escala a confirmar.
Grau de Risco	                      Categoria de risco do registro.	                      STRING	                        Baixo, Médio, Alto e Crítico, após padronização.
Compromissos (Regra de Ouro)	      Regra de segurança associada ao registro.            	STRING	                        Código e descrição da regra; inclui “Não se Aplica” e marcador D, a validar.

A substituição da coluna Chave na versão preparada para análise por id_registro proporcionou preenchimento completo e unicidade, permitindo distinguir cada linha da tabela analítica. Após a gravação, os identificadores serão reutilizados nas etapas seguintes, garantindo sua persistência. Essa transformação não comprova a ausência de registros duplicados em conteúdo nem conclui a anonimização dos demais atributos.


## 4. Carga e Pipeline

### 4.1 Arquitetura do pipeline

O pipeline foi planejado segundo a arquitetura medalhão, que organiza os dados em três camadas com diferentes níveis de tratamento e
preparação para análise: Bronze, Silver e Gold.

| Camada | Finalidade no projeto |
|---|---|
| Bronze | Preservar os dados recebidos, com alterações técnicas mínimas, permitindo rastreabilidade e reprocessamento. |
| Silver | Aplicar validações, padronização, tratamento de inconsistências e medidas de proteção dos dados pessoais. |
| Gold | Disponibilizar a tabela analítica no modelo flat e os indicadores necessários às perguntas de negócio. |

As camadas representam etapas lógicas dentro do mesmo ambiente Databricks, sem necessidade de plataformas separadas para armazenamento
e análise.

Foram criados os schemas `mvp_bronze`, `mvp_silver` e `mvp_gold` no catálogo `workspace`, estabelecendo a organização das camadas
da arquitetura medalhão.

Também foram realizados o upload da planilha para o Volume `arquivos_mvp`, a criação e persistência do identificador técnico,
a exclusão da coluna `Chave` da versão analítica e o perfilamento inicial.

Foram criados os schemas `mvp_bronze`, `mvp_silver` e `mvp_gold` no catálogo `workspace`. A tabela `workspace.mvp_bronze.acidentes`
foi carregada e verificada, apresentando 1.718 registros, 23 colunas e armazenamento em formato Delta.

A implementação das tabelas Silver e Gold e das transformações
entre as camadas permanece pendente.


### 4.2 Ingestão dos dados no Databricks

A base original, descrita na seção Coleta de Dados, possui 1.718 registros e 63 colunas. Para esta etapa, foi utilizada a versão
`BD_Acidentes_tratada_v3.xlsx`, composta por 1.718 registros e 23 colunas, resultante da preparação anterior ao upload.

A relação dos campos removidos ou modificados e os critérios utilizados nessa preparação ainda precisam ser documentados. As transformações
apresentadas nesta seção têm como ponto de partida a versão de 23 colunas.

A carga inicial foi realizada por meio do upload desse arquivo para o Volume `arquivos_mvp`, pertencente ao schema `default` do catálogo
`workspace`, no Databricks. O notebook de processamento foi salvo na pasta de usuário do Workspace.

**Caminho do arquivo de entrada:**

```text
/Volumes/workspace/default/arquivos_mvp/BD_Acidentes_tratada_v3.xlsx
```

A leitura da aba `Acidentes` foi realizada em Python, utilizando
as bibliotecas pandas e openpyxl. A conferência confirmou
1.718 registros e 23 colunas.

**Resultado obtido:**

```text
Quantidade de registros: 1718
Quantidade de colunas: 23
Colunas: ['Empregado', 'Classificação', 'Empresa', 'Segmento', 'Sexo', 'Tempo de Empresa', 'Dia da Semana', 'Mês', 'Ano', 'Hora', 'Descrição', 'Local', 'Organização do trabalho', 'Estado', 'Diretoria', 'Agente Causador', 'Tipo de Lesão', 'Parte do Corpo Atingida', 'Gravidade', 'Potencial', 'Grau de Risco', 'Compromissos (Regra de Ouro)', 'Chave']
```

### 4.3 Carga da camada Bronze

A partir do arquivo apresentado no item 4.2, foi preparada a rotina
de carga da tabela Delta `workspace.mvp_bronze.acidentes`.

A preparação técnica normaliza os nomes das colunas e define todos
os campos como STRING, preservando o conteúdo recebido para os
tratamentos posteriores. A rotina verifica a existência da tabela
antes da gravação, evitando sua sobrescrita.

A validação da carga compara a quantidade de registros e a estrutura
das colunas com os dados de entrada e verifica o formato Delta.

A carga foi concluída e a consulta à tabela confirmou os seguintes
resultados:

| Verificação | Resultado obtido |
|---|---|
| Tabela | `workspace.mvp_bronze.acidentes` |
| Registros | 1.718 |
| Colunas | 23 |
| Formato | Delta |

#### Amostra dos dados carregados na Bronze

Após a persistência da tabela, foi realizada a visualização de uma
amostra de 10 registros da camada Bronze, com o objetivo de verificar
visualmente se os dados foram carregados corretamente e se a estrutura
das colunas foi preservada após a ingestão.

A consulta utilizada foi:

```python
display(
    spark.table("workspace.mvp_bronze.acidentes")
         .limit(10)
)

### 4.4 Criação do identificador técnico e exclusão da chave original

Antes da implementação da tabela Bronze, foi preparada uma cópia Excel com o identificador `id_registro` e sem a coluna `Chave`.
Essa versão foi utilizada no perfilamento inicial. Os identificadores persistidos deverão ser preservados na implementação da Silver,
cuja vinculação aos registros da Bronze ainda precisa ser definida e validada.

Foi criado o campo `id_registro`, composto por um UUID aleatório para cada linha. Esse identificador é independente dos atributos da fonte
e permite identificar os registros sem incorporar informações pessoais em sua composição.

A coluna original `Chave` foi excluída da versão analítica. O arquivo de origem foi preservado, e a transformação não alterou a quantidade
de registros.

Os identificadores foram gerados uma única vez e persistidos na nova versão. Nas execuções posteriores, a versão salva deve ser carregada
para evitar a atribuição de novos códigos às mesmas linhas.

### 4.5 Persistência e validação da versão resultante

Durante a execução, a gravação direta do Excel no Volume apresentou erro de entrada e saída. A persistência foi realizada pela criação
do arquivo em armazenamento temporário local, seguida de sua cópia para o Volume.

Após a gravação, o arquivo foi relido para verificar a quantidade de linhas e colunas, a preservação dos identificadores e a ausência da
coluna `Chave`.

**Resultado obtido:**
```text
Arquivo existente carregado. IDs preservados.
Registros: 1718
Colunas: 23
IDs únicos: 1718
Coluna Chave presente: False
```

O procedimento reutiliza o arquivo de destino quando ele já existe.
Essa lógica preserva os IDs, mas não implementa a incorporação automática de novos registros ou de alterações posteriores na fonte.

### 4.6 Resultados da transformação

As verificações realizadas no Databricks, por meio do código
apresentado na seção 4.4, produziram os seguintes resultados:

**Resultado obtido:**

| Verificação | Resultado |
|---|---:|
| Quantidade de registros | 1.718 |
| Quantidade de colunas | 23 |
| Identificadores distintos em `id_registro` | 1.718 |
| Valores ausentes em `id_registro` | 0 |
| Presença da coluna `Chave` | Não |

A substituição de `Chave` por `id_registro` preservou a quantidade
de registros e de colunas da base. Os identificadores gerados
são únicos e não apresentam valores ausentes.

A unicidade de `id_registro` permite distinguir as linhas, mas
não comprova a ausência de duplicidades de conteúdo nem assegura
que cada registro corresponda a um acidente distinto.

A versão resultante foi armazenada no arquivo
`/Volumes/workspace/default/arquivos_mvp/BD_Acidentes_com_id_sem_chave_v2.xlsx`.

Nas execuções seguintes, o procedimento reutiliza esse arquivo e preserva os identificadores já atribuídos.

### 4.7 Encadeamento das etapas e escopo implementado

O fluxo executado compreendeu:

1. Upload da planilha BD_Acidentes_tratada_v3.xlsx para o Volume.
2. Leitura e conferência da aba Acidentes.
3. Padronização técnica dos nomes das colunas.
4. Carga e validação da tabela Bronze em formato Delta.
5. Criação do identificador técnico id_registro na versão preparada para análise.
6. Exclusão da coluna Chave dessa versão.
7. Persistência e conferência de BD_Acidentes_com_id_sem_chave_v2.xlsx.
8. Releitura da versão salva.
9. Perfilamento inicial da qualidade dos dados.

O diagnóstico de valores ausentes, tipos de dados e valores distintos é apresentado na seção Qualidade de Dados.

Até esta etapa, o fluxo utiliza arquivos Excel armazenados em um Volume.
A camada Bronze já foi implementada como tabela Delta e validada quanto à quantidade de registros, estrutura e formato de armazenamento. As camadas Silver e Gold, bem como as transformações entre essas etapas, permanecem pendentes de implementação e documentação. Essas etapas deverão ser documentadas conforme forem executadas.

A substituição da chave original não conclui a anonimização da base.
Descrições livres e combinações de atributos ainda requerem avaliação antes de qualquer divulgação. Os arquivos detalhados não integram
os materiais públicos do projeto.

## 5. Qualidade de Dados

A avaliação inicial da qualidade foi realizada no Databricks, utilizando Python e pandas, sobre o arquivo
`BD_Acidentes_com_id_sem_chave_v2.xlsx`, com 1.718 registros e 23 colunas. Foram analisados o preenchimento dos campos,
os tipos reconhecidos na leitura e a quantidade de valores distintos.

Os resultados apresentados correspondem a essa versão Excel. Não representam, ainda, o perfilamento direto da tabela Bronze
nem a validação de uma camada Silver implementada.

### 5.1 Completude dos dados

A completude foi avaliada pela quantidade e pelo percentual de valores ausentes em cada coluna. Células vazias e textos compostos apenas por espaços foram considerados ausentes no perfilamento.

A leitura utilizou o reconhecimento padrão de valores ausentes do pandas. Adicionalmente, textos compostos apenas por espaços foram considerados
ausentes na cópia de avaliação. Os percentuais refletem esse procedimento, e não necessariamente apenas células fisicamente vazias no Excel.

As seguintes colunas apresentaram valores ausentes:

| Campo | Quantidade de valores ausentes | Percentual de valores ausentes |
|---|---:|---:|
| Parte do Corpo Atingida | 637 | 37,08% |
| Gravidade | 522 | 30,38% |
| Tipo de Lesão | 351 | 20,43% |
| Agente Causador | 344 | 20,02% |
| Tempo de Empresa | 169 | 9,84% |
| Sexo | 98 | 5,70% |
| Local | 45 | 2,62% |
| Organização do trabalho | 29 | 1,69% |
| Hora | 8 | 0,47% |
| Dia da Semana | 1 | 0,06% |
| Empregado | 1 | 0,06% |

Os maiores percentuais de ausência foram identificados em Parte do Corpo Atingida e Gravidade. Essas limitações devem ser consideradas nas análises
que utilizam tais campos, explicitando a cobertura dos dados disponíveis.

A ausência de informação não representa necessariamente erro. Em quase acidentes e desvios críticos, por exemplo, campos relacionados a lesões
podem não ser aplicáveis. A distinção entre “não informado” e “não se aplica” depende de regras validadas com a fonte.

As quantidades da tabela não devem ser somadas para determinar o total de registros incompletos, pois uma mesma linha pode apresentar ausência
em vários campos.

### 5.2 Consistência

A avaliação inicial identificou diferenças de preenchimento que podem fragmentar categorias equivalentes e afetar os agrupamentos analíticos.

O campo Dia da Semana apresentou 18 valores distintos, embora seu domínio esperado corresponda aos sete dias da semana. Foram observadas variações
de espaços, grafia e acentuação. Hora apresentou 30 valores distintos, incluindo diferenças de formatação das faixas horárias.

Também foram identificadas variações de espaços em Segmento e Grau de Risco. No campo Gravidade, além de Baixo, Médio e Alto, foram observados
os valores “-” e “16”, que exigem validação antes de qualquer correção.

O campo Tempo de Empresa combina faixas de duração com descrições pontuais, demandando critérios para eventual uniformização. Marcadores
como “D” e “NA” também precisam ter seus significados esclarecidos.

Esses resultados constituem um diagnóstico. A padronização das categorias e a validação das relações entre campos ainda não foram concluídas.

### 5.3 Unicidade

O campo `id_registro` apresentou 1.718 valores distintos e nenhum valor ausente, confirmando a unicidade dos identificadores das linhas na
versão analisada.

Esse resultado decorre da atribuição de um UUID a cada registro e não comprova a ausência de duplicidades de conteúdo. Registros diferentes
podem apresentar informações semelhantes ou estar associados ao mesmo evento.

A identificação de acidentes distintos depende de uma chave de evento ou de critérios validados com a fonte. Portanto, as contagens da base
representam registros de segurança, e não necessariamente acidentes distintos ou pessoas envolvidas.

### 5.4 Acurácia

A acurácia corresponde à correspondência entre os dados registrados e os fatos que representam. O perfilamento realizado permite identificar
problemas de preenchimento e valores potencialmente inconsistentes, mas não comprova a exatidão factual das informações.

Não foi realizada conferência sistemática com documentos de origem, relatórios de investigação ou responsáveis pelos registros. Dessa forma, a acurácia factual dos registros não foi confirmada por validação independente.

Para os indicadores de fatalidade, foi estabelecido que a identificação deve utilizar as categorias Fatalidade e Fatalidade Trajeto do campo
Classificação. O campo Gravidade não deve ser utilizado isoladamente para essa finalidade, pois a categoria Alto também ocorre em registros
não classificados como fatais.

Os resultados analíticos deverão ser interpretados conforme as classificações registradas, sem pressupor validação independente dos fatos.

### 5.5 Outliers

Não foi realizada, nesta etapa, uma análise estatística específica de valores extremos. A base é predominantemente categórica, e valores ou
categorias pouco frequentes não devem ser classificados automaticamente como erros.

O campo Potencial apresentou 24 valores distintos, entre 1 e 25.
A avaliação de valores atípicos nesse campo depende do conhecimento da escala utilizada e de seus limites admissíveis. Caso represente uma
escala ordinal ou um código, métodos estatísticos destinados a medidas contínuas podem não ser apropriados.

O valor “16” no campo Gravidade constitui uma inconsistência de domínio a investigar, e não um outlier estatístico confirmado.

Nenhum registro foi excluído por apresentar valor extremo ou categoria rara. As fatalidades, embora pouco frequentes, são relevantes para as
perguntas de negócio e devem ser preservadas na análise.

### 5.6 Tratamentos realizados

Até esta etapa, foram realizadas as seguintes operações:

| Operação | Finalidade | Resultado |
|---|---:|---:|
| Criação de `id_registro` | Identificar cada linha por um código independente dos atributos originais. | 1.718 UUIDs distintos e nenhum valor ausente. |
| Exclusão de `Chave` da versão analítica | Retirar o identificador original dessa versão. | A base permaneceu com 23 colunas após a substituição. |
| Gravação e releitura da nova planilha | Persistir os identificadores e conferir sua preservação. | Versão salva e conferida, com 1.718 registros. |
| Reconhecimento de textos compostos apenas por espaços como ausentes | Evitar subestimação de valores ausentes no perfilamento. | Regra aplicada somente à cópia em memória utilizada na avaliação, sem alteração do arquivo de origem.|

A versão resultante foi salva como `BD_Acidentes_com_id_sem_chave_v2.xlsx`. Nas etapas seguintes, esse arquivo será utilizado para preservar os identificadores já atribuídos.

Não foram executadas, no fluxo documentado até aqui, a imputação de valores ausentes, a exclusão de duplicidades de conteúdo, a padronização
completa das categorias ou a remoção de outliers. Essas ações dependerão de regras justificadas e deverão ser acompanhadas de nova avaliação
da qualidade.

A etapa atual corresponde ao diagnóstico inicial da versão Excel. A tabela Bronze já foi carregada em formato Delta, mas seu perfilamento
direto e a validação dos tratamentos da Silver ainda serão realizados.

A conclusão da avaliação de qualidade compreenderá a definição das regras para os campos utilizados nas análises, a investigação de
possíveis duplicidades de conteúdo e a comparação dos indicadores antes e depois dos tratamentos. As alterações e as decisões de manter
valores sem correção deverão ser justificadas e documentadas.


## 6. Análise das Perguntas de Negócio

Nesta etapa, os dados consolidados na camada Gold são utilizados para responder às perguntas de negócio definidas para o MVP. As análises buscam identificar a distribuição das ocorrências, sua evolução ao longo do tempo e os principais fatores associados aos eventos de maior grau de risco.

Os resultados são apresentados por meio de tabelas, gráficos e análises descritivas, permitindo relacionar os dados tratados nas etapas anteriores aos objetivos do projeto.

### 6.1 Quantas ocorrências estão registradas na base e como elas se distribuem por grau de risco?

A base analisada possui **1.718 ocorrências**.

| Grau de risco | Quantidade de ocorrências | Percentual |
|---|---:|---:|
| Médio | 797 | 46,39% |
| Baixo | 576 | 33,53% |
| Alto | 303 | 17,64% |
| Crítico | 42 | 2,44% |
| **Total** | **1.718** | **100,00%** |

Os eventos classificados como **Alto ou Crítico** totalizam **345 ocorrências**, correspondendo a **20,08%** da base analisada.

Embora predominem os eventos de grau Médio e Baixo, a participação dos eventos Alto e Crítico justifica o aprofundamento das análises seguintes, especialmente na investigação dos agentes causadores e das características associadas às ocorrências de maior grau de risco.

#### Distribuição das ocorrências por grau de risco

![Distribuição das ocorrências por grau de risco](imagens/6_1_distribuicao_grau_risco.png)

#### Análise

Foram registradas **1.718 ocorrências** na base analisada. A maior concentração está no grau de risco **Médio**, com 797 registros (46,39%), seguida pelo grau **Baixo**, com 576 ocorrências (33,53%).

Os eventos classificados como **Alto** representam 303 ocorrências (17,64%), enquanto os classificados como **Crítico** totalizam 42 registros (2,44%). Somados, os graus **Alto e Crítico correspondem a 345 ocorrências**, aproximadamente **20,08% do total da base**.

Os resultados mostram predominância dos graus Médio e Baixo, que juntos representam **79,92%** das ocorrências. Entretanto, a participação dos eventos Alto e Crítico reforça a importância de aprofundar as análises sobre os fatores associados às ocorrências de maior grau de risco.


### 6.2  Como as ocorrências evoluem ao longo dos anos e dos meses?

A avaliação temporal foi dividida em duas perspectivas:

- **6.2.1 Evolução anual**, permitindo observar o comportamento das ocorrências entre os diferentes anos;
- **6.2.2 Evolução mensal**, permitindo uma análise mais detalhada da variação das ocorrências ao longo dos meses.

### 6.2.1 Evolução anual das ocorrências

A evolução anual das ocorrências foi analisada a partir do agrupamento dos registros pelo campo `ano`, utilizando os dados disponíveis na camada Gold.

Foram considerados os **1.718 registros** da base. A distribuição anual encontrada foi:

| Ano | Quantidade de ocorrências |
|---|---:|
| 2020 | 48 |
| 2021 | 91 |
| 2022 | 164 |
| 2023 | 338 |
| 2024 | 485 |
| 2025 | 592 |

Os resultados mostram um crescimento contínuo da quantidade de ocorrências registradas ao longo do período analisado. O menor número de registros foi observado em **2020**, com **48 ocorrências**, enquanto **2025** apresentou o maior valor da série, com **592 ocorrências**.

Em termos de participação no total da base, **2025 representa aproximadamente 34,5% dos registros**, seguido por **2024, com cerca de 28,2%**, e **2023, com aproximadamente 19,7%**. Em conjunto, os anos de **2023, 2024 e 2025 concentram cerca de 82,4% das 1.718 ocorrências**.

Também se observa que o crescimento mais acentuado entre anos consecutivos ocorreu entre **2022 e 2023**, quando a quantidade de registros passou de **164 para 338 ocorrências**.

Entretanto, esse comportamento deve ser interpretado como uma característica da base de dados analisada. O aumento da quantidade de registros não permite concluir, isoladamente, que houve crescimento real do número de acidentes, uma vez que fatores como ampliação da cobertura dos registros, mudanças nos processos de notificação ou maior disponibilidade de dados nos anos mais recentes também podem influenciar essa distribuição.

O gráfico a seguir apresenta visualmente a evolução anual das ocorrências:

![Evolução anual das ocorrências](imagens/6_2_1_evolucao_anual.png)

A análise anual fornece uma visão temporal geral do conjunto de dados e serve como referência para as análises seguintes, nas quais serão avaliados aspectos como gravidade, agentes causadores, segmentos, localização e outros fatores relacionados às ocorrências.

#### Análise

A evolução anual mostra crescimento contínuo dos registros de ocorrências ao longo do período analisado, passando de 48 registros em 2020 para 592 em 2025, maior valor da série. Os anos de 2023, 2024 e 2025 concentram aproximadamente 82,4% dos 1.718 registros, indicando maior concentração nos anos mais recentes.

O maior crescimento entre anos consecutivos ocorreu de 2022 para 2023, quando os registros passaram de 164 para 338 ocorrências.

Esse comportamento deve ser interpretado como uma característica da base analisada, pois o aumento dos registros não significa necessariamente aumento real dos acidentes, podendo também refletir diferenças na cobertura, no processo de notificação ou na disponibilidade de dados ao longo do período.

### 6.2.2 Evolução mensal das ocorrências

A análise mensal evidencia oscilações na quantidade de registros ao longo do período estudado, com maior concentração nos anos mais recentes.

O maior número mensal de registros foi observado em **outubro de 2025**, com **68 ocorrências**, enquanto o menor ocorreu em **novembro de 2020**, com **1 ocorrência**.

O gráfico mostra que, apesar das variações entre os meses, os maiores volumes estão concentrados principalmente nos períodos mais recentes, comportamento coerente com a tendência identificada na evolução anual.

![Evolução mensal das ocorrências](imagens/6_2_2_evolucao_mensal.png)

#### Análise dos resultados

A análise mensal evidencia oscilações na quantidade de ocorrências registradas ao longo do período estudado, com maior concentração de registros nos anos mais recentes.

O gráfico mostra que, apesar das variações entre os meses, os volumes mensais se tornam mais elevados principalmente a partir de 2023, comportamento coerente com a tendência identificada na análise anual.

Esses resultados devem ser interpretados como características da base analisada. As diferenças observadas entre os meses não permitem concluir, isoladamente, que houve aumento ou redução real dos acidentes, pois fatores como cobertura dos registros, processo de notificação e disponibilidade histórica dos dados também podem influenciar essa distribuição.

### Síntese da análise temporal

As análises anual e mensal mostram aumento da quantidade de registros ao longo do período analisado, com maior concentração nos anos mais recentes.

Na visão anual, os registros passaram de **48 ocorrências em 2020** para **592 em 2025**, sendo que **2023, 2024 e 2025 concentram aproximadamente 82,4% dos 1.718 registros** da base. Na análise mensal, o maior volume foi observado em **outubro de 2025**, com **68 ocorrências**, enquanto o menor ocorreu em **novembro de 2020**, com **1 ocorrência**.

Em conjunto, os resultados indicam uma concentração crescente de registros ao longo do tempo. Entretanto, esse comportamento deve ser interpretado como uma característica da base analisada, não sendo suficiente, isoladamente, para concluir que houve aumento real dos acidentes, pois fatores relacionados à cobertura, notificação e disponibilidade dos dados também podem influenciar essa distribuição.


### 6.3  Quais agentes causadores apresentam maior número de eventos classificados com grau de risco Alto ou Crítico?

Para responder a esta pergunta, a análise foi dividida em três etapas complementares. Primeiro, é apresentada a distribuição dos eventos classificados como Alto ou Crítico por agente causador. Em seguida, é avaliada a participação dos principais agentes no conjunto desses eventos. Por fim, os resultados são interpretados de forma consolidada, destacando os agentes que mais se repetem entre as ocorrências de maior grau de risco.

- **6.3.1 Distribuição dos eventos Alto/Crítico por agente causador**
- **6.3.2 Participação dos principais agentes causadores nos eventos Alto/Crítico**
- **6.3.3 Análise dos resultados**

### 6.3.1 Distribuição dos eventos Alto/Crítico por agente causador

Para identificar quais agentes causadores aparecem com maior frequência entre os eventos classificados com grau de risco **Alto ou Crítico**, foram considerados apenas os registros pertencentes a essas duas categorias de risco.

Os agentes foram agrupados pela quantidade de eventos e ordenados do maior para o menor, considerando os dez com maior frequência.

| Agente causador | Quantidade de eventos |
|---|---:|
| D | 43 |
| Eletricidade | 40 |
| Carro - Colisão | 29 |
| Moto - Queda | 26 |
| Golpeado por (atingido por objeto em movimento) | 23 |
| Carro - Capotamento | 22 |
| Queda com diferença de nível | 16 |
| Moto - Colisão | 13 |
| Explosão | 9 |
| Atingido entre ou abaixo (esmagado ou amputado) | 7 |

![Distribuição dos eventos Alto/Crítico por agente causador](imagens/6_3_1_agentes_alto_critico.png)

#### Análise dos resultados

Os resultados mostram que o código **“D”** apresentou a maior quantidade de eventos Alto/Crítico, com **43 ocorrências**. Conforme a nota explicativa da base de dados utilizada neste trabalho, entretanto, o código **“D”** corresponde a registros em que o agente causador não foi classificado durante o preenchimento, não representando, portanto, um agente causador específico.

Entre os agentes efetivamente identificados, **Eletricidade** apresentou a maior frequência, com **40 ocorrências**, seguida por **Carro - Colisão**, com **29**, e **Moto - Queda**, com **26 eventos**.

Também se destacaram **Golpeado por (atingido por objeto em movimento)**, com **23 ocorrências**, e **Carro - Capotamento**, com **22**.

Os resultados indicam que determinados agentes aparecem com maior recorrência entre os eventos classificados com maior grau de risco. No entanto, a análise representa uma associação por frequência e não permite, isoladamente, estabelecer relação direta de causa e efeito entre o agente causador e o grau de risco.

### 6.3.2 Participação dos principais agentes causadores nos eventos Alto/Crítico

Para complementar a análise anterior, foi calculada a participação percentual dos agentes causadores nos eventos classificados com grau de risco **Alto ou Crítico**.

Conforme a nota explicativa da base de dados, o código **“D”** representa registros cujo agente causador não foi classificado durante o preenchimento. Por esse motivo, esse código foi excluído desta análise, permitindo avaliar apenas os eventos com agente causador efetivamente identificado.

Foram considerados **241 eventos Alto/Crítico com agente causador classificado**.

| Agente causador | Quantidade de eventos | Participação (%) |
|---|---:|---:|
| Eletricidade | 40 | 16,60 |
| Carro - Colisão | 29 | 12,03 |
| Moto - Queda | 26 | 10,79 |
| Golpeado por (atingido por objeto em movimento) | 23 | 9,54 |
| Carro - Capotamento | 22 | 9,13 |
| Queda com diferença de nível | 16 | 6,64 |
| Moto - Colisão | 13 | 5,39 |
| Explosão | 9 | 3,73 |
| Atingido entre ou abaixo (esmagado ou amputado) | 7 | 2,90 |
| Animal | 6 | 2,49 |

![Participação dos principais agentes causadores nos eventos Alto/Crítico](imagens/6_3_2_participacao_agentes_alto_critico.png)

#### Análise dos resultados

Entre os agentes causadores efetivamente classificados, **Eletricidade** apresentou a maior participação nos eventos Alto/Crítico, com **40 ocorrências**, correspondendo a **16,60%** do total. Em seguida aparecem **Carro - Colisão**, com **12,03%**, e **Moto - Queda**, com **10,79%**.

Os cinco principais agentes causadores concentram aproximadamente **58,09%** dos eventos analisados. Considerando os dez agentes apresentados na tabela e no gráfico, essa participação alcança aproximadamente **79,24%** do total.

Os resultados indicam uma concentração relevante dos eventos de maior grau de risco em um conjunto relativamente reduzido de agentes causadores, com destaque para **Eletricidade** entre as categorias efetivamente identificadas.

A análise percentual complementa a frequência absoluta apresentada no item 6.3.1, permitindo avaliar a representatividade de cada agente causador no conjunto de eventos Alto/Crítico. Ressalta-se que os resultados representam frequência e associação, não permitindo, isoladamente, estabelecer relação direta de causa e efeito entre o agente causador e o grau de risco.


## 7. Autoavaliação


