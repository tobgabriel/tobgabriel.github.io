---
layout: post
title:  "Rotas para Framework MVC - Parte 1"
date:   2021-12-29 11:07:38 -0300
categories: php mvc
tag: php mvc rotas
---
Este projeto tem como objetivo reproduzir um sistema de rotas para um futuro framework MVC.O projeto é baseado nos artigos do blog do [alxbbarbosa](https://github.com/alxbbarbosa 'alexbbarbosa'). Tentei refatora-lo da forma mais inteligível a mim.

## Sumário
___
* [Por que utilizar um sistema de Rotas?](#Por-que-utilizar-um-sistema-de-rotas?)
	* [URLs Amigáveis](###URLs-Amigáveis)
* [A classe RouteCollection](##A-classe-RouteCollection)
	* [Utilitários: parseUri](###Utilitários:-parseUri)
	* [Utilitários: definePattern](###Utilitários:-definePattern)

## Por que utilizar um sistema de Rotas?
___
### O protocolo HTTP
### URLs Amigáveis
<p sttyle='text-align: justify;'></p>

## A classe RouteCollection
___

Esta classe é responsável por prover meios de salvar e recuperar rotas. Dependendo da rota cadastrada retornará os parâmetros que o usuário forneceu na uri.

### Utilitários: parseUri

Remove '/' indesejável para armazenar a uri.Caso faça referência ao própio domínio retorna '/'

### Utilitários: definePattern

Prepara a uri para ser utilizado como expressão regular a ser buscada.

### Utilitários : checkUrl

<p align="justify">Este método é responsável por checar se a rota passada coincide com o padrão cadastrado. Coincidindo, o método que o solicita <b>getRoute</b> deve entender se é uma uri estática ou com parâmetros.
Se a uri contiver parâmetros o valor de retorno é 2,se for fixa retorna 1, senão coincidir retorna 0.</p>
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

<p align="justify">Este método cria um array associativo chave-valor de acordo com os parâmetros passado pelo usuário.Este método só é chamado por <b>getRoute</b> quando ele reconhece uma uri com parâmetros.</p>


### addRoute

Utilizado para salvar as rotas.

### getRoute

Utilizado para recuperar a rota e os parâmetros fornecidos pela uri do usuário.

1. Verifica se tem rotas cadastradas para o método;
2. Compara cada rota salva do método com a uri informada pelo usuário e decide:
	1. Se a rota for dinâmica obtém os parâmetros fornecidos pelo usuário e retorna juntamente com o callback;
	2. Se a rota for estática retorna apenas o callback;
	3. Se nenhuma rota for válida retorna um _false_;
