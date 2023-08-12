Descrição do projeto
Para um aplicativo em nível de produção, é necessário suporte para vários ambientes. Por exemplo, você precisará usar um banco de dados diferente para ambientes diferentes. Existem muitos usos e esse é apenas um deles.

Hoje veremos como podemos gerenciar múltiplos ambientes em um aplicativo NodeJS.

Abordagem tradicional
A maneira mais direta de usar variáveis de ambiente é usando nosso arquivo package.json.
Vamos criar dois comandos apenas para demonstrar 2 ambientes.

json

"scripts": {
    "start-dev": "PORT=3000 ts-node-dev --respawn src/index.ts",
    "start-prod": "PORT=4000 ts-node-dev --respawn src/index.ts"
}
Definimos dois comandos. Para desenvolvimento, definimos start-dev, onde a variável PORT deve ter o valor 3000. E para produção, será 4000.

Se executarmos nosso aplicativo

sh

yarn start-dev
Você verá a seguinte mensagem.

json

Conectado com sucesso na porta 3000
E executar start-prod resultará em

sh

Conectado com sucesso na porta 4000
Portanto, conseguimos tornar nosso aplicativo configurável com sucesso.

Vamos dar um passo adiante
E se precisarmos de outra variável que mudará em diferentes ambientes? Podemos simplesmente adicionar outra variável a essa string.

json

"scripts": {
    "start-dev": "PORT=3000 SOMETHING_ELSE=dev ts-node-dev --respawn src/index.ts",
    "start-prod": "PORT=4000 SOMETHING_ELSE=dev ts-node-dev --respawn src/index.ts"
}
aqui, SOMETHING_ELSE será acessível por meio de process.env.SOMETHING_ELSE

Mas na vida real, temos muitos ambientes que precisam ser alterados. Como gerenciamos isso?

Vamos criar um arquivo de ambiente
Para gerenciar muitas variáveis de ambiente, podemos usar um arquivo de ambiente. Vá para o seu projeto e crie um arquivo .env e cole o seguinte nele.

json

PORT=3000
SOMETHING_ELSE=dev
e remova os ambientes dos comandos package.json.

json

"scripts": {
    "start-dev": "ts-node-dev --respawn src/index.ts",
    "start-prod": "ts-node-dev --respawn src/index.ts"
}
E vamos executar!

sh

yarn start-dev
Isso resultará na seguinte saída.

sh

Conectado com sucesso na porta 5000
Agora isso é estranho. Porque deveríamos ver a porta 3000. Mas está nos dando 5000.

A razão é que, embora tenhamos adicionado as variáveis de ambiente, o NodeJS não as lerá automaticamente.

Vamos ler nossas variáveis de ambiente
Temos um pacote muito popular para fazer isso por nós. O nome é dotenv

Vamos instalá-lo primeiro.

sh

yarn add dotenv
E importe-o dentro do nosso aplicativo como a seguir.

js

import 'dotenv/config';
ou assim

js

import dotenv from 'dotenv';
const configuration: any = dotenv.config().parsed;
E isso é suficiente. Você não precisa fazer mais nada para ler os arquivos de ambiente. Este pacote tornará as variáveis disponíveis para nosso aplicativo por meio de process.env

Para testar isso, você pode executar

sh

yarn start-dev
e verá o aplicativo sendo executado na porta 3000 novamente.

Múltiplos ambientes
Agora vamos deixar nosso aplicativo pronto para produção. vamos criar 2 arquivos separados para ambientes diferentes.

Crie arquivos de acordo com o ambiente, como .env.development e .env.production

Agora precisamos carregar os arquivos durante a inicialização. Ambientes Windows às vezes enfrentam problemas ao carregar os ambientes. Para resolver isso, vamos instalar um pacote chamado cross-env

sh

yarn add -D cross-env
Em seguida, crie scripts separados para eles dentro do package.json

json

"dev": "cross-env NODE_ENV=development ts-node-dev --respawn src/index.ts",
"prod": "cross-env NODE_ENV=production ts-node-dev --respawn src/index.ts"
Em seguida, modifique o arquivo de configuração assim:

js

import dotenv from 'dotenv';
dotenv.config({ path: __dirname + `/../../.env.${process.env.NODE_ENV}` }); // altere de acordo com sua necessidade

const config = {
  port: process.env.APPLICATION_PORT,
  dbUrl: process.env.DB_URL,
  dbPassword: process.env.DB_PASSWORD,
};

export default config;
E agora podemos usar a configuração em todo o nosso aplicativo.

js

import Config from './utils/Configuration';

console.log(Config.port);
Isso é tudo! Agora você pode ter quantos ambientes desejar e lidar com eles facilmente.

Palavras finais
Não se esqueça de adicionar os arquivos .env ao seu arquivo .gitignore. Caso contrário, todos os seus segredos serão expostos.

Abra seu arquivo .gitignore e adicione o seguinte

sh

.env.development
.env.production
É isso aí.

Repositório no GitHub