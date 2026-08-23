4.1 - 1. Prejudica. Motivos: atrapalha a fácil visualização por operadores do sistema, aumenta a exposição do endereço para terceiros em caso de vazamento do código fonte, troca posterior exigiria recompilação do sistema.

2. Ausencia de transparencia já que torna o resultado da string suscetível a falhas de sintaxe, aumenta a probabilidade de falta de padrão e causa acoplamento rigído ao prender a lógica de montagem da url ao codigo fonte.

3. O cliente apontaria para um recurso inexistente ou simplesmente errado. Nenhuma das quatro alternativas escaparia disso, já que todas são baseadas em código hardcoded, sem utilizar uma lógica de escolha por configurações via variáveis de ambiente.

4.3 - 1. A transparência de acesso, já que o programa desenvolvido pelo mesmo acessar esses recursos de um servidor remoto via requisições com um formato e interfaces claras, seja com base nos padrões REST ou gRPC, por exemplo.

2. Não. Chamdas remotas possuem particularidades que as locais não possuem, como latência de rede elevada e por muitas vezes com picos imprevisíveis. Mascarar essa diferença pode levar a problemas de rastreabilidade em caso de tratamento de erros.

5.5 - 1. Evita erros humanos, documenta a API e garante que cliente e servidor usem o mesmo formato

2. Indica que o combo gRPC + protocol buffers é agnóstico a linguagens de programação, possibilitando, por exemplo, colaboração menos restritiva entre profissionais especializados em diferentes tecnologias.

3. A classe gerada automaticamente CentralAtendimentoGrpc.java pertence ao serviço central.CentralAtendimento, sendo o método getConsultarHorarioMethod responsável pela funcionalidade de ConsultarHorario.

6.6 - 1. Em alto nível: (a) o cliente serializa a mensagem PerguntaHorario em bytes (protobuf) e monta um frame HTTP/2; (b) esses bytes trafegam pela rede até o servidor, numa conexão HTTP/2 já estabelecida com o canal; (c) no servidor, o runtime do gRPC desserializa os bytes de volta em um objeto PerguntaHorario e invoca o método correspondente (consultarHorario/ConsultarHorario) - a resposta faz o caminho inverso até o stub devolver o objeto RespostaHorario para quem chamou.

2. No ClienteTCP do roteiro anterior, "montar a mensagem" era feito manualmente pelo próprio código do cliente (concatenar/formatar uma string e converter para bytes com getBytes()), e "interpretar a resposta" era o servidor fazendo parsing dessa string à mão. No gRPC, esse trabalho todo (serialização, desserialização, definição do formato) é feito pelo código gerado a partir do central.proto - o programador só monta o objeto da mensagem (PerguntaHorario) e lê os campos da resposta (RespostaHorario), sem escrever nenhuma lógica de parsing.

3. Com o servidor desligado, a chamada stub.consultarHorario(pergunta) não trava silenciosamente nem retorna null: ela lança uma exceção (StatusRuntimeException em Java, grpc.RpcError em Python) com um status como UNAVAILABLE, indicando que não foi possível estabelecer conexão com o servidor. Isso é bem diferente do TCP "na mão", onde normalmente é preciso tratar uma ConnectException/ConnectionRefusedError no próprio código de sockets - aqui o gRPC já entrega um erro padronizado e tipado.

