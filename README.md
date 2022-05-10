# Ativar Luz do Teclado em Notebook SONY VAIO SVE14138CXB no UBUNTU e derivados


Solução encontrada para ativar a retroiluninação do teclado em notebook
Sony Vaio, que desativava a iluninação do teclado ao digitar no Ubuntu e 
derivados.


Após muita pesquisa e leitura na internet descobri que o arquivo referente
a luz do teclado estava em `/sys/devices/platform/sony-laptop/kbd_backlight`
e usando o comando `cat /sys/devices/platform/sony-laptop/kbd_backlight` no terminal temos 0(zero como retorno)  


```bash
cat /sys/devices/platform/sony-laptop/kbd_backlight 
0
```

O valor 0(zero) nesse arquivo indica para o sistema desativar a 
retroiluminacao, se mudarmos para o valor 1(um), ativamos a luz, mas temos
que fazer isso como sudo

```bash
sudo echo 1 > /sys/devices/platform/sony-laptop/kbd_backlight 
```

Esse ultimo comando escreve o valor 1 no arquivo. Se tudo deu certo seu 
teclado deve ascender ao pressionar alguma tecla.

Mas o problema ainda não foi resolvido, pois após reiniciar o sistema, 
o arquivo por ser proprietário volta ao valor 0(zero) fazendo com que 
façamos o mesmo processo para ativar novamente. Pra contornar isso
podemos criar uma unit para que o systemd inicie nosso script assim
que iniciar o sistema.

# SOLUÇÃO

## Passo 1

Execute no Terminal:
```bash
sudo nano /etc/systemd/system/initscr.service
```
O código acima cria o arquivo `initscr.service`, no caminnho 
`/etc/systemd/system/` com o nano.

Acrescente ao arquivo `initscr.service` o código abaixo e salve
```bash
[Unit]
Description=ScriptInit 

[Service]
Type=simple
RemainAfterExit=yes
ExecStart=/opt/initScripts/init.sh start
ExecStop=/opt/initScripts/init.sh stop
ExecReload=/opt/initScripts/init.sh restart 

[Install]
WantedBy=multi-user.target
```

Repare que na linha `ExecStart=/opt/initScripts/init.sh start` informamos
o caminho do arquivo que iremos criar na pasta `/opt/`.

## Passo 2

Execute no terminal

```bash
sudo nano /opt/initScripts/init.sh
```
No arquivo `init.sh` digite o script abaixo e salve

```bash
#!/bin/bash
cd /opt/initScripts/scr
bash script-luz-teclado.sh
```
De permissão de execução para o arquivo `init.sh`

```bash
chmod +x init.sh
```
## Passo 3

Crie uma pasta dentro de `initScripts` com o nome `scr`. É nessa pasta
que iremos colocar os scripts que queremos que iniciem junto com o sistema

No terminal execute:
```bash
mkdir src
cd src
nano script-luz-teclado.sh
```

No arquivo `script-luz-teclado.sh` digite e salve
```bash
#!/bin/bash
echo 1 > /sys/devices/platform/sony-laptop/kbd_backlight
```

## Passo 4
Atualizar o systemd e habilitar o que adicionamos execute no terminal
```bash
systemctl daemon-reload
systemctl enable initscr
```

Pronto! Agora o sistema vai executar o script de ativar a luz do teclado
sempre que o sistema iniciar!