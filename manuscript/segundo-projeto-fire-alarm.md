## Segundo projeto: Alarme de incêndio

O nosso segundo projeto de exemplo será o de um alarme de incêndio inteligente. O nosso alarme de incêndio verificará pela temperatura e, em caso de incêndio, acionará o alarme sonoro e enviará um SMS para o celular cadastrado.

Um exemplo simples, mas que mostra alguns pontos de integração interessantes, como integração com API's, leitura de dados de um sensor de temperatura e integração com o sensor sonoro Piezo.


### Anatomia de um alarme de incêndio


O projeto foi baseado em um sistema domiciliar simples de alarme de incêndio. Por padrão sistemas deste tipo fazem uma verificação de tempos em tempos e ativam o alarme de algum padrão anormal de temperatura é encontrado.


### Material necessário

Para este projeto utilizaremos:

- 1 Protoboard: Uma protoboard nada mais é que uma placa com furos e conexões condutoras para montagem de circuitos elétricos experimentais, sem a necessidade de soldagem;
- 1 sensor de alarme Piezo: Ele será utilizado para o feedback sonoro ao usuário final, no nosso caso ao(s) inquilino(s) do imóvel. Este sensor é bem simples e custa menos de R$ 2,00 podendo ser encontrado em qualquer loja de elétrica;
- 1 sensor de temperatura: O Johnny-Five trabalha os mais diversos sensores de temperatura. Neste caso usaremos o sensor de temperatura LM35. Este sensor é bem simples e custa menos de R$ 1,50 podendo ser encontrado em qualquer loja de elétrica;

* Para obter a lista completa dos sensores suportados acesse a página Wiki do Johnny para sensores de temperatura e verifique a lista dos códigos de referência dos sensores.

Alguns sensores necessitam de uma porta de voltagem específica para o seu correto funcionamento (alguns sensores chamam de VND). No nosso caso usaremos uma porta de 5 volts para o sensor de temperatura.

Outros sensores podem precisar de uma porta analógica. Portas analógicas são utilizadas para que o sensor envie dados de voltagem para o Arduino e assim possamos ler e interpretar as suas informações.

Para este projeto vamos montar os sensores no Arduino da seguinte maneira:

- Sensor de alarme piezo: encaixe o fio preto do piezo na porta GND e o fio vermelho na porta de número 3 do nosso arduino;
- Sensor de temperatura LM35: encaixe o pino de aterramento na porta GND, o pino de voltagem na porta de 5 volts e o pino de saída de dados na porta analógica "A0";

A imagem a seguir ilustra a montagem dos componentes integrados com o Arduino.


### Controlando o sensor de chamas

Com o sensor LM35 conectado à placa Arduino, vamos agora fazer a leitura da informação da temperatura do ambiente. Para isso utilizaremos a classe Thermometer do Johnny five. Ao instanciarmos um novo objeto Thermometer, devemos nos atentar a alguns parâmetros:

- controller: Nome do sensor utilizado. Você pode consultar a lista completa dos sensores suportados pelo Johnny Five na documentação da classe Thermometer;
- pin: A informação do pino utilizado para a conexão analógica no Arduino. Ele é utilizado em sensores analógicos para leitura da informação de temperatura;
- toCelsius: Um método opcional que podemos reescrever para lidar com o dado analógico e transformá-lo no formato de temperatura de sua preferência. No nosso caso utilizaremos o formato Celsius;

Baseado nestas informações, o arquivo principal será do nosso alarme de incêndio será:

```javascript
var five = require('johnny-five');
var board = new five.Board();

function FireAlarm() {
  return new five.Thermometer({
    controller: "LM35",
    pin: "A0"
  });
};

board.on("ready", function() {
  var temperatureSensor = new FireAlarm();

  setInterval(function() {
    console.log("celsius: %d", temperatureSensor.celsius);
  });
});
```

Vamos então validar a funcionalidade do nosso código com a plataforma Arduino. Dentro da pasta do nosso projeto, vamos digitar no nosso prompt de comando/terminal o seguinte comando:

```bash
$ node src/index.js
```

E este será o resultado do nosso código.

O código é bastante simples, como podem perceber. No próximo tópico vamos pensar um pouco mais sobre a nossa arquitetura e como evoluir este código para algo mais fácil de manter.


### Evoluindo o nosso código inicial

O nosso código inicial está funcional, mas evoluir este código para os próximos passos é algo complexo. Para facilitarmos as próximas etapas do nosso projeto, vamos fazer algumas adaptações no nosso código inicial.

Vocês podem perceber que temos várias configurações, tais como o nome do controller e pin do sensor. Para melhorarmos o manusear destas configurações que são constantes, ou seja, não mudam durante todo o tempo de vida da nossa aplicação, vamos adicioná-las em um arquivo com as nossas configurações específicas.

Este será adicionado ao nosso arquivo `src/configuration.js`

```javascript
// src/configuration.js
module.exports = {
  FIRE_ALARM: {
    // https://github.com/rwaldron/johnny-five/wiki/thermometer
    CONTROLLER: "LM35",
    PIN: 'A0'
  },
  INTERVAL: 1000
};
```

Este é o conteúdo do nosso arquivo src/fire-alarm.js. Percebam que agora estamos invocando o código de configuração externo e adicionando os valores na variável CONFIG. Esta etapa é interessante, pois desvinculamos as configurações básicas do nosso projeto da classe, que agora possui a responsabilidade de lidar com os sensores do projeto fire alarm.

```javascript
// src/fire-alarm.js
var CONFIG = require('./configuration');
var five = require('johnny-five');
var intervalId = null;

function FireAlarm() {
  this.temperatureSensor = new five.Thermometer({
    controller: CONFIG.FIRE_ALARM.CONTROLLER,
    pin: CONFIG.FIRE_ALARM.PIN
  });
};

FireAlarm.prototype.stopPolling = function() {
  clearInterval(intervalId);
};

FireAlarm.prototype.startPolling = function() {
  self = this;
  intervalId = setInterval(function() {
    console.log('celsius: %d', self.temperatureSensor.celsius);
  }, CONFIG.INTERVAL);
};

module.exports = FireAlarm;
```

E o nosso arquivo src/index.js principal vai ter um conteúdo mais simples, tendo a responsabilidade de iniciar o projeto e o polling da instância da classe FireAlarm.

```javascript
// src/index.js
var FireAlarm = require('./fire-alarm');
var five = require('johnny-five');
var board = new five.Board();

board.on('ready', function() {
  fireAlarm = new FireAlarm();
  fireAlarm.startPolling();
});
```

Quando executamos o nosso código a partir do comando

```javascript
$ npm start
```

Veremos o seguinte resultado na nossa linha de comando/prompt de comando.

[IMAGEM DO OUTPUT DO COMANDO]

Com essas alterações temos um código de fácil manutenção e muito mais legibilidade para ser utilizado na nossa aplicação. Claro que esta é uma das várias abordagens que podem ser utilizadas no seu projeto, mas o foco deste tópico é passar a idéia de sempre pensar em como melhorar o nosso projeto.


### Integrando com o Piezo para aviso sonoro

Twilio é um serviço que permite aos desenvolvedores incorporar voz, VoIP e mensagens SMS em aplicativos a partir de uma API RESTful que fornece os recursos de voz e SMS para aplicativos.

O sensor de alarme que vamos utilizar é o Piezo por se tratar de um componente simples de manipular e bastante barato, tendo o seu valor em torno de R$ 2,00.

A integração do Piezo ao nosso projeto é algo bem trivial, pois o Johnny Five já possui a classe five.Piezo.

```javascript
...
this.piezo = new five.Piezo(3);
...
```

Esta classe aceita o número do pino que está vinculado ao sensor e ao ativar temos o método play, que aceita um objeto com as informações:

- tempo: intervalo entre cada nota;
- song: array que aceita arrays de 2 posições com as informações de nota musical e o tempo;

Você pode perceber um exemplo do método play sendo utilizado no código abaixo.

```javascript
self.piezo.play({
  song: [
    ['G5', 1/4]
  ],
  tempo: 200
});
```

No nosso projeto vamos fazer a integração em dividida em 2 partes. Primeiramente vamos adicionar a instância do piezo no contrutor da nossa classe, para que o piezo fique acessível nos outros métodos.

```javascript
...
function FireAlarm() {
  this.piezo = new five.Piezo(3);
  ...
}
...
```

Com a nossa instância adicionada e acessível, vamos utilizá-lo no nosso método startPolling. Vamos adicionar mais uma validação, acessando o boolean piezo.isPlaying que contém a informação de inicialização do piezo no nosso projeto e, caso a temperatura esteja acima do limite e o piezo esteja acessível, acionaremos o nosso alarme sonoro. Com isto o método ficará como o código a seguir.

```javascript
...
FireAlarm.prototype.startPolling = function() {
  self = this;
  intervalId = setInterval(function() {
    if (self.temperatureSensor.celsius >= CONFIG.FIRE_ALARM.LIMIT && !self.piezo.isPlaying) {
      self.piezo.play({
        song: [
          ['G5', 1/4],
          [null, 7/4]
        ],
        tempo: 200
      });
      console.log('Up to the limit:', self.temperatureSensor.celsius);
    } else {
      console.log('That\'s ok:', self.temperatureSensor.celsius);
    }
  }, CONFIG.INTERVAL);
};
...
```

O nosso código final do FireAlarm terá o seguinte conteúdo:

```javascript
var CONFIG = require('./configuration');
var request = require('request');
var five = require('johnny-five');
var intervalId = null;

function FireAlarm() {
  this.piezo = new five.Piezo(3);
  this.temperatureSensor = new five.Thermometer({
    pin: CONFIG.FIRE_ALARM.PIN
  });
};


FireAlarm.prototype.stopPolling = function() {
  clearInterval(intervalId);
};

FireAlarm.prototype.startPolling = function() {
  self = this;
  intervalId = setInterval(function() {
    if (self.temperatureSensor.celsius >= CONFIG.FIRE_ALARM.LIMIT && !self.piezo.isPlaying) {

      self.piezo.play({
        song: [
          ['G5', 1/4],
          [null, 7/4]
        ],
        tempo: 200
      });

      console.log('Up to the limit:', self.temperatureSensor.celsius);
    } else {
      console.log('That\'s ok:', self.temperatureSensor.celsius);
    }

  }, CONFIG.INTERVAL);
};

module.exports = FireAlarm;
```

Com isto temos o primeiro feedback para os usuários da nossa aplicação. A próxima etapa será adicionar a funcionalidade de envio de SMS utilizando a API do Twilio.


### Enviando SMS para o seu celular usando o Twilio

Twilio é um serviço que permite aos desenvolvedores incorporar voz, VoIP e mensagens SMS em aplicativos a partir de uma API RESTful que fornece os recursos de voz e SMS para aplicativos. As bibliotecas de cliente estão disponíveis em vários idiomas e, claro, possuem um cliente para o NodeJS chamado twilio-node.

Integrar SMS no nosso aplicativo então será algo bem simples. Vamos ao site to Twilio e vamos habilitar o serviço de SMS.

Ele possui alguns números pagos, caso queira utilizar em algum produto realmente. No nosso caso utilizaremos o número gerado pelo próprio serviço e criaremos uma conta trial na plataforma.

Agora, com o Twilio configurado, vamos então começar a integração com o nosso alarme de incêndio. Primeiramente vamos adicionar as informações do telefone, chave SSID da sua conta e token de autenticação do Twilio. Vamos adicionar estas informações no nosso `src/configuration.js`.

```javascript
module.exports = {
     LIMIT: 30,
     PIN: 'A0',
     PHONE_NUMBER: ''
  },
  TWILIO: {
    PHONE_NUMBER: '',
    ACCOUNT_SSID: '',
    AUTH_TOKEN: ''
  },
  INTERVAL: 1000
};
```



E vamos con



```javascript
// fire-alarm.js

var CONFIG = require('./configuration');
var request = require('request');
var five = require('johnny-five');
var twilio = require('twilio');
var intervalId = null;

var client = new twilio.RestClient(CONFIG.TWILIO.ACCOUNT_SSID, CONFIG.TWILIO.AUTH_TOKEN);

function FireAlarm() {
  this.piezo = new five.Piezo(3);
  this.temperatureSensor = new five.Thermometer({
    pin: CONFIG.FIRE_ALARM.PIN
  });
};

FireAlarm.prototype.stopPolling = function() {
  clearInterval(intervalId);
};

FireAlarm.prototype.startPolling = function() {
  self = this;
  intervalId = setInterval(function() {
    if (self.temperatureSensor.celsius >= CONFIG.FIRE_ALARM.LIMIT && !self.piezo.isPlaying) {

      self.piezo.play({
        song: [
          ['G5', 1/4],
          [null, 7/4]
        ],
        tempo: 200
      });

      client.messages.create({
        body: 'Something is wrong with your fire alarm. Please, call the local fire brigade.',
        to: CONFIG.FIRE_ALARM.PHONE_NUMBER,
        from: CONFIG.TWILIO.PHONE_NUMBER
      });

      console.log('Up to the limit:', self.temperatureSensor.celsius);
    } else {
      console.log('That\'s ok:', self.temperatureSensor.celsius);
    }
  }, CONFIG.INTERVAL);
};

module.exports = FireAlarm;
```

Vamos agora executar o nosso código digitando o comando npm start e esta será  a informação impressa no nosso prompt/linha de comando.

[O QUE VAI SER IMPRESSO NA LINHA DE COMANDO]

Ao executarmos o nosso código e a temperatura exceder o limite configurado, além do sensor Piezo que acionará o alarme o nosso cliente do Twilio será acionado e nos enviará um SMS utilizando as configurações previamente adicionadas. Então você receberá a seguinte mensagem no seu celular.

Com isto vimos a integração completa do nosso Fire Alarm. Claro que este é o primeiro passo, você pode evoluir o código e adicionar novas funcionalidades. Como já sabemos: o céu é o limite!

Mas e quanto aos testes unitários? Sabemos que está funcionando, mas temos que ter certeza de que o código tem um nível aceitável de qualidade até mesmo para evoluirmos o nosso projeto e adicionar novas funcionalidades. Vamos agora criar os nossos testes unitários.


#### Criando testes unitários para o Fire Alarm


Como comentado no conteúdo "Criando testes unitários para o build checker", teste unitário é apenas uma das várias maneiras de testar o seu software e ter uma confiabilidade no produto final com algumas validações automáticas antes de efetuarmos o deploy do nosso projeto para produção.

Vamos agora criar uma pasta para os nossos testes unitários com o nome test. Como padrão, instaremos o framework de teste MochaJS, SinonJS para spies, stubs e mocks e ShouldJS para as assertions como dependência de desenvolvimento do nosso projeto.

```bash
$ mkdir test
$ npm install --save-dev mocha sinon should
```

Para os nossos testes reutilizaremos o conteúdo do test/spec-helper.js e do test/mocha.opts que criamos para o nosso projeto do build checker. Ele utiliza o pacote mock-Firmata para setup dos testes na nossa aplicação Johnny-Five.

```javascript
//test/spec-helper.js
require('should');
var mockFirmata = require('mock-firmata');
var five = require('johnny-five');

var board = new five.Board({
  io: new mockFirmata.Firmata(),
  debug: false,
  repl: false
});
```

E este será o conteúdo inicial do nosso mocha.opts.

```bash
--reporter spec
--recursive
--require test/spec-helper.js
--slow 1000
--timeout 5000
```

Uma explicação rápida sobre as informações de configuração utilizadas:

--reporter spec: Tipo de reporter utilizado para mostrar as mensagens das informações dos testes;
--recursive: flag para identificar que os testes devem rodar de maneira recursiva dentro da pasta;
--require test/spec-helper.js: arquivo de setup a ser carregado antes de rodarmos os testes unitários;
--slow 1000: Tempo máximo em milissegundos de tolerância entre os testes. Caso ultrapasse este tempo será mostrado o tempo total daquele teste com uma cor diferenciada para que possamos efetuar as devidas alterações;
--timeout 5000: Tempo máximo em milissegundos de tolerância para a finalização de cada asserção. Caso ultrapasse este tempo os nossos testes retornarão com uma mensagem de erro;

Agora sim, vamos criar os cenários para os nossos testes. Vamos então definir os cenários que devemos cobrir nos nossos testes:

- Configurações iniciais da aplicação;
- Informações iniciais quando criamos a instância do FireAlarm;
- Quando iniciamos o polling e o sensor sinaliza que a temperatura está dentro do limite aceitável;
- Quando iniciamos o polling e o sensor sinaliza que a temperatura está acima do limite aceitável;
- Quando o sensor piezo é acionado;
- Quando a API SMS do Twilio é acionada;

Uma forma de validarmos quando o fire alarm deve acionar o alarme sonoro é criarmos um stub para trigger e usaremos alguns spies para a API do Twilio.

Como primeiro passo criaremos os testes de nosso arquivo de configuração, que dividiremos entre as informações do sensor de temperatura e da API do Twilio. Faremos a requisição do nosso src/configuration.js e verificaremos a informação do pino que vai ser vinculado ao sensor, o valor do intervalo a ser utilizado para checar as informações do sensor e o limite aceitável da temperatura do ambiente no qual o FireAlarm estará funcionando e o telefone configurado para receber o SMS.

```javascript
var CONFIG = require('../src/configuration');

describe('Configuration', function() {

  describe('Temperature information', function() {

    it('should have the sensor port configured', function(){
      CONFIG.FIRE_ALARM.should.have.property('PIN').which.is.a.String()
    });

    it('should have the sensor alarm limit configured', function(){
      CONFIG.FIRE_ALARM.should.have.property('LIMIT').which.is.a.Number()
    });

    it('should have the user phone configured', function(){
      CONFIG.FIRE_ALARM.should.have.property('PHONE_NUMBER').which.is.a.String()
    });
  });
});
```

Agora avaliamos as informações que serão passadas para o Twilio, que no nosso caso serão a chave SSID, token de autenticação e o número telefônico fornecido pelo Twilio.

```javascript
  ...
  describe('SMS information', function() {

    it('should have account ssid configured', function(){
      CONFIG.TWILIO.should.have.property('ACCOUNT_SSID').which.is.a.String()
    });

    it('should have auth token configured', function(){
    CONFIG.TWILIO.should.have.property('AUTH_TOKEN').which.is.a.String()
    });

    it('should have the user phone configured', function(){
      CONFIG.TWILIO.should.have.property('PHONE_NUMBER').which.is.a.String()
    });
  });
  ...
```

O nosso arquivo de testes do src/configuration.js ficará assim.

```javascript
var CONFIG = require('../src/configuration');

describe('Configuration', function() {

  describe('Temperature information', function() {

    it('should have the sensor port configured', function(){
      CONFIG.FIRE_ALARM.should.have.property('PIN').which.is.a.String()
    });

    it('should have the sensor alarm limit configured', function(){
      CONFIG.FIRE_ALARM.should.have.property('LIMIT').which.is.a.Number()
    });

    it('should have the user phone configured', function(){
      CONFIG.FIRE_ALARM.should.have.property('PHONE_NUMBER').which.is.a.String()
    });
  });

  describe('SMS information', function() {

    it('should have account ssid configured', function(){
      CONFIG.TWILIO.should.have.property('ACCOUNT_SSID').which.is.a.String()
    });

    it('should have auth token configured', function(){
      CONFIG.TWILIO.should.have.property('AUTH_TOKEN').which.is.a.String()
    });

    it('should have the user phone configured', function(){
      CONFIG.TWILIO.should.have.property('PHONE_NUMBER').which.is.a.String()
    });
  });

  it('should have the interval polling information', function(){
    CONFIG.should.have.property('INTERVAL').which.is.a.Number()
  });

});
```

Vamos então criar o cenário para validar o código do nosso FireAlarm. Uma das coisas que devemos ter em mente sobre o nosso framework de teste unitário é que o seu método beforeEach, que acontece antes de cada método it vai ser utilizado como documentação para reprodução de cada cenário específico.


Vamos então explicar mais sobre o conteúdo deste arquivo e o porquê de cada teste. Criamos os testes da instância do nosso FireAlarm e seus atributos iniciais.

```javascript
var proxyquire = require('proxyquire');
var CONFIG = require('../src/configuration');
var five = require('johnny-five');
var request = require('request');
var sinon = require('sinon');

describe('FireAlarm', function() {

  beforeEach(function(){
    this.sandbox = sinon.sandbox.create();
    this.createMessagesSpy = this.sandbox.spy();
    var restClientMessage = {
      messages: {
        create: this.createMessagesSpy
      }
    };

    twilioMock = {
      RestClient: function() {
        return restClientMessage
      }
    };

    var FireAlarm = proxyquire('../src/fire-alarm', {
      twilio: twilioMock
    });
    fireAlarm = new FireAlarm();
  });

  afterEach(function() {
    this.sandbox.restore();
  });

  it('should have the termometer sensor configured', function(){
    (fireAlarm.temperatureSensor instanceof five.Thermometer).should.be.equal(true);
  });

});
```

Vamos validar quando paramos o nosso polling. Vamos usar agora o método spy do sinon para verificar se o código utilizou o método clearInterval para finalizar com as requisições. Para isso verificaremos se o global.clearInterval foi utilizado uma vez, acessando o boolean calledOnce, que é um contador interno adicionado pelo método sinon.spy para os testes.

```javascript
  ...
  describe('#stopPolling', function(){
    beforeEach(function(){
      this.sandbox.spy(global, 'clearInterval');
      fireAlarm.stopPolling();
    });

    it('should remove interval', function(){
      global.clearInterval.calledOnce.should.be.true;
    });
  });
  ...
```

E agora os cenários que acontecem quando o sensor é iniciado. Para isto vamos atribuir alguns spies para as funções setInterval e para o sensor do piezo. A nossa primeira validação será a certificação de que o intervalo está sendo iniciado quando invocamos o método fireAlarm.startPolling().

```javascript
  ...
  describe('#startPolling', function(){
    beforeEach(function(){
      this.piezoPlaySpy = this.sandbox.spy();

      var piezoStub = {
        isPlaying: false,
        play: this.piezoPlaySpy
      };
      this.sandbox.stub(fireAlarm, 'piezo', piezoStub);

      this.sandbox.spy(global, 'setInterval');
      fireAlarm.startPolling();
    });

    afterEach(function(){
      global.setInterval.restore();
    });

    it('should creates polling', function(){
      global.setInterval.calledOnce.should.be.true;
    });
  });
  ...
```

Para a validação do caso da temperatura acima do limite utilizaremos o método clock.tick com a informação contida no arquivo de configuração, simulando as alterações de horário e tempo no momento do teste, o que nos ajuda a forçar a chamada do evento de polling, que utiliza o setInterval.

Neste cenário utilizamos o stub para simular o retorno do sensor com o valor acima do aceitável e nos certificamos de que o sensor Piezo e a Api do Twilio foram acionados.

```javascript
    ...
    describe('When the temperature is up to the limit', function(){

      beforeEach(function() {
        clock = this.sandbox.useFakeTimers();

        this.sandbox.stub(fireAlarm, 'temperatureSensor', {
          celsius: CONFIG.FIRE_ALARM.LIMIT + 1
        });
        fireAlarm.startPolling();
        clock.tick(CONFIG.INTERVAL);
      });

      afterEach(function(){
        clock.restore();
      });

      it('should trigger piezo sensor alarm', function(){
        this.piezoPlaySpy.calledOnce.should.be.true;
      });

      it('should send the SMS to user', function() {
        this.createMessagesSpy.calledOnce.should.be.true;
      });

    });
    ...
```

Agora vamos validar o cenário da temperatura ambiente em níveis aceitáveis. As verificações utilizam as mesmas abordagens da anterior, mas neste caso nos certifamos de que o sensor piezo e a API do Twilio não foram acionadas.

```javascript
    ...
    describe('When the temperature is NOT up to the limit', function(){


      beforeEach(function() {
        clock = this.sandbox.useFakeTimers();


        this.sandbox.stub(fireAlarm, 'temperatureSensor', {
          celsius: CONFIG.FIRE_ALARM.LIMIT - 1
        });


        fireAlarm.startPolling();
        clock.tick(CONFIG.INTERVAL);
      });


      afterEach(function(){
        clock.restore();
      });


      it('should NOT trigger piezo sensor alarm', function(){
        this.piezoPlaySpy.calledOnce.should.be.false;
      });


      it('should NOT send the SMS to user', function() {
        this.createMessagesSpy.calledOnce.should.be.false;
      });


    });
    ...
```

Com base nos nossos cenários de teste, este é o conteúdo do teste do nosso arquivo fire alarm:

```javascript
var proxyquire = require('proxyquire');
var CONFIG = require('../src/configuration');
var five = require('johnny-five');
var request = require('request');
var sinon = require('sinon');

describe('FireAlarm', function() {

  beforeEach(function(){
    this.sandbox = sinon.sandbox.create();
    this.createMessagesSpy = this.sandbox.spy();
    var restClientMessage = {
      messages: {
        create: this.createMessagesSpy
      }
    };

    twilioMock = {
      RestClient: function() {
        return restClientMessage
      }
    };

    var FireAlarm = proxyquire('../src/fire-alarm', {
      twilio: twilioMock
    });
    fireAlarm = new FireAlarm();
  });

  afterEach(function() {
    this.sandbox.restore();
  });

  it('should have the termometer sensor configured', function(){
    (fireAlarm.temperatureSensor instanceof five.Thermometer).should.be.equal(true);
  });

  describe('#stopPolling', function(){
    beforeEach(function(){
      this.sandbox.spy(global, 'clearInterval');
      fireAlarm.stopPolling();
    });

    it('should remove interval', function(){
      global.clearInterval.calledOnce.should.be.true;
    });
  });

  describe('#startPolling', function(){
    beforeEach(function(){
      this.piezoPlaySpy = this.sandbox.spy();

      var piezoStub = {
        isPlaying: false,
        play: this.piezoPlaySpy
      };
      this.sandbox.stub(fireAlarm, 'piezo', piezoStub);

      this.sandbox.spy(global, 'setInterval');
      fireAlarm.startPolling();
    });

    afterEach(function(){
      global.setInterval.restore();
    });

    it('should creates polling', function(){
      global.setInterval.calledOnce.should.be.true;
    });

    describe('When the temperature is up to the limit', function(){

      beforeEach(function() {
        clock = this.sandbox.useFakeTimers();

        this.sandbox.stub(fireAlarm, 'temperatureSensor', {
          celsius: CONFIG.FIRE_ALARM.LIMIT + 1
        });
        fireAlarm.startPolling();
        clock.tick(CONFIG.INTERVAL);
      });

      afterEach(function(){
        clock.restore();
      });

      it('should trigger piezo sensor alarm', function(){
        this.piezoPlaySpy.calledOnce.should.be.true;
      });

      it('should send the SMS to user', function() {
        this.createMessagesSpy.calledOnce.should.be.true;
      });

    });

    describe('When the temperature is NOT up to the limit', function(){

      beforeEach(function() {
        clock = this.sandbox.useFakeTimers();

        this.sandbox.stub(fireAlarm, 'temperatureSensor', {
          celsius: CONFIG.FIRE_ALARM.LIMIT - 1
        });

        fireAlarm.startPolling();
        clock.tick(CONFIG.INTERVAL);
      });

      afterEach(function(){
        clock.restore();
      });

      it('should NOT trigger piezo sensor alarm', function(){
        this.piezoPlaySpy.calledOnce.should.be.false;
      });

      it('should NOT send the SMS to user', function() {
        this.createMessagesSpy.calledOnce.should.be.false;
      });
    });
  });
});
```

Com isso terminamos o nosso primeiro projeto com testes unitários, cobrindo todos os nossos possíveis cenários. No próximo capítulo veremos outros serviços que facilitam a nossa vida ao enviarmos o código para produção, tais como servidores de integração contínua, cobertura de código e de complexidade de maneira automatizada.
