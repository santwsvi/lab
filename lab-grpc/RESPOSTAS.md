# RESPOSTAS - Roteiro 3 (gRPC)

## Parte A - Transparências em Sistemas Distribuídos

### 4.1 - Reflexão sobre o laboratório anterior (TCP/UDP/Multicast/WebSocket)

1. Prejudica. Motivos: atrapalha a fácil visualização por operadores do sistema, aumenta a exposição do endereço para terceiros em caso de vazamento do código fonte, troca posterior exigiria recompilação do sistema.

2. Ausência de transparência, já que torna o resultado da string suscetível a falhas de sintaxe, aumenta a probabilidade de falta de padrão e causa acoplamento rígido ao prender a lógica de montagem da mensagem/URL ao código fonte.

3. O cliente apontaria para um recurso inexistente ou simplesmente errado. Nenhuma das quatro alternativas escaparia disso, já que todas são baseadas em código hardcoded, sem utilizar uma lógica de escolha por configurações via variáveis de ambiente.

### 4.3 - Perguntas

1. A transparência de acesso, já que o programa desenvolvido pelo mesmo acessa esses recursos de um servidor remoto via requisições com um formato e interfaces claras, seja com base nos padrões REST ou gRPC, por exemplo.

2. Não. Chamadas remotas possuem particularidades que as locais não possuem, como latência de rede elevada e por muitas vezes com picos imprevisíveis. Mascarar essa diferença pode levar a problemas de rastreabilidade em caso de tratamento de erros.

3. (Respondida após concluir C e D) O ClienteTCP do roteiro anterior exige "pensar em rede": abrir o socket, montar a mensagem como string, chamar send/receive e fazer parsing manual da resposta. Já o cliente gRPC permite "pensar no problema": basta montar um objeto PerguntaHorario e chamar stub.consultarHorario(...), como se fosse um método comum - toda a parte de rede (serialização, conexão, envio, desserialização) fica escondida atrás do stub gerado. Isso se relaciona diretamente com a transparência de acesso, já que a forma de acessar o recurso remoto passa a "parecer" uma chamada local.

## Parte B - Protocol Buffers e o contrato do serviço

### 5.5 - Perguntas

1. Evita erros humanos, documenta a API e garante que cliente e servidor usem o mesmo formato.

2. Indica que o combo gRPC + Protocol Buffers é agnóstico a linguagens de programação, possibilitando, por exemplo, colaboração menos restritiva entre profissionais especializados em diferentes tecnologias.

3. A classe gerada automaticamente CentralAtendimentoGrpc.java pertence ao serviço central.CentralAtendimento, sendo o método getConsultarHorarioMethod responsável pela funcionalidade de ConsultarHorario.

## Parte C - RPC unário: ConsultarHorario

### 6.6 - Perguntas

1. Em alto nível: (a) o cliente serializa a mensagem PerguntaHorario em bytes (protobuf) e monta um frame HTTP/2; (b) esses bytes trafegam pela rede até o servidor, numa conexão HTTP/2 já estabelecida com o canal; (c) no servidor, o runtime do gRPC desserializa os bytes de volta em um objeto PerguntaHorario e invoca o método correspondente (consultarHorario/ConsultarHorario) - a resposta faz o caminho inverso até o stub devolver o objeto RespostaHorario para quem chamou.

2. No ClienteTCP do roteiro anterior, "montar a mensagem" era feito manualmente pelo próprio código do cliente (concatenar/formatar uma string e converter para bytes com getBytes()), e "interpretar a resposta" era o servidor fazendo parsing dessa string à mão. No gRPC, esse trabalho todo (serialização, desserialização, definição do formato) é feito pelo código gerado a partir do central.proto - o programador só monta o objeto da mensagem (PerguntaHorario) e lê os campos da resposta (RespostaHorario), sem escrever nenhuma lógica de parsing.

3. Com o servidor desligado, a chamada stub.consultarHorario(pergunta) não trava silenciosamente nem retorna null: ela lança uma exceção (StatusRuntimeException em Java, grpc.RpcError em Python) com um status como UNAVAILABLE, indicando que não foi possível estabelecer conexão com o servidor. Isso é bem diferente do TCP "na mão", onde normalmente é preciso tratar uma ConnectException/ConnectionRefusedError no próprio código de sockets - aqui o gRPC já entrega um erro padronizado e tipado.

## Parte D - RPC com streaming de servidor: AcompanharAvisos

### 7.6 - Perguntas

1. Seria preciso guardar, no servidor, a lista de StreamObservers/contexts de todos os clientes inscritos (por exemplo, numa lista thread-safe) e, ao gerar cada Aviso, fazer um laço chamando onNext() (Java) ou fazendo yield (Python, dentro de uma estrutura que envie a todos) para cada um dos observadores/streams abertos, em vez de responder só ao Iterator/generator de uma única chamada. Ou seja, o servidor precisaria administrar múltiplas conexões simultâneas e replicar manualmente o envio para cada uma - o gRPC não faz isso "de graça" para múltiplos clientes num único stream, diferente do que o Multicast fazia com um único envio para o grupo.

2. As duas abordagens chegam ao mesmo resultado, mas a versão em Python com yield pareceu mais natural: o método simplesmente "pausa" e "retoma" a cada aviso, como uma função geradora comum, sem precisar manipular explicitamente um objeto observer. Em Java, o uso de StreamObserver e chamadas explícitas a onNext()/onCompleted() é mais verboso e exige entender o padrão Observer, mas deixa mais visível o ciclo de vida do stream (quando ele começa, quando cada item é enviado e quando termina).

3. Se o cliente fechar a conexão no meio do envio dos avisos, o servidor detecta isso na próxima tentativa de onNext() (Java) ou no próximo yield (Python): a chamada lança uma exceção (por exemplo, StatusRuntimeException com CANCELLED, ou io.grpc.StatusRuntimeException/grpc.RpcError dependendo da linguagem), e o laço é interrompido - no código Java isso cai no catch (InterruptedException e) se o cancelamento ocorrer durante o Thread.sleep, e em Python o generator simplesmente para de ser iterado. O servidor não trava nem continua enviando avisos para um cliente que não existe mais.
