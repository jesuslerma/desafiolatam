# Acuerdos
1. Las lineas que empiezan con $ se deben de ejecutar en terminal
2. Para indicar la ruta del archivo que se va a modificar se escribe un hashtag previo y el path es partiendo el root del proyecto. Por ejemplo: #app/models/user.rb
3. Para indicar codigo en sql se pone al inicio de la linea sql>
4. Para indicar codigo en rails console se pone el comentario ejecutamos rails c
5. Yo voy a estar trabajando siempre con la misma base de datos form_development para conectarme a ella uso el siguiente comando
```bash
$ psql -h localhost form_development  desafio  -W.
```
Pueden utilizar la misma bd o crear una nueva previamente a cada ejercicio. Siempre vamos a conectarnos a psql.

El contenido mostrado a continuación es de ayuda para el material didáctico de desafio latam 2016. Este archivo hace referencia a la clase
47 referente a testing unitario

## Antes de empezar la clase
Antes de empezar con la clase creamos un nuevo proyecto

```bash
$ rails new uploader --database=postgresql
$ cd uploader
$ git init
$ git checkout -b dev
$ rails g scaffold movie name:string duration:integer photo:string
```
Agregamos 'carrierwave' al Gemfile

Generamos un uploader
```bash
$ rails generate uploader Photo
```

En el archivo app/models/movie.rb agregamos lo siguiente
```ruby
#app/models/movie.rb
...
mount_uploader :photo, PhotoUploader
...
# ejecutamos en terminal
$ rails s
```

Podemos entrar al siguiente enlace http://localhost:3000/movies/new

Sin embargo el campo de Photo se muestra como un string

Para cambiarlo tenemos que modificar

```html
<!-- app/views/movies/_form.html
 reemplazamos -->
<%= form_for(@movie) do |f| %>
<!-- por -->
<%= form_for(@movie, html:{multipart:true}) do |f| %>

<!-- reemplazamos -->
<%= f.text_field :photo %>
<!-- por -->
<%= f.text_field :photo %>
```

Para poder ver la imagen que subimos
```html
<!-- en app/views/movies/show.html.erb
reemplazamos -->
<%= @movie.photo %>
<!-- por -->
<%= image_tag(@movie.photo) if @movie.photo? %>

<!-- en app/views/movies/index.html.erb
  reemplazamos -->
<td><%= movie.photo %></td>
<!-- por -->
<td><%= image_tag(movie.photo) if movie.photo? %></td>
```

Agregar default_url para que se muestre una imagen por defecto cuando no se ha
escogido alguna
```ruby
# en app/uploaders/PhotoUploader
# agregar
def default_url
  'https://i.ytimg.com/vi/_bF2V8K79wk/maxresdefault.jpg'
end
# tenemos que cambiar los siguientes archivos
# en app/views/movies/show.html.erb
# reemplazamos
# <%= image_tag(@movie.photo) if @movie.photo? %>
# por
# <%= image_tag(@movie.photo) %>
# en app/views/movies/index.html.erb
# reemplazamos
# <td><%= image_tag(movie.photo) if movie.photo? %></td>
# por
# <td><%= image_tag(movie.photo) %></td>
#
```

Ahora vamos a ver como hacer los thumbs (visualizacion miniatura de las imgs)

```bash
$ sudo apt-get install imagemagick #ubuntu
$ brew install imagemagick #mac os
```

Agregamos al Gemfile lo siguiente: gem 'mini_magick'

Descomentamos la linea include CarrierWave::MiniMagick en el uploader
app/uploaders/photo_uploader.rb

Y habilitamos las transformaciones
```ruby
# app/uploaders/photo_uploader.rb

include CarrierWave::MiniMagick

process resize_to_fit: [400, 400]

version :thumb do
  process resize_to_fill: [100, 100]
end

# eliminamos los registros existentes
# ejecutamos en terminal
Movie.delete_all
```

Ahora vamos a usar esta version en todas las previsualizaciones

```html
<!-- en app/views/movies/show.html.erb
reemplazamos -->
<%= image_tag(@movie.photo)  %>
<!-- por -->
<%= image_tag(@movie.photo.thumb)  %>

<!-- en app/views/movies/index.html.erb
  reemplazamos -->
<td><%= image_tag(movie.photo) %></td>
<!-- por -->
<td><%= image_tag(movie.photo.thumb) %></td>
```
Para que ell default_url tenga el formato de thumb debemos de agregar lo sig
```ruby
# en app/uploaders/PhotoUploader
# agregar
def default_url
  'https://i.ytimg.com/vi/_bF2V8K79wk/maxresdefault.jpg'
end
```

En caso de que el formulario muestre un error podemos mostrar
un preview de la imagen subida

Primero debemos de poner una validación para el modelo

```ruby
# app/models/movie.rb
...
validates :name, presence: true
...
```
