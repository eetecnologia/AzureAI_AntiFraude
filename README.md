# **Análise de Documentos Anti-Fraude com Python e Azure AI Document Intelligence**

Este projeto é um guia prático para a construção de uma aplicação simples em Python que utiliza o serviço **Azure AI Document Intelligence** (anteriormente conhecido como Form Recognizer) para analisar documentos de identidade e ajudar na detecção de possíveis fraudes.

A aplicação extrai informações-chave de documentos como RGs, CNHs e passaportes, avaliando os níveis de confiança de cada campo extraído, um passo fundamental em processos de verificação de identidade (KYC \- Know Your Customer).

## **Funcionalidades Principais**

* **Extração de Dados:** Captura de informações como nome completo, data de nascimento, número do documento, data de validade, etc.  
* **Análise de Confiança:** Cada campo extraído vem com um score de confiança, permitindo identificar possíveis adulterações ou baixa qualidade da imagem.  
* **Modelo Pré-treinado:** Utiliza o modelo prebuilt-idDocument da Azure, que é otimizado para documentos de identidade de vários países, incluindo o Brasil.  
* **Simplicidade:** O código é direto e fácil de entender, focado em demonstrar a integração com o serviço da Azure.

## **🚀 Pré-requisitos**

Antes de começar, você precisará ter o seguinte:

1. **Python 3.8 ou superior:** [Instalar Python](https://www.python.org/downloads/).  
2. **Conta no Azure:** Uma assinatura ativa. Você pode [criar uma conta gratuita](https://azure.microsoft.com/free/).  
3. **Recurso de Azure AI Document Intelligence:** Criado no seu portal do Azure para obter as chaves de acesso.  
4. **Git:** Para clonar o repositório.

## **🛠️ Configuração do Ambiente**

Siga os passos abaixo para configurar o projeto localmente.

### **1\. Crie o Recurso no Azure**

* Acesse o [Portal do Azure](https://portal.azure.com/).  
* Clique em **"Criar um recurso"** e procure por **"Document Intelligence"**.  
* Crie o recurso, selecionando sua assinatura, um grupo de recursos e uma região. O nível de preço Free F0 é suficiente para este projeto.  
* Após a criação, vá para o recurso e, no menu à esquerda, clique em **"Chaves e Ponto de Extremidade"**. Copie os valores de Ponto de Extremidade e uma das Chaves (KEY 1).

### **2\. Configure o Projeto Local**

Clone este repositório (ou crie a estrutura de arquivos abaixo) e configure o ambiente virtual.

\# Clone o repositório (exemplo)  
git clone \[https://github.com/seu-usuario/seu-repositorio.git\](https://github.com/seu-usuario/seu-repositorio.git)  
cd seu-repositorio

\# Crie um ambiente virtual  
python \-m venv venv

\# Ative o ambiente virtual  
\# Windows  
venv\\Scripts\\activate  
\# macOS/Linux  
source venv/bin/activate

### **3\. Crie os Arquivos do Projeto**

Crie os seguintes arquivos na raiz do seu projeto:

requirements.txt  
Este arquivo lista as bibliotecas Python necessárias.  
azure-ai-documentintelligence  
python-dotenv

.env  
Este arquivo armazenará suas credenciais do Azure de forma segura. Nunca adicione este arquivo ao Git\!  
\# .env  
AZURE\_DOCUMENT\_INTELLIGENCE\_ENDPOINT="COLE\_SEU\_PONTO\_DE\_EXTREMIDADE\_AQUI"  
AZURE\_DOCUMENT\_INTELLIGENCE\_KEY="COLE\_SUA\_CHAVE\_AQUI"

analyze.py  
Este é o script principal da nossa aplicação.  
import os  
from dotenv import load\_dotenv  
from azure.core.credentials import AzureKeyCredential  
from azure.ai.documentintelligence import DocumentIntelligenceClient  
from azure.ai.documentintelligence.models import AnalyzeResult

\# Carrega as variáveis de ambiente do arquivo .env  
load\_dotenv()

\# Obtém as credenciais do Azure a partir das variáveis de ambiente  
endpoint \= os.getenv("AZURE\_DOCUMENT\_INTELLIGENCE\_ENDPOINT")  
key \= os.getenv("AZURE\_DOCUMENT\_INTELLIGENCE\_KEY")

def analyze\_identity\_document(document\_url: str):  
    """  
    Analisa um documento de identidade a partir de uma URL.

    Args:  
        document\_url (str): A URL pública do documento a ser analisado.  
    """  
    if not endpoint or not key:  
        print("ERRO: As variáveis de ambiente AZURE\_DOCUMENT\_INTELLIGENCE\_ENDPOINT e AZURE\_DOCUMENT\_INTELLIGENCE\_KEY não foram definidas.")  
        return

    print(f"Analisando o documento a partir da URL: {document\_url}\\n")  
      
    \# Inicializa o cliente do Document Intelligence  
    document\_intelligence\_client \= DocumentIntelligenceClient(  
        endpoint=endpoint, credential=AzureKeyCredential(key)  
    )

    \# Inicia a análise usando o modelo pré-treinado para documentos de identidade  
    poller \= document\_intelligence\_client.begin\_analyze\_document(  
        "prebuilt-idDocument", analyze\_request={"urlSource": document\_url}  
    )  
      
    \# Aguarda o resultado da análise  
    id\_documents: AnalyzeResult \= poller.result()

    if id\_documents.documents:  
        print("--- Resultado da Análise \---")  
        for idx, id\_document in enumerate(id\_documents.documents):  
            print(f"\\n--- Documento \#{idx \+ 1} \---")  
              
            \# Extrai e exibe os campos do documento  
            extract\_and\_print\_field(id\_document, "FirstName", "Primeiro Nome")  
            extract\_and\_print\_field(id\_document, "LastName", "Sobrenome")  
            extract\_and\_print\_field(id\_document, "DocumentNumber", "Número do Documento")  
            extract\_and\_print\_field(id\_document, "DateOfBirth", "Data de Nascimento")  
            extract\_and\_print\_field(id\_document, "DateOfExpiration", "Data de Expiração")  
            extract\_and\_print\_field(id\_document, "Sex", "Sexo")  
            extract\_and\_print\_field(id\_document, "CountryRegion", "País/Região")  
            extract\_and\_print\_field(id\_document, "Address", "Endereço")  
            extract\_and\_print\_field(id\_document, "Signature", "Assinatura")

            \# Avaliação de risco baseada na confiança  
            confidence \= id\_document.confidence  
            print(f"\\nConfiança geral do documento: {confidence:.2%}")  
            if confidence \< 0.75:  
                print("ALERTA: A confiança geral do documento é baixa. Recomenda-se verificação manual.")

    else:  
        print("Nenhum documento de identidade foi encontrado.")  
          
    print("\\n--- Fim da Análise \---")

def extract\_and\_print\_field(document, field\_name: str, display\_name: str):  
    """  
    Extrai um campo específico do resultado, exibe seu valor e confiança.  
    """  
    field \= document.fields.get(field\_name)  
    if field:  
        field\_value \= field.get("content", "N/A")  
        confidence \= field.get("confidence", 0\)  
        print(f"{display\_name}: '{field\_value}' (Confiança: {confidence:.2%})")  
        if confidence \< 0.8:  
            print(f"  \-\> ALERTA DE BAIXA CONFIANÇA para o campo '{display\_name}'\!")  
    else:  
        print(f"{display\_name}: Não encontrado.")

if \_\_name\_\_ \== "\_\_main\_\_":  
    \# URL de um documento de exemplo para análise  
    \# Substitua pela URL do documento que você deseja analisar  
    sample\_url \= "\[https://raw.githubusercontent.com/Azure-Samples/cognitive-services-REST-api-samples/master/curl/form-recognizer/sample-license.png\](https://raw.githubusercontent.com/Azure-Samples/cognitive-services-REST-api-samples/master/curl/form-recognizer/sample-license.png)"  
    analyze\_identity\_document(sample\_url)

### **4\. Instale as Dependências**

Com o ambiente virtual ativado, instale as bibliotecas listadas no requirements.txt.

pip install \-r requirements.txt

## **▶️ Como Executar a Aplicação**

Para executar a análise, basta rodar o script analyze.py. O script usará a URL de exemplo fornecida no código.

python analyze.py

**Saída de Exemplo:**

Analisando o documento a partir da URL: \[https://raw.githubusercontent.com/Azure-Samples/cognitive-services-REST-api-samples/master/curl/form-recognizer/sample-license.png\](https://raw.githubusercontent.com/Azure-Samples/cognitive-services-REST-api-samples/master/curl/form-recognizer/sample-license.png)

\--- Resultado da Análise \---

\--- Documento \#1 \---  
Primeiro Nome: 'JENNIFER' (Confiança: 99.50%)  
Sobrenome: 'PRITZKER' (Confiança: 99.50%)  
Número do Documento: 'WD000000' (Confiança: 99.50%)  
Data de Nascimento: '1992-01-01' (Confiança: 99.00%)  
Data de Expiração: '2025-01-01' (Confiança: 99.00%)  
Sexo: 'F' (Confiança: 98.80%)  
País/Região: 'USA' (Confiança: 99.20%)  
Endereço: '1234 Main Street, Redmond, WA 98052' (Confiança: 98.00%)  
Assinatura: 'J. Pritzker' (Confiança: 95.00%)

Confiança geral do documento: 98.50%

\--- Fim da Análise \---

## **🧐 Interpretando os Resultados para Anti-Fraude**

A detecção de fraude não se baseia apenas em um fator, mas na combinação de vários sinais de alerta:

* **Baixa Confiança Geral:** Se a confiança do documento como um todo for baixa, pode indicar uma imagem de má qualidade, um documento não padrão ou uma possível adulteração.  
* **Baixa Confiança em Campos Específicos:** Fique atento a campos como Data de Nascimento, Número do Documento e Assinatura. Uma confiança baixa aqui (\< 80%) é um forte indicador para uma revisão manual.  
* **Inconsistências:** Compare os dados extraídos com as informações fornecidas pelo usuário em um cadastro. Divergências em nomes, datas ou números são suspeitas.  
* **Campos Ausentes:** Se o modelo não conseguir encontrar campos essenciais, como o número do documento, isso é um sinal de alerta.

## **🚀 Próximos Passos**

* **Analisar Arquivos Locais:** Modifique o script para analisar arquivos do seu computador em vez de uma URL.  
* **Integração com Web App:** Incorpore esta lógica a um framework web como Flask ou Django para criar um endpoint de upload e análise.  
* **Modelos Personalizados:** Para documentos muito específicos da sua empresa (como faturas ou contratos), treine um modelo personalizado no Azure para obter uma precisão ainda maior.  
* **Armazenar Resultados:** Salve os resultados da análise em um banco de dados para auditoria e análises futuras.
