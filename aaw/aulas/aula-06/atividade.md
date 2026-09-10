Atividade - Aula 06

Síncrono ou Assíncrono?

Nomes: Luiz Fernando de Oliveira Ribeiro e Kaua Spina
Turma: CC, ADS
Data: 10/09/2026

Cenário 01 - PagFácil

1. Estilo recomendado: Síncrono.

2. Fluxo:

[Cliente] -- pagar --> [Pagamentos] -- consultar saldo/limite --> [Contas]

[Contas] -- resposta --> [Pagamentos] -- resultado da compra --> [Cliente]

3. Justificativa:

Eu usaria a comunicação síncrona porque o cliente precisa da resposta na hora e o pagamento depende da consulta ao saldo ou limite.

4. Principal risco:

Se Contas ficar lento ou fora do ar, a compra não pode ser concluída. Eu colocaria um tempo limite, sem aprovar automaticamente.

Cenário 02 - CadastraJá

1. Estilo recomendado: Assíncrono, usando uma fila.

2. Fluxo:

[Usuario] -- cadastrar --> [Cadastro]
[Cadastro] -- conta criada --> [Usuario]

[Cadastro] -- pedido de envio --> [Fila de e-mails]
                                         |
                                         v
                                [Servico de e-mail]
                                         |
                                         v
                                [Provedor de e-mail]

O cadastro salva a conta e o pedido de envio juntos, sem esperar o e-mail para responder ao usuário.

3. Justificativa:

Eu usaria uma fila porque o e-mail pode chegar depois e uma falha no provedor não deve atrasar o cadastro. Se o envio falhar, o serviço tenta novamente.

4. Principal risco:

Os e-mails podem se acumular na fila se o provedor ficar indisponível por muito tempo.

Cenário 03 - MegaMarket

1. Estilo recomendado: Assíncrono, com uma fila que mantenha as mensagens salvas.

2. Fluxo:

[Checkout] -- registrar venda --> [Vendas]
                                     |
                                     v
                      [Banco: venda + mensagem pendente]
                                     |
                                     v
                         [Publicador de mensagens]
                                     |
                                     v
                               [Fila duravel]
                                     |
                                     v
                                 [Estoque]

[Estoque] -- confirma depois de salvar a baixa --> [Fila duravel]

Eu salvaria a venda e a mensagem pendente na mesma transação. O envio seria repetido em caso de falha, e o Estoque só confirmaria depois de salvar a baixa.

3. Justificativa:

Eu usaria uma fila para absorver os picos sem travar o checkout, já que alguns segundos de atraso na baixa são aceitáveis.

4. Principal risco:

O estoque pode ficar desatualizado e permitir vendas sem produto. Seria necessário reservar as unidades antes da confirmação definitiva.

Cenário 04 - AppBanco

1. Estilo recomendado: API Gateway/BFF, com um BFF para mobile e outro para web.

2. Fluxo:

[App mobile] --> [BFF mobile] --+
                              |
[App web] ----> [BFF web] -----+
                              |
                              +--> [Saldo]
                              +--> [Cartao]
                              +--> [Investimentos]
                              +--> [Emprestimos]
                              +--> [Cashback]

[Servicos] -- dados --> [BFF correspondente] -- resposta unica --> [App]

As consultas aos serviços seriam feitas em paralelo.

3. Justificativa:

Eu usaria um BFF para reunir os dados em uma resposta, reduzindo as chamadas do celular e adaptando os formatos e a autenticação. Separaria mobile e web para cada versão receber só o que precisa.

4. Principal risco:

O BFF pode virar um ponto de falha, e um serviço lento pode atrasar a tela.

Desafio - A mesma mensagem chegando duas vezes

Cenário escolhido: 03 - MegaMarket.

Uma mensagem repetida poderia descontar o estoque duas vezes pela mesma venda.

Eu usaria um identificador único por evento e registraria os já processados no banco, sem permitir IDs repetidos. Esse registro e a baixa seriam salvos na mesma transação. Se o evento já estivesse registrado, eu confirmaria a mensagem sem descontar novamente. Isso é idempotência
