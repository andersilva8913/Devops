# Devops

Criando ambiente virtual com Tomcat e  MySQL utilizando o Vagrant

Requisitos:

- Baixar o vagrant no site: https://www.vagrantup.com/downloads.html

- Será necessário também o Virtural Box(https://www.virtualbox.org/wiki/Downloads)


Criar a pasta do projeto, no meu caso a pasta vai ser chamar confin, depois dentro do diretório, será necessário criar o arquivo chamado Vagrant


```
Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/trusty64"
    config.vm.define :web do |web_config|
    end     
end
```

Depois de criado o arquivo entre no terminal e rode o seguinte comando `vagrant up`, isso fará com que o vagrant criei a máquina virtual e instale o ubuntu.

Com a máquina instalada, você pode se conectar utilizando o protocolo SSH, com comando `vagrant ssh`. Então podemos verificar que estamos dentro de um ambiente linux.

Agora precisamos instalar o Tomcat e Mysql, mas antes disso iremos configurar um ip, para facilitar o nosso acesso à maquina, então adicionaremos o seguinte trecho no arquivo Vagrant

```
   ...
    config.vm.define :web do |web_config|
      web_config.vm.network "private_network", ip: "192.168.56.10"
    end 
   ...
```
Agora será necessário exceutar o comando `vagrant reload`, para adicionarmos a configuração do IP e assim temos a máquina instalada e funcionando, e vamos para as instalações do Tomcat e Mysql.

Começaremos pelo Tomcat. Para isso iremos criar um diretório chamado manifests e dentro dele o arquivo chamado web.pp, neste arquivo configuraremos os comandos como se estivessemos dentro no terminal linux, então precisamos configurar os comandos de instalação:

```
exec { "apt-update":
  command => "/usr/bin/apt-get update"
}

package { ["openjdk-7-jre", "tomcat7"]:
    ensure => installed,
    require => Exec["apt-update"]
}
```

Note que para a instalação do tomcat, também foi necessário a instalação da jdk7, no caso estamos usando a Open JDK.


Agora iremos instalar o mysql, e configuraremos do mesmo jeito que fizemos com o tomcat


```
package { ["openjdk-7-jre", "tomcat7", "mysql-server"]:
    ensure => installed,
    require => Exec["apt-update"]
}
```

Agora com o mysql instalado é preciso criar o banco de dados que a nossa aplicação irá usar

```
exec { "controlefinanceiro":
    command => "mysqladmin -uroot cre7ate controle_financeiro",
    unless => "mysql -u root controle_financeiro",
    path => "/usr/bin",
    require => Service["mysql"]
}

exec { "mysql-password" :
    command => "mysql -uroot -e \"GRANT ALL PRIVILEGES ON * TO 'controlefinanceiro'@'%' IDENTIFIED BY '123456';\" controlefinanceiro",
    unless  => "mysql -ucontrolefinanceiro -p123456 controlefinanceiro",
    path => "/usr/bin",
    require => Exec["controlefinanceiro"]
}
```

E finalmente com o nosso ambiente montado podemos automatizar o deploy da nossa aplicação, para isso é necessário garantir que os serviços do tomcat e mysql estejam funcionando:


```
service { "tomcat7":
    ensure => running,
    enable => true,
    hasstatus => true,
    hasrestart => true,
    require => Package["tomcat7"]    
}

service { "mysql":
    ensure => running,
    enable => true,
    hasstatus => true,
    hasrestart => true,
    require => Package["mysql-server"]
}
```

Por último podemos adicionar a task para enviarmos o nosso arquivo war para o tomcat

```
file { "/var/lib/tomcat7/webapps/controle-financeiro.war":
    source => "/vagrant/manifests/controle-financeiro.war",
    owner => "tomcat7",
    group => "tomcat7",
    mode => 0644,
    require => Package["tomcat7"],
    notify => Service["tomcat7"]
}
```
...






































