---
layout: post
title:  "Rotas MVC - Parte 1"
date:   2021-12-29 11:07:38 -0300
categories: php mvc
tag: php mvc rotas
---
<p align="justify">
Este projeto tem como objetivo reproduzir um sistema de rotas para um sitema MVC. Ele é baseado nos artigos do blog do <a href="https://github.com/alxbbarbosa">alxbbarbosa</a>. Tentei refatora-lo da forma mais inteligível a mim.
</p>

## Por que utilizar um sistema de Rotas?
___
### O protocolo HTTP
<p align="justify">
Conforme o <a href="https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Overview">MDN Web Docs</a> o protocolo HTTP é um protocolo de aplicação cliente-servidor que permite a obetenção de recursos,como documentos HTML.
Os <b>clientes</b>, geralmente navegadores web, se comunicam com os <b>servidores</b> através de mensagens individuais. O cliente envia uma <b>request</b> ao servidor que gera uma <b>response</b>.
Um documento pode ser reconstruído a partir de diversos outros sub-documentos como textos,imagens,vídeos e demais.
</p>
<div align="center" style="padding:2pt; margin-bottom:2pt;">
<img src="/assets/rotasmvc/http01.png"><legend>Créditos:MDN Web Docs.</legend>
</div>


<p align="justify">
O fluxo de comunicação do protocolo HTTP parte de uma conexão TCP que será utilizada para enviar requisições e receber respostas.O cliente pode abrir uma nova conexão, reusar uma conexão existente, ou abrir várias conexões aos servidores.
Depois, o cliente envia mensagens HTTP para o servidor e espera a resposta.
</p>

<div align="center" style="padding:2pt; margin-bottom:2pt;">
<img src="/assets/rotasmvc/http02.png"><legend>Créditos:MDN Web Docs.</legend>
</div>
<div align="center" style="padding:2pt; margin-bottom:2pt;">
<img src="/assets/rotasmvc/http03.png"><legend>Créditos:MDN Web Docs.</legend>
</div>

<p align="justify">
As requisições HTTP contém os seguintes elementos:
<ul>
<li>Um método HTTP,geralmente um verbo GET,POST,DELETE,PUT e etc, ou um substantivo como OPTIONS ou HEAD. O método define qual operação o cliente deseja realizar.
</li>
<li>
O caminho do recurso,geralmente representado por uma URL.
</li>
<li>
Cabeçalhos opcionais que contém informações adicionais para os servidores.
</li>
<li>
Ou um corpo de dados, para alguns métodos como POST, similares aos corpos das respostas, que contém o recurso requisitado.
</li>
</ul>
</p>

Respostas consistem dos seguintes elementos:

* A versão do protocolo HTTP que elas seguem.
* Um código de status, indicando se a requisição foi bem sucedida, ou não, e por quê.
* Uma mensagem de status, uma pequena descrição informal sobre o código de status.
* Cabeçalhos HTTP, como aqueles das requisições.
* Opcionalmente, um corpo com dados do recurso requisitado.


<p align="justify">
Os vídeos do <a href="https://www.youtube.com/watch?v=V4XZ81vRGtM">Programador a Bordo</a> são bem didaticos para entender o protocolo HTTP recomendo-os.
</p>
### URLs Amigáveis
<p align="justify">
Conforme o <a href="https://pt.wikipedia.org/wiki/URL">Wikipedia</a> uma URL pode ser definida como
</p>

>URL
```
esquema ou protocolo://domínio:porta/caminho/recurso?query_string#fragmento
```
>
* O esquema é o protocolo. Poderá ser HTTP, HTTPS, FTP etc.
* O domínio é o endereço da máquina: designa o servidor que disponibiliza o documento ou recurso solicitado.
* A porta é o ponto lógico no qual se pode executar a conexão com o servidor. (opcional)
* O caminho especifica o local (geralmente num sistema de arquivos) onde se encontra o recurso, dentro do servidor.
* A query string é um conjunto de um ou mais pares "pergunta-resposta" ou "parâmetro-argumento" (como por exemplo nome=fulano, em que nome pode ser, por exemplo, uma variável, e fulano é o valor (argumento) atribuído a nome). É uma string enviada ao servidor para que seja possível filtrar ou mesmo criar o recurso. (opcional)
* O fragmento é uma parte ou posição específica dentro do recurso. (opcional)
<p align="justify">
Basicamente um url amigável é uma URL facilmente memorizável,não extensa e que não utiliza query string para carregar funções.
</p>
>Exemplo
```
	http://wwww.meudominio.com/blog/post
```

<p align="justify">
Elas podem ser estáticas ou dinâmicas. Por dinâmicas estamos dizendo que existem variáveis que recebem parâmetros que podem alterar o que será exibido.
</p>

>Rota que lista usuários
```
http://www.meudominio.com/sistema/usuarios
```

>Rota para visualisar o cadastro de um usuário específico.
```
http://www.meudominio.com/sistema/usuario/{idDoUsuario}
```

<br/>
<p align="justify">
Tendo o conceito do protocolo HTTP em mente e urls amigáveis podemos construir um redirecionador de url para facilitar o tratamento das requisições por parte do servidor.
</p>

## Visão Geral do sistema
___
<p align="justify">A função básica do Roteador é: ao receber uma requisição HTTP e direciona-la para o devido <i>Controller</i> baseado na url fornecida.</p>
<p align="justify">Para isso o roteador utiliza um objeto <i>RouteCollection</i> para salvar e recuperar a coleção de rotas.
Uma vez recuperada a rota, o <i>Dispatcher</i> interpreta a ação entregando o controle para o devido <i>Controller</i> ou executando a ação.</p>

```
	app/
	|
	 ->src/
		|
		 ->Router/
			|
			 ->Router.php
			|
			 ->RouteCollection.php
			|
			 ->Dispatcher.php
			|
			 ->Route.php
		|
	         ->MVC/
			|
			 ->Controller.php
	|
	 ->public/
		|
		 ->index.php
		|
		 ->bootstrap.php
		|
		 ->.htaccess
	|
	 ->.htaccess						
			   
```

## A classe RouteCollection
___

<p align="justify">
Esta classe é responsável por prover meios de salvar e recuperar rotas. Dependendo da rota cadastrada retornará os parâmetros que o usuário forneceu na uri.
</p>
### Utilitários: parseUri

<p align="justify">
Remove '/' indesejável para armazenar a uri.Caso faça referência ao própio domínio retorna '/'. Fazemos isto para facilitar a busca através de expressôes regulares.
</p>

~~~php
parseUri( $uri ):string
~~~

### Utilitários: definePattern

Prepara a uri para ser utilizado como expressão regular a ser buscada.

~~~php
definePattern( $pattern ):string
~~~

### Utilitários : checkUrl

<p align="justify">Este método é responsável por checar se a rota passada coincide com o padrão cadastrado. Coincidindo, o método que o solicita <b>getRoute</b> deve entender se é uma uri estática ou com parâmetros.
Se a uri contiver parâmetros o valor de retorno é 2,se for fixa retorna 1, senão coincidir retorna 0.</p>

~~~php
checkURL( $route,$pattern ):int
~~~

<p align="justify">Para descobrir se existem parâmetros na rota primeiramente é necessário verificar a ocorrência de <i>{ }</i> com um nome válido.</p>
<div align="center">
<img src="/assets/rotasmvc/rgx01.jpg"><legend><i>Ocorrência de { } na uri cadastrada.</i></legend>
</div>
```php
	//Caso a uri contenha parâmetros
	if(preg_match('/\{[\w]+\}/',$pattern)){
	}
```
<p align="justify">O próximo passo é separar a parte fixa da uri cadastrada e verificar sua ocorrência exata na uri que estamos checando.</p>
<div align="center">
<img src="/assets/rotasmvc/rgx02.jpg"><legend>Comparação entre as duas partes fixas.</legend>
</div>
```php
	//Caso a uri contenha parâmetros
	if(preg_match('/\{[\w]+\}/',$pattern)){
		//Remove sujeira na rota a ser comparada
		$route= $this->parseUri($route);
		//$fixPart será usado como regex na busca 
		//primeiramente trate os / transformando-os em \/
		$fixPart=str_replace('/','\/',$pattern);
		//depois substitua a parte dinâmica da uri cadastrada
		//por um padrão que espera um nome válido
		$fixPart=preg_replace('/\{[\w]+\}/','[\w]+',$fixPart);
		//depois deve iniciar e terminar exatemente como previsto
		$fixPart="/^".$fixPart;
		//Se a expresão regular coincidir com a rota passada
		//previamente tratada
		if(preg_match($fixPart,$route)){
			return 2;
		}
	}
```
<p align="justify">Por exemplo com a uri cadastrada: <i>teste/{teste}/teste-{teste}</i> a expressão regular retornada será: <i>/^teste\/[\w]+\/teste-[\w]+$/</i> .</p>

<div align="center">
<img src="/assets/rotasmvc/rgx03.jpg"><legend>Busca parte fixa na uri passada.</legend>
</div>

<p align="justify">O próximo caso é se a uri for estática. Neste caso só preciso ver se a rota passada coincide com algum padrão cadastrado. Se não passar em nenhum dois dois testes então a rota não coincidiu.</p>
<div align="center">
<img src="/assets/rotasmvc/rgx04.jpg"><legend>Comparação de uri estática.</legend>
</div>
```php
	//Caso a uri contenha parâmetros
	if(preg_match('/\{[\w]+\}/',$pattern)){
		//Remove sujeira na rota a ser comparada
		$route= $this->parseUri($route);
		//$fixPart será usado como regex na busca 
		//primeiramente trate os / transformando-os em \/
		$fixPart=str_replace('/','\/',$pattern);
		//depois substitua a parte dinâmica da uri cadastrada
		//por um padrão que espera um nome válido
		$fixPart=preg_replace('/\{[\w]+\}/','[\w]+',$fixPart);
		//depois deve iniciar e terminar exatemente como previsto
		$fixPart="/^".$fixPart;
		//Se a expresão regular coincidir com a rota passada
		//previamente tratada
		if(preg_match($fixPart,$route)){
			return 2;
		}
	}elseif(preg_match($this->definePattern($route),$pattern)){
            return 1;
        }        
    return 0;
```
### Utilitários: getParams

<p align="justify">
Este método cria um array associativo chave-valor de acordo com os parâmetros passado pelo usuário.Este método só é chamado por <b>getRoute</b> quando ele reconhece uma uri com parâmetros.
</p>

~~~php
getParams( $route,$pattern ):array
~~~

<p align="justify">
Para criarmos a associação de parâmetros temos que identificar as chaves na uri cadastrada($pattern) e aponta-la para o valor na uri repassada($route).
Primeiramente precisamos apagar a parte fixa da uri repassada.
</p>

~~~php
function getParams($route,$pattern){
    //Variável que salva as partes fixas
    $fixedPartsToRemove=$pattern;
    //Removemos as partes dinâmicas
    $fixedPartsToRemove=preg_replace('/\{[\w]+\}/','|',$fixedPartsToRemove);
    //Tratamos a rota repassada
    $values=$this->parseUri($route);
    //Removemos cada parte fixa da rota colocando um separador
    foreach(explode('|',$fixedPartsToRemove) as $part){
		$values=str_replace($part,'|',$values);
    }
    //Removemos seperadores indesejáveis no começo e final
    // depois criamos um array de valores    
    $values=explode('|',trim($values,'|'));
}
~~~

<p align="justify">
Agora precisamos criar outro array com as chaves. Para isso utilizaremos a seguinte expressão regular para conseguir os nomes dentro dos { }
</p>

<div align="center">
<img src="/assets/rotasmvc/rgx05.jpg"><legend>Busca as chaves da uri dinâmica.</legend>
</div>

~~~php
function getParams($route,$pattern){
    //Variável que salva as partes fixas
    $fixedPartsToRemove=$pattern;
    //Removemos as partes dinâmicas
    $fixedPartsToRemove=preg_replace('/\{[\w]+\}/','|',$fixedPartsToRemove);
    //Tratamos a rota repassada
    $values=$this->parseUri($route);
    //Removemos cada parte fixa da rota colocando um separador
    foreach(explode('|',$fixedPartsToRemove) as $part){
		$values=str_replace($part,'|',$values);
    }
    //Removemos seperadores indesejáveis no começo e final
    // depois criamos um array de valores    
    $values=explode('|',trim($values,'|'));
    //Busca os nomes das chaves
    preg_match_all('/(?!\{)[\w]+(?=\})/',$pattern,$matches);
    //Combina as arrays para criar o array chave valor
    $params=array_combine($matches[0],$values);
    return $params; 
}
~~~

### addRoute

<p align="justify">
Método público utilizado para salvar as rotas.
</p>

~~~php
addRoute( $method,$route,$action )
~~~
<p align="justify">
Primeiramente utilizamos o método utilitário <b>parseUri</b> para tratar o uri que será salvo. Depois decidimos baseado no método em qual chave será salvo.
</p>


>Padrão de armazenamento
~~~php
$routes['MÉTODO']['URI'] = 'AÇÃO'
$routes['MÉTODO']['URI-DINÂMICA' = 'exemplo/{exemplo}'] = 'AÇÃO'
$routes['MÉTODO']['URI'] = 'namespace@controller@method'
$routes['MÉTODO']['URI-DINÂMICA' = 'exemplo/{exemplo}'] = 'namespace@controller@method'
~~~

<p align="justify">
O padrão da ação <b>namespace@controller@method</b> será usado para referenciar um método de um controller.
</p>

~~~php
addRoute(string $method, string $route, $action){
		//Tratamos a rota
        $route = $this->parseUri($route);
        //Decidimos em qual método salvar
        switch ($method) {
            case "GET":
                $this->routes["GET"][$route] = $action;
                break;
            case "POST":
                $this->routes["POST"][$route] = $action;
                break;
            case "PUT":
                $this->routes["PUT"][$route] = $action;
                break;
            case "DELETE":
                $this->routes["DELETE"][$route] = $action;
                break;
            default:
                throw new \Exception(
                    "ERROR 0x0001:cadastro de requisições para este método não implementada."
                );
        }
    }
~~~

### getRoute

<p align="justify">
Utilizado para recuperar a rota e os parâmetros fornecidos pela uri do usuário. Retorna um objeto a ser tratado pela classe <b>Router</b>.
</p>

~~~php
getRoute( $method, $route ):object stdClass
~~~

<p align="justify">
Para verificar se existe rota cadastrada primeiro verifica-se a existência de rotas cadastradas para o método, depois comparamos cada rota salva do método com a uri informada pelo usuário e decide-se:
se a rota for dinâmica obtém os parâmetros fornecidos pelo usuário e retorna juntamente com o callback, se a rota for estática retorna apenas o callback, se nenhuma rota for válida retorna um _false_.
</p>

~~~php
function getRoute($method, $route){
        $result = new \stdClass;
        //Se existe rota cadastrada para o método
        if (array_key_exists($method, $this->routes)) {
		//Para cada rota cadastrada do método
            foreach ($this->routes[$method] as $existent_path => $callback) {
		//escolha de acordo com o retorno da função checkUrl
		// e monte o objeto de retorno
                switch ($this->checkUrl($route, $existent_path)) {
                    case 2:
                        $result->params = $this->getParams(
                            $route,
                            $existent_path
                        );
                        $result->callback = $callback;
                        return $result;
                        break;
                    case 1:
                        $result->callback = $callback;
                        return $result;
                        break;
                    case 0:
                        continue;
                        break;
                    default:
                        throw new \Exception(
                            "ERROR 0x0002:falha na recuperação da rota."
                        );
                }
            }
        }
        return false;
    }
} 
~~~
