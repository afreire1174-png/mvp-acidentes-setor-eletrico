# mvp-acidentes-setor-eletrico

MVP de Engenharia de Dados para análise de acidentes e ocorrências de segurança no setor elétrico, utilizando Databricks e tabelas Delta.

## 1. Contexto do Negócio

O setor elétrico envolve atividades que expõem trabalhadores a diferentes riscos ocupacionais, tornando a segurança do trabalho um tema relevante para a gestão das organizações. Nesse contexto, a análise de registros de acidentes, quase-acidentes e outras ocorrências pode contribuir para compreender as circunstâncias dos eventos e produzir informações que apoiem a discussão de medidas preventivas, especialmente aquelas voltadas à preservação da vida. Vale mencionar que essas ocorrências  podem apresentar diferentes níveis de gravidade, desde eventos sem lesão fatal até acidentes com óbito. Conhecer apenas o total de registros não é suficiente para orientar ações preventivas: é preciso entender como as ocorrências se distribuem e quais características aparecem com maior frequência nos casos fatais. Essa análise pode ajudar a identificar situações que merecem investigação e prioridade nas ações de prevenção.

###1.1. Objetivo

O objetivo deste MVP é descrever a distribuição das ocorrências registradas na base e comparar os acidentes fatais com as demais ocorrências. A análise buscará identificar diferenças observáveis nas variáveis disponíveis, como período, local, atividade e características do evento, conforme os campos efetivamente presentes no banco de dados. Os resultados terão caráter descritivo e exploratório; associações encontradas não serão tratadas como causas dos acidentes.

###1.2. Perguntas do Negócio

  • Quantas ocorrências estão registradas e como se distribuem por gravidade e ao longo do tempo?
  • Em quais locais, atividades ou categorias disponíveis na base há maior concentração de ocorrências?
  • Qual é a proporção de acidentes fatais no conjunto de registros?
  • Quais características são mais frequentes nos acidentes fatais e como sua distribuição difere da observada nas demais ocorrências?
  • Há padrões recorrentes que possam orientar investigações e ações de prevenção?

## 2. Coleta de Dados

A coleta dos dados deste MVP foi realizada por meio da obtenção de uma planilha corporativa contendo registros históricos de acidentes e ocorrências de segurança no setor elétrico. Trata-se de uma fonte de dados secundários, pois as informações já haviam sido registradas pela organização e foram utilizadas no projeto para fins de análise.

O arquivo de origem, denominado “BD_Acidentes_Dados_Brutos.xlsx”, foi obtido em formato Excel e contém uma única aba, chamada “Acidentes”, composta por 1.718 registros e 63 colunas, referentes às ocorrências registradas entre 2020 e 2025. Os dados dessa aba foram posteriormente exportados para o formato CSV para utilização nas etapas seguintes do pipeline.

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

###3.1 Estrutura e definição do modelo de dados

Neste MVP, foi adotado o modelo de tabela única desnormalizada (flat table), com base no arquivo BD_Acidentes_tratada.xlsx, resultante dos procedimentos de tratamento e proteção dos dados pessoais. A base é composta por 1.718 registros e 23 colunas, referentes ao período de 2020 a 2025. Os atributos de caracterização, localização, tempo, vínculo e consequências das ocorrências estão reunidos na mesma estrutura, permitindo consultas e agregações sem necessidade de junções entre tabelas.

A granularidade corresponde a uma linha por registro da base recebida. O campo Chave apresenta 1.718 valores distintos e nenhum valor ausente, sendo candidato a identificador único dos registros. Essa unicidade, contudo, não assegura que cada linha represente um acidente distinto, pois um mesmo evento pode envolver várias pessoas ou gerar múltiplos registros. Até que essa relação seja confirmada com a fonte, os totais devem ser interpretados como quantidades de registros.

A escolha pelo modelo flat considera o volume de dados e os objetivos analíticos do projeto. A estrutura permite avaliar a distribuição dos registros por período, classificação, gravidade, segmento, estado, categoria de local e agente causador, além de comparar as características dos registros fatais com as demais ocorrências. Nesse modelo, não há separação em tabelas fato e dimensão nem relacionamentos por chaves estrangeiras.

Para implementação no Databricks, propõe-se a tabela acidentes_analiticos, com tipos de dados definidos conforme o significado dos campos. Ano, Mês e Potencial devem ser representados como números inteiros. Os demais campos devem ser inicialmente armazenados como texto, incluindo Hora, que contém faixas horárias, e Tempo de Empresa, que apresenta intervalos e descrições de duração. O campo Chave deve ser mantido como identificador textual.

A análise temporal deve respeitar o detalhamento disponível na fonte. Como a planilha não contém a data completa das ocorrências, as séries temporais serão organizadas por ano e mês. Os campos Dia da Semana e Hora permitem análises complementares de distribuição, mas não possibilitam reconstruir a data exata dos eventos.

A identificação dos registros fatais será baseada no campo Classificação, considerando as categorias Fatalidade e Fatalidade Trajeto. O campo Gravidade, por representar níveis de gravidade que não correspondem diretamente à fatalidade, não será utilizado isoladamente para essa identificação. Propõe-se a criação de um indicador derivado que diferencie registros fatais, demais classificações reconhecidas e situações com classificação ausente ou não reconhecida.

###3.2 Catálogo e dicionário de dados

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

A substituição da coluna chave por id_registro proporcionou preenchimento completo e unicidade, permitindo distinguir cada linha da tabela analítica. Após a gravação, os identificadores serão reutilizados nas etapas seguintes, garantindo sua persistência. Essa transformação não comprova a ausência de registros duplicados em conteúdo nem conclui a anonimização dos demais atributos.






## 4. Carga e Pipeline
