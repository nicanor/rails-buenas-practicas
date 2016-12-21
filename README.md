# Snappler Rails Best Practices

Este documento pretende convertirse en:

1. Una guía colaborativa de mejores prácticas para desarrollar aplicaciones Rails en _Snappler_
2. Una guía de estilos para ayudarnos a escribir código de una forma consistente.

-----------------------------

# Documentación


El archivo **README.md** debe contener la información necesaria para que
un desarrollador nuevo pueda comenzar a ser productivo cuanto antes.

Tiene que estar escrito en [formato markdown de Github](https://guides.github.com/features/mastering-markdown/)

Por lo menos tiene que contar con la siguiente información.

1. En qué consiste la aplicación.
2. Pasos a seguir para instalar y usar la aplicación.
3. Estructura de la aplicación.
4. Otras explicaciones que puedan ser de utilidad al nuevo desarrollador.

Ejemplo de un readme [aquí](EJEMPLO_README.md).

[Aspectos técnicos](https://jacobian.org/writing/technical-style/) de una buena documentación.


------------------------------

# Guía de estilos

El código es leido mucho más frecuentemente de lo que es escrito.

Con este documento se pretende mejorar la legibilidad del código.

Una guía de estilos es acerca de consistencia, y ser consistentes y prolijos nos permite escribir código fácil de mantener y de compartir entre nosotros.

Nos basaremos en la [guía de estilos de Ruby de bbatsov](https://github.com/bbatsov/ruby-style-guide).

Hay casos en los que no es posible seguir las recomendaciones de una guía de estilos.

En esos casos es recomendado usar el criterio propio.

Particularmente, no destruyas compatibilidad de tu código sólo para adaptarte a esta guía.


-------------------------------

# Prácticas en Rails

A continuación listaremos algunos **malos olores** comunes en aplicaciones Rails,
y cómo se pueden evitar.


## Rutas RESTful

En Rails se tiende a favoreces una arquitectura RESTful.

Una ruta RESTful se logra a través de los helpers **resources** o **resource**, que proveen un mapeo entre distintos verbos HTTP y URLs a acciones de un controlador

Además de estas rutas, Rails provee soporte para agregar rutas arbitrarias

Si ves en tu código varias rutas definidas de la siguiente manera, estás en presencia de un mal olor:

``` ruby
post 'books/send_report', to: 'books#send_report', as: :send_report
get 'books/:id/confirmation', to: 'books#confirmation', as: :book_confirmation
get 'books/:id/edit_confirmation', to: 'books#edit_confirmation', as: :edit_book_confirmation
patch 'books/:id/edit_confirmation', to: 'books#update_confirmation', as: :update_book_confirmation
resources :books
```

Una mejora es definir las rutas usando los métodos **member** y **collection**:

``` ruby
resources :books do
  get :send_report, on: :collection
  member do
    get :confirmation
    get :edit_confirmation
    patch :update_confirmation
  end
end
```

Pero, como dice en las guías de Rails:

> Si te encontrás agregando muchas acciones extra a una ruta RESTful, es un buen momento para preguntarte si no estás disfrazando la presencia de otro recurso:

``` ruby
resources :books do
  get :send_report, on: :collection
  resource :confirmation, only: [:show, :edit, :update]
end
```

## Route concerns

Es común en aplicaciones rails tener archivos de rutas que repitan patrones para diferentes recursos:

``` ruby
get 'pages/all_images', to: 'pages#all_images', as: :pages_order_images
post 'pages/:id/upload_image', to: 'pages#upload_image', as: :page_upload_image
delete 'pages/:id/delete_image', to: 'pages#delete_image', as: :page_delete_image
resources :pages

get 'articles/list', to: 'articles#list', as: :articles_list
resources :articles, only: [:show, :create, :edit, :update]

get 'books/list', to: 'books#list', as: :books_list
resources :books

get 'tags/list', to: 'tags#list', as: :tags_list
get 'tags/all_images', to: 'tags#all_images', as: :tags_order_images
post 'tags/:id/upload_image', to: 'tags#upload_image', as: :tag_upload_image
delete 'tags/:id/delete_image', to: 'tags#delete_image', as: :tag_delete_image
resources :tags

get 'gifts/list', to: 'gifts#list', as: :airports_list
get 'gifts/all_images', to: 'gifts#all_images', as: :gifts_order_images
post 'gifts/:id/upload_image', to: 'gifts#upload_image', as: :gift_upload_image
delete 'gifts/:id/delete_image', to: 'gifts#delete_image', as: :gift_delete_image
resources :gifts
```

En estos casos es una buena idea usar [concerns](http://guides.rubyonrails.org/routing.html#routing-concerns) para limpiar las partes repetidas.

``` ruby
concern :listable do
  get :list, on: :collection
end

concern :imageable do
  get :order_images, on: :collection
  member do
    post :upload_image
    delete :delete_image
  end
end

resources :articles, concerns: [:listable], only: [:show, :create, :edit, :update]
resources :books   , concerns: [:listable]
resources :gifts   , concerns: [:listable, :imageable]
resources :pages   , concerns: [:imageable]
resources :tags    , concerns: [:listable, :imageable]
```


## Scopes

Es mejor que los scopes en los modelos estén definidos en la clase correspondiente.

**Antes:**
``` ruby
class Blog
  #...
  def recipe_pages
    pages.where('pages.page_type = :q', q: 'Recipe')
  end

  def article_pages
    pages.where('pages.page_type = :q', q: 'Article')
  end
end
```

**Después:**
``` ruby
class Blog
  #...
  def recipe_pages
    pages.recipe
  end

  def article_pages
    pages.article
  end

end

class Page
  #...
  scope :recipe , -> {where(page_type: 'Recipe')}
  scope :article, -> {where(page_type: 'Article')}
end
```

De esta forma un modelo no necesita saber nada sobre la estructura interna de otro modelo.

Y se cumple la [ley de demeter](https://en.wikipedia.org/wiki/Law_of_Demeter#In_object-oriented_programming)

## Usar View Helper Methods

Rails provee varios métodos auxiliares para poder escribir en las vistas de forma segura.

Ejemplos de estos métodos son **tag** y **content_tag**.

Siempre que sea posible usar esos métodos en lugar de escribir las etiquetas a mano y usar _.html_safe_.

**Antes:**
``` ruby
def display_status(status)
  case status
    when 'En preparación'
      "<span class='label label-default'> #{I18n.t('project_status.incoming')} </span>".html_safe
    when 'Activo'
      "<span class='label label-success'> #{I18n.t('project_status.active')} </span>".html_safe
    when 'Cerrado'
      "<span class='label label-danger'> #{I18n.t('project_status.closed')} </span>".html_safe
    end
end
```

**Después:**
``` ruby
def display_status(status)
  case status
    when 'En preparación'
      content_tag :span, I18n.t('project_status.incoming'), class: 'label label-default'
    when 'Activo'
      content_tag :span, I18n.t('project_status.active'), class: 'label label-success'
    when 'Cerrado'
      content_tag :span, I18n.t('project_status.closed'), class: 'label label-danger'
  end
end
```

Más info [aquí](https://bibwild.wordpress.com/2013/12/19/you-never-want-to-call-html_safe-in-a-rails-template/)


## Gemfile

Revisar que se estén usando todas las gemas y eliminar las que no se usen.
Es muy común ver definidas en los proyectos las gemas **byebug** y **pry**, o las gemas **kaminari** y **will_paginate**.


## Abuso de self
Es mejor no usar _self_ innecesariamente. Esto:
``` ruby
  self.flight_type.eql?('arrival') ? self.date_in : self.date_out
```

Debería ser:
``` ruby
  flight_type.eql?('arrival') ? date_in : date_out
```

## Asignaciones dentro de expresiones en if y case

Salvo que explicitamente se utilize un _return_, los ifs y los case retornan siempre la última expresión.

**Antes:**
``` ruby
def something
  hour = nil
  if type.eql?('awesome')
    if eta_lt
      hour = "#{eta_lt} LT"
    elsif eta_utc
      hour = "#{eta_utc} UTC"
    end
  else
    if etd_lt
      hour = "#{etd_lt} LT"
    else
      hour = "#{etd_utc} UTC"
    end
  end
  hour
end
```

En este caso podemos obviar completamente la variable _hour_:

**Antes:**
``` ruby
def something
  if type.eql?('awesome')
    if eta_lt
      "#{eta_lt} LT"
    elsif eta_utc
      "#{eta_utc} UTC"
    end
  else
    if etd_lt
      "#{etd_lt} LT"
    else
      "#{etd_utc} UTC"
    end
  end
end
```

Como un plus, en este caso particular podemos evitar el sobreanidamiento duplicando una de las condiciones.

Y se logra **discutiblemente** un código más prolijo y fácil de leer y mantener.

**Después:**
``` ruby
def something
  if type.eql?('awesome') && eta_lt
    "#{eta_lt} LT"
  elsif type.eql?('awesome') && eta_utc
    "#{eta_utc} UTC"
  elsif etd_lt
    "#{etd_lt} LT"
  else
    "#{etd_utc} UTC"
  end
end
```

## Or/Equals
Usar los operadores [or_equals o similares ](http://www.rubyinside.com/what-rubys-double-pipe-or-equals-really-does-5488.html) cuando sea posible.

``` ruby
order = order || 'id'
# Se convierte en:
order ||= 'id'
```


## Blank?

En general se puede usar el método rails _blank?_ en lugar de preguntar por ambos _nil?_ y _empty?_.

**Antes:**
``` html
<% if @book.comments.nil? || @book.comments.empty? %>
  <p class="help-block">The book hasn't comments.<p>
<% else %>
  <div class="list-group-item">
    <%= @book.comments %>
  </div>
<% end %>
```

**Reemplazando por blank?:**
``` html
<% if @book.comments.blank? %>
  <p class="help-block">The book hasn't comments.<p>
<% else %>
  <div class="list-group-item">
    <%= @book.comments %>
  </div>
<% end %>
```

**Invirtiendo _if/else_:**

Además, en un _if/else_ es mejor empezar con la condición positiva.

Se siente más natural y es más fácil de leer y entender.

En este caso, podemos reemplazar _blank?_ por _present?_, e invertir las expresiones:

``` html
<% if @book.comments.present? %>
  <div class="list-group-item">
    <%= @book.comments %>
  </div>
<% else %>
  <p class="help-block">The book hasn't comments.<p>
<% end %>
```
