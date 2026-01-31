---
id: tofri9unupa6piou46kq9bb
title: 2026 01 30 Trailing Slash and Iris
desc: ''
updated: 1769838956145
created: 1769838915050
---
Skip to content
Chat history
You said:
OK, I think I need to abandon the trailing slash as marker of "non-file-denoting-IRIs". \/ is confusing, and people are already used to not terminating the terms with slashes.  It's interesting that VIAF using the opposite split: no-slash ending means the concept, and it's description uses the slash. That's actually consistent with github pages behaviour. If you request the no-slash (and the request doesn't have a file to get mapped to) you redirect to the slash-ending description. Adopting that as a solution is actually okayish. 
ChatGPT said:

Dropping the “trailing slash inside the term” idea is the right move. You were heading toward an ecosystem tax: Turtle/SPARQL prefixed names generally don’t tolerate / in the local part in a portable way, and \/ is a readability hazard even when a parser accepts it.

That said, you’re mixing two different claims, and they pull in opposite directions:

    “Abandon trailing slash as a marker of non-file IRIs.”

    “Adopt the VIAF split (no-slash = thing, slash = description), which matches GitHub Pages behavior.”

Those are not the same idea. #2 still uses the slash as a semantic marker—just inverted, and coupled to hosting quirks.
What VIAF actually did (and what it does now)

The classic VIAF pattern (as documented by World Wide Web Consortium) was:

    Real-world object: .../viaf/96994048 → 303 See Other

    Generic document: .../viaf/96994048/ → 200 OK (content-negotiated)

So your recollection is grounded historically. But today, when you hit example VIAF IDs, both slash and no-slash appear to converge via redirects into a language-scoped path (e.g. /en/viaf/...). In other words: VIAF is not a stable precedent for long-lived slash semantics.
Why “GitHub Pages adds the slash” is a shaky foundation

The “no-slash → slash” redirect on many hosts is a filesystem/directory convention (requesting a directory without / gets redirected to the canonical directory URL with /). That behavior is widely described as server “directory requires trailing slash” logic, not an intentional “this URI denotes a non-document thing” semantic-web move.

So if you lean on that redirect as meaningful, you’re letting an incidental hosting rule define your ontology’s identity rules.
The real risk if you adopt “no-slash thing / slash doc”

/Mesh and /Mesh/ are different IRIs. If both leak into RDF in the wild, you’ve minted two resources and will be stuck either:

    asserting equivalence everywhere (owl:sameAs-ish), or

    living with accidental duplication.

If you do this, you need a hard rule: only one form is ever used in RDF identifiers, and the other is purely a documentation convenience.
A cleaner pattern for Semantic Flow

If your goal is “human-friendly pages on GitHub Pages” + “clean IRIs for RDF/SPARQL”, the least painful options are:
Option A: Explicit /id and /doc (recommended if you want semantic clarity)

    Thing/term IRI: https://…/id/Mesh

    HTML doc: https://…/doc/Mesh/

    RDF distributions: https://…/data/Mesh.ttl (and friends)

Then you can follow “Cool URIs” guidance: /id/... redirects (303) to /doc/... or /data/... depending on negotiation/format.

This avoids the slash/no-slash identity trap entirely.
Option B: Same IRI for term + doc, via content negotiation (recommended if you want simplicity)

Use https://…/Mesh as the single canonical IRI and serve HTML/RDF representations. This punts on httpRange-14 purism, but it’s operationally simple and consistent with “URI hosting practice” style guidance from the TAG.
If you still want the “VIAF-style” split anyway

Do it deliberately, not as a side-effect of GitHub:

    Canonical RDF identifier: …/Mesh (no slash)

    Canonical HTML doc URL: …/Mesh/ (slash, directory + index.html)

    Ensure every generated RDF file uses only the no-slash IRI in subjects, ontology term IRIs, @base, etc.

    Add a linter check: reject any IRI ending with / where the no-slash form exists (or vice versa).

    Make redirects one-way and permanent, and put a canonical link in HTML so search engines/tools converge.

This is “okay-ish” only if you treat the slash form as presentation plumbing, not as a second identifier that’s allowed to participate in the graph.

If you tell me which of the three goals you care about most—(1) semantic-web orthodoxy, (2) GitHub Pages zero-config hosting, (3) CURIE/SPARQL ergonomics—I can give you a single crisp convention set (base IRIs, folder layout, and redirect rules) that doesn’t fight your toolchain.
You said:
No content negotiation. Github doesn't support it. Remember this: No content negotiation, ever. No redirect either. 
ChatGPT said:

With no content negotiation and no redirects, you lose the two standard mechanisms that let a “thing IRI” be dereferenceable while returning different representations (HTML vs RDF) or while keeping “non-information resource” orthodoxy. That’s not a moral judgment; it’s a mechanical constraint.

So you have exactly three viable publishing patterns. Pick one and then enforce it everywhere (generator, docs, examples), otherwise you’ll mint accidental duplicates.
Pattern 1 — Term IRIs are HTML document URLs

This is the only pattern that preserves “dereference the term IRI and you get a useful page” under your constraints.

    Canonical term IRI used in RDF:
    …/Mesh.html

    RDF distributions:
    …/Mesh.ttl, …/Mesh.jsonld, …/Mesh.trig (whatever you ship)

    Linkage triples (in RDF and/or HTML embeds):

        …/Mesh.html rdfs:seeAlso …/Mesh.ttl

        or …/Mesh.ttl foaf:primaryTopic …/Mesh.html (choose one direction and stick to it)

Pros: works on GitHub Pages; no redirects/negotiation needed; always dereferenceable.
Cons: you are explicitly conceding httpRange-14 purism: the IRI is clearly a document URL.

If you can live with that concession (and your recent “okayish” tone suggests you can), this is the cleanest operationally.
Pattern 2 — Term IRIs are not dereferenceable; doc/data are separate

If you insist that the term IRI should not look like a file, and you won’t use redirects/negotiation, then the term IRI cannot also be the retrieval URL.

    Canonical term IRI used in RDF:
    …/Mesh (or …/Mesh/ if you really wanted—though you said you don’t)

    Doc page:
    …/Mesh.html

    RDF distributions:
    …/Mesh.ttl etc.

    Linkage:
    …/Mesh foaf:isPrimaryTopicOf …/Mesh.html (or schema:mainEntityOfPage, etc.)

Pros: term IRIs stay extensionless and “non-file-looking.”
Cons: dereferencing the term IRI won’t work (or will 404), which undercuts a core Semantic Web affordance and some of Semantic Flow’s “resource page” story.

If dereferenceability is a first-class Semantic Flow requirement, this pattern is self-sabotage.
Pattern 3 — Hash URIs for vocabulary terms

This is the classic “no redirects needed” solution from W3C guidance when you can’t (or won’t) do 303/content-negotiation.

    Vocabulary document (HTML):
    …/ontology.html

    Terms:
    …/ontology.html#Mesh

    RDF vocabulary file (separate, no negotiation):
    …/ontology.ttl

    In ontology.ttl: use the same term IRIs (…/ontology.html#Mesh) so they dereference to the HTML doc in browsers.

Pros: no redirects; no content negotiation; terms are dereferenceable (they resolve to the HTML doc).
Cons: you give up slash-term aesthetics for vocab terms (you can still use slash IRIs for everything else in your mesh).

Given your constraints + your desire to stop fighting Turtle/SPARQL local-name syntax, this is the least internally contradictory option for ontology terms.
The inconsistency you need to resolve

You can’t have all three at once:

    No redirects, no negotiation

    Term IRIs are extensionless slash IRIs (e.g., …/Mesh)

    Term IRIs dereference to a human page on GitHub Pages

Pick any two.

If Semantic Flow’s “dereferenceable resource pages” are non-negotiable, then Pattern 1 or 3 is where you land. If you also want to keep slash aesthetics for mesh resources, I’d do:

    Ontology terms: Pattern 3 (hash in ontology.html)

    NamedIndividual pages / mesh resources: Pattern 1 (explicit .html pages as canonical dereferenceable IRIs)

    Payload artifacts: explicit extensions (.ttl, .trig, .jsonld) as you already do

If you don’t want the .html in the canonical IRI for non-vocab resources, you’re back in Pattern 2 and you should explicitly admit “term IRIs are not dereferenceable,” because otherwise the system will keep trying to sneak redirects back in via directory behavior (which GitHub Pages does for folders).
You said:
I can have all three at once. You just have to use Folders for the Term IRIs, with index.html files containing the HTML for the human page. I don't expect the Term IRIs to return (only) RDF. But their RDF could be embedded in the HTML.
ChatGPT said:
You said:
By "No redirects" I mean no redirect other than the standard "redirect non-slash ending IRIs to slash-ending ones" that happens automatically with github and most web servers.
ChatGPT said:

“Folder term IRIs with index.html” works mechanically on GitHub Pages, but it only “has all three” if you’re precise about what your canonical identifier is and you actively prevent IRI-splitting.

Right now you’re implicitly relying on exactly one redirect: …/Mesh → …/Mesh/ (directory canonicalization). That’s fine if you explicitly treat it as a transport detail, not an identity signal.
The real constraint you can’t dodge

With this setup, two distinct IRIs will exist in the wild:

    https://…/Mesh

    https://…/Mesh/

Even if you never mint one of them in RDF, browsers will show the slash form, and humans will copy/paste it into data. So your system must choose one form as canonical in RDF, and have a mitigation strategy for the other.

If you don’t, you’ll reintroduce the same “two identifiers for one thing” problem you were trying to avoid with the trailing-slash marker.
A convention set that is internally consistent
1) Canonical term IRI (in RDF) MUST be the no-slash form

Use the no-slash IRI as the identifier you publish in Turtle/JSON-LD, because:

    it avoids the / in the local part problem (so sflo:Mesh is valid),

    it matches your desire to avoid trailing slashes in term names.

Canonical in RDF: …/Mesh
Retrieval URL in browsers: …/Mesh/ (due to directory canonicalization)
2) The folder is an implementation detail

On disk:

    Mesh/index.html is the human page

    optionally Mesh/index.jsonld, Mesh/index.ttl, etc., but those will only be fetchable via explicit URLs (no negotiation).

Your term IRI is still …/Mesh in RDF; the HTML is simply a representation you happen to retrieve via …/Mesh/.
3) Embed RDF in HTML, but make the @id canonical

If you embed JSON-LD in index.html, the top-level identifier must be the canonical no-slash IRI:

{
  "@context": "...",
  "@id": "https://…/Mesh",
  "@type": "owl:Class",
  "rdfs:label": "Mesh"
}

This ensures that even if someone scrapes RDF from the HTML at …/Mesh/, they still extract triples about …/Mesh.
4) Add an explicit equivalence triple (optional but pragmatic)

Because the slash IRI will leak into the world, you can preemptively collapse it:

    https://…/Mesh/ owl:sameAs https://…/Mesh

But be honest: this is a band-aid that admits the split exists. It’s still better than silently letting both propagate without reconciliation.
5) Lint rule: forbid trailing-slash term IRIs in authored RDF

Your generator can enforce:

    reject any IRI in your namespace ending with / unless it is explicitly a container/resource-page URL class you’ve defined.

This is the only way you keep the slash form from becoming “normal” in your RDF corpus.
The semantic point you’re conceding (and that’s okay if you mean it)

You said: “I don’t expect the Term IRIs to return (only) RDF.” That’s effectively a decision that the “term IRI” is fine being dereferenced to an HTML document and that strict httpRange-14 separation isn’t a priority.

That’s coherent—just don’t simultaneously claim “the IRI denotes a non-document thing” and treat it operationally as the document URL. Under no-negotiation/no-redirect (except canonicalization), you don’t have the machinery to maintain that distinction cleanly.
The one place your proposal can still bite you

Even if you choose …/Mesh as canonical, most humans will share …/Mesh/. Without either:

    an equivalence triple, or

    a social/UX mechanism that makes the canonical IRI easy to copy (display it prominently and make it a one-click copy),

you’ll slowly accumulate mixed identifiers across repositories.

If you want, I can give you a concrete “resource page template” snippet (HTML + embedded JSON-LD + a visible canonical-Iri block) that makes the no-slash IRI the thing people actually copy.
You said:
equivalence triple is useless except under reasoning. It doesn't help SPARQL, and it requires special application behaviour. But yes, maybe I'll put it everywhere just to help this ugly situation. I also don't expect. Social/UX mechanism is fine too.
ChatGPT said:

Correct: an equivalence triple (e.g., owl:sameAs) does nothing for plain SPARQL unless you run entailment or do query rewriting. It also forces every consumer to decide how seriously to take the equivalence (and owl:sameAs is very strong).

If your real problem is “people will copy /Mesh/ from the browser and paste it into RDF,” then the best fixes are prevention and UX, not ontology-level equivalence.
Do this first (prevention)

    Hard canonical rule: in authored RDF, term IRIs are always no-slash (…/Mesh).
    Enforce in your build/lint step: fail the build if any subject/predicate/object IRI in-namespace ends with / (with narrow exceptions like actual containers).

    Embed RDF in the HTML, but make @id the no-slash IRI.
    Even though the page is at /Mesh/, the data describes /Mesh.

    Make the HTML page “teach” the canonical IRI.
    Put a prominent “Canonical IRI: …/Mesh” block and a one-click copy button.

The single most effective UX hack

On page load, if the path ends with /, call history.replaceState to remove the trailing slash in the address bar (except for / itself). This does not trigger a network redirect; it just changes what users copy.

    Request still happens to /Mesh/ (GitHub canonicalization stays as-is).

    Address bar becomes /Mesh, so copy/paste produces the canonical IRI you want.

    If a user reloads, GitHub will canonicalize again—which you already allow.

This is the only approach that directly attacks the “leaky slash” problem without relying on RDF reasoning or SPARQL entailment.
If you still want a “reconciliation triple,” don’t use owl:sameAs

If you keep any linkage, prefer a document→topic relation, because it’s true even if you care about the document/thing distinction:

    /Mesh/ is a document page

    /Mesh is the thing/term

Use one of:

    foaf:primaryTopic (classic)

    schema:about / schema:mainEntity (more common on the web)

This doesn’t magically unify identifiers in SPARQL, but it avoids baking a dubious identity claim into your core graph.
If you insist on equivalence anyway

Then isolate it:

    Put equivalence assertions in a dedicated “canonicalization” dataset/series (your _reference-flow is a natural place), not in every core term definition. That keeps the “ugly situation” contained and lets strict consumers ignore it.

Bottom line: treat /Mesh/ as an unavoidable retrieval URL, and make it difficult for humans to accidentally mint it as an RDF identifier. The history-rewrite + copy button combo is what actually changes outcomes.


ChatGPT can make mistakes. Check important info.
