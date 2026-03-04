---
layout: page
title: BioConnect — Multi-Database Biological Knowledge Platform
img: assets/img/bioconnect_neo4j_diseases.png
importance: 4
category: AI projects
---

<div class="row mt-3">
    <div class="col-sm-8">
        <p>
            A research platform designed to empower biologists and pharmacologists in exploring
            <strong>human protein–drug–disease interaction networks</strong>, developed as a group project
            for the <em>Large Scale and Multi-Structured Databases</em> course at the
            <strong>University of Pisa</strong> (MSc in AI and Data Engineering, AY 2024–2025).
        </p>
        <p>
            BioConnect combines a <strong>MongoDB</strong> document store (proteins, drugs, users) with a
            <strong>Neo4j</strong> graph database (interaction networks) behind a <strong>Spring Boot</strong>
            REST API — enabling both rich document queries and graph traversal over more than
            20k proteins and 16k drugs sourced from UniProt, DrugBank, and DisGeNET.
        </p>
    </div>
    <div class="col-sm-4 mt-3 mt-sm-0">
        <a href="https://github.com/ivanbrillo/BioConnect" target="_blank" class="btn btn-sm z-depth-1 w-100" style="background-color: #24292e; color: white;">
            <i class="fab fa-github"></i> &nbsp; View on GitHub
        </a>
        <a href="https://ivanbrillo.github.io/BioConnect/" target="_blank" class="btn btn-sm z-depth-1 w-100 mt-2" style="background-color: #1a3a5c; color: white;">
            📄 &nbsp; SwaggerUI API Docs
        </a>
        <p class="mt-2" style="font-size: 0.8rem; color: #888;">
            Team: Andrea Bochicchio, Daniel Pipitone, Ivan Brillo
        </p>
    </div>
</div>

---

## Platform Overview

BioConnect is designed around three axes of functionality. **Data exploration** lets any visitor search proteins by functional pathway and subcellular location, search drugs by name or category, and browse associated publications and patents. **Network analysis** leverages the Neo4j graph to answer structural queries: finding drugs that target proteins similar to a given protein, tracing the shortest protein-interaction path between two diseases, identifying diseases linked to a specific drug, and discovering drugs with opposite effects on the same protein. **Trend analysis** surfaces temporal patterns in scientific publications and expired patents across drug categories and protein pathways.

Registered users can annotate any protein or drug with comments and manage their own contributions. Administrators can add, update, or delete proteins and drugs across both databases atomically, manage user accounts, and moderate comments.

<div class="row justify-content-center mt-3">
    <div class="col-sm-10">
        {% include figure.liquid loading="eager" path="assets/img/bioconnect_mockup.png" title="BioConnect UI mockup — protein detail page showing interaction graph, subcellular location, publications, and comment section" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

---

## Data Model and Databases

The dataset was assembled from three primary sources: **UniProt** for human proteins, **DrugBank** (academic license) for drug data including patents and toxicity, and **DisGeNET** for protein–disease associations. Publication dates were enriched via the National Library of Medicine (NLM) API. Python scripts cleaned and harmonized all sources into JSON for MongoDB ingestion and CSV for Neo4j population. The final database footprint is 42 MB in MongoDB and 15 MB in Neo4j CSV files.

**MongoDB** holds three collections — `Protein` (~34 MB), `Drug` (~7 MB), and `User` (~1 MB) — using a flexible document schema. Comments are embedded within user documents rather than stored in a separate collection, since they grow slowly and are almost always accessed alongside user data. **Neo4j** models five relationship types: `INTERACTS_WITH` and `SIMILAR_TO` between proteins (bidirectional), `INVOLVED_IN` from protein to disease, and `ENHANCED_BY` / `INHIBITED_BY` from protein to drug (unidirectional).

---

## Key Queries

**MongoDB aggregation pipelines** follow a uniform `$match → $project → $unwind → $group → $sort` structure. The publication trend query filters proteins by pathway, unwinds the publications array, groups by year and type, and returns a chronological count series. The pathway recurrence query uses a case-insensitive regex on the sequence field and groups by pathway name. The expired patents query filters by drug category, retains only patents older than 20 years, and groups by country. All four query families are accelerated by dedicated indexes on `name` (text index, Drug and Protein), `categories` (Drug), and `pathways` (Protein), which reduce execution time from tens of milliseconds to near-zero.

**Neo4j graph queries** cover four patterns. *Drugs targeting similar proteins* traverses `(Drug)–[INHIBITED_BY|ENHANCED_BY]→(Protein)–[SIMILAR_TO]→(Protein p2)` and returns drugs connected to proteins similar to the input. *Shortest path between diseases* uses `allShortestPaths` with a maximum depth of 5, constraining intermediate nodes to be of type `Protein`. *Diseases linked to a drug* follows `(Disease)←[INVOLVED_IN]–(Protein)–[](Drug)` to surface all diseases associated with a drug's target proteins. *Drugs with opposite effects* combines two `UNION` subqueries to find drugs that enhance a protein inhibited by the query drug, and vice versa.

<div class="row justify-content-center mt-3">
    <div class="col-sm-10">
        {% include figure.liquid loading="eager" path="assets/img/bioconnect_neo4j_diseases.png" title="Neo4j result graph: diseases linked to a drug, traversing protein interaction nodes" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

---

## Implementation

The backend adopts a standard Spring Boot layered architecture: **Controller** → **Service** → **Repository** → **Model**, with DTOs decoupling internal entities from API responses. Security is handled by a JWT filter chain (`JwtTokenFilter` + `JwtTokenProvider`) with BCrypt password hashing, two roles (`REGISTERED`, `ADMIN`), and route-level authorization via Spring Security. The API is fully documented through an embedded Swagger/OpenAPI endpoint and a [Postman collection](https://documenter.getpostman.com/view/21790252/2sAYQfCTyZ).

Inter-database consistency for CRUD operations on proteins and drugs is maintained through two independent `@Transactional` managers — one for Neo4j, one for MongoDB — configured as Spring Beans. The save flow first commits to Neo4j, then atomically writes to MongoDB; on failure, the Neo4j transaction rolls back. Updates remove then re-save the graph node before updating the document. Deletes cascade from Neo4j through MongoDB and also purge all related user comments.

---

## Database Design Choices

The system is positioned in the **AP** region of the CAP theorem, prioritizing availability and partition tolerance over strong consistency — a deliberate trade-off given that write operations are rare and data staleness risk is minimal. MongoDB replication uses a three-node replica set (`w=1, journal=true, readPreference=nearest`), balancing durability with low read latency. Indexes on `Drug-id`, `Protein-id`, and `Disease-id` are implemented as Neo4j uniqueness constraints, reducing node lookup from 50k+ DB hits to 4 DB hits and from 54 ms to 1 ms. Sharding was designed but judged unnecessary given the finite scope of human biological data and the predominantly read-heavy workload.

---

## Performance Evaluation

Read performance was benchmarked with Postman under a peak load of 100 virtual users over 5 minutes. With `readPreference=nearest` and both databases active, the system achieved 204 requests/second at an average response time of 211 ms with 0% error rate. Switching to `readPreference=primary` degraded MongoDB response times while improving Neo4j, confirming that distributing reads across the replica set is beneficial. When Neo4j was taken offline entirely, all graph queries failed while MongoDB performance improved, validating the system's partial-availability behavior.

Write performance was tested by sequentially inserting 1,000 proteins under six MongoDB write-concern configurations. The `w=1, journal=true` setting — our production choice — completed in ~10.5 s, offering a practical balance between durability and throughput compared to stricter majority-write or w=3 configurations, which ranged up to ~18 s.