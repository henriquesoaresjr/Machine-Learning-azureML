
# Trabalhando com Machine Learning na prática no Azure ML

Neste exercício, você usará um conjunto de dados de detalhes históricos de aluguel de bicicletas para treinar um modelo que prevê o número de aluguel de bicicletas esperado em um determinado dia, com base em características sazonais e meteorológicas. Vou mostrar a seguir o caminho que fiz para isso, de acordo com instruções do Microsoft Learning.

## Criando um recurso
Antes de tudo, criei um espaço de trabalho na minha conta do Azure, após isso criei um recurso dentro do portal com as seguintes configurações indicadas:

* Assinatura : sua assinatura do Azure .
* Grupo de recursos : Crie ou selecione um grupo de recursos .
* Nome : Insira um nome exclusivo para seu espaço de trabalho .
* Região : Selecione a região geográfica mais próxima .
* Conta de armazenamento : observe a nova conta de armazenamento padrão que será criada para seu espaço de trabalho .
* Cofre de chaves : Observe o novo cofre de chaves padrão que será criado para seu espaço de trabalho .
* Insights de aplicativo : observe o novo recurso padrão de insights de aplicativo que será criado para seu espaço de trabalho .
* Registro de contêiner : Nenhum ( um será criado automaticamente na primeira vez que você implantar um modelo em um contêiner ).
Após isso, fui no menu revisar e criar e selecionei criar para ter meu recurso.

# Usando aprendizado de maquina automatizado para treinar o modelo
O próximo passo foi usar o conjunto de dados históricos fornecidos para treinar o modelo de aprendizadode máquina para realizar a previsão de aluguel de bicicletas. A maneira para isso foi a seguinte:
* No Azure Machine Learning Studio , veja a página Automated ML (em Authoring ).

* Crie um novo trabalho de ML automatizado com as seguintes configurações, usando Next conforme necessário para avançar pela interface do usuário:
### Configurações básicas :
* Nome do trabalho : mslearn-bike-automl
* Novo nome do experimento : mslearn-bike-rental
* Descrição : Aprendizado de máquina automatizado para previsão de aluguel de bicicletas
* Marcadores : nenhum
### Tipo de tarefa e dados :

* Selecione o tipo de tarefa : Regressão
* Selecionar conjunto de dados : crie um novo conjunto de dados com as seguintes configurações:
### Tipo de dados :
* Nome : aluguel de bicicletas
* Descrição : dados históricos de aluguel de bicicletas
* Tipo : Tabular
* Fonte de dados : Selecione dos arquivos da web
* URL da Web: https://aka.ms/bike-rentals
* Ignorar validação de dados : não selecionar
### Configurações :
* Formato de arquivo : Delimitado
* Delimitador : Vírgula
* Codificação : UTF-8
* Cabeçalhos de coluna : apenas o primeiro arquivo possui cabeçalhos
* Pular linhas : Nenhum
* O conjunto de dados contém dados multilinhas : não selecione
### Esquema :
* Incluir todas as colunas exceto Caminho
* Revise os tipos detectados automaticamente
Após esses procedimentos, selecionei a opção criar, e após a criação do conjunto de dados, selecionei o conjunto de dados de aluguel de bicicletas para continuar a enviar o trabalho de ML automatizado.

### Configurações de tarefa :

* Tipo de tarefa : Regressão
* Conjunto de dados : aluguel de bicicletas
* Coluna de destino : Aluguéis (inteiro)
### Configurações adicionais :
* Métrica primária : raiz do erro quadrático médio normalizado
* Explique o melhor modelo : Não selecionado
* Usar todos os modelos suportados : Desmarcado (você restringirá o trabalho para tentar apenas alguns algoritmos específicos).
* Modelos permitidos : selecione apenas RandomForest e LightGBM — normalmente você gostaria de tentar o máximo possível, mas cada modelo adicionado aumenta o tempo necessário para executar o trabalho.
### Limites : expanda esta seção
* Máximo de testes : 3
* Máximo de testes simultâneos : 3
* Máximo de nós : 3
* Limite de pontuação da métrica : 0,085 ( para que, se um modelo atingir uma pontuação da métrica de erro quadrático médio normalizado de 0,085 ou menos, o trabalho termina. )
* Tempo limite : 15 (minutos)
* Tempo limite de iteração : 15 (minutos)
* Habilitar rescisão antecipada : selecionado
### Validação e teste :
* Tipo de validação : divisão de validação de trem
* Porcentagem de dados de validação : 10
* Conjunto de dados de teste : Nenhum
### Calcular :
* Selecione o tipo de computação : sem servidor
* Tipo de máquina virtual : CPU
* Camada de máquina virtual : Dedicada
* Tamanho da máquina virtual : Standard_DS3_V2*
* Número de instâncias : 1
* Se a sua assinatura restringir os tamanhos de VM disponíveis para você, escolha qualquer tamanho disponível.

Feito isso, enviei o trabalho de treinamento e aguardei alguns minutos para a realização do trabalho.

# Avaliando o melhor modelo
Após a conclusão do trabalho automatizado de aprendizado de máquina, observei qual foi o melhor resumo do modelo, na guia visão geral, e tambem visualizei mais detalhes clicando no campo "Nome do algoritmo", e revisei os gráficos residuais e predito_true. O gráfico de resíduos mostra os resíduos (as diferenças entre os valores previstos e reais) como um histograma. O gráfico predito_true compara os valores previstos com os valores verdadeiros.

# Implantando e testando o modelo
Na guia "Modelo" do melhor modelo treinado pelo meu trabalho automatizado de Machine Learning, selecionei a opção implantar e em seguida utilizei a opção de serviço Web para implantar o modelo com as seguintes configurações:
* Nome : prever-aluguéis
* Descrição : Prever aluguel de bicicletas
* Tipo de computação : Instância de Contêiner do Azure
* Habilitar autenticação : selecionado

Feito isso, iniciei e aguardei a finalização da implantação (status Succeeded).

# Testando o serviço Implantado 
Agora, havia chegado a hora de testar o serviço implantado. No menu "Pontos de extremidade", selecionei o modelo implantado e selcionei a opção "Testar". No painel de entrada para testar, substitui o modelo JSON pelos seguintes dados de entrada: 
```
 {
   "Inputs": { 
     "data": [
       {
         "day": 1,
         "mnth": 1,   
         "year": 2022,
         "season": 2,
         "holiday": 0,
         "weekday": 1,
         "workingday": 1,
         "weathersit": 2, 
         "temp": 0.3, 
         "atemp": 0.3,
         "hum": 0.3,
         "windspeed": 0.3 
       }
     ]    
   },   
   "GlobalParameters": 1.0
 }
```

Após isso, cliquei no botão teste e obtive o resultado:
```
{
  "Results": [
    353.84325661820674
  ]
}
```
Finalizado a atividade, fiz o processo de limpeza do espaço de trabalho, para evitar o consumo de dados desnecessarios na minha conta do Azure.

Esse foi um resumo simples do meu trabalho nessa atividade, que foi de muito bom proveito para expandir um pouco meu conhecimento!

Vou deixar abaixo dois links que me ajudaram muito e com muitos mais detalhes de como voce pode fazer esse processo, um da própria pagina do Microsoft Learninge outro de um artigo que vi no Github, que explica tambem como é feito o processo.

## Referência

 - [Explorando Machine Learning automatizado no Azure Machine Learning](https://microsoftlearning.github.io/mslearn-ai-fundamentals/Instructions/Labs/01-machine-learning.html)
 - [Artigo Github de GustavoPereira sobre Machine Learning no Azure ML](https://github.com/GustavoPereira-Dev/Machine-Learning-in-Azure-ML?tab=readme-ov-file)


