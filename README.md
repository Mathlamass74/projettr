# User Front README - Mapping Complet (Element -> Fichier:Ligne)

Ce document liste les éléments utilisés dans le front user, avec leur rôle exact et leur emplacement.

## 1. Routes user (App Router)

| Élément | Type | Ce que ça fait | Emplacement |
|---|---|---|---|
| `UserLayout` | Route layout (`/user/*`) | Enveloppe toutes les pages user avec `UserProvider` pour exposer l'état/actions user. | `frontend/app/(user)/layout.tsx:3` |
| `UserProfilePage` | Page `/user/profile` | Affiche le profil connecté. Monte `Navbar`, `PageContainer`, puis `UserProfile`. | `frontend/app/(user)/user/profile/page.tsx:5` |
| `UserProfileByIdPage` | Page `/user/profile/[id]` | Affiche le profil public d'un autre user, en passant `profileId` à `UserProfile`. | `frontend/app/(user)/user/profile/[id]/page.tsx:5` |
| `UserSettingsPage` | Page `/user/settings` | Affiche les paramètres du user connecté via `UserSettings`. | `frontend/app/(user)/user/settings/page.tsx:5` |

## 2. Context user

| Élément | Type | Ce que ça fait | Emplacement |
|---|---|---|---|
| `UserProvider` | Provider React Context | Instancie `useUserData()` et partage son retour à tout le sous-arbre user. | `frontend/features/user/UserContext.tsx:8` |
| `useUser` | Hook Context consumer | Donne accès à l'état/actions user et lève une erreur hors `UserProvider`. | `frontend/features/user/UserContext.tsx:18` |

## 3. Composants user

| Élément | Type | Ce que ça fait | Emplacement |
|---|---|---|---|
| `UserProfile` | Composant principal profil/amis | Gère l'écran profil + onglet amis: recherche, demandes, blocages, notifications, actions sociales. | `frontend/features/user/UserProfile.tsx:66` |
| `StatusBadge` | Sous-composant UI | Affiche pastille + texte `online/offline` selon le statut. | `frontend/features/user/UserProfile.tsx:30` |
| `ProfileIdentity` | Sous-composant UI | Affiche avatar profil avec fallback image par défaut en cas d'erreur. | `frontend/features/user/UserProfile.tsx:43` |
| `UserSettings` | Composant principal settings | Gère édition profil, visibilité stats, upload avatar, suppression compte. | `frontend/features/user/UserSettings.tsx:17` |
| `UserListIdentity` | Composant partagé liste users | Affiche mini identité (avatar, username, statut) cliquable vers profil. | `frontend/features/user/UserShared.tsx:15` |
| `VisibilityToggleRow` | Composant partagé settings | Affiche un toggle Oui/Non pour un flag de visibilité de profil. | `frontend/features/user/UserShared.tsx:55` |

## 4. Hook métier `useUserData` (fonctions exposées)

| Élément | Ce que ça fait | Emplacement |
|---|---|---|
| `useUserData` | Point d'entrée state + actions du module user. | `frontend/features/user/hooks/useUserData.ts:78` |
| `refreshAll` | Recharge profil + social + notifications au montage et sur refresh manuel. | `frontend/features/user/hooks/useUserData.ts:207` |
| `updateMe` | Met à jour les champs profil (`display_name`, `bio`, langue, flags visibilité...). | `frontend/features/user/hooks/useUserData.ts:229` |
| `setStatus` | Met à jour le statut (`online`, `offline`, `in_loop`). | `frontend/features/user/hooks/useUserData.ts:243` |
| `uploadAvatar` | Upload un fichier avatar vers l'API users. | `frontend/features/user/hooks/useUserData.ts:257` |
| `deleteAccount` | Supprime le compte et nettoie l'état user local. | `frontend/features/user/hooks/useUserData.ts:271` |
| `refreshSocial` | Reconstruit `friends`, `pendingReceived`, `pendingSent`, `blockedUsers` depuis `/social`. | `frontend/features/user/hooks/useUserData.ts:135` |
| `sendFriendRequest` | Envoie une demande d'ami après garde-fous anti doublon/état connu. | `frontend/features/user/hooks/useUserData.ts:321` |
| `acceptFriendRequest` | Accepte une demande reçue puis refresh social/notifications. | `frontend/features/user/hooks/useUserData.ts:343` |
| `rejectFriendRequest` | Refuse une demande reçue puis refresh social/notifications. | `frontend/features/user/hooks/useUserData.ts:360` |
| `cancelFriendRequest` | Annule une demande envoyée puis refresh social/notifications. | `frontend/features/user/hooks/useUserData.ts:377` |
| `blockUser` | Bloque un user puis refresh social. | `frontend/features/user/hooks/useUserData.ts:394` |
| `unblockUser` | Débloque un user puis refresh social. | `frontend/features/user/hooks/useUserData.ts:413` |
| `unfriendUser` | Supprime un ami puis refresh social/notifications. | `frontend/features/user/hooks/useUserData.ts:432` |
| `getFriendshipStatus` | Récupère le statut de relation entre user courant et user cible. | `frontend/features/user/hooks/useUserData.ts:449` |
| `refreshNotifications` | Recharge les notifications et recalcule `unreadCount`. | `frontend/features/user/hooks/useUserData.ts:190` |
| `markNotificationAsRead` | Marque une notification comme lue puis refresh notifications. | `frontend/features/user/hooks/useUserData.ts:461` |
| `markAllAsRead` | Marque toutes les notifications du user comme lues puis refresh. | `frontend/features/user/hooks/useUserData.ts:477` |
| `searchUsersByUsername` | Recherche des users par username (avec fallback `404 -> []`). | `frontend/features/user/hooks/useUserData.ts:493` |

## 5. Helpers internes `useUserData`

| Élément | Ce que ça fait | Emplacement |
|---|---|---|
| `getUserId` | Extrait l'id user depuis le profil courant (`profile?.user_id`). | `frontend/features/user/hooks/useUserData.ts:30` |
| `getMetadataString` | Lit une clé string dans `notification.metadata` de manière safe. | `frontend/features/user/hooks/useUserData.ts:34` |
| `filterFriendRequestNotifications` | Filtre/dédoublonne les notifications `friend_request` selon les demandes pendantes réelles. | `frontend/features/user/hooks/useUserData.ts:40` |
| `setError` | Transforme erreurs API/JS en message lisible pour l'UI. | `frontend/features/user/hooks/useUserData.ts:102` |
| `setMessage` | Pose un message succès et nettoie l'erreur courante. | `frontend/features/user/hooks/useUserData.ts:122` |
| `setProfileFromResponse` | Met à jour l'état `profile` avec la réponse backend users. | `frontend/features/user/hooks/useUserData.ts:130` |
| `runSocialMutation` | Helper générique mutation sociale + refresh (défini mais non utilisé). | `frontend/features/user/hooks/useUserData.ts:297` |

## 6. API layer `userApi` (wrappers)

| Élément | Endpoint principal | Ce que ça fait | Emplacement |
|---|---|---|---|
| `getMe` | `GET /users/me` | Récupère le profil du user connecté. | `frontend/features/user/userApi.ts:26` |
| `updateMe` | `PATCH /users/me` | Met à jour le profil du user connecté. | `frontend/features/user/userApi.ts:29` |
| `getById` | `GET /users/:id` | Récupère un profil public par id. | `frontend/features/user/userApi.ts:36` |
| `searchUsersByUsername` | `GET /users/search` | Cherche des users par username et mappe `data.users`. | `frontend/features/user/userApi.ts:40` |
| `setMyStatus` | `PATCH /users/me/status` | Met à jour le statut de présence du user connecté. | `frontend/features/user/userApi.ts:52` |
| `deleteMe` | `DELETE /users/me` | Supprime le compte du user connecté. | `frontend/features/user/userApi.ts:59` |
| `uploadMyAvatar` | `POST /users/me/avatar` | Upload avatar via `FormData`. | `frontend/features/user/userApi.ts:62` |
| `listFriendships` | `GET /social?limit&offset` | Retourne les relations sociales (accepted/pending/blocked). | `frontend/features/user/userApi.ts:73` |
| `sendFriendRequest` | `POST /social/friend-request` | Envoie une demande d'ami. | `frontend/features/user/userApi.ts:77` |
| `acceptFriendRequest` | `POST /social/friend-request/:id/accept` | Accepte une demande d'ami. | `frontend/features/user/userApi.ts:84` |
| `rejectFriendRequest` | `POST /social/friend-request/:id/reject` | Refuse une demande d'ami. | `frontend/features/user/userApi.ts:91` |
| `cancelFriendRequest` | `POST /social/friend-request/:id/cancel` | Annule une demande envoyée. | `frontend/features/user/userApi.ts:98` |
| `block` | `POST /social/block` | Bloque un utilisateur. | `frontend/features/user/userApi.ts:105` |
| `unblock` | `POST /social/unblock` | Débloque un utilisateur. | `frontend/features/user/userApi.ts:112` |
| `unfriend` | `POST /social/unfriend` | Retire un ami. | `frontend/features/user/userApi.ts:119` |
| `getFriends` | `GET /social/friends/:userId` | Lit la liste des amis d'un user donné. | `frontend/features/user/userApi.ts:126` |
| `getPendingReceived` | `GET /social/pending/received/:userId` | Lit les demandes reçues d'un user (wrapper dispo). | `frontend/features/user/userApi.ts:129` |
| `getPendingSent` | `GET /social/pending/sent/:userId` | Lit les demandes envoyées d'un user (wrapper dispo). | `frontend/features/user/userApi.ts:132` |
| `getFriendshipStatus` | `POST /social/status` | Lit le statut de relation entre deux users. | `frontend/features/user/userApi.ts:135` |
| `getMyCanvasesCount` | `GET /gallery/private?page=1&limit=1` | Calcule le total de canvases privés du user connecté. | `frontend/features/user/userApi.ts:143` |
| `getNotificationsByUser` | `GET /notification/user/:userId` | Récupère les notifications paginées d'un user. | `frontend/features/user/userApi.ts:160` |
| `getUnreadNotifications` | `GET /notification/user/:userId/unread` | Wrapper présent mais non appelé actuellement. | `frontend/features/user/userApi.ts:164` |
| `markNotificationAsRead` | `POST /notification/:id/read` | Marque une notification donnée comme lue. | `frontend/features/user/userApi.ts:168` |
| `markAllAsRead` | `POST /notification/user/:id/read-all` | Marque toutes les notifications comme lues. | `frontend/features/user/userApi.ts:175` |

## 7. Handlers et états clés dans `UserProfile`

| Élément | Ce que ça fait | Emplacement |
|---|---|---|
| `FRIENDS_SEEN_REQUESTS_STORAGE_KEY` | Clé localStorage pour mémoriser les demandes déjà vues. | `frontend/features/user/UserProfile.tsx:27` |
| `FRIENDS_SEEN_BLOCKED_STORAGE_KEY` | Clé localStorage pour mémoriser les users bloqués déjà vus. | `frontend/features/user/UserProfile.tsx:28` |
| `isOwnProfile` | Distingue profil perso vs profil public. | `frontend/features/user/UserProfile.tsx:111` |
| `shownProfile` | Profil réellement affiché (perso ou `viewedProfile`). | `frontend/features/user/UserProfile.tsx:112` |
| Sync statut depuis auth | Si connecté -> `online`, sinon `offline`, synchronisé via API. | `frontend/features/user/UserProfile.tsx:116` |
| Init `seen*` depuis localStorage | Hydrate les badges seen/unseen au montage. | `frontend/features/user/UserProfile.tsx:124` |
| Debounce recherche (`250ms`) | Lance `searchUsersByUsername` sans spam réseau. | `frontend/features/user/UserProfile.tsx:139` |
| Chargement profil public | Appelle `userApi.getById(profileId)` sur `/user/profile/[id]`. | `frontend/features/user/UserProfile.tsx:163` |
| Calcul statut relation | Déduit/charge `friendshipStatus` pour le profil consulté. | `frontend/features/user/UserProfile.tsx:191` |
| Chargement stats profil | Calcule `friendsCount` + `createdCanvasesCount`. | `frontend/features/user/UserProfile.tsx:220` |
| `openProfile` | Navigation vers `/user/profile/:id`. | `frontend/features/user/UserProfile.tsx:283` |
| `confirmAndBlockUser` | Confirm natif + blocage + retour arrière optionnel. | `frontend/features/user/UserProfile.tsx:287` |
| Badge amis (`friendsBadgeCount`) | Agrège unseen requests + notif amis non lues (hors friend_request). | `frontend/features/user/UserProfile.tsx:334` |
| Save seen ids | Persiste `seen*` quand onglet friends est ouvert. | `frontend/features/user/UserProfile.tsx:336` |
| Rafraîchissement notif popup | Recharge notif à l'ouverture du popup. | `frontend/features/user/UserProfile.tsx:364` |
| Click outside popup | Ferme popup notifications si clic extérieur. | `frontend/features/user/UserProfile.tsx:370` |
| `UserCanvases` render | Affiche la grille de canvases du profil affiché. | `frontend/features/user/UserProfile.tsx:591` |

## 8. Handlers clés dans `UserSettings`

| Élément | Ce que ça fait | Emplacement |
|---|---|---|
| `MAX_AVATAR_FILE_SIZE_BYTES` | Fixe la taille max upload avatar à 2MB. | `frontend/features/user/UserSettings.tsx:15` |
| Hydratation form depuis `profile` | Pré-remplit display name, bio, langue, flags visibilité. | `frontend/features/user/UserSettings.tsx:41` |
| `onSaveProfile` | Soumet les champs profil principaux à `updateMe`. | `frontend/features/user/UserSettings.tsx:54` |
| `onSelectAvatarFile` | Valide le fichier sélectionné + message d'erreur si trop grand. | `frontend/features/user/UserSettings.tsx:65` |
| `onUploadAvatar` | Upload du fichier avatar sélectionné. | `frontend/features/user/UserSettings.tsx:84` |
| `onDeleteAccount` | Confirmation suppression, suppression backend, puis logout. | `frontend/features/user/UserSettings.tsx:92` |
| `updateVisibility` | Met à jour localement les toggles puis push `updateMe`. | `frontend/features/user/UserSettings.tsx:100` |

## 9. Types du module user (`User.types.ts`)

| Type | Ce que ça représente | Emplacement |
|---|---|---|
| `UserStatus` | Statut présence d'un user (`online/offline/in_loop`). | `frontend/features/user/User.types.ts:1` |
| `UserRole` | Rôle applicatif (`user/moderator/admin`). | `frontend/features/user/User.types.ts:2` |
| `UserLanguage` | Langue de profil. | `frontend/features/user/User.types.ts:3` |
| `UserProfileDTO` | Contrat complet d'un profil user. | `frontend/features/user/User.types.ts:5` |
| `UserMeResponse` | Payload `GET /users/me`. | `frontend/features/user/User.types.ts:24` |
| `UserByIdResponse` | Payload `GET /users/:id`. | `frontend/features/user/User.types.ts:28` |
| `UserSearchResultDTO` | Résultat élémentaire de recherche user. | `frontend/features/user/User.types.ts:32` |
| `UserSearchResponse` | Payload brut recherche (`users[]`). | `frontend/features/user/User.types.ts:40` |
| `UpdateMeDTO` | Payload de mise à jour profil connecté. | `frontend/features/user/User.types.ts:44` |
| `UpdateUsernameDTO` | Payload update username (type dispo). | `frontend/features/user/User.types.ts:54` |
| `SetStatusDTO` | Payload update statut présence. | `frontend/features/user/User.types.ts:58` |
| `FriendshipStatus` | État relation sociale (`pending/accepted/blocked/rejected/null`). | `frontend/features/user/User.types.ts:62` |
| `SocialUser` | Modèle user simplifié pour listes sociales. | `frontend/features/user/User.types.ts:64` |
| `FriendshipRow` | Contrat brut d'une relation sociale backend. | `frontend/features/user/User.types.ts:75` |
| `SendFriendRequestDTO` | Payload envoi demande d'ami. | `frontend/features/user/User.types.ts:83` |
| `FriendActionDTO` | Payload actions sur `friendId` (accept/reject/cancel). | `frontend/features/user/User.types.ts:88` |
| `BlockUserDTO` | Payload block/unblock. | `frontend/features/user/User.types.ts:92` |
| `FriendshipStatusDTO` | Payload lookup statut relation entre deux users. | `frontend/features/user/User.types.ts:97` |
| `UnfriendUserDTO` | Payload suppression relation d'amitié. | `frontend/features/user/User.types.ts:102` |
| `NotificationDTO` | Contrat notification utilisateur. | `frontend/features/user/User.types.ts:107` |
| `CreateNotificationDTO` | Payload création notification (type dispo). | `frontend/features/user/User.types.ts:120` |
| `MarkNotificationReadDTO` | Payload marquage notification lue. | `frontend/features/user/User.types.ts:129` |
| `MarkAllReadResponse` | Réponse backend marquage global (`updated`). | `frontend/features/user/User.types.ts:133` |
| `UserStats` | Objet local pour stats affichées profil. | `frontend/features/user/User.types.ts:137` |
| `UserProfileVisibilitySettings` | Objet local toggles visibilité settings. | `frontend/features/user/User.types.ts:142` |

## 10. Dépendances externes utilisées par le user front

| Élément utilisé | Ce que ça apporte | Emplacement |
|---|---|---|
| `useAuth` (`AuthContext`) | Statut d'auth pour sync présence + action logout. | `frontend/features/user/UserProfile.tsx:10`, `frontend/features/user/UserSettings.tsx:9` |
| `ROUTES` | Navigation centralisée vers profile/settings/public profiles. | `frontend/features/user/UserProfile.tsx:9`, `frontend/features/user/UserSettings.tsx:10` |
| `galleryApi` | Récupération stats canvases des profils publics. | `frontend/features/user/UserProfile.tsx:20` |
| `UserCanvases` | Rendu de la galerie de canvases sous le profil. | `frontend/features/user/UserProfile.tsx:19` |
| `Navbar` | Barre de navigation des pages user. | `frontend/app/(user)/user/profile/page.tsx:1`, `frontend/app/(user)/user/settings/page.tsx:1` |
| `PageContainer` | Wrapper layout/padding des pages user. | `frontend/app/(user)/user/profile/page.tsx:2`, `frontend/app/(user)/user/settings/page.tsx:2` |

## 11. Remarque maintenance

Quand tu modifies une fonction/composant/type dans ce module, mets à jour sa ligne ici pour garder ce README comme index technique rapide.