---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: MODULE 5 — Technologie, Infrastructure & Outils
info: |
  ## Cours Agentic AI
# apply UnoCSS classes to the current slide
class: text-center
# https://sli.dev/features/drawing
pdf: true
export: true
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable Comark Syntax: https://comark.dev/syntax/markdown
comark: true
download: true
# duration of the presentation
duration: 35min
---
# Module 5 : Technologie, Infrastructure & Outils
## Déployer, orchestrer et évaluer des agents dans le monde réel

<div class="pt-12">
  <span class="px-3 py-1 rounded bg-sky-600/60 text-sm font-mono">Stack technique, Sécurité & MLOps</span>
  <p class="mt-4 text-gray-300 text-sm">Focus d'architecture : Modèle hébergé souverain / Harnais d'exécution local</p>
</div>


---
layout: center
---

# Une architecture classique : le principe en une phrase

<div class="text-xl font-bold my-4">
Votre <span class="text-primary">interface</span> reste chez vous,<br/>
l'<span class="text-primary">intelligence</span> part dans un cloud souverain.
</div>

<div class="grid grid-cols-3 gap-4 text-sm mt-6">
  <div class="border rounded-lg p-3 shadow-sm">💻 <b>Le logiciel client</b><br/>AnythingLLM, Claude Code… installés sur votre machine, avec vos habitudes</div>
  <div class="border rounded-lg p-3 shadow-sm">🔌 <b>L'API</b><br/>un simple « lien », une convention de fait (format compatible OpenAI, normalisée par les passerelles type LiteLLM)</div>
  <div class="border rounded-lg p-3 shadow-sm">🧠 <b>Le modèle</b><br/>hébergé sur l'infrastructure fédérée ILaaS</div>
</div>

<div class="mt-6 text-sm opacity-80">
Comme la TV branchée sur un service de streaming : le contenu est produit ailleurs, on le consomme chez soi avec sa télécommande — sans verrouillage.
</div>

<!-- Notes : Analogie messagerie aussi possible (Thunderbird + serveur de mail). Le point clé : on découple l'outil du cerveau. C'est ce qui rend la mutation du métier absorbable. Nuance : ce qui transite (diffs, snippets) reste de l'IP sensible — à cadrer par contrat/DPA. -->

---
layout: two-cols-header
---

# L'architecture

::left::

<diagramme>
<div class="border rounded-lg p-4 mb-3 shadow-sm bg-white">
  <b>VOTRE MACHINE</b><br/>
  · Claude Code (harnais + boucle agentique)<br/>
  · Skills académiques (markdown)<br/>
  · Fichiers, code, données <i>restent ici</i>
</div>
<div class="text-center text-2xl">⬇ 🔑 API (Bearer)</div>
<div class="border rounded-lg p-4 shadow-sm bg-white">
  <b>ex. ILaaS — cloud souverain de l'ESR</b><br/>
  · Passerelle LiteLLM<br/>
  · Modèles : Qwen, Mistral, Gemma…<br/>
  · <i>devllm.ilaas.fr</i>
</div>
</diagramme>

::right::

<div class="pl-6">

### Ce qui reste chez nous
- Documents, base vectorielle, embeddings
- Code, dépôt, hooks de sécurité
- Les traces / logs d'exécution *(le calcul du raisonnement, lui, s'exécute côté modèle distant)*

### Ce qui part dans le cloud
- Uniquement le **contexte nécessaire**
- Extraits de documents, snippets de code
- Le calcul du modèle, puis le retour

<br/>

✓ Confidentialité maîtrisée — on n'expose jamais tout, seulement le strict nécessaire.

</div>

<!-- Notes : L'important à faire passer : seuls des fragments utiles quittent la machine. On ne « donne pas ses données » à l'IA. Mais même ces fragments (diffs, extraits) sont de l'IP sensible : cadrer par contrat, pas par supposition. -->

---

## Evolution des modèles ouverts : pourquoi c'est possible

![](/img/d7.png)

---
layout: default
---

# Étude de cas d'architecture : L'approche Claude Code

Comment concilier la puissance d'un agent de code avec les impératifs de conformité et de secret industriel ?

<div class="mt-3 flex justify-center [&>svg]:max-w-4xl">

```mermaid {scale:0.6}
flowchart LR
    subgraph LOCAL ["Poste Développeur / Bastion Local (On-Premise)"]
        direction TB
        CLI["💻 Harnais CLI (ex. Claude Code)"]
        FS[("📁 Code Source & Git")]
        SH["⚡ Shell / Bash / Sandbox"]
        CLI <-->|Lecture / Diff / Patch| FS
        CLI <-->|Exécution tests & builds| SH
    end

    subgraph CLOUD ["Cloud Souverain / Enclave Sécurisée (AWS Bedrock EU, Private API)"]
        direction TB
        LLM["🧠 Modèle de Fondation<br/><i>(Qwen, Mistral Large, Llama — open-weights, région souveraine)</i>"]
    end

    CLI -->|"1. Tokens de contexte (Diffs, logs, prompt)"| LLM
    LLM -->|"2. Tokens d'intention (Tool call JSON)"| CLI

    style LOCAL fill:#f8fafc,stroke:#0284c7,stroke-width:2px,color:#000
    style CLOUD fill:#f0fdf4,stroke:#16a34a,stroke-width:2px,color:#000
    style CLI fill:#0284c7,color:#fff
    style LLM fill:#16a34a,color:#fff
```

</div>

<div class="grid grid-cols-3 gap-3 mt-4 text-xs">
  <div class="p-2 border rounded bg-gray-50 dark:bg-gray-800">
    <b>1. L'Agent s'exécute en local :</b> Les actions (git commit, build, tests) tournent directement dans l'environnement du dev, sous son contrôle d'approbation.
  </div>
  <div class="p-2 border rounded bg-gray-50 dark:bg-gray-800">
    <b>2. Pas de stockage tiers :</b> Le code n'est jamais indexé de façon permanente dans une base vectorielle externe ; seuls les snippets actifs transitent.
  </div>
  <div class="p-2 border rounded bg-gray-50 dark:bg-gray-800">
    <b>3. Enclave d'inférence contractuelle :</b> Le modèle tourne dans une région souveraine ; la non-rétention des logs et le non-usage des prompts sont <b>garantis par contrat (DPA / clause fournisseur)</b>, à vérifier modèle par modèle.
  </div>
</div>


---
layout: default
---

# Architecture Réelle : L'intermédiaire du Routeur Local

En pratique, l'agent ne contacte pas directement le Cloud : il passe par une **passerelle locale** (ex. *9Router*, *LiteLLM*).

<div class="mt-2 flex justify-center [&>svg]:max-w-4xl">

```mermaid {scale: 0.5}
flowchart LR
    subgraph MACHINE ["Poste Développeur / Bastion Local"]
        direction TB
        CLI["💻 Harnais CLI / IDE<br/><i>(Claude Code, Cursor)</i>"]

        subgraph ROUTER ["Routeur Local (ex. 9Router / LiteLLM)"]
            direction TB
            ADAPT["🔄 Adaptateur de protocoles<br/><i>(OpenAI ↔ Anthropic ↔ Bedrock)</i>"]
            CACHE["⚡ Filtre / Compression tokens / Fallback"]
            SLM["🦙 Modèle Local (Ollama / Small SLM)<br/><i>(Tâches légères, triage, anonymisation PII)</i>"]
        end

        CLI <-->|Requête standard /v1| ROUTER
    end

    subgraph CLOUD ["Cloud Souverain / Enclave Sécurisée"]
        BIG_LLM["🧠 Modèle Lourd de Raisonnement<br/><i>(Qwen, Mistral Large — open-weights disponibles en cloud ESR)</i>"]
    end

    ROUTER <-->|"Tâches complexes & raisonnement (chiffré)"| BIG_LLM

    style MACHINE fill:#f8fafc,stroke:#0284c7,stroke-width:2px,color:#000
    style ROUTER fill:#f0f9ff,stroke:#0369a1,stroke-width:1px,stroke-dasharray: 4 4,color:#000
    style CLOUD fill:#f0fdf4,stroke:#16a34a,stroke-width:2px,color:#000
    style CLI fill:#0284c7,color:#fff
    style BIG_LLM fill:#16a34a,color:#fff
    style SLM fill:#0284c7,color:#fff
```

</div>

<!-- Notes : DeepSeek-R1 ne figure pas au catalogue ILaaS actuel ; il s'utilise en auto-hébergement. La « souveraineté » porte sur l'infrastructure (localisation/hébergeur ESR), pas sur l'origine du modèle (Qwen = Alibaba, non européen). -->

---
layout: default
---

# Architecture Réelle : L'intermédiaire du Routeur Local

En pratique, l'agent ne contacte pas directement le Cloud : il passe par une **passerelle locale** (ex. *9Router*, *LiteLLM*).


<div class="grid grid-cols-3 gap-3 mt-3 text-xs">

<div class="p-2 border rounded bg-gray-50 dark:bg-gray-800">
  <div class="font-bold text-sky-600 mb-1">1. Normalisation de protocole</div>
  <p class="text-gray-500">
    L'agent parle en standard (OpenAI <code>/v1/chat</code>). Le routeur traduit à la volée vers l'API cible (Anthropic natif, AWS Bedrock EU, Azure OpenAI ou Vertex) sans toucher au harnais.
  </p>
</div>

<div class="p-2 border rounded bg-gray-50 dark:bg-gray-800">
  <div class="font-bold text-sky-600 mb-1">2. Tâches traitées 100% en local</div>
  <p class="text-gray-500">
    Avant d'appeler le Cloud, le routeur peut déléguer à un petit modèle local (SLM via Ollama) : classification de commande, masquage de données sensibles (PII) ou compression de logs de build.
  </p>
</div>

<div class="p-2 border rounded bg-gray-50 dark:bg-gray-800">
  <div class="font-bold text-sky-600 mb-1">3. Résilience & Fallback automatique</div>
  <p class="text-gray-500">
    En cas de saturation de quota, de <i>rate-limit 429</i> ou de panne réseau sur le cloud souverain principal, 9Router bascule instantanément vers un modèle de secours sans interrompre le développeur.
  </p>
</div>

</div>

<div class="mt-2 p-1.5 bg-sky-50 dark:bg-sky-950 border border-sky-400/40 rounded text-center text-[11px]">
  💡 <b>Gain d'architecture :</b> Le développeur manipule un point de terminaison unique (`http://localhost:port`). Toute la complexité d'authentification, de quotas et de conformité est encapsulée dans le routeur.
</div>

<!-- Notes : ⚠️ Le fallback multi-modèles casse la reproductibilité : documenter un « modèle de référence » (model of record) par tâche, et évaluer les évals sur le modèle effectivement utilisé. Le masquage PII n'est pas une fonctionnalité cœur de 9Router/LiteLLM — précisez si vous exigez une étape d'anonymisation séparée. -->

---
layout: default
---

# 2. Panorama des Frameworks Agentiques

Le paysage a mûri : on passe des bibliothèques "magiques" aux moteurs de graphes d'états stricts.

<div class="grid grid-cols-3 gap-3 mt-4 text-xs">

<div class="p-3 border rounded bg-gray-50 dark:bg-gray-800">
  <div class="font-bold text-sm text-sky-500 mb-1">LangGraph</div>
  <div class="text-gray-400 mb-2 italic">Standard d'ingénierie pour workflows complexes</div>
  <ul class="space-y-1">
    <li>Représente les flux sous forme de <b>graphe d'états</b> : acyclique (workflow linéaire) ou <b>cyclique</b> (boucles agentiques retry / re-planification)</li>
    <li>Persistance native de l'état (State Machine)</li>
    <li>Idéal pour le <i>Human-in-the-loop</i> et le time-travel debugging</li>
  </ul>
  <div class="mt-2 text-[10px] text-emerald-600 font-bold">🎯 Choix recommandé en production industrielle</div>
</div>

<div class="p-3 border rounded bg-gray-50 dark:bg-gray-800">
  <div class="font-bold text-sm text-indigo-500 mb-1">CrewAI / Microsoft Agent Framework</div>
  <div class="text-gray-400 mb-2 italic">Orientés collaboration multi-agents</div>
  <ul class="space-y-1">
    <li>Basés sur la personnification (Rôles, Objectifs, Backstory)</li>
    <li>Prise en main très rapide pour le prototypage</li>
    <li>Moins déterministes ; attention aux boucles d'échanges infinies</li>
    <li>⚠️ <b>AutoGen : maintenance mode depuis oct. 2025</b> → sur les nouveaux projets, préférer <b>AG2</b> (fork actif) ou le <b>Microsoft Agent Framework</b></li>
  </ul>
  <div class="mt-2 text-[10px] text-amber-500 font-bold">🎯 Idéal pour : POCs, simulations, remue-méninges</div>
</div>

<div class="p-3 border rounded bg-gray-50 dark:bg-gray-800">
  <div class="font-bold text-sm text-purple-500 mb-1">Harnais Dédiés (Claude Code, OpenHands)</div>
  <div class="text-gray-400 mb-2 italic">Agents verticaux spécialisés code/système</div>
  <ul class="space-y-1">
    <li>Intégration CLI native, gestion fine des tokens et des patchs git</li>
    <li>Boucle d'auto-réflexion branchée directement sur les linters</li>
    <li>Pas besoin d'écrire le graphe : l'agent est clé en main</li>
  </ul>
  <div class="mt-2 text-[10px] text-sky-500 font-bold">🎯 Idéal pour : Développement logiciel assisté</div>
</div>

</div>

<div class="mt-3 p-2 bg-gray-100 dark:bg-gray-800 rounded text-center text-xs">
  💡 <b>Conseil d'architecture :</b> Évitez les sur-couches abstraites pour les tâches critiques. Un script Python typé appelant le SDK officiel natif + Pydantic suffit souvent.
</div>

---
layout: default
---

# 3. Le standard MCP (Model Context Protocol)

Comment standardiser l'exposition des outils internes à n'importe quel agent ?

<div class="grid grid-cols-2 gap-6 mt-4 items-center">
<div>

- **La fin des silos :** Un standard ouvert (Open Source) initié fin 2024 pour unifier la connexion modèle ↔ outils.
- **Principe Client / Serveur :**
  - **Serveur MCP :** Expose des ressources, des prompts et des outils via **stdio** (en local) ou **Streamable HTTP** (à distance — un seul endpoint REST, auth OAuth possible). ⚠️ Le transport historique « SSE / HTTP » est **déprécié** depuis la spec **2025-03-26**.
  - **Client MCP :** Votre application, votre IDE (Cursor) ou un agent CLI (Claude Code).
- **Écosystème unifié :** Un serveur MCP écrit pour PostgreSQL ou GitLab fonctionne instantanément avec n'importe quel client compatible.
- ⚠️ **Serveurs de référence obsolètes :** `@modelcontextprotocol/server-postgres` et `mcp/github` sont **archivés/dépréciés depuis 2025** (le serveur Postgres présente un bypass de la restriction `read_only` par injection SQL). Utilisez des serveurs maintenus ou un wrapper basé sur le **SDK officiel**.

</div>

<div class="p-3 bg-gray-900 text-gray-100 rounded-lg font-mono text-[11px] leading-relaxed">
  <div class="text-sky-400 font-bold">// Configuration Claude Code / Cursor (mcp.json) :</div>
  {<br/>
  &nbsp;&nbsp;"mcpServers": {<br/>
  &nbsp;&nbsp;&nbsp;&nbsp;"github-officiel": {<br/>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"type": "http",<br/>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"url": "https://api.githubcopilot.com/mcp/",<br/>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"headers": { "Authorization": "Bearer &lt;GITHUB_TOKEN&gt;" }<br/>
  &nbsp;&nbsp;&nbsp;&nbsp;}<br/>
  &nbsp;&nbsp;}<br/>
  }
</div>
</div>

<!-- Notes : Pour PostgreSQL privilégiez un serveur maintenu (ex. « Postgres MCP Pro ») ou un petit wrapper SDK officiel, avec utilisateur en lecture seule stricte. Insistez : un méta-outil qui exécute du SQL doit TOUJOURS être en lecture seule a minima (règle de moindre privilège). -->

<style>
li {
  font-size: 0.78em;
}
</style>

---
layout: default
---

# 4. Mémoire & Connaissances : RAG vs Contexte géant

L'avènement des contextes géants — **1M+ tokens chez les fournisseurs frontières** (Claude Sonnet 4, Gemini 2.x), **128k à 262k sur les modèles hébergés ILaaS** — et du **Prompt Caching** transforme la gestion mémoire :

<div class="grid grid-cols-2 gap-6 mt-4 text-xs">

<div class="p-4 border rounded-xl bg-gray-50 dark:bg-gray-800">
  <div class="font-bold text-sm text-sky-500 mb-2">L'approche RAG Classique (Vector Search)</div>
  <ul class="space-y-2">
    <li><b>Principe :</b> Découpage en chunks de <b>~200 à 500 tokens</b> (≈ 150 à 350 mots) avec <b>overlap de 10–20 %</b> + recherche par similarité cosinus.</li>
    <li><b>Quand l'utiliser ?</b>
      - Corpus de taille massive (> 100 millions de tokens, ex. tous les emails de l'entreprise).
      - Besoin de cloisonner strictement les droits d'accès par utilisateur.
    </li>
    <li><b>Point faible :</b> Incapable de répondre à une question transversale (*« Quels sont tous les projets ayant dépassé leur budget en 2025 ? »*).</li>
  </ul>
</div>

<div class="p-4 border-2 border-sky-500 rounded-xl bg-sky-50 dark:bg-sky-950">
  <div class="font-bold text-sm text-sky-600 mb-2">Contexte géant + Prompt Caching (La nouvelle norme)</div>
  <ul class="space-y-2">
    <li><b>Principe :</b> Chez les fournisseurs frontières, on injecte l'intégralité d'un repository git dans le contexte. Sur ILaaS (128k–262k), on adapte : RAG d'abord, injection partielle ensuite.</li>
    <li><b>Prompt Caching :</b> Les tokens du préfixe documenté sont mis en cache côté serveur :
      - <b>Coût du préfixe mis en cache divisé par 10</b> (≈ 90 % d'économie)
      - TTFT réduit jusqu'à ~80 % sur les longs prompts — <i>uniquement si le préfixe est strictement identique</i> (aucune insertion au milieu) et selon la politique du fournisseur.
    </li>
    <li>⚠️ <b>Garder la mesure :</b> au-delà de ~150–200k tokens, coût unitaire élevé et effets <i>lost-in-the-middle</i> (le modèle « oublie » le milieu du contexte).</li>
  </ul>
</div>

</div>

<div class="mt-4 p-2 bg-sky-50 dark:bg-sky-950 border border-sky-400 rounded text-center text-xs">
  💡 <b>Design Pattern moderne :</b> RAG pour filtrer les 5 gros documents pertinents ➔ Injection in extenso dans le contexte avec Caching (lorsque la fenêtre le permet).
</div>

<!-- Notes : Les chiffres 90 % et ~80 % correspondent aux gains revendiqués par les fournisseurs frontières sur des préfixes longs et stables — à re-mesurer sur votre fournisseur (LiteLLM/ILaaS), pas à transposer tels quels. -->

---
layout: default
---

# 5. Orchestration en Production & Observabilité

Les agents en production introduisent des modes d'échec non déterministes qu'il faut tracer méthodiquement

<div class="grid grid-cols-2 gap-6 mt-4 text-xs">

<div>

### Ce qui doit être supervisé (Traces & Logs) :
- **Arbre de décision (Spans) :** Tracer chaque étape : `Prompt ➔ Réflexion ➔ Outil appelé ➔ Erreur ➔ Re-jeu`.
- **Dérive de budget :** Alertes en temps réel sur la consommation de tokens par utilisateur/tâche.
- **Taux de conversion :** Pourcentage d'objectifs atteints sans intervention humaine.

### La stack d'observabilité agentique :
- **Standards ouverts :** OpenTelemetry (OTel)
- **Outils spécialisés :** Langfuse, Arize Phoenix, LangSmith

</div>

<div class="p-3 bg-gray-900 text-gray-100 rounded-lg font-mono text-[11px]">
  <div class="text-sky-400 font-bold">// Exemple de trace Langfuse / OpenTelemetry :</div>
  <br/>
  TRACE [Session-8924] - 42.1s - $0.18<br/>
  ├── 🟢 LLM Call: Planner (Qwen / Mistral — cible souveraine) [820ms]<br/>
  ├── 🟢 Tool Call: search_db(query="user_42") [120ms]<br/>
  ├── 🔴 Tool Call: run_migration() [TIMEOUT 10s]<br/>
  │   └── ⚠️ Error captured: Connection timed out<br/>
  ├── 🟢 LLM Call: Evaluator (Analyse de l'erreur) [1.1s]<br/>
  └── 🟢 Tool Call: retry_with_backoff() [450ms] ✅
</div>

</div>

---
layout: default
---

# 6. Évaluation des agents : Au-delà du BLEU/ROUGE

Comment savoir si votre agent s'améliore après un changement de prompt ou d'outil ?

<div class="grid grid-cols-3 gap-3 mt-4 text-xs">

<div class="p-3 border rounded bg-gray-50 dark:bg-gray-800">
  <div class="font-bold text-sm text-sky-500 mb-1">SWE-bench</div>
  <div class="text-gray-400 mb-2 italic">Le standard d'or du génie logiciel</div>
  <p>L'agent reçoit une <i>Issue GitHub réelle</i> et le repo complet. Il doit écrire le patch.</p>
  <div class="mt-2 text-emerald-500 font-bold">Métrique : taux de résolution (pass@1) sur deux ensembles — <b>FAIL_TO_PASS</b> (les tests qui échouaient passent grâce au patch) + <b>PASS_TO_PASS</b> (pas de régression). État de l'art ≈ 70 % sur Verified (2025‑2026), ≈ 40 % sur SWE-bench Pro.</div>
</div>

<div class="p-3 border rounded bg-gray-50 dark:bg-gray-800">
  <div class="font-bold text-sm text-indigo-500 mb-1">GAIA & BrowseComp</div>
  <div class="text-gray-400 mb-2 italic">Recherche experte & scénarios « assistant »</div>
  <p>BrowseComp (OpenAI, 2025) : formuler une question d'expert, chercher sur le web et croiser les sources. GAIA : scénarios « assistant » classiques (fichiers, web, tableurs, e-mails).</p>
  <div class="mt-2 text-indigo-500 font-bold">Métrique : Exact Match (EM) sur la réponse factuelle finale (notation manuelle d'experts pour BrowseComp).</div>
</div>

<div class="p-3 border-2 border-sky-500 rounded bg-sky-50 dark:bg-sky-950">
  <div class="font-bold text-sm text-sky-600 mb-1">Evals Métier Unitaires</div>
  <div class="text-gray-400 mb-2 italic">Votre propre harnais de test CI/CD</div>
  <p>Créer 30 scénarios représentatifs de votre SI avec assertions sur l'état final du système (BD, fichiers créés).</p>
  <div class="mt-2 text-sky-600 font-bold">Métrique : Taux de succès à la non-régression avant déploiement.</div>
</div>

</div>

<div class="mt-4 p-2 bg-gray-100 dark:bg-gray-800 rounded text-center text-xs">
  🎯 <b>Principe fondamental :</b> On n'évalue pas un agent sur ce qu'il <i>dit</i>, mais sur l'<i>état final du système</i> après ses actions.
</div>

<!-- Notes : L'EM de GAIA/BrowseComp pénalise les réponses correctes mais reformulées → compléter systématiquement par un juge LLM (LLM-as-a-judge) sur vos évals métier. -->

---
layout: default
---

# Synthèse : Checklist d'industrialisation

Pour passer un système agentique en production sereinement :

<div class="p-4 bg-sky-50 dark:bg-sky-950 border border-sky-500 rounded-xl mt-4">
  <ul class="text-xs space-y-2 text-gray-700 dark:text-gray-200">
    <li>🔒 <b>Architecture réseau :</b> Privilégier le harnais local proche des données sensibles avec appels vers des endpoints LLM sous contrat entreprise (<b>non-rétention négociée par DPA</b>, à vérifier modèle par modèle).</li>
    <li>🔌 <b>Interconnexion standardisée :</b> Adopter <b>MCP</b> (transports stdio / Streamable HTTP) pour vos connecteurs internes afin d'éviter la dépendance à un framework applicatif.</li>
    <li>🛡️ <b>Sécurité d'exécution :</b> Isoler tout outil de code dans une sandbox étanche (MicroVM / conteneurs éphémères non root, <i>pas Docker "nu"</i>) avec confirmation humaine sur les actions critiques.</li>
    <li>📊 <b>Observabilité complète :</b> Instrumenter les spans d'exécution via OpenTelemetry / Langfuse dès le premier jour pour diagnostiquer les boucles.</li>
    <li>🧪 <b>Culture d'évaluation continue :</b> Évaluer systématiquement toute montée de version de modèle ou de prompt sur un jeu d'épreuve automatisé (Evals) — en documentant le <b>modèle de référence</b> utilisé pour rester reproductible.</li>
  </ul>
</div>

---
layout: end
class: text-center
---

# Une première démo
### Créer une page web à l'aide de claude code connecté à iLaaS


---
layout: end
class: text-center
---

# Fin du Module 5
### Questions, outillage & discussions d'infrastructure !
