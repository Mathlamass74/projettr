# API Gateway README - Mapping Complet (Element -> Fichier:Ligne)

Ce document liste les éléments utilisés dans `api-gateway`, avec leur rôle exact et leur emplacement.

## 1. Entrée application

| Élément | Type | Ce que ça fait | Emplacement |
|---|---|---|---|
| `bootstrap` | Fonction de démarrage | Lance toute l'initialisation runtime (Vault, HTTPS, proxies, Swagger, CORS, listen, upgrade WS). | `api-gateway/src/main.ts:35` |
| `fetchVaultSecrets` | Fonction utilitaire startup | Charge les secrets Vault (`secret/data/transcendence/env`) avec retries puis fusionne dans `process.env`. | `api-gateway/src/main.ts:8` |

## 2. Configuration runtime (dans `bootstrap`)

| Élément | Ce que ça fait | Emplacement |
|---|---|---|
| `httpsOptions` | Lit les certificats `/secrets/api-gateway.key` et `/secrets/api-gateway.crt` pour démarrer Nest en HTTPS. | `api-gateway/src/main.ts:37` |
| `NestFactory.create(AppModule, { httpsOptions })` | Crée l'app NestJS avec transport HTTPS. | `api-gateway/src/main.ts:41` |
| `authTarget` | URL service auth depuis env. | `api-gateway/src/main.ts:42` |
| `galleryTarget` | URL service gallery depuis env. | `api-gateway/src/main.ts:43` |
| `canvasTarget` | URL service canvas depuis env. | `api-gateway/src/main.ts:44` |
| `usersTarget` | URL service users depuis env. | `api-gateway/src/main.ts:45` |
| `aiTarget` | URL service AI depuis env (utilisé dans health controller, pas dans router API principal). | `api-gateway/src/main.ts:47` |
| `httpsAgent` | Agent HTTPS avec `rejectUnauthorized: false` pour upstreams certifiés self-signed. | `api-gateway/src/main.ts:49` |
| `corsOrigins` | Construit la whitelist CORS depuis `CORS_ORIGINS`/`FRONTEND_URL`/fallback. | `api-gateway/src/main.ts:52` |

## 3. Proxies Swagger JSON

| Élément | Type | Ce que ça fait | Emplacement |
|---|---|---|---|
| `docsProxy(target, service)` | Factory proxy | Crée un proxy `/docs-json/<service>` vers l'upstream cible. | `api-gateway/src/main.ts:64` |
| `/api/docs-json/auth` | Route proxy docs | Proxy Swagger JSON auth service. | `api-gateway/src/main.ts:74` |
| `/api/docs-json/canvas` | Route proxy docs | Proxy Swagger JSON canvas service. | `api-gateway/src/main.ts:75` |
| `/api/docs-json/gallery` | Route proxy docs | Proxy Swagger JSON gallery service. | `api-gateway/src/main.ts:76` |
| `/api/docs-json/users` | Route proxy docs | Proxy Swagger JSON users service. | `api-gateway/src/main.ts:77` |

## 4. Routing API principal (`/api/*`)

| Élément | Type | Ce que ça fait | Emplacement |
|---|---|---|---|
| `apiRouter` | Router function proxy | Route dynamiquement selon le préfixe du path après suppression de `/api`. | `api-gateway/src/main.ts:81` |
| Mapping `/auth* -> authTarget` | Règle routing | Envoie les routes auth vers auth-service. | `api-gateway/src/main.ts:85` |
| Mapping `/users*|/social*|/notification* -> usersTarget` | Règle routing | Envoie users/social/notification vers users-service. | `api-gateway/src/main.ts:86` |
| Mapping `/canvas*|/loop* -> canvasTarget` | Règle routing | Envoie canvas/loop vers canvas-service. | `api-gateway/src/main.ts:87` |
| Mapping `/gallery* -> galleryTarget` | Règle routing | Envoie gallery vers gallery-service. | `api-gateway/src/main.ts:88` |
| `apiProxy` | Proxy HTTP+WS | Proxy principal `/api` avec router dynamique, `xfwd`, `pathRewrite`, handlers events. | `api-gateway/src/main.ts:92` |
| `pathRewrite: /^\/api/ -> ''` | Réécriture path | Forward `/api/x` vers `/x` côté microservice. | `api-gateway/src/main.ts:101` |
| `on.error` | Handler erreur proxy | Log erreur + append dans `/tmp/proxy-error.txt`. | `api-gateway/src/main.ts:104` |
| `on.proxyRes` | Handler réponse proxy | Intercepte réponses auth et lit `set-cookie` (actuellement sans traitement complémentaire). | `api-gateway/src/main.ts:108` |
| `app.use('/api', middleware)` | Point d'entrée API | Bypass `/api/health` vers Nest controller, sinon délègue au `apiProxy`. | `api-gateway/src/main.ts:170` |

## 5. Proxy WebSocket / Socket.IO

| Élément | Type | Ce que ça fait | Emplacement |
|---|---|---|---|
| `socketProxy` | Proxy WS | Proxy dédié `/socket.io` vers canvas-service avec `ws: true`. | `api-gateway/src/main.ts:120` |
| `app.use('/socket.io', socketProxy)` | Route proxy WS | Monte le proxy HTTP Socket.IO. | `api-gateway/src/main.ts:128` |
| `server.on('upgrade', ...)` | Upgrade handler | Route explicitement les upgrades WS vers `socketProxy` quand URL match `socket.io`. | `api-gateway/src/main.ts:181` |

## 6. Swagger UI Gateway

| Élément | Type | Ce que ça fait | Emplacement |
|---|---|---|---|
| Guard `NODE_ENV !== 'production'` | Condition runtime | Active Swagger UI uniquement hors production. | `api-gateway/src/main.ts:131` |
| `DocumentBuilder` | Config Swagger | Définit title/description/version/server (`/api`). | `api-gateway/src/main.ts:132` |
| `SwaggerModule.createDocument` | Génération doc | Crée le document OpenAPI de la gateway. | `api-gateway/src/main.ts:139` |
| `SwaggerModule.setup('api/docs', ...)` | UI Swagger | Expose l'UI docs à `/api/docs` avec les URLs multi-services. | `api-gateway/src/main.ts:140` |
| `swaggerOptions.urls[]` | Agrégation docs | Branche Auth/Canvas/Gallery/Users dans l'UI via `/api/docs-json/*`. | `api-gateway/src/main.ts:143` |

## 7. CORS

| Élément | Type | Ce que ça fait | Emplacement |
|---|---|---|---|
| `app.enableCors(...)` | Config CORS | Active CORS global avec validation dynamique d'origine. | `api-gateway/src/main.ts:155` |
| `origin(origin, callback)` | Callback validation origin | Autorise origine vide (non-browser) ou origine whitelistée; sinon bloque. | `api-gateway/src/main.ts:156` |
| `credentials: true` | Option CORS | Permet l'envoi de cookies cross-origin. | `api-gateway/src/main.ts:165` |
| `optionsSuccessStatus: 204` | Option CORS | Réponse OPTIONS standardisée à 204. | `api-gateway/src/main.ts:166` |

## 8. Serveur HTTP(S)

| Élément | Type | Ce que ça fait | Emplacement |
|---|---|---|---|
| `const port = 9093` | Constante runtime | Port d'écoute de la gateway. | `api-gateway/src/main.ts:176` |
| `app.listen(port)` | Démarrage serveur | Démarre l'application Nest. | `api-gateway/src/main.ts:177` |

## 9. Modules / Controllers / Services Nest

| Élément | Type | Ce que ça fait | Emplacement |
|---|---|---|---|
| `AppModule` | Module racine | Assemble `HealthModule`, `AppController`, `AppService`. | `api-gateway/src/app.module.ts:6` |
| `AppController` | Controller racine | Expose `GET /` pour endpoint hello test/stub. | `api-gateway/src/app.controller.ts:4` |
| `getHello()` | Handler HTTP | Retourne le message de `AppService`. | `api-gateway/src/app.controller.ts:9` |
| `AppService` | Service | Retourne `Hello World!` (stub). | `api-gateway/src/app.service.ts:4` |
| `HealthModule` | Module fonctionnel | Monte `HealthController` et importe `HttpModule`. | `api-gateway/src/health/health.module.ts:5` |
| `HealthController` | Controller santé | Expose `GET /api/health` et agrège l'état des microservices. | `api-gateway/src/health/health.controller.ts:6` |
| `health()` | Handler health | Ping `/health` sur auth/users/canvas/gallery/AI et renvoie `ok/degraded` + détail par service. | `api-gateway/src/health/health.controller.ts:11` |

## 10. Health-check agrégé (détail)

| Élément | Ce que ça fait | Emplacement |
|---|---|---|
| `authBase/usersBase/canvasBase/galleryBase/aiBase` | Résout les URLs de base des services depuis env (avec fallback). | `api-gateway/src/health/health.controller.ts:12` |
| `services[]` | Déclare la liste des services à vérifier et leurs URLs `/health`. | `api-gateway/src/health/health.controller.ts:17` |
| `httpsAgent(rejectUnauthorized:false)` | Autorise appels HTTPS internes self-signed pendant les checks. | `api-gateway/src/health/health.controller.ts:27` |
| `Promise.all(services.map(...))` | Exécute les checks en parallèle. | `api-gateway/src/health/health.controller.ts:29` |
| `results[service.name] = 'ok'|'down'` | Stocke le statut par service. | `api-gateway/src/health/health.controller.ts:33` |
| `healthStatus` | Calcule `ok` si tous `ok`, sinon `degraded`. | `api-gateway/src/health/health.controller.ts:40` |

## 11. Tests présents

| Élément | Type | Ce que ça fait | Emplacement |
|---|---|---|---|
| `AppController` unit test | Test unitaire | Vérifie que `getHello()` retourne `Hello World!`. | `api-gateway/src/app.controller.spec.ts:18` |
| `AppController (e2e)` | Test e2e | Vérifie `GET /` -> `200` + `Hello World!`. | `api-gateway/test/app.e2e-spec.ts:19` |

## 12. Dépendances clés utilisées

| Dépendance | Ce que ça apporte | Où utilisée |
|---|---|---|
| `@nestjs/core` / `@nestjs/common` | Bootstrap Nest et décorateurs modules/controllers/services. | `api-gateway/src/main.ts:1`, `api-gateway/src/app.module.ts:1`, `api-gateway/src/app.controller.ts:1` |
| `http-proxy-middleware` | Reverse proxy HTTP + WS (`createProxyMiddleware`). | `api-gateway/src/main.ts:4` |
| `@nestjs/swagger` | Swagger UI gateway + agrégation docs JSON services. | `api-gateway/src/main.ts:3` |
| `@nestjs/axios` + `rxjs` | Health checks HTTP vers microservices (`HttpService` + `firstValueFrom`). | `api-gateway/src/health/health.controller.ts:2`, `api-gateway/src/health/health.controller.ts:3` |
| Node `https` | Agent HTTPS custom (self-signed). | `api-gateway/src/main.ts:6`, `api-gateway/src/health/health.controller.ts:4` |
| Node `fs` | Lecture certs HTTPS + écriture log erreurs proxy. | `api-gateway/src/main.ts:5` |

## 13. Variables d'environnement effectivement lues

| Variable | Usage | Emplacement |
|---|---|---|
| `VAULT_ADDR` | Endpoint Vault pour récupérer secrets. | `api-gateway/src/main.ts:9` |
| `VAULT_TOKEN` | Token d'accès Vault. | `api-gateway/src/main.ts:10` |
| `AUTH_SERVICE_URL` | Cible proxy auth. | `api-gateway/src/main.ts:42` |
| `GALLERY_SERVICE_URL` | Cible proxy gallery. | `api-gateway/src/main.ts:43` |
| `CANVAS_SERVICE_URL` | Cible proxy canvas/socket. | `api-gateway/src/main.ts:44` |
| `USERS_SERVICE_URL` | Cible proxy users/social/notification. | `api-gateway/src/main.ts:45` |
| `CORS_ORIGINS` | Liste whitelist CORS (CSV). | `api-gateway/src/main.ts:46` |
| `AI_SERVICE_URL` | URL AI (stockée bootstrap + utilisée dans health controller fallback/env). | `api-gateway/src/main.ts:47`, `api-gateway/src/health/health.controller.ts:16` |
| `FRONTEND_URL` | Fallback CORS si `CORS_ORIGINS` absent. | `api-gateway/src/main.ts:48` |
| `NODE_ENV` | Toggle activation Swagger UI. | `api-gateway/src/main.ts:131` |
| `AUTH_URL` | Base URL health auth (health controller). | `api-gateway/src/health/health.controller.ts:12` |
| `USERS_URL` | Base URL health users (health controller). | `api-gateway/src/health/health.controller.ts:13` |
| `CANVAS_URL` | Base URL health canvas (health controller). | `api-gateway/src/health/health.controller.ts:14` |
| `GALLERY_URL` | Base URL health gallery (health controller). | `api-gateway/src/health/health.controller.ts:15` |

## 14. Endpoints exposés par la gateway

| Endpoint | Source | Ce que ça fait |
|---|---|---|
| `GET /` | `AppController` | Endpoint hello de base (`Hello World!`). |
| `GET /api/health` | `HealthController` | Renvoie l'état agrégé des microservices. |
| `/api/docs` | Swagger setup | UI Swagger agrégée multi-services (hors prod). |
| `/api/docs-json/auth` | Proxy docs | Swagger JSON auth-service. |
| `/api/docs-json/canvas` | Proxy docs | Swagger JSON canvas-service. |
| `/api/docs-json/gallery` | Proxy docs | Swagger JSON gallery-service. |
| `/api/docs-json/users` | Proxy docs | Swagger JSON users-service. |
| `/api/auth/*` | API proxy router | Reverse proxy vers auth-service. |
| `/api/users/*` | API proxy router | Reverse proxy vers users-service. |
| `/api/social/*` | API proxy router | Reverse proxy vers users-service. |
| `/api/notification/*` | API proxy router | Reverse proxy vers users-service. |
| `/api/canvas/*` | API proxy router | Reverse proxy vers canvas-service. |
| `/api/loop/*` | API proxy router | Reverse proxy vers canvas-service. |
| `/api/gallery/*` | API proxy router | Reverse proxy vers gallery-service. |
| `/socket.io/*` | Socket proxy | Reverse proxy WS vers canvas-service. |

## 15. Remarque maintenance

Quand tu modifies une route proxy, un module/controller ou une variable d'env de la gateway, mets à jour la ligne ici pour garder ce README comme index technique rapide.