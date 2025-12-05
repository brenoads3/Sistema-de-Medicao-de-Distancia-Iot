Sistema-de-Medicao-de-Distancia-Iot
 Descrição do Projeto

O projeto Sistema de Medição de Distância utilizando IoT tem como objetivo monitorar, em tempo real, a distância entre um sensor ultrassônico e um objeto, utilizando o microcontrolador ESP32. As medições são realizadas pelo sensor HC-SR04 e enviadas via Wi-Fi ao HiveMQ Cloud utilizando o protocolo MQTT. O sistema também permite o acionamento remoto de um LED, demonstrando uma comunicação bidirecional entre o hardware e a plataforma em nuvem.

O projeto exemplifica a integração entre sensoriamento, atuadores e comunicação via internet, seguindo os princípios fundamentais da Internet das Coisas (IoT).

Funcionamento e Uso

O funcionamento do sistema ocorre da seguinte forma:

O sensor HC-SR04 emite pulsos ultrassônicos e calcula a distância até o objeto com base no tempo de retorno do eco.

O ESP32 processa essa distância e publica o valor no broker MQTT.

O dispositivo também permanece inscrito em um tópico específico, onde recebe comandos remotos.

O LED conectado ao ESP32 pode ser ligado ou desligado por meio de mensagens enviadas por um cliente MQTT (HiveMQ Web Client ou MQTT Explorer).

A leitura do sensor é realizada automaticamente a cada 2 segundos, garantindo monitoramento contínuo.

Para reproduzir o projeto, basta montar o circuito conforme o diagrama, ajustar no código as credenciais de Wi-Fi e do HiveMQ Cloud, enviar o firmware por meio da Arduino IDE e acompanhar os dados pelo cliente MQTT da sua escolha.

Software e Documentação de Código

O software foi desenvolvido em C++ utilizando a Arduino IDE. As bibliotecas utilizadas foram:

WiFi.h – Gerencia a conexão do ESP32 à rede sem fio.

WiFiClientSecure.h – Estabelece comunicação segura via TLS/SSL com o broker MQTT.

PubSubClient.h – Implementa o protocolo MQTT e permite publicação e recebimento de mensagens.

Arquivos presentes no repositório:

Código-fonte principal (arquivo .ino).

Diagrama da montagem do circuito.

Fluxograma do funcionamento.

Arquitetura do sistema.

O código contém comentários explicativos sobre conectividade, leitura do sensor, envio dos dados ao broker e controle do LED por meio de mensagens MQTT.

Hardware Utilizado

O hardware empregado no projeto inclui:

ESP32 DevKit V1 – Microcontrolador principal, responsável pela leitura do sensor, acionamento do LED e comunicação MQTT.

Sensor ultrassônico HC-SR04 – Realiza medições de distância utilizando sinais de TRIG e ECHO.

LED – Atuador visual acionado por comandos MQTT.

Jumpers – Cabos utilizados para realizar as conexões elétricas.

Protoboard – Utilizada para montagem do circuito sem solda.

Resistores de 330 Ω – Utilizados para limitar a corrente do LED.

Cabo USB-C – Responsável por alimentar o ESP32 e permitir o upload do código.

Interfaces, Protocolos e Módulos de Comunicação

O sistema utiliza as seguintes interfaces e protocolos:

Interfaces de comunicação:

Comunicação digital entre o ESP32 e o sensor HC-SR04 pelos pinos TRIG e ECHO.

Acionamento digital do LED por GPIO.

Conexão Wi-Fi para acesso à internet.

Protocolos utilizados:

TCP/IP – Base para a comunicação remota.

MQTT – Protocolo leve utilizado para publicação e recepção de mensagens entre o ESP32 e o servidor.

TLS/SSL – Protocolo de segurança utilizado na porta 8883 para criptografar a comunicação.

Bibliotecas e módulos:

Módulo Wi-Fi interno do ESP32.

Biblioteca PubSubClient para gerenciar tópicos MQTT.

Biblioteca WiFiClientSecure.h para autenticação e criptografia.

Comunicação via Internet e MQTT

A comunicação remota utiliza o HiveMQ Cloud como broker MQTT. Os parâmetros utilizados foram:

Servidor: 9dd3751a8eea408ca765c1d3066a96d5.s1.eu.hivemq.cloud

Porta: 8883

Usuário MQTT: BRENOSOUZA

Tópico de publicação: minha_casa/sensor_distancia

Tópico de assinatura: minha_casa/luz

As medições de distância são enviadas periodicamente ao tópico configurado, enquanto comandos remotos recebidos permitem o acionamento do LED.

Autores

Breno Oliveira de Souza
