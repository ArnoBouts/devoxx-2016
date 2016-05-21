# 3 heures pour développer un microservice avec les micro frameworks java

Laurent Baresse & Igor Laborie

> Un grand nombre de solutions techniques sont envisageables pour l'implémentation de services REST aujourd'hui. Les défis à relever sont la productivité et la maturité des équipes de développement. Spring a grandement simplifié l'accès à son écosystème après la sortie de SpringBoot, mais cette technologie requiert encore un ticket d'entrée important. Dans certaines situations l'utilisation d'un framework tel que NodeJS est tout à fait justifié de par sa simplicité et sa productivité. C'est ici que les micro frameworks Java entrent en scène.

> Les __micro frameworks Java__ apportent la productivité de NodeJS, la simplicité de SpringBoot aux développeurs Java sans payer de ticket d'entrée.

> __SparkJava__ est un micro framework qui permet de développer très facilement des serveurs en Java de façon élégante avec Java 8. __Feign__ est un client REST open-source de Netflix qui est à la fois simple et extensible.

> Vous développerez un service RESTfull avec SparkJava et Feign s'intégrant au sein d'une architecture "microservices".

[Exercices](https://github.com/ilaborie/FeignSparkJava-exos)

Hands on s'appuyant sur une application de gestion de cave à vin (je retiens l'idée) durant lequel nous avons mis en oeuvre Feign et SparkJava.

Je retiens essentiellement __Feign__ qui à l'image de spring-data-jpa permet d'appeler des resources REST grace à quelques anotations sur une interface : simple et efficace.

```java
package devoxx.microframeworks.exos.services;

import devoxx.microframeworks.exos.models.Stock;
import feign.Headers;
import feign.Param;
import feign.RequestLine;

public interface StockService {

    @RequestLine("GET /api/wines/{id}/qty")
    Stock findByWine(@Param("id") String wid);

    @RequestLine("POST /api/wines/{id}/order?qty={nbr}")
    @Headers("Content-Type: application/json")
    public String createOrder(@Param("id") String wid, @Param("nbr") int quantity);

}
```

__SparkJava__ n'est pas non plus dénué d'intérêts, mais il me semble qu'il existe pas mal d'alternatives. Il s'agit d'un micro framework permetant de créer des applications Web facilement.

Main :

```java
package devoxx.microframeworks.exos;

import devoxx.microframeworks.exos.routes.CellarRoute;;
import spark.ResponseTransformer;

import static spark.Spark.*;

public class Main {
    public static void main(String... args) {
        Configuration configuration = Configuration.INSTANCE;
        ResponseTransformer encoder = object -> configuration.getGson().toJson(object);

        //...

        // Déclaration des fichiers static
        staticFileLocation("/public");

        // Déclaration des routes
        CellarRoute cellarRoute = new CellarRoute();
        get("api/cellar", cellarRoute::handleMyCellar, encoder);
        post("api/cellar/drink/:wid", cellarRoute::handleDrink, encoder);

        //...

        // Gestion des erreurs
        exception(SecurityException.class, (e, request, response) -> {
            response.status(403);
        });

        // CORS
        options("/*", (request, response) -> "");
        after((request, response) -> {
            response.header("Access-Control-Allow-Methods", "GET,PUT,POST,DELETE,OPTIONS");
            response.header("Access-Control-Allow-Origin", "*");
            response.header("Access-Control-Allow-Headers", "Content-Type,Authorization,X-Requested-With,Content-Length,Accept,Origin,");
            response.header("Access-Control-Allow-Credentials", "true");
        });
    }

}
```

Au delà des architectures microservices, je vous conseille vivement de regarder Feign dès que vous avez besoin de faire des appels REST en Java.
