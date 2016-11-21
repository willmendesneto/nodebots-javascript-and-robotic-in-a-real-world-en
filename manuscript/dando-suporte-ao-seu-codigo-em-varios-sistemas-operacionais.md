## Dando suporte ao seu código em vários sistemas operacionais

Nesta etapa do livro iremos então validar e verificar a cobertura de testes no nosso projeto em diferentes sistemas operacionais, além de habilitarmos diferentes serviços web para melhorias de workflow, como ferramentas de integração contínua e cobertura de código.

Esta etapa é muito importante, pois estas ferramentas nos auxiliam no processo de segurança do nosso código, checando diferentes critérios de aceitação da nossa aplicação de maneira automatizada.


### Adicionando servidores de integração contínua ao seu projeto


Como todo projeto de qualidade, o nosso projeto Nodebots vai se preocupar em alguns outros aspectos, como automatização da suíte de testes, build e outras tarefas relevantes ao nosso projeto.


Para isso vamos contar com o auxílio de um servidor de integração contínua. Existem vários no mercado, sendo gratuitos ou pagos, e nesta etapa do livro vamos conhecer um pouco mais do funcionamento e configuração de dois deles: Travis-CI e Appveyor.


#### Travis-CI: checando seu código no  Linux e OSX

Sabendo que atualmente os sistemas operacionais mais utilizados são Unix/Linux, Windows e OSX vamos criar verificações para cada um deles e para isto entra em cena o Travis-CI.


Ele é um dos mais famosos serviços de integração contínua e auxilia no processo de integração das novas funcionalidades ou correção de bugs do código do atual projeto em vários ambientes, podendo até mesmo efetuar o deploy para produção, caso todos os passos de validação estejam corretos.


Vamos então ao site oficial do projeto travis-ci e habilitar o acesso utilizando a nossa conta do Github. Clique no botão "Sign up" e habilite acesso aos seus repositórios.

Após esta etapa você será redirecionado para uma  nova página com todos os seus repositórios. Para adicionarmos um novo basta clicar no ícone "+" ao lado do texto "My Repositories".

Após esta etapa você será redirecionado para uma  nova página com todos os seus repositórios. Para adicionarmos um novo basta clicar no ícone "+" ao lado do texto "My Repositories".

Esta próxima etapa é bem simples já que a página possui um tutorial mostrando cada um dos passos para habilitar a integração do Travis-CI com o seu repositório no Github, como podemos observar na imagem abaixo.

Na mesma página, será listado todos os seus repositórios para que você escolha e habilite a integração do Travis-CI ao seu projeto. Para habilitar basta clicar no botão cinza com um "X" e quando ele mudar a cor para verde quer dizer que esta tudo correu como esperado e o seu repositório está sincronizado com o Travis-CI.

O Travis-CI é totalmente configurável e você pode adicionar informações das mais diversas, sendo elas desde comandos a serem invocados antes, durante ou depois do build, podendo até mesmo configurar os tipos de sistemas operacionais que as tarefas devem acontecer.

Estas configurações ficarão no arquivo .travis.yml que ficará na pasta raíz do nosso projeto. Vamos explicar um pouco mais sobre como configurar estas tarefas no Travis-CI.

Primeiramente, no arquivo .travis.yml, vamos adicionar o campo "os", com as devidas informações dos sistemas operacionais utilizados para os nossos testes.

```
...
os:
  - linux
  - osx
...
```

Adicionaremos também o campo `"node_js"`, aonde ficarão as nossas informações sobre as versões de NodeJS que as tasks deverão ser utilizados nas nossas tasks. No nosso caso adicionaremos somente uma versão, mas poderíamos adicionar várias outras com base nas nossas necessidades de suporte, por exemplo.

```
...
node_js:
  - '5.3.0'
...
```

O nosso servidor de integração contínua nada mais é do que um container com um sistema operacional completo. Sendo assim podemos também configurar variáveis de ambiente nele. Neste caso adicionaremos a variável NO_SERIALPORT_INSTALL, especificando que não devemos instalar o pacote "serialport" neste caso, pois trata-se de um teste que utiliza um mock de um board físico.


OBS: A idéia deste livro é de focar nos conceitos relacionados diretamente com Nodebots e integrações com o repositório javascript criado, por isso não explicarei sobre o conceito de containers. Caso queira saber mais sobre este conceito utilizado pelo Travis-CI, acesse a página oficial do projeto Docker.

```
...
env:
  - NO_SERIALPORT_INSTALL=1
...
```

Podemos também definir o conjunto de tarefas que serão utilizados antes e depois do nosso script do travis. Neste caso utilizaremos "before" para os comandos que devem ocorrer antes do nosso script principal e "after" para os comandos que devem ocorrer depois dos comandos travis, como vocês podem visualizar no trecho de código a seguir:

```
...
before_script:
  - 'npm install'


after_script:
  - 'make test'
...
```

Neste caso estamos instalando as nossas dependências e rodando os nossos testes. Tudo isto de uma maneira bem simples e bem definida. O conteúdo do nosso arquivo .travis.yml com todas as alterações será o seguinte:

```
language: node_js
os:
  - linux
  - osx
node_js:
  - '5.3.0'
before_script:
  - 'npm install'


after_script:
  - 'make test'
env:
  - NO_SERIALPORT_INSTALL=1
```

Podemos perceber que o build do Travis-CI agora é um pouco diferente, pois estamos rodando o mesmo setup nos sistemas operacionais Linux e OSX, identificados pelos ícones de cada sistema operacional.

Com a integração testada, vamos então colocar o badge do travis-ci no nosso arquivo README.md do repositório. Com isto você verá uma imagem com o status do build.

```
[![Build Status](https://travis-ci.org/willmendesneto/build-checker.png?branch=master)](https://travis-ci.org/willmendesneto/build-checker)
```

Com isto terminamos a nossa integração com o servidor de integração contínua Travis-CI e temos toda a nossa suite de testes rodando nos sistemas Linux e OSX. Nesta próxima etapa vamos configurar as mesmas tarefas, mas para serem verificadas no sistema operacional Windows, utilizando outro servidor de integração contínua chamado Appveyor.


#### Appveyor: checando seu código no Windows

Muitos projetos são desenvolvidos em sistemas operacionais baseados no Unix por padrão e adicionar suporte ao Windows era considerado um grande um desafio para alguns, pois montar um ambiente de teste do Windows não era algo trivial, exigindo a compra de licenças de software.

O serviço de integração contínua Appveyor é uma das soluç	oes usadas para testes de projetos hospedados no GitHub em ambientes Windows, facilitando este processo e assegurando que o nosso código é cross-platform, funcionando nos principais sistemas operacionais.

Adicionar o suporte do Appveyor ao nosso projeto é uma tarefa bastante simples. Vamos então visitar o site oficial do projeto e criar um login com as nossas informações.

Na página de login temos algumas opções listadas com suporte a alguns dos principais repositórios de código na internet. Nesta opção vamos utilizar o Github, para facilitar os próximos passos, mas vale lembrar que você pode utilizar quaisquer das opções suportadas para login.

Com o seu usuário criado e seu acesso funcionando, a página principal após o login é uma página listando todos os seus repositórios baseado em uma categoria localizada no lado esquerdo do site. No nosso caso, vamos escolher o projeto "build-checker" e clicaremos no botão "Add" para adicionarmos o suporte ao nosso projeto.

Veremos então a página do nosso projeto com as informações específicas do mesmo, atualmente.

Agora que já temos o nosso projeto configurado vamos criar então o nosso arquivo appveyor.yml, onde ficarão as nossas configurações de testes. Este arquivo é bem similar ao do Travis-CI em alguns aspectos.

O conteúdo do nosso arquivo appveyor.yml com todas as alterações será o seguinte:

Adicionaremos a versão utilizada do NodeJS no campo environment do nosso arquivo de configuração.

```
...
environment:
  matrix:
    - nodejs_version: "5.3.0"
...
```

O campo platform será utilizado para descrever as plataformas utilizadas. Percebam que neste caso ao invés de termos a diferenciação entre sistemas operacionais, temos entre plataformas do sistema operacional (x86 e 64x).

Isso é interessante também caso haja a necessidade de ter uma noção da diferença de desempenho, performance e outros aspectos da mesma aplicação em diferentes plataformas.

```
...
platform:
  - x86
  - x64
...
```

O campo install terá a lista dos nossos comandos iniciais. Perceba que o ps é um comando para instalarmos o NodeJS com a especificada no arquivo. Após esta etapa limpamos o cache usando o comando npm cache clean por medida de segurança, a fim de evitar possíveis falsos positivos nos nossos testes e, terminado este comando, vamos então instalar as nossas dependências utilizando o comando npm install.

```
...
install:
  - ps: Install-Product node $env:nodejs_version
  - npm cache clean
  - npm install
...
```

O campo test_script terá a lista dos nossos comandos a serem executados no momento de execução dos nossos testes. Estamos acessando diretamente a pasta node_modules e invocando os testes a partir delas com o comando `node_modules/.bin/istanbul cover node_modules/mocha/bin/_mocha -- -R dot`, pelo fato de utilizarmos o comando make test no nosso comando `npm test`.

```
...
test_script:
  # Run the test
  - cmd: node_modules/.bin/istanbul cover node_modules/mocha/bin/_mocha -- -R dot
```

Como o nosso caso não é necessário a criação de um build, vamos adicionar a informação no nosso arquivo com o valor `off` e vamos configurar o nosso build para ser finalizado o mais rápido possível adicionando o campo `fast_finish` com o valor `true`.

```
build: off
matrix:
  fast_finish: true
...
```

O conteúdo final do nosso arquivo appveyor.yml com todas as alterações será o seguinte:

```
# Fix line endings on Windows
init:
  - git config --global core.autocrlf true
environment:
  matrix:
    - nodejs_version: "5.3.0"
platform:
  - x86
  - x64
install:
  - ps: Install-Product node $env:nodejs_version
  - npm cache clean
  - npm install
test_script:
  # Run the test
  - cmd: node_modules/.bin/istanbul cover node_modules/mocha/bin/_mocha -- -R dot
build: off
matrix:
  fast_finish: true
```

Percebam que neste caso temos a lista dos nossos build diferenciada por plataformas na página de listagem.

Com a nova integração testada, vamos então atualizar o arquivo README.md do repositório com o badge do Appveyor. Com isto você verá uma imagem com o status do build, assim como o que já inserimos anteriormente.

```
[![Build Windows Status](https://ci.appveyor.com/api/projects/status/github/<nome-do-seu-usuario-ou-organização>/<nome-do-seu-repositório>?svg=true)](https://ci.appveyor.com/project/<nome-do-seu-usuario-ou-organização>/<nome-do-seu-repositório>/branch/master)
```

Perceba que temos duas tags neste trecho de código. Substitua estas informações da seguinte forma:

- `<nome-do-seu-usuario-ou-organização>`: nome do seu usuário ou organização;
- `<nome-do-seu-repositório>`: nome do seu repositório;

Por exemplo, baseado no repositório de exemplo, o nosso badge terá o seguinte conteúdo.

```
[![Build Windows Status](https://ci.appveyor.com/api/projects/status/github/willmendesneto/build-checker?svg=true)](https://ci.appveyor.com/project/willmendesneto/build-checker/branch/master)
```

Como vocês puderam perceber adicionar suporte a vários sistemas operacionais e plataformas é uma tarefa bastante simples com o Appveyor. Os próximos passos do livro serão mais voltados ao quesito de melhorias na automação da checagem da nossa cobertura de código.


### Code coverage para o seu código

Finalizada a comunicação do nosso repositório com os serviços de integração contínua, vamos agora adicionar novas ferramentas. Desta vez o foco é a cobertura de nosso código, checando se tudo está sendo validado corretamente de maneira automatizada antes mesmo de continuarmos com as outras etapas de desenvolvimento do nosso Nodebots.


#### Checando a cobertura de código do nosso projeto: Conhecendo o Istanbul


O istanbul é um pacote NodeJS para verificar a cobertura de código no nosso repositório utilizando vários parâmetros, como cobertura por linha de código, funções, declarações e engenharia reversa.

Vamos então adicionar este pacote ao nosso projeto utilizando o seguinte comando.

```bash
$ npm install --save-dev istanbul
```

Para verificarmos se ele está integrado no nosso repositório, basta digitarmos no nosso prompt/linha de comando.

```bash
$ ./node_modules/.bin/istanbul help
```

Como integramos anteriormente o MochaJS ao nosso repositório quando criamos os testes do projeto, podemos simplesmente digitar o seguinte comando no nosso prompt/linha de comando.

```bash
$ ./node_modules/.bin/istanbul cover ./node_modules/.bin/_mocha
```

O retorno será o mesmo da imagem abaixo. Podem reparar que agora temos algumas novas informações no rodapé das mensagens dos testes, tais com porcentagens de linhas, funções, branches e declarações de métodos, classes ou objetos.



Percebam que agora temos uma nova pasta chamada coverage com alguns arquivos e todas estas informações listadas na nossa linha de comando. Utilizaremos elas nos próximos passos para a integração com o serviço Coveralls.


#### Integrando o servidor de integração contínua com o coveralls

Com as informações de cobertura de código coletadas, vamos então integrar um novo serviço chamado coveralls. Ele será utilizado para integrar os dados de cobertura de código e deixar ele visível, adicionando um badge no nosso README.md.

O login é bem simples e você terá que habilitar a integração com o seu Github. Após esta etapa você verá uma lista com todos os seus repositórios cadastrados no Github. Clique no botão ao lado esquerdo do seu repositório listado e espere a mensagem `"Off"` transformar-se em `"On"`.

Perceba que, com o repositório ativado, temos agora um link para a página detalhes. Ao clicarmos neste link seremos direcionados para uma página com todas as informações iniciais para o setup do projeto no coveralls. Para a nossa solução usaremos a opção de adicionarmos a informação do coveralls no arquivo `.coveralls.yml`.

Vamos então copiar este conteúdo da opção do arquivo na página de setup e criaremos o novo arquivo no nosso projeto. Dentro do nosso repositório local, vamos digitar o seguinte comando via prompt/linha de comando.

```bash
$ touch .coveralls.yml
```

Vamos abrir este arquivo no nosso editor e adicionaremos o conteúdo dentro deste arquivo. Após esta etapa adicionaremos o pacote NodeJS à nossa lista de dependências de desenvolvimento para integrarmos a infraestrutura do coveralls ao nosso projeto digitando o seguinte comando.

```bash
$ npm install --save-dev coveralls
```

Assim que enviarmos um novo código, podemos perceber que teremos as informações de porcentagem de cobertura de código visível no site do coveralls, na área do nosso repositório. Com isto podemos acompanhar todas as variações de cobertura de código, criamos validações e muito mais.

Depois disso podemos adicionar um novo badge com as informações do code coverage do nosso projeto no arquivo README.md, contido no repositório do projeto. O padrão do badge é algo bem simples:

```
[![Coverage Status](https://coveralls.io/repos/<nome-do-seu-usuario-ou-organização>/<nome-do-seu-repositório>/badge.svg?branch=master)](https://coveralls.io/r/<nome-do-seu-usuario-ou-organização>/<nome-do-seu-repositório>?branch=master)
```

Perceba que temos duas tags neste trecho de código. Substitua estas informações da seguinte forma:

- `<nome-do-seu-usuario-ou-organização>`: nome do seu usuário ou organização;
- `<nome-do-seu-repositório>`: nome do seu repositório;


Por exemplo, baseado no repositório de exemplo, o nosso badge terá o seguinte conteúdo.

```
[![Coverage Status](https://coveralls.io/repos/willmendesneto/build-checker/badge.svg?branch=master)](https://coveralls.io/r/willmendesneto/build-checker?branch=master)
```

Após adicionarmos e salvarmos este código, o resultado final a ser renderizado será algo similar ao da imagem a seguir.

E com isto concluímos a nossa integração com o serviço do coveralls. Este é só um exemplo simples de uma das várias funcionalidades deste serviço e recomendo fortemente que dêem uma lida na documentação do coveralls para que vocês tenham uma maior sobre este serviço.


#### Verificando complexidade do código com o PlatoJS

PlatoJS é um pacote NodeJS que nos ajudará em algumas validações do nosso código nodebots. Ele cria um relatório utilizando alguns dados gerados via análise estática do código do nosso projeto que nos mostra algumas informações como complexidade do código, dificuldade de manutenção, linhas de código, possiveis erros de implementação, dentre outros dados relevantes.


OBS: Caso queira saber mais sobre o PlatoJS, por favor acesse o repositório no Github do projeto (https://github.com/es-analysis/plato)

Sua instalação é muito fácil. Basta digitarmos o comando:

```bash
$ npm install --save-dev plato
```

E feito esta etapa, o plato foi instalado localmente como dependência de desenvolvimento na nossa pasta node_modules do nosso projeto. Nosso próximo passo é adicionar um novo comando NPM. Agora nós teremos o comando code-analysis que vai acionar o plato ao nosso projeto.

```json
{
  ...
  "scripts": {
    "start": "nodemon ./src/index.js -e js,json --watch ./src",
    "test": "make test",
    "code-analysis": "plato -r -d report src test"
  },
  ...
}
```

E para acionarmos o PlatoJS, basta digitarmos:

```bash
$ npm run code-analysis
```

E após este comando será criada uma pasta de nome report com as informações da análise do nosso repositório.


Dentro desta pasta teremos vários arquivos com as informações retornadas da análise do PlatoJS que podemos visualizar mais detalhes acessando o arquivo index.html no nosso navegador.

Esta página terá informações de cada arquivo e gráficos mostrando dados como nível de complexidade e linhas de código, como podemos ver na figura abaixo.
