# Changelog

Toutes les évolutions notables de `pageup-remote-file` sont documentées ici.

Format : [Keep a Changelog](https://keepachangelog.com/fr/1.1.0/) — versionnage : [SemVer](https://semver.org/lang/fr/).

Ce plugin est le fork PageUp de [jenkinsci/remote-file-plugin](https://github.com/jenkinsci/remote-file-plugin) (plugin `remote-file`, dernière version 1.24), qui n'est plus maintenu.

## [1.0.0] - 2026-11-26

Première version du fork, basée sur `remote-file` 1.24.

### Contexte

Depuis Jenkins **2.579**, la bibliothèque Apache Commons Lang 2 (`org.apache.commons.lang.*`) ne fait plus partie du cœur ([changelog 2.579](https://www.jenkins.io/changelog/2.579/)). Le plugin `remote-file` 1.24 l'utilisait : au scan Multibranch, il levait `NoClassDefFoundError: org/apache/commons/lang/StringUtils` dans `RemoteJenkinsFileWorkflowBranchProjectFactory.getSCMSourceCriteria`. Résultat : l'indexation échouait et les webhooks GitHub (reçus en `200 OK`) ne déclenchaient aucun build. Voir [jenkinsci/remote-file-plugin#197](https://github.com/jenkinsci/remote-file-plugin/issues/197).

L'upstream étant abandonné, PageUp reprend le plugin sous son propre identifiant.

### Modifié

- **Nouvel identifiant : `pageup-remote-file`** (groupId `fr.pageup.jenkins`), nom affiché « Remote Jenkinsfile Provider (PageUp) ». Ainsi, Jenkins ne proposera jamais une future version de `remote-file` publiée par jenkinsci comme mise à jour de ce fork, et le fork est identifiable dans le gestionnaire de plugins.
  - Les noms de classes Java et les packages sont **inchangés** : la configuration des jobs (`config.xml`) existants est relue telle quelle.
  - `remote-file` et `pageup-remote-file` contiennent les mêmes classes et **ne doivent pas être installés en même temps** : désinstaller `remote-file` avant de déployer celui-ci (voir le README).
  - Les liens d'aide des formulaires pointent vers `/plugin/pageup-remote-file/`.
- **Version minimale de Jenkins : 2.580.1** (avant : 2.361.4). Compiler contre une version qui n'embarque plus Commons Lang 2 garantit qu'un futur usage de `org.apache.commons.lang.*` fera échouer la compilation au lieu de casser en production.
- Parent POM `org.jenkins-ci.plugins:plugin` 6.2128 → 6.2250, BOM `bom-2.580.x` 7401.vf0f166b_7fd69.
- Java 17 minimum (imposé par le parent POM et Jenkins 2.580). La surcharge `java.version=11` a été retirée.
- Version explicite `1.0.0` au lieu de `999999-SNAPSHOT`.

### Corrigé

- Suppression de toute utilisation de Commons Lang 2 : `StringUtils.isEmpty(x)` est remplacé par `hudson.Util.fixEmpty(x) == null` (API du cœur Jenkins, même comportement, aucune dépendance ajoutée). Concerne `RemoteJenkinsFileWorkflowBranchProjectFactory`, `RemoteJenkinsFileWorkflowMultiBranchProjectFactory`, `LocalMarkerSCMSourceCriteria` et les tests.
- `pom.xml` : correction de deux fautes de frappe (`${jenkins.baseline}}`) qui cassaient la propriété `jenkins.version` (déclarée deux fois) et le nom de la BOM.

### Supprimé

- Dépendances `commons-lang3` et `joda-time` : déclarées mais jamais utilisées dans le code, elles étaient embarquées pour rien dans le `.hpi`.
