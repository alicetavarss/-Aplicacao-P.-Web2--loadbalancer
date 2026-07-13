#Load Balancer com Nginx e Docker

Este projeto implementa uma infraestrutura de alta disponibilidade e balanceamento de carga local utilizando o Nginx como Load Balancer e 5 nós (containers) replicados, servindo uma aplicação front-end desenvolvida em ReactJS.

##1 - Criando a rede do Docker
Primeiro, criamos uma rede isolada para que todos os containers consigam se comunicar utilizando os seus respectivos nomes como endereço (DNS interno do Docker):
``` bash
docker network create web2-net
```
##2 - Iniciar os 5 nós da aplicação React
A partir da raiz do projeto (onde estão os seus arquivos de configuração e a pasta dist ou build), executamos os comandos para iniciar os 5 nós. 
Eles utilizam uma imagem leve do Nginx (alpine) e mapeiam a aplicação compilada e o arquivo default.conf via volumes do host:

```bash
 docker run -d --name node1 --network web2-net -v "$(pwd)/dist:/usr/share/nginx/html" -v "$(pwd)/default.conf:/etc/nginx/conf.d/default.conf" nginx:alpinecomando: 
 docker run -d --name node2 --network web2-net -v "$(pwd)/dist:/usr/share/nginx/html" -v "$(pwd)/default.conf:/etc/nginx/conf.d/default.conf" nginx:alpinecomando:
 docker run -d --name node3 --network web2-net -v "$(pwd)/dist:/usr/share/nginx/html" -v "$(pwd)/default.conf:/etc/nginx/conf.d/default.conf" nginx:alpinecomando:
 docker run -d --name node4 --network web2-net -v "$(pwd)/dist:/usr/share/nginx/html" -v "$(pwd)/default.conf:/etc/nginx/conf.d/default.conf" nginx:alpinecomando:
 docker run -d --name node5 --network web2-net -v "$(pwd)/dist:/usr/share/nginx/html" -v "$(pwd)/default.conf:/etc/nginx/conf.d/default.conf" nginx:alpinecomando:
```

##3 - Iniciando o Load Balancer

Iniciamos o container que atuará como o balanceador de carga principal. Ele interceptará as requisições na porta 8080 do seu computador e lerá as regras do arquivo nginx.conf:

`docker run -d --name loadbalancer --network web2-net -p 8080:80 -v "$(pwd)/nginx.conf:/etc/nginx/nginx.conf" nginx:alpine`

##Detalhes de Configuração da Infraestrutura 

###Algoritmo de Balanceamento
O Nginx utiliza por padrão o algoritmo Round Robin.
Isso significa que as requisições que chegam na porta 8080 são distribuídas de forma cíclica e igualitária entre os nós (Node 1 -> Node 2 -> Node 3 -> Node 4 -> Node 5 -> Node 1...).

###Preservação e Envio de IPs Reais
Para cumprir o requisito de identificar a origem das requisições dentro dos nós, o arquivo nginx.conf foi configurado para repassar os cabeçalhos (headers) HTTP originais do cliente:

  - proxy_set_header Host: Mantém o host original da requisição.

  - proxy_set_header X-Real-IP: Envia o IP real de quem fez a requisição diretamente para o nó.

  - proxy_set_header X-Forwarded-For: Mantém o histórico de IPs pelos quais a requisição passou.

##Como Testar e Validar?
1- No navegador, acesse o endereço: `"http://localhost:8080`

2- Para verificar o balanceamento acontecendo em tempo real e as requisições alternando entre os nós, execute no terminal o comando de monitoramento de logs:
`docker logs -f loadbalancer`
   
