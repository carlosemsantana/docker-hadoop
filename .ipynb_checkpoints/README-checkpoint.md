# Instalação e Configuração de container Docker Apache Hadoop 


### Resumo


Esta página foi elaborada com informações básicas sobre como você pode preparar um ambiente de testes [Apache Hadoop](https://hadoop.apache.org/) usando um "*container*" [Docker](https://www.docker.com/) em sua máquina local.

Construir um ambiente de desenvolvimento e testes em um "**container**" Docker local não será uma tarefa fácil, mas nos permitirá, estudar e aprender, depurar erros e no meu caso executar aplicações de análise de dados propostas nas aulas do curso de [Formação Cientista de Dados da Data Science Academy](https://www.datascienceacademy.com.br) o qual estou concluíndo.

No curso de Formação Cientista de Dados aprendemos a instalar e configurar de forma manual e a partir do zero em uma VM VirtualBox com Linux CentOS o Apache Hadoop e alguns dos produtos do seu Ecossistema.


``` Objetivo deste documento é ajudar a você instalar e configurar de forma manual em um container Docker o Apache Hadoop em modo Pseudo-Distribuído (Pseudo-Distributed). ``` 


No final desta jornada teremos um ambiente de testes Apache Hadoop para estudos onde será produtivo experimentar o principal Framework para soluções de Big Data do mercado, tanto para armazenamento quando processamento de dados distribuído. 


### Aviso


Esta é uma sugestão de uma configuração inicial para uso em máquinas de testes. Não implemente em ambiente de produção.


### Pré-requisitos:    

    - Máquina com arquitetura 64Bits e sistema operacional Linux instalado.
    - Você precisa ter o Docker instalado em sua máquina local.
    - Noções básicas do funcionamento do Docker.


### Iniciar a instalação


- Baixar uma [imagem Docker da distribuição Linux CentOS](https://hub.docker.com/_/centos)


```python
$ docker pull centos
```

![](img/docker-pull.png)

<!-- #region -->
- Caso você tenha uma imagem local, use a instrução:

```python
$ docker pull localhost:5000/centos
```

<!-- #endregion -->

![](img/docker-pull-localhost.png)


### Executar o Containers Docker CentOS


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

### Preparar o ambiente CentOS 


Agora vamos instalar alguns utilitários, pacotes adicionais e configurar o serviço SSH.

<!-- #region -->
```bash
$ yum update -y
$ yum install kernel -y
$ yum install kernel-devel -y
$ yum install kernel-headers -y
$ yum install initscripts
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
```
<!-- #endregion -->

### Instalar e configurar o SSH

<!-- #region -->
```bash
$ yum install openssh-server -y
$ yum install openssh-clients -y
```
<!-- #endregion -->

Editar o arquivo **/etc/ssh/sshd_config** e ajustar os seguintes parâmetros:
``` 
Port 22
ListemAdress 0.0.0.0
PermitRootLogin No
AllowUsers santana, hadoop 
```
(**Lembrando que:** no parâmetro AllowUsers estou usando o meu usuário, mas na sua máquina você ajusta o seu.)


Para editar os arquivos de configuração no console do Linux eu uso o comando: ```nano``` que é mais amigável que o ```vim```.

<!-- #region -->
```bash
$ sudo nano /etc/ssh/sshd_config
```
<!-- #endregion -->

![](img/docker-sshd.png)


Continuação da edição do arquivo de configuração di SSHD_CONFIG


![](img/docker-sshd2.png)


Para gravar as alterações use o atalho do teclado: **(Ctrl + x + Y)**


### Criar os usuários de trabalho e administração

<!-- #region -->
```bash
$ adduser santana (cria usuário de trabalho)
$ adduser hadoop  (cria usuário de administração)
$ passwd santana
$ passwd hadoop
```
<!-- #endregion -->

Editar o arquivo ***/etc/sudoers*** e configure as permissões de acesso dos usuários, no meu caso, 
ajustei as permissões do usuário hadoop idênticas ao do usuário root no arquivo.

<!-- #region -->
```bash
$ sudo nano /etc/sudoers
```
<!-- #endregion -->

![](img/sudoers.png)


### Atenção 

Não use o comando "exit" para sair do console, se fizer seu container morre e você perderá todos os pacotes instalados até este ponto. 

Use o seguinte atalho do teclado: 


### mantenha o botão Crtl pressionado +p + q ### 


Assim, você sairá do container e ele continuará em execução.


### Gerar chaves SSH no Linux


Para gerar uma chave SSH em seu servidor Linux, execute o comando ``` ssh-keygen ```. 

<!-- #region -->
```bash
$ ssh-keygen -A 
```
<!-- #endregion -->

![](img/sshd-pwd.png)


### Configurar acesso SSH sem senha para o usuário hadoop


Antes, verifique se você pode fazer o ssh para o localhost sem uma senha longa:

<!-- #region -->
```python 
$ ssh localhost
``` 
<!-- #endregion -->

![](img/teste-login1.png)


Não foi possível fazer o ssh para localhost sem uma senha, então, execute a sequência de comandos abaixo:

<!-- #region -->
```python
  $ su hadoop
  $ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
  $ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
  $ chmod 0600 ~/.ssh/authorized_keys
  $ sudo rm /run/nologin
```
<!-- #endregion -->

Verifique o acesso com o usuário root.

<!-- #region -->
```python
$ su root
$ ssh localhost
```
<!-- #endregion -->

![](img/root.png)


***Correto! o usuário root não tem permissão para acesso remoto no serviço.***


Verifique o acesso com o usuário hadoop.

<!-- #region -->
```python
$ su hadoop
$ ssh localhost
```
<!-- #endregion -->

![](img/hadoop.png)


**Funcionou!** o usuário hadoop tem acesso sem senha no ssh, agora podemos continuar a instalação do Apache Hadoop


### Backup


Agora que finalizamos a primeira etapa da preparação do ambiente, vamos gerar uma imagem customizada do container CentOS e enviá-lo para o repositório local. 


Lembre-se de que para sair do container e deixá-lo ainda em execução é necessário pressionar **Crtl + p + q**. 

<!-- #region -->
```python 
$ docker container ls
```
<!-- #endregion -->

![](img/docker-container-ls.png)


Vamos listar os containers em execução para descobrir qual é o ID do CONTAINER que estamos customizando. 

<!-- #region -->
```python 
$ docker commit -m "Apache Hadoop" <ID do Container>
```
<!-- #endregion -->

![](img/commit1.png)


Imagem gerada com sucesso! agora vamos enviar a imagem para o repositório.

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


### Vamos criar [Volumes Docker](https://docs.docker.com/storage/volumes/) para compartilharmos arquivos com a máquina local.


Ao compartilhar uma pasta com a máquina local nos permitirá maior agilidade no download e cópia dos pacotes de softwares da máquina local para o container onde estamos instalando o Apache Hadoop. 


### Criar volumes

<!-- #region -->
```python 
$ docker volume create datasets
$ docker volume create hadoop_home
``` 
<!-- #endregion -->

![](img/docker-volume.png)


Os volumes foram criados, agora vamos subir nova instância do container customizada para acesso as pastas.


Para o container atual

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


O container foi encerrado. Agora, vamos subir o novo container com os acessos compartilhados.

<!-- #region -->
```python
$ docker container run -ti \
    --mount type=volume,source=datasets,destination=/opt \
    --mount type=volume,source=hadoop_home,destination=/home/hadoop \
    localhost:5000/centos:8.0
```
<!-- #endregion -->

![](img/shell.png)


Pronto, o novo container foi criado. Vamos testar os compartilhamentos de arquivos, vou copiar arquivos para os diretótios **datasets** e **hadoop_home** na máquina local. Que serão listados respectivamente em **/opt** e **/home/hadoop** do container.


Para descobrir onde os volumes foram criados na máquina local digite no console da máquina local:

<!-- #region -->
```python
$ docker inspect datasets
```
<!-- #endregion -->

![](img/datasets.png)


``` Acesse cd /var/lib/docker/volumes/datasets/_data ``` vou padronizar neste diretório a cópia de todos os pacotes do Apache Hadoop que serão baixaremos.


![](img/opt_local.png)


Para testarmos o mapeamento copiei o JDK que instalaremos.

<!-- #region -->
```python
$ docker inspect hadoop_home
```
<!-- #endregion -->

![](img/hadoop_home.png)


``` Acesse cd /var/lib/docker/volumes/hadoop_home/_data ``` Este será um diretório persistente do usuário hadoop.


## Agora vamos a Instalção do Apache Hadoop


### O ambiente Hadoop será instalado e configurado com o login hadoop. 


Para logar no container com usuário hadoop, digite no console do container ``` su hadoop ```


### Iniciar a instalação do Java 1.8


Precisamos instalar o Java 8 porque alguns pacotes do Apache Hadoop, não são compatíveis com versões superiores do JDK. 


1. Faça o [download do pacote JDK](https://www.oracle.com/br/java/technologies/javase/javase-jdk8-downloads.html) no site da Oracle para sua máquina local e depois copie o pacote para: 


``` 
 /var/lib/docker/volumes/datasets/_data
```


![](img/download-java.png)


Eu vou usar o pacote TAR para a customização e padronização do ambiente.


O Java 1.8 foi baixado e copiado para o diretório``` datasets ``` na máquina local que é o diretório ``` /opt ``` no container Hadoop, esta customização nos permitirá agilidade na troca de arquivos entre os ambientes de teste e desenvolvimento.


2. Para instalar o java você precisa descompactar o arquivo jdk-8u281-linux-x64.tar.gz em /opt e configurar as variáveis de ambiente.


**Para instalar o JDK execute a sequencia de instruções abaixo no console do container:**

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


Feito, o java foi copiado para /opt e ajustamos a nomenclatura as permissões. Agora vamos configuras as variáveis de ambiente e testar.


**Vamos editar o arquivo .bashrc e incluir as variáveis de memória do JAVA (JDK)**


Execute a sequencia de instruções abaixo:

<!-- #region -->
```python
$ cd 
$ nano .bashrc (nano é um editor de texto leve e amigável para o Shell do Linux)
$ source .bashrc
```
<!-- #endregion -->

![](img/bashrc_java.png)


Vamos testar e verificar se o Java (JDK) foi instalado corretamente.


![](img/java-version.png)


Feito! o java 1.8 foi instalado com sucesso. 

```python
continua...
```

```python

```

```python

```

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

```python

```
