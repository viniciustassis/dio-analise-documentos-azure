# Implementação de Upload de Imagem para o Azure Storage

Este guia detalha a implementação de uma função em Python para realizar upload de imagens para o Azure Blob Storage, utilizando a biblioteca `azure.storage.blob`.

## 1. Configuração Inicial e Conexão com o Azure Storage

### Carregar Variáveis de Ambiente

Primeiro, carregue as variáveis de ambiente, utilizando o arquivo `Config.py`:

```python
from src.utils.Config import Config
```

Para confirmar que a variável de ambiente foi carregada corretamente:
```python
Config.CONTAINER_NAME
```

```python
'cartoes'
```

Nota: Utilizar variáveis de ambiente com Dotenv é uma maneira relativamente segura de lidar com keys de conexão.

### Estabelecendo a Conexão com o Azure Blob Storage
A classe BlobServiceClient da biblioteca azure.storage.blob é utilizada para estabelecer a conexão com o Blob Storage:

```python
from azure.storage.blob import BlobServiceClient
```

Para criar a conexão, use a string de conexão disponível em Config.STORAGE_CONNECTION:

```python
blob_service_client = BlobServiceClient.from_connection_string(Config.STORAGE_CONNECTION)
```

### Verificando a Conexão e Listando Blobs no Container
Podemos verificar a conexão listando os arquivos presentes no container cartoes:
```python
container_client = blob_service_client.get_container_client(container=Config.CONTAINER_NAME)
blob_list = container_client.list_blobs()
for blob in blob_list:
    print(f"Name: {blob.name}")
```

Saída esperada:
```makefile
Name: 6dcbfb18-a936-45b6-96aa-18c97a780943.jpg
Name: 9ee8b449-afee-442c-a4e6-67c539031b50.png
...
```

## 2. Criar Função para Fazer o Upload de Imagens
Vamos agora implementar a função de upload de imagem para o Blob Storage.

### Upload de um Texto para Teste

Para verificar o funcionamento básico, podemos enviar um texto para o blob:
```python
blob_client = blob_service_client.get_blob_client(Config.CONTAINER_NAME, blob="teste_upload.txt")
data = b"Vamos enviar esse texto no nosso container pra ver o que acontece."
blob_client.upload_blob(data, blob_type="BlockBlob")
```

Para verificar o conteúdo, baixe o arquivo e leia o conteúdo:
```python
blob_client = blob_service_client.get_blob_client(container=Config.CONTAINER_NAME, blob="teste_upload.txt")
downloader = blob_client.download_blob(max_concurrency=1, encoding='UTF-8')
blob_text = downloader.readall()
print(f"Blob contents: {blob_text}")
```

Saída esperada:
```yaml
Blob contents: Vamos enviar esse texto no nosso container pra ver o que acontece.
```

### Upload de Arquivo de Imagem
Para enviar uma imagem, abra o arquivo em modo de leitura binária e faça o upload:
```python
blob_client = blob_service_client.get_blob_client(Config.CONTAINER_NAME, 'foto.png')
with open(file="data/cartao-pre-pago-standard.jpg", mode="rb") as data:
    blob_client.upload_blob(data, overwrite=True)
``` 

Após o upload, a URL do blob pode ser obtida:
```python
blob_client.url
```

Saída esperada:
```
'https://stdiolab2.blob.core.windows.net/cartoes/foto.png'
```

Dica: Certifique-se de abrir o arquivo com open em modo "rb" (leitura binária) para garantir que a imagem seja enviada corretamente.

## 3. Criar Função Geral para Upload de Imagem
Agora, vamos encapsular o processo em uma função chamada upload_to_blob, que será armazenada no arquivo blob_service.py em src/services.

```python
# %%writefile src/services/blob_service.py
from azure.storage.blob import BlobServiceClient 
from src.utils.Config import Config 

def upload_to_blob(source, filename=None) -> str:
    \"\"\"
    Faz o upload de uma imagem para o Azure Blob Storage.

    Args:
        source (str): Caminho completo da imagem. Exemplo: "img/imagem.png".
        filename (str): Nome opcional para o arquivo no Blob Storage.

    Returns: 
        str: URL do blob após o upload.
    \"\"\" 
    # Definindo o nome do arquivo com extensão
    if not filename:        
        file = source.split('.')[0].split('/')[-1]
        extension = source.split('.')[-1]
        filename = f"{file}.{extension}"
        
    # Conexão e upload para o Blob Storage
    blob_service_client = BlobServiceClient.from_connection_string(Config.STORAGE_CONNECTION)
    blob_client = blob_service_client.get_blob_client(Config.CONTAINER_NAME, filename)
    with open(file=source, mode="rb") as data:
        blob_client.upload_blob(data, overwrite=True)
    
    # Retorna a URL do blob
    return blob_client.url
```

Salve o código acima no arquivo src/services/blob_service.py.
