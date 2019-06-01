# BOLT #0: Introdução e Índice

Bem vindos, **amigos**! Os documentos de Bases da Tecnologia Lightning (BOLT)
descrevem um protocolo de segunda camada para a transferência *off-chain* de
bitcoin através de cooperação mútua, dependente de transações *on-chain* para
o cumprimento das regras se necessário.

O objetivo é simplificar os termos e propriedades que fazem parte dos protocolos
correspondentes à Lightning Network, auxiliando como referência para a criação de
aplicações que utilizam a Lightning como base, ou mesmo para uma nova implementação.

OBS: alguns termos não possuem tradução literal para o português, ou mesmo não podem
ser modificados para melhor entendimento do protocolo, portanto, estas expressões
devem ser mantidas no idioma original, mas com a devida explicação sobre o sentido
semântico da expressão em questão e sua aplicação no protocolo.

Esta é a versão 0 do protocolo geral, conforme definida no repositório original.

1. [BOLT #1](01-messaging.md): Protocolo Base
2. [BOLT #2](02-peer-protocol.md): Protocolo de Peers para Gerenciamento de Canais
3. [BOLT #3](03-transactions.md): Transações de Bitcoin e formatos de Script
4. [BOLT #4](04-onion-routing.md): Protocolo de Roteamento em Camadas (Onion)
5. [BOLT #5](05-onchain.md): Recomendações para a Manipulação de Transações On-chain
7. [BOLT #7](07-routing-gossip.md): Protocolo P2P e Descoberta de Canais
8. [BOLT #8](08-transport.md): Transporte Criptografado e Autenticado
9. [BOLT #9](09-features.md): Flags de Propriedades
10. [BOLT #10](10-dns-bootstrap.md): DNS Bootstrap e Localização de Nodes Assistida
11. [BOLT #11](11-payment-encoding.md): Protocolo de Cobrança para Pagamentos Lightning

## The Spark: Uma pequena introdução à Lightning

A Lightning é um protocolo para a realização de pagamentos rápidos com Bitcoin,
por meio do uso de uma rede de canais.

### Canais

A Lightning funciona através do estabelecimento de *canais*: dois participantes
criam um canal de pagamentos Lightning que contém alguma quantia de bitcoin
(por exemplo: 0.1 bitcoin) no qual realizam uma "trava" na rede do Bitcoin. Esta
quantia só pode ser gasta com ambas as assinaturas dos participantes.

Inicialmente, ambos detêm uma transação de bitcoin (UTXO) que envia todos os
fundos de volta para as partes. As partes podem posteriormente assinar uma nova
transação que divide os fundos de maneira diferente. Por exemplo: 0.09 bitcoin
para uma parte e 0.01 bitcoin para a outra, invalidando a transação anterior, para
que esta não seja gasta.

Veja a [BOLT #2: Abertura de Canal](02-peer-protocol.md#channel-establishment) para
saber mais sobre a abertura de canais, e a [BOLT #3: Output de Transação de Depósito](03-transactions.md#funding-transaction-output) para ver o formato da transação de criação do canal. Veja a [BOLT #5: Recomendações para a manipulação de Transações On-chain](05-onchain.md) para os requisitos de quando os participantes discordam ou falham em abrir o canal, e a transação
co-assinada deve ser gasta.

### Pagamentos Condicionais

Um canal Lightning permite apenas pagamentos entre dois participantes, mas os canais
podem ser conectados entre si para formar uma rede que permite pagamentos entre todos
os membros desta. Isto requer a tecnologia de pagamentos condicionais, que pode ser
adicionada a um canal. Por exemplo:

> "Você receberá 0.01 bitcoin se revelar o 'segredo' dentro de 6 horas"

Assim que o destinatário apresentar o "segredo", esta transação de bitcoin é
substituída por uma onde não há o pagamento condicional, e adicionará os fundos
a um _output_ do destinatário.

Veja a [BOLT #2: Adicionando um HTLC](02-peer-protocol.md#adding-an-htlc-update_add_htlc), que especifica os
comandos utilizados por um participante para acrescentar um pagamento condicional,
e a [BOLT #3: Transação de Execução](03-transactions.md#commitment-transaction) para o formato completo da
transação.

### Redirecionamento

Tal pagamento condicional pode ser redirecionado de maneira segura para outro participante
com um limite de tempo menor para a resolução da condição. Por exemplo:

> "Você receberá 0.01 bitcoin se revelar o 'segredo' dentro de 5 horas"

Isto permite que os canais sejam conectados numa rede sem a necessidade de
confiar nos intermediários.

Veja a [BOLT #2: Redirecionando HTLCs](02-peer-protocol.md#forwarding-htlcs) para os detalhes do redirecionamento de pagamentos, e a [BOLT #4: Estrutura do Pacote](04-onion-routing.md#packet-structure) para saber como as instruções de pagamentos são transportadas e roteadas pela rede.

### Topologia da Rede

Para realizar um pagamento, um participante precisa saber por quais canais poderá
enviá-lo. Os participantes comunicam uns aos outros sobre a criação de canais e *nodes*,
bem como as atualizações de estado dos canais. Isto serve para que um *peer* tenha
informações sobre canais intermediários entre o *node de origem* e o *node final*, bem
como a capacidade destes canais intermediários.

Verifique a [BOLT #7: P2P Node and Channel Discovery](07-routing-gossip.md)
para obter detalhes sobre o protocolo de comunicação, e a [BOLT #10: DNS Bootstrap and Assisted Node Location](10-dns-bootstrap.md) para informações sobre o *bootstrap* inicial da rede.

### Cobranças de Pagamento

Um participante da rede recebe cobranças que indicam quais pagamentos devem ser
feitos. Diferentemente do Bitcoin, a Lightning Network utiliza o modelo de cobranças
ao invés de simples endereços. Estas cobranças possuem as propriedades e informações de
um pagamento a ser efetuado.

Veja a [BOLT #11: Protocolo de Cobrança para Pagamentos Lightning](11-payment-encoding.md) para
ver o protocolo que descreve o modelo de cobrança, que inclui o destinatário e a finalidade de um
pagamento, de tal modo que o pagador possa posteriormente provar a execução bem-sucedida do pagamento.


## Glossário e Termos

* *Announcement*:
   * Uma mensagem enviada entre *peers* com o objetivo de auxiliar o descobrimento de novos *canais*
   e/ou *nodes*.

* `chain_hash`:
   * O hash identificador único da blockchain-alvo (geralmente o hash do bloco genesis).
     Isto permite aos *nodes* criarem e referenciarem **canais** em diferentes blockchains.
     Os nodes devem ignorar quaisquer mensages que referenciam um `chain_hash` desconhecido.
     Diferentemente da `bitcoin-cli`, o hash não é invertido, mas usado diretamente.

     Para a cadeia principal (main chain) da blockchain do Bitcoin, o valor do `chain_hash`
     deve ser (codificado em hexadecimal):
     `6fe28c0ab6f1b372c1a6a246ae63f74f931e8365e15a089c68d6190000000000`.

* *Canal*:
   * Um método off-chain rápido de troca mútua entre dois *peers*.
   Para transacionar os fundos, os peers realizam uma troca de assinaturas para
   criar uma *transação de execução (commitment transaction)*.
   * _Veja os métodos de fechamento de canais: fechamento mútuo (mutual close),
   fechamento por transação de revogação (revoked transaction close), fechamento unilateral (unilateral close)_
   * _Relacionado: rota_

* *Transação de fechamento (closing transaction)*:
   * Uma transação gerada como parte de um _fechamento mútuo (mutual close)_.
   Uma transação de fechamento é similar a uma _transação de execução (commitment transaction)_,
   mas sem pagamentos pendentes, e sem a presença de um *HTLC*.
   * _Relacionado: transação de execução (commitment transaction),
   transação de depósito (funding transaction), transação de penalidade (penalty transaction)_

* *Número de Commitment (Commitment number)*:
   * Um "contador" de 48 bits incrementado a cada *transação de execução (commitment transaction)*;
   contadores são únicos para cada *peer* no *canal* e começam em 0 (zero).
   * _Pertencente a: transação de execução (commitment transaction)_
   * _Relacionado: transação de fechamento (closing transaction),
   transação de depósito (funding transaction), transação de penalidade (penalty transaction)_

* *Chave Privada de Revogação*:
   * Toda *transação de execução (commitment transaction)* possui uma chave privada
   de revogação única, que permite ao outro *peer* gastar todos os *outputs* imediatamente.
   Revelar esta chave privada é a forma de revogar uma *transação de execução (commitment transaction)* 
   propagada unilateralmente. Para auxiliar na revogação,
   cada *output* da *transação de execução* referencia uma *chave pública de revogação*.

   * _Pertencente a: transação de execução (commitment transaction)_
   * _Origem: per-commitment secret_

* *Transação de Execução (Commitment transaction)*:
   * Uma transação que tem uma *transação de depósito (funding transaction)* como
   input.
   Cada *peer* do canal armazena a assinatura de sua contraparte relativa a esta transação,
   assim ambos possuem sempre uma *transação de execução* a qual
   podem gastar, seja para evitar trapaça da contraparte, ou para fechar unilateralmente caso
   sua contraparte não responda mais às mensagens enviadas ao seu *node*.
   Quando os *peers* realizam uma transação entre si por meio de um canal, uma nova
   *transação de execução* é gerada, e a antiga é *revogada*.
   * _Componentes: número de commitment (commitment number), chave privada de revogação,
   HTLC, per-commitment secret, outpoint_
   * _Relacionado: transação de fechamento (closing transaction), transação de depósito
   (funding transaction), transação de penalidade (penalty transaction)_
   * _Tipos de transação de execução: transação de execução revogada (revoked commitment transaction)_

* *Node final*:
   * O destinatário final de um pacote que roteia um pagamento de um _node de origem_ através de um certo número de _hops_.
   Este é também o último *peer recebedor* numa sequência de pagamentos.
   * _Categoria: node_
   * _Relacionado: node de origem, node processador_

* *Transação de depósito (funding transaction)*:
   * Uma transação *on-chain* irreversível, que paga ambos os *peers* envolvidos num *canal*.
   Só pode ser gasta por meio de consenso mútuo.
   * _Relacionado: transação de fechamento (closing transaction), transação de execução (commitment transaction),
   transação de penalidade (penalty transaction)_

* *Hop*:
   * Um *node*. Geralmente, um *node* intermediário posicionado entre um *node de origem* e um *node final*.
   Neste caso, é utilizado como *node* de roteamento de uma transação por meio de seus canais em comum.
   * _Categoria: node_

* *HTLC*: Hashed Time Locked Contract.
   * Um contrato de pagamento condicional entre dois *peers*: o destinatário pode gastar o
   pagamento informando uma assinatura da transação e uma *preimagem de pagamento*,
   caso contrário o remetente do pagamento poderá cancelar este contrato de pagamento condicional
   e resgatar os fundos após um determinado tempo.
   Os HTLCs são originados dos *outputs* de uma *transação de execução (commitment transaction)*.
   * _Pertencente a: transação de execução_
   * _Componentes: Hash de Pagamento, Preimagem de Pagamento_

* *Cobrança*: Uma solicitação de pagamento na Lightning Network. Pode incluir
    informações sobre o pagamento em si, como: tipo de pagamento, quantia, data de expiração, etc.
    Pagamentos na Lightning Network não são efetuados através de endereços, como
    na rede do Bitcoin, mas sim em forma de cobranças.

* *It's ok to be odd*:
   * Regra aplicada a alguns campos numéricos, que indicam tanto suporte opcional
     ou obrigatório de alguma propriedade do protocolo. Números pares (*even*) indicam
     que ambos os endpoints (*nodes* participantes de um canal), DEVEM suportar uma
     propriedade em questão, enquanto que os números ímpares (*odd*) indicam que
     determinada propriedade PODE ser ignorada pelo outro endpoint.

* *MSAT*:
   * Millisatoshi. É uma métrica de contabilidade interna equivalente a 0.001 satoshi,
   ou 0.0000000001 bitcoin.
   Utilizada para definir o valor de um pagamento numa cobrança, por exemplo.

* *Fechamento Mútuo*:
   * Fechamento cooperativo de um canal, efetuado por meio da transmissão do gasto de
   um pagamento não condicional da *transação de depósito (funding transaction)*, com um
   *output* para cada *peer* do canal, a menos que um dos *outputs* seja pequeno demais,
   logo, deve ser ignorado, ou mesmo quando um dos *peers* do canal recebeu todos os fundos durante
   as transações intermediárias daquele canal.
   * _Relacionado: Fechamento por Transação Revogada (revoked transaction close), Fechamento Unilateral_

* *Node*:
   * Um computador ou qualquer outro dispositivo que faça parte da Lightning Network.
   * _Relacionado: peers_
   * _Tipos de node: node final, hop, node de origem, node de processamento, node de recebimento,
   node de envio_

* *Node de origem*:
   * O _node_ que origina um pacote que roteia um pagamento através de alguns _hops_ para um
   _node final_. É também o primeiro _peer de envio (sending peer)_ em uma sequência de _peers_
   de um pagamento roteado entre canais.
   * _Categoria: node_
   * _Relacionado: node final, node de processamento_

* *Outpoint*:
  * Um hash de transação e índice de saída que identifica uma saída de transação não gasta (UTXO).
  É necessário para a composição de uma nova transação, sendo utilizado como *input*.
  * _Relacionado: transação de depósito (funding transaction), transação de execução (commitment transaction)_

* *Hash de Pagamento (Payment Hash)*:
   * Um *HTLC* possui o chamado **hash de pagamento**, que é o hash de uma *preimagem*, hash este
   que condiciona o pagamento. A *preimagem* é utilizada como "segredo" a ser revelado para realizar
   o desbloqueio deste pagamento condicional.
   * _Pertencente a: HTLC_
   * _Origem: Preimagem de Pagamento (Payment Preimage)_

* *Preimagem de Pagamento (Payment Preimage)*:
   * Comprovação do recebimento do pagamento, mantida pelo destinatário final do pagamento,
   sendo este o único conhecedor deste "segredo". O destinatário final revela a *preimagem*
   para realizar a liberação dos fundos. Esta *preimagem* passa por uma função de hash, de
   modo que resulta no *Hash de Pagamento* do *HTLC*.
   * _Pertencente a: HTLC_
   * _Deriva: Hash de Pagamento (Payment Hash)_

* *Peers*:
   * Dois *nodes* que comunicam entre si.
      * Dois *peers* podem comunicar entre si antes de estabelecer um canal.
      * Dois *peers* podem estabelecer um *canal*, pelo qual podem transacionar.
   * _Relacionado: node_

* *Transação de Penalidade (Penalty Transaction)*:
   * Uma transação que gasta todos os *outputs* de uma transação de revogação,
   usando a chave privada de execução de revogação. Um *peer* utiliza este tipo
   de transação caso o outro *peer* de um canal tente trapacear por meio da
   propagação de uma *transação de execução revogada (revoked commitment transaction)*,
   utilizando a *chave privada de revogação* para obter para si os fundos.
   * _Relacionado: transação de fechamento (closing transaction), transação de execução
   (commitment transaction), transação de depósito (funding transaction)_

* *Per-commitment secret*:
   * Every *commitment transaction* derives its keys from a per-commitment secret,
     which is generated such that the series of per-commitment secrets
     for all previous commitments can be stored compactly.
   * _See container: commitment transaction_
   * _See derivation: commitment revocation private key_

* *Processing node*:
   * A *node* that is processing a packet that originated with an *origin node* and that is being sent toward a *final node* in order to route a payment. It acts as a _receiving peer_ to receive the message, then a _sending peer_ to send on the packet.
   * _See category: node_
   * _See related: final node, origin node_

* *Node recebedor*:
   * Um *node* que está recebendo uma mensagem.
   * _Categoria: node_
   * _Relacionado: node emissor_

* *Peer recebedor*:
   * O *node* de destino de uma mensagem enviada por um *peer* ao qual está conectado.
   * _Categoria: peer_
   * _Relacionado: peer emissor_

* *Transação de Execução Revogada (Revoked commitment transaction)*:
   * Uma *transação de execução (commitment transaction)* que fora revogada após a criação 
   de uma nova *transação de execução* depois de uma nova negociação de mudança de estado 
   (transferência de valores) entre *peers* de um canal.
   * _Categoria: transação de execução (commitment transaction)_

* *Fechamento por Transação Revogada (Revoked transaction close)*:
   * Fechamento inválido de um *canal*, efetuado por meio da transmissão de uma 
   *transação de execução revogada (revoked commitment transaction)*. Tendo em vista
   que a contraparte do canal fechado por uma *transação de execução revogada* possui
   a *chave privada de revogação* para anular esta transação, este pode então usar tal 
   chave privada para aplicar uma penalidade através da *transação de penalidade (penalty transaction)*.
   * _Relacionado: Fechamento Mútuo, Fechamento Unilateral_

* *Rota*: Caminho que um pagamento percorre através da Lightning Network 
que permite a transferência de valores de *node de origem* a um *node final* 
através de um ou mais *hops* intermediários.
  * _Relacionado: canal_

* *Node emissor*:
   * O *node* que envia uma mensagem.
   * _Categoria: node_
   * _Relacionado: node recebedor_

* *Peer emissor*:
   * O *peer* que envia uma mensagem.
   * _Categoria: peer_
   * _Relacionado: peer recebedor_.

* *Fechamento Unilateral*:
   * Um fechamento não cooperativo de um *canal*, efetuado por meio da transmissão
   de uma *transação de execução (commitment transaction)*. Esta transação é maior
   e menos eficiente que uma *transação de fechamento (closing transaction)*, e o
   *peer* que realiza a transmissão de sua *transação de execução (commitment transaction)*
   não pode gastar os *outputs* de seus fundos até que se passe o tempo acordado
   previamente no contrato (*HTLC*).
   * _Relacionado: Fechamento Mútuo, revoked transaction close_

## Música tema

      Why this network could be democratic...
      Numismatic...
      Cryptographic!
      Why it could be released Lightning!
      (Release Lightning!)


      We'll have some timelocked contracts with hashed pubkeys, oh yeah.
      (Keep talking, whoa keep talkin')
      We'll segregate the witness for trustless starts, oh yeah.
      (I'll get the money, I've got to get the money)
      With dynamic onion routes, they'll be shakin' in their boots;
      You know that's just the truth, we'll be scaling through the roof.
      Release Lightning!
      (Go, go, go, go; go, go, go, go, go, go)


      [Chorus:]
      Oh released Lightning, it's better than a debit card..
      (Release Lightning, go release Lightning!)
      With released Lightning, micropayments just ain't hard...
      (Release Lightning, go release Lightning!)
      Then kaboom: we'll hit the moon -- release Lightning!
      (Go, go, go, go; go, go, go, go, go, go)


      We'll have QR codes, and smartphone apps, oh yeah.
      (Ooo ooo ooo ooo ooo ooo ooo)
      P2P messaging, and passive incomes, oh yeah.
      (Ooo ooo ooo ooo ooo ooo ooo)
      Outsourced closure watch, gives me feelings in my crotch.
      You'll know it's not a brag when the repo gets a tag:
      Released Lightning.


      [Chorus]
      [Instrumental, ~1m10s]
      [Chorus]
      (Lightning! Lightning! Lightning! Lightning!
       Lightning! Lightning! Lightning! Lightning!)


      C'mon guys, let's get to work!


   -- Anthony Towns <aj@erisian.com.au>

## Authors

[ FIXME: Insert Author List ]

![Creative Commons License](https://i.creativecommons.org/l/by/4.0/88x31.png "License CC-BY")
<br>
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).
