### Requisitos

- Ter instalado `npm` e `npx`
- Ter instalado eas (`npm install -g eas-cli`)

### Sistema de notificações utilizando RN e firebase como back-end (android)

Inicie um novo projeto react native usando o expo com o comando:
```bash
npx create-expo-app --template
```
Em seguida selecione `Blank` como template do seu projeto.

Após a criação do projeto entre na pasta `root` do projeto e instale as dependências necessárias para utilizar o firebase.
```bash
npm install --save @react-native-firebase/app
npm install --save @react-native-firebase/messaging
```
Em seguida instale o expo-dev-client com o seguinte comando:
```bash
npx expo install expo-dev-client
```
Agora é possível criar a primeira build do nosso projeto, que será necessária para se obter o `package name` do nosso projeto, para que possamos criar um projeto android no firebase, utilize o seguinte comando:
```shell
eas build --profile development --platform android
```
Agora que temos a build do nosso projeto, entre em [firebase console](https://console.firebase.google.com/u/0/) e crie um projeto, e siga o seguinte caminho:

configurações do projeto -> contas de serviço -> gera nova chave privada

Esse processo deve gerar para você um arquivo `.json` que será automaticamente instalado em seu ambiente de trabalho, copie esse arquivo e cole ele na pasta `root` do seu projeto.

No terminal digite o seguinte comando:
```shell
eas credentials
```
 e selecione as seguintes opções:
 
Android -> production -> google service account -> FCM V1

Após esse processo, se você estiver na pasta root do seu projeto ele deve reconhecer automaticamente o arquivo `.json` que foi instalado anteriormente, confirme.

OBS.: se você não possuir uma conta em [expo dev](https://expo.dev/) será talvez seja necessário cria-la nessa etapa, pois é lá que a nossa build ficara hospedada, alem disso para seguir o próximo passo é necessário possuir uma conta expo dev.

Para verificar se esse processo deu certo entre em [expo dev](https://expo.dev/) e siga o seguinte caminho:

clique no seu projeto -> credentials -> clique na credencial presente em Android -> FCM V1 service account key.

Siga para o seu projeto no firebase e clique em `android` insira o `package name` presente no seu `app.json` escolha um nome para o seu projeto e para a chave SHA1 siga os seguintes passos:

1. No terminal digite o comando ```eas credentials```.
2. Selecione Android -> development -> Keystore -> set up new keystore.
3. aceite as perguntas seguintes pressionando `y`.

depois de finalizar você vai perceber que possui duas chaves SHA1, copie a primeira e cole na configuração do firebase e registre o aplicativo no firebase, baixe o `google-services.json` e cole na root do seu projeto.

Nas configurações do seu projeto, em geral, procure a opção adicionar impressão digital e nela cole a segunda chave SHA1 que você obteve.

Vá para o arquivo `app.json` no seu projeto e especifique o caminho do google services, adicionando a seguinte linha:
```json
{
  "expo": { 
	"android": {
	  "googleServicesFile": "./path/to/google-services.json"
	},
  }
}
```
 após essa modificação dê um reload na sua aplicação usando o comando: `eas build --profile development --platform android`, isso será necessário sempre que uma modificação for feita no arquivo `app.json` do seu projeto.

Finalizada a configuração e o processo de build adicione o seguinte código em `App.js`:
```javascript
import { StatusBar } from 'expo-status-bar';
import { StyleSheet, Text, View, Alert } from 'react-native';
import messaging from "@react-native-firebase/messaging"
import React, { useEffect } from "react"

export default function App() {
  const requestUserPermission = async () => {
    const authStatus = await messaging().requestPermission();
    const enabled =
      authStatus === messaging.AuthorizationStatus.AUTHORIZED ||
      authStatus === messaging.AuthorizationStatus.PROVISIONAL;

    if (enabled) {
      console.log("Authorization status:", authStatus) 
    }
  };

  useEffect (() => {
    if (requestUserPermission()) {
      messaging()
        .getToken()
        .then((token) => {
          console.log(token);
        });
    } else {
      console.log("Permission not granted", authStatus);

    }
  
    // Verifique se uma notificação inicial está disponível
    messaging()
      .getInitialNotification()
      .then(async (remoteMessage) => {
        if (remoteMessage) {
          console.log("Notifications caused app to open from quit state:", remoteMessage.notification);
        }
      });
    
    // Suponha que uma notificação de mensagem contenha uma propriedade de "type" na carga de dados da tela a ser aberta
    messaging().onNotificationOpenedApp((remoteMessage) => {
      console.log("nNotification caused app to open from a background state:", remoteMessage.notification);
    });
    
    // Registrar handler para o background
    messaging().setBackgroundMessageHandler( async (remoteMessage) => {
      console.log("Mensage handled in the background!", remoteMessage);
    });

    const unsubscribe = messaging().onMessage( async (remoteMessage) => {
      Alert.alert("A new FCM message arrived!", JSON.stringify(remoteMessage));
    });

    return unsubscribe;
  }, []);
  return (
    <View style={styles.container}>
      <Text>FCM tutorial</Text>
      <StatusBar style="auto"/>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: "#fff",
    alignItems: "center",
    justifyContent:"center"
  }
})
```
Após salvar as modificações, gere uma preview do seu aplicativo com o comando `eas build -p android --profile preview`, depois que o processo de build terminar escaneie o QR code com o celular e baixe a aplicação que estará em `.apk` no site do expo.dev.

Tudo certo, já podemos testar entrando no firebase e selecionando Executar -> Messaging -> Criar minha primeira campanha.

OBS.: lembre-se de permitir as notificações da sua aplicação no celular.
