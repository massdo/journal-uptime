# journal-uptime

Sonde de disponibilité externe pour un service MCP privé.

## Ce que fait ce dépôt

Toutes les cinq minutes, un workflow GitHub Actions envoie une requête `POST` **sans
en-tête d'authentification** à l'endpoint MCP surveillé, et vérifie qu'il reçoit bien un
`401`.

Ce `401` est le signal recherché : pour le produire, la requête doit avoir traversé la
résolution DNS, l'établissement TLS, le routage par la bordure Cloudflare, le tunnel, puis
avoir été réellement traitée par le processus applicatif. Un délai dépassé, une absence de
réponse, ou n'importe quel autre code signifie que la chaîne ne se comporte plus comme
prévu, et fait échouer le workflow — ce qui déclenche la notification GitHub.

Une seconde tentative espacée de vingt secondes évite qu'un aléa réseau isolé ne produise
une fausse alerte.

## Pourquoi ce dépôt est public et ne contient aucun secret

La sonde n'a pas besoin de s'authentifier : elle attend précisément un refus
d'authentification. Aucun jeton ne circule et aucun n'est stocké ici.

L'adresse surveillée est un nom de domaine public, déjà découvrable via les journaux de
Certificate Transparency. Le dépôt est public parce que GitHub Actions y est gratuit et
sans quota, contrairement aux dépôts privés — ce qui rend soutenable une cadence de cinq
minutes.

## Ce que cette sonde ne dit pas

Elle constate une indisponibilité, elle ne l'explique pas.

Le `401` est émis avant tout accès à la base de données : il ne prouve donc pas que
celle-ci est ouverte ni que le bon schéma est servi. Cette vérification demande un endpoint
de readiness côté application.

La mesure part des runners GitHub. Elle ne permet pas de distinguer une panne globale d'une
dégradation limitée au chemin réseau d'un client particulier ; une seconde source de mesure
dans une autre région resterait souhaitable.

Enfin, les déclenchements planifiés des runners partagés sont fréquemment retardés de
plusieurs minutes : la granularité réelle de détection est supérieure à la cadence
configurée. L'horodatage qui fait foi est celui de l'exécution, pas celui du planning.
