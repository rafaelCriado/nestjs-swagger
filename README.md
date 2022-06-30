# OpenAPI(Swagger) + NestJS

O Swagger (OpenApi) é uma biblioteca que gera um site interno no seu backend que descreve, com muitos detalhes, cada endpoint e estrutura de entidades presentes na sua aplicação atraveś de especificações.

A OpenAPI Specification (OAS) define uma descrição de interface padrão e independente de linguagem de programação para APIs HTTP, que permite que humanos e computadores descubram e compreendam os recursos de um serviço sem exigir acesso ao código-fonte, documentação adicional ou inspeção do tráfego de rede . Quando definido corretamente via OpenAPI, um consumidor pode entender e interagir com o serviço remoto com uma quantidade mínima de lógica de implementação. Semelhante ao que as descrições de interface fizeram para programação de nível inferior, a especificação OpenAPI remove suposições ao chamar um serviço.

Além disso, também oferece uma interface para que a API seja testada, sem precisar de um cliente HTTP externo (Postman, Insomnia, Thunder Client, etc) e, principalmente, com configuração ZERO!

Como de costume, o Nest resolve toda a parte chata da configuração e a gente só precisa declarar algumas linhas de código para integrar essa poderosa API.

## Instalação

- `@nestjs/swagger`
- `swagger-ui-express`

```bash
npm install --save @nestjs/swagger swagger-ui-express
```

> Também é possível usar o Swagger no Nest com o Fastify, basta usar a lib do swagger `fastify-swagger`

## Configuração

Abra o arquivo `main.ts` e adicione o seguinte conteúdo:

`src/main.ts`

```typescript
import { NestFactory } from '@nestjs/core';
import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const config = new DocumentBuilder()
    .setTitle('Documentação com Swagger')
    .setDescription(
      'O Swagger (OpenApi) é uma biblioteca que gera um site interno no seu backend que descreve, com muitos detalhes, cada endpoint e estrutura de entidades presentes na sua aplicação.',
    )
    .setVersion('1.0')
    .addTag('users')
    .build();

  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api', app, document);

  await app.listen(3000);
}

bootstrap();
```

## Acessando

Execute a aplicação

```bash
npm run start:dev
```

Abra a URL no seu navegador

http://localhost:3000/api/

Pronto!

## Tags

As tags servem para organizar endpoints.

Elas são declaradas no `main.ts` e utilizadas nos `controllers`.

### Vamos gerar um recurso

- `users`

```bash
nest g resource users
```

Assim que os arquivos forem gerados, basta marcar o `UsersController` com a tag `users`. Para isso, utilizamos o decorator `@ApiTags('tag')` que vem com a biblioteca `@nestjs/swagger`, da seguinte forma:

`src/users/users.controller.ts`

```typescript
@ApiTags('users')
@Controller('users')
export class UsersController {
  // Endpoints
}
```

Salve o arquivo para que o Nest recarregue a aplicação automaticamente e atualize a página do swagger.

## Testando requisições

Para testar a requisições, basta clicar em cima de um dos endpoints desejados para visualizar a sua estrutura.

Clique no botão `Try it out` e depois em `Execute` para realizar a requisição HTTP.


## Detalhando um pouco melhor os endpoints

Alguns endpoints, como o `PATCH`, que atualiza uma entidade, possuem parâmetros e uma estrutura de `Body` que deve ser enviada.

Os parâmetros primitivos (`string`, `number`, etc) são identificados de forma automática pelo Swagger, entretanto, os parâmetros mais complexos (objetos customizados, etc) precisam ser detalhados.

Vamos construir a estrutura de `CreateUserDto` e detalhar cada propriedade para deixar nossa aplicação mais informativa.

`src/users/dto/create-user.dto.ts`

```typescript
export class CreateUserDto {
  name: string;

  email: string;

  password: string;
}
```

Para que o Swagger entenda as propriedade de um objeto, precisamos usar alguns `decorators`.

Para dados primitivos (`string`, `number`, etc), utilizaremos o `decorator` `@ApiProperty()`, adicionando dados de exemplo na propriedade `example` e a descrição do campo na propriedade `description`.

`src/users/dto/create-user.dto.ts`

```typescript
import { ApiProperty } from '@nestjs/swagger';

export class CreateUserDto {
  @ApiProperty({
    example: 'Rafael Criado',
    description: `O nome será utilizado para qualquer coisa (Perfil, Home Page, etc) que precise exibir informações da pessoa conectada.`,
  })
  name: string;

  @ApiProperty({
    example: 'email@email.com',
    description: `O e-mail é necessário apra o login, mas não necessariamente precisa ser o mesmo e-mail da rede social que estiver conectada. Login sem rede social precisa de uma senha.`,
  })
  email: string;

  @ApiProperty({
    example: '123@abc',
    description: `É possível conectar com redes sociais sem uma senha, mas para login usando o e-mail diretamente é necessário informar uma senha.`,
  })
  password?: string;
}
```
## Melhorando a declaração das entidades

Em vez de utilizar os `decorators` para anotar cada propriedade, você pode adicionar comentários antes de cada propriedade e o Swagger irá utilizar as informações dos comentários para exibir na documentação de endpoints.

Dessa forma:

`src/users/dto/create-user.dto.ts`

```typescript
export class CreateUserDto {
  /**
   * O nome será utilizado para qualquer coisa (Perfil, Home Page, etc) que precise exibir
   * informações da pessoa conectada.
   * @example Rafael Criado
   */
  name: string;

  /**
   * O e-mail é necessário apra o login, mas não necessariamente precisa ser o mesmo e-mail da
   * rede social que estiver conectada. Login sem rede social precisa de uma senha.
   * @example email@email.com
   */
  email: string;

  /**
   * É possível conectar com redes sociais sem uma senha, mas para login usando o e-mail diretamente
   * é necessário informar uma senha.
   * @example 123@abc
   */
  password?: string;
}
```

Para isso, precisamos ativar a opção `introspectComments` de configuração do `@nestjs/swagger` no arquivo `nest-cli.json`.

`nest-cli.json`

```json
{
  "collection": "@nestjs/schematics",
  "sourceRoot": "src",
  "compilerOptions": {
    "plugins": [
      {
        "name": "@nestjs/swagger",
        "options": {
          "classValidatorShim": false,
          "introspectComments": true
        }
      }
    ]
  }
}
```

> Após fazer isso, precisamos encerrar a aplicação do Nest e executar novamente, para que o mecanismo seja ativado por completo.

E, pronto! Teremos o mesmo resultado 🚀

## Próximos passos

Para configurar uma documentação completa, será necessário a utilização de vários outros tipos e especificações mais detalhadas.

Para isso, consulte os seguintes links da documentação:

- Tipos e parâmetros: https://docs.nestjs.com/openapi/types-and-parameters
- Tipos mapeados: https://docs.nestjs.com/openapi/mapped-types

> Também é possível criar documentações separadas, incluindo apenas alguns `Modules` em cada documentação.
>
> Isso é útil para quando um conjunto de endpoints é interessante para uma determinada aplicação de Frontend Web, e outros endpoints são interessantes para uma aplicação de Frontend Mobile, mas que utilizam o mesmo backend.
>
> Confira a implementação em https://docs.nestjs.com/openapi/other-features

## Conclusão

Curtiu? Ficou fácil incluir uma documentação bem robusta nas nossas aplicações, não?!

Agora não tem mais desculpa pra não gerar uma boa documentação de endpoints.

🧡
