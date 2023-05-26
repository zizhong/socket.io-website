---
title: Introduction
sidebar_position: 1
slug: /
---

import ThemedImage from '@theme/ThemedImage';
import useBaseUrl from '@docusaurus/useBaseUrl';

## Ce qu'est Socket.IO {#what-socketio-is}

Socket.IO est une bibliothèque qui permet une communication **à faible latence**, **bidirectionnelle** et **basée sur les événements** entre un client et un serveur.

<ThemedImage
  alt="Diagram of a communication between a server and a client"
  sources={{
    light: useBaseUrl('/images/bidirectional-communication2.png'),
    dark: useBaseUrl('/images/bidirectional-communication2-dark.png'),
  }}
/>

Il repose sur le protocole [WebSocket](https://fr.wikipedia.org/wiki/WebSocket) et offre des garanties supplémentaires telles qu'un mode dégradé en HTTP long-polling ou la reconnexion automatique.


:::info

Le protocole WebSocket permet d'ouvrir un canal de communication bidirectionnel (ou "full-duplex") sur un socket TCP pour les navigateurs et les serveurs web. Vous trouverez plus d'informations [ici](https://fr.wikipedia.org/wiki/WebSocket).

:::

Plusieurs implémentations de serveur Socket.IO sont disponibles :

- JavaScript (dont la documentation se trouve ici sur ce site)
  - [Installation](../02-Server/server-installation.md)
  - [API](../../server-api.md)
  - [Code source](https://github.com/socketio/socket.io)
- Java: https://github.com/mrniko/netty-socketio
- Java: https://github.com/trinopoty/socket.io-server-java
- Python: https://github.com/miguelgrinberg/python-socketio

Et des implémentations client dans la plupart des langages majeurs :

- JavaScript (qui peut être exécuté soit dans le navigateur, en Node.js ou dans React Native)
  - [Installation](../03-Client/client-installation.md)
  - [API](../../client-api.md)
  - [Code source](https://github.com/socketio/socket.io-client)
- Java: https://github.com/socketio/socket.io-client-java
- C++: https://github.com/socketio/socket.io-client-cpp
- Swift: https://github.com/socketio/socket.io-client-swift
- Dart: https://github.com/rikulo/socket.io-client-dart
- Python: https://github.com/miguelgrinberg/python-socketio
- .Net: https://github.com/doghappy/socket.io-client-csharp
- Golang: https://github.com/googollee/go-socket.io
- Golang: https://github.com/zishang520/socket.io/
- Rust: https://github.com/1c3t3a/rust-socketio
- Kotlin: https://github.com/icerockdev/moko-socket-io

Voici un exemple de base avec des WebSockets :

*Serveur* (basé sur le module [ws](https://github.com/websockets/ws))

```js
import { WebSocketServer } from "ws";

const server = new WebSocketServer({ port: 3000 });

server.on("connection", (socket) => {
  // envoi d'un message au client
  socket.send(JSON.stringify({
    type: "bonjour du serveur",
    content: [ 1, "2" ]
  }));

  // réception d'un message envoyé par le client
  socket.on("message", (data) => {
    const packet = JSON.parse(data);

    switch (packet.type) {
      case "bonjour du client":
        // ...
        break;
    }
  });
});
```

*Client*

```js
const socket = new WebSocket("ws://localhost:3000");

socket.addEventListener("open", () => {
  // envoi d'un message au serveur
  socket.send(JSON.stringify({
    type: "bonjour du client",
    content: [ 3, "4" ]
  }));
});

// réception d'un message envoyé par le serveur
socket.addEventListener("message", ({ data }) => {
  const packet = JSON.parse(data);

  switch (packet.type) {
    case "bonjour du serveur":
      // ...
      break;
  }
});
```

Et voici le même exemple avec Socket.IO :

*Serveur*

```js
import { Server } from "socket.io";

const io = new Server(3000);

io.on("connection", (socket) => {
  // envoi d'un message au client
  socket.emit("bonjour du serveur", 1, "2", { 3: Buffer.from([4]) });

  // réception d'un message envoyé par le client
  socket.on("bonjour du client", (...args) => {
    // ...
  });
});
```

*Client*

```js
import { io } from "socket.io-client";

const socket = io("ws://localhost:3000");

// envoi d'un message au serveur
socket.emit("bonjour du client", 5, "6", { 7: Uint8Array.from([8]) });

// réception d'un message envoyé par le serveur
socket.on("bonjour du serveur", (...args) => {
  // ...
});
```

Les deux exemples sont très similaires, mais sous le capot, Socket.IO fournit des fonctionnalités supplémentaires qui masquent la complexité de l'exploitation en production d'une application basée sur des WebSockets. Ces fonctionnalités sont répertoriées [ci-dessous](#features).

Mais d'abord, précisons ce que Socket.IO n'est pas.

## Ce que Socket.IO n'est pas {#what-socketio-is-not}

:::caution

Socket.IO n'est **PAS** une implémentation WebSocket.

:::

Bien que Socket.IO utilise effectivement les WebSockets lorsque cela est possible, il dispose de son propre protocole de communication. C'est pourquoi un client WebSocket ne pourra pas se connecter à un serveur Socket.IO, et un client Socket.IO ne pourra pas non plus se connecter à un serveur WebSocket ordinaire.

```js
// ATTENTION ! Le client ne sera pas en mesure de se connecter !
const socket = io("ws://echo.websocket.org");
```

Si vous recherchez un serveur WebSocket, vous pouvez vous diriger vers le module [ws](https://github.com/websockets/ws) ou bien [µWebSockets.js](https://github.com/uNetworking/uWebSockets.js).

Il existe également des [discussions](https://github.com/nodejs/node/issues/19308) pour inclure un serveur WebSocket dans le noyau Node.js.

Côté client, vous pourriez être intéressé par le module [robust-websocket](https://github.com/nathanboktae/robust-websocket).

:::caution

Socket.IO n'est pas destiné à être utilisé en arrière-plan dans une application mobile.

:::

La bibliothèque Socket.IO maintient une connexion TCP ouverte au serveur, ce qui peut entraîner une décharge importante de la batterie pour vos utilisateurs. Veuillez utiliser une plate-forme de messagerie dédiée telle que [FCM](https://firebase.google.com/docs/cloud-messaging) pour cet usage.

## Fonctionnalités {#features}

Voici les fonctionnalités fournies par Socket.IO par rapport une connexion WebSocket classique :

### Mode dégradé en HTTP long-polling {#http-long-polling-fallback}

La connexion sera dégradée en HTTP long-polling au cas où la connexion WebSocket ne pourrait pas être établie.

Cette fonctionnalité était la principale raison pour laquelle les gens utilisaient Socket.IO lorsque le projet a été créé il y a plus de dix ans (!), car la prise en charge des navigateurs pour WebSockets en était encore à ses balbutiements.

Même si la plupart des navigateurs prennent désormais en charge WebSockets (plus de [97 %](https://caniuse.com/mdn-api_websocket)), cela reste une fonctionnalité intéressante car nous recevons toujours des rapports d'utilisateurs qui ne peuvent pas établir de connexion WebSocket car ils sont derrière un proxy mal configuré.

### Reconnexion automatique {#automatic-reconnection}

Dans certaines conditions particulières, la connexion WebSocket entre le serveur et le client peut être interrompue sans que les deux parties soient conscientes de l'état rompu du lien.

C'est pourquoi Socket.IO inclut un mécanisme de ping/pong, qui vérifie périodiquement l'état de la connexion.

Et lorsque le client finit par être déconnecté, il se reconnecte automatiquement avec un délai d'attente exponentiel, afin de ne pas submerger le serveur.

### Mise en mémoire tampon des paquets {#packet-buffering}

Les paquets sont automatiquement mis en mémoire tampon lorsque le client est déconnecté et seront envoyés lors de la reconnexion.

Plus d'informations [ici](../03-Client/client-offline-behavior.md#buffered-events).

### Accusés de réception {#acknowledgements}

Socket.IO fournit un moyen pratique d'envoyer un événement et de recevoir une réponse :

*Émetteur*

```js
socket.emit("bonjour", "ô monde", (response) => {
  console.log(response); // "bien reçu !"
});
```

*Receveur*

```js
socket.on("bonjour", (arg, callback) => {
  console.log(arg); // "ô monde"
  callback("bien reçu !");
});
```

Vous pouvez également ajouter un délai maximum de réponse :

```js
socket.timeout(5000).emit("bonjour", "ô monde", (err, response) => {
  if (err) {
    // le destinataire n'a pas accusé réception de l'événement dans le délai imparti
  } else {
    console.log(response); // "bien reçu !"
  }
});
```

### Diffusion d'événements {#broadcasting}

Côté serveur, vous pouvez envoyer un événement à [tous les clients connectés](../04-Events/broadcasting-events.md) ou [à un sous-ensemble de clients](../04-Events/rooms.md ) :

```js
// à tous les clients connectés
io.emit("bonjour");

// à tous les clients connectés dans la room "news"
io.to("news").emit("bonjour");
```

Cela fonctionne également avec [plusieurs serveurs Socket.IO](../02-Server/using-multiple-nodes.md).

### Multiplexage  {#multiplexing}

Les *Namespaces* vous permettent de diviser la logique de votre application sur une seule connexion partagée. Cela peut être utile par exemple si vous souhaitez créer un canal "admin" que seuls les utilisateurs autorisés peuvent rejoindre.

```js
io.on("connection", (socket) => {
  // utilisateurs classiques
});

io.of("/admin").on("connection", (socket) => {
  // administrateurs
});
```

Plus d'informations à ce sujet [ici](../06-Advanced/namespaces.md).

## Questions fréquentes {#common-questions}

### Socket.IO est-il encore nécessaire aujourd'hui ? {#is-socketio-still-needed-today}

C'est une bonne question, puisque les WebSockets sont pris en charge [presque partout](https://caniuse.com/mdn-api_websocket) maintenant.

Cela étant dit, nous pensons que si vous utilisez des WebSockets classiques pour votre application, vous aurez finalement besoin d'implémenter la plupart des fonctionnalités déjà incluses (et testées !) dans Socket.IO, comme la [reconnexion automatique](#automatic-reconnection), les [accusés de réception](#acknowledgements) ou la [diffusion d'événements](#broadcasting).

### Quelle est la surcharge du protocole Socket.IO ? {#what-is-the-overhead-of-the-socketio-protocol}

`socket.emit("hello", "world")` sera envoyé sous la forme d'une seule « frame » WebSocket contenant `42["hello","world"]` avec :

- `4` étant le type de paquet "message" du protocol Engine.IO
- `2` étant le type de paquet "message" du protocol Socket.IO
- `["hello","world"]` étant la version `JSON.stringify()`-ée du tableau d'arguments

Donc quelques octets supplémentaires pour chaque message, qui peuvent être encore réduits par l'utilisation d'un [*parser* personnalisé](../06-Advanced/custom-parser.md).

:::info

La taille du « bundle » client lui-même est de [`10.4 kB`](https://bundlephobia.com/package/socket.io-client) (minifié et gzippé).

:::

### Quelque chose ne fonctionne pas correctement, au secours ! {#something-does-not-work-properly-please-help}

Veuillez consulter le [Guide de dépannage](../01-Documentation/troubleshooting.md).

## Étapes suivantes {#next-steps}

- [Tutoriel](/get-started/chat)
- [Installation côté serveur](../02-Server/server-installation.md)
- [Installation côté client](../03-Client/client-installation.md)
