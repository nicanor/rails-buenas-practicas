# Snappler Rails Best Practices

Este documento pretende convertirse en un documento **colaborativo** donde:

1. Definamos pautas para desarrollar aplicaciones Rails en _Snappler_
2. Acordemos una guía de estilos para escribir código de forma consistente
3. Identifiquemos y describamos soluciones a _malos olores_ comunes en nuestro código


-----------------------------

# Código limpio
> Clean code always looks like it was written by someone who cares.

##### Es deseable que el código de un programa:

0. haga su trabajo
1. sea corto y enfocado
2. sea expresivo
3. sea placentero para leer
4. esté bien documentado
5. sea fácilmente extendible por otro programador
6. tenga dependencias mínimas
7. tenga tests automáticos de unidad y aceptación


-------------------------------

# Guía de estilos
> Programs are meant to be read by humans and only incidentally for computers to execute.

Debemos entender al código como una forma de comunicación con otros desarrolladores.

El código está destinado a ser mantenido por otra persona.

No sólo por otros miembros de tu equipo de trabajo en el presente, sino también con miembros de trabajo en el futuro.

Es importante que el código comunique su propósito al observador casual.

Es por esto que es bueno seguir una **guía de estilos**.

Una guía de estilos es acerca de **consistencia**.

La consistencia dentro de un equipo de trabajo **mejora la comunicación** con nuestros compañeros.

Además nos impulsa a escribir código **prolijo y fácil de mantener y compartir**.

Seguir una guía de estilos nos ayuda a enfocarnos en lo importante.


##### Una guía de estilos define entre otras cosas:

* Cómo y dónde usar comentarios
* Cómo identar el código
* Uso apropiado de espacios en blanco
* Nombramiento apropiado de variables y funciones
* Cómo agrupar y organizar el código
* Buenos patrones e idiomas para usar
* Patrones a evitar

Propuesta: [guía de estilos de Ruby de bbatsov](https://github.com/bbatsov/ruby-style-guide).


##### Lecturas interesantes:

* [Why coding style matters?](https://www.smashingmagazine.com/2012/10/why-coding-style-matters/)
* [Why use a style guide?](http://www.codereadability.com/why-use-a-style-guide/)


-------------------------------

# Documentación
> Documentation, when done successfully, can keep forward momentum in place and keep the team focused.

El archivo **README.md** debe contener la información necesaria para que
un desarrollador nuevo pueda comenzar a ser productivo cuanto antes.

Tiene que estar escrito en [formato markdown de Github](https://guides.github.com/features/mastering-markdown/)

##### Por lo menos tiene que contar con la siguiente información.

1. En qué consiste la aplicación.
2. Pasos a seguir para instalar y usar la aplicación.
3. Estructura de la aplicación.
4. Otras explicaciones que puedan ser de utilidad al nuevo desarrollador.

Lecturas interesantes:

* [Ejemplo de un readme](EJEMPLO_README.md).
* [Writting great documentation](https://jacobian.org/writing/great-documentation/)


------------------------------

# Tamaño y carga de la aplicación
> No code runs faster than no code. No code has fewer bugs than no code. No code uses less memory than no code. No code is easier to understand than no code.

## application.rb

Rails provée muchos módulos que no siempre necesitamos.

Si miramos el archivo _application.rb_, notaremos la linea `require 'rails/all'`

Si vamos al [codigo fuente](https://github.com/rails/rails/blob/master/railties/lib/rails/all.rb),
encontraremos que esa linea se puede reemplazar por:

``` ruby
require 'active_record/railtie'
require 'action_controller/railtie'
require 'action_view/railtie'
require 'action_mailer/railtie'
require 'active_job/railtie'
require 'action_cable/engine'
require 'rails/test_unit/railtie'
require 'sprockets/railtie'
```

Luego de reemplazar la linea, es posible hacer más liviana una aplicación Rails identificando los módulos que no se usarán y comentándolos:

Por ejemplo, si se que mi aplicación no mandará mails y no usaré websockets ni trabajos asincrónicos, me quedará:

``` ruby
require 'active_record/railtie'
require 'action_controller/railtie'
require 'action_view/railtie'
#require 'action_mailer/railtie'
#require 'active_job/railtie'
#require 'action_cable/engine'
require 'rails/test_unit/railtie'
require 'sprockets/railtie'
```

Tener en cuenta que al hacerlo, también habrá que comentar las configuraciones en los environments según correspondan.


## Rails API

Muchas veces no necesitamos toda las funcionalidades que nos provée una aplicación Rails estandar, y sólo queremos definir una API.

Las aplicaciones de tipo API se crean con la opción `--api`.

```
rails new my_app --api
```


## Gemfile

**Mal olor:** demasiadas gemas.

Revisar que se estén usando todas las gemas y eliminar las que no se usen.

Puntualmente es muy común ver definidas en los proyectos las gemas **byebug** y **pry**, y las gemas **kaminari** y **will_paginate**, pese a que cumplan la misma función.

En estos casos, eliminar aquella que no se use.

En general es buena idea tener la menor cantidad de dependencias posibles.

Lectura interesante: [Kill your dependencies](http://www.mikeperham.com/2016/02/09/kill-your-dependencies/)

------------------------------

# Rutas

## Rutas RESTful

En Rails se tiende a favoreces una arquitectura RESTful.

Una ruta RESTful se logra a través de los helpers **resources** o **resource**, que proveen un mapeo entre distintos verbos HTTP y URLs a acciones de un controlador

Además de estas rutas, Rails provee soporte para agregar rutas arbitrarias

Si ves en tu código varias rutas definidas de forma excesivamente verbosa estás en presencia de un mal olor.

Ejemplo:

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

**Nota:** usar _:only_ y _:except_ para generar sólo las rutas necesarias disminuye la memoria necesaria y aumenta la velocidad del proceso de ruteo.


## Route concerns

Es común en aplicaciones rails tener archivos de rutas que repitan patrones para diferentes recursos.

Tener estas estructuras repetidas en un mal olor. Ejemplo:

``` ruby
get 'pages/all_images', to: 'pages#all_images', as: :pages_show_images
post 'pages/:id/upload_image', to: 'pages#upload_image', as: :page_upload_image
delete 'pages/:id/delete_image', to: 'pages#delete_image', as: :page_delete_image
resources :pages

get 'articles/list', to: 'articles#list', as: :articles_list
resources :articles, only: [:show, :create, :edit, :update]

get 'books/list', to: 'books#list', as: :books_list
resources :books

get 'tags/list', to: 'tags#list', as: :tags_list
get 'tags/all_images', to: 'tags#all_images', as: :tags_show_images
post 'tags/:id/upload_image', to: 'tags#upload_image', as: :tag_upload_image
delete 'tags/:id/delete_image', to: 'tags#delete_image', as: :tag_delete_image
resources :tags

get 'gifts/list', to: 'gifts#list', as: :airports_list
get 'gifts/all_images', to: 'gifts#all_images', as: :gifts_show_images
post 'gifts/:id/upload_image', to: 'gifts#upload_image', as: :gift_upload_image
delete 'gifts/:id/delete_image', to: 'gifts#delete_image', as: :gift_delete_image
resources :gifts
```

En estos casos, además de aplicar **member** y **collection** es una buena idea usar [concerns](http://guides.rubyonrails.org/routing.html#routing-concerns) para limpiar las partes repetidas.


``` ruby
concern :listable do
  get :list, on: :collection
end

concern :imageable do
  get :show_images, on: :collection
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

Este caso particular podemos aplicar nuevamente la idea de rutas RESTful.

``` ruby
concern :imageable do
  resources :images, only: [:index, :update, :delete]
end
```

Se debe tener en cuenta que realizar estos cambios en general implica cambiar código en varios lugares, como vistas y controladores.

Pero al mismo tiempo nos ayuda a tener un código con menos duplicación y mayor fácilidad de mantenimiento.

------------------------------

# Controladores

Los controladores son los encargados de leer los datos de entrada (solicitud), elegir las acciones apropiadas y retornar una salida.

Es una buena idea mantener a los controladores **simples**.

Si encontramos lógica de negocio en los controladores estamos en presencia de un mal olor.

Deberíamos transladar esta lógica al **modelo** o a **servicios**.

#### Lógica que está bien en un controlador:

* Lógica de sesión (autenticación, autorización, mensajes flash)
* Seguridad contra ataques externos y protección de parámetros de entrada
* Llamadas a métodos de modelos o servicios
* Determinación de formato de respuesta
* Ejecución de código de las vistas (render, redirect, etc)

**Lectura interesante:** [7 design patterns to refactor MVC](https://www.sitepoint.com/7-design-patterns-to-refactor-mvc-components-in-rails/)

------------------------------

# Modelos

Active Record

* Representa modelos y sus datos
* Representa asociaciones entre estos modelos
* Valida modelos antes de ser persistidos en la base de datos
* Ejecuta operaciones en la base de datos en una forma orientada a objetos

##### Malos olores:
* Tener código de vistas en el modelo. Solución: transladar código a un helper, view object o presenter.


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



## Abuso de self
Es mejor no usar _self_ innecesariamente. Esto:
``` ruby
  self.flight_type.eql?('arrival') ? self.date_in : self.date_out
```

Debería ser:
``` ruby
  flight_type.eql?('arrival') ? date_in : date_out
```

------------------------------

# Helpers y Views

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

------------------------------

# Idiomas Ruby

#### Pensar en Ruby: Método retorna algo vs es algo

``` ruby
# En lugar de:
def word_count
  return words.size
end

# Usar:
def word_count
  words.size
end
```

#### Usar funciones de alto orden

Ruby cuenta con funciones de alto orden, como _select_, _map_, _inject_, _any?_, _all?_, entre otras.

``` ruby
# En general, una estructura como la siguiente es un mal olor:
def keep_evens
  result_array = []
  for num in my_array
    result_array << num if num % 2 == 0
  end
  return result_array
end

# Lo mismo se puede escribir de una forma más simple, segura y expresiva:
def keep_evens
  my_array.select {|item| item.even?}
end
```

#### Usar notación **&**

``` ruby
# En lugar de:
['gato', 'perro', 'loro'].map { |x| x.upcase }
# Usar:
['gato', 'perro', 'loro'].map(&:upcase)
```

Explicación [aquí](http://blog.thoughtfolder.com/2008-02-25-a-detailed-explanation-of-ruby-s-symbol-to-proc.html)


#### Usar funciones específicas de colecciones

``` ruby
# En lugar de:
[1, 2, 3].map { |x| [x, x+1] }.flatten
# Usar:
[1, 2, 3].flat_map { |x| [x, x+1] }

# En lugar de:
[1, 2, 3].select { |x| x > 2 }.count
# Usar:
[1, 2, 3].count { |x| x > 2 }

# En lugar de:
[1, 2, 3].shuffle.first
# Usar:
[1, 2, 3].sample

# En lugar de:
(1..10).select { |num| num % 3 == 0 }.first
# Usar:
(1..10).find { |num| num % 3 == 0 }
```


#### Cuando usar keywords arguments

Si una función tiene muchos parámetros, a veces es mejor definirla con keywords arguments.

De esta forma, la función se vuelve mucho más expresiva y no es necesario recordar el orden de los parámetros.

``` ruby
# En lugar de:
def travel_info(location_id, currency_id, departure, arrival)
  #...
end

travel_info(query[:site_id], "USD", query[:initial_date], query[:end_date])

# Usar:
def travel_info(location_id:, currency_id:, departure:, arrival:)
  #...
end

travel_info(
  location_id: query[:site_id],
  currency: "USD"
  departure: query[:initial_date],
  arrival: query[:end_date]
)
```

#### El orden de las condiciones de los if es importante

Los if son más fácil de leer cuando comienzan con la condición positiva.

``` ruby
# En lugar de:
if !valid?
  "There is an error"
else
  "All ok"
end

# Usar:
if valid?
  "All ok"
else
  "There is an error"
end
```

También es mejor no usar _unless_ con _else_.



#### El código debe contar una historia


El código que se muestra a continuación, se encarga de recibir información de un usuario, e intentar encontrar un usuario con un token, o en caso de fallar,
intentar encontrar el usuario a partir del email. O en caso de fallar revisar si el dominio del email es un dominio permitido, en cuyo caso crea el usuario.
Caso contrario retorna false.

``` ruby
# Código original:

def self.find_for_google_oauth2(access_token, signed_in_resource=nil)
    data = access_token.info
    user = User.where(:provider => access_token.provider, :uid => access_token.uid ).first
    if user
      return user
    else
      registered_user = User.where(:email => access_token.info.email).first
      if registered_user
        return registered_user
      else
        users_allowed = %w(esteban.informatica@gmail.com)
        domains_allowed = %w(snappler.com aerolaplata.com.ar aero.tur.ar)
        domain = data["email"].split('@').last
        if (users_allowed.include? data["email"]) or (domains_allowed.include? domain)
          user = User.create(first_name: data["name"],
            provider:access_token.provider,
            email: data["email"],
            uid: access_token.uid ,
            password: Devise.friendly_token[0,20],
            )
        else
          false
        end
      end
    end
  end

# Ruby es muy expresivo, y si aprovechamos esto podemos escribir código
# tal que podamos entender su funcionamiento con un simple vistazo:

class OauthAutenticator

  def call
    find_by_oauth || find_by_email || create_user_if_allowed
  end

  def initialize(token)
    @provider = token.provider
    @uid      = token.uid
    @email    = token.info.email
    @name     = token.info.name
  end

  private

  def find_by_oauth
    User.find_by(provider: @provider, uid: @uid)
  end

  def find_by_email
    User.find_by(email: @email)
  end

  def create_user_if_allowed
    allowed_email? && User.create(
      first_name: @name,
      provider:   @provider,
      email:      @email,
      uid:        @uid,
      password:   Devise.friendly_token[0,20]
    )
  end

  def allowed_email?
    allowed_users   = %w(esteban.informatica@gmail.com)
    allowed_domains = %w(snappler.com aerolaplata.com.ar aero.tur.ar)
    domain = @email.split('@').last
    allowed_users.include?(@email) || allowed_domains.include?(domain)
  end
end
```
