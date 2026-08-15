STORE / LOGGING / AUDIT / STATE — STANDARDS LANDSCAPE
======================================================

Owner: iso-agent
Scope: map the persistence layer (DECISIONS D-016 / D-017) — a Store interface
with three first-class interfaces (logging, audit, state) — to existing
standards, and identify where we are first movers. No code, no dependencies,
no processes run. Web-verified Aug 2026; cite only current editions.

THE OBJECT (from DECISIONS D-016 / D-017)
-----------------------------------------

D-016: Persistence is storage-agnostic; all state and logs flow through a Store
interface (Append/Read/List), file-backed by default, provider-swappable (git,
remote, database). The interface is the invariant, the backend is the variable.

D-017: The persistence layer ships multiple first-class interfaces —
  1. logging — append
  2. audit   — append-only, hash-chained, tamper-evident
  3. state   — read/write
  — each over a Store interface, multi-backend from the first implementation
  (filesystem, git, remote, database).

So the standards question is not "what governs a log file" but two distinct
questions, kept separate throughout:
  (A) which standards govern each interface's NAME + STRUCTURE (fields,
      semantics, retention) so our surface is externally citable; and
  (B) which standards, if any, govern the STORE ABSTRACTION itself (a
      provider-agnostic persistence interface) — this is where first-mover
      status is tested.


THE LANDSCAPE (catalog of the relevant standards)
-------------------------------------------------

Five clusters cover the three interfaces. Verified editions are cited; where an
edition is omitted the family is the point, not the year.

CLUSTER 1 — SECURITY LOGGING & MONITORING (the `logging` interface)
  ISO/IEC 27002:2022  — control 8.15 "Logging" (capture events, protect against
      tampering and unauthorized access), 8.16 "Monitoring activities", 8.17
      "Clock synchronisation". Guidance under ISO/IEC 27001:2022 (already in
      iso-map.md). Governs WHAT a log must record and HOW it must be protected;
      does not govern the interface.
  NIST SP 800-92 "Guide to Computer Security Log Management" (2006) — the
      canonical US log-management taxonomy: log infrastructure, priorities,
      event categories, retention/analysis. Maps to NIST SP 800-53 AU
      (Audit and Accountability) family.
  RFC 5424 (Syslog) — the de-facto log transport/shape: severity (0-7:
      emerg/alert/crit/err/warning/notice/info/debug), facility, timestamp,
      structured data. The field's shared vocabulary for severity levels.
  OCSF (Open Cybersecurity Schema Framework) — vendor-neutral event taxonomy
      (AWS/Cloudflare/Splunk-backed). De-facto, not ISO; the closest thing to a
      canonical event-field schema for logs.

CLUSTER 2 — RECORDS MANAGEMENT (the `audit` interface, semantic half)
  ISO 15489-1:2016 — Records management, concepts and principles. The four
      defining record properties: authenticity, reliability, integrity,
      usability. This is the standard that turns "a log" into "a record."
  ISO 30301 (management systems for records — requirements; ISO 30300/30301/
      30302 family, 2011/2015) — org-level records governance (Annex SL), the
      management-system wrapper around 15489.
  ISO 23081-1:2006 / -2:2009 — metadata for records (principles + concepts):
      the record's context/identity metadata must be captured at creation and
      carried for the record's life. Backs our audit-entry metadata fields.
  ISO 16175-2:2011 / -3:2010 — functional requirements for records in
      electronic/business systems. The closest existing statement of "what a
      records-capable system must expose": capture, classification, retention,
      disposition, access, audit trail. Directly maps to our audit interface's
      compliance surface.
  ISO 13008:2012 — digital records conversion/migration (retention across
      format change — relevant to provider-swapping the audit backend).

CLUSTER 3 — TAMPER-EVIDENCE / INTEGRITY (the `audit` interface, cryptographic
           half)
  ISO/IEC 18014-3:2009 — Time-stamping services, Part 3 "mechanisms producing
      linked tokens". This is the only ISO standard that specifies a
      hash-chain/linked-token structure — the same primitive as our
      hash-chained audit log. Parts 1 (framework), 2 (independent tokens),
      4 (traceability of time sources) complete the set. It governs the
      TIME-STAMPING service, not a general audit-log interface.
  RFC 9162 (Dec 2021) "Certificate Transparency v2.0" (obsoletes RFC 6962) —
      the reference for an append-only Merkle log: Signed Tree Head, inclusion
      proofs, consistency proofs, append-only property. Domain-specific
      (TLS certs) but the cleanest public spec of the hash-chain/Merkle audit
      primitive we are reusing.
  (Adjacent de-facto, not ISO): Sigstore/Rekor (transparency log), Trillian
      (Google's Merkle log), git's SHA-1/SHA-256 object graph + signed tags.
      These prove the pattern is proven at scale; none is an ISO interface.

CLUSTER 4 — DIGITAL PRESERVATION (retention/preservation of audit + state)
  ISO 14721:2025 (OAIS, = CCSDS 650.0-M-3 issue 3, Dec 2024) — Reference Model
      for an Open Archival Information System. Defines SIP/AIP/DIP information
      packages and "fixity" (checksums/digests proving content unchanged). Our
      hash chain IS a fixity mechanism; OAIS gives it a name and a home.
  ISO 16363:2012 — Audit and certification of trustworthy digital repositories
      (the OAIS conformance/audit standard; criteria for a repository that can
      prove authenticity and integrity over time).

CLUSTER 5 — METADATA / NAMING (the `state` interface + cross-cut)
  ISO/IEC 11179 (multi-part) — metadata registry semantics + naming/
      identification (Parts 3/4/5, already in iso-map.md). The authority for
      naming and identifying our state keys, record types, and field names.
  ISO 8601 / RFC 3339 — UTC timestamp representation (the single timestamp
      form all three interfaces share).


PART 1 — WHICH STANDARDS TO ALIGN INTERFACE NAMING / STRUCTURE / HOOKS TO
-------------------------------------------------------------------------

The rule is the same one iso-map.md applies: align to what EXISTS as an
externally citable vocabulary, and attach our compliance hooks to a named
standard rather than to taste. Per interface:

1. STORE (Append/Read/List) — the cross-cut
   Align STRUCTURE to: ISO/IEC/IEEE 42010:2022 (write the Store + three
   interfaces as one AD — the three interfaces are three VIEWPOINTS over one
   system-of-interest, with a correspondence rule stating each interface is a
   projection of the Store). This is the same discipline already mandated for
   every seam.
   Align NAMING to: ISO/IEC 11179-5 (the three verbs Append/Read/List and the
   backend names filesystem/git/remote/database are the data-element names of
   the Store's metamodel; 11179-5 gives the identification rule).
   Compliance hook: the Store spec's correspondence rules = "logging, audit,
   and state are three interfaces over one Store; a backend must implement all
   three or be ineligible." This is D-017 restated as a 42010 correspondence
   rule.

2. LOGGING (append)
   Align NAMING to: RFC 5424 severity vocabulary (debug/info/warning/error/
   crit...) as the fixed severity enum; ISO 8601/RFC 3339 for the timestamp.
   Align STRUCTURE to: ISO/IEC 27002:2022 8.15/8.16/8.17 and NIST SP 800-92 —
   the log record carries, at minimum: timestamp (clock-synchronised source),
   event/category, source, and message — and the write path is protected from
   tampering (8.15). OCSF is the optional richer event-taxonomy alignment.
   Compliance hook: expose a clock source (8.17) and a severity field whose
   values are the RFC 5424 set — so any SIEM/27002 auditor can consume the log
   without translation.

3. AUDIT (append-only, hash-chained, tamper-evident)
   This interface has TWO halves and must cite BOTH.
   (a) Semantic half — records management. Align to ISO 15489-1:2016 (the
       record's four properties: authenticity, reliability, integrity,
       usability), ISO 23081-1/-2 (metadata captured at creation), and
       ISO 16175-2/-3 (functional requirements: capture, classification,
       retention, disposition, audit trail). Concrete hooks: every audit entry
       carries creation metadata (23081); the record carries a retention/
       disposition field (15489/16175); append-only + non-deletion is the
       integrity property made mechanical.
   (b) Cryptographic half — tamper-evidence. Align STRUCTURE to ISO/IEC
       18014-3 (linked tokens) and RFC 9162 (Merkle append-only log: signed
       tree head + inclusion/consistency proofs). Concrete hooks: every entry
       carries prev-hash (the link); the interface exposes a verify() and a
       head/root commitment (the STH analogue), so a third party can prove
       append-only-ness and detect retroactive edit.
   Alignment to gate spec.txt: the gate's AUDIT ENTRY (subject, action, object,
       arguments_hash, verdict, grant_id, timestamp) is the audit interface's
       FIRST record type — it must become a 15489 record (authenticity/
       integrity) with a hash-chain link, not a plain line.

4. STATE (read/write)
   Align NAMING to: ISO/IEC 11179-3/-5 (key naming, value-domain naming).
   Align STRUCTURE to: ISO/IEC/IEEE 42010:2022 (the state schema is the model
   kind; read/write are the interface). Preservation of state that must outlive
   a backend swap aligns to ISO 14721:2025 (OAIS) fixity + ISO 13008:2012
   (migration across format change).
   Compliance hook: state keys are 11179-5-compliant identifiers; state
   content carries a digest (fixity, OAIS) so a backend swap can be verified.

NET ALIGNMENT (same priority tiering as iso-map.md):
  Artifact tier (adopt now, cheap): 42010 (AD structure), 11179 (naming),
  RFC 5424 (severity), ISO 8601 (timestamps).
  Security/records tier (adopt as the spec is written): 27002 8.15-8.17,
  NIST SP 800-92, ISO 15489-1, ISO 23081, ISO 16175, ISO/IEC 18014-3,
  RFC 9162.
  Preservation/management tier (deferred/optional): ISO 14721 (OAIS),
  ISO 16363, ISO 30301, ISO 13008.


PART 2 — WHERE WE ARE FIRST MOVERS
----------------------------------

Classification rule (unchanged from first-mover.md): a standard owns the
territory only if it specifies the INTERFACE we would ship. By that rule, three
things are first-mover at the interface level:

1. THE STORE ABSTRACTION (Append/Read/List, provider-swappable) — FIRST MOVER.
   No ISO/IEC standard defines a storage-provider-agnostic persistence
   interface. What exists either governs a FORMAT (SQL/ISO 9075, JSON/21778),
   a WIRE, or a MANAGEMENT SYSTEM (27001/30301) — never the abstraction "all
   state and logs flow through one Append/Read/List surface regardless of
   backend." D-016's core claim (the interface is the invariant, the backend
   the variable) is unclaimed territory. This is the highest-leverage
   first-mover position in the persistence layer.

2. THE AUDIT INTERFACE AS A GENERAL HASH-CHAINED TAMPER-EVIDENT LOG —
   FIRST MOVER at the interface level.
   The cryptographic PRIMITIVE is not new: RFC 9162 (CT) and ISO/IEC 18014-3
   (linked time-stamp tokens) both specify hash chains/Merkle structures — but
   each is bound to one domain (TLS certs; time-stamping), and neither defines
   a general append-only, hash-chained, tamper-evident AUDIT-LOG INTERFACE
   with verify()/head() as first-class operations. We borrow the primitive,
   we own the interface.

3. THE THREE-INTERFACE SPLIT OVER ONE STORE (logging vs audit vs state as
   distinct first-class interfaces) — FIRST MOVER.
   Existing standards treat operational logging (27002/800-92), records
   management (15489), and storage (nothing) as SEPARATE worlds; no standard
   declares them three named interfaces over a single provider-agnostic Store.
   D-017's "collapsing them into one log file is a failure mode" is a
   first-mover framing — the distinction is an interface-level claim nobody
   else has standardized.

FIRST-MOVER PRIORITY (greenfield x leverage x feasibility), within this layer:
  1. Store abstraction (Append/Read/List) — the foundational NP candidate.
  2. Audit interface (hash-chained, tamper-evident, with verify/head) — the
     highest-value NP; the primitive is proven (RFC 9162/18014-3), the
     interface is not.
  3. State interface — lower; mostly 11179 naming + OAIS fixity, little novel.

Committee home note (consistent with sponsor-path.md): these are software-
engineering / information-security interface specs, NOT AI specs — the honest
home is JTC 1/SC 27 (information security) for the audit/tamper-evidence
primitive and JTC 1/SC 7 (software & systems engineering, owns 42010) for the
Store abstraction, not SC 42/WG 5. This is the same WG-placement fork already
flagged for seam 6; do not default to SC 42.


SUMMARY (one line per finding)
------------------------------

  logging  — align: RFC 5424 (severity) + ISO 8601 (time) + 27002 8.15-8.17 /
             NIST 800-92 (what/why to log); NOT first mover.
  audit    — align: 15489-1 + 23081 + 16175 (record semantics) AND 18014-3 +
             RFC 9162 (hash-chain primitive); FIRST MOVER as a general
             tamper-evident audit-log INTERFACE.
  state    — align: 11179-3/-5 (naming) + OAIS 14721 fixity; mostly align,
             not first mover (the interface itself is generic).
  Store    — align: 42010 (AD) + 11179 (naming); FIRST MOVER (provider-agnostic
             persistence abstraction is unclaimed).

SOURCES (fetched Aug 2026)
  en.wikipedia.org/wiki/Open_Archival_Information_System (OAIS = ISO 14721:2025,
    CCSDS 650.0-M-3 issue 3 Dec 2024; ISO 16363:2012)
  en.wikipedia.org/wiki/ISO_15489 (15489-1:2016; 30300/30301/30302; 23081;
    16175; 13008:2012; 22310:2006)
  en.wikipedia.org/wiki/ISO/IEC_18014 (18014-1:2008, -2:2009, -3:2009 linked
    tokens, -4:2015)
  csrc.nist.gov/pubs/sp/800/92/final (NIST SP 800-92, Sep 2006)
  rfc-editor.org/rfc/rfc9162 (Certificate Transparency v2.0, Dec 2021,
    obsoletes RFC 6962)
  ISO/IEC 27002:2022 control numbers (8.15 Logging, 8.16 Monitoring, 8.17 Clock
    synchronisation) from iso-map.md context; edition check pending against the
    ISO catalogue (403-blocked to scraping — same caveat as sponsor-path.md).
