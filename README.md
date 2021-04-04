LETI 2020/21 | Segurança Informática em Redes e Sistemas

---

# Guia de Laboratório 5 - Kathara-Firewall

## Objectivo

O objetivo deste laboratório consiste em criar uma pequena rede de DNS e entender como gerir interações entre servidores de DNS.

Aprender a usar o comando dig para diagnosticar o protocolo DNS.

Aprender a configurar DNS em clientes (resolvd) e servidores (BIND).

Mostrar um ataque baseado em DNS.

## Exercício 1 - DNS setup

Neste exericio vamos observar uma pequena rede com DNS (Domain Name System) - um protocolo que permite a traducao entre domain names (nomes de dominios) e informacao relative a esses dominios (iex: enderecos ips, nomes de servicos internos (iex: mail, dns), texto arbitrario). 
O DNS e um protocolo hierarquico, em que temos dominios de topo (iex: *.org, *.com, *.pt), e que cada nameserver de dominio de topo tem entradas para dados sobre nomes dentro dessa zona (iex: foo.org tem o endereco ip 1.2.3.4) ou para nameservers responsaveis pelos subdominios (iex: podemos ter um nameserver responsavel por *.sirs.org, nesse caso o nameserver de *.org tera uma entrada a dizer quem e responsavel pelo subdominio *.sirs.org).

Neste exercicio, temos uma rede preconfigurada, em que temos 2 servidores de DNS dentro da rede do tecnico, cada um e responsavel pelos dominios *.tagus.sirs.org e *.alameda.sirs.org. Ha ainda um servidor central responsavel por *.sirs.org que informa que cada um desses name servers e responsavel por cada subdominio. Temos depois o pc1 e pc2 ligado em localizacoes diferentes na rede. 

1. Desenhe o diagrama da rede com os enderecos ips de todas as interfaces de rede e ligacoes entre elas, com base nos lab.conf e nos ficheiros .startup. Indique as subredes existentes e qual a subrede de internet.

2. O pc1, que se encontra ligado ao routertagus, ja tem o DNS configurado corretamente. Vamos tentar fazer alguns pedidos de DNS e observar o que acontece. Para isso corra:

```
# No pc1
dig +trace gnu.org
```

Observe os pedidos e resposta de DNS efetuados e indique que tipo de dados consegue encontrar sobre gnu.org e em quais nameservers.

3. O dig executa o protocolo DNS, que pode ser transportado tanto sobre UDP como sobre TCP. Ao fazer o pedido dig gnu.org, sao enviados e recebidos pacotes sobre TCP ou UDP? Qual a porta a que sao enviados os pacotes pelo pc1?


4. Neste lab existem as seguintes zonas de DNS:
+ *(root)
+ *.org 
+ *.sirs.org
+ *.alameda.sirs.org
+ *.tagus.sirs.org
Para cada zona de DNS, indique qual o servidor responsavel por essa zona. 

5. Vamos agora fazer um pedido a varios namespaces, observe qual o comportamento para nomes que nao existem:
```
# No pc1
dig www.tagus.sirs.org
dig www.alameda.sirs.org
dig superpc
dig examanswers.sirs.org
dig www.org
dig superpc
dig pc1
dig pc2
```

6. Observe os ficheiros named.conf e db em cada nameserver e compare com a alinea 4. As entradas da tabela de DNS tem todas o seguinte formato:
```
<name> IN <RECORD_TYPE>  <RECORD_VALUE>  [COMMENTS]
```
Compare com o conteudo das teoricas, mas antes de prosseguir garanta que entende o que significam os record types NS, A e CNAME.

# Exercicio 2 - Configurar o DNS de um cliente
1. No pc1, observe o ficheiro /etc/resolv.conf

2. Crie um novo pc, pc2, ligue-o ao router routeralameda (nao importa como o ligar, ate pode ligar diretamente a uma interface fisica ja existente adequada, desde que fique dentro da rede interna), atribua-lhe um endereco ip e configure o DNS para usar como servidor DNS principal o servidor routeralameda. Para configurar o DNS do pc2, apenas tem de indicar o servidor de DNS no ficheiro /etc/resolve.conf .

3. Confirme que o dig continua a funcionar.

# Exercicio 3 - Adicionar um novo subdomain
1. Neste exercicio, vamos adicionar alguns nomes novos a servidores de dns. Para confirmar que adicionou corretamente os nomes, pode executar o utilitario dig e confirmar que obtem uma resposta.

2. Ponha `student.sirs.org` a apontar para o ip do pc1 dentro da rede do tagus. Que db modificou e em que maquina?

3. Ponha `student.tagus.sirs.org.` a apontar para o ip do pc2 dentro da rede da alameda. Que db modificou e em que maquina?

4. Vamos agora adicionar uma nova zona `*.external.tagus.sirs.org`. Vamos colocar esta zona a ser gerida pelo servidor routeralameda. Que modificacoes deve fazer para esta tabela estar correta? Coloque na tabela DNS desta nova zona uma entrada TXT a apontar de lab5.external.tagus.sirs.org, com o valor "DNS e facil!".

5. A partir do pc1, faca um pedido dig sobre lab5.external.tagus.sirs.org e observe que consegue observar o texto pretendido:
```
dig +trace -t TXT lab5.external.tagus.sirs.org
```
 
## Exercício 4 - DNS spoofing

E possivel utilizar ARP spoofing de modo a fazer um cliente a receber respostas erradas de um servidor de DNS. De igual modo, tambem e possivel fazer com que a cache de um servidor de DNS recursivo fique com entradas erradas. Neste exercicio queremos observar um ataque simples em que nos ligamos a um cliente, esperamos que este faca um pedido DNS, e enviamos uma resposta errada.


1. O nosso pc1 e o cliente que queremos atacar. O nosso ataque consistira em redirecionar um ping para o pc errado. Ponha este pc a fazer pings ao routeralameda repetidamente:
```
while true; do 
  ping -c 1 dnsalameda.sirs.org;
done
```

2. Podera no wireshark observar que por cada pacote enviado e feito um pedido DNS pelo ip dnsalameda.sirs.org.

3. Observe no tcpdump/wireshark os pacotes recebidos e pense como os modificaria para trocar o ip na resposta.

4. Ligue um pc de um atacante diretamente na interface que liga o pc1 a um router e envie uma resposta dns cujo ip seja alterado para qualquer outro ip. Podera utilizar o nemesis para enviar um pacote DNS udp com o conteudo que quiser.

5. Observe que o pc1 envia um pacote ICMP para o ip errado.

6. Para prevenir o ataque acima, existe DNS autenticado - DNSSEC, que corresponde a nas respostas de DNS enviar uma assinatura do conteudo da resposta, que apenas o servidor DNS pode produzir, mas que qualquer cliente pode verificar. Pode observar na configuracao do debian como configurar dns autenticado com o [BIND][2].

## Exercício 5 - DNS rebinding

Num ataque semelhante ao anterior, um atacante que seja um dono de um dominio pode alterar no dns server entradas de DNS rapidamente, pondo-as a apontar para outros servidores, de modo a fazer cliente enviarem pedidos arbitrarios para outros servidores.

Por exemplo, imagine que controla um servidor para fazer downloads de ficheiros chamado www.download.com com um endpoint /giveownership?to=nomedoficheiro, e que temporariamente poe o seu domain name a apontar para www.banco.com. Isso podera fazer com que um cliente que estivesse a fazer um pedido o reenvie para www.banco.com/giveownership?to=nomedoficheiro .

Este tipo de ataques no geral e utilizado para contornar proteccoes de browsers. Discuta com o seu grupo como faria uma pagina web que estaria coordenada com um domain DNS para fazer pedidos arbitrarios a uma maquina local sem que o codigo da pagina o mostre.

## Referências

-   Kathara, [https://github.com/KatharaFramework/Kathara/wiki][1]

-   BIND, [https://wiki.debian.org/Bind9][2]

-   man dig, [https://linux.die.net/man/1/dig][3]

-   man nemesis, [https://github.com/troglobit/nemesis][4]

-   man resolved, [https://manpages.debian.org/stretch/systemd/systemd-resolved.service.8.en.html][5]

  [1]: https://github.com/KatharaFramework/Kathara/wiki
  [2]: https://wiki.debian.org/Bind9
  [3]: https://linux.die.net/man/1/dig
  [4]: https://github.com/troglobit/nemesis
  [5]: https://manpages.debian.org/stretch/systemd/systemd-resolved.service.8.en.html