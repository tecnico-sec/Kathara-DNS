Instituto Superior Técnico, Universidade de Lisboa

**Segurança Informática em Redes e Sistemas**

# Guia de Laboratório - *Kathará DNS*

## Objetivo

O objetivo deste laboratório é criar uma pequena rede com DNS (*Domain Name System*) para  permitir a tradução de nomes de domínio em endereços IP.

Pretende-se também:

- Aprender a gerir interações entre servidores de DNS;
- Aprender o comando `dig` para diagnosticar o protocolo DNS;
- Aprender a configurar DNS em clientes `resolvd` e servidores `BIND`;
- Demonstrar um ataque baseado em DNS.

---

## Exercício 1 -- configuração do DNS

Neste exercício vamos observar uma pequena rede com DNS (*Domain Name System*).
O DNS é um protocolo que permite, a partir de nomes de domínios (*domain names*), obter informação relativa a esses domínios, por exemplo, endereços IPs, nomes de servicos internos (p.ex: *email*), texto arbitrário, etc.

O DNS é um protocolo hierárquico, em que temos domínios de topo, como `.org`. `.com`, `.pt`; e que cada servidor de nomes (*nameserver*) de domínio de topo tem entradas sobre nomes dentro dessa zona (p.ex: `foo.org` tem o endereço IP `1.2.3.4`) ou para *nameservers* responsáveis pelos subdomínios (p.ex: podemos ter um servidor responsável por `*.sirs.org`, nesse caso o nameserver de `*.org` terá uma entrada a dizer quem é o responsável pelo subdomínio `*.sirs.org`).

Neste exercício, temos uma rede pré-configurada, em que temos 2 servidores de DNS dentro da rede do Técnico, cada um é responsável pelos domínios `*.tagus.sirs.org` e `*.alameda.sirs.org`.
Há ainda um servidor central responsável por `*.sirs.org` que informa que cada um desses servidores é responsável por cada subdomínio. 
Temos depois o `pc1` e `pc2`, ligados em localizações diferentes na rede.

1. Desenhe o diagrama da rede com os endereços IP de todas as interfaces de rede e ligações entre elas, com base nos ficheiros `lab.conf` e nos ficheiros `.startup`.
Indique as subredes existentes e qual a subrede de internet.

2. O `pc1`, que se encontra ligado ao `routertagus`, já tem o DNS configurado corretamente. 
Vamos tentar fazer alguns pedidos de DNS e observar o que acontece. 
Para isso corra:

```bash
# No pc1
dig +trace gnu.org
```

Observe os pedidos e resposta de DNS efetuados e indique que tipo de dados consegue encontrar sobre `gnu.org` e em que *nameservers*.

3. O dig executa o protocolo DNS, que pode ser transportado tanto sobre UDP como sobre TCP.
Ao fazer o pedido `dig gnu.org`, são enviados e recebidos pacotes sobre TCP ou UDP?
Qual a porta para a qual são enviados os pacotes pelo `pc1`? Use o comando `tcpdump -n` para os portos aparecerem em formato numérico.

4. Neste lab existem as seguintes zonas de DNS:

+ `*(root)`
+ `*.org`
+ `*.sirs.org`
+ `*.alameda.sirs.org`
+ `*.tagus.sirs.org`  

Para cada zona de DNS, indique qual o servidor responsável por essa zona.

5. Vamos agora fazer um pedido a vários *namespaces*.
Observe qual o comportamento para nomes que não existem:

```bash
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

6. Observe os ficheiros `named.conf` e `db` em cada *nameserver* e compare com a alínea 4. 
As entradas da tabela de DNS têm todas o seguinte formato:

```dns
<name> IN <RECORD_TYPE>  <RECORD_VALUE>  [COMMENTS]
```

Compare com o contéudo das teóricas, mas antes de prosseguir garanta que entende o que significam os record types NS, A e CNAME.

---

# Exercício 2 -- Configurar o DNS de um cliente

1. No `pc1`, observe o ficheiro `/etc/resolv.conf`.

2. Crie um novo pc, `pc2` e ligue-o ao *router* `routeralameda`.
Pode ligar diretamente a uma interface física já existente, desde que fique dentro da rede interna).
Atribua-lhe um endereço IP e configure o DNS para usar como servidor DNS principal o servidor `routeralameda`.
Para configurar o DNS do `pc2`, apenas tem de indicar o servidor de DNS no ficheiro `/etc/resolv.conf`.

3. Confirme que o `dig` funciona no `pc2`.

---

# Exercício 3 -- Adicionar um novo subdomínio

1. Neste exercicio, vamos adicionar alguns novos nomes a servidores de DNS.
Para confirmar que adicionou corretamente os nomes, pode executar o utilitário `dig` e confirmar que obtém uma resposta.

2. Ponha `student.sirs.org` a apontar para o IP do `pc1`.
Que `db` modificou e em que máquina?

3. Ponha `student.tagus.sirs.org.` a apontar para o IP do `pc2` dentro da rede da Alameda.
Que `db` modificou e em que máquina?

4. Vamos agora adicionar uma nova zona `*.external.tagus.sirs.org`.
Vamos colocar esta zona a ser gerida pelo servidor `routeralameda`.
Que modificações deve fazer para que esta tabela fique correta?
Coloque na tabela DNS desta nova zona, uma entrada TXT a apontar de `lab5.external.tagus.sirs.org`, com o valor *"DNS is easy!"*.

5. A partir do `pc1`, faça um pedido `dig` sobre `lab5.external.tagus.sirs.org`.
Confirme que consegue observar o texto inserido anteriormente:

```bash
dig +trace -t TXT lab5.external.tagus.sirs.org
```

---

## Exercício 4 -- *DNS spoofing*

É possível utilizar *ARP spoofing* de modo a fazer um cliente receber respostas de um servidor de DNS errado.
De igual modo, tambem é possivel fazer com que a *cache* de um servidor de DNS recursivo fique com entradas erradas.
Neste exercício queremos observar um ataque simples em que nos ligamos a um cliente, esperamos que este faca um pedido DNS, e enviamos uma resposta errada.

1. O nosso `pc1` é o cliente que queremos atacar.
O nosso ataque consistirá em redirecionar um *ping* para o pc errado.
Ponha este `pc1` a fazer *pings* ao `routeralameda` repetidamente:

```bash
while true; do 
  ping -c 1 dns.alameda.sirs.org;
done
```

2. Usar *tcpdump*/*Wireshark* para observar que, por cada pacote enviado, é feito um pedido DNS pelo IP `pc1.sirs.org`.

3. Observe no *tcpdump*/*Wireshark* os pacotes recebidos e pense como os modificaria para trocar o IP na resposta.

4. Ligue um pc de um atacante diretamente na interface que liga o `pc1` a um *router* e envie uma resposta DNS cujo IP seja alterado para qualquer outro IP.
Poderá utilizar o *nemesis* para enviar um pacote DNS UDP com o contéudo que quiser.

5. Observe que o `pc1` envia um pacote ICMP para o IP errado.

6. Para prevenir o ataque acima, é possível autenticar as respostas DNS com DNSSEC.
Uma resposta autêntica contém uma assinatura digital do contéudo da resposta, que apenas o servidor DNS pode produzir, mas que qualquer cliente pode verificar.
(c.f. instruções para configurar a autenticação DNS no *Debian* com o [BIND][2]).

---

## Exercício 5 - *DNS rebinding*

Num ataque semelhante ao anterior, um atacante que seja um dono de um domínio pode alterar entradas de DNS rapidamente, pondo-as a apontar para outros servidores, de modo a fazer cliente enviar pedidos arbitrários para outros servidores.
Por exemplo, imagine que controla um servidor para obter ficheiros chamado `www.download.com` com um endereço `/giveownership?to=nomedoficheiro`, e que, temporariamente, põe o seu nome de domínio a apontar para `www.evil.com`.
Isso poderá fazer com que um cliente que estivesse a fazer um pedido, o reenvie para `www.evil.com/giveownership?to=nomedoficheiro`.

Em geral, este tipo de ataques é utilizado para contornar proteções dos *browsers*.
Discuta com o seu grupo como faria uma página *web* que estaria em conluio com um domínio DNS para fazer pedidos arbitrários a uma máquina local, sem que o código da página o mostre.

---

## Referências

-   Kathará, [https://github.com/KatharaFramework/Kathara/wiki][1]

-   BIND, [https://wiki.debian.org/Bind9][2]

-   man dig, [https://linux.die.net/man/1/dig][3]

-   man nemesis, [https://github.com/troglobit/nemesis][4]

-   man resolved, [https://manpages.debian.org/stretch/systemd/systemd-resolved.service.8.en.html][5]

  [1]: https://github.com/KatharaFramework/Kathara/wiki
  [2]: https://wiki.debian.org/Bind9
  [3]: https://linux.die.net/man/1/dig
  [4]: https://github.com/troglobit/nemesis
  [5]: https://manpages.debian.org/stretch/systemd/systemd-resolved.service.8.en.html
