---
layout: post
title: "puppet-server-install"
date: 2017-07-24 01:15:00 +0300
comments: true
categories: 
  - puppet
  - git
  - r10k
---
Установка puppet server 5.x.

Все действия я буду производить на CentOs 7.x

Очень важно, чтобы время на серверах совпадало, поэтому установим **ntp**:  
`sudo yum -y install ntp`  
`sudo systemctl start ntpd`  
`sudo systemctl enable ntpd`  
(хотя позже я просто взвел его через puppet).

Для начала надо добавить puppet репозиторий (нужный вам вы можете найти [тут](https://docs.puppet.com/puppet/5.0/puppet_platform.html){:target="_blank"}):  
`sudo rpm -Uvh https://yum.puppetlabs.com/puppet5/puppet5-release-el-7.noarch.rpm`

И установить puppet server:  
`yum install puppetserver`
<!--more-->
Для возможности ревью изменений, я настрою работу _environments_ через git.  
Вы можете поднять свой сервер, я же воспользуюсь сервером gitlab.com, так как у них можно бесплатно завести приватный репозиторий.

Заходим на [gitlab](https://gitlab.com/){:target="_blank"}, заводим приватный репозиторий, делаем основную ветку **production**.

Так как у меня r10k (инструмент, который позволяет работать с динамическими окружениями) запускается из-под рута, то я добавил публичный ключ рута в gitlab.

Паппетлабовцы создали репозиторий, который можно склонировать, для работы с динамическим окружением и написали [инструкцию](https://github.com/puppetlabs/control-repo){:target="_blank"}. Выполните шаги, которые там указаны и возвращайтесь сюда.

Устанавливаем [r10k](https://github.com/adrienthebo/r10k){:target="_blank"} через puppet на мастер сервере:  
`/opt/puppetlabs/puppet/bin/gem install r10k`

Создаем для него конфиг:  
```
cat /etc/puppetlabs/r10k/r10k.yaml
# The location to use for storing cached Git repos
:cachedir: '/var/cache/r10k'

# A list of git repositories to create
:sources:
  # This will clone the git repository and instantiate an environment per
  # branch in /etc/puppetlabs/code/environments
  :my-org:
    remote: 'git@gitlab.com:azalio-puppet/control-repo.git'
    basedir: '/etc/puppetlabs/code/environments'
```
Как вы видете в **remote** надо указать свой репозиторий.

Все готово для запуска puppet и r10k.  
`systemctl start puppetserver`  
`systemctl enable puppetserver`  
(если у вас ругнулось на память, то можете уменьшить настройки в файле `/etc/sysconfig/puppetserver` *JAVA_ARGS*)

Запускаем r10k:  
`/opt/puppetlabs/puppet/bin/r10k deploy environment -pv`

Он подтянет все бранчи из вашего git репозитория.

Вот в принципе и все.  
Далее я сделал запуск r10k из крона, автоматическое подписывание сертификатов на основе доменного имени и запустил _puppet agent_ как демон.

P.S.:  
Предположим вам надо протестировать какую-то фичу и вы не хотите сразу пушить в мастер:

* Создаете ветку в вашем репозитории;  
* Коммитите, пушите;  
* На сервере, вызываете puppet agent с указанием нужного окружения:  
`puppet agent -tv --environment test_env`.

