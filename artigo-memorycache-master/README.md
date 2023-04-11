
# Artigo ASP.NET Core e cache de memória: melhorando a performance de sua API usando o IMemoryCache


Neste artigo, falarei sobre cache de memória, e como utilizá-la em uma aplicação ASP.NET Core.


Como tento sempre em meus artigos, ele vai começar apresentando o tipo de problema em que o cache de memória te ajudará. Após isso, discutirei suas características, e então apresentarei um exemplo de seu uso com ASP.NET Core.


## O cache de memória
Situação: Você tem uma API que retorna dados que são alterados com pouca frequência. Por exemplo, listagem de cidades ou países, tipos de algo relacionado ao negócio, entre outros. São APIs que são frequentemente requisitadas, realizam acesso ao banco de dados, e retornam dados pouco alterados.

Uma maneira de aumentar o desempenho dessas requisições, e ao mesmo tempo reduzir a quantidade de acessos ao banco de dados, é utilizar um cache de memória.

O cache de memória utiliza a memória do servidor para armazenar dados. Com isso, o acesso a esses dados é mais rápido do que ao banco de dados.

Porém, tem que se ter atenção de que ele é adequado para um servidor, já que em caso de vários servidores existe o risco de uma requisição atingir um servidor em que o dado não esteja na memória.

Mesmo assim, ainda em casos de múltiplos servidores, é possível utilizar o que é chamado de sticky sessions (ou sessões adesivas, traduzindo). Com isso, as requisições de clientes serão roteadas e processadas no mesmo servidor. Uma solução melhor para situações de múltiplos servidores seria um cache distribuído, como o Redis.

## O IMemoryCache
A interface utilizada para a gestão dos dados do cache de memória é a IMemoryCache. Para utilizá-la, basta fazer uma chamada do método AddMemoryCache, desde um objeto IServiceCollection.

Isso pode ser feito no método ConfigureServices, da classe Startup.


Adicionando serviço de cache de memória em sua aplicação
Com isso, a instância do serviço pode ser acessada via injeção de dependência em sua aplicação.

Os principais métodos a serem utilizados são:

- **Set(string key, TItem object):** cria uma entrada na cache de memória com a chave key e objeto de tipo genérico TItem, substituindo caso já exista um objeto na mesma chave;

- **Remove(string key):** remove o objeto guardado na cache, de chave key;
- **TryGetValue(string key, out object object):** tenta obter um objeto armazenado com chave key, e em caso de existir atribui à variável object;
Para o exemplo, utilizo a RestCountry, uma API pública sobre os países do mundo. Informações sobre ela podem ser encontrados aqui. Como essa API retorna MUITOS dados de cada país, escolhi algumas propriedades simples e criei a classe Country.

## Definição da classe Country
Em seguida, implemento a Action GetCountries, acessada pelo método HTTP GET para o caminho api/countries.


### Exemplo de API implementada utilizando o cache de memória
No exemplo acima, note o fluxo:

 - 1 Checo se existe um registro com chave especificada na constante COUNTRIES_KEY.
Se sim, retorno OK com a lista. Esse é o caminho mais rápido.
- 2 Em caso negativo, defino a URL para chamada HTTP.
Instancio um objeto HttpClient
- 3 Realizo a chamada para a URL definida, realizando a deserialização em uma lista de objetos Country em seguida.
- 4 Instancio as configurações de cache de memória, definindo um tempo absoluto de expiração (de 1 hora, ou 3600 segundos), e um tempo relativo cuja contagem vai sendo reiniciada a cada acesso (de 20 minutos, ou 1200). Com isso, com 20 minutos sem acesso algum a entrada na memória já é removida. Ou, em caso de muitos acessos, em 1 hora ela será removida de qualquer forma.
- 5 Salvo o objeto retornado pela API no cache de memória.
- 6 Retorno OK com a lista.





## Referência

[Conteúdo do canal do LuisDev](https://www.youtube.com/watch?v=93VuXFslkLE)

 [Blog de artigos](https://www.luisdev.com.br/2020/08/26/asp-net-core-e-cache-de-memoria-melhorando-a-performance-de-sua-api-usando-o-imemorycache/)

## Relacionados

 Documentação da interface IMemoryCache
[Documentação Microsoft](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.caching.memory.imemorycache)


