# Laboratório - Transparências em Sistemas Distribuídos e gRPC

Central de Atendimento da Turma via gRPC, implementada em Java e Python a partir do contrato definido em `proto/central.proto`.

## Estrutura

- `proto/central.proto` - contrato do serviço `CentralAtendimento` (mensagens e RPCs)
- `java/grpc-central/` - implementação Java (Maven), servidor e cliente
- `python/grpc_central/` - implementação Python, servidor e cliente
- `evidencias/` - prints de tela das execuções (unário e streaming, Java e Python)
- `RESPOSTAS.md` - respostas às perguntas de cada parte do roteiro

## OFFSET usado

`OFFSET = 249` (dois últimos dígitos da matrícula/RA), aplicado às portas conforme seção 3.3 do roteiro:

| Servidor    | Porta-base | Porta usada |
|-------------|-----------:|------------:|
| gRPC - Java   |      50051 |       50132 |
| gRPC - Python |      50061 |       50142 |

## Como executar

### Java

```powershell
cd java/grpc-central
mvn compile exec:java -Dexec.mainClass=br.pucminas.labdamd.central.ServidorCentral   # terminal 1
mvn compile exec:java -Dexec.mainClass=br.pucminas.labdamd.central.ClienteCentral    # terminal 2
```

### Python

```powershell
cd python/grpc_central
pip install grpcio grpcio-tools
python -m grpc_tools.protoc -I ../../proto --python_out=. --grpc_python_out=. ../../proto/central.proto
python servidor_central.py     # terminal 1
python cliente_central.py      # terminal 2
```

Cada execução realiza a chamada unária `ConsultarHorario` e, em seguida, a chamada com streaming de servidor `AcompanharAvisos` (5 avisos, um a cada 2 segundos).
