E29 — EVOXA Search Architecture
Engineering Specification

Pertenece a: Engineering Specification
Volumen: volume-09-engineering
Capítulo: E29 — Search Architecture
Anterior: E28 — Read Model Architecture
Siguiente: E30 — Reporting Architecture

1. Propósito

E29 define la arquitectura de Search de EVOXA.

Search establece cómo el sistema:

indexa información;
encuentra información mediante texto o criterios;
ejecuta búsquedas;
calcula relevancia;
aplica filtros y facetas;
soporta autocomplete;
gestiona ranking;
mantiene índices;
sincroniza índices con sus fuentes;
garantiza aislamiento, seguridad y observabilidad.

La pregunta fundamental es:

¿Cómo encuentra EVOXA información relevante de forma rápida, precisa, segura y escalable?

2. Concepto Fundamental
Source
   │
   ▼
Search Projection
   │
   ▼
Search Index
   │
   ▼
Search Query
   │
   ├── text
   ├── filters
   ├── facets
   ├── sorting
   └── ranking
   │
   ▼
Search Result

Search debe considerarse una capacidad especializada de lectura.

3. Search vs Query
Query
    ↓
structured retrieval

Search
    ↓
discovery / relevance retrieval

Una Query puede preguntar:

customerId = 123

Search puede preguntar:

"enterprise analytics platform"

y devolver resultados ordenados por relevancia.

4. Search vs Read Model
Read Model
    ↓
optimized read representation

Search Index
    ↓
optimized discovery representation

Un Search Index puede derivarse de un Read Model:

Read Model
    ↓
Search Projection
    ↓
Search Index

pero no necesariamente tiene que hacerlo.

5. Search Architecture
                         SEARCH ARCHITECTURE
                                  │
             ┌────────────────────┼────────────────────┐
             ▼                    ▼                    ▼
          Sources             Read Models           External
             │                    │                    │
             └────────────────────┼────────────────────┘
                                  ▼
                         Search Projection
                                  │
                                  ▼
                            Search Index
                                  │
                    ┌─────────────┼─────────────┐
                    ▼             ▼             ▼
                 Search       Autocomplete     Facets
                    │             │             │
                    └─────────────┼─────────────┘
                                  ▼
                            Search Result
6. Search Characteristics

Una arquitectura Search debe ser:

fast
scalable
relevance-aware
filterable
observable
secure
tenant-aware
rebuildable
versionable
7. Search Types

EVOXA debe contemplar:

Full-Text Search
Keyword Search
Exact Search
Prefix Search
Autocomplete
Fuzzy Search
Faceted Search
Filtered Search
Semantic Search
Hybrid Search
Geospatial Search
Temporal Search
Cross-Entity Search

No todos los casos requieren el mismo motor.

8. Full-Text Search

Busca términos dentro de contenido:

"engineering architecture"

Puede utilizar:

tokenization
stemming
ranking
field weighting
9. Exact Search

Busca coincidencias exactas:

customerCode = "C-10291"

Debe preferirse para identificadores y valores donde la relevancia textual no aplica.

10. Prefix Search

Ejemplo:

"engi"

puede devolver:

engineering
engineer
engineering-module

Es especialmente útil para autocomplete.

11. Fuzzy Search

Permite tolerar errores:

"enginering"

→

"engineering"

Debe controlarse para evitar resultados inesperados y costes excesivos.

12. Faceted Search

Permite:

query
+
filters
+
facets

Ejemplo:

Search:
"platform"

Filters:
domain = Engineering

Facets:
status
owner
type
13. Filtered Search

La búsqueda puede restringirse mediante:

tenant
domain
status
category
date
owner
permissions
14. Semantic Search

Puede utilizar representaciones semánticas:

Query
 ↓
Embedding
 ↓
Vector Search
 ↓
Semantic Results

La semántica de búsqueda debe permanecer separada del almacenamiento vectorial concreto.

15. Hybrid Search

Puede combinar:

lexical search
+
semantic search

Ejemplo:

BM25
+
Vector Similarity

para mejorar recall y relevance.

16. Geospatial Search

Puede soportar:

near
within radius
bounding box
distance

cuando el dominio lo requiera.

17. Temporal Search

Puede buscar:

createdAt
updatedAt
validFrom
validTo
eventTime

con rangos temporales.

18. Cross-Entity Search

Puede buscar múltiples tipos:

Customer
Order
Project
Document
Agent
Workflow

en una misma experiencia.

Debe definirse claramente el contrato de cada resultado.

19. Search Index

El Search Index representa una estructura especializada para recuperación.

Conceptualmente:

SearchIndex
├── documents
├── fields
├── analyzers
├── indexes
├── ranking configuration
└── version
20. Search Document

Un documento indexado no tiene que coincidir con una entidad:

Domain Entity ≠ Search Document

Ejemplo:

Customer

puede convertirse en:

CustomerSearchDocument
├── id
├── displayName
├── description
├── status
├── tags
└── searchableText
21. Search Document Design

Debe diseñarse desde:

search intent
query patterns
ranking needs
filter needs
result needs
22. Search Projection

Flujo:

Source
  ↓
Search Projection
  ↓
Search Document
  ↓
Search Index

La Search Projection transforma información de origen en una representación indexable.

23. Search Projection vs Read Projection

Read Projection:

Source
 ↓
Read Model

Search Projection:

Source
 ↓
Search Document

Pueden compartir parte de su lógica, pero no deben acoplarse artificialmente.

24. Search Source

La fuente puede ser:

Domain State
Read Model
Events
CDC
External Dataset

Debe documentarse la fuente canónica.

25. Source of Truth

El Search Index no debería convertirse accidentalmente en la única fuente de verdad:

Source of Truth
      │
      ▼
Search Index

El índice es normalmente derivado.

26. Index Rebuild

Debe ser posible:

Source
 ↓
Replay / Extraction
 ↓
Search Projection
 ↓
New Index

sin depender exclusivamente del estado actual del índice.

27. Index Versioning

Puede utilizarse:

search-index-v1
search-index-v2

para permitir:

migration
rebuild
blue-green deployment
rollback
28. Index Migration

Patrón:

V1 ACTIVE
    │
    ├── build V2
    │
    ▼
V2 VALIDATE
    │
    ▼
V2 ACTIVE
    │
    ▼
V1 RETIRE
29. Zero-Downtime Reindexing

Preferir:

Build New Index
      ↓
Validate
      ↓
Switch Alias
      ↓
Retire Old Index

sobre modificar destructivamente el índice activo.

30. Search Alias

Un alias lógico puede apuntar a:

evoxa-search
       ↓
evoxa-search-v7

y posteriormente:

evoxa-search
       ↓
evoxa-search-v8

Esto desacopla consumidores de versiones físicas.

31. Index Sharding

Para grandes volúmenes:

Index
 ├── Shard A
 ├── Shard B
 ├── Shard C
 └── Shard D

La estrategia debe considerar:

tenant
document count
query volume
geography
32. Index Replication

Puede utilizar:

Primary
 ├── Replica
 ├── Replica
 └── Replica

para:

availability
read throughput
fault tolerance
33. Tenant Search Isolation

En multi-tenant:

Search Request
      ↓
Tenant Context
      ↓
Search Scope
      ↓
Authorized Results

Nunca debe confiarse únicamente en un filtro proporcionado por el cliente.

34. Tenant-Aware Indexing

Opciones:

shared index + tenant field

o:

tenant-specific index

o:

tenant partitions

La elección depende de:

tenant count
tenant size
isolation requirements
operational cost
35. Security Filtering

La búsqueda debe garantizar que:

Unauthorized Document

no aparezca:

even as a search result

No basta con ocultar el documento después del ranking.

36. Field-Level Security

Los resultados pueden requerir:

field filtering
secure projection
redaction
authorization-aware projection
37. Search Authorization

Debe existir una relación explícita:

Identity
 ↓
Authorization Policy
 ↓
Search Scope
 ↓
Search Query
38. Search Query

Conceptualmente:

SearchQuery
├── text
├── filters
├── facets
├── sort
├── pagination
├── fields
└── ranking
39. Search Text

Debe diferenciar:

user text
system query
structured filters

para evitar mezclar semánticas.

40. Search Filters

Puede soportar:

equals
notEquals
in
range
exists
prefix

según el motor.

41. Search Filter Allowlist

Los campos filtrables deben estar definidos:

allowedFilterFields

Evitar que el consumidor consulte arbitrariamente cualquier campo interno.

42. Search Sorting

Puede ordenar por:

relevance
date
name
priority
distance
custom score
43. Relevance

La relevancia debe considerar:

term match
field importance
frequency
recency
business signals
semantic similarity

según el caso.

44. Ranking

Conceptualmente:

Search Query
     ↓
Candidate Retrieval
     ↓
Ranking
     ↓
Top Results
45. Ranking Signals

Pueden incluir:

text relevance
field boost
recency
popularity
authority
quality
business priority
semantic similarity

Cada señal debe estar justificada.

46. Field Boosting

Puede priorizar:

title > description > metadata

por ejemplo.

La configuración debe ser explícita y versionable.

47. Business Ranking

Un dominio puede requerir:

priority
status
customer tier
availability

como señales.

No debe confundirse con reglas de negocio transaccionales.

48. Ranking vs Business Rules
Business Rule
    ↓
determines validity

Ranking
    ↓
determines ordering

Una búsqueda no debería convertir ranking en autorización.

49. Search Pagination

Puede utilizar:

offset
cursor
search-after

según el motor.

Para grandes índices se recomienda una estrategia estable y eficiente.

50. Search Result

Conceptualmente:

SearchResult
├── items
├── total / estimatedTotal
├── facets
├── cursor
└── metadata
51. Search Hit

Cada resultado puede incluir:

SearchHit
├── id
├── type
├── score
├── source
└── highlights
52. Total Count

Debe distinguir:

exact total

de:

estimated total

cuando el motor optimiza conteos grandes.

53. Highlighting

Puede devolver:

matched fragments

para mostrar por qué un documento coincide.

Debe evitar exponer campos que el usuario no está autorizado a ver.

54. Facets

Los facets pueden incluir:

status
type
category
owner
tenant
date

según autorización y caso de uso.

55. Facet Security

Los facets pueden revelar información sensible incluso si los documentos no aparecen.

Por ejemplo:

"3 confidential documents"

puede ser una fuga de información.

La seguridad debe aplicarse también a agregaciones y facets.

56. Autocomplete

Flujo:

User types
    ↓
Prefix Query
    ↓
Autocomplete Index
    ↓
Suggestions

Debe optimizarse para latencia extremadamente baja.

57. Suggestion Model

Puede almacenar:

term
frequency
popularity
context
tenant

según el caso.

58. Autocomplete Isolation

Las sugerencias no deben revelar:

private entities
other tenant names
restricted terms
59. Spell Correction

Puede soportar:

typo
 ↓
correction

pero debe evitar correcciones que cambien significativamente la intención.

60. Synonyms

Puede definir:

car ↔ automobile

o términos específicos del dominio.

Los diccionarios de sinónimos deben ser versionados y gobernados.

61. Search Analyzer

El análisis puede incluir:

tokenization
normalization
lowercasing
stemming
stopwords
synonyms
language analysis
62. Language Awareness

Search debe considerar:

language
locale
accent
case
stemming
tokenization

cuando sea necesario.

63. Multilingual Search

Puede utilizar:

language-specific analyzers
per-language fields
language detection
cross-language search

según los requisitos.

64. Normalization

Puede normalizar:

case
accents
punctuation
spacing

pero debe preservar el valor original cuando sea necesario para display.

65. Searchable vs Display Fields

Debe distinguir:

searchable fields
filterable fields
sortable fields
display fields

No todos los campos deben habilitar todas las capacidades.

66. Search Field Classification

Ejemplo:

title
 ├── searchable
 ├── sortable
 └── display

description
 ├── searchable
 └── display

status
 ├── filterable
 ├── sortable
 └── display

internalNotes
 └── restricted
67. Search Data Minimization

Indexar sólo los datos necesarios:

Source
 ↓
Minimal Search Document

Evitar copiar:

secrets
credentials
unnecessary PII
internal metadata
68. Search Document Lifecycle
CREATE
  ↓
INDEX
  ↓
UPDATE
  ↓
REFRESH
  ↓
DELETE
  ↓
PURGE

según las políticas del sistema.

69. Update Propagation
Source Change
      ↓
Event / CDC
      ↓
Search Projection
      ↓
Index Update
70. Event-Driven Indexing

En un sistema event-driven:

EntityCreated
EntityUpdated
EntityDeleted

pueden producir:

IndexDocument
UpdateDocument
DeleteDocument
71. Idempotent Indexing

Procesar dos veces:

EntityUpdated
EntityUpdated

debe producir el mismo estado final.

72. Event Ordering

Cuando las actualizaciones dependen del orden:

Update V2
Update V3

no debe terminar:

V2 overwrites V3

por procesamiento fuera de orden.

73. Version Guard

El documento puede incluir:

sourceVersion

y rechazar actualizaciones antiguas:

incomingVersion < indexedVersion
74. Delete Semantics

Debe distinguir:

logical deletion
physical deletion
retention
75. Tombstone

Un evento:

EntityDeleted

puede provocar:

SearchIndex.delete(entityId)

y posteriormente una eliminación física del documento.

76. Index Refresh

Debe entenderse la diferencia entre:

document indexed

y:

document searchable

dependiendo del motor.

77. Search Consistency

Puede ser:

strong
near-real-time
eventual

El contrato debe declarar cuál aplica.

78. Search Freshness

Debe monitorizarse:

sourceTimestamp
indexTimestamp
lag
79. Search SLA

Ejemplo:

Index freshness:
P95 < 5 seconds

Search latency:
P95 < 150 ms
P99 < 500 ms

Los valores concretos deben definirse por caso de uso.

80. Search Performance

Debe optimizar:

query latency
indexing throughput
refresh latency
storage efficiency
network traffic
ranking cost
81. Search Complexity

Debe limitar:

query clauses
nested depth
wildcards
regex
fuzzy distance
vector candidates
aggregation complexity

para evitar consultas abusivas.

82. Expensive Search Protection

Las consultas potencialmente costosas deben poder:

reject
limit
throttle
rewrite
execute asynchronously

según el caso.

83. Wildcard Queries

Wildcards arbitrarios pueden ser costosos.

Debe existir:

allowlist
maximum complexity
field restrictions

cuando se permitan.

84. Regex Search

Debe tratarse como capacidad avanzada.

Evitar:

unbounded regex

en endpoints públicos.

85. Fuzzy Query Limits

Debe controlar:

maximum edit distance
minimum term length
maximum candidates
86. Search Rate Limiting

Puede limitar:

requests/sec
concurrent queries
autocomplete frequency
heavy searches

por:

tenant
user
API key
client
87. Search Caching

Puede cachear:

popular queries
autocomplete
facets
static filters

pero las claves deben incorporar el contexto necesario.

88. Search Cache Key

Puede incluir:

tenant
user scope
query
filters
locale
index version
ranking version
89. Cache Invalidation

Cuando cambia:

index version
ranking configuration
synonyms
authorization scope

las entradas relevantes pueden quedar inválidas.

90. Search Resilience

Debe considerar:

index unavailable
node unavailable
timeout
partial shard failure
network failure
corrupted index
91. Partial Search Failure

Debe definirse si el sistema:

fails entire request

o:

returns partial results

La elección debe formar parte del contrato.

92. Search Fallback

Posibles fallbacks:

cached results
simplified query
secondary index
database query

pero deben evitarse resultados engañosos.

93. Search Degradation

Puede degradarse:

semantic ranking
facets
high-cost highlighting
advanced filters

para preservar disponibilidad.

94. Search Observability

Métricas mínimas:

search_requests_total
search_errors_total
search_latency
search_timeout_total
search_results_count
search_zero_results_total
indexing_events_total
indexing_errors_total
indexing_lag
index_size
95. Search Quality Metrics

Además de infraestructura:

zero-result rate
click-through rate
conversion rate
precision
recall
ranking quality

cuando el producto lo requiera.

96. Search Relevance Evaluation

Debe poder compararse:

Ranking V1
vs
Ranking V2

mediante:

test queries
judged results
offline evaluation
A/B tests
97. Search Golden Queries

Debe existir un conjunto de consultas representativas:

Golden Query Set
├── common queries
├── edge cases
├── typo cases
├── multilingual cases
├── security cases
└── zero-result cases
98. Search Regression Testing

Cada cambio de:

analyzer
synonym
ranking
schema
projection
index version

debe poder evaluarse contra las consultas de referencia.

99. Search Security Testing

Debe comprobar:

tenant isolation
authorization
field restrictions
facet leakage
highlight leakage
autocomplete leakage
100. Search Integration Testing

Debe probar:

Source
 ↓
Event
 ↓
Search Projection
 ↓
Index
 ↓
Search Query
 ↓
Result
101. Search Rebuild Testing

Debe verificar:

rebuild correctness
document counts
version consistency
query equivalence
ranking behavior
102. Search Disaster Recovery

Debe definirse:

index backup
rebuild source
rebuild duration
RPO
RTO
103. Search Backup

Cuando el índice pueda reconstruirse fácilmente:

Source of Truth

puede ser más importante que backups completos del índice.

Pero los backups pueden reducir el tiempo de recuperación.

104. Search Data Retention

Debe declarar:

index retention
document retention
deleted document purge
snapshot retention
105. Search Deletion Compliance

La eliminación de información sensible debe propagarse a:

Search Index
Autocomplete
Cache
Vector Index
Secondary Indexes

cuando corresponda.

106. Search and Vector Index

Para Semantic Search:

Document
 ↓
Embedding
 ↓
Vector Index

El vector index debe considerarse otro artefacto derivado.

107. Vector Search

Conceptualmente:

Query
 ↓
Embedding
 ↓
Vector Similarity
 ↓
Candidates
 ↓
Ranking
108. Hybrid Search

Arquitectura:

              Query
                │
        ┌───────┴────────┐
        ▼                ▼
   Lexical Search   Vector Search
        │                │
        └───────┬────────┘
                ▼
             Fusion
                │
                ▼
             Ranking
                │
                ▼
             Results
109. Vector Security

Los embeddings pueden codificar información sensible.

Por tanto:

Vector Index

también requiere:

tenant isolation
authorization
retention
deletion
access control
110. Semantic Search Governance

Debe documentarse:

embedding model
embedding version
dimensions
distance metric
index version
rebuild strategy
111. Embedding Versioning

Un cambio de modelo:

Embedding Model V1
        ↓
Embedding Model V2

normalmente requiere:

re-embedding
re-indexing
validation
112. Search and AI

Search puede proporcionar contexto para:

AI
Agents
RAG
recommendation
knowledge retrieval

pero el retrieval debe respetar los mismos límites de autorización que una Query normal.

113. Search and RAG
User Query
    ↓
Search / Retrieval
    ↓
Authorized Documents
    ↓
Context Projection
    ↓
AI Model

El modelo de IA nunca debe recibir documentos que el usuario no pueda consultar.

114. Search and Agents
Agent
  ↓
Search Capability
  ↓
Authorization
  ↓
Search Index
  ↓
Results

Search debe ser una capability controlada, no un acceso directo al índice.

115. Search and API Architecture
API
 ↓
Search Contract
 ↓
Search Service
 ↓
Search Engine
 ↓
Search Result

La API no debe exponer detalles internos del motor.

116. Search Abstraction

Evitar que los consumidores dependan directamente de:

Lucene
Elasticsearch
OpenSearch
Solr
Vector DB

cuando existe un boundary de Search propio de EVOXA.

117. Search Adapter

Puede existir:

Search Service
      │
      ▼
Search Adapter
      │
 ┌────┼────┐
 ▼    ▼    ▼
Engine A B C

Esto permite evolución tecnológica.

118. Search Provider Contract

Debe abstraer:

index
update
delete
search
suggest
aggregate
health

según las capacidades realmente soportadas.

119. Search Engine Capability Matrix

Cada engine debe evaluarse respecto a:

full-text
facets
vector
geo
ranking
autocomplete
aggregations
filtering
scaling
availability
120. Search Governance

Cada Search Index debe declarar:

name
purpose
owner
source
projection
schema
engine
version
consumers
security
SLA
freshness
rebuild strategy
retention
121. Search Naming

Preferir:

customer-search
document-search
workflow-search
agent-search

y versiones internas:

customer-search-v3

cuando sean necesarias.

122. Search Ownership

El owner responde por:

schema
index
relevance
performance
security
freshness
rebuild
migration
123. Search Lifecycle
DEFINE
   ↓
DESIGN
   ↓
CREATE INDEX
   ↓
BACKFILL
   ↓
VALIDATE
   ↓
ACTIVATE
   ↓
SERVE
   ↓
OBSERVE
   ↓
EVOLVE
   ↓
REINDEX
   ↓
DEPRECATE
   ↓
REMOVE
124. Search Creation Criteria

Crear un Search Index cuando exista necesidad de:

full-text
relevance
fuzzy matching
autocomplete
facets
semantic retrieval
high-volume discovery
125. Search Avoidance Criteria

No utilizar Search cuando:

simple primary-key lookup suffices
simple indexed database query suffices
search complexity adds no value

Search debe resolver un problema real de retrieval.

126. Search Cost

Debe considerarse:

storage
indexing compute
query compute
replication
rebuild
embedding cost
operational complexity
127. Search Capacity Planning

Debe estimarse:

documents
document size
index growth
queries/sec
index updates/sec
storage growth
rebuild throughput
128. Search Operational States
CREATED
BUILDING
ACTIVE
DEGRADED
STALE
REINDEXING
FAILED
DEPRECATED
RETIRED
129. Search Health

Health debe incluir:

engine availability
index availability
index freshness
query latency
indexing lag
replication status
130. Definition of Done

E29 queda definido cuando EVOXA dispone de:

✓ Search definition
✓ Search boundary
✓ Search vs Query
✓ Search vs Read Model
✓ Search Architecture
✓ Search types
✓ Full-text search
✓ Exact search
✓ Prefix search
✓ Fuzzy search
✓ Faceted search
✓ Filtered search
✓ Semantic search
✓ Hybrid search
✓ Geospatial search
✓ Temporal search
✓ Cross-entity search
✓ Search Index
✓ Search Document
✓ Search Projection
✓ Source of Truth
✓ Index rebuild
✓ Index versioning
✓ Index migration
✓ Zero-downtime reindexing
✓ Search aliases
✓ Sharding
✓ Replication
✓ Tenant isolation
✓ Security filtering
✓ Field-level security
✓ Authorization
✓ Search Query
✓ Filters
✓ Sorting
✓ Relevance
✓ Ranking
✓ Field boosting
✓ Business ranking
✓ Pagination
✓ Search Results
✓ Hits
✓ Highlighting
✓ Facets
✓ Autocomplete
✓ Spell correction
✓ Synonyms
✓ Analyzers
✓ Multilingual search
✓ Field classification
✓ Data minimization
✓ Document lifecycle
✓ Event-driven indexing
✓ Idempotent indexing
✓ Event ordering
✓ Version guards
✓ Delete semantics
✓ Refresh semantics
✓ Search consistency
✓ Freshness
✓ Performance
✓ Complexity limits
✓ Rate limiting
✓ Caching
✓ Resilience
✓ Partial failure strategy
✓ Degradation
✓ Observability
✓ Search quality metrics
✓ Relevance evaluation
✓ Golden queries
✓ Regression testing
✓ Security testing
✓ Rebuild testing
✓ Disaster recovery
✓ Retention
✓ Compliance deletion
✓ Vector search
✓ Hybrid retrieval
✓ Embedding versioning
✓ AI integration
✓ RAG integration
✓ Agent integration
✓ API integration
✓ Search abstraction
✓ Search adapters
✓ Provider contract
✓ Capability matrix
✓ Governance
✓ Naming
✓ Ownership
✓ Lifecycle
✓ Creation criteria
✓ Avoidance criteria
✓ Cost
✓ Capacity
✓ Operational states
✓ Health
131. Position in Engineering Specification

La secuencia ahora queda:

E23 — Serialization Architecture
        ↓
E24 — Transformation Architecture
        ↓
E25 — Mapping Architecture
        ↓
E26 — Projection Architecture
        ↓
E27 — Query Architecture
        ↓
E28 — Read Model Architecture
        ↓
E29 — Search Architecture
        ↓
E30 — Reporting Architecture

Y la relación conceptual:

                    SOURCE
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
        READ MODEL          SEARCH PROJECTION
             │                   │
             ▼                   ▼
           QUERY             SEARCH INDEX
             │                   │
             │          ┌────────┼────────┐
             │          ▼        ▼        ▼
             │       TEXT      FACETS   VECTOR
             │          │        │        │
             └──────────┼────────┼────────┘
                        ▼
                     RESULTS
                        │
                        ▼
                    PROJECTION
                        │
                        ▼
                    CONSUMER

La frontera arquitectónica queda especialmente clara:

E27 Query
    │
    │ structured retrieval
    ▼
E28 Read Model
    │
    │ optimized read state
    ▼
E29 Search
    │
    │ discovery / relevance
    ▼
E30 Reporting
    │
    │ historical / analytical presentation
    ▼
Consumer

Principio central de E29:
Search es una capacidad derivada y especializada de retrieval. El índice debe optimizarse para descubrimiento, relevancia, filtrado y velocidad; debe permanecer reconstruible desde una fuente definida, respetar autorización y aislamiento multi-tenant, y no debe convertirse accidentalmente en la fuente canónica de verdad.

Siguiente: E30 — EVOXA Reporting Architecture.
