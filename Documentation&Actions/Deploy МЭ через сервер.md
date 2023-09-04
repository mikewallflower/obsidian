###### tags: #
###### links: [[]]
___
- [ ] Проблема с ролями
команды описаны в malie-etaji.ru/deploy.php

- файл деплой лежит в malie-etaji.ru/public_html/deploy.php

В гитхаб [[Webhook|webhook]]  находится по malie-etaji>settings>webhook
[[Pasted image 20230831161101.png]]
```pascal
Git Deployment Script

$echo $PWD  
/home/admin/web/malie-etaji.ru/public_html/public  
  
$whoami  
admin  
  
$git commit -m temp  
  
  
$git fetch --all  
  
  
$git reset --hard origin/main  
  
  
$git status  
On branch main nothing to commit, working tree clean  
  
$git submodule sync  
  
  
$git submodule update  
  
  
$git submodule status
```

После создания билда, коммита и пуша вводим в консоли 
```pascal
ssh root@188.246.224.99
```
Вводим "***yes***"
Вводим пароль от сервера:
```pascal
S/)z3EG@fg/9{P)T
```
Мы должны  быть  на ```

``` pascal
cd /home/admin/web/malie-etaji.ru/public_html
```
 
```pascal
git fetch --all
git reset --hard origin/main
```