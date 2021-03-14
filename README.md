LETI 2020/21 | Segurança Informática em Redes e Sistemas

---

# Guia de Laboratório 5 - Kathara-Firewall

## Objectivo

O objetivo deste laboratório consiste em criar uma pequena rede de dns e entender como gerir
interações com um servidor de dns externo.

## Exercício 1 - DNS setup

Mostrar um SERVIDOR DE DNS usado para dar um nome ao webserver e ao sqlserver.

## Exercício 2 - DNS request poisoning via ARP spoofing

Mostrar um atacante a pretender ser um servidor de dns via arp spoofing para enviar paginas web falsas.

Alternativamente podemos mostrar um atacante a usar dns spoofing para fazer o servidor usar uma base de dados errada / obter a sha1 da password do mariadb.

## Exercício 3 - DNS rebinding
Isto e um bocado mais robuscado, mas e um ataque mais realista. 
Consistente em registar um nome com TTL pequeno e deixar o browser fazer pedidos ao proprio hostname, mas entretanto registar um endereço novo para o próprio domínio que aponte para o localhost e ir trocando entre os 2 pare receber e enviar pacotes.
(iex: 
dns www.example.com -> 5.5.5.5 -> script.html
timeout
dns www.example.com -> 10.0.0.1
script.html faz uma query a www.example.com:3306 que e redirecionada para a rede local e funciona porque o clietne tem permissoes.
dns www.example.com -> 5.5.5.5
script.html envia o resultado de volta para 5.5.5.5
)
Não dá para se mostrar sem ter um browser que corra js, mas dá para se entender a ideia.

## Exercício 4 - DNSSEC

DNS + Authenticação do servidor

## Exercício 5 - PIR 

DNS + Impedir o servidor de saber que páginas são pedidas

## Referências

-   Kathara, [https://github.com/KatharaFramework/Kathara/wiki][1]

  [1]: https://github.com/KatharaFramework/Kathara/wiki
