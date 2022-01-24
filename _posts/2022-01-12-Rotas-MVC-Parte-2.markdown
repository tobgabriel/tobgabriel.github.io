---
layout: post
title:  "Rotas MVC - Parte 2"
date:   2022-01-12 09:20:38 -0300
categories: php mvc
tag: php mvc rotas
---
<p align="justify">
Neste post iremos detalhar as demais classes do sistema,o singleton utilizado e o helper.
</p>

## A classe Request
___

<p align="justify">
A função desta classe é capturar a url e dados passados pelo usuário que serão consumidos pelo Router. No momento do acesso ao domínio um objeto <i>Request</i> é instanciado capturando as informações da requisição.
</p>

### Atributos

A classe prevê três atributos:

* **uri** salva a uri fornecida pelo usuário
* **method** o método HTTP da requisição
* **data** objeto que salva os dados repassados pelo método POST e GET

### O método setData

<p align="justify">
Este método é por capturar os dados repassados por meio dos métodos POST ou GET e salva-los no atributo data.
</p>

~~~php
private function setData(){
        switch($this->method){
            case 'POST':
                $this->data=$_POST;
                break;
            case 'GET':
                $this->data=$_GET;
                break;
        }  
    } 
~~~

### O método filter_uri

<p align="justify">
Este método é responsável por remover uma possível <i>query string</i> da uri repassada pelo usuário.
</p>

<div align="center">
<img src="/assets/rotasmvc/rgx06.jpg"><legend>Filtra query strings da uri.</legend>
</div><br/>

~~~php
private function filter_uri($previousUri){
        return preg_replace('/\?([\w\W\s]+(=|\&)?)+/i','/',$previousUri);
    }
~~~

### Os métods uri e method data

São métodos acessores dos respectivos atributos uri,method e data.

~~~php
    public function uri(){
        return $this->uri;
    }
    public function method(){
        return $this->method;
    }
    public function data(){
        return $this->data;
    }
~~~

### O método construtor

Inicializa os atributos na quando a instância é criada.

~~~php
    public function __construct(){
        $this->method=strtoupper($_SERVER['REQUEST_METHOD']);
        $this->uri=$this->filter_uri($_SERVER['REQUEST_URI']);
        $this->setData();
    }
~~~

## A classe Router
___

<p align="justify">
Classe que agrega os outros objetos afim de orquestrar o processamento. Será instanciada como um singleton. Instancia um objeto do tipo RouteCollection e Dispatcher na sua construção.
</p>

### Utilitários: get put update delete

<p align="justify">
Método de cadastro de rota para o verbo especifico, chama o método addRoute de RouteCollection e cadastrada a rota passada.
</p>

### O método find

<p align="justify">
Resolve a requisição instanciada em request buscando a ação cadastrada para a rota caso não encontre retorna um <i>Not Found</i>, caso encontre se houver dados acrescenta o valor ao objeto que será entrega ao despachante.
</p>

~~~php
public function find($request){
        try {
            $result = $this->route_collection->getRoute($request->method(), $request->uri());
            if (!$result) {
                echo "Página não encontrada";
            } else {
                if (isset($request->data)) {
                    $result->data = $request->data;
                }
                $this->dispatcher->dispatch($result);
            }
        } catch (Exception $e) {
            $e->getMessage();
        }
    }
~~~

## A classe Dispatcher
___

<p align="justify">
Esta classe é responsável por interpretar a ação salva, dependendo do formato salvo podemos ter um <i>callable</i> ou um caminho para um método de um controller.
</p>
<p align="justify">
Para extrair informação da classe controller alvo o <i>Dispatcher</i> utiliza-se da classe <b>ReflectionClass</b>. Com ela podemos extrair informações de uma classe

</p>
