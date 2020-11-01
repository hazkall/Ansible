<br />
<p align="center">
  <a href="https://github.com/othneildrew/Best-README-Template">
    <img src="assets/Ansible_logo.png" alt="Logo" width="300">
  </a>
  <h3 align="center"> Ansible - Laboratório e Aprendizado </h3>
</p>

# Arquivo de configuração Ansible

  - Principal arquivo de configuração do ansible
  - Sua localização padrão é /etc/ansible/ansible.cfg
  - Alterações e configuração são interpretadas respeitando a seguinte ordem:
    - Variavel ANSIBLE_CONFIG
    - ansible.cfg no diretório corrente
    - .ansible.cfg no diretório home
    - /etc/ansible/ansible.cfg
  - Customizaçao do ansible.cfg para a otimizaçao do trabalho, visto que o original vem com muitos parametros.
  - Todas as configuraçoes em /etc/ansible/ansible.cfg são aplicadas de forma global
  - Os arquivos de inventario tem hieraquia maior na aplicaçao das politicas
# ADHOC

  - Comando AD-HOC usa o binario do ansible para automatizar uma unica tarefa em um ou mais nos gerenciados. Os comandos sao rapidos e faceis, mas nao sao reutilizaveis.
  - Criando arquivo de inventario, atraves do arquivos hosts.
    - ansible -i <arquivo_inventario>
  - Caso nao seja nada especificado, por padrao ele ira ate /etc/ansible buscando o arquivo hosts. ( Isso acontece porque dentro do arquivo ansible.cfg, tem o parametro [inventory = /etc/ansible/hosts]
  - Como ja colocamos o IP da maquina teste no arquivo hosts, podemos usar o parametro "ALL" ou o proprio IP da maquina "192.168.56.3"
    - Parametro para indicar o modulo [-m] ( Modulo utilizado nesse esse exemplo --> ping)

```sh

ansible 192.168.56.3 -m ping

```
  - Executando esse comando teremos o retorno de erro de problema de conexao com o destino utilizando a conta root. Isso ocorre devido ao arquivo de configuraçao ansible.cfg ter o seguinte parametro [remote_user = root]
  - Sendo necessario alterar para outro usuario do sistema, nesse exemplo o usuario "felipe".
  - Sera necessario tambem realizar a configuracao para solicitar a senha do SSH, seguintes parametros [ask_pass = yes ; ask-sudo_pass = yes]
  - É importante levar em consideração que essa configuração é global, pois está no arquivo ansible.cfg, então é interessante manter o root e o ask_password em "no" e utilizarmos o parametro somente para esse comando especifico. Parametros:
    - "-u" --> indicar o usuário
    - "-k" --> solicitar a senha do usuário comum durante a execução do comando.

```sh

ansible 192.168.56.3 -u felipe -k -m ping

```
  
  - Tela de alerta do intepretador, por padrão quando ele executa uma tarefa ele procura o intepretador do python na maquina destino. Podemos colocar o auto-discover para ignorar esse alerta realizando o procedimento normalmente.Acrescentando ao ansible.cfg o parametro [interpreter_python = auto_legacy_silent](default é se o _silent)

  - Ansible Facts -> Quando executado playbooks, ad-hocs e afins, ele varre todos os targets montando um inventario dele, através de variaveis, montando um inventario completo da máquina. Esse inventario da maquina é feito através do modulo setup [-m setup]

```sh

ansible 192.168.56.3 -u felipe -k -m setup

```
  - Esse inventario pode facilitar na criação de playbooks, pois ele expoe muitas variaveis que podem ser utilizadas na criação de playbooks.
  - Modulo systemd (-m systemd) --> trabalhar com gerenciamento de serviços do sistema operacional destino(Necessário argumento)[-a]
    - Exemplo para restart no serviço do ssh daemon
    - Para realizar esse tipo de tarefa é necessário elevar o privilegio na máquina destino, então utilizamos o parametro [-b]
    - Utilizamos também a opção -K [-K, --ask-become-pass: ask for privilege escalation password]

```sh

ansible 192.168.56.3 -u felipe -b -K -m systemd -a "name=ssh state=restarted"
#name --> nome do serviço daemon 
#state --> ação realizada no serviço do ssh daemon

```

  - Ponto importante a observação é a cor do resultado e o estado de saida, no caso laranja CHANGED, ou alteração e Verde para sem alteração.

  - Modulo shell [-m shell] --> permite executar qualquer comando no host destino.

```sh

ansible 192.168.56.3 -u felipe -b -K -m shell -a "systemctl status sshd"

```
  - Caso não seja especificado nenhum modulo, será utilizado o modulo padrão de command, exemplo abaixo


```sh

ansible 192.168.56.3 -u felipe -k -a "pwd"

```

# SSH Keys

  - Autenticação entre troca de chaves de SSH Private and Public keys (Configurando a chave publica na máquina host)
  - Todas as configurações do SSH ficam no diretorio ~/.ssh/ como as chaves publicas e privadas.
  - Para geração das chaves publicas e privadas utilizamos:

```sh

ssh-keygen -t rsa -b 2048

#-t para informar o tipo de chave
#-b para informar a quantidade de bytes
#ele irá realizar algumas confirmações antes de gerar a key.

#Copiando a chave publica ao host client

ssh-copy-id -i id_rsa felipe@192.168.56.3

#Importante estar dentro do diretorio onde está a chave publica, antes de executar o comando.

```

  - Após a execução desse comando teremos uma chave Publica e Privada no diretorio apontado
  - Caso não fosse copiado para a máquina é possivel fazer no momento da conexão do comando ssh através do parametro -i [rsa]

# Inventory

  - Arquivo de texto onde será cadastrado todos os nossos alvos.
  - Dentro do arquivo hosts por padrão vem todos os exemplos para ajuda.
  - Caso não seja especificado nenhuma arquivo de inventario durante a execução do ansible [parametro -i] ele irá executar automáticamente o arquivo /etc/ansible/hosts.
  - Não há padrão de nome ou localização para o arquivo inventario, caso seja usado o [parametro -i] apontando arquivo de inventario.
 - Quando apontado o arquivo inventory, podemos utilizar um ip/hostname/group especifico ou apenas all para pegar todos os ips/hosts dentro do arquivo inventory.


```sh

ansible -i ~/Documents/Devops/Ansible/Invetory/hosts all -u felipe -m ping

ansible -i ~/Documents/Devops/Ansible/Invetory/hosts all -u felipe -b -K -m shell -a "systemctl status sshd"

```

# Grupos e SubGrupos

  - Para facilitar a administração de inumeros hosts, podemos acrescentar grupos dentro do arquivo inventory
    - Definindo grupo dentro do inventory --> [Group_Name]

Inventory file : hosts

```sh

[web_servers]
192.168.56.3

[deb_servers]
192.168.56.2

[windows_servers]
192.168.56.4

```
  - Agora apontamos somente o nome do grupo de execução do inventory, lembrando que agora a autenticação está sendo realizada através da SSH Public Key.
  
```sh

ansible -i hosts web_servers -u felipe -m ping

```

  - Podemos trabalhar também com subgrupos
    - Exemplo podemos criar um Grupo [Server] colocando dentro dele os outros 2 servidores criados de web e db.

Inventory file : hosts

```sh

[web_servers]
192.168.56.3

[db_servers]
192.168.56.2

[windows_servers]
192.168.56.4

[linux_servers:children]
web_servers
db_servers

```
```sh

ansible -i hosts linux_servers -u felipe -m ping

```
# Trabalhando com variaveis para Hosts e Grupos

- Hosts

  - Dentro do arquivos hosts podemos declarar uma variavel que irá representar aquele host no inventory.
  - Para isso usamos [variavel ansible_ssh_host="IP"]
  - Outras opções muito utilizadas também são:
    - ansible_ssh_user=felipe
    - ansible_ssh_pass=12345678

Inventory file : hosts

```sh

[web_servers]
apache ansible_ssh_host=192.168.56.3 ansible_ssh_user=felipe ansible_ssh_pass=12345678

[db_servers]
postgree ansible_ssh_host=192.168.56.2

[windows_servers]
192.168.56.4

[linux_servers:children]
web_servers
db_servers

```
  - Outras opções interessantes:
    - ansible_become=yes --> permite execução com elevação de privilegio
    - ansible_become_method=sudo --> metodo da elevação do privilegio
    - ansible_become_user=felipe --> defini o usuário da elevação do privilegio
    - ansible_become_pass=12345678 --> defini a senha da elevação do privilegio
    - ansible_shell_executable=/bin/bash
    - ansible_python_interpreter=/usr/bin/python
    - ansible_ssh_port=22 --> aponta a porta do SSH
    - ansible_connection=ssh --> tipo de conexão
    - Exemplo


Inventory file : hosts

```sh

[web_servers]
node ansible_ssh_host=192.168.56.3 ansible_ssh_user=felipe ansible_ssh_pass=12345678 ansible_become=yes ansible_become_method=sudo ansible_become_user=felipe ansible_become_pass=12345678

```
- Grupos

  - Para facilitar é possivel criar variaveis por grupos, onde as mesmas são aplicadas a todos os hosts do grupo
    - Para isso criamos um novo campo [web_servers:vars]

```sh

[web_servers]
apache ansible_ssh_host=192.168.56.3

#variaveis destinadas ao grupo web_servers
[web_servers:vars]
ansible_ssh_user=felipe
ansible_ssh_port=22
ansible_become=yes
ansible_become_method=sudo
ansible_become_user=felipe

```

```sh

ansible -i hosts web_servers -m shell -a "systemctl status sshd"

```

  - Comando acima executado como SUDO buscando todos os parametros em hosts apontados.

# Roles & Taks

  - As roles são um conjunto de itens independantes destinados a provisionar uma determinada aplicação/infraestrutura.
  - Itens:
    - Variaveis
    - Modulos
    - Modelos
    - Tarefas
    - Ações
  OBS:. É possivel associar roles com projetos.

  - As roles possuem uma estrutura padrão de diretorios para seus projetos:
```
playbook.yml
roles
└── common
    ├── defaults
    ├── files
    ├── handlers
    ├── meta
    ├── tasks
    ├── templates
    └── vars
```
  - Finalidades:
    - tasks --> lista tarefas para serem executadas em uma roles
    - handlers --> são manipuladores/eventos acionados por uma task
    - files --> arquivos utilizados para deploy dentro de uma role
    - templates --> modelos para deploy dentro de uma role (permite o uso de Variaveis)
    - vars --> variaveis adicionadas a uma role.
    - defaults --> variaveis padrao de uma role. Prioridade maxima
    - meta --> trata dependencias de uma role por outra role - primeiro diretorio a ser analisado.

Nota:. dentro dos diretorios tasks, handlers, vars, defaults e meta deverá existir um arquivo com o nome de main.yml para que o mesmo seja interpretado.

- O caminho tradicional para utilizar-se de uma role pode ser definido basicamente por:
  
```
- hosts: web_servers
  roles:
    - common
    - nginx
    - php
    - mysql

```
Nota: O que determina a execução de uma role são as tasks, cadastradas no arquivo tasks/main.yml

- Variaveis
  - São utilizadas pelo ansible para ajuda a trabalhar com diferentes tipos de sistema, arquiteturas e/ou auxiliar no processo de repetição durante a execução de uma role
  - O Ansible interpreta as variaveis de diferentes arquivos. Para isso, o mesmo mantem a seguinte ordem de prioridades(do menor para o maior)
  
    22 -  command line values (eg “-u user”) ( Menor prioridade -  Ultimo a ser interpretado)

    21 -  role defaults

    20 -  inventory file or script group vars

    19 - inventory group_vars/all

    18 - playbook group_vars/all

    17 - inventory group_vars/*

    16 - playbook group_vars/*

    15 - inventory file or script host vars

    14 - inventory host_vars/*

    13 - playbook host_vars/*

    12 - host facts / cached set_facts

    11 - play vars

    10 - play vars_prompt

    9 - play vars_files

    8 - role vars (defined in role/vars/main.yml)

    7 - block vars (only for tasks in block)

    6 - task vars (only for the task)

    5 - include_vars

    4 - set_facts / registered vars

    3 - role (and include_role) params

    2 - include params

    1 -  extra vars (always win precedence) ( Maior prioridade -  Primeiro a ser interpretado)


  - São comumente utilizadas pelos SysAdmin para facilitar no provisionamento de seus sistemas/infraestrutura. Entretanto, o ansible permite através do módulo setup obter o que chamamos de System Facts.
  - System Facts são descobertas pelo ansible através do módulo setup, trazendo informações de todo o sistema.

- Variaveis no arquivo inventory
  - É possivel reduzir a quantidade de declarações de variaveis no arquivo inventory usando subgrupos. Como exemplo abaixo:

```sh
#Nesse exemplo temos a declaração de variaveis 2x para web e db servers, onde ambos são iguais
#Podemos reduzir isso apontado a declaração da variavel para o Grupo linux_servers, onde ambos os grupos estão inclusos.

[linux_servers:children]
web_servers
db_servers

#variaveis para o grupo web_servers
[web_servers:vars]
ansible_ssh_user=felipe
ansible_ssh_port=22
ansible_become=yes
ansible_become_method=sudo
ansible_become_user=felipe

#variaveis para o grupo web_db
[web_db:vars]
ansible_ssh_user=felipe
ansible_ssh_port=22
ansible_become=yes
ansible_become_method=sudo
ansible_become_user=felipe


#Declaração após a remoção da declaração duplicada
################################################################
[linux_servers:children]
web_servers
db_servers
#variaveis para o grupo linux_servers
[linux_servers:vars]
ansible_ssh_user=felipe
ansible_ssh_port=22
ansible_become=yes
ansible_become_method=sudo
ansible_become_user=felipe
################################################################

```

  - Importante: caso seja definido por exemplo uma variavel de porta ssh para 22 e no grupo para 2222, irá prevalecer a porta do host pois ela tem maior prioridade. Exemplo:


```sh

[web-servers]
apache ansible_ssh_host=192.168.56.3 ansible_ssh_port=2222

[linux_servers:children]
web_servers
db_servers
#variaveis para o grupo linux_servers
[linux_servers:vars]
ansible_ssh_user=felipe
ansible_ssh_port=22
ansible_become=yes
ansible_become_method=sudo
ansible_become_user=felipe

```

- Podemos também apontar variaveis utilizando diretorios:
  - host_vars
  - group_vars
- Eles devem ficar localizados no diretorio raiz do Role, ou onde está sendo executado o Ansible.

```sh

.
├── ansible.md
├── assets
│   └── Ansible_logo.png
├── group_vars
├── host_vars
├── Invetory
│   └── hosts
├── Playbooks
└── roles
    └── common
        ├── defaults
        ├── files
        ├── handlers
        ├── meta
        ├── tasks
        ├── templates
        └── vars
```
- Dentro do diretorios criados, devemos criar um arquivo com os mesmos nomes de host associados no arquivo de inventario.
- Vamos usar o exemplo do host nome criado para o web_server: apache --> apache ansible_ssh_host=192.168.56.3
- devemos criar o arquivo dentro do diretorio hosts_vars com nome do apache, e dentro desse arquivo devemos declarar as variaveis especificas para esse host.
  - Obs:. nesse tipo de declaração não colocamos a declaração da variavel com "=" e sim com ":"

```sh

cd host_vars
cat << EOF > apache
ansible_ssh_port: 62222
ansible_become: yes
ansible_become_method: sudo
ansible_become_user: felipe
EOF

```

- Após essa declaração da variavel a porta foi alterada mesmo a configuração da porta estar no arquivo Inventory, pois o arquivo host_vars, possui maior prioridade, conforme ja explicado acima nos niveis de prioridade

- É possivel também declarar variaveis para os grupos utilizando o arquivo group_vars. Importante o nome do arquivo dever ser exatamente o nome do grupo.

```sh

cd group_vars
cat << EOF > web_servers
ansible_ssh_port: 62222
ansible_become: yes
ansible_become_method: sudo
ansible_become_user: felipe
EOF

cat << EOF > db_servers
ansible_ssh_port: 22
ansible_become: yes
ansible_become_method: sudo
ansible_become_user: felipe
EOF

#Pode ser declarado também para o Grupo dos "grupos"

cat << EOF > linux_servers
ansible_ssh_port: 22
ansible_become: yes
ansible_become_method: sudo
ansible_become_user: felipe
EOF


```

- Nesse caso no nivel de interpretação o group_vars tem menos prioridade que o file Inventory, então no caso ele vai interpretar o que está no arquivos hosts.
  - Dica ao inves de trabalhar com declaração de variaveis dentro do arquivo Inventory, podemos trabalhar com os arquivos de variaveis, tornando mais organizado.
  
    - Limpamos completamente o arquivo de Inventory, deixando somente o nome dos hosts e os grupos e subgrupos

```sh

[web_servers]
apache ansible_ssh_host=192.168.56.3 

[db_servers]
postgree  ansible_ssh_host=192.168.56.3192.168.56.2

[windows_servers]
windows ansible_ssh_host=192.168.56.4

[linux_servers:children]
web_servers
db_servers

#Deixamos criado o group_vars/linux_servers, para declaração de todas as variaveis necessárias para os servidores linux

ansible_ssh_user: felipe
ansible_ssh_port: 22
ansible_become: yes
ansible_become_method: sudo
ansible_become_user: felipe
ansible_connection: ssh

```

- Variaveis em Tasks e Templates

  - Fazendo a criação do arquivo main.yml dentro de tasks
  - Vamos fazer um exemplo de criação de task para instação de pacotes basicos:
  - Além da utilização de Loop com lista para item com {{item}} é possivel também utiliza $item
```sh
# Nesse exemplo criamos a variavel common_packages e dentro dela contem os pacotes que quero instalar
name: Instalação de Pacotes Exemplo
yum: name={{item}} state=latest
with_items:
  - "{{ common_packages }}"

#Ou fazemos de forma manual

name: Instalação de Pacotes Exemplo
yum: name=$item state=latest
with_items:
  - vim
  - net-tools
  - tcpdumb

#Ou da forma mais simplicada

name: Instalação de Pacotes Exemplo
yum: name={{ common_packages }}

#Nesses 2 exemplos o Ansible irá varrer a estrutura de diretorios em busca da variavel "common_packages"

```

- A declaração da variavel "common_packages" pode ser feita dentro do YAML main.yml no diretorio default do Roles.

Exemplo:

```sh

cd roles/common/default
vi main.yml

YAML file:

common_packages:
  - tcpdumb
  - vim
  - gcc

```

- Aplicando variaveis para Templates(Templates são arquivos que vou importar para dentro do meu target. Diferença entre file e templates? 
- Files são arquivos estaticos, já o Template é dinamico, onde por exemplo conseguimos pegar variaveis do Modulo inventario "setup" e utilizar no template para por exemplo utilizar o endereço IPV4 do host target)

Exemplo: 
    No comando "ansible -i hosts -m setup" temos umas das saidas a seguinte variavel:
```
    "ansible_all_ipv4_addresses": [
            "10.0.2.15",
            "192.168.56.3"
        ],

```
    Podemos utilizar a variavel "ansible_all_ipv4_addresses" dentro do nosso template.

```sh
cd roles/common/templates
vim motd

IP=ansible_all_ipv4_addresses
Nome.: {{ nome_empresa }}#Nesse caso o ansible irá varrer os diretorios em busca dessa variavel acima.

```
# Apoio com Linguagem YAML

- Sintax YAML
  - Utiliza-se de chave/valor para escrever roles, tasks e/ou playbooks no ansible. Ex:.
```
Yaml File:
- felipe:
    nome: Felipe
    sobrenome: Nobrega
```

  - Quase todos os arquivos .yml no ansible começam/contém com uma lista composta por chaves e valores. Todos os membros da lista iniciam-se com "- " (hifen seguido de um espaço)

- frutas:
    - maça
    - laranja
    - moranho

- A linguagem YAML possui a particularidade de segmentar seu arquivo em duas partes, inicio e fim, onde:

  --- --> indica o inicio do arquivo
  ... --> indica o fim do arquivo

- Variaveis são aplicadas entre "{{ chaves duplas dentro de aspas duplas}}".
- Apesar de não ser comum no ansible, pode-se especificar valores booleanos como verdadeiro/falso:

Ex:.
    criar_chave: yes ou true ou True ou TRUE
    instalar_pacote: no ou false ou False ou FALSE

- Exemplo de Sintax:

```yml

- felipe:
        name: Felipe Dias
        job: Devops
        skill:
              - linux
#ou

felipe: {name: felipe Dias, job: Devops, skill: linux}

```
# Tasks

  - São tarefas a serem executadas dentro de um host especifico. É obrigatorio a existencia do arquivo main.yml dentro do diretorio taks.
  - Dentro das tasks utilizamos varios modulos para executar as tarefas necessárias.
  - Modulos são componentes que vão auxiliar a criar ações especificas para uma role ou task.
  - É interessante antes pensar como será feito o processo da automação antes e depois buscar os modulos que irão atender aquela necessidade de solução.
  - Documentação oficial com o Index de todos os modulos https://docs.ansible.com/ansible/2.8/modules/modules_by_category.html

- Primeira Task
- Alvo -> web_server -> apache
- Passos:
  - 1 : criar o arquivo main.yml dentro do diretorio common/tasks
  - Nessa primeira task, 3 atividades serão executadas:
    - Atualizar o S.O através do yum
    - Instalação de alguns pacotes com yum
    - Criar link simbolico para criação da time zone.

Obs:. name=* --> significa todos os pacotes do sistema
      state=latest --> significa ultima versão do pacote disponivel
      vars --> declaramos a variavel packages e dentro dela os pacotes
      Utilizamos 2 modulos do ansible --> yum e file
```yml

---

#Yum update

- name: Atualizaçao do S.O
  yum: name=* state=latest update_cache=yes

#Yum install
- name: Instalaçao de pacotes
  yum:
    name: "{{ packages }}"
    state: latest
  vars:
    packages:
      - net-tools
      - vim
      - neofetch
      - nmap
#Criaçao de link para forçar time zone.
- name: Configuraçao de timezone
  file: src=/usr/share/zoneinfo/America/Sao_Paulo dest=/etc/localtime state=link force=yes

...

```


- Depois de criada nossa task, para executa-la precisamos criar o nosso playbook (Ele tem a função de orquestrar todas as roles e tarefas)
  - para isso precisamos criar o arquivo (Pode ter qualquer nome, no caso chamamos de playbook)
  - #vim playbook.yml

```yml
---
- name: Default Playbook - Starting Deploy
  hosts: postgree
  roles:
    - common 
...

```

Obs:. Foi necessário alterar o ansible_become_user: root para que fosse feita a elevação de privilegio necessário.

```sh

ansible-playbook -i Inventory/hosts postgree playbook.yml

```
# Files e Templates

  - Files --> Arquivos estaticos que vou copiar para o meu target destino
  - Templates --> Copia arquivos dinamicos, sendo possivel passar variaveis dentro do arquivo, o tornando dinamico.

  - Para o teste com file, foi criando um arquivo backup.tar.xz e foi movido para o diretorio files dentro de Roles

  - Por padrão o Ansible irá buscar o arquivo pelo nome dentro do Files, quando a task for executada  

```yml

- name: Copia de arquivos!
  copy: src=backup.tar.xz dest=/tmp/backup.tar.xz

```

Arquivo copiado para o diretorio tmp dentro do Target

 - Template
    - Podemos criar um arquivo de texto por exemplo utilizando variaveis do ansible_facts

Modelo de template:

[criado arquivo motd em roles/common/template/]

-------------------------------------------------------------------------------------

                            DB Server
                            
hostname: "{{ ansible_fqdn }}"
IP: "{{ ansible_all_ipv4_addresses }}"
Processador: "{{ ansible_processor }}"

-------------------------------------------------------------------------------------


```yml

- name: Copia de template!
  copy: src=motd dest=/tmp/motd

```
# Handlers

Por padrão o arquivo template será buscando dentro do diretorio template no role

- Handlers
  - Destinado a trabalhar com ações posteriores a determinada task. Exemplo: Feita a instalação do Nginx, após a instalação é necessáriio iniciar o serviço de forma automatica.
  - Ao inves de criar uma nova task para executar o serviço, é possivel criar um handler para chamar o inicio desse serviço.

- Vamos fazer a instalação do Nginx através da task:

```yml

- name: Instalação do Nginx
  yum: name=nginx state=latest
  notify: Restart nginx
  # A chamada do handler vai na task atrvés do parametro notify:
# Precisar ser exatamente o name: do handler

```

- Configurando o main.yml para o handler

```yml

- name: Restart nginx
  systemd: state=restarted enabled=yes daemon_reload=yes name=nginx

```

# Meta

  - Dentro do meta ficam as dependencias da role
  - Necessário a criação do arquivo main.yml dentro de roles/common/meta/
  - Exemplo: Voce pode criar uma ordem de como será executada as roles, definindo quem será o primeiro, segundo e em diante. Criando assim uma dependencia entre roles.

  - Nesse exemplo criamos uma nova role chamado php, estrutura

```

.
├── common
│   ├── defaults
│   ├── files
│   │   └── backup.tar.xz
│   ├── handlers
│   │   └── main.yml
│   ├── meta
│   ├── tasks
│   │   └── main.yml
│   ├── templates
│   │   └── motd
│   └── vars
└── php
    ├── defaults
    ├── files
    ├── handlers
    ├── meta
    ├── tasks
    │   └── main.yml
    ├── templates
    └── vars

```

Dentro do arquivo main.yml do meta [role=common]

```yml
---
dependencies:
  - { role:php }
...
```
  - Após criada a dependencia vamos criar uma nova task para a role --> php

```yml

- name: Instalação PHP
  yum: name=php state=latest

```

  - Quando a playbook for executada como foi apontado no meta do role common que o php é uma dependencia essa role será executada primeiro.

# Trabalhando com Condiições no Ansible

  - Exemplo de aplicação, por exemplo para identificar qual o sistema operacional e aplicar o pacote correto.
    - Através de variaveis do ansible facts ele conseguirá identificar qual é o sistema operacional do target.
    - Variavel do ansible_facts --> {{ ansible_distribution }}

- Para evidenciar os testes vamos criar um novo role, chamado update
  - Nesse role vamos ter 1 task com upate tanto para RedHat distros quanto para Debian Distros

```

.
├── common
│   ├── defaults
│   ├── files
│   │   └── backup.tar.xz
│   ├── handlers
│   │   └── main.yml
│   ├── meta
│   │   └── main.yml
│   ├── tasks
│   │   └── main.yml
│   ├── templates
│   │   └── motd
│   └── vars
├── php
│   ├── defaults
│   ├── files
│   ├── handlers
│   │   └── main.yml
│   ├── meta
│   ├── tasks
│   │   └── main.yml
│   ├── templates
│   └── vars
└── update
    ├── defaults
    ├── files
    ├── handlers
    ├── meta
    ├── tasks
    │   └── main.yml
    ├── templates
    └── vars

```

Dentro da task vamos criar 2 novas tasks: Yum e APT

Para realizar o processo de condição utilizamos o parametro "when", esse when irá verificar a variavel {{ ansible_distribution }}

```yml
---
#Update RedHat like

- name: Update system redhat
  yum: name=* state=latest security=yes
  when: ansible_distribution == 'CentOS' or ansible_distribution == 'RedHat'

#Update Debian like

- name: Update system debian
  apt: update_cache=yes upgrade=yes
  when: ansible_distribution == 'Debian' or ansible_distribution == 'Ubuntu'

...

```

Na saida do comando será possivel observar que o Redhat e o Ubuntu foram pulados e o Centos e Debian retornaram como ok.

```sh

#Comando executado da playbook.
ansible-playbook -i Inventory/hosts playbook.yml

```

```log



ok: [apache]
ok: [postgree]

TASK [update : Update system redhat] **********************************************************************************************
skipping: [apache]
ok: [postgree]

TASK [update : Update system debian] **********************************************************************************************
skipping: [postgree]
ok: [apache]

PLAY RECAP ************************************************************************************************************************
apache                     : ok=2    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
postgree                   : ok=2    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0  

```
# Ansible Playbooks

  - Playbooks são maneiras completamente diferente de usar o modo execução de tarefa ansible do que no modo adhoc e são particularmente poderosos. Simplificando, os playbooks são a base para um sistema de gerencimento de configuração e multi-máquina realmente simples, diferente de qualquer um que já existia, e um que seja muito adequado para implantar aplicativos complexos.
  - As playbooks são expressadas em YAML como mínimo de linguagem possivel.
  - De forma intencional, tenta não ser uma linguagem de programação e/ou script, e sim uma sequencia de instruções e/ou processo
  - Playbooks são compostos por um ou mais plays(tasks)
  - O objetivo é reunir um conjunto de hosts (alvos) para roles pré definidas, representados por tarefas.

  - Task nada mais é do que uma chamada para um modulo do ansible
  - Com a utilização de multiplas plays/tarefas, o ansible orquestra as implementações ali pré-definidas

  Linhas de comando com ansible-playbooks

  --> comando do Ansible para playbooks #ansible-playbook
      --> -i Aponta um arquivo de inventario
      --> Caso o ansible identifique uma falha de comunicação com algum host durante a execução ele irá pular a executar os proximos e depois criará um arquivo "playbook".retry onde nesse arquivo voce poderá executar somente os hosts que não foram alcançados.(Dentro dele ele terá os inventarios das maquinas nao atinjida)
      --> --limit Com a opção --limit é passado um unico alvo, no caso no exemplo apontamos o arquivo de inventario criado automaticamente playbook.retry (Pode ser usado para somente um host do arquivo inventory original apontado usando por exemplo --limit apache)
OBS>. Ponto importante para que essa função de retry funcione é necessário alterar o arquivo /etc/ansible.ansible.cfg/, acrescentando as seguintes linhas:
      [defaults]
```cfg 

#--- General settings
retry_files_enabled = True
retry_files_save_path = ~/.ansible-retry

```

```sh

      ansible-playbook -i Invetory/hosts playbook.yml
      #Na execução desse comando ambas as máquinas estavam desligadas apresentando o erro de não conseguir alcança-las
      #Foi gerado o arquivo dentro do home em ~/.ansible-retry/playbook.retry
      #Com isso vamos rodar novamente o comando ansible-playbook com o parametro --limit, para executar somente nos hosts com problema de conexão.
      ansible-playbook -i Invetory/hosts playbook.yml --limit @/home/felipe/.ansible-retry/playbook.retry

```
    --> --syntax-check faz a verificação se existe algum erro de sintaxe no playbooks ou as roles apontados no playbook.


```sh

      ansible-playbook -i Invetory/hosts playbook.yml --limit apache --syntax-check      

```
  --> --check faz a execução da playbook de forma simulada para verificar se está tudo ok.

  ```sh

      ansible-playbook -i Invetory/hosts playbook.yml --limit apache --check      

```
  -f --> numero de processos a ser executado ao mesmo tempo, pode ser alterado para um número maior caso o equipe que está rodando o Ansible possua mais potencia de processamento (Padrão = 5)
  --list-hosts --> mostra a lista de alvos que ele vai executar
  --list-tasks --> mostra a lista de tasks que serão executadas
  -v --> modo verbose


  - Escrevendo playbook

    - É possivel dentro do playbook alem de passar roles também passar tasks, handlers diretamente.
    - É possivel também fazer a declaração das variaveis diretamente no playbook.

playbook2.yml

Exemplo:. 

```yml
---
- hosts: linux_servers
  remote_user: felipe
  become: yes
  become_user: root
  order: sorted
  tasks:
  - name: Instalação do Nginx
    yum:
      name: nginx
      state: latest
    notify: 
      - restart nginx
      - stop apache 
  handlers:
  - name: restart nginx
    systemd:
      name: nginx
      state: restarted  
  - name: stop apache
    systemd:
      name: apache
      state: stopped  
...
```

É possivel também utilizar uma lista de itens para por exemplo remoção de pacotes

Usando padrão [u'pacote', u'n...']

Exemplo:


```yml
---
- hosts: linux_servers
  remote_user: felipe
  become: yes
  become_user: root
  order: sorted
  tasks:
  - name: Instalação do Nginx
    yum:
      name: "[u'httpd', u'nginx']"
      state: absent
```
Podemos também fazer a declaração de variaveis usando o padrão de variaveis --> {{ variavel }}


```yml
---
- hosts: linux_servers
  remote_user: felipe
  become: yes
  become_user: root
  order: sorted
  tasks:
  - name: Instalação do Nginx
    yum:
      name: "{{ packages }}"
      state: absent
    vars:
      packages:
        - httpd
        - nginx
```
# Windows Server WinRM

- Requisitos minimos para execução em Windows Server
  - Desktop -> Windows 7, 8.1 e 10
  - Servers -> 2008, 2008 R2, 2012, 2012 R2, 2016
  - Powershell 3.0
  - .NET 4.0
  - WinRM ativo

- WinRM
- Protocolo de comunicação remota utilizada pelo Windows Server
- Serviço escuta na porta 5985 HTTP e 5986 HTTPS
  - Para validar a porta de escuta(listening), execute o comando winrm enumerate winrm/config/Listener no poweshell
  - No arquivo de inventory, utiliza-se as variaveis abaixo para definir a conexao
    - ansible_winrm_scheme: http/https
    - ansible_port: 5985/5986
    - ansible_winrm_server_cert_validation: ignore
  - WinRM é nativo a partir do Windows Server 2012
  - Ansible utiliza-se de um plugin do python denominado de pywinrm para se comunicar com os alvos Windows
  - Por padrão, o WinRM permite conexões apenas de contas locais pertecentes ao grupo de Administadores
  - Para ajustar os privilegios de conexões, execute o comando winrm configSDDL default no powershell

# Mecanismos de autenticação do WinRM

- Autenticação basica
  - Metodo de autenticação mais simples e menos seguro
  - As senhas são codificadas em base64
  - Se não utilizado um canal seguro (https), as senhas podem facilmente ser decodificadas
  - Disponivel somente para contas locais

- Não vem ativas por padrão no Windows. Porem podem ser habilitadas utilizando a linha de comando no powershell:

```powershell

Set-Item -Path WSMan:\localhost\Service\Auth\Basic -Value $true

```
- Variaveis de conexão
```yml
ansible_user: LocalUsername
ansible_password: Password
ansible_connection: winrm
ansible_winrm_transport: basic
```
- Autenticação por certificado
  - Processo parecido com a troca de chaves do SSH porém com processo de geração diferente.
  - Forma segura e sem utilização de senhas para autenticação
  - disponiveis apenas para contas localizados
  - Não vem ativo por padrão no windows. Para habilitar:

```powershell

Set-Item -Path WSMan:\localhost\Service\Auth\Certificate -Value $true

```

- Variaveis de conexão
```yml
ansible_connection: winrm
ansible_winrm_cert_pem: /path/to/certificate/public/key.pem
ansible_winrm_cert_key_pem: /path/to/certificate/private/key.pem
ansible_winrm_transport: certificate

```

Os certificados podem ser gerados utilizando as ferramentas: Yum

- OpenSSL
- New-SelfSignedCertificate (PowerShell)
  - sera gerado um certificado PFX e deverá ser convertido para uma chave privada que será utilizado pelo pynwinrm.

Padrão de autenticação NTLM

  - Habilitado por padrão em alvos Windows, sem a necessidade de configurações previas
  - Disponiveis para contais locais e de dominio
  - Mais seguro que autenticação basica. Entretanto, por ser um protocolo mais antigo de autenticação, não suporta protocolos de criptografia mais recentes.

- Variaveis de conexão
```yml
ansible_user: LocalUsername
ansible_password: Password
ansible_connection: winrm
ansible_winrm_transport: ntlm

```

Padrão de autenticação Kerberos

- Opção mais recomendada para autenticação em ambientes de dominio
- Uma das opções mais seguras para se executar o WinRM
- Possui criptografia de mensagens HTTP e delegações de credenciais
- Necessita da instalação do pacote pywinrm[kerberos] e pacotes adicionais para configuração do kerberos na máquina onde está sendo executado o ansible.

- Variaveis de conexão
```yml
ansible_user: username@domain
ansible_password: Password
ansible_connection: winrm
ansible_winrm_transport: kerberos

```

# Preparando o ambiente Windows Server 2012

- o Ansible disponibiliza 2 Scripts prontos para executar no servidor Windows para fazer a preparação para receber o ansible

```powershell
#ConfigureRemotingForAnsible
$url = "https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"
$file = "$env:temp\ConfigureRemotingForAnsible.ps1"

(New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)

powershell.exe -ExecutionPolicy ByPass -File $file 

#Upgrade-Powershell_ps1

$url = "https://raw.githubusercontent.com/jborean93/ansible-windows/master/scripts/Upgrade-PowerShell.ps1"
$file = "$env:temp\Upgrade-PowerShell.ps1"
$username = "Administrator"
$password = "Password"

(New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Force

# version can be 3.0, 4.0 or 5.1
&$file -Version 5.1 -Username $username -Password $password -Verbose

```

- Efetuando a configuração da máquina linux com Ansible

Arquivo hosts:

```config
[windows_server]
windows ansible_ssh_host=192.168.56.4

```
group_vars --> windows_server

```yml

ansible_user: administrador
ansible_password: sdfdsfdsfds
ansible_port: 5986
ansible_connection: winrm
ansible_winrm_server_cert: ignore

```

Instalação da biblioteca do python no linux

```sh

sudo pip install "pywinrm>=0.3.0"

```

Realizando o teste com ping por meio do ADHOC

```sh

ansible -i Inventory/hosts -m winping
ansible -i Inventory/hosts -m setup

```

- Playbook basica no Windows Server
- Modo Chocolatey --> usando para gerenciar a instalação de pacotes através de linha de comando
- Processo de instalação do Chocolotey no Windows Server --> https://chocolatey.org/install

Criação da playbook:

- task: main.yml

```yml
---
- name: USERS | Adicionando usuário
  with_domain_user:
    name: felipe
    firstname: Felipe
    surname: Nobrega
    company: Saint-Gobain
    password: 123
    state: present
    groups:
      - Domain Admins
      - Administrators

- name: COPY | copiando arquivos de teste
  win_copy:
    src: /tmp/teste.txt
    dest: C:\teste.xt

- name: CHOCOLATEY | Instalando Chocolotey
  win_shell: Set-ExecutionPolicy ByPass -Scope Process -Force; iex ((New-Object System.Net.WebClient)).DownloadString('https://chocolatey.org/install.ps1'))

- name: CHOCOLATEY | Desabilitando enhanced exit codes
  win_chocolatey_feature:
    name: useEnhancedExitCodes
    state: disabled

- name: CHOCOLATEY | Instalando softwares
  win_chocolatey:
    name: "[u'git', u'fireox', u'putty']"
    state: latest

...
```
Executando a playbook

```sh

ansible-playbook -i Inventory/hosts playbook_windows.yml

```
# Ansible Galaxy

  - Comunidade de roles pré-feitas com o Galaxy
  --> https://galaxy.ansible.com/

  - Em linha de comando:
    - Para clonar um role pronto é necessário entrar na sessão desejada das roles no site do ansible galaxy e fazer a copia da linha do galaxy install para realizar o clone. (Bem parecido de como é feito no git clone)
    - É importante se atentar onde o clone está sendo realizado com padrão. Dependendo ele poderá tentar gravar em local com elevação de privilegio, para mudar o caminho, usamos o parametro --roles_path ou -p
      - Exemplo: ansible-galaxy collection install community.kubernetes --roles_path ~/Documents/Devops/Ansible/roles/
    - Caso ocorra uma atualização dessa role é possivel forçar a atualização da mesma usando o mesmo comando porém com parametro --force ou -f no fim.
    - Temos o parametro info que da informações sobre o projeto galaxy
      - Exemplo: ansible-galaxy info community.kubernetes --roles_path ~/Documents/Devops/Ansible/roles/
      - Podemos usar o Galaxy --list para listar nossas roles e versões. É importante realizar o versionamento das nossas roles através de arquivos dentro de meta.
      - Caso deseje remover a role do galaxy:
        - ansible-galaxy remove community.kubernetes --roles_path ~/Documents/Devops/Ansible/roles/

    - Criar estrutura de diretorios automatico:
      - ansible-galaxy init <role_name>
  
# Ansible Doc

  - linha de comando --> ansible-doc
  - Mostra a documentação de determinado modulo, sem a necessidade de estar conectado na internet.
  - ansible-doc -l ou --list --> lista todos os modulos existentes dentro do Ansible.
  - Para ver a documentação do modulo basta usar:
    - ansible-doc <modulo> -> Exemplo: ansible-doc aw3_s3
      - Com isso será apresentado a documentação completa do modulo e também exemplos de utilização.
  - Parametro -f além de mostrar o modulo existente, ele mostra também o caminho que está alocado na máquina atual.
  - Parametro -s lista um playbook de um determinado modo, como pode ser utilizado.

# Ansible Vault

  - Utilizado para pode criptografar os dados de um arquivo, onde para poder visualizar os dados de um arquivo, será necessário colocar uma senha.
    - Utilizado através do comando ansible-vault <file_name>
    - Assim que o comando é executado, é solicitada uma senha para esse arquivo.
    - Assim que concluida a criação, caso tente dar um cat ou abrir normalmente o arquivo estará 100% criptografado.
    - Caso seja necessário editar esse arquivo:
      - ansible-vault edit <file_name> -> será solicitada a senha criada.
      - ansible-vault view <file_name> -> será solicitada a senha criada. Voce poderá visualizar o conteudo do arquivo.
    - Caso seja necessário descriptograr o arquivo:
      - ansible-vault decrypt <file_name>
Exemplo de aplicação:

task para criar um usuário simples dentro do linux:

```sh

ansible-galaxy init create_user

```
roles/create_user/tasks/main.yml

```yml
---
- name: Create user
  user:
    name: teste
    shell: /bin/bash
    password: "{{ password }}"
    append: yes
...
```
roles/create_user/default/main.yml

```yml
password: 1234567890

```

Agora vamos criptografar o YAML com a senha:

```sh

ansible-vault encrypt roles/default/main.yml

```
Obs: É possivel apontar mais de um arquivo de cada vez ansible-vault encrypt <file_name1> <file_name2>

Executando a playbook

```sh

ansible-playbooks Inventory/hosts playbook5.yml --ask-vault-pass #sera solicitada a senha definida no encrypt

```

Criptografar senhas, dentro de arquivos:

https://stackoverflow.com/questions/30209062/ansible-how-to-encrypt-some-variables-in-an-inventory-file-in-a-separate-vault

# Utilização de varias tasks no mesmo role

- É possivel a utilização de varios tasks dentro da mesmo role apontando as mesmas dentro do arquivo main.yml 
- Para isso utilizamos a seguinte padronização:

arquivo tasks\main.yml

```yml
---
# Estamos importando as 2 tasks com nome php.yml e phpfpm.yml e será executada somente se estiver com status booleano enabled.

- import_tasks: php.yml
  when: php_enabled | bool
  tags: sysctl

- import_tasks: phpfpm.yml
  when: phpfpm_enabled | bool
  tags: prelink
...
~
```
  - Para o parametro enabled ser atendido é necessário que a variavel esteja declarada ou default ou vars do roles

arquivo default\main.yml

```yml
php_enabled: true
phpfpm_enabled: true

```

Caso algum deles esteja como falso a task correspondente será pulada.

# tags

As TAGS no Ansible é utilizada muito quando se cria Playbooks/Roles longas, nas quals são usadas para separál-las, e com isso, caso queira executar apenas as tasks de uma Role/Playbook que contenham a TAG XYZ, basta passar o parâmetro --tags XYZ durante a execução da Playbook.


Sobre o exemplo que você passou, esta sendo apenas aplicado as tags. Suponhando que você tenha mais de uma task com a TAG sysctl, e queir executá-la apenas ela, bastaria explicitar o --tags sysctl durante a execução da Playbook.


Outra possibilidade, é aplicar diretamente na Playbook a TAG que quer executar (observe a opção tags):

```yml
    - hosts: all
      tags: sysctl
      tasks:
        - import_tasks: php.yml
          when: php_enabled | bool
          tags: sysctl
     
    - import_tasks: phpfpm.yml
      when: phpfpm_enabled | bool
      tags: prelink
```




# Loops - Ansible

Padrão já conhecido com item como {{}}

```yml
- name: install vim, tmux
    name: "{{item}}"
    state: latest
  with_items:
    - vim
    - tmux
```

Mesma opção agora com o parametro loop (Mais atual e recomendado em documentações)

```yml

- name: install vim, tmux
    name: "{{item}}"
    state: latest
  loop:
    - vim
    - tmux


- name: Add users
  user:
    name: "{{item.name}}"
    state: present
    group: "{{item.groups}}"
  loop:
    - { name: 'teste1', groups: 'users'}
    - { name: 'teste2', groups: 'root'}

```

Outro exemplo de utilização de loops:


```yml

#Somente exibi o echo quando for diferente de 1 lembrando que está sendo jogado para uma variavel
- shell: echo "{{item}}"
  loop:
    - one
    - two
  registrer: echo
  changed_when: echo.stdout != "one"

```

Usamos nesse exemplo o parametro registrer. Ele tem a função de pegar uma saida e registrar numa variavel.

Muito usado com o modulo debug que mostra uma msg na tela. Como por exemplo o resultado de uma variavel.

```yml
- debug:
    msg: "{{item}}"
  loop: "{{ groups['all']}}" # Faz um loop no arquivo inventory e executa em determinado grupo( No caso desse exemplo será o grupo all)
  
```

Proximo exemplo é o parametro loop_control e pause. Limita um tempo para pause(intervalo entre os loops)

Nesse exemplo usamos uma pausa de 5 segundos

```yml
- name: adicionando users com delay de 5segundos
  user:
    name: "{{item}}"
    state: present
    groups: "users"
  loop:
    - user1
    - user2
  loop_control:    pause: 5

```