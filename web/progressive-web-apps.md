# Progressive Web Apps

Cyril Balit - Florian Orpelière

> Le mobile est maintenant omniprésent et nous nous posons tous la question des choix d'implémentations. Natif, hybride ou webapp chaque solutions a bien sur ces avantages et ces inconvénients. Mais si il existait une solution intermédiaire, plus progressive pour allier le meilleurs des 3 mondes. Nous verrons comment les progressives webapps se proposent de relever le challenge pour s'imposer comme la solution d'implémentation pour nos projets mobiles.

[Vidéo](https://www.youtube.com/watch?v=kqi4Xa1ViOQ)

Les Progressives WebApp s'appuyent sur un certain nombre d'outils pour définir des principes de base pour le développement d'applications mobiles aux vues d'être déployées sous forme d'applications mobiles.

Le but des progressives WebApps est d'augmenter l'utilisation et l'utilisabilité des applications Web. Mais ces concepts peuvent être détournés pour obtenir une meilleur expérience utilisateur lors de l'utilisation d'applications web depuis des terminaux mobiles.

Ce qui caractérise une progressive WebApp :
- Progressive

  L'application fonctionne partout, certaines fonctionnalités étant reservées à certains navigateurs.

- Responsive
- Offline

  L'application doit proposer au moins un mode dégragé en mode offline.

- Sécurisée
- Up to date

  Les informations fournies par l'application sont fraiches.

- Installable
- Engageante

- performante


## Performance

- Compression
- async defer
- http2
- critical CSS vs loadCSS
- Une bonne gestion du cache
- pas de paint / layout

Concept d'application shell : on propose à l'utilisateur le plus rapidement possible une information, une page de chargement allimentée par la suite en javascript (pour éviter l'effet page blanche lors du chargement d'une page web).

## Offline

- Service Worker

Proxy coté client doté d'un cache qui voit toutes les requêtes passer (_Compatible Chrome et Firefox (probablement bientot Edge)_)

Avec ce simple exemple, les données chargées lors de la navigation dans l'application sont mises en cache.

Chargé dans la page HTML :
```javascript
if('serviceWorker') in navigator) {
  navigator.serviceWorker.register('service-worker.js').then(function() {

  });
}
```

service-worker.js
```javascript
var filestoCache = [
  '/offline.html',
  '/img/offline.jpg'
];

self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request).then(function(response) {
      return response || fetch(event.request).then(function(responseF) {
        return caches.open('dynamic-cache').then(function (cache) {
          cache.put(event.request, responseF.clone());
          return responseF;
        });
      });
    }).catch(function() {
      return caches.match('/offline.html');
    });
  );
});

self.addEventListener('install', function(event) {
  self.skipWaiting();
  event.waitUntil(
    caches.open('offline-cache').then(function(cache) {
      cache.addAll(filestoCache);
    });
  );
});
```


Modes offline :
- on install

On place des objets dans le cache lors de l'installation de l'application.

- offline first

On regarde dans le cache avant d'interroger le réseau.

- etc...

_cf [The offline cookbook de Jake Archibald](https://jakearchibald.com/2014/offline-cookbook/)_


## Installable

prérequis :
- utilisation du service worker
- 2 interactions en moins de 5 minutes
- un manifest

```html
<link rel="manifest" href="/manifest.json">
```

```json
{
  "name": "Mon Application",
  "short_name": "MonApp",
  "icons": [
    {
      "src": "../img/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    }
  ],
  "start_url": "index.html",
  "display": "standalone",
  "background_color": "#3E4EB8",
  "theme_color": "#2F3BA2",
  "gcm_sender_id": "xxxxx"
}
```

short names et icons définissent les informations affichées sur le bureau.

le background définit la couleur du splash screen qui s'affichent au démarrage.

le theme défini la couleur de la barre de notifications.

## Engageante

En passant par un serveur tel que [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging/), on va pouvoir, via le Service Worker, envoyer des notifications à l'utilisateur même si l'application n'est pas démarrée.

- Vérification des souscriptions

```javascript
if('serviceWorker') in navigator) {
  navigator.serviceWorker.register('service-worker.js').then(function(reg) {
    reg.pushManager.getSubscription().then(function(sub) {
      //Gestion des souscriptions
    })
  });
}
```

- Abonnement

```javascript
function suscribe() {
  navigator.serviceWorker.getRegistration().then(function(reg) {
    reg.pushManager.subscribe({userVisibleOnly: true})
      .then(function(sub) {
        //Informer le serveur avec la subscription sub
      }).catch(function(error) {
        //Impossible de s'abonner
      });
  });
}
```

- Désabonnement

```javascript
function unsuscribe() {
  navigator.serviceWorker.getRegistration().then(function(reg) {
    reg.pushManager.getSubscription().then(function(sub) {
      if(sub) {
        sub.unsuscribe();
      }
    });
  }).catch(function(error) {
    //Impossible de se désabonner
  });
}
```

- Traitement d'une notifications

```javascript
  self.addEventListener('push', function(event) {
    event.waitUntil(function() {
      self.registration.showNotification('Title', {
        body: 'the message body',
        icon: 'images.icon.png',
        tag: 'tag'
      });
    });
  });
```

Ou de manière moins static

```javascript
  self.addEventListener('push', function(event) {
    event.waitUntil(
      fetch('/notification.json').then(function(response)) {
        return response.json();
      }).then(function(data) {
        self.registration.showNotification(data.title, {
          body: data.body,
          icon: data.icon,
          tag: data.tag,
          actions: [
            {action: 'open', title: 'Open'},
            {action: 'cancel', title: 'Cancel'}
          ]
        });
      })
    );
  });
```

- Gestion de l'action

```javascript
  self.addEventListener('notificationClick', function(event) {
    event.notification.close();
    if(event.action ==='open') {
      clients.openWindow('http://monserveur/resource/' + event.notification.tag);
    }
  })
```

`Clients.openWindow` ouvre l'application avec l'url spécifiée.
