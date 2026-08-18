4.1 - 1. Prejudica. Motivos: atrapalha a fácil visualização por operadores do sistema, aumenta a exposição do endereço para terceiros em caso de vazamento do código fonte, troca posterior exigiria recompilação do sistema.

2. Ausencia de transparencia já que torna o resultado da string suscetível a falhas de sintaxe, aumenta a probabilidade de falta de padrão e causa acoplamento rigído ao prender a lógica de montagem da url ao codigo fonte.

3. O cliente apontaria para um recurso inexistente ou simplesmente errado. Nenhuma das quatro alternativas escaparia disso, já que todas são baseadas em código hardcoded, sem utilizar uma lógica de escolha por configurações via variáveis de ambiente.

4.3 - 1. A transparência de acesso, já que o programa desenvolvido pelo mesmo acessar esses recursos de um servidor remoto via requisições com um formato e interfaces claras, seja com base nos padrões REST ou gRPC, por exemplo.

2. Não. Chamdas remotas possuem particularidades que as locais não possuem, como latência de rede elevada e por muitas vezes com picos imprevisíveis. Mascarar essa diferença pode levar a problemas de rastreabilidade em caso de tratamento de erros.

5.5 - 1. Evita erros humanos, documenta a API e garante que cliente e servidor usem o mesmo formato

2. Indica que o combo gRPC + protocol buffers é agnóstico a linguagens de programação, possibilitando, por exemplo, colaboração menos restritiva entre profissionais especializados em diferentes tecnologias.

3. A classe gerada automaticamente CentralAtendimentoGrpc.java pertence ao serviço central.CentralAtendimento, sendo o método getConsultarHorarioMethod responsável pela funcionalidade de ConsultarHorario.