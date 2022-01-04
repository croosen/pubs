# API keys gebruiken in AWS Lambda met SAM CLI

In deze korte post leg ik uit hoe je veilig API keys kunt gebruiken in AWS Lambda in combinatie met AWS Serverless Application Model (SAM CLI).

API calls via AWS Lambda
Om het voorbeeld simpel te houden gaan we fictief data ophalen van de OpenWeather API. Het actuele weer haal je op met deze URL:

`https://api.openweathermap.org/data/2.5/onecall?lat={lat}&lon={lon}&exclude={part}&appid={API key}`

In Python zou dat er zo uit kunnen zien:

``` python
import requests
import json

api_key = "0123456789abcdef0123456789abcdef"
lat = "12.123456"
lon = "12.123456"
url = "https://api.openweathermap.org/data/2.5/onecall?lat=%s&lon=%s&appid=%s&units=metric" % (lat, lon, api_key)

response = requests.get(url)
data = json.loads(response.text)
print(data)
```

Maar je wilt je API key natuurlijk niet op deze manier in je code zetten en deployen naar Lambda. 
Nu kun je werken met .env files of je kunt aan de slag met de AWS Key Management Service. Over het verschil tussen KMS en Secrets Manager:

> AWS KMS returns a plaintext data key and a copy of that data key encrypted under the KMS key. 
Secrets Manager uses the plaintext data key and the Advanced Encryption Standard (AES) algorithm to encrypt the secret value outside of AWS KMS. 
It removes the plaintext key from memory as soon as possible after using it.

## AWS Secrets Manager
Ik heb gekozen voor het gemak van Secrets Manager. Om een Secret aan te maken log je in in je AWS account en loop je door de stappen heen die je vindt bij de 
Secrets Manager. In de eerste instantie kun je alle settings default laten en je API key van OpenWeather invullen. AWS maakt dan een arn voor je aan die er zo uitziet:

`arn:aws:secretsmanager:${AWS::Region}:${AWS::AccountId}:secret:openweatherapi-[xxxxxx]`

Je API key staat nu in de Secrets Manager waarna jij hem veilig kunt ophalen met de volgende (psuedo) code. Deze code gebruik je in je Lambda code.
AWS geeft je de juiste codevoorbeelden in verschillende talen na het aanmaken van je Secret. Goed om te weten dat je meerdere keys kunt opslaan in een Secret. Na het ophalen van je Secret krijg je een stukje JSON terug met je key(s).

``` python
client = session.client(
  service_name='secretsmanager',
  region_name=region_name
)

get_secret_value_response = client.get_secret_value(
  SecretId=secret_name
)
```

## Lokaal testen met SAM CLI
Als je je Lambda functie lokaal wilt testen met SAM CLI kun je `sam local invoke FunctionName` gebruiken. Wanneer je een Secret ophaalt in je code dan merk je dat je ook netjes de JSON terug krijgt. Maar de pret begon toen ik de Lambda code gedeployed had naar AWS. **Er bleek geen JSON terug te komen vanuit de call naar de Secrets Manager**. 

## Secrets Manager en SAM CLI en template.yaml policies
Om je Secret op te halen wanneer je je functie gedeployed hebt naar AWS moeten we nog één belangrijk stapje nemen. **We moeten de arn toevoegen aan de template.yaml** zodat onze Lambda functie de juiste policy heeft om je Secret ook daadwerkelijk op te mogen halen wanneer je code eenmaal in de cloud staat.

``` yaml
Resources:
  FunctionName:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: function_name/
      Handler: app.lambda_handler
      Runtime: python3.9
      Architectures:
        - x86_64
      Policies:
        - AWSSecretsManagerGetSecretValuePolicy:
            SecretArn: !Sub "arn:aws:secretsmanager:${AWS::Region}:${AWS::AccountId}:secret:openweatherapi-[xxxxxx]"
```

## Conclusie
API keys gebruiken in je Lambda code kun je op verschillende manieren doen. 
Ik heb vanwege de veiligheid maar ook vanwege het gemak gekozen voor de AWS Secrets Manager. 
Om je Secret werkend te krijgen in je Lambda code geeft AWS je de voorbeeldcode. 
Dit alleen is niet voldoende om je Secret te kunnen benaderen wanneer je je code gedeployed hebt naar de cloud. 
Er moet nog een juiste policy worden toegevoegd aan je template.yaml om je Lambda functie de juiste rechten te geven.

## Links
Meer over policy templates in SAM CLI: [https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-policy-templates.html](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-policy-templates.html)  
Meer over SAM CLI:
[https://docs.aws.amazon.com/serverless-application-model/index.html](https://docs.aws.amazon.com/serverless-application-model/index.html)



