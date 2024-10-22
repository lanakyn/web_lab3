# Лабораторная работа №3 "Создание простого REST API" по дисцпилине "Основы разработки Web-приложений"

В данной работе мы реализовали небольшое серверное приложения на ASP.NET Core, которое предоставляет REST API для управления ресурсом "Фильмы". В качестве источника данных используется статический класс с данными в памяти. Мы реализуем основные операции CRUD (Create, Read, Update, Delete) с использованием методов GET, POST, PUT, DELETE и OPTIONS.

**Шаг 1: Создание проекта**

Используйте командную строку или терминал для создания нового проекта ASP.NET Core:
```
dotnet new webapi -n MovieApi
cd MovieApi
```

**Шаг 2: Определение модели данных**

Создайте файл Movie.cs в папке Models:
```
namespace MovieApi.Models
{
    public class Movie
    {
        public int Id { get; set; }
        public string Title { get; set; }
        public string Genre { get; set; }
        public int Year { get; set; }
    }
}
```

**Шаг 3: Реализация источника данных**

Создайте файл MovieRepository.cs в папке Repositories:
```

using System.Collections.Generic;
using System.Linq;
using MovieApi.Models;

namespace MovieApi.Repositories
{
    public static class MovieRepository
    {
        private static List<Movie> _movies = new List<Movie>()
        {
            new Movie { Id = 1, Title = "The Shawshank Redemption", Genre = "Drama", Year = 1994 },
            new Movie { Id = 2, Title = "The Godfather", Genre = "Crime", Year = 1972 },
            new Movie { Id = 3, Title = "The Dark Knight", Genre = "Action", Year = 2008 }
        };

        public static List<Movie> GetAll() => _movies;

        public static Movie GetById(int id) => _movies.FirstOrDefault(movie => movie.Id == id);

        public static void Add(Movie movie)
        {
            movie.Id = _movies.Max(m => m.Id) + 1; // Simple ID assignment
            _movies.Add(movie);
        }

        public static void Update(int id, Movie movie)
        {
            var index = _movies.FindIndex(m => m.Id == id);
            if (index != -1)
            {
                movie.Id = id;
                _movies[index] = movie;
            }
        }

        public static void Delete(int id)
        {
            var movie = GetById(id);
            if (movie != null)
                _movies.Remove(movie);
        }
    }
}

```

**Шаг 4: Создание контроллера**

Создайте файл MoviesController.cs в папке Controllers:

```

using Microsoft.AspNetCore.Mvc;
using MovieApi.Models;
using MovieApi.Repositories;

namespace MovieApi.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class MoviesController : ControllerBase
    {
        [HttpGet]
        public ActionResult<IEnumerable<Movie>> GetMovies()
        {
            return Ok(MovieRepository.GetAll());
        }

        [HttpGet("{id}")]
        public ActionResult<Movie> GetMovie(int id)
        {
            var movie = MovieRepository.GetById(id);
            if (movie == null)
                return NotFound();
            return Ok(movie);
        }

        [HttpPost]
        public ActionResult<Movie> CreateMovie([FromBody] Movie movie)
        {
            MovieRepository.Add(movie);
            return CreatedAtAction(nameof(GetMovie), new { id = movie.Id }, movie);
        }

        [HttpPut("{id}")]
        public ActionResult UpdateMovie(int id, [FromBody] Movie movie)
        {
            MovieRepository.Update(id, movie);
            return NoContent();
        }

        [HttpDelete("{id}")]
        public ActionResult DeleteMovie(int id)
        {
            MovieRepository.Delete(id);
            return NoContent();
        }

        [HttpOptions]
        public IActionResult Options()
        {
            Response.Headers.Add("Allow", "GET, POST, PUT, DELETE, OPTIONS");
            return Ok();
        }
    }
}

```

Шаг 5: Конфигурация приложения

В файле Startup.cs, убедитесь, что все необходимые сервисы и маршруты настроены:

```

using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

namespace MovieApi
{
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllers();
        }

        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseRouting();
            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers();
            });
        }
    }
}

```
Шаг 6: Запуск приложения

Запустите приложение с помощью команды:
```

dotnet run

```
Теперь вы можете использовать REST API, доступное по адресу http://localhost:5000/api/movies. Вот примеры запросов:

- GET http://localhost:5000/api/movies — получить список всех фильмов.
- GET http://localhost:5000/api/movies/{id} — получить фильм по ID.
- POST http://localhost:5000/api/movies — создать новый фильм (передать данные фильма в формате JSON).
- PUT http://localhost:5000/api/movies/{id} — обновить фильм по ID (передать обновленные данные фильма в формате JSON).
- DELETE http://localhost:5000/api/movies/{id} — удалить фильм по ID.
- OPTIONS http://localhost:5000/api/movies — получить доступные методы для данного ресурса.

В этом примере было создано базовое REST API для управления сущностью "Фильм" с использованием ASP.NET Core и данных, хранящихся в памяти. Это прекрасная основа, которую можно расширять и улучшать в зависимости от требований вашего проекта.
