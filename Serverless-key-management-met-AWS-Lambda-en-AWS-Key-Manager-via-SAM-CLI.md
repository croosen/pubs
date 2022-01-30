Serverless key management met AWS Lambda en AWS Key Manager via SAM CLI
Sla je API keys veilig op en gebruik ze met Lambda

In deze korte post leg ik uit hoe je veilig API keys kunt gebruiken in Lambda. AWS heeft een key manager maar het was nog even vogelen om uit te vinden hoe dit werkende te krijgen met SAM CLI.

API calls via AWS Lambda met API keys
Om het voorbeeld simpel te houden gaan we data ophalen van de OpenWeather API. Het actuele weer haal je op met deze URL:

https://api.openweathermap.org/data/2.5/onecall?lat={lat}&lon={lon}&exclude={part}&appid={API key}

In Python zou dat er zo uit kunnen zien:

import requests
import json

api_key = "0123456789abcdef0123456789abcdef"
lat = "48.208176"
lon = "16.373819"
url = "https://api.openweathermap.org/data/2.5/onecall?lat=%s&lon=%s&appid=%s&units=metric" % (lat, lon, api_key)

response = requests.get(url)
data = json.loads(response.text)
print(data)

Maar je wilt je API key natuurlijk niet op deze manier in je code zetten en deployen naar Lambda. Grote kans dat je hem ook commit naar je repo. Nu kun je natuurlijk werken met .env files of je kunt aan de slag met de AWS Key Management Service. Over het verschil tussen KMS en Secrets Manager:

AWS KMS returns a plaintext data key and a copy of that data key encrypted under the KMS key. Secrets Manager uses the plaintext data key and the Advanced Encryption Standard (AES) algorithm to encrypt the secret value outside of AWS KMS. It removes the plaintext key from memory as soon as possible after using it.

Ik heb gekozen voor het gemakt van de Secrets Manager. Om een Secret aan te maken log je in in je AWS account en loop je door de stappen heen die je vindt bij de Secrets Manager. In de eerste instantie kun je alles default laten en je API key van OpenWeather invullen. AWS maakt dan een arn voor je aan die er zo uitziet:

arn:aws:secretsmanager:${AWS::Region}:${AWS::AccountId}:secret:openweatherapi-[xxxxxx]

Je API key staat nu in de Secrets Manager waarna jij hem veilig kunt ophalen met de volgende (psuedo) code. AWS geeft je de juiste codevoorbeelden in verschillende talen na het aanmaken van je key.

    client = session.client(
        service_name='secretsmanager',
        region_name=region_name
    )
 get_secret_value_response = client.get_secret_value(
            SecretId=secret_name
        )

Deze code gebruik je in je Lambda code om je API key(s) op te halen. Je kunt dus meerdere API keys per Secret opslaan. Handig in het geval van een API en een API SECRET key. Je krijgt een stukje json terug waarin jouw API keys staan.

Lokaal testen met SAM CLI
Als je je Lambda functie lokaal wilt testen met SAM CLI kun je `sam local invoke FUNCTION_NAME` gebruiken. Wanneer je een Secret ophaalt in je code dan merk je dat je ook netjes je json terug krijgt. Maar de fun begon toen ik de Lambda code gedeployed had naar AWS. Er bleek geen json terug te komen vanuit de call naar de secret. 

Secrets Manager en SAM CLI
Om je Secret op te halen wanneer je je functie gefeployed hebt naar AWS moeten we nog één belangrijk stapje nemen. We moeten de arn toevoegen aan de template.json zodat onze Lambda functie de juiste policy heeft om je Secret ook daadwerkelijk op te mogen halen. Blijkbaar is dat met lokaal testen geen probleem, wanneer je code eenmaal op Lambda staat wel.

Resources:
  PrivateTraderFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: private_trader/
      Handler: app.lambda_handler
      Runtime: python3.9
      Architectures:
        - x86_64
      Policies:
        - AWSSecretsManagerGetSecretValuePolicy:
            SecretArn: !Sub "arn:aws:secretsmanager:${AWS::Region}:${AWS::AccountId}:secret:openweatherapi-[xxxxxx]"

Meer over policy templates in SAM CLI: https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-policy-templates.html


Conclusie
API keys gebruiken in je Lambda code kun je op verschillende manieren doen. Ik heb vanwege de veiligheid maar ook vanwege het gemakt gekozen voor de AWS Secrets Manager. Om je Secret werkende te krijgen in je Lambda code geeft AWS je de voorbeeldcode. Dit alleen is niet voldoende om je Secret te kunnen benaderen wanneer je je code gedeployed hebt naar Lambda. Er moet nog een juiste policy worden toegevoegd aan je template.yml om je Lambda functie de juiste rechten te geven.

Verdere gedachten
Om je code clean te houden zou een apart .env file voor je template.yml gewenst zijn. SAM CLI lost dit op met een samconfig.toml. Je zou je region en account id daarin op kunnen nemen. Meer hierover: https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-config.html

Meer over SAM CLI
https://docs.aws.amazon.com/serverless-application-model/index.html



