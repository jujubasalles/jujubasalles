<h1 align="center"> Lambda iupp_transfer_events_ip6 </h1>

![Badge em Desenvolvimento](http://img.shields.io/static/v1?label=STATUS&message=EM%20PRODUÇÃO&color=GREEN&style=for-the-badge)


# Descrição do Projeto
A lambda iupp_transfer_events_ip6 possui a função de coletar os eventos do Kafka da ZUP, separar em mensagens apartadas se necessário, enriquecer com metadados e enviar para a API Gateway do Itaú (analytics-gerirdadosiupp-v1-ext-aws).



## Triggers da lambda

Lambda possui conexão via trigger com AWS MSK nos seguintes Kafkas e tópicos:

|Servidor KAFKA|Nome do tópico|
|--------------|-------------|
|iupp-prd-iupp-msk|iupp.checkout-manager.stream.order-cancelled|
|iupp-prd-iupp-msk|iupp.checkout-manager.stream.order-paid|
|iupp-prd-iupp-msk|iupp.checkout-manager.stream.order-error|
|itau-shop-stg-mkplace-msk|itaushop.mkplace.stream.order.updated|

## Estrutura de pastas

A estrutua de pastas está dividida da seguinte forma:

```text
├───mylayer
├───src
│   └───service
├───utils
```



## Segundo a estrutura de pastas , seguem seus respectivos codigos python e suas funcionalidades
```text
> mylayer
```
Nessa pasta temos um único arquivo .txt
No arquivo requirements.txt podemos incluir as bibliotecas que
precisamos importar dentro da lambda

```text
> src/service
```

`access_token_getter.py`

##### :hammer: def get_ip6_access_token:
Constrói o header para chamada via request da API de transferência de eventos

##### :hammer: def build_ip6_access_header:
Invoca a lambda iupp_certificate para coletar token e token_type 
para criar o header de invocação da API de transferência de eventos

```text
> utils
```
`message_processing.py`

##### :hammer: def extract_messages
Extrai as mensagens dos eventos recebidos
Se tiver mais de uma mensagem por evento ele faz a extração de uma por uma

##### :hammer: def enrich_payload
Realiza o enrequecimento do payload para o correto funcionamento da API de transferência
"fields": [
            {
                "specversion": specversion,
                "type": namespace,
                "source": f"urn:ip6:{name_snake_case}",
                "id": id_,
                "time": timestamp,
                "eventversion": eventversion,
                "transactionid": transactionid,
                "datacontenttype": datacontenttype,
                "data": json.loads(base64.b64decode(payload))
            }
        ]
Gera no final o enriched_payload

##### :hammer: def enrich_message
Realiza um segundo enriquecimento com metadados mínimos para 
o corret funcionamento da API de transferência
{
        "schema": schema,
        "id": id_,
        "version": version,
        "topic": topic,
        "partition": partition,
        "offset": offset,
        "timestamp": timestamp,
        "date": datetime.now().strftime("%Y-%m-%d"),
        "data": enriched_payload
    }

##### :hammer: def download_s3
Função para coletar a Key no bucket s3 de certificado

##### :hammer: def tranfer_messages_ip6
Função invoca o download_s3 para coletar CERT_FILENAME_CERT, CERT_FILENAME_KEY
Realiza a chamada da API via request e envia o payload+metadados 


`lambda_handler.py`
Realiza a chamada das funções na ordem correta para 
criação do payload+metadados para envio a API de transferência

