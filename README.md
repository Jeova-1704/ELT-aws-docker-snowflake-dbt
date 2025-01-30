# Projeto engenharia de dados - ELT
![Captura de tela_20250106_153008](https://github.com/user-attachments/assets/1f55ad38-3c06-446a-a364-a311215ed068)

![projeto](https://github.com/user-attachments/assets/b16f855d-c6cd-4ab2-9ee9-f074ff76b65f)

### Dados para conexão ao banco de dados:
```
host: 159.223.187.110
dbname ou database ou banco de dados: novadrive
user ou usuário: etlreadonly
password ou senha: novadrive376A@
port ou porta: 5432
```

1. Banco de dados:
    1. Ele é um banco de dados operacional
    2. Banco de dados Postgresql da aplicação é um banco vivo, ou seja sempre são add novos itens.
    3. O banco de dados é acessado por meio de um usuário e senha validos apenas para leitura, assim com em um ambiente real.
2. Pipeline
    1. Vamos ter uma conta na aws
    2. Subir uma máquina ubuntu 
    3. subir o meu airflow usando docker
    4. Configurar o meu airflow 
3. Camada analitica 
    1. 2 bancos de dados, um para analitica e a de stage 
4. Tecnologias utilizadas:
    1. Postgres
    2. Airflow
    3. Docker
    4. AWS
        1. Ubuntu
        2. EC2 (Maquinas virtuais)
    5. snowflake (data warehouse)
    6. DBT (vai pegar os meus dados da camada de stage e transformar e colocar na camada analitica)
    7. na camada analitica os meus dados vão está prontos para consumo e é a unica camada que é exposta para os clientes e pessoas 

 

### Resumindo o meu projeto:

1. O postgres já está pronto, vou apenas pegar o meu usuario autenticado e pegar os dados por meio da leitura dos mesmos, já que o banco de dado é de uma aplicação real
2. O airflow vai orquestra todo o processo, controla as dependências, falhas, novas tentativas  etc
3. DBT é uma ferramenta de transformação dos dados dentro do banco de dados, usa a linguagem SQL e jinja2, não faz etl
4. Snowflake é um banco de dados analítico para tomada de decisões, fazer dashborad, machine learning, etc. Semelhante ao postgres porém voltado para analise de dados e tem uma performace maior em leituras e agregações.
![etapas projeto](https://github.com/user-attachments/assets/29bb7cc0-308c-4301-bab0-d08655796b09)

![Captura de tela_20250113_192616](https://github.com/user-attachments/assets/e5f2dcde-0ee1-447e-9501-a95c57e6ce36)
link para o sistema de vendas: http://143.244.215.137:3002/

#### Para acesso ao Sistema de Vendas:
http://143.244.215.137:3002/
**Login**: vendedor1
**Senha**: ia_nova_drive
Para consultar uma venda pelo ID da venda:
http://143.244.215.137:3002/procura

## Configurando AWS com airflow no aws com EC2 e Ubuntu

- Primeiro vamos criar uma conta na aws
- Vamos configurar o nosso docker com o airflow
    
    ```docker
    #1- Atualizar a lista de pacotes do APT:
    sudo apt-get update
     
    #2-Instalar pacotes necessários para adicionar um novo repositório via HTTPS:
    sudo apt-get install ca-certificates curl gnupg lsb-release
     
    #3- Criar diretório para armazenar as chaves de repositórios 
    sudo mkdir -m 0755 -p /etc/apt/keyrings
     
    #4- Adicionar a chave GPG do repositório do Docker:
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
     
    #5- Adicionar o repositório do Docker às fontes do APT:
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
     
    #6- Atualiza a lista de pacotes após adicionar o novo repositório do Docker
    sudo apt-get update
     
    #7- Instalar o Docker e componentes
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
     
    #8- Baixar o arquivo docker-compose.yaml do Airflow:
    curl -LfO 'https://airflow.apache.org/docs/apache-airflow/stable/docker-compose.yaml'
     
    #9- Criar diretórios para DAGs, logs e plugins:
    mkdir -p ./dags ./logs ./plugins
     
    #10- Criar um arquivo .env com o UID do usuário, usado pelo docker para permissões
    echo -e "AIRFLOW_UID=$(id -u)" > .env
     
    #11- inicia o airflow
    sudo  docker compose up airflow-init
     
    #12 -subir o Airflow em modo desacoplado
    sudo docker compose up -d
     
    #aguardar até tudo estar healty
     
    # copiar Public IPv4 DNS e colocar porta 8080, 
    # este é só um exemplo:
    http://ec2-35-175-126-189.compute-1.amazonaws.com:8080/
     
    #editar
    nano /home/ubuntu/docker-compose.yaml
     
    #mudar para falso
    AIRFLOW__CORE__LOAD_EXAMPLES: 'true'
     
    #reiniciar
    sudo docker compose stop
    sudo docker compose up -d
    sudo docker ps
    ```
    
- Vamos criar uma Instancia da EC2 na aws
    - Damos um nome para a nossa instância “NovaDriverMotors”
    - Vamos configurar uma máquina do tipo Ubuntu
    - Subi uma maquina paga com 0.09 dolares por hora, então é bom desligar ela recorrentemente
    - Vamos criar um par de chaves para usar e configurar, lembrar de salvar esse par pois vamos usar no processo de configuração
    - Depois de criar a maquina virtual podemos configurar para abrir a porta 8080 dela para efetuar as configurações
- Depois vamos salvar e verificar se temos o ssh instalado em nossos computadores
- eu já tenho
- vamos acessar a nossa maquina virtual pelo ssh terminal, para isso vamos usar o pair keys para acessar
- vamos acessar a nossa instancia
    - Vamos clicar me conectar
    - lá ele vai me dar a forma de como me conectar a minha instancia
        
        ```docker
        ssh -i "exemple-name.pem" ubuntu@numeros.algumacoisa.amazonaws.com
        ```
        
    - devo entrar na pasta onde guardei a minha chave
    - depois é só rodar o comando acima e funciona
    - Vamos agora rodar o comando acima do docker na nossa maquina para rodar o projeto, o comando está acima para instalar tudo e rodar o docker
- Depois disso tudo já estamos com tudo configurado e com o airflow rodando na porta 8080, podemos testar pegando o DNS IPv4 público e colocalndo :8080 ao final, ex:
    - [http://ec233.exemplenumbers.compute-1.amazonaws.com:8080/](http://ec2-54-147-250-145.compute-1.amazonaws.com:8080/)
- Logar com as credenciais padrões que são: **airlfow** para usuario e senha
- Vamos editar o nosso airflow para não deixar lá as dags de exemplo e poluir o nosso ambiente de trabalho
    
    ```docker
    nano docker-compose.yaml
    ```
    
    - depois vamos alterar     AIRFLOW__CORE__LOAD_EXAMPLES: 'true’ e colocar como falso.
    - Depois damos: crto + o depois enter e por ultimo crtl + x
    - Agora vamos reiniciar o nosso docker, com os seguintes comando:
        
        ```docker
        sudo docker compose stop
        sudo docker compose up -d
        sudo docker ps
        ```
        

## SnowFlake

- Criado em 2012 por especialistas em DataWarehouse
    - Trabalharam na Oracle em sistemas de MPP
    - Idealizaram a criação de um novo produto com melhorias em relação a DataWarehouses tradicionais.
- Lançado em 2014
- Plataforma SaaS que roda em servidores na nuvem como AWS, GCP e Azure.
- Organiza os dados por meio de um banco de dados, Schemas e tabelas
- Ele armazena em forma de colunas, o que facilita e agiliza a leitura ao contrario de outros banco de dados de linhas que facilita a inserção
- Escalabilidade:
    - Escala horizontal  → Aumenta o poder do datawarehouse
    - Escala vertical → add mais nós ao cluster
- Compartilhamento de dados de formas que não ocorre duplicações dados na hora do compartilhamento entre contas e usuários

→ Abaixo o script para configurar e criar os objetos necessários no Snowflake

```sql
create database novadrive;
create schema stage;
 
CREATE WAREHOUSE DEFAULT_WH;
 
CREATE TABLE veiculos (
    id_veiculos INTEGER,
    nome VARCHAR(255) NOT NULL,
    tipo VARCHAR(100) NOT NULL,
    valor DECIMAL(10, 2) NOT NULL,
    data_atualizacao TIMESTAMP_LTZ,
    data_inclusao TIMESTAMP_LTZ
);
 
CREATE TABLE estados (
    id_estados INTEGER,
    estado VARCHAR(100) NOT NULL,
    sigla CHAR(2) NOT NULL,
    data_inclusao TIMESTAMP_LTZ,
    data_atualizacao TIMESTAMP_LTZ
);
 
CREATE TABLE cidades (
    id_cidades INTEGER,
    cidade VARCHAR(255) NOT NULL,
    id_estados INTEGER NOT NULL,
    data_inclusao TIMESTAMP_LTZ,
    data_atualizacao TIMESTAMP_LTZ
 
);
 
CREATE TABLE concessionarias (
    id_concessionarias INTEGER,
    concessionaria VARCHAR(255) NOT NULL,
    id_cidades INTEGER NOT NULL,
    data_inclusao TIMESTAMP_LTZ,
    data_atualizacao TIMESTAMP_LTZ
);
 
CREATE TABLE vendedores (
    id_vendedores INTEGER,
    nome VARCHAR(255) NOT NULL,
    id_concessionarias INTEGER NOT NULL,
    data_inclusao TIMESTAMP_LTZ,
    data_atualizacao TIMESTAMP_LTZ
);
 
CREATE TABLE clientes (
    id_clientes INTEGER,
    cliente VARCHAR(255) NOT NULL,
    endereco TEXT NOT NULL,
    id_concessionarias INTEGER NOT NULL,
    data_inclusao TIMESTAMP_LTZ,
    data_atualizacao TIMESTAMP_LTZ
);
 
CREATE TABLE vendas (
    id_vendas INTEGER,
    id_veiculos INTEGER NOT NULL,
    id_concessionarias INTEGER NOT NULL,
    id_vendedores INTEGER NOT NULL,
    id_clientes INTEGER NOT NULL,
    valor_pago DECIMAL(10, 2) NOT NULL,
    data_venda TIMESTAMP_LTZ,
    data_inclusao TIMESTAMP_LTZ,
    data_atualizacao TIMESTAMP_LTZ
);
```

vamos jogar essas querys e criar o nosso banco de dados:


![Captura de tela_20250120_111943](https://github.com/user-attachments/assets/53048b80-164b-438c-b368-527e4d6da6b5)

### Obs:

Durante o processo vamos precisar diversas vezes das nossas credenciais do snowFlake, vamos anotar abaixo:

**Connection id:** snowflake

**password**: Sua senha

**database**: NOVADRIVE

**warehouse**: DEFAULT_WH

**schema**: STAGE 

**Login:**  Seu login

**account**: Sua account

Para encontrar o acount busque na url do snowflake após criar a conta, substitua a / por -.

Por exemplo:

https://app.snowflake.com/ktnlqur/va81917/#/data/databases

será ktnlqur-va81917

obs: na útlima seção vamos conectar o Looker Studio ao Snowflake, neste caso você deve completar account com a url, conforme exemplo abaixo:

ktnlqur-va81917.snowflakecomputing.com

## AirFlow

- Open Source → Criado pelo Airbnb em 2015 e mantido pela fundação Apache
- Desenvolvido em python
- Extensível
- O AirFlow não processa dados, ele coordena o processamento
- O processamento ocorre em:
    - Sistema operacional
    - Banco de dados
    - spark
    - Elastic Search
    - Etc.
- Em batch

Podemos acessar o Airflow por meio de duas opções, sendo elas:

- WEB → A interface gráfica do AirFlow
- CLI → As linhas de comando

**DAG: Pipeline:** São uma sequencia de passos/etapas que podem ser rodadas em paralelos ou individualmente. Ela define os processo que os operadores(tarefas) vão fazer. Não pode haver ciclos e nem voltas.

![Captura de tela_20250120_160703](https://github.com/user-attachments/assets/7131c7ae-2892-4158-85c7-47f9f17c8499)

Principais pontos do AirFlow:

- Controle de erros
- Registro de logs/auditoria
- Monitoramento/alerta
- Recuperação a partir de um ponto
- Dados históricos/diferenciação
- Alta disponibilidade
- Distribuição de Carga

**Operators**

- Algo a ser feito no pipeline
- Uma vez instanciado é uma tarefa
- Existem muitos tipos, derivam de baseOperator

![Captura de tela_20250120_161426](https://github.com/user-attachments/assets/5c1bb41f-7883-4137-896d-74ecf27137d4)

**Providers**

- Interação com outras ferramentas:
    - AWS
    - Databricks
    - MySQL
    - MongoDB

![Captura de tela_20250120_161542](https://github.com/user-attachments/assets/c48a13e4-3921-4514-a71f-d3a2256a0e48)


### Gerenciamento de falhas

- Permite novas tentativas na execução de task em caso de falhas
- O número de novas tentativas, a cada quanto tempo pode tentar novamente, etc,  pode ser configurado:
    - Globalmente(airflow.cfg)
    - Na dag
    - Na task
- O principal responsável por agendar a execução de DAGs é o **Scheduler**
- Operators são componentes que definem a execução de uma única tarefa
- Task instance é uma execução específica de uma tarefa em um momento no tempo
- A vantagem de usar Hooks no apache a airflow é facilitar a integração com sistemas e serviços externos

### Formas tradicionais de cargas de dados

- **Full Load**
    - **Carga completa:** A tabela de destino é truncada, e toda a tabela de origem é carregada novamente
    - **Vantagens:**
        - Garante a consistência dos dados
        - útil quando as atualizações são frequentes
        - A tabela de origem não deve ser muito grande
        - Não necessita controle de PK ou data de atualização ou inclusão
    - **Desvantagem:**
        - Ineficiente para grandes volumes de dados
        - Custo maior
- **Incremental:**
    - Apenas novos registros são incluídos. Baseados em chaves primárias ou algum timestamps. Atualização em registros existem não são considerados
    - **Vantagens:**
        - Mais eficiente que o full load para grandes volumes de dados
    - **Desvantagens:**
        - Ignora atualizações em registros existentes, o que pode levar a inconsistências de dados se os registros na origem forem alterados.
- **Upsert:**
    - Combina inserções e atualizações.
    - Novos registros são inseridos e registros existentes que foram modificados são atualizados
    - **Vantagens:**
        - Mantém a sincronia de dado entre origem e destino, capturando tanto novos dados quanto atualizações dos mesmo.
    - **Desvantagens:**
        - Pode ser mais complexo de implementar e mais lento do que o método incremental simples.
- **Append Only**
    - Todas as atualizações e inclusões são tratadas como novos registros. Não há atualização de registros existentes, mantendo um histórico completo de mudanças.
    - **Vantagens:**
        - Permite a manutenção de um histórico completo de alteração, facilitando a auditoria e a análise de tendências ao longo do tempo
    - **Desvantagens:**
        - O volume de dados cresce rapidamente, o que pode aumentar os custos de armazenamento e afetar o desempenho das consultas.
     

### No Projeto:

No caso do projeto vamos usar o ELT, pois vamos extrair, carregar os dado e transformar eles usando o DBT, um fluxo diferente do ETL, que se fossemos utilizar ele, séria feito de uma forma diferente, com o tratamento sendo feito com o spark ou com o proprio pandas e depois seria feito o carregamento dos dados.

### Criação da DAG

- Vamos usar o VSCode para criar a dag
- Quando estiver pronta vamos “colcar” no servidor do Airflow
    - **OBS:** Sabendo que na vida real a forma da gente fazer isso é através de CI/CD, e não copiando e colocando o código. Quando fazemos o código em python, damos um push e ele sobe automaticamente.

### Estrutura da DAG:

- Duas tasks para cada tabela:
    - get_max_id
    - load_incremental_data
- **Execução em paralelo por tabela:**
    
![Captura de tela_20250121_171102](https://github.com/user-attachments/assets/366e7f79-fbcf-43f3-b991-2358fe962e6f)

```python
from datetime import datetime, timedelta
from airflow.decorators import dag, task
from airflow.providers.postgres.hooks.postgres import PostgresHook
from airflow.providers.snowflake.hooks.snowflake import SnowflakeHook
 
default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2024, 1, 1),
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 0,
    'retry_delay': timedelta(minutes=1),
}
 
@dag(
    dag_id='postgres_to_snowflake',
    default_args=default_args,
    description='Load data incrementally from Postgres to Snowflake',
    schedule_interval=timedelta(days=1),
    catchup=False
)

def postgres_to_snowflake_etl():
    table_names = ['veiculos', 'estados', 'cidades', 'concessionarias', 'vendedores', 'clientes', 'vendas']
 
    for table_name in table_names:
        @task(task_id=f'get_max_id_{table_name}')
        def get_max_primary_key(table_name: str):
            with SnowflakeHook(snowflake_conn_id='snowflake').get_conn() as conn:
                with conn.cursor() as cursor:
                    cursor.execute(f"SELECT MAX(ID_{table_name}) FROM {table_name}")
                    max_id = cursor.fetchone()[0]
                    return max_id if max_id is not None else 0
 
        @task(task_id=f'load_data_{table_name}')
        def load_incremental_data(table_name: str, max_id: int):
            with PostgresHook(postgres_conn_id='postgres').get_conn() as pg_conn:
                with pg_conn.cursor() as pg_cursor:
                    primary_key = f'ID_{table_name}'
                    
                    pg_cursor.execute(f"SELECT column_name FROM information_schema.columns WHERE table_name = '{table_name}'")
                    columns = [row[0] for row in pg_cursor.fetchall()]
                    columns_list_str = ', '.join(columns)
                    placeholders = ', '.join(['%s'] * len(columns))
                    
                    pg_cursor.execute(f"SELECT {columns_list_str} FROM {table_name} WHERE {primary_key} > {max_id}")
                    rows = pg_cursor.fetchall()
                    
                    with SnowflakeHook(snowflake_conn_id='snowflake').get_conn() as sf_conn:
                        with sf_conn.cursor() as sf_cursor:
                            insert_query = f"INSERT INTO {table_name} ({columns_list_str}) VALUES ({placeholders})"
                            for row in rows:
                                sf_cursor.execute(insert_query, row)
 
        max_id = get_max_primary_key(table_name)
        load_incremental_data(table_name, max_id)
 
postgres_to_snowflake_etl_dag = postgres_to_snowflake_etl()
 
```

Agora vamos verificar a nossa conexão:

![Captura de tela_20250121_182004](https://github.com/user-attachments/assets/a6213a07-d5ff-4156-ab72-ebd95fd87525)

### Conectar ao airflow

- Vamos executar a nossa maquina virtual novamente
- entrar no airflow
- Entrar em admin-connections
![Captura de tela_20250121_183038](https://github.com/user-attachments/assets/3ebcecd7-65e0-42f6-8698-0b7dac214e23)
- Vamos clicar em add new record
- e então cadastrar a conexão
![Captura de tela_20250121_183408](https://github.com/user-attachments/assets/e34c27e2-eb1b-43ab-afe2-88ce8dd3e01a)
- Agora vamos fazer a conexão com o snowflake seguindo os mesmo passos
- Aqui está o exemplo de como fica após a criação
![Captura de tela_20250121_183737](https://github.com/user-attachments/assets/04041793-5899-4a44-bb9a-907bbd885fa2)
Com isso temos tudo pronto para rodar a nossa dag, para isso vamos nos conectara nossa maquina virtual e fazer o processo por lá

- após nos conectar, vamos entrar na pasta dags, cd dags
- dentro da pasta vamos criar um arquivo python touch novadrive.py
- vamos entrar no arquivo para editar ele, com o comando nano novadrive.py
- Vamos copiar o código python da nossa dag e colar aqui apertando shift + ins
- agora vamos salvar e sair, aperte crtl + x e depois enter
- depois vamos confirmar se salvou com o comando cat novadrive.py
- Agora vamos apenas esperar aparecer no airflow a nossa dag, pode demorar alguns minutos
    - Depois de um tempo atualize a pagina e então poderá ver ela criada:
![Captura de tela_20250122_093853](https://github.com/user-attachments/assets/9b642e5d-f854-486e-9b59-41cbf204fbea)
Com a nossa dag pronta, podemos agora clicar no nome dela e ter uma visualização melhor:
![Captura de tela_20250122_101251](https://github.com/user-attachments/assets/0aae05d6-bcc3-40ee-af06-8056d85f9bd3)
Com isso agora vamos clicar em tigger dag no icone de start, no recanto superior direito.
![Captura de tela_20250122_103850](https://github.com/user-attachments/assets/9b37a695-8113-467f-8115-5921ed4cb5be)

Podemos verificar que a dag foi executada com sucesso e podemos verificar agora o que ela vez no geral e como ficou o nosso sistema.
Agora vamos no nosso snowflake e vamos dar um select para ver se os dados já estão disponíveis lá
Execute o seguinte código abaixo para validar se existe algum dados:

```sql
select * from cidades limit 1;
select * from clientes limit 1;
select * from concessionarias limit 1;
select * from estados limit 1;
select * from veiculos limit 1;
select * from vendas limit 1;
select * from vendedores limit 1;
```

Com isso todos os meus dados já estão na área de stage e agora vamos apenas trabalhar com o DBT e o SnowFlake.

Com isso podemos desativar a nossa maquina virtual para não gerar custos desnecessários.

### DBT (Data build Tool)

Os dados estão na área de Stage, agora vamos fazer todo o tratamento com o DBT e jogar novamente no snowflake, mas dessa vez em um banco de dados analítico, isso agora sendo um banco de dados especifico para outras pessoas utilizarem ele, como no caso do pessoal que faz BI.

- Porque transformar no DW
    - DW otimizado para transformações
    - Mesmo ambiente, aumenta a simplicidade
    - Flexibilidade: mais fácil ajustar ou criar novas transformações
    - Dados brutos: Permite ser base para outras análises
- Basicamente o DBT é uma ferramenta para transformação de dados dentro do data warehouse
- dbt cloud
    - Na nuvem, mais fácil de utilizar
- dbt core
    - Utilizar a partir da linha de comando
- Modelo
    - Arquivo SQL
    - Usa a mesma sintaxe SQL do DW
    - Permite o uso de jinja2 (linguagem de template)
- O resultado da execução do modelo é um objeto(tabela ou view) com o mesmo nome do arquivo do modelo
- Permite consultar outros modelos: Pode-se criar um “pipeline” de transformação
- **Materialização:**
    - view: Cria view. Executando no momento da consulta
    - Table: Cria ou recria a tabela. Armazenados fisicamente
    - Incremental: Adiciona ou atualiza linhas com base em chave ou timestamp
    - Ephemeral: Não cria tabela. O sql é usado nas consultas subsequentes.
- **Macros**
    - Código reutilizável
    - Semelhante do conceito de funções
    - Baseado em jinja2
- **Testes:**
    - Testa a qualidade do processo de transformação
    - Também pode ser usado para regras de negócio
    - Um teste é uma consulta SQL que:
        - Se retornar vazio: **passou**
        - Se retornar uma ou mais linhas: **Falhou**
- Ele gera automaticamente documentação completo do projeto
    - Tabelas
    - Modelos
    - Diagramas
    - Etc

- Principais comandos do dbt:
    - dbt run
    - dbt run --models dim_clientes
    - dbt docs generate

- **Environment**
    - Development: Apenas 1
    - Deploy:
        - Pode definir outra conexão
        - Outro Schema
        - Roda a versão da branch main
    - Jobs:
        - Agendado ou sob demanda
        - contínuo: Quando há um pull request do git
- O dbt suporta a geração de infográficos como parte de seu processo de documentação

### Projeto:

- Camada de stage
- Camada de dimensões: Aspectos de negócios
- Camada de fatos: Dados quantitativos
- Camada analitica

![Captura de tela_20250122_194859](https://github.com/user-attachments/assets/8b79c763-7804-43d8-a905-4abb2eb107a1)

- **Origem (Stage Original)**
    - Os dados chegam de várias fontes e são armazenados no **pipeline do Airflow**.
    - Essas fontes são replicadas no seu próprio *Stage*, garantindo que os dados brutos sejam armazenados antes de qualquer transformação.
- **Stage (Pré-processamento e Organização)**
    - Aqui os dados são carregados das origens e organizados de maneira mais estruturada.
    - O objetivo é preparar os dados para a criação das dimensões de forma eficiente.
- **Dimensões (Camada de Otimização)**
    - Camada para padronizar, otimizar e permitir a reutilização.
    - Exemplos:
        - `dim_clientes`: Contém informações dos clientes.
        - `dim_veiculos`: Detalhes sobre os veículos.
        - Outras dimensões: vendedores, concessionárias, cidades, estados.
    - As dimensões servem como referência (FK) para a próxima etapa.
- **Fatos (Fact Table)**
    - Criada uma tabela de fatos chamada `fct_vendas`.
    - Esta tabela reúne todas as informações relevantes conectando as dimensões por chaves estrangeiras (FKs).
    - É aqui que os dados transacionais são integrados.
- **Análises (Resultados)**
    - Essa é a camada final, onde os dados processados são utilizados para criar análises e visualizações.
    - Exemplos:
        - `analise_vendas_vendedor`: Análise de vendas por vendedor.
        - `analise_vendas_veiculo`: Vendas agrupadas por tipo de veículo.
        - Outras análises: concessionárias, temporal.

### Primeiros passos com o DBT

OBS: Todo o código dessa etapa vai está disponível no github

- Vamos criar nossa conta lá
- Vamos criar um projeto e add os dados para nos conectarmos ao snowflake, dados esses que foram passados antes que utilizamos no pipeline.
- Depois vamos selecionara essa conexão e preencher os dados de conta, com username e senha, esses mesmo sendo o snowflake
- Depois vamos fazer a configuração de setup do repositório, tendo algumas opções:
![Captura de tela_20250122_221244](https://github.com/user-attachments/assets/4b0d4ae7-ac0f-4347-83a4-1eb1381c608a)

- Vamos usar o managed, pois evita futuras configurações, além de que é apenas para aprendizado e ele faz toda a configuração e gerenciamento das branchs, o que facilita o desnevolvimento.
- Após isso vamos iniciar um novo projeto assim que todas as etapas derem certo.
- Podemos ter uma visualização de como fica o nosso stage:
    
![Captura de tela_20250122_230523](https://github.com/user-attachments/assets/946c15a2-c382-4675-a8f7-2650523fdaea)

- Para cada modelo que vamos criar, rodando o seguinte comando para criar eles:  dbt run
- caso dê certo ele cria no nosso snowflake uma tabela já com os dados de cada modelo de forma trada e salva todos eles padronizando na forma que escolhemos
- Após criar todos os modelos vamos criar as dimensões
    
![Captura de tela_20250123_094451](https://github.com/user-attachments/assets/6c263f5c-1458-4b8e-9aab-55c6f5e7139e)

- Com isso já temos as nossas views e as nossas tables criadas, só irmos conferir lá no snowflake
- Vamos criar agora a nossa fct_vendas.sql, que quando criarmos vamos ter o seguinte diagrama:
    
![Captura de tela_20250123_100057](https://github.com/user-attachments/assets/0cf9fd05-db00-49c8-9c90-afc19b0c323b)

- E um diagrama geral que segue a seguinte forma:
![Captura de tela_20250123_100203](https://github.com/user-attachments/assets/54971ae5-949d-4c03-9326-0b002dee747e)

### Análises

- Agora vamos fazer as analises dos nossos modelos e que o nosso cliente pediu
    1. Vendas por concessionária 
    2. Vendas por modelo de veículo 
    3. Vendas por vendedor 
    4. Vendas temporal (Uma série temporal de vendas de veículos)
    
- Com tudo pronto é só rodar o comando dbt run e então abrir o snowflake para visualizar tudo o que fizemos.
- Com isso já temos toda a estrutura do projeto finalizada



