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
$ rails new cinema
$ cd cinema
$ git init
$ git checkout -b dev
$ rails g model movie name:string duration:integer
```

## Nuestro primer test
Vamos a agregar una validación en nuestro modelo movie.
Vamos a validar que esta validación se realice correctamente
```ruby
#app/models/movie.rb
...
validates :name, presence: true
...
#test/models/movie_test.rb
...
test 'movie without name must be invalid' do
  m = Movie.new name:nil
  assert_not m.valid?
end
...

#otra alternativa sería hacerlo con movie fixture
#test/fixtures/movies.yml
batman_vs_superman:
  name: Batman vs Superman

#test/models/movie_test.rb
...
test 'movie without name must be invalid' do
  m = movies(:batman_vs_superman)
  m.name = nil
  assert_not m.valid?, 'A new movie must have a name'
end
...
```
Para ejecutar los tests corremos:
```bash
$ rake test test/models/movie_test.rb
```

## Agregar guard al proyecto
```ruby
#Gemfile
...
group :development do
  ...
  gem 'guard'
  ...
end
$ bundle

#ejecutanos el sig comando en terminal para crear un archivo guard de conf
$ bundle exec guard init
#modificamos el archivo Guardfile
```
## Validar que una pelicula no se repita
```ruby
#app/models/movie.rb
...
validates :name, uniqueness: true
...
#test/models/movie_test.rb
test 'movie with repeated name must be invalid' do
  Movie.create(name: "Piratas de Silicon Valley")
  m = Movie.new(name:"Hola")
  assert_not m.valid?, 'Movie name can\'t be repeated'
end
```

## Validar Relaciones
Debemos de tener un modelo ranking que haga relacion a movies. En donde una pelicula
puede tener muchos rankings

```bash
$ rails g model ranking stars:integer movie:references
```
Ahora preparamos el testing
```ruby
#test/fixtures/movies.yml
...
fuera_del_cielo:
  name: Fuera Del Cielo
...
#test/fixtures/rankings.yml
one:
  stars: 1
  movie: batman vs_superman
two:
  stars: 2
  movie: fuera_del_cielo
#test/models/movie_test.rb
...
def setup
  @movie = movies(:batman_vs_superman)
end

test "movie has rankings" do
  assert_includes @movie.rankings, rankings(:one), "Batman vs superman movie should have one ranking"
end
...
#test/models/ranking_test.rb
...
test "ranking has movie" do
  assert_not_nil rankings(:one).movie, "Ranking one should have a movie"
end
...
```
Despues debemos de ejecutar el test y debe de estar fallando. Esto es TDD!
```bash
$ rake test test/models/movie_test.rb
$ rake test test/models/ranking_test.rb
```

Entonces sí debemos de agregar las relaciones al modelo

```ruby
#app/models/movie.rb
...
has_many :rankings
...
#app/models/ranking.rb
...
belongs_to :movie
...
```
## Validar el borrado en cascada
```ruby
#test/models/movie_test.rb
test 'deleting ranks on cascade' do
  @movie.destroy
  assert_empty Ranking.where(movie: @movie.id)
end
#correr pruebas
$ rake test test/models/movie_test.rb
```

Las pruebas muestran que fallan, por qué? Esto es TDD!
```ruby
#app/models/movie.rb
has_many :rankings, dependent: :destroy

#si ahora ejecutamos las pruebas todas deberian de funcionar :)
$ rake test test/models/movie_test.rb
```
