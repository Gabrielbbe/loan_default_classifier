# Detecção de inadimplentes na concessão de empréstimos

<p align="center">
  <img src="https://github.com/user-attachments/assets/e5dbc382-f9ac-402c-b6c7-df6a11cdf267" />
 </p>
 
O objetivo principal do projeto é fazer um modelo estatístico para detectar possíveis inadimplentes para a concessão de empréstimos para diminuir os gastos e aumentar o lucro do banco. Para isso foi utilizado técnicas avançadas de análise de dados para detectar padrões e extrair insights dos dados, modelos de regressão binária clássico e bayesiano com função de ligação asssimétrica e modelos ensemble.
Foi utilizada a metodologia CRISP-DM para a análise em geral, na parte de entendimento dos dados é verificado a qualidade e consistência dos dados, ao final é sugerida mais uma etapa de calibragem do modelo em produção.
Na modelagem foi utilizada uma abordagem preditiva (onde nosso foco é apenas a capacidade de predição do modelo sem se importar com a interpretação do modelo) e utilizada uma abordagem padrão para tratar os dados desbalanceados, onde a distribuição original dos dados desbalanceados é mantida intocada.

[link report de negócios](https://github.com/Gabrielbbe/loan_default_classifier/blob/main/reports/report_negocios.pdf)

[link resumo técnico da análise](https://github.com/Gabrielbbe/loan_default_classifier/blob/main/reports/resumo_tecnico.pdf)

# Business Understanding

Quando um empréstimo é concebido e pago posteriormente o banco lucra, então se torna fundamental possuir critérios para a concessão de crédito para que o prejuízo gerado pelo clientes que não pagarem os empréstimos não cause prejuízo para o banco, para mostrar o quão efetivo pode ser o uso de um modelo irei mostrar um cenário onde todos recebem empréstimos e não é usado nenhum modelo e um cenário onde é utilizado um modelo. 

### Mas antes algumas métricas sobre os dados e o modelo importantes para o negócio:
 - Porcentagem de inadimplentes: 21.81%
 - Porcentagem de não inadimplentes: 78.81%
 - Quantidade média de um empréstimo de um não inadimplente: R$9.242
 - Quantidade média de um empréstimo de um inadimplente: R$10.861
 - Lucro médio gerado em um empréstimo dado a um não inadimplente: R$989
 - Prejuízo médio causado por um inadimplente é igual a quantidade média de um empréstimo + o suposto lucro médio em cima do empréstimo é igual a R$12.296
 - Custo médio de classificar um bom pagador como mal pagador é igual a R$989 pois perdemos a oportunidade.
 - Porcentagem de falsos negativos e falsos positivos no modelo final.

# traduzindo as necessidades de negócios para o objetivo da análise de dados
 Queremos detectar os clientes inadimplentes e os clientes não inadimeplentes de maneira balanceada, não errando muito julgando todos como inadimplentes para não diminiur os lucros, e não errando em julgar todos como não inadimplentes aumentando assim os gastos e nos prejudicando, então teremos que utilizar técnicas que consideram os verdadeiros positivos e os verdadeiros negativos. Para isso irei calibrar os modelos considerando as métricas KAPPA, MCC, G-Mean, F1-Score e KS. Irei utilizá-las pois o KAPPA, MCC e G-MEAN consideram as observações que foram positivas e negativas, o F1-score considera a precisão e recall, ou seja não considera os verdadeiro negativos, mas utilizei apenas para verificar como estava essa relação mas poderíamos descartá-la sem problemas e a métrica KS é um indicativo do quão bem o nosso modelo está distinguindo cada uma das classes em termos de probabilidades.
 Como modelos irei utilizar regressão binária clássica com função de ligação assimétrica (loglog e cloglog) e regressão bayesiana com função de ligação cauchito pois temos dados a variável resposta dicotômica e desbalanceada. Além disso irei utilizar dois modelos ensemble famosos, o Random Forest e o gradient boosted trees(xgboost) pois tendem a performar bem em tarefas de predição em classificação.

# Comparação do cenário sem modelo com o cenário com modelo

No cenário sem modelo não havia lucro, pegando a soma quantidade de dinheiro de todos os empréstimos vezes a taxa de juros menos a soma da quantidade de dinheiro nos empréstimos vezes a taxa de juros dos clientes inadimplentes havia um prejuízo de R$ 37.327.036.
Utilizando o modelo foi possível reverter esse cenário e obter lucro, aumento o lucro em 455% no final gerando R$132.637.701 de lucro.
 - Com uma porcentagem de falsos positivos(classificar um bom pagador como mal pagador) igual a aproximadamente 2%
 - porcentagem de falsos negativos(classificar um mal pagador como bom pagador) igual a aproximadamente 7%
 - porcentagem de verdadeiros negativos(classificar bom pagador como bom pagador) igual a aproximadamente 75%
 - porcentagem de verdadeiros positivos(classificar mal pagador como mal pagador) igual a aproximadamente 14%
   
(Como houve um arredondamento de casas decimais para fins de simplificação as porcentagens não somam em 100%, mas considerando as casas decimais no código a soma é igual a 100%.)

# Analisando o modelo
Comparação dos modelos : 
<p align="center">
<img src="https://github.com/user-attachments/assets/7f44b26e-dfdf-4c55-b4c0-0d269824ea21" />
  
 <p> <em>Nesta imagem temos a média da performance dos modelos em 3 conjuntos de dados estratificados de acordo com a variável resposta. rf = random forest, xgbc=gradient boosted trees, loglog/cloglog = regressão clássica com função de ligação loglog/cloglog, rpc = regressão bayesiana com função de ligação cauchito </em> </p>
</p>

Após testar todos os modelos e pela avaliação das métricas KS, MCC, Kappa, g-mean e f1-score foi escolhido o modelo gradient boosted trees e utilizando um conjunto de validação e o método GridSearchCV(), juntamente com a estimativa do Threshold que otimizava a métrica Kappa, foi possível otimizar o modelo e obter os seguintes parâmetros:

max_depth=10

min_samples_leaf=3

min_sample_split=7

threshold(regra de corte)=0.01 que foi estimada de tal forma a ser o valor que maximiza o coeficeinte KAPPA.

Além disso no modelo escolhido podemos observar que a métrica KS que tem valor entre 0.6 e 0.7 o que indica que o modelo tem uma boa capacidade de distinção entre as classes. Um fato importante de se comentar é que a regressão bayesiana com função de ligação cauchito poderia ter um desempenho melhor se otimizarmos alguns de seus hiperparâmetros, mas como ela é muito custosa compucationalmente e demora muito tempo para treinar e como os outros modelos estavam com um desempenho OK acabei deixando essa otimização de lado e prossegui com os outros modelos. 

Algumas estimativas do financeiro, de acordo com o conjunto de dados usando o modelo:
 - Faturamento estimado = R$151.097.396
 - Gasto com inadimplentes e o empréstimo dado aos não inadimplentes = R$18.459.695
 - Lucro total estimado com os clientes não inadimplentes = R$132.637.701

# Detalhes da análise técnica

Resumo da pipeline dos dados:
<p align="center">
<img src="https://github.com/user-attachments/assets/4387fa42-b22d-4368-aa12-a027ee70f47e" />
</p>

Mais detalhes sobre como e o porque de todos os passos de todas as etapas da análise de dados podem ser encontrados na pasta reports/resumo_tecnico.pdf [link](https://github.com/Gabrielbbe/loan_default_classifier/blob/main/reports/resumo_tecnico.pdf)

# Próximos passos

Alguns experimentos poderiam ser feitos, como por exemplo:  
- Modificar a transformação que é feita nas variável categóricas, utilizando outras técnicas como por exemplo o WOE(Weight of Evidence), entre outras já que existem várias, e verificar o impacto que isso tem no modelo.
- Adicionar mais variáveis ao modelo.
- Ajustar outros modelos de classificação.

