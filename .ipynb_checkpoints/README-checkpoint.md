# Instalação e Configuração de um container Docker Apache Hadoop 


### Resumo


Este documento foi elaborado, com informações básicas sobre como você pode preparar um ambiente de testes [Apache Hadoop](https://hadoop.apache.org/), usando um *container* [Docker](https://www.docker.com/) em sua máquina local.

Construir um ambiente de desenvolvimento e testes em um *container* Docker local não será uma tarefa fácil, mas nos permitirá, estudar, aprender, depurar erros, assim como, executar aplicações de análise de dados que forem propostas nas aulas do curso de [Formação Cientista de Dados da Data Science Academy](https://www.datascienceacademy.com.br).

O curso de Formação Cientista de Dados explica passo-a-passo como instalar e configurar em uma VM VirtualBox com Linux CentOS, o Apache Hadoop e alguns dos produtos do seu Ecossistema.


``` Objetivo deste documento, é ajudar na instalação e configuração manual de um container Docker Apache Hadoop, em modo Pseudo-Distribuído (Pseudo-Distributed). ``` 


No final desta jornada, teremos um ambiente de testes para estudos, onde será produtivo experimentar um framework para soluções de Big Data, tanto para armazenamento, quando processamento de dados distribuído. 


### Aviso


Esta é uma sugestão de uma configuração inicial para uso em máquinas de testes. Não implemente em ambiente de produção.


### Pré-requisitos:   

    - Máquina com arquitetura 64Bits e sistema operacional Linux instalado;
    - Você precisa ter o Docker instalado em sua máquina local;
    - Noções básicas do funcionamento do Docker;

<!-- #region -->
### Lembrando que:

Quando estiver trabalhando no container, não use o comando *exit* para sair do console, se fizer seu container morre e você perderá todos os pacotes instalados até este ponto. 

Use o seguinte atalho do teclado: 


### Mantenha o botão Crtl pressionado +p + q ### 

Assim, você sairá do container e ele continuará em execução.

```python 
$ docker attach <ID do Container> (voltar ao container)

```


![](img/container.png)




Na ilustração acima, os ambientes, máquina local e container Docker estão isolados, apesar de compartilharem dos recursos de equipamento físico da máquina local.

<!-- #region -->
# Iniciar a instalação

- Baixe uma [imagem Docker da distribuição Linux CentOS](https://hub.docker.com/_/centos);


```python
$ docker pull centos
```

![](img/docker-pull.png)
<!-- #endregion -->
<!-- #region -->
- Caso você tenha uma imagem local, use a instrução:

```python
$ docker pull localhost:5000/centos
```

<!-- #endregion -->

![](img/docker-pull-localhost.png)


### Execute o Containers Docker CentOS


- Para subir um container em modo interativo, use instrução:

<!-- #region -->
```python
$ docker container run -ti localhost:5000/centos 
```

<!-- #endregion -->

![](img/docker-container-run.png)


- Caso sua imagem esteja local, use a instrução:

<!-- #region -->
```python
$ docker container run -ti centos 
```
<!-- #endregion -->

### Preparando o ambiente CentOS


Instalaremos alguns utilitários, pacotes adicionais e configuraremos o serviço SSH. Execute as instruções abaixo:

<!-- #region -->
```bash
$ yum update -y
$ yum install kernel -y
$ yum install kernel-devel -y
$ yum install kernel-headers -y
$ yum install initscripts -y
$ yum install gcc -y
$ yum install make -y
$ yum install perl -y
$ yum install ncurses -y
$ yum install bzip2 -y
$ yum install unzip -y
$ yum install rsync -y
$ yum install wget -y
$ yum install net-tools -y
$ yum install nano -y
$ yum install iproute -y
$ yum install passwd -y
$ yum install sudo -y
$ yum install openssh-server -y
$ yum install openssh-clients -y
```
<!-- #endregion -->

### Configure o SSH


Edite o arquivo **/etc/ssh/sshd_config** e ajuste os seguintes parâmetros:
``` 
Port 22
ListemAdress 0.0.0.0
PermitRootLogin No
AllowUsers hadoop 
```


Editaremos com o *nano*, mas pode ser usado qualquer editor.

<!-- #region -->
```bash
$ sudo nano /etc/ssh/sshd_config
```
<!-- #endregion -->

![](img/docker-sshd.png)


Continuando...


![](img/docker-sshd2.png)


Para gravar as alterações, use o atalho do teclado: **(Ctrl + x + Y)**


###  Incluir os usuários de trabalho e administração


Com usuário administrador *root* execute as instruções abaixo:

<!-- #region -->
```bash
$ adduser santana (cria usuário de trabalho)
$ adduser hadoop  (cria usuário de administração)
$ passwd santana
$ passwd hadoop
```
<!-- #endregion -->

Edite o arquivo *sudoers* e ajuste as permissões do usuário *hadoop*, idênticas ao do usuário *root* no arquivo.

<!-- #region -->
```bash
$ sudo nano /etc/sudoers
```
<!-- #endregion -->

![](img/sudoers.png)


### Crie as chaves SSH no CentOS


Para gerar uma chave SSH em seu servidor Linux, execute o comando ``` ssh-keygen ```. 

<!-- #region -->
```bash
$ ssh-keygen -A 
```
<!-- #endregion -->

![](img/sshd-pwd.png)


### Configure acesso SSH, sem senha para o usuário *hadoop*


Verifique o acesso *ssh localhost*, é possível acessar sem uma senha? execute a instrução abaixo:

<!-- #region -->
```python 
$ ssh localhost
``` 
<!-- #endregion -->

![](img/teste-login1.png)


No console, não foi possível acessar sem uma senha. O Apache Hadoop precisa que o acesso seja via SSH e sem senha.


Para resolver esse problema, execute as instruções abaixo:

<!-- #region -->
```python
  $ su hadoop
  $ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
  $ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
  $ chmod 0600 ~/.ssh/authorized_keys
  $ sudo rm /run/nologin
```
<!-- #endregion -->

Verifique o acesso com o usuário *root*.

<!-- #region -->
```python
$ su root
$ ssh localhost
```
<!-- #endregion -->

![](img/root.png)


**Correto! o usuário root não tem permissão para acesso remoto no serviço.**


Verifique o acesso com o usuário *hadoop*.

<!-- #region -->
```python
$ su hadoop
$ ssh localhost
```
<!-- #endregion -->

![](img/hadoop.png)


**Funcionou!** o usuário *hadoop* tem acesso sem senha no ssh, agora podemos continuar a instalação do Apache Hadoop.


A partir deste ponto, você pode acessar o container via comandos ```docker attach``` ou ```ssh ```.


Para descobrir o IP do container use a instrução ```docker inspect <ID do Container>```.


### Backup


Agora que finalizamos a primeira etapa da preparação do ambiente, enviaremos uma imagem customizada do container CentOS, para o repositório local. .


Lembre-se, para sair do container e deixá-lo ainda em execução, é necessário pressionar **Crtl + p + q**. 

<!-- #region -->
```python 
$ docker container ls
```
<!-- #endregion -->

![](img/docker-container-ls.png)


Listaremos os containers em execução para descobrir qual é o ID do CONTAINER que estamos customizando. 

<!-- #region -->
```python 
$ docker commit -m "Apache Hadoop" <ID do Container>
```
<!-- #endregion -->

![](img/commit1.png)


Imagem gerada com sucesso! agora enviaremos a imagem para o repositório local. (Backup)

<!-- #region -->
```python 
$ docker image ls
```
<!-- #endregion -->

![](img/docker-image-ls.png)


A imagem customizada foi gerada, agora renomearemos a TAG e o repositório.

<!-- #region -->
```python 
$ docker tag 6361bba42a00 localhost:5000/centos:8.0
```
<!-- #endregion -->

![](img/docker-image-tag.png)


Feito! a imagem foi renomeada, agora vamos enviá-la para o repositório Docker Local.

<!-- #region -->
```python 
$ docker push localhost:5000/centos:8.0
```
<!-- #endregion -->

![](img/docker-push-centos-customizado.png)


Pronto! imagem arquivada com sucesso.


### Criaremos [Volumes Docker](https://docs.docker.com/storage/volumes/) para compartilharmos arquivos com a máquina local.


Para criar um volume, execute as instruções abaixo:

<!-- #region -->
```python 
$ docker volume create datasets
$ docker volume create hadoop_home
``` 
<!-- #endregion -->

![](img/docker-volume.png)


Os volumes foram criados, agora subiremos nova instância do container customizada para acesso às pastas.


Para exibir os containers ativos, execute as instruçôes abaixo:

<!-- #region -->
```python
$ docker ps
``` 
<!-- #endregion -->

![](img/docker-ps.png)

<!-- #region -->
```python
$ docker stop 6aa313b91f6d
``` 
<!-- #endregion -->

![](img/docker-ps2.png)


O container foi encerrado. Agora, subiremos o novo container com os acessos compartilhados.


Para instanciar o novo container, execute a instrução abaixo:

<!-- #region -->
```python
$ docker container run -ti \
    --mount type=volume,source=datasets,destination=/opt \
    --mount type=volume,source=hadoop_home,destination=/home/hadoop \
    localhost:5000/centos:8.0
```
<!-- #endregion -->

![](img/shell.png)


Pronto, o novo container foi criado. Testaremos os compartilhamentos de arquivos.


Para descobrir onde os volumes foram criados na máquina local, digite no console da máquina local a instrução abaixo:

<!-- #region -->
```python
$ docker inspect datasets
```
<!-- #endregion -->

![](img/datasets.png)


``` Acesse cd /var/lib/docker/volumes/datasets/_data ``` padronize neste diretório, cópia de todos os pacotes do Apache Hadoop baixados.


![](img/opt_local.png)


Para testarmos o mapeamento, faça uma copia do JDK para o diretório.

<!-- #region -->
```python
$ docker inspect hadoop_home
```
<!-- #endregion -->

![](img/hadoop_home.png)


``` Acesse cd /var/lib/docker/volumes/hadoop_home/_data ``` Este será um diretório persistente do usuário hadoop.


## Início da Instalação do Apache Hadoop


### O ambiente Hadoop será instalado e configurado com o login hadoop. 


Para logar no container com usuário hadoop, digite no console do container ``` su hadoop ```


### Iniciar a instalação do Java 1.8


Precisamos instalar o Java 8 porque alguns pacotes do Apache Hadoop, não são compatíveis com versões superiores do JDK. 


Faça o [download do pacote JDK](https://www.oracle.com/br/java/technologies/javase/javase-jdk8-downloads.html) no site da Oracle para sua máquina local e depois copie o pacote para: 


``` 
 /var/lib/docker/volumes/datasets/_data
```


![](img/download-java.png)


Eu vou usar o pacote TAR para a customização e padronização do ambiente.


O Java 1.8 foi baixado e copiado para o diretório``` datasets ``` na máquina local que é o diretório ``` /opt ``` no container Hadoop, esta customização nos permitirá agilidade na troca de arquivos entre os ambientes de teste e desenvolvimento.


2. Para instalar o java você precisa descompactar o arquivo jdk-8u281-linux-x64.tar.gz em /opt e configurar as variáveis de ambiente.


**Para instalar o JDK execute a sequencia de instruções abaixo, no console do container:**

<!-- #region -->
```python
$ su hadoop
$ cd /opt
$ sudo tar xvf jdk-8u281-linux-x64.tar.gz
$ sudo chown hadoop:hadoop -R jdk1.8.0_281/
$ sudo mv jdk1.8.0_281/ jdk/
$ sudo chmod 775 -R jdk

```
<!-- #endregion -->

![](img/java_opt.png)


Feito, o java foi copiado para /opt, a nomenclatura e as permissões foram ajustadas. Agora configuraremos as variáveis de ambiente e testaremos.


**Edite o arquivo .bashrc e inclua as variáveis de ambiente do JAVA (JDK)**


Execute a sequencia de instruções abaixo:

<!-- #region -->
```python
$ cd 
$ nano .bashrc (nano é um editor de texto leve e amigável para o Shell do Linux)

# JAVA
export JAVA_HOME=/opt/jdk
export PATH=$PATH:$JAVA_HOME/bin

$ source .bashrc
```
<!-- #endregion -->

![](img/bashrc_java.png)


Testaremos e verificaremos se o Java (JDK) foi instalado corretamente.


![](img/java-version.png)


O java 1.8 foi instalado com sucesso. 


# Instalação do Hadoop


Faça o [download do pacote Hadoop](http://hadoop.apache.org/) no site da Apache Software Fundation para sua máquina local.


![](img/download-hadoop.png)


Quando o download terminar, copie o pacote para: ```/var/lib/docker/volumes/datasets/_data ``` em sua máquina local, e inicie a instalação do Hadoop. 

**Resumindo:** 

A instalação do Hadoop, consiste em descompactar, e copiar o arquivo baixado para o diretório **/opt**, ajustar as permissões, nomenclaturas e inserir as variáveis de ambiente no arquivo **.bashrc** do usuário **hadoop**. Conforme sequência de instruções abaixo:

<!-- #region -->
```bash
$ sudo cp hadoop-3.2.2.tar.gz /var/lib/docker/volumes/datasets/_data
$ docker attach bb2b1c7c82fb
$ cd /opt
$ id (caso não esteja logado como hadoop, digite su haddop)
$ sudo tar xvf hadoop-3.2.2.tar.gz
$ sudo mv hadoop-3.2.2 hadoop
$ sudo chown hadoop:hadoop hadoop -R
$ sudo chmod 775 hadoop -R
$ mkdir /home/hadoop/Downloads
$ sudo mv hadoop-3.2.2.tar.gz /home/hadoop/Downloads
$ clear
$ ls -lia
``` 
<!-- #endregion -->

![](img/opt-hadoop.png)


Agora ajustaremos as variáveis de ambiente no arquivo *.bashrc*, seguindo as instruções abaixo:

<!-- #region -->
```bash
$ cd
$ nano .bashrc

# Apache Hadoop
export HADOOP_HOME=/opt/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

$ source .bashrc

``` 
<!-- #endregion -->

![](img/bashrc-hadoop.png)


As variáveis de ambiente foram configuradas. Execute a instrução abaixo:

<!-- #region -->
```bash
$  hadoop version
``` 
<!-- #endregion -->

![](img/versa-hadoop.png)


Feito! toda a configuração foi realizada com sucesso.


# Configuração do Hadoop


Configuraremos o Hadoop no Modo Pseudo-Distribuído (Pseudo-Distributed). 


Edite os arquivos de configuração, que estão no diretório ``` /opt/hadoop/etc/hadoop ``` e insira respectivamente os parâmetros abaixo:

1. core-site.xml

```xml
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
</property>
```


![](img/core-site-xml.png)


2. hdfs-site.xml

```xml
<property>
   <name>dfs.replication</name>
   <value>1</value>
</property>
```


![](img/hdfs-site-xml.png)


**O processo de instalação e configuração do Apache Hadoop foi concluído.**


# Iniciaremos o HDFS - Gerenciador de arquivos distribuído


Execute as seguintes instruções:


1. Formate o sistema de arquivos:

<!-- #region -->
``` bash
$ hdfs namenode -format
```
<!-- #endregion -->

![](img/namenode-format.png)


O sistema de arquivos foi formatado com sucesso!


2. Inicializando o HDFS. Para inicializar o HDFS, execute a instrução abaixo:

<!-- #region -->
``` bash
$ start-dfs.sh
```
<!-- #endregion -->

![](img/start-dfs-sh.png)


Ocorreu um erro! este problema ocorre porque precisamos inicializar o serviço SSH. 


3. Para inicializar o SSH no container digite:

<!-- #region -->
``` bash
$ sudo /usr/sbin/sshd
```
<!-- #endregion -->

4. Iniciaremos novamente o HDFS.

<!-- #region -->
``` bash
$ start-dfs.sh
```
<!-- #endregion -->

![](img/start-hdfs.sh2.png)


**Pronto, o serviço HDFS foi inicializado!**

# Experimento

<!-- #region -->
``` bash
$ jps (Esta é uma instrução, que permite visualizar todos os serviços do Hadoop em execução.)
```
<!-- #endregion -->

![](img/jps.png)


Os serviços estão rodando.


Para você acessar o *Namenode* via *browser*, e ver detalhes do ambiente.

<!-- #region -->
Abra o seu navegador favorito, digite o IP do container e a porta.

``` bash
$ http://172.17.0.3:9870/ 
```
<!-- #endregion -->

![](img/localhost.png)


Recomendo, navegar e explorar as opções.


**Comandos adicionais:**

<!-- #region -->
``` bash
$ hdfs dfs -ls /
$ hdfs dfs -mkdir /user
$ hdfs dfs -mkdir /user/hadoop
$ hdfs dfs -put /opt/hadoop/etc/hadoop/*xml /user/hadoop
$ hdfs dfs -ls /user/hadoop
$ stop-dfs.sh
```
<!-- #endregion -->

![](img/hdf-ls.png)


Pronto! ambiente funcionando perfeitamente.


# Configurando e inicializando o YARN


O YARN é um gerenciador de recursos para Apache Hadoop. 


O YARN roda sobre o HDFS e permite diferentes mecanismos de processamento de dados. O Apache YARN é considerado o sistema operacional de dados do Hadoop.


Para habilitar o YARN edite os arquivos de configuração, mapred-site.xml e yarn-site.xml, que estão no diretório /opt/hadoop/etc/hadoop, e insira respectivamente os parâmetros abaixo:


1. mapred-site.xml:
```xml
<property>
  <name>mapreduce.framework.name</name>
  <value>yarn</value>
</property>
<property>
  <name>mapreduce.application.classpath</name>
  <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
</property>
```


![](img/mapred-site-xml.png)


Grave a configuração.


1. yarn-site.xml:
```xml
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
<property>
    <name>yarn.nodemanager.env-whitelist</name>
    <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
</property>
```


![](img/yarn-site-xml.png)


Grave a configuração.


# Inicializaando o YARN


Execute as seguintes instruções:

<!-- #region -->
``` bash
$ start-yarn.sh
```
<!-- #endregion -->

![](img/start-yarn.png)


Os serviços foram inicializados.

<!-- #region -->
``` bash
$ jps
```
<!-- #endregion -->

![](img/jps2.png)


Os serviços HDFS e YARN estão rodando.


Para acessar o *ResourceManager* via *browser* e ler os detalhes do ambiente, abra o seu browser favorito, informe o seu *IP*, conforme instrução abaixo:

<!-- #region -->
``` bash
$ http://172.17.0.3:8088/
```
<!-- #endregion -->

![](img/localhost-yarn.png)

<!-- #region -->
Recomendo, navegar e explore as opções. E, antes de sair, encerre os serviços.

``` bash

$ stop-yarn.sh (YARN)
$ stop-dfs.sh (HDFS)
$ docker container stop <ID do Container> (Container Docker que está rodando o Hadoop)

```
    
<!-- #endregion -->

**Pronto!** 

Chegou o final da jornada, para a instalação e configuração do Apache Hadoop em um container Docker.



Espero ter contribuido com o seu desenvolvimento de alguma forma.


[Carlos Eugênio](https://carlosemsantana.github.io/)


### Referências 


1. [Curso de Formação Cientista de Dados da Data Science Academy](https://www.datascienceacademy.com.br)
2. [Docker Hub](https://hub.docker.com/_/centos)
3. Livro: Descomplicando o Docker 2a edição<br>
   Jeferson Fernando Noronha Vitalino<br>
   Marcus André Nunes Castro<br>
4. [The Hadoop Ecosystem Table](http://hadoopecosystemtable.github.io/)
5. [Gerando e usando chaves SSH para autenticação de host remoto](https://cloud.ibm.com/docs/ssh-keys?topic=ssh-keys-generating-and-using-ssh-keys-for-remote-host-authentication)
6. [Configurar acesso Cluster Pseudo-Distribuído](https://hadoop.apache.org/docs/r3.2.2/hadoop-project-dist/hadoop-common/SingleCluster.html#Pseudo-Distributed_Operation)
7. [Volumes Dockers](https://docs.docker.com/storage/volumes/)
8. [Apache Hadoop](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/SingleCluster.html)
9. [Apache Hadoop YARN](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html)
