# Cotizador API

Backend del Cotizador de Aero.

Realizado en RubyOnRails **5.0.0**

Versión de Ruby usada en desarrollo: **Ruby 2.3.0**

## Instrucciones:

1) Descargar e instalar _Redis_:

```
sudo apt-get install libgmp-dev libgmp3-dev
sudo add-apt-repository ppa:chris-lea/redis-server
sudo apt-get update
sudo apt-get install redis-server
```

2) Descargar aplicación, crear _settings.rb_ y _database.yml_, realizar setup de aplicación.

```
git clone git@gitlab.snappler.com:nica/cotizador-api.git
cd cotizador-api
cp config/initializers/settings-example.rb config/initializers/settings.rb
cp config/initializers/database-example.yml config/initializers/database.yml
bundle
bundle exec rake db:create
bundle exec rake db:migrate
bundle exec rake db:seed
bundle exec rails s
```

3) A través de un cliente web ir a _localhost:3000_ y entrar con credenciales:
* usr: admin@admin.com
* pdw: 123456

## Explicación

**Cotizador API** es una aplicación Rails 5 qué en conjunto con la aplicación **React/Redux** CotizadorFrontEnd (desarrollada por **Celerative**), forman el Cotizador de Aero.

**Cotizador API** provée una API que permite la búsqueda, confirmación y creación de reserva en la aplicación **MiddleOffice** de Hoteles, ProductosTerrestres, Cupos aéreos, Vuelos y Paquetes y Circuitos.

Para hacer esto se comunica con las siguientes aplicaciones:

* **nemo-adapter:** Comunicación con servicios de Nemo (Price Surfer). Búsqueda y booking de Hoteles
* **aero-adapter:** Búsqueda y booking de Hoteles de Aero
* **cupos:** Búsqueda y booking de Cupos Aéreos de Aero
* **productos_terrestres:** Búsqueda de visas, excursiones, transporte, asistencia al viajero y otros llamados "productos terrestres"
* **Aplicación de cambio:** Usada para ver las conversiones peso-dolar-euro
* **geography:** Aplicación para buscar locations y adaptarlo para destinos de aero y destinos de nemo
* **Aplicación de agencias:** Aplicación para buscar información de agencias
* **specialtours-adapter:** Aplicación para búsqueda y booking de Circuitos
* **rails-cas:** Aplicación para manejar autenticación en los sistemas de Aero
* **MiddleOffice:** Aplicación de gestión interna de Aero, donde se envía la información de las reservas finalizadas.

## Endpoints

Última actualización: **Viernes 21-10-2016**

```
GET    carts
GET    carts/new
GET    carts/:id
POST   carts/:id/items/:item_id
DELETE carts/:id/items/:item_id
POST   carts/:id/passengers/add
DELETE carts/:id/passengers/:passenger_id
GET    carts/:id/verify
POST   carts/:id/confirm
POST   carts/:id/send_email

GET    packages
GET    packages/other_rates

GET    flights
GET    flights/airlines
GET    flights/all
GET    flights/by_country

GET    hotels
GET    hotels/:hotel_id
GET    hotels/chains
GET    hotels/regimes
GET    hotels/other_rates
GET    hotels/names
GET    hotels/reduced_info
GET    hotels/cancellation_fees

GET    land_products/names
GET    land_products/(:kind)

POST   confirm_item

POST   login
GET    sellers
GET    agencies
GET    operators
GET    locations
GET    user_info
GET    popular_locations
GET    reserves
GET    cities
```