---
layout: post
title: 'Criando bons construtores em JavaScript'
tags:
  - JavaScript
---

Este texto não se trata de uma introdução a Orientação a Objetos, para isto, [este artigo da MDN serve melhor](https://developer.mozilla.org/pt-PT/docs/Javascript_orientado_a_objetos).

Tive o prazer de [palestrar sobre os paradigmas do JavaScript](https://speakerdeck.com/jcemer/o-fantastico-mundo-do-javascript) no [BrazilJS](http://braziljs.com.br) deste ano, este texto é um complemento com alguns *insights* sobre Orientação a Objetos e em especial: construtores.

## Inspiração

Costumo sempre ficar de olhos abertos para sugar ao máximo o que diferentes linguagens e suas comunidades tem a oferecer, este é meu maior conselho, absorva ao máximo.

Aprendi Ruby há alguns anos atrás. Na época, o que mais me chamou atenção, era que quase tudo pode se comportar como um objeto, assim como no JavaScript. Mas fica tranquilo, minha intenção aqui não é fazer com que você aprenda Ruby, só precisarei dele por alguns parágrafos para defender um ponto.

Nossa inspiração será uma versão exageradamente simplificada da principal classe responsável pelos *models* no [Ruby on Rails](http://rubyonrails.org).

~~~ ruby
class ActiveRecord::Base
  def initialize(attributes)
    assign_attributes(attributes)
  end
end
~~~

Considerando um *model* `User`, posso executar `User.new(name: 'Jean')` para instanciar um objeto. Neste caso, o método `initialize` acima é chamado e `name` é armazenado no objeto que acabei de criar.

**O ponto chave: nenhum efeito colateral é ou deve ser desencadeado com a simples instanciação de um objeto**. Esta execução, por exemplo, não salva este usuário no banco de dados ou o persiste de qualquer outra maneira que não seja como atributo do objeto recem criado.

------------

Adicionalmente, nossa classe possui o método `create`. Este método pode ser chamado ao invés do `new`, que é o construtor padrão.

~~~ ruby
class ActiveRecord::Base
  def self.create(attributes)
    object = new(attributes)
    object.save
    object
  end
end
~~~

Como você deve suspeitar, `User.create(name: 'Jean')` instancia um objeto e o salva no banco de dados. Sim, agora temos efeito colateral, mas isto é claro pois tenho um método específico que possui este comportamento documentado.

## Construtores ideais

O construtor ideal é aquele que não adiciona *listeners* de eventos ou elementos no DOM e muito menos dispara um `alert`. E é óbvio que isto não é tão simples e a maioria dos códigos não seguem esta regra.

Cuidado, bibliotecas como [Backbone](http://backbonejs.org), que é um dos *cases* mais fantásticos de herança em JavaScript que conheço, tentam inviabilizar esta abordagem para as views definidas por você. A documentação incentiva o uso da propriedade `events`, que associa *listeners* na instanciação da view. Tem também o método `this.listenTo` geralmente usado no `initialize`, outro que é chamado na instanciação.

Mas repare bem, todos os construtores do Backbone, se não extendidos, podem ser instanciados sem efeitos colaterais. Isto vale para `new Backbone.Model()`, `new Backbone.View()`, `new Backbone.History()`. Pode experimentar, faça estas chamadas no *console* do seu navegador quando estiver acessando o endereço http://backbonejs.org.

### Modelo

Vamos portar nossa [inspiração](#Inspiração) para JavaScript na forma de uma das aplicações mais comuns (e detestadas) de vermos em websites: um carrossel.

~~~ javascript
function Carousel(container) {
  this.container = $(container);
}

Carousel.init = function (container) {
  var instance = new this(container);
  instance.init();
  return instance;
};

Carousel.prototype.init = function () {
  this.addEventListeners();
  this.startAutomaticTransition();
};
~~~

Note, a única função do construtor é armazenar o *container* utilizado como base para o carrosel. O método de instância `init` é que irá disparar o comportamento. Um exemplo de uso.

~~~ javascript
var productsCarousel = new Carousel('[data-carousel="products"]');
productsCarousel.init();
~~~

Temos outra forma de uso com o mesmo resultado. Repare que desta vez não usamos o operador `new`.

~~~ javascript
Carousel.init('[data-carousel="products"]');
~~~

### Vantagens

A principal vantagem é poder **instanciar um objeto sem precisar se preocupar com efeitos colaterais**, este é o ganho.

Herança de construtores em navegadores modernos, indiscutivelmente, deve ser apoiada em [Object.create](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create#Classical_inheritance_with_Object.create). Uma técnica mais compatível, via *prototype*, pode ser alcançada com facilidade quando o construtor é ideal.

~~~ javascript
function CarouselWithLasers(container) {
  Carousel.call(this, container);
}
CarouselWithLasers.prototype = new Carousel();
~~~

Viabilizar os testes é outra grande vantagem em não ter comportamento definido no construtor. Desta forma, fica possível aplicar [stubs](http://sinonjs.org/docs/#stubs) no método `init` para poder testar e ser feliz.