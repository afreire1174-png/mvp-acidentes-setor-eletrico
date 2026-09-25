# mvp-acidentes-setor-eletrico

MVP de Engenharia de Dados para análise de acidentes e ocorrências de segurança no setor elétrico, utilizando Databricks e tabelas Delta.

## 1. Contexto do Negócio

O setor elétrico envolve atividades que expõem trabalhadores a diferentes riscos ocupacionais, tornando a segurança do trabalho um tema relevante para a gestão das organizações. Nesse contexto, a análise de registros de acidentes, quase-acidentes e outras ocorrências pode contribuir para compreender as circunstâncias dos eventos e produzir informações que apoiem a discussão de medidas preventivas, especialmente aquelas voltadas à preservação da vida. Vale mencionar que essas ocorrências  podem apresentar diferentes níveis de gravidade, desde eventos sem lesão fatal até acidentes com óbito. Conhecer apenas o total de registros não é suficiente para orientar ações preventivas: é preciso entender como as ocorrências se distribuem e quais características aparecem com maior frequência nos casos fatais. Essa análise pode ajudar a identificar situações que merecem investigação e prioridade nas ações de prevenção.

### 1.1. Objetivo

O objetivo deste MVP é descrever a distribuição das ocorrências registradas na base e comparar os acidentes fatais com as demais ocorrências. A análise buscará identificar diferenças observáveis nas variáveis disponíveis, como período, local, atividade e características do evento, conforme os campos efetivamente presentes no banco de dados. Os resultados terão caráter descritivo e exploratório; associações encontradas não serão tratadas como causas dos acidentes.

### 1.2. Perguntas do Negócio

  • Quantas ocorrências estão registradas e como se distribuem por gravidade e ao longo do tempo?
  • Em quais locais, atividades ou categorias disponíveis na base há maior concentração de ocorrências?
  • Qual é a proporção de acidentes fatais no conjunto de registros?
  • Quais características são mais frequentes nos acidentes fatais e como sua distribuição difere da observada nas demais ocorrências?
  • Há padrões recorrentes que possam orientar investigações e ações de prevenção?

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

Neste trabalho, a apresentação da modelagem de dados foi organizada em duas subseções: Estrutura e definição do modelo de dados e Catálogo e dicionário de dados, considerando a base resultante dos procedimentos de anonimização. A primeira descreve a avaliação da consistência, da completude e da padronização dos dados, destacando as limitações e as necessidades de tratamento. A segunda documenta a estrutura da tabela analítica, apresentando os campos, seus significados, tipos de dados e regras de interpretação, de modo a apoiar sua utilização nas análises do projeto.

### 3.1 Estrutura e definição do modelo de dados

Neste MVP, foi adotado o modelo de tabela única desnormalizada (flat table), com base no arquivo BD_Acidentes_tratada.xlsx, resultante dos procedimentos de tratamento e proteção dos dados pessoais. A base é composta por 1.718 registros e 23 colunas, referentes ao período de 2020 a 2025. Os atributos de caracterização, localização, tempo, vínculo e consequências das ocorrências estão reunidos na mesma estrutura, permitindo consultas e agregações sem necessidade de junções entre tabelas.

A granularidade corresponde a uma linha por registro da base recebida. O campo Chave apresenta 1.718 valores distintos e nenhum valor ausente, sendo candidato a identificador único dos registros. Essa unicidade, contudo, não assegura que cada linha represente um acidente distinto, pois um mesmo evento pode envolver várias pessoas ou gerar múltiplos registros. Até que essa relação seja confirmada com a fonte, os totais devem ser interpretados como quantidades de registros.

A escolha pelo modelo flat considera o volume de dados e os objetivos analíticos do projeto. A estrutura permite avaliar a distribuição dos registros por período, classificação, gravidade, segmento, estado, categoria de local e agente causador, além de comparar as características dos registros fatais com as demais ocorrências. Nesse modelo, não há separação em tabelas fato e dimensão nem relacionamentos por chaves estrangeiras.

Para implementação no Databricks, propõe-se a tabela acidentes_analiticos, com tipos de dados definidos conforme o significado dos campos. Ano, Mês e Potencial devem ser representados como números inteiros. Os demais campos devem ser inicialmente armazenados como texto, incluindo Hora, que contém faixas horárias, e Tempo de Empresa, que apresenta intervalos e descrições de duração. O campo Chave deve ser mantido como identificador textual.

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

A substituição da coluna "chave" noa tabela original por id_registro proporcionou preenchimento completo e unicidade, permitindo distinguir cada linha da tabela analítica. Após a gravação, os identificadores serão reutilizados nas etapas seguintes, garantindo sua persistência. Essa transformação não comprova a ausência de registros duplicados em conteúdo nem conclui a anonimização dos demais atributos.


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

Foram criados os schemas `mvp_bronze`, `mvp_silver` e `mvp_gold`
no catálogo `workspace`, estabelecendo a organização das camadas
da arquitetura medalhão.

Também foram realizados o upload da planilha para o Volume
`arquivos_mvp`, a criação e persistência do identificador técnico,
a exclusão da coluna `Chave` da versão analítica e o perfilamento inicial.

Até esta etapa, os schemas estão disponíveis, mas as tabelas Delta
e as transformações entre Bronze, Silver e Gold ainda não foram
implementadas.


### 4.2 Ingestão dos dados no Databricks

A base original, descrita na seção Coleta de Dados, possui 1.718 registros
e 63 colunas. Para esta etapa, foi utilizada a versão
`BD_Acidentes_tratada_v3.xlsx`, composta por 1.718 registros e 23 colunas,
resultante da preparação anterior ao upload.

A relação dos campos removidos ou modificados e os critérios utilizados
nessa preparação ainda precisam ser documentados. As transformações
apresentadas nesta seção têm como ponto de partida a versão de 23 colunas.

A carga inicial foi realizada por meio do upload desse arquivo para
o Volume `arquivos_mvp`, pertencente ao schema `default` do catálogo
`workspace`, no Databricks. O notebook de processamento foi salvo
na pasta de usuário do Workspace.

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
A confirmação dos resultados será registrada após a execução
bem-sucedida.




### 4.3 Criação do identificador técnico e exclusão da chave original

Foi criado o campo `id_registro`, composto por um UUID aleatório para cada linha. Esse identificador é independente dos atributos da fonte
e permite identificar os registros sem incorporar informações pessoais em sua composição.

A coluna original `Chave` foi excluída da versão analítica. O arquivo de origem foi preservado, e a transformação não alterou a quantidade
de registros.

Os identificadores foram gerados uma única vez e persistidos na nova versão. Nas execuções posteriores, a versão salva deve ser carregada
para evitar a atribuição de novos códigos às mesmas linhas.

### 4.4 Persistência e validação da versão resultante

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

### 4.5 Resultados da transformação

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

### 4.6 Encadeamento das etapas e escopo implementado

O fluxo executado compreendeu:

1. Upload da planilha para o Volume.
2. Leitura da aba Acidentes.
3. Criação do identificador técnico.
4. Exclusão da coluna Chave da cópia analítica.
5. Gravação e conferência da versão resultante.
6. Releitura do arquivo salvo para o perfilamento de qualidade.

O diagnóstico de valores ausentes, tipos de dados e valores distintos é apresentado na seção Qualidade de Dados.

Até esta etapa, o fluxo utiliza arquivos Excel armazenados em um Volume.
Ainda não foi demonstrada a criação de tabelas Delta nem a implementação completa das camadas Bronze, Silver e Gold. Essas etapas deverão ser
documentadas conforme forem executadas.

A substituição da chave original não conclui a anonimização da base.
Descrições livres e combinações de atributos ainda requerem avaliação antes de qualquer divulgação. Os arquivos detalhados não integram
os materiais públicos do projeto.

## 5. Qualidade de Dados

A avaliação inicial da qualidade foi realizada no Databricks, utilizando Python e pandas, sobre a base com 1.718 registros e 23 colunas. Foram
analisados o preenchimento dos campos, os tipos reconhecidos na leitura e a quantidade de valores distintos. Os resultados e as limitações
identificadas são apresentados a seguir.

### 5.1 Completude dos dados

A completude foi avaliada pela quantidade e pelo percentual de valores ausentes em cada coluna. Células vazias e textos compostos apenas por espaços foram considerados ausentes no perfilamento.

As seguintes colunas apresentaram valores ausentes:

| Campo | Quantidade de valores ausentes | Percentual de valores ausentes |

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

Não foi realizada conferência sistemática com documentos de origem, relatórios de investigação ou responsáveis pelos registros. Dessa forma,
a acurácia permanece parcialmente não verificada.

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

| Criação de `id_registro` | Identificar cada linha por um código independente dos atributos originais. | 1.718 UUIDs distintos e nenhum valor ausente. |
| Exclusão de `Chave` da versão analítica | Retirar o identificador original dessa versão. | A base permaneceu com 23 colunas após a substituição. |
| Gravação e releitura da nova planilha | Persistir os identificadores e conferir sua preservação. | Versão salva e conferida, com 1.718 registros. |
| Reconhecimento de textos compostos apenas por espaços como ausentes | Evitar subestimação de valores ausentes no perfilamento. | Regra aplicada à cópia utilizada na avaliação, sem comprovação de gravação dessa alteração na base persistida. |

A versão resultante foi salva como `BD_Acidentes_com_id_sem_chave_v2.xlsx`. Nas etapas seguintes, esse arquivo será utilizado para preservar os identificadores já atribuídos.

Não foram executadas, no fluxo documentado até aqui, a imputação de valores ausentes, a exclusão de duplicidades de conteúdo, a padronização
completa das categorias ou a remoção de outliers. Essas ações dependerão de regras justificadas e deverão ser acompanhadas de nova avaliação
da qualidade.

## 6. Análise dos Resultados

A análise tem como objetivo responder às perguntas de negócio por meio
de contagens, distribuições percentuais, séries temporais e comparações
entre grupos. Serão utilizados os dados da tabela analítica após
a aplicação e a validação dos tratamentos documentados.

A unidade de análise corresponde a um registro de segurança.
A base inclui acidentes, quase acidentes, desvios críticos e doenças
ocupacionais. Portanto, o total de registros não deve ser interpretado
automaticamente como número de acidentes distintos ou de vítimas.

Os resultados serão apresentados com consultas, tabelas ou gráficos,
seguidos de interpretação e limitações. As análises deverão explicitar
o tratamento dos valores ausentes e o denominador dos percentuais.





## 7. Autoavaliação


