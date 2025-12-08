%%%
title = "OpenID Connect Federation 1.1"
abbrev = "openid-connect-federation-1.1"
ipr = "none"
workgroup = "OpenID Connect Working Group"
keyword = ["OpenID", "OpenID Connect", "Federation", "Multilateral Federation", "Federation Entity", "Federation Operator", "Trust Anchor", "Trust Chain", "Trust Establishment", "Trust Mark"]

[seriesInfo]
name = "Internet-Draft"
value = "openid-connect-federation-1.1"
stream = "IETF"

[[author]]
initials="R."
surname="Hedberg"
fullname="Roland Hedberg"
organization="independent"
    [author.address]
    email = "roland@catalogix.se"

[[author]]
initials="M.B."
surname="Jones"
fullname="Michael B. Jones"
organization="Self-Issued Consulting"
    [author.address]
    email = "michael_b_jones@hotmail.com"

[[author]]
initials="A.Å."
surname="Solberg"
fullname="Andreas Åkre Solberg"
organization="Sikt"
    [author.address]
    email = "Andreas.Solberg@sikt.no"

[[author]]
initials="J."
surname="Bradley"
fullname="John Bradley"
organization="Yubico"
    [author.address]
    email = "ve7jtb@ve7jtb.com"

[[author]]
initials="G."
surname="De Marco"
fullname="Giuseppe De Marco"
organization="independent"
    [author.address]
    email = "demarcog83@gmail.com"

[[author]]
initials="V."
surname="Dzhuvinov"
fullname="Vladimir Dzhuvinov"
organization="Connect2id"
    [author.address]
    email = "vladimir@connect2id.com"

%%%

{mainmatter}

# Introduction

*[Content conversion from XML to markdown would go here]*




A federation can be expressed as an agreement between parties that trust each other. In bilateral federations, direct trust can be established between two organizations belonging to the same federation. In a multilateral federation, bilateral agreements might not be practical, in which case, trust can be mediated by a third party. That is the model used in this specification.

An Entity in the federation must be able to trust that other Entities it interacts with belong to the same federation. It must also be able to trust that the information the other Entities publish about themselves has not been tampered with during transport and that it adheres to the federation's policies.

This specification defines basic components to build multilateral federations. It also defines how to apply them in the contexts of OpenID Connect and OAuth 2.0. These components can be used by other application protocols for the purpose of establishing trust.











## Introduction

This specification describes how two Entities that would like to interact can establish trust between them by means of a trusted third party called a Trust Anchor. A Trust Anchor is an Entity whose main purpose is to issue statements about Entities. An identity federation can be realized using this specification using one or more levels of authorities. Examples of authorities are federation operators, organizations, departments within organizations, and individual sites. This specification provides the basic technical trust infrastructure building blocks needed to create a dynamic and distributed trust network, such as a federation.

Note that this specification only concerns itself with how Entities in a federation get to know about each other. An organization MAY be represented by more than one Entity in a federation. An Entity MAY also belong to more than one federation. Determining that two Entities belong to the same federation is the basis for establishing trust between them in this specification.

Of course, the word "trust" is also used in the vernacular to encompass confidence in the security, reliability, and integrity of entities and their actions. This kind trust is often established through empirical proof, such as past performance, security certifications, or transparent operational practices, which demonstrate a track record of adherence to security standards and ethical conduct. To be clear, this broader meaning of trust, while important, is largely beyond the scope of what this specification accomplishes.

Below is an example of two federations rooted at two different Trust Anchors with some members in common. Every Entity is able to establish mutual trust with any other Entity by means of having at least one common Trust Anchor between them.

[]{#name-two-coexisting-federations-}

<figure id="figure-1">
<div id="section-1-5.1" class="alignLeft art-text artwork">
<pre><code>.-----------------.            .-----------------.
|  Trust Anchor A |            |  Trust Anchor B |
&#39;------.--.-------&#39;            &#39;----.--.--.------&#39;
       |  |                         |  |  |
    .--&#39;  &#39;---. .-------------------&#39;  |  |
    |         | |                      |  |
.---v.  .-----v-v------.   .-----------&#39;  |
| OP |  | Intermediate |   |              |
&#39;----&#39;  &#39;--.--.--.-----&#39;   |    .---------v----.
           |  |  |         |    | Intermediate |
   .-------&#39;  |  &#39;------.  |    &#39;---.--.--.----&#39;
   |          |         |  |        |  |  |
.--v-.      .-v--.     .v--v.   .---&#39;  |  &#39;----.
| RP |      | RS |     | OP |   |      |       |
&#39;----&#39;      &#39;----&#39;     &#39;----&#39;   |   .--v-.   .-v--.
                                |   | RP |   | RP |
                                |   &#39;----&#39;   &#39;----&#39;
                                |
                        .-------v------.
                        | Intermediate |
                        &#39;----.--.--.---&#39;
                             |  |  |
                       .-----&#39;  |  &#39;----.
                       |        |       |
                    .--v-.   .--v-.   .-v--.
                    | OP |   | RP |   | AS |
                    &#39;----&#39;   &#39;----&#39;   &#39;----&#39;
</code></pre>
</div>
<figcaption><a href="#figure-1" class="selfRef">Figure 1</a>: <a href="#name-two-coexisting-federations-" class="selfRef">Two Coexisting Federations with Some Members in Common</a></figcaption>
</figure>



### Requirements Notation and Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [[RFC2119](#RFC2119){.cite .xref}] [[RFC8174](#RFC8174){.cite .xref}] when, and only when, they appear in all capitals, as shown here.

All uses of JSON Web Signature (JWS) [[RFC7515](#RFC7515){.cite .xref}] and JSON Web Encryption (JWE) [[RFC7516](#RFC7516){.cite .xref}] data structures in this specification utilize the JWS Compact Serialization or the JWE Compact Serialization; the JWS JSON Serialization and the JWE JSON Serialization are not used.





### Terminology

This specification uses the terms "Claim", "Claim Name", "Claim Value", "JSON Web Token (JWT)", and "JWT Claims Set" defined by [JSON Web Token (JWT)](#RFC7519){.internal .xref} [[RFC7519](#RFC7519){.cite .xref}], the terms "OpenID Provider (OP)" and "Relying Party (RP)" defined by [OpenID Connect Core 1.0](#OpenID.Core){.internal .xref} [[OpenID.Core](#OpenID.Core){.cite .xref}], and the terms "Authorization Endpoint", "Authorization Server", "Client", "Client Authentication", "Client Identifier", "Client Secret", "Grant Type", "Protected Resource", "Redirection URI", "Refresh Token", and "Token Endpoint" defined by [OAuth 2.0](#RFC6749){.internal .xref} [[RFC6749](#RFC6749){.cite .xref}].

This specification also defines the following terms:

[]{.break}

Entity
:   Something that has a separate and distinct existence and that can be identified in a context.
:   

Entity Identifier
:   A globally unique string identifier that is bound to one Entity. All Entity Identifiers defined by this specification are URLs that use the `https` scheme, have a host component, and MAY contain port and path components. It MUST NOT contain query parameter or fragment components. Profiles of this specification MAY define other kinds of Entity Identifiers and processing rules that accompany them.
:   

Trust Anchor
:   An Entity that represents a trusted third party.
:   

Federation Entity
:   An Entity for which it is possible to construct a Trust Chain from the Entity to a Trust Anchor.
:   

Entity Statement
:   A signed JWT that contains the information needed for an Entity to participate in federation(s), including metadata about itself and policies that apply to other Entities for which it is authoritative.
:   

Entity Configuration
:   An Entity Statement issued by an Entity about itself. It contains the Entity's signing keys and further data used to control the Trust Chain resolution process, such as authority hints.
:   

Subordinate Statement
:   An Entity Statement issued by a Superior Entity about an Entity that is its Immediate Subordinate.
:   

Entity Type
:   A role and function that an Entity plays within a federation. An Entity MUST be of at least one type and MAY be of many types. For example, an Entity can be both an OpenID Provider and Relying Party at the same time.
:   

Entity Type Identifier
:   String identifier for an Entity Type.
:   

Federation Operator
:   An organization that is authoritative for a federation. A federation operator administers the Trust Anchor(s) for Entities in its federation.
:   

Intermediate Entity
:   An Entity that issues an Entity Statement appearing somewhere in between those issued by the Trust Anchor and the subject of a Trust Chain (which is typically a Leaf Entity). The terms Intermediate Entity and Intermediate are used interchangeably in this specification.
:   

Leaf Entity
:   An Entity with no Subordinate Entities. Leaf Entities typically play a protocol role, such as an OpenID Connect Relying Party or OpenID Provider. The terms Leaf Entity and Leaf are used interchangeably in this specification.
:   

Subordinate Entity
:   An Entity that is somewhere below a Superior Entity (a Trust Anchor or Intermediate) in the trust hierarchy, possibly with Intermediates between them. The terms Subordinate Entity and Subordinate are used interchangeably in this specification.
:   

Superior Entity
:   An Entity that is somewhere above one or more Entities (a Leaf or Intermediate) in the trust hierarchy, possibly with Intermediates between them. The terms Superior Entity and Superior are used interchangeably in this specification.
:   

Immediate Subordinate Entity
:   An Entity that is immediately below a Superior Entity in the trust hierarchy, with no Intermediates between them. The terms Immediate Subordinate Entity and Immediate Subordinate are used interchangeably in this specification.
:   

Immediate Superior Entity
:   An Entity that is immediately above one or more Subordinate Entities in the trust hierarchy, with no Intermediates between them. The terms Immediate Superior Entity and Immediate Superior are used interchangeably in this specification.
:   

Federation Entity Discovery
:   A process that starts with the Entity Identifier for the subject of the Trust Chain and collects Entity Statements until the chosen Trust Anchor is reached. From the collected Entity Statements, a Trust Chain is constructed and verified. The result of the Federation Entity Discovery is that the metadata for the Trust Chain subject is constructed from the Trust Chain.
:   

Trust Chain
:   A sequence of Entity Statements that represents a chain starting at an Entity Configuration that is the subject of the chain (typically of a Leaf Entity) and ending in a Trust Anchor.
:   

Trust Mark
:   Statement of conformance to a well-scoped set of trust and/or interoperability requirements as determined by an accreditation authority. Each Trust Mark has a Trust Mark type identifier.
:   

Trust Mark Issuer
:   A Federation Entity that issues Trust Marks.
:   

Trust Mark Owner
:   An Entity that owns the right to a Trust Mark type identifier.
:   

Federation Entity Keys
:   Keys used for the cryptographic signatures required by the trust mechanisms defined in this specification. Every participant in a Federation publishes its public Federation Entity Keys in its Entity Configuration.
:   

Resolved Metadata
:   The metadata that results from applying the metadata policy in the Trust Chain to the metadata in the Entity Configuration for the subject of the Trust Chain. The Resolved Metadata is the metadata that is used when interacting with the Entity.
:   







## Overall Architecture

The basic component is the Entity Statement, which is a cryptographically signed [JSON Web Token (JWT)](#RFC7519){.internal .xref} [[RFC7519](#RFC7519){.cite .xref}]. A set of Entity Statements can form a path from an Entity (typically a Leaf Entity) to a Trust Anchor. Entity Configurations issued by Entities about themselves control the Trust Chain resolution process.

The Entity Configuration of a Leaf or Intermediate Entity contains one or more references to its Immediate Superiors in the `authority_hints` parameter described in [Section 3.2](#authority_hints){.auto .internal .xref}. These references can be used to download the Entity Configuration of each Immediate Superior. One or more Entity Configurations are traversed during the Federation Entity Discovery until the Trust Anchor is reached.

The Trust Anchor and its Intermediates issue Entity Statements about their Immediate Subordinate Entities called Subordinate Statements. The sequence of Entity Configurations and Subordinate Statements that validate the relationship between a Superior and a Subordinate, along a path towards the Trust Anchor, forms the proof that the subject of Trust Chain (typically a Leaf Entity) is a member of the federation rooted at the Trust Anchor.

The chain that links the statements to one another is verified with the signature of each statement, as described in [Section 4](#trust_chain){.auto .internal .xref}.

Once there is a verified Trust Chain, the federation policy is applied and the metadata for the Trust Chain subject within the Federation is derived, as described in [Section 6](#federation_policy){.auto .internal .xref}.

This specification deals with trust operations; it does not cover or touch protocol operations other than metadata derivation and exchange. In OpenID Connect terms, these are the protocol operations specified in [OpenID Connect Discovery 1.0](#OpenID.Discovery){.internal .xref} [[OpenID.Discovery](#OpenID.Discovery){.cite .xref}] and [OpenID Connect Dynamic Client Registration 1.0](#OpenID.Registration){.internal .xref} [[OpenID.Registration](#OpenID.Registration){.cite .xref}].

OpenID Connect is used in many of the examples in this specification, however this does not mean that this specification can only be used with OpenID Connect. On the contrary, it can also be used to build federations for other protocols.



### Cryptographic Trust Mechanism

The objects defined by this specification that are used to establish cryptographic trust between participants are secured as signed JWTs using public key cryptography. In particular, the keys used for securing these objects are managed by the Entities controlling those objects, with the public keys securing them being distributed through those objects themselves. This kind of trust mechanism has been utilized by research and academic federations for over a decade.

Note that this cryptographic trust mechanism intentionally does not rely on Web PKI / TLS certificates for signing keys. Which TLS certificates are considered trusted can vary considerably between systems depending upon which certificate authorities are considered trusted and there are have been notable examples of ostensibly trusted certificates being compromised. For those reasons, this specification explicitly eschews reliance on Web PKI in favor of self-managed public keys, in this case, keys represented as JSON Web Keys (JWKs) [[RFC7517](#RFC7517){.cite .xref}].







## Entity Statement

An Entity Statement contains the information needed for the Entity that is the subject of the Entity Statement to participate in federation(s). An Entity Statement is a signed JWT. The subject of the JWT is the Entity itself. The issuer of the JWT is the party that issued the Entity Statement. All Entities in a federation publish an Entity Statement about themselves called an Entity Configuration. Superior Entities in a federation publish Entity Statements about their Immediate Subordinate Entities called Subordinate Statements.

Entity Statement JWTs MUST be explicitly typed, by setting the `typ` header parameter to `entity-statement+jwt` to prevent cross-JWT confusion, per Section 3.11 of [[RFC8725](#RFC8725){.cite .xref}]. Entity Statements without a `typ` header parameter or with a different `typ` value MUST be rejected.

The Entity Statement is signed using one of the private keys of the issuer Entity in the form of a [JSON Web Signature (JWS)](#RFC7515){.internal .xref} [[RFC7515](#RFC7515){.cite .xref}]. Implementations SHOULD support signature verification with the RSA SHA-256 algorithm because OpenID Connect Core requires support for it (`alg` value of `RS256`). Federations MAY also specify different mandatory-to-implement algorithms. Note that a Trust Chain can contain Entity Statements signed using different signing algorithms, as long as each signature uses a signature algorithm supported by the trust framework and implementations in use.

Entity Statement JWTs MUST include the `kid` (Key ID) header parameter with its value being the Key ID of the signing key used.

The claims in an Entity Statement are listed below. Applications and protocols utilizing Entity Statements MAY specify and use additional claims.



### Claims that MUST or MAY appear in both Entity Configurations and Subordinate Statements

[]{.break}

iss
:   REQUIRED. The Entity Identifier of the issuer of the Entity Statement. If the `iss` and the `sub` are identical, the issuer is making an Entity Statement about itself called an Entity Configuration.
:   

sub
:   REQUIRED. The Entity Identifier of the subject.
:   

iat
:   REQUIRED. Number. Time when this statement was issued. This is expressed as Seconds Since the Epoch, per [[RFC7519](#RFC7519){.cite .xref}].
:   

exp
:   REQUIRED. Number. Expiration time after which the statement MUST NOT be accepted for processing. This is expressed as Seconds Since the Epoch, per [[RFC7519](#RFC7519){.cite .xref}].
:   

jwks

:   REQUIRED. A [JSON Web Key Set (JWKS)](#RFC7517){.internal .xref} [[RFC7517](#RFC7517){.cite .xref}] representing the public part of the subject's Federation Entity signing keys. The corresponding private key is used by the Entity to sign the Entity Configuration about itself, by Trust Anchors and Intermediate Entities to sign Subordinate Statements about their Immediate Subordinates, and for other signatures made by Federation Entities, such as Trust Mark signatures. This claim is only OPTIONAL for the Entity Statement returned from an OP when the client is doing Explicit Registration; in all other cases, it is REQUIRED. Every JWK in the JWK Set MUST have a unique `kid` (Key ID) value. It is RECOMMENDED that the Key ID be the JWK Thumbprint [[RFC7638](#RFC7638){.cite .xref}] using the SHA-256 hash function of the key.

    These Federation Entity Keys SHOULD NOT be used in other protocols. (Keys to be used in other protocols, such as OpenID Connect, are conveyed in the `metadata` elements for the protocol's Entity Type Identifiers, such as the metadata under the `openid_provider` and `openid_relying_party` Entity Type Identifiers.)

:   

metadata

:   OPTIONAL. JSON object that declares roles that the Entity plays - its Entity Types - and that contains metadata for those Entity Types. Each member name of the JSON object is an Entity Type Identifier, and each value MUST be a JSON object containing metadata parameters according to the metadata schema of the Entity Type.

    When an Entity participates in a federation or federations with one or more Entity Types, its Entity Configuration MUST contain a `metadata` claim with JSON object values for each of the corresponding Entity Type Identifiers, even if the values are the empty JSON object `{}` (when the Entity Type has no associated metadata or Immediate Superiors supply any needed metadata).

    An Immediate Superior MAY provide selected or all metadata parameters for an Immediate Subordinate, by using the `metadata` claim in a Subordinate Statement. When `metadata` is used in a Subordinate Statement, it applies only to those Entity Types that are present in the subject's Entity Configuration. Furthermore, the metadata applies only to the subject of the Subordinate Statement and has no effect on the subject's Subordinates. Metadata parameters in a Subordinate Statement have precedence and override identically named parameters under the same Entity Type in the subject's Entity Configuration. If both `metadata` and `metadata_policy` appear in a Subordinate Statement, then the stated `metadata` MUST be applied before the `metadata_policy`, as described in [Section 6.1.4.2](#metadata_policy_application){.auto .internal .xref}.

:   

crit
:   OPTIONAL. Entity Statements require that the `crit` (critical) claim defined in [Section 13.4](#critClaim){.auto .internal .xref} be understood and processed. Claims in the Entity Statement whose claim names are in the array that is the value of this claim MUST be understood and processed. Claims specified for use in Entity Statements by this specification MUST NOT be included in the list.
:   





### Claims that MUST or MAY appear in Entity Configurations but not Subordinate Statements

[]{.break}

authority_hints

:   
    OPTIONAL. An array of strings representing the Entity Identifiers of Intermediate Entities or Trust Anchors that are Immediate Superiors of the Entity. This claim is REQUIRED in Entity Configurations of the Entities that have at least one Superior above them, such as Leaf and Intermediate Entities. Its value MUST contain the Entity Identifiers of its Immediate Superiors and MUST NOT be the empty array `[]`. This claim MUST NOT be present in Entity Configurations of Trust Anchors with no Superiors.
    

:   

trust_marks

:   OPTIONAL. An array of JSON objects, each representing a Trust Mark.

    []{.break}

    trust_mark_type
    :   REQUIRED. Identifier for the type of the Trust Mark. The value of this claim MUST be the same as the value of the `trust_mark_type` claim contained in the Trust Mark JWT that is the value of the `trust_mark` claim in this object.
    :   

    trust_mark
    :   REQUIRED. A signed JSON Web Token that represents a Trust Mark.
    :   

    Trust Marks are described in [Section 7](#trust_marks){.auto .internal .xref}.

:   

trust_mark_issuers
:   OPTIONAL. A Trust Anchor MAY use this claim to tell which combination of Trust Mark type identifiers and issuers are trusted by the federation. This claim MUST be ignored if present in an Entity Configuration for an Entity that is not a Trust Anchor. It is a JSON object with member names that are Trust Mark type identifiers and each corresponding value being an array of Entity Identifiers that are trusted to represent the accreditation authority for Trust Marks with that identifier. If the array following a Trust Mark type identifier is empty, anyone MAY issue Trust Marks with that identifier. Trust Marks are described in [Section 7](#trust_marks){.auto .internal .xref}.
:   

trust_mark_owners

:   OPTIONAL. If a Federation Operator knows that a Trust Mark type identifier is owned by an Entity different from the Trust Mark Issuer, then that knowledge MUST be expressed in this claim. This claim MUST be ignored if present in an Entity Configuration for an Entity that is not a Trust Anchor. It is a JSON object with member names that are Trust Mark type identifiers and each corresponding value being a JSON object with these members:

    []{.break}

    sub
    :   REQUIRED Identifier of the Trust Mark Owner.
    :   

    jwks
    :   REQUIRED [JSON Web Key Set (JWKS)](#RFC7517){.internal .xref} [[RFC7517](#RFC7517){.cite .xref}] containing the owner's Federation Entity Keys used for signing.
    :   

    Other members MAY also be defined and used.

:   





### Claims that MUST or MAY appear in Subordinate Statements but not Entity Configurations

[]{.break}

constraints
:   OPTIONAL. JSON object that defines Trust Chain constraints, as described in [Section 6.2](#chain_constraints){.auto .internal .xref}. The constraints apply to the Entity that is the subject of this Subordinate Statement as well as to all Entities that are Subordinate to it.
:   

metadata_policy
:   OPTIONAL. JSON object that defines a metadata policy, as described in [Section 6.1](#metadata_policy){.auto .internal .xref}. The metadata policy applies to the Entity that is the subject of this Subordinate Statement as well as to all Entities that are Subordinate to it. Applying to Subordinate Entities distinguishes `metadata_policy` from `metadata`, which only applies to the subject itself.
:   

metadata_policy_crit
:   OPTIONAL. Array of strings specifying critical metadata policy operators other than the standard ones defined in [Section 6.1.3.1](#standard_metadata_policy_operators){.auto .internal .xref} that MUST be understood and processed. When included its value MUST NOT be the empty array `[]`. If any of the listed policy operators are not understood and supported, then the Subordinate Statement and thus the Trust Chain that includes it MUST be considered invalid.
:   

source_endpoint
:   OPTIONAL. String containing the fetch endpoint URL from which the Entity Statement was issued, as specified in [Section 8.1](#fetch_endpoint){.auto .internal .xref}. This parameter enables an optimized refresh of the Subordinate Statements and hence the Trust Chain, by skipping the request to the Entity Configuration that is normally required to discover the `federation_fetch_endpoint` of the issuing authority. If an Entity Statement cannot be retrieved from the `source_endpoint`, which can occur if the endpoint URL has changed, the current `federation_fetch_endpoint` location can be determined by retrieving the Entity Configuration of the issuer.
:   




### Claims Specific to Explicit Registration Responses

[]{.break}

trust_anchor
:   OPTIONAL. Its value MUST be the Entity Identifier of the Trust Anchor that the OP selected to process the Explicit Registration request. This claim is specific to Explicit Registration responses and is not a general Entity Statement claim.
:   




### Entity Statement Validation

Entity Statements MUST be validated in the following manner. These steps MAY be performed in a different order, provided that the result - accepting or rejecting the Entity Statement - is the same.

1.  
    The Entity Statement MUST be a signed JWT.
    

2.  
    The Entity Statement MUST have a `typ` header parameter with the value `entity-statement+jwt`.
    

3.  
    The Entity Statement MUST have an `alg` (algorithm) header parameter with a value that is an acceptable JWS signing algorithm; it MUST NOT be `none`.
    

4.  
    The Entity Identifier of the Entity to which the Entity Statement refers MUST match the value of the `sub` (subject) Claim.
    

5.  
    The Entity Statement MUST have an `iss` (issuer) Claim with a value that is a valid Entity Identifier.
    

6.  
    When the `iss` (issuer) Claim value matches the `sub` (subject) Claim value, then the Entity Statement is this Entity's Entity Configuration. When they do not match, the Entity Statement is a Subordinate Statement. When the Entity Statement is a Subordinate Statement, the `iss` Claim value MUST match one of the values in the `authority_hints` array in the Entity Configuration for the Entity whose Entity Identifier is the value of the `sub` Claim; otherwise the Federation graph is not well-formed.
    

7.  
    The current time MUST be after the time represented by the `iat` (issued at) Claim (possibly allowing for some small leeway to account for clock skew).
    

8.  
    The current time MUST be before the time represented by the `exp` (expiration) Claim (possibly allowing for some small leeway to account for clock skew).
    

9.  
    The `jwks` (JWK Set) Claim MUST be present, with a value that is a valid JWK Set [[RFC7517](#RFC7517){.cite .xref}].
    

10. 
    Obtain the Entity Configuration for the issuing Entity - the Entity with the Issuer Identifer found in the Entity Statement's `iss` (issuer) Claim. When the `iss` and `sub` Claim values match, this is the Entity Statement being validated itself. Otherwise, this can be obtained either from a Trust Chain or by retrieving it as described in [Section 9](#federation_configuration){.auto .internal .xref}.
    

11. 
    The Entity Statement's `kid` (Key ID) header parameter value MUST be a non-zero length string and MUST exactly match the `kid` value for a key in the `jwks` (JWK Set) Claim of the Entity Configuration of the issuing Entity.
    

12. 
    The Entity Statement's signature MUST validate using the issuing Entity's key identified by the `kid` value.
    

13. 
    If the `crit` Claim is present, then each array element in this claim's value MUST be a string representing an Entity Statement claim that is not defined by this specification and that claim MUST be understood and be able to be processed by the implementation.
    

14. 
    If the `authority_hints` Claim is present, the Entity Statement MUST be an Entity Configuration. Verify that its value is syntactially correct, as specified in [Section 3.2](#authority_hints){.auto .internal .xref}. Implementations MAY also validate that the Entity is a Subordinate of each Entity whose Entity Identifier is listed in the `authority_hints` array.
    

15. 
    If the `metadata` Claim is present, verify that its value is syntactially correct, not using `null` as metadata values, as specified in [Section 5](#metadata){.auto .internal .xref}.
    

16. 
    If the `metadata_policy` Claim is present, the Entity Statement be a Subordinate Statement. Verify that its value is syntactially correct, as specified in [Section 6.1](#metadata_policy){.auto .internal .xref}.
    

17. 
    If the `metadata_policy_crit` Claim is present, the Entity Statement be a Subordinate Statement. Each array element in this claim's value MUST be a string representing a Metadata Policy operator that is not defined by this specification and that operator MUST be understood and be able to be processed by the implementation.
    

18. 
    If the `constraints` Claim is present, the Entity Statement be a Subordinate Statement. Verify that its value is syntactically correct, as specified in [Section 6.2](#chain_constraints){.auto .internal .xref}.
    

19. 
    If the `trust_marks` Claim is present, the Entity Statement MUST be an Entity Configuration. Validate that the syntax of this Claim Value conforms to the Claim definition. In particular, for each element of the array that is the Claim value, validate that there is a `trust_mark_type` member whose value matches the `trust_mark_type` Claim value in the Trust Mark JWT that is the value of the `trust_mark` member. Validating the syntax is separate from evaluating whether particular Trust Marks are issued by a trusted party and are trusted; that process is described in [Section 7.3](#trust-mark-validation){.auto .internal .xref} and MAY be performed as a separate step from syntactic validation.
    

20. 
    If the `trust_mark_issuers` Claim is present, the Entity Statement MUST be an Entity Configuration. Validate that its Claim value is a JSON object with Trust Mark type identifiers as the member names and arrays of Entity Identifiers as the values.
    

21. 
    If the `trust_mark_owners` Claim is present, the Entity Statement MUST be an Entity Configuration. Validate that its Claim value is a JSON object with Trust Mark type identifiers as the member names and values that are JSON objects containing a `sub` member with a value that is an Entity Identifier and a `jwks` member with a value that is a JSON Web Key Set.
    

22. 
    If the `source_endpoint` Claim is present, the Entity Statement MUST be a Subordinate Statement. Validate that its Claim value is a URL. Implementations MAY also make a fetch call to the URL to validate that this is the fetch endpoint from which the Entity Statement was issued.
    

23. 
    If the `trust_anchor` Claim is present, validate that its value is a URL using the `https` scheme. Implementations SHOULD validate that the Entity Identifier matches one of the Trust Anchors configured for the deployment. Furthermore, implementations SHOULD validate that the Entity Configuration for the Entity Identifier contains information compatible with the configured Trust Anchor information - especially the keys. This Claim MUST NOT be present in Entity Statements that are not Explicit Registration responses.
    

If any of these validation steps fail, the Entity Statement MUST be rejected.





### Entity Statement Examples

The following is a non-normative example of the JWT Claims Set for an Entity Statement. The example contains a critical extension `jti` (JWT ID) to the Entity Statement and one critical extension to the policy language `regexp` (Regular expression).

[]{#name-example-entity-statement-jw}

<figure id="figure-2">
<div id="section-3.6-2.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;iss&quot;: &quot;https://feide.no&quot;,
  &quot;sub&quot;: &quot;https://ntnu.no&quot;,
  &quot;iat&quot;: 1516239022,
  &quot;exp&quot;: 1516298022,
  &quot;jwks&quot;: {
    &quot;keys&quot;: [
      {
        &quot;kty&quot;: &quot;RSA&quot;,
        &quot;alg&quot;: &quot;RS256&quot;,
        &quot;use&quot;: &quot;sig&quot;,
        &quot;kid&quot;: &quot;NzbLsXh8uDCcd-6MNwXF4W_7noWXFZAfHkxZsRGC9Xs&quot;,
        &quot;n&quot;: &quot;pnXBOusEANuug6ewezb9J_...&quot;,
        &quot;e&quot;: &quot;AQAB&quot;
      }
    ]
  },
  &quot;metadata&quot;: {
    &quot;openid_provider&quot;: {
      &quot;issuer&quot;: &quot;https://ntnu.no&quot;,
      &quot;organization_name&quot;: &quot;NTNU&quot;
    },
    &quot;oauth_client&quot;: {
      &quot;organization_name&quot;: &quot;NTNU&quot;
    }
  },
  &quot;metadata_policy&quot;: {
    &quot;openid_provider&quot;: {
      &quot;id_token_signing_alg_values_supported&quot;:
        {&quot;subset_of&quot;: [&quot;RS256&quot;, &quot;RS384&quot;, &quot;RS512&quot;]},
      &quot;op_policy_uri&quot;: {
        &quot;regexp&quot;:
          &quot;^https://[w-]+.example.com/[w-]+.html&quot;}
    },
    &quot;oauth_client&quot;: {
      &quot;grant_types&quot;: {
        &quot;one_of&quot;: [&quot;authorization_code&quot;, &quot;client_credentials&quot;]
      }
    }
  },
  &quot;constraints&quot;: {
    &quot;max_path_length&quot;: 2
  },
  &quot;crit&quot;: [&quot;jti&quot;],
  &quot;metadata_policy_crit&quot;: [&quot;regexp&quot;],
  &quot;source_endpoint&quot;: &quot;https://feide.no/federation_api/fetch&quot;,
  &quot;jti&quot;: &quot;7l2lncFdY6SlhNia&quot;
}</code></pre>
</div>
<figcaption><a href="#figure-2" class="selfRef">Figure 2</a>: <a href="#name-example-entity-statement-jw" class="selfRef">Example Entity Statement JWT Claims Set</a></figcaption>
</figure>

The following is a non-normative example of a `trust_mark_owners` claim value:

[]{#name-example-trust_mark_owners-c}

<figure id="figure-3">
<div id="section-3.6-4.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;https://refeds.org/wp-content/uploads/2016/01/Sirtfi-1.0.pdf&quot;:
    {
      &quot;sub&quot;: &quot;https://refeds.org/sirtfi&quot;,
      &quot;jwks&quot; : {
        &quot;keys&quot;: [
          {
            &quot;alg&quot;: &quot;RS256&quot;,
            &quot;e&quot;: &quot;AQAB&quot;,
            &quot;kid&quot;: &quot;key1&quot;,
            &quot;kty&quot;: &quot;RSA&quot;,
            &quot;n&quot;: &quot;pnXBOusEANuug6ewezb9J_...&quot;,
            &quot;use&quot;: &quot;sig&quot;
          }
        ]
      }
    }
}</code></pre>
</div>
<figcaption><a href="#figure-3" class="selfRef">Figure 3</a>: <a href="#name-example-trust_mark_owners-c" class="selfRef">Example <code>trust_mark_owners</code> Claim Value</a></figcaption>
</figure>

The following is a non-normative example of a `trust_mark_issuers` claim value:

[]{#name-example-trust_mark_issuers-}

<figure id="figure-4">
<div id="section-3.6-6.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;https://openid.net/certification/op&quot;: [],
  &quot;https://refeds.org/wp-content/uploads/2016/01/Sirtfi-1.0.pdf&quot;:
    [&quot;https://swamid.se&quot;]
}</code></pre>
</div>
<figcaption><a href="#figure-4" class="selfRef">Figure 4</a>: <a href="#name-example-trust_mark_issuers-" class="selfRef">Example <code>trust_mark_issuers</code> Claim Value</a></figcaption>
</figure>







## Trust Chain

Entities whose statements build a Trust Chain are categorized as:

[]{.break}

Trust Anchor
:   An Entity that represents a trusted third party.
:   

Leaf
:   In an OpenID Connect identity federation, an RP or an OP, or in an OAuth 2.0 federation, a Client, Authorization Server, or Protected Resource.
:   

Intermediate
:   Neither a Leaf Entity nor a Trust Anchor.
:   

A Trust Chain begins with an Entity Configuration that is the subject of the Trust Chain, which is typically a Leaf Entity. The Trust Chain has zero or more Subordinate Statements issued by Intermediates about their Immediate Subordinates, and includes the Subordinate Statement issued by the Trust Anchor about the top-most Intermediate (if there are Intermediates) or the Trust Chain subject (if there are no Intermediates). The Trust Chain logically always ends with the Entity Configuration of the Trust Anchor, even though it MAY be omitted from the JSON array representing the Trust Chain in some cases.

The Trust Chain contains the configuration of the federation as it applies to the Trust Chain subject at the time of the evaluation of the chain.

A simple example: If we have an RP that belongs to Organization A that is a member of Federation F, the Trust Chain for such a setup will contain the following Entity Statements:

1.  
    an Entity Configuration about the RP published by itself,
    

2.  
    a Subordinate Statement about the RP published by Organization A,
    

3.  
    a Subordinate Statement about Organization A published by the Trust Anchor for F,
    

4.  
    an Entity Configuration about the Trust Anchor for F published by itself.
    

Let us refer to the Entity Statements in the Trust Chain as ES[j], where j = 0,...,i, with 0 being the index of the first Entity Statement and i being the zero-based index of the last. Then:

- 
  ES[0] (the Entity Configuration of the Trust Chain subject) is signed using a key in ES[0]["jwks"].
  

- 
  The `iss` claim in an Entity Statement is equal to the `sub` claim in the next. Restating this symbolically, for each j = 0,...,i-1, ES[j]["iss"] == ES[j+1]["sub"].
  

- 
  An Entity Statement is signed using a key from the `jwks` claim in the next. Restating this symbolically, for each j = 0,...,i-1, ES[j] is signed by a key in ES[j+1]["jwks"].
  

- 
  ES[i] (the Trust Anchor's Entity Configuration in the Trust Chain) is signed using a key in ES[i]["jwks"].
  

The Trust Anchor's public keys are used to verify the signatures on ES[i] (the Trust Anchor's Entity Configuration) and ES[i-1] (the Trust Anchor's Subordinate Statement about its Immediate Subordinate in the Trust Chain). The Trust Anchor's public keys are distributed to Entities that need to verify a Trust Chain in some secure out-of-band way not described in this document.



### Beginning and Ending Trust Chains

A Trust Chain begins with the Entity Configuration of an Entity for which trust is being established. This is the subject of the Trust Chain. This Entity typically plays a protocol role, such as being an OpenID Provider or OpenID Relying Party.

While such Entities are typically Leaf Entities, other topologies are possible. For instance, an Entity might simultaneously be an OpenID Provider and an Intermediate Entity, with OpenID Relying Parties and/or other Intermediates being Subordinate to it. This is a case in which the subject of a Trust Chain would not be a Leaf Entity.

A Trust Chain ends with the Entity Configuration of a Trust Anchor. While Trust Anchors typically have no Superiors, a Trust Anchor will have a Superior when the Trust Anchor also acts as an Intermediate Entity in another federation. This will be the case when there is a hierarchy of federations.

Thus, while it is typical for a Trust Chain to end with a Trust Anchor with no Superiors, there are situations in which the Trust Chain will end with a Trust Anchor that nonetheless has Superior Entities.





### Trust Chain Example

The following is an example of a Trust Chain consisting of a Leaf's Entity Configuration and Subordinate Statements issued by an Intermediate Entity and a Trust Anchor. It shows the relationship between the three Entities, their Entity Configurations, and their Subordinate Statements. The Subordinate Statements are obtained from the `federation_fetch_endpoint` of the subject's Immediate Superior. The URL of the `federation_fetch_endpoint` is discovered in the Immediate Superior's Entity Configuration. Note that the first member of the Trust Chain (the Leaf) is depicted at the bottom of the diagram and that the last member (the Trust Anchor) is depicted at the top.

[]{#name-relationships-between-feder}

<figure id="figure-5">
<div id="section-4.2-2.1" class="alignLeft art-text artwork">
<pre><code>.----------------.  .---------------------------.         .---------------------------.
| Role           |  | .well-known/              |         | Trust Chain               |
|                |  | openid-federation         |         |                           |
.----------------.  .---------------------------.         .---------------------------.
| .------------. |  | .-----------------------. |         | .-----------------------. |
| |            | |  | | Entity Configuration  | |         | | Entity Configuration  | |
| |Trust Anchor+-+--+-&gt;                       +-+---------+-&gt;                       | |
| |            | |  | | Federation Entity Keys| |         | | Federation Entity Keys| |
| &#39;-----.------&#39; |  | | Metadata              | |         | | Metadata              | |
|       |        |  | | Trust Mark Issuers    | |         | | Trust Mark Issuers    | |
|       |        |  | |                       | |         | |                       | |
|       |        |  | &#39;-----------------------&#39; |         | &#39;-----------------------&#39; |
|       |        |  |                           |         |                           |
|       |        |  |                           |Fetch    | .-----------------------. |
|       |        |  |                           |Endpoint | | Subordinate Statement | |
|       +--------+--+---------------------------+---------+-&gt;                       | |
|                |  |                           |         | | Federation Entity Keys| |
|                |  |                           |         | | Metadata Policy       | |
|                |  |                           |         | | Metadata              | |
|                |  |                           |         | | Constraints           | |
|                |  |                           |         | |                       | |
|                |  |                           |         | &#39;-----------.-----------&#39; |
| .------------. |  | .-----------------------. |         |             |             |
| |            | |  | | Entity Configuration  | |         |             |             |
| |Intermediate+-+--+-&gt;                       | |         |             |sub and key  |
| |            | |  | | Federation Entity Keys| |         |             | binding     |
| &#39;------.-----&#39; |  | | Metadata              | |         | .-----------v-----------. |
|        |       |  | | Trust Marks           | |         | | Subordinate Statement | |
|        |       |  | |                       | |         | |                       | |
|        |       |  | |                       | |         | | Federation Entity Keys| |
|        |       |  | &#39;-----------------------&#39; |Fetch    | | Metadata Policy       | |
|        |       |  |                           |Endpoint | | Metadata              | |
|        +-------+--+---------------------------+---------+-&gt;                       | |
|                |  |                           |         | &#39;-----------.-----------&#39; |
|                |  |                           |         |             |sub and key  |
|                |  |                           |         |             | binding     |
| .------------. |  | .-----------------------. |         | .-----------v-----------. |
| |            | |  | | Entity Configuration  | |         | | Entity Configuration  | |
| | Leaf       +-+--+-&gt;                       +-+---------+-&gt;                       | |
| |            | |  | | Federation Entity Keys| |         | | Federation Entity Keys| |
| &#39;------------&#39; |  | | Metadata              | |         | | Metadata              | |
|                |  | | Trust Marks           | |         | | Trust Marks           | |
|                |  | |                       | |         | |                       | |
|                |  | |                       | |         | |                       | |
|                |  | &#39;-----------------------&#39; |         | &#39;-----------------------&#39; |
&#39;----------------&#39;  &#39;---------------------------&#39;         &#39;---------------------------&#39;
</code></pre>
</div>
<figcaption><a href="#figure-5" class="selfRef">Figure 5</a>: <a href="#name-relationships-between-feder" class="selfRef">Relationships between Federation Entities and Statements Issued in a Trust Chain</a></figcaption>
</figure>





### Trust Chain Header Parameter

The `trust_chain` JWS header parameter is a JSON array containing the sequence of the statements that proves the trust relationship between the issuer of the [JWS](#RFC7515){.internal .xref} [[RFC7515](#RFC7515){.cite .xref}] and the selected Trust Anchor, sorted as shown in [Section 4](#trust_chain){.auto .internal .xref}. The Trust Chain contains the public key used to verify the [JWS](#RFC7515){.internal .xref} [[RFC7515](#RFC7515){.cite .xref}]. The issuer of the JWS SHOULD select the Trust Anchor that it has in common with the audience of the JWS. Otherwise, the issuer is free to select a Trust Anchor. Most signed JWTs MAY include the `trust_chain` JWS header parameter, with a few exceptions. Entity Configurations and Subordinate Statements MUST NOT contain the `trust_chain` header parameter, as they are integral components of a Trust Chain. Additionally, the Authorization Signed Request Object SHOULD NOT contain the `trust_chain` header parameter, because the `trust_chain` parameter is intended to be part of its JWS payload, as defined in [Section 12.1.1.1](#trust_chain_param){.auto .internal .xref}. Use of this header parameter is OPTIONAL.

The following is a non-normative example of a JWS header with the `trust_chain` parameter.

[]{#name-example-jws-header-with-a-t}

<figure id="figure-6">
<div id="section-4.3-3.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;typ&quot;: &quot;...&quot;,
  &quot;alg&quot;: &quot;RS256&quot;,
  &quot;kid&quot;: &quot;SUdtUndEWVY2cUFDeDV5NVlBWDhvOXJodVl2am1mNGNtR0pmd&quot;,
  &quot;trust_chain&quot;: [
    &quot;eyJ0eXAiOiJlbnRpdHktc3RhdGVtZW50K2p3dCIsImFsZyI6IlJTMjU2Iiwia2lkIjoiVUdaSGF6aHVZalpoT0Y5amNUaHBWa05KU1VkWWQxVnlaR1JGVXpWd09FVXlSMDQwU2tjMk1XMXVPQSJ9.eyJtZXRhZGF0YSI6IHsib3BlbmlkX2NyZWRlbnRpYWxfaXNzdWVyIjogeyJqd2tzIjogeyJrZXlzIjogW3sia3R5IjogIlJTQSIsICJraWQiOiAiUjJSelJYQTBSVkJ5ZHpGT1ZHMWZkV1JUTVRaM1lUUm1ObkUxVjNGZk1FMW9NVVpMZWtsaVkxTllPQSIsICJlIjogIkFRQUIiLCAibiI6ICI1SF9YaDd4Z0RXVHhRVmJKcW1PR3Vyb2tFOGtyMmUxS2dNV2NZT0E3NE9fMVBYZDJ1Z2p5SXE5dDFtVlBTdXd4LXR5U2syUEtwanAtLVdySG4zQTRVS0prdVIxMXpobWRMQnNVOFRPQkJ1NU1aOGF0RHVqZlJ3SUxYZEtzRVhrbHZhQjZQTFQ0emRab2RnQ3MwNUt5MmU1c2I1ejZfQ2lEcWdVVm5XUG1KTE1rZ3BCdFota01kX2xiOVNvb1psbGZVR2xUa3NhdUoyX2dWUS1WcEZVTVhZam9Kak54OTdldWthWW5SRW9DQzNUYV8tOGJjUm9zbHgyeHJJYnVfVUdWcWlwZU4zTlAtbWVmZjlWVFpXWU0zZ21vbHd1cG5NQ1hYaWlrY1I1VVNMVmcwZV9nejZPZm9SVkdLQVdJcFJSTFR6MmFpcXVrVkhaZFpYOXRObXowbXcifV19fSwgImZlZGVyYXRpb25fZW50aXR5IjogeyJvcmdhbml6YXRpb25fbmFtZSI6ICJPcGVuSUQgQ3JlZGVudGlhbCBJc3N1ZXIgZXhhbXBsZSIsICJvcmdhbml6YXRpb25fdXJpIjogImh0dHBzOi8vY3JlZGVudGlhbF9pc3N1ZXIuZXhhbXBsZS5vcmcvaG9tZSIsICJwb2xpY3lfdXJpIjogImh0dHBzOi8vY3JlZGVudGlhbF9pc3N1ZXIuZXhhbXBsZS5vcmcvcG9saWN5IiwgImxvZ29fdXJpIjogImh0dHBzOi8vY3JlZGVudGlhbF9pc3N1ZXIuZXhhbXBsZS5vcmcvc3RhdGljL2xvZ28uc3ZnIiwgImNvbnRhY3RzIjogWyJ0ZWNoQGNyZWRlbnRpYWxfaXNzdWVyLmV4YW1wbGUub3JnIl19fSwgImF1dGhvcml0eV9oaW50cyI6IFsiaHR0cHM6Ly9pbnRlcm1lZGlhdGUuZWlkYXMuZXhhbXBsZS5vcmciXSwgImp3a3MiOiB7ImtleXMiOiBbeyJrdHkiOiAiUlNBIiwgImtpZCI6ICJNR281VVY4dGIwRkpha2xEVm10aFpuTmpZWE50V1hvd1J6aG9OVzQwTTBkTVJqTlRTRWd3TWpNeE9BIiwgIm4iOiAielVITzlvQWJUOXg4TWhGNEdTY0JPU3BkaVRoMGM3dWh0eFo3WG9vRFMxckZyQXRXVUNRLV9nUUFjVmVsMEFjcUIxdEoyQmFTaEdobkdsX1ZVYTlqSXE5WkJpc2xQZS1tZ01STFFYNm1HdnhKR19Gb0FRQ3BOUGNXRWRCT21uNUFkVGxXeTFoYk9mMlY5YV9TUzdKeUpnaVJJMXBodm43bmhyNVVmMFRUX1B2NzJsS0tBZ3drUmdEOXZtWVRSOFBTSzF3b2d6WVJSMFYwYVg2M3I5Xy0wb2ZZN3hRRXIwR2xVY2dqZlVyS0dzS0JJNWswalQtQlIxdXpxaWJVSHlsblQ5UXBNajhBQ0docmktYXRnb2t0Z1ZtNklUTmJXYTBCcnIyMWVLVUpKaHZ1eGVQTUxhUDVPdUoxRy1CanhxakNwOHJGaTB0bWJOU0pWanNiUlplQ3lRIiwgImUiOiAiQVFBQiJ9XX0sICJzdWIiOiAiaHR0cHM6Ly9jcmVkZW50aWFsX2lzc3Vlci5leGFtcGxlLm9yZyIsICJpc3MiOiAiaHR0cHM6Ly9pbnRlcm1lZGlhdGUuZWlkYXMuZXhhbXBsZS5vcmciLCAiaWF0IjogMTc1ODUyNzgxOCwgImV4cCI6IDE3NTg4Mjc4MTh9.W3fbv3JrAxcDsV0MHk00MzcgC1DddTrzkRN8vdT1IRwq3qy9NtsiC2532oInHoxxjlKBu7D8cqF9yG0Tb1v4gk_tejyUo0M9xJqyz32RU2iZP0lpbNHHUDrMxuGv1wPDap2mKisRgbb7pxm6dx_aaIrydhx0uKDM6EwX1RzpMxJeqNMeLK9992_xyCZvsi3kVGRCJDSqd-rXBS_2LFWKUViC1_5GcsWAkRABBgRDeARqQah3FvJVWiAcNv2Te2k6SMW1MNVCT7Q3uf3c2vVuaVA1OwY_wUTrkfFRjKEOmU1sgZ1TndxJKQW0XZZmTxcoJfmrjmCyxtteucYznESMnw&quot;,
    &quot;eyJ0eXAiOiJlbnRpdHktc3RhdGVtZW50K2p3dCIsImFsZyI6IlJTMjU2Iiwia2lkIjoiVUdaSGF6aHVZalpoT0Y5amNUaHBWa05KU1VkWWQxVnlaR1JGVXpWd09FVXlSMDQwU2tjMk1XMXVPQSJ9.eyJzdWIiOiAiaHR0cHM6Ly9jcmVkZW50aWFsX2lzc3Vlci5leGFtcGxlLm9yZyIsICJqd2tzIjogeyJrZXlzIjogW3sia3R5IjogIlJTQSIsICJraWQiOiAiTUdvNVVWOHRiMEZKYWtsRFZtdGhabk5qWVhOdFdYb3dSemhvTlc0ME0wZE1Sak5UU0Vnd01qTXhPQSIsICJuIjogInpVSE85b0FiVDl4OE1oRjRHU2NCT1NwZGlUaDBjN3VodHhaN1hvb0RTMXJGckF0V1VDUS1fZ1FBY1ZlbDBBY3FCMXRKMkJhU2hHaG5HbF9WVWE5aklxOVpCaXNsUGUtbWdNUkxRWDZtR3Z4SkdfRm9BUUNwTlBjV0VkQk9tbjVBZFRsV3kxaGJPZjJWOWFfU1M3SnlKZ2lSSTFwaHZuN25ocjVVZjBUVF9QdjcybEtLQWd3a1JnRDl2bVlUUjhQU0sxd29nellSUjBWMGFYNjNyOV8tMG9mWTd4UUVyMEdsVWNnamZVcktHc0tCSTVrMGpULUJSMXV6cWliVUh5bG5UOVFwTWo4QUNHaHJpLWF0Z29rdGdWbTZJVE5iV2EwQnJyMjFlS1VKSmh2dXhlUE1MYVA1T3VKMUctQmp4cWpDcDhyRmkwdG1iTlNKVmpzYlJaZUN5USIsICJlIjogIkFRQUIifV19LCAiaXNzIjogImh0dHBzOi8vaW50ZXJtZWRpYXRlLmVpZGFzLmV4YW1wbGUub3JnIiwgImlhdCI6IDE3NTg1Mjc4MTgsICJleHAiOiAxNzU4ODI3ODE4fQ.hF0ABpYlQBPzeMJrANe-5VOStDbZrEFdYVnNVwRgxphCMEMyoBfHDVv9kQceJtKqKbzFyFFCiO6QPY-GGt-eI37YxdzG5F8GCr9hBXfoUtTSiWaEop3B0AVXH0SelRO5zvN8W2SlR0rPTJ75kLv2vaVbgES_IzMhneteRvx2HGvCxOdvA4kermxkFT7MSP3YGyuNGJ4JAEvLXT6TmL9wiitGJ3SO34ImWZ4uI9zmSzWqvRIFKZ05dD2_RVybJbKQcOuRS3Th2yn0uq4YPzPw-Na2mw0FcYXMKQhq-SvdkI4Rt4eQMbIyyminMuTxrdIeQD6-rvSOxUTjF31sjnekvA&quot;,
    &quot;eyJ0eXAiOiJlbnRpdHktc3RhdGVtZW50K2p3dCIsImFsZyI6IlJTMjU2Iiwia2lkIjoiTjFsNWVWWTJhMmszU0ZCSGNtcFRjMVJJYkZGM1R6VmlZbkp2VTBNNGJuaE1ZMXBxTkdod01VcEZTUSJ9.eyJzdWIiOiAiaHR0cHM6Ly9pbnRlcm1lZGlhdGUuZWlkYXMuZXhhbXBsZS5vcmciLCAiandrcyI6IHsia2V5cyI6IFt7Imt0eSI6ICJSU0EiLCAia2lkIjogIlVHWkhhemh1WWpaaE9GOWpjVGhwVmtOSlNVZFlkMVZ5WkdSRlV6VndPRVV5UjA0MFNrYzJNVzF1T0EiLCAibiI6ICJvTlhGczdmajR5Q0k2TUotS3JSbDM0WkpBbGR1N2x3ZE5rdHk2N1FHTUx5Wmh6dFpkT0VYV2w3Z1ZRUldIeUM3M01kTGlhU1VJUFB0R2daak5iSVJEUDF0UDFJZWlObU52bmhWTnNCbWNiVjhLRUNkXy1GRGlYWDRTT0VNU0ItNXVhTGpsay1VTGpZajdwZGhPSjJqcWdIQ1dHMmFhMkozYXdUY25zSXc1NDF0YVdoc2NGZlZIT0ktMVZnbzVCTDdmRGdEakRLcGtpeGg4SE5PUVVQTXJrMWdZb2tiakt3V0RKZXphUndtamVNWldOczB1aTFvdmlFWW0xQzRfTVNSWW8wUEVOWHk0QmQ2QzVHLTVTanZyNU83U1dLN19GUWpsY09yZXhrUERyNmdQQWZHZVZnNW1CblJYOGp3b1l0TnlMal94SlpSRGZKMDdjTTRfNXl4cVEiLCAiZSI6ICJBUUFCIn1dfSwgImlzcyI6ICJodHRwczovL3RydXN0LWFuY2hvci5leGFtcGxlLm9yZyIsICJpYXQiOiAxNzU4NTI3ODE4LCAiZXhwIjogMTc1ODgyNzgxOH0.G3DP1Nqt8UTXwes0ozKzADd1KmAYl0K0DhyjQMJOKR7pgqYp_S91ObMKPrBJDh1Mnpy7mpHICUM23pUEMFt7v_-MdqjPTOG5ebrxqjFtemizFk6iHWvwhCcheQIsNMRXf-y64Ox6SgBVXP_JaKHtA_YInTQB9-_bdb-mWvwpOlvvGH-l5Hl7TQten_IoewT9bfC62UkOmKzn0L1VlVFtYn9R7sRFUtnfcqsvvRLIHNfJwaJi-lpODgNv2l7arHssS_FmfX5ZN-3WylJJbZhtkFku1ITKx7c4bvGsZSvEx9m_ixGJYunfD-iKWAmfE1suc6X-ywR7dgdp_BBpMvytlg&quot;,
    &quot;eyJ0eXAiOiJlbnRpdHktc3RhdGVtZW50K2p3dCIsImFsZyI6IlJTMjU2Iiwia2lkIjoiTjFsNWVWWTJhMmszU0ZCSGNtcFRjMVJJYkZGM1R6VmlZbkp2VTBNNGJuaE1ZMXBxTkdod01VcEZTUSJ9.eyJtZXRhZGF0YSI6IHsiZmVkZXJhdGlvbl9lbnRpdHkiOiB7ImZlZGVyYXRpb25fZmV0Y2hfZW5kcG9pbnQiOiAiaHR0cHM6Ly90cnVzdC1hbmNob3IuZXhhbXBsZS5vcmcvZmV0Y2giLCAiZmVkZXJhdGlvbl9yZXNvbHZlX2VuZHBvaW50IjogImh0dHBzOi8vdHJ1c3QtYW5jaG9yLmV4YW1wbGUub3JnL3Jlc29sdmUiLCAiZmVkZXJhdGlvbl9saXN0X2VuZHBvaW50IjogImh0dHBzOi8vdHJ1c3QtYW5jaG9yLmV4YW1wbGUub3JnL2xpc3QiLCAib3JnYW5pemF0aW9uX25hbWUiOiAiVEEgZXhhbXBsZSIsICJvcmdhbml6YXRpb25fdXJpIjogImh0dHBzOi8vdHJ1c3QtYW5jaG9yLmV4YW1wbGUub3JnL2hvbWUiLCAicG9saWN5X3VyaSI6ICJodHRwczovL3RydXN0LWFuY2hvci5leGFtcGxlLm9yZy9wb2xpY3kiLCAibG9nb191cmkiOiAiaHR0cHM6Ly90cnVzdC1hbmNob3IuZXhhbXBsZS5vcmcvc3RhdGljL2xvZ28uc3ZnIiwgImNvbnRhY3RzIjogWyJ0ZWNoQHRydXN0LWFuY2hvci5leGFtcGxlLm9yZyJdfX0sICJjb25zdHJhaW50cyI6IHsibWF4X3BhdGhfbGVuZ3RoIjogMX0sICJqd2tzIjogeyJrZXlzIjogW3sia3R5IjogIlJTQSIsICJraWQiOiAiTjFsNWVWWTJhMmszU0ZCSGNtcFRjMVJJYkZGM1R6VmlZbkp2VTBNNGJuaE1ZMXBxTkdod01VcEZTUSIsICJuIjogIm9pcm5UMEVjMkNva2JPZFJQT0VlbTI0SXc5LXhUeW5DcGwtOWx0cjBQclNUTEJXbHp6VW1IbzNBS25qemttNUg4MmxudVRMVTd1Skd3a0tqMjJaeklCMmFiMDIzendEX09rdDRaOXFyeW1kZUFYc0MtYXFhcVQ5RjZjZjFsbUJVczBvQXlwdWlTSnh1bU5WSVVucF9wbWhvbjlfR3NHUnlYcnJnT3lJclNKZ2hoX3p4RXREb3BnN0NQb3Q3Mmk3QlRpd0U1cXp0MFhGOHBnczM2VndYM0F3WnNMVkhpaGo0WVNUYXlYUUV3aUszMDUxRnhXXzVCbWNIRXJxM0J5WkU0UmZOSjBzbU96Z1NwMEtET0dkVjUwNHR5cTZJS1BjM04ybkhJd3hjUl9NbmczaWlfVWlFVXFYVEkwYkk5TlY4VHV4VWtrUldRcFZab0w5T3V2am9NUSIsICJlIjogIkFRQUIifV19LCAic3ViIjogImh0dHBzOi8vdHJ1c3QtYW5jaG9yLmV4YW1wbGUub3JnIiwgImlzcyI6ICJodHRwczovL3RydXN0LWFuY2hvci5leGFtcGxlLm9yZyIsICJpYXQiOiAxNzU4NTI3ODE4LCAiZXhwIjogMTc1ODgyNzgxOH0.lYDvz35LPE0V6nVUfO6Qwe8-AVQ9HBBeup-hg79HPAV-PqGa9dI9YX7GMiwb_khVGL8ASLfru4pFjlk4kjvgWj6u-ZXmWeElo48rAubCTtHKU8XOkHhvPXp04havTUevTCs3J0rBZKiJBLCBS9046yTrLPP8yN2hh_wR_3YExfW9O6w5tuEkghP4pQHZbWVZjE2pXTd-iIL5OFOWF_3050bAMTUQP_XUH5ZAPlj2-_qyDmARM9sh83SMDFwWkSqsDQDrOpWs1Uy7uT_iwqgPBuMSwUl9s4iRV-_CJy3IOdP0DCJOM3IyTWoHSlcMotEKGLmf4zTSnABr9PCPc5Bgrg&quot;
  ]
}
</code></pre>
</div>
<figcaption><a href="#figure-6" class="selfRef">Figure 6</a>: <a href="#name-example-jws-header-with-a-t" class="selfRef">Example JWS Header with a <code>trust_chain</code> Parameter</a></figcaption>
</figure>







## Metadata

This section defines how to represent and use metadata about Entities. It uses existing OpenID Connect and OAuth 2.0 metadata standards that are applicable to each Entity Type.

As described in [Section 3](#entity-statement){.auto .internal .xref}, Entity metadata is located in the `metadata` claim of an Entity Statement, whose value is a JSON object. The member names of this object are Entity Type Identifiers, as specified in [Section 5.1](#entity_types){.auto .internal .xref}. The metadata data structure following each Entity Type Identifier is a JSON object; it MAY be the empty JSON object `{}`, which may be the case when Superiors supply any needed metadata values.

Top-level JSON object members in the metadata data structures MAY use any JSON value other than `null`; the use of `null` is prohibited to prevent likely implementation errors caused by confusing members having `null` values with omitted members.



### Entity Type Identifiers

The Entity Type Identifier uniquely identifies the Entity Type of a federation participant and the metadata format for that Entity Type. This section defines a `federation_entity` Entity Type Identifier as well as identifiers for OpenID Connect and OAuth 2.0 Federation Entities.

Additional Entity Type Identifiers MAY be defined to support use cases outside OpenID Connect and OAuth 2.0 federations.



#### Federation Entity

The Entity Type Identifier is `federation_entity`.

The Entities that contain any of Federation Entity properties defined below MUST use this Entity Type. The following Federation Entity properties are defined:

[]{.break}

federation_fetch_endpoint
:   OPTIONAL. The fetch endpoint described in [Section 8.1](#fetch_endpoint){.auto .internal .xref}. This URL MUST use the `https` scheme and MAY contain port, path, and query parameter components; it MUST NOT contain a fragment component. Intermediate Entities and Trust Anchors MUST publish a `federation_fetch_endpoint`. Leaf Entities MUST NOT.
:   

federation_list_endpoint
:   OPTIONAL. The list endpoint described in [Section 8.2](#entity_listing){.auto .internal .xref}. This URL MUST use the `https` scheme and MAY contain port, path, and query parameter components; it MUST NOT contain a fragment component. Intermediate Entities and Trust Anchors MUST publish a `federation_list_endpoint`. Leaf Entities MUST NOT.
:   

federation_resolve_endpoint
:   OPTIONAL. The resolve endpoint described in [Section 8.3](#resolve){.auto .internal .xref}. This URL MUST use the `https` scheme and MAY contain port, path, and query parameter components; it MUST NOT contain a fragment component. Any federation Entity MAY publish a `federation_resolve_endpoint`.
:   

federation_trust_mark_status_endpoint
:   OPTIONAL. The Trust Mark Status endpoint described in [Section 8.4](#status_endpoint){.auto .internal .xref}. Trust Mark Issuers SHOULD publish a `federation_trust_mark_status_endpoint`. This URL MUST use the `https` scheme and MAY contain port, path, and query parameter components; it MUST NOT contain a fragment component.
:   

federation_trust_mark_list_endpoint
:   OPTIONAL. The endpoint described in [Section 8.5](#tm_listing){.auto .internal .xref}. This URL MUST use the `https` scheme and MAY contain port, path, and query parameter components; it MUST NOT contain a fragment component. Trust Mark Issuers MAY publish a `federation_trust_mark_list_endpoint`.
:   

federation_trust_mark_endpoint
:   OPTIONAL. The endpoint described in [Section 8.6](#tm_endpoint){.auto .internal .xref}. This URL MUST use the `https` scheme and MAY contain port, path, and query parameter components; it MUST NOT contain a fragment component. Trust Mark Issuers MAY publish a `federation_trust_mark_endpoint`.
:   

federation_historical_keys_endpoint
:   OPTIONAL. The endpoint described in [Section 8.7](#historical_keys){.auto .internal .xref}. This URL MUST use the `https` scheme and MAY contain port, path, and query parameter components; it MUST NOT contain a fragment component. All Federation Entities MAY publish a `federation_historical_keys_endpoint`.
:   

endpoint_auth_signing_alg_values_supported
:   OPTIONAL. JSON array containing a list of the supported [JWS](#RFC7515){.internal .xref} [[RFC7515](#RFC7515){.cite .xref}] algorithms (`alg` values) for signing the [JWT](#RFC7519){.internal .xref} [[RFC7519](#RFC7519){.cite .xref}] used for `private_key_jwt` when authenticating to federation endpoints, as described in [Section 8.8](#ClientAuthentication){.auto .internal .xref}. No default algorithms are implied if this entry is omitted. Servers SHOULD support `RS256`. The value `none` MUST NOT be used.
:   

Additional Federation Entity properties MAY be defined and used.

It is RECOMMENDED that each Federation Entity contain an `organization_name` claim, as defined in [Section 5.2.2](#informational_metadata){.auto .internal .xref}.

The following is a non-normative example of the `federation_entity` Entity Type:

[]{#name-example-of-federation_entit}

<figure id="figure-7">
<div id="section-5.1.1-7.1" class="alignLeft art-text artwork">
<pre><code>&quot;federation_entity&quot;: {
  &quot;federation_fetch_endpoint&quot;:
    &quot;https://amanita.caesarea.example.com/federation_fetch&quot;,
  &quot;federation_list_endpoint&quot;:
    &quot;https://amanita.caesarea.example.com/federation_list&quot;,
  &quot;federation_trust_mark_status_endpoint&quot;: &quot;https://amanita.caesarea.example.com/status&quot;,
  &quot;federation_trust_mark_list_endpoint&quot;: &quot;https://amanita.caesarea.example.com/trust_marked_list&quot;,
  &quot;organization_name&quot;: &quot;Ovulo Mushroom&quot;,
  &quot;organization_uri&quot;: &quot;https://amanita.caesarea.example.com&quot;
}</code></pre>
</div>
<figcaption><a href="#figure-7" class="selfRef">Figure 7</a>: <a href="#name-example-of-federation_entit" class="selfRef">Example of <code>federation_entity</code> Entity Type</a></figcaption>
</figure>





#### OpenID Connect Relying Party

The Entity Type Identifier is `openid_relying_party`.

All parameters defined in Section 2 of [OpenID Connect Dynamic Client Registration 1.0](#OpenID.Registration){.internal .xref} [[OpenID.Registration](#OpenID.Registration){.cite .xref}], in [[OpenID.RP.Choices](#OpenID.RP.Choices){.cite .xref}], and in [Section 5.2](#common_metadata){.auto .internal .xref} are applicable, as well as additional parameters registered in the IANA "OAuth Dynamic Client Registration Metadata" registry [[IANA.OAuth.Parameters](#IANA.OAuth.Parameters){.cite .xref}].

In addition, the following RP metadata parameter is defined:

[]{.break}

client_registration_types
:   RECOMMENDED. An array of strings specifying the client registration types the RP supports. Values defined by this specification are `automatic` and `explicit`. Additional values MAY be defined and used, without restriction by this specification.
:   

The following is a non-normative example of the JWT Claims Set for an RP's Entity Configuration:

[]{#name-example-relying-party-entit}

<figure id="figure-8">
<div id="section-5.1.2-6.1" class="alignLeft art-text artwork">
<pre><code>    {
      &quot;iss&quot;: &quot;https://openid.sunet.se&quot;,
      &quot;sub&quot;: &quot;https://openid.sunet.se&quot;,
      &quot;iat&quot;: 1516239022,
      &quot;exp&quot;: 1516298022,
      &quot;metadata&quot;: {
        &quot;openid_relying_party&quot;: {
          &quot;application_type&quot;: &quot;web&quot;,
          &quot;redirect_uris&quot;: [
            &quot;https://openid.sunet.se/rp/callback&quot;
          ],
          &quot;organization_name&quot;: &quot;SUNET&quot;,
          &quot;logo_uri&quot;: &quot;https://www.sunet.se/sunet/images/32x32.png&quot;,
          &quot;grant_types&quot;: [
            &quot;authorization_code&quot;,
            &quot;implicit&quot;
          ],
          &quot;signed_jwks_uri&quot;:&quot;https://openid.sunet.se/rp/signed_jwks.jose&quot;,
          &quot;jwks_uri&quot;: &quot;https://openid.sunet.se/rp/jwks.json&quot;,
          &quot;client_registration_types&quot;: [&quot;automatic&quot;]
        }
      },
      &quot;jwks&quot;: {
        &quot;keys&quot;: [
          {
            &quot;alg&quot;: &quot;RS256&quot;,
            &quot;e&quot;: &quot;AQAB&quot;,
            &quot;kid&quot;: &quot;key1&quot;,
            &quot;kty&quot;: &quot;RSA&quot;,
            &quot;n&quot;: &quot;pnXBOusEANuug6ewezb9J_...&quot;,
            &quot;use&quot;: &quot;sig&quot;
          }
        ]
      },
      &quot;authority_hints&quot;: [
        &quot;https://edugain.org/federation&quot;
      ]
    }
</code></pre>
</div>
<figcaption><a href="#figure-8" class="selfRef">Figure 8</a>: <a href="#name-example-relying-party-entit" class="selfRef">Example Relying Party Entity Configuration JWT Claims Set</a></figcaption>
</figure>





#### OpenID Connect OpenID Provider

The Entity Type Identifier is `openid_provider`.

All parameters defined in Section 3 of [OpenID Connect Discovery 1.0](#OpenID.Discovery){.internal .xref} [[OpenID.Discovery](#OpenID.Discovery){.cite .xref}] and [Section 5.2](#common_metadata){.auto .internal .xref} are applicable, as well as additional parameters registered in the IANA "OAuth Authorization Server Metadata" registry [[IANA.OAuth.Parameters](#IANA.OAuth.Parameters){.cite .xref}]. For instance, the `require_signed_request_object` and `require_pushed_authorization_requests` metadata parameters can be used.

The `issuer` parameter value in the `openid_provider` metadata MUST match the Federation Entity identifier (the `iss` parameter within the Entity Configuration).

In addition, the following OP metadata parameters are defined:

[]{.break}

client_registration_types_supported
:   RECOMMENDED. An array of strings specifying the client registration types the OP supports. Values defined by this specification are `automatic` and `explicit`. Additional values MAY be defined and used, without restriction by this specification.
:   

federation_registration_endpoint
:   OPTIONAL. URL of the OP's federation-specific Dynamic Client Registration Endpoint. If the OP supports Explicit Client Registration Endpoint this URL MUST use the `https` scheme and MAY contain port, path, and query parameter components; it MUST NOT contain a fragment component. If the OP supports Explicit Client Registration as described in [Section 12.2](#explicit){.auto .internal .xref}, then this claim is REQUIRED.
:   

The following is a non-normative example of the JWT Claims Set for an OP's Entity Configuration:

[]{#name-example-openid-provider-ent}

<figure id="figure-9">
<div id="section-5.1.3-7.1" class="alignLeft art-text artwork">
<pre><code>{
   &quot;iss&quot;:&quot;https://op.umu.se&quot;,
   &quot;sub&quot;:&quot;https://op.umu.se&quot;,
   &quot;exp&quot;:1568397247,
   &quot;iat&quot;:1568310847,
   &quot;metadata&quot;:{
      &quot;openid_provider&quot;:{
         &quot;issuer&quot;:&quot;https://op.umu.se&quot;,
         &quot;signed_jwks_uri&quot;:&quot;https://op.umu.se/openid/signed_jwks.jose&quot;,
         &quot;authorization_endpoint&quot;:&quot;https://op.umu.se/openid/authorization&quot;,
         &quot;client_registration_types_supported&quot;:[
            &quot;automatic&quot;,
            &quot;explicit&quot;
         ],
         &quot;grant_types_supported&quot;:[
            &quot;authorization_code&quot;,
            &quot;implicit&quot;,
            &quot;urn:ietf:params:oauth:grant-type:jwt-bearer&quot;
         ],
         &quot;id_token_signing_alg_values_supported&quot;:[
            &quot;ES256&quot;,
            &quot;RS256&quot;
         ],
         &quot;logo_uri&quot;:&quot;https://www.umu.se/img/umu-logo-left-neg-SE.svg&quot;,
         &quot;op_policy_uri&quot;:&quot;https://www.umu.se/en/legal-information/&quot;,
         &quot;response_types_supported&quot;:[
            &quot;code&quot;,
            &quot;code id_token&quot;,
            &quot;token&quot;
         ],
         &quot;subject_types_supported&quot;:[
            &quot;pairwise&quot;,
            &quot;public&quot;
         ],
         &quot;token_endpoint&quot;:&quot;https://op.umu.se/openid/token&quot;,
         &quot;federation_registration_endpoint&quot;:&quot;https://op.umu.se/openid/fedreg&quot;,
         &quot;token_endpoint_auth_methods_supported&quot;:[
            &quot;client_secret_post&quot;,
            &quot;client_secret_basic&quot;,
            &quot;client_secret_jwt&quot;,
            &quot;private_key_jwt&quot;
         ],
         &quot;pushed_authorization_request_endpoint&quot;:&quot;https://op.umu.se/openid/par&quot;,
         &quot;request_object_signing_alg_values_supported&quot;: [
            &quot;ES256&quot;,
            &quot;RS256&quot;
         ],
         &quot;token_endpoint_auth_signing_alg_values_supported&quot;: [
            &quot;ES256&quot;,
            &quot;RS256&quot;
         ]
      }
   },
   &quot;authority_hints&quot;:[
      &quot;https://umu.se&quot;
   ],
   &quot;jwks&quot;:{
      &quot;keys&quot;:[
         {
            &quot;e&quot;:&quot;AQAB&quot;,
            &quot;kid&quot;:&quot;dEEtRjlzY3djcENuT01wOGxrZlkxb3RIQVJlMTY0...&quot;,
            &quot;kty&quot;:&quot;RSA&quot;,
            &quot;n&quot;:&quot;x97YKqc9Cs-DNtFrQ7_vhXoH9bwkDWW6En2jJ044yH...&quot;
         }
      ]
   }
}</code></pre>
</div>
<figcaption><a href="#figure-9" class="selfRef">Figure 9</a>: <a href="#name-example-openid-provider-ent" class="selfRef">Example OpenID Provider Entity Configuration JWT Claims Set</a></figcaption>
</figure>





#### OAuth Authorization Server

The Entity Type Identifier is `oauth_authorization_server`.

All parameters defined in Section 2 of [[RFC8414](#RFC8414){.cite .xref}] and [Section 5.2](#common_metadata){.auto .internal .xref} are applicable, as well as additional parameters registered in the IANA "OAuth Authorization Server Metadata" registry [[IANA.OAuth.Parameters](#IANA.OAuth.Parameters){.cite .xref}].

The `issuer` parameter value in the `oauth_authorization_server` metadata MUST match the Federation Entity identifier (the `iss` claim in the Entity Configuration).





#### OAuth Client

The Entity Type Identifier is `oauth_client`.

All parameters defined in Section 2 of [OAuth 2.0 Dynamic Client Registration Protocol](#RFC7591){.internal .xref} [[RFC7591](#RFC7591){.cite .xref}] and [Section 5.2](#common_metadata){.auto .internal .xref} are applicable, as well as additional parameters registered in the IANA "OAuth Dynamic Client Registration Metadata" registry [[IANA.OAuth.Parameters](#IANA.OAuth.Parameters){.cite .xref}].





#### OAuth Protected Resource

The Entity Type Identifier is `oauth_resource`. The parameters defined in [Section 5.2](#common_metadata){.auto .internal .xref} are applicable. In addition, deployments MAY use the protected resource metadata parameters defined in [[RFC9728](#RFC9728){.cite .xref}].







### Common Metadata Parameters

This section defines additional metadata parameters that MAY be used with all the Entity Types above, with the exception for JWK Sets noted below.



#### Extensions for JWK Sets in Entity Metadata

The following metadata parameters define ways of obtaining JWK Sets for an Entity Type of the Entity. Note that these keys are distinct from the Federation Entity Keys used to sign Entity Statements, which are in the `jwks` claim of the Entity Statement, and not within the `metadata` claim. These extensions for JWK Sets MUST NOT be used in `federation_entity` Entity Type metadata.

[]{.break}

signed_jwks_uri
:   OPTIONAL. URL referencing a signed JWT having the Entity's JWK Set document for that Entity Type as its payload. This URL MUST use the `https` scheme. The JWT MUST be signed using a Federation Entity Key. A successful response from the URL MUST use the HTTP status code 200 with the content type `application/jwk-set+jwt`. When both signing and encryption keys are present, a `use` (public key use) parameter value is REQUIRED for all keys in the referenced JWK Set to indicate each key's intended usage.
:   

:   Signed JWK Set JWTs are explicitly typed by setting the `typ` header parameter to `jwk-set+jwt` to prevent cross-JWT confusion, per Section 3.11 of [[RFC8725](#RFC8725){.cite .xref}]. Signed JWK Set JWTs without a `typ` header parameter or with a different `typ` value MUST be rejected.
:   

:   Signed JWK Set JWTs MUST include the `kid` (Key ID) header parameter with its value being the Key ID of the signing key used.
:   

:   The following claims are specified for use in the payload, all of which except `keys` are defined in [[RFC7519](#RFC7519){.cite .xref}]:

    []{.break}

    keys
    :   REQUIRED. Array of JWK values in the JWK Set, as specified in Section 5.1 of [[RFC7517](#RFC7517){.cite .xref}].
    :   

    iss
    :   REQUIRED. The "iss" (issuer) claim identifies the principal that issued the JWT.
    :   

    sub
    :   REQUIRED. This claim identifies the owner of the keys. It SHOULD be the same as the issuer.
    :   

    iat
    :   OPTIONAL. Number. Time when this signed JWK Set was issued. This is expressed as Seconds Since the Epoch, per [[RFC7519](#RFC7519){.cite .xref}].
    :   

    exp
    :   OPTIONAL. Number. This claim identifies the time when the JWT is no longer valid. This is expressed as Seconds Since the Epoch, per [[RFC7519](#RFC7519){.cite .xref}].
    :   

    More claims are defined in [[RFC7519](#RFC7519){.cite .xref}]; of these, `aud` SHOULD NOT be used since the issuer cannot know who the audience is. `nbf` and `jti` are not particularly useful in this context and SHOULD be omitted.

:   

:   Additional claims MAY be defined and used in conjunction with the claims above.

    The following is a non-normative example of the JWT Claims Set for a signed JWK Set.

    []{#name-example-jwt-claims-set-for-}

    <figure id="figure-10">
    <div id="section-5.2.1-2.10.3.1" class="alignLeft art-text artwork">
    <pre><code>    {
          &quot;keys&quot;: [
            {
              &quot;kty&quot;: &quot;RSA&quot;,
              &quot;kid&quot;: &quot;SUdtUndEWVY2cUFDeDV5NVlBWDhvOXJodVl2am1mNGNtR0pmd&quot;,
              &quot;n&quot;: &quot;y_Zc8rByfeRIC9fFZrDZ2MGH2ZnxLrc0ZNNwkNet5rwCPYeRF3Sv
                    5nihZA9NHkDTEX97dN8hG6ACfeSo6JB2P7heJtmzM8oOBZbmQ90n
                    EA_JCHszkejHaOtDDfxPH6bQLrMlItF4JSUKua301uLB7C8nzTxm
                    tF3eAhGCKn8LotEseccxsmzApKRNWhfKDLpKPe9i9PZQhhJaurwD
                    kMwbWTAeZbqCScU1o09piuK1JDf2PaDFevioHncZcQO74Obe4nN3
                    oNPNAxrMClkZ9s9GMEd5vMqOD4huXlRpHwm9V3oJ3LRutOTxqQLV
                    yPucu7eHA7her4FOFAiUk-5SieXL9Q&quot;,
              &quot;e&quot;: &quot;AQAB&quot;
            },
            {
              &quot;kty&quot;: &quot;EC&quot;,
              &quot;kid&quot;: &quot;MFYycG1raTI4SkZvVDBIMF9CNGw3VEZYUmxQLVN2T21nSWlkd3&quot;,
              &quot;crv&quot;: &quot;P-256&quot;,
              &quot;x&quot;: &quot;qAOdPQROkHfZY1daGofOmSNQWpYK8c9G2m2Rbkpbd4c&quot;,
              &quot;y&quot;: &quot;G_7fF-T8n2vONKM15Mzj4KR_shvHBxKGjMosF6FdoPY&quot;
            }
          ],
          &quot;iss&quot;: &quot;https://example.org/op&quot;,
          &quot;sub&quot;: &quot;https://example.org/op&quot;,
          &quot;iat&quot;: 1618410883
        }
    </code></pre>
    </div>
    <figcaption><a href="#figure-10" class="selfRef">Figure 10</a>: <a href="#name-example-jwt-claims-set-for-" class="selfRef">Example JWT Claims Set for a Signed JWK Set</a></figcaption>
    </figure>

:   

jwks_uri
:   OPTIONAL. URL referencing a JWK Set document containing the Entity's keys for that Entity Type. This URL MUST use the `https` scheme. When both signing and encryption keys are present, a `use` (public key use) parameter value is REQUIRED for all keys in the referenced JWK Set to indicate each key's intended usage.
:   

jwks
:   OPTIONAL. JSON Web Key Set document, passed by value, containing the Entity's keys for that Entity Type. When both signing and encryption keys are present, a `use` (public key use) parameter value is REQUIRED for all keys in the JWK Set to indicate each key's intended usage. This parameter is intended to be used by participants that, for some reason, cannot use the `signed_jwks_uri` parameter. An upside of using `jwks` is that the Entity's keys for the Entity Type are recorded in Trust Chains.
:   



##### Usage of jwks, jwks_uri, and signed_jwks_uri in Entity Metadata

It is RECOMMENDED that an Entity Configuration use only one of `jwks`, `jwks_uri`, and `signed_jwks_uri` in its OpenID Connect or OAuth 2.0 metadata. However, there may be circumstances in which it is desirable to use multiple JWK Set representations, such as when an Entity is in multiple federations and the federations have different policies about the JWK Set representation to be used. Also note that some implementations might not understand all these representations. For instance, while `jwks_uri` will certainly be understood in OpenID Connect OP metadata, `signed_jwks_uri` might not be understood by all OpenID Connect implementations, and so a JWK Set representation that is understood also needs to be present.

When multiple JWK Set representations are used, the keys present in each representation SHOULD be the same. Even if they are not completely the same at a given instant in time (which may be the case during key rollover operations), implementations MUST make them consistent in a timely manner.







#### Informational Metadata Extensions

The following metadata parameters define ways of obtaining information about the Entity for an Entity Type.

[]{.break}

organization_name
:   OPTIONAL. A human-readable name representing the organization owning this Entity. If the owner is a physical person, this MAY be, for example, the person's name. Note that this information will be publicly available.
:   

display_name
:   OPTIONAL. A human-readable name of the Entity to be presented to the End-User.
:   

description
:   OPTIONAL. A human-readable brief description of this Entity presentable to the End-User.
:   

keywords
:   OPTIONAL. JSON array with one or more strings representing search keywords, tags, categories, or labels that apply to this Entity.
:   

contacts
:   OPTIONAL. JSON array with one or more strings representing contact persons at the Entity. These MAY contain names, e-mail addresses, descriptions, phone numbers, etc.
:   

logo_uri
:   OPTIONAL. String. A URL that points to the logo of this Entity. The file containing the logo SHOULD be published in a format that can be viewed via the web.
:   

policy_uri
:   OPTIONAL. URL of the documentation of conditions and policies relevant to this Entity.
:   

information_uri
:   OPTIONAL. URL for documentation of additional information about this Entity viewable by the End-User.
:   

organization_uri
:   OPTIONAL. URL of a Web page for the organization owning this Entity.
:   

These metadata parameters MAY be present in the Entity's metadata for any Entity Types that it uses.









## Federation Policy



### Metadata Policy

Trust Anchors and Intermediate Entities MAY define policies that apply to the metadata of their Subordinates.

A federation may utilize metadata policies to achieve specific objectives. For example, in a federation of [OpenID Connect](#OpenID.Core){.internal .xref} [[OpenID.Core](#OpenID.Core){.cite .xref}] Entities, one objective may be to ensure the metadata published by OpenID Providers and Relying Parties are interoperable with one another. Another objective may be to ensure the Entity metadata complies with a security profile, for example, [FAPI](#FAPI){.internal .xref} [[FAPI](#FAPI){.cite .xref}].

Note that the `metadata_policy` is not intended to check and validate the JSON value types of metadata parameters. Such checks SHOULD be performed at the application layer, after obtaining the Entity metadata from a successfully resolved Trust Chain.



#### Principles

OpenID Federation enables the definition of metadata policies with the following properties:

[]{.break}

Hierarchy

:   Once applied to a metadata parameter, a metadata policy cannot be repealed or made more permissive by Intermediate Entities that are subordinate in the Trust Chain.

    The hierarchy of policies is preserved in nested federations where a Trust Anchor in one federation acts as an Intermediate Entity in another federation.

:   

Equal Opportunity
:   All Superior Entities in a Trust Chain can contribute metadata policies on an equal basis, provided their contributions result in a combined metadata policy that is logically sound. For instance, any Intermediate can further restrict the metadata of its Subordinates relative to what its Superiors specified. An Intermediate that introduces a conflict among the metadata policies causes the Trust Chain to be deemed invalid.
:   

Specificity and Granularity

:   Just like metadata, a metadata policy is bound to a specific Entity Type. This ensures policies for different Entity Types are independent and isolated from one another.

    Policies are expressed at the level of individual metadata parameters. The policies for a given Entity Type metadata parameter are thus independent and isolated from those for other parameters.

    When a Trust Anchor or an Intermediate Entity defines a metadata policy for an Entity Type, it applies to the metadata of all Subordinate Entities of that type in the Trust Chain.

    Because the place to define a policy is the Subordinate Statement and every Entity Statement is issued for a specific subject, a federation authority can choose to define a common Entity Type metadata policy for all its Subordinates, or specific Entity Type metadata policies for specific Subordinates.

:   

Operation
:   A policy operates by performing a check, a modification, or both on a given metadata parameter. This specification defines a set of standard operators, described in [Section 6.1.3.1](#standard_metadata_policy_operators){.auto .internal .xref}. A federation MAY specify and use additional operators, provided they comply with the principles laid out in this section and with [Section 6.1.3](#metadata_policy_operators){.auto .internal .xref} and [Section 6.1.3.2](#additional_metadata_policy_operators){.auto .internal .xref}.
:   

Integral Metadata Enforcement

:   The resolution and application of metadata policies is an integral part of the Trust Chain resolution process, as described in [Section 10](#resolving_trust){.auto .internal .xref}.

    This means:

    - 
      A Trust Chain for which the metadata policy resolution fails due to an error, for example, due to an Intermediate Entity's policy conflicting with a Superior's policy, is deemed invalid.
      

    - 
      A Trust Chain with Entity metadata that does not comply with the resolved metadata policies is deemed invalid.
      

:   

Determinism
:   The resolution and application of metadata policies in a Trust Chain is deterministic. Trust Anchors and Intermediate Entities are thus able to formulate policies that exhibit predictable and reproducible outcomes.
:   





#### Structure

Metadata policies are expressed in the `metadata_policy` claim of a Subordinate Statement, as described in [Section 3](#entity-statement){.auto .internal .xref}. The claim value is a JSON object that has a data structure consisting of three levels:

1.  
    Metadata policies for Entity Types.

    The top level contains one or more members, each representing the metadata policy for an Entity Type. Each member name is an Entity Type Identifier, as specified in [Section 5.1](#entity_types){.auto .internal .xref}, for example, `openid_relying_party`. The member value is a JSON object that contains metadata parameter policies.
    

2.  
    Metadata parameter policies for the Entity Type.

    The second level contains one or more members, each representing a policy for a metadata parameter for the Entity Type. Each member name is a metadata parameter name, for example `id_token_signed_response_alg`. The name MAY include a language tag, as described in [Section 14](#ClaimsLanguagesAndScripts){.auto .internal .xref}, in which case the metadata parameter policy applies only to the metadata parameter with the specified language tag. The member value is a JSON object that contains policy operators.
    

3.  
    Operators for the metadata parameter policy for the Entity Type.

    The third level contains one or more members, each representing an operator that checks or modifies the metadata parameter, as described in [Section 6.1.3](#metadata_policy_operators){.auto .internal .xref}. Only operators that are allowed to be combined with one another MUST be included, as described in the specification for each operator.
    

Duplicate JSON object member names MUST NOT be present at any of the three levels of the `metadata_policy` claim data structure.

The following is a non-normative example of a metadata policy for an OpenID Relying Party that consists of a single policy for the `id_token_signed_response_alg` metadata parameter and uses two operators, `default` and `one_of`:

[]{#name-example-metadata_policy-cla}

<figure id="figure-11">
<div id="section-6.1.2-5.1" class="alignLeft art-text artwork">
<pre><code>&quot;metadata_policy&quot; : {
  &quot;openid_relying_party&quot;: {
    &quot;id_token_signed_response_alg&quot;: {
      &quot;default&quot;: &quot;ES256&quot;,
      &quot;one_of&quot;: [&quot;ES256&quot;, &quot;ES384&quot;, &quot;ES512&quot;]
    }
  }
}</code></pre>
</div>
<figcaption><a href="#figure-11" class="selfRef">Figure 11</a>: <a href="#name-example-metadata_policy-cla" class="selfRef">Example <code>metadata_policy</code> Claim</a></figcaption>
</figure>





#### Operators

A metadata policy operator:

- 
  Is identified by a unique case-sensitive name.
  

- 
  Acts on a single metadata parameter. The operator definition MUST specify the metadata parameter JSON value types that are mandatory to support, and MAY also specify JSON value types that are optional to support. When the metadata parameter has a JSON value type that is not supported, the operator MUST produce a policy error.
  

- 
  The action on the metadata parameter can be a value check, a value modification, or both. When the operator's action is a value modification, it MAY remove the metadata parameter.
  

- 
  The action of the operator is configured by a JSON value. The operator definition MUST specify the JSON value types that are mandatory to support, and MAY also specify JSON value types that are optional to support. When the operator is configured with a JSON value type that is not supported, the operator MUST produce a policy error.
  

- 
  MUST declare what other operators it may be combined with, which applies to both individual as well as merged metadata parameter policies, as described in [Section 6.1.2](#metadata_policy_structure){.auto .internal .xref} and [Section 6.1.4](#metadata_policy_enforcement){.auto .internal .xref}. A combination may be unconditional, or conditional, requiring the configured values of the two operators to meet certain criteria. Combinations that are not allowed MUST produce a policy error.
  

- 
  MUST declare in what order it is to be applied to a metadata parameter, absolute or relative to other operators in the metadata parameter policy. Value check operators SHOULD generally be applied after operators that perform value modifications.
  

- 
  MUST specify, when more than one Subordinate Statement in a Trust Chain has a policy for an Entity Type metadata parameter that uses the same operator, whether the operator values are allowed to be merged to produce an identical or more restrictive policy, and if so, under what conditions. The order of the result of such an operator value merge is not defined. If the operator does not allow such a merge, it MUST produce a policy error.
  

- 
  An operator MUST NOT output a metadata parameter with the `null` value.
  

- 
  Metadata parameters and policies that conform to the JSON grammar but do not represent interoperable uses of JSON, as per Sections 4 and 8 of [[RFC8259](#RFC8259){.cite .xref}], can cause unpredictable behavior.
  



##### Standard Operators

This specification defines the following metadata policy operators:



###### value

Name: `value`

Action: The metadata parameter MUST be assigned the value of the operator. When the value of the operator is `null`, the metadata parameter MUST be removed.

Metadata parameter JSON values:

- 
  Mandatory to support: string, number, boolean, array
  

Operator JSON values:

- 
  Mandatory to support: string, number, boolean, array, null
  

Combination with other operators in a metadata parameter policy:

- 
  MAY be combined with `add`, in which case the values of `add` MUST be a subset of the values of `value`.
  

- 
  MAY be combined with `default` if the value of `value` is not `null`.
  

- 
  MAY be combined with `one_of`, in which case the value of `value` MUST be among the `one_of` values.
  

- 
  MAY be combined with `subset_of`, in which case the values of `value` MUST be a subset of the values of `subset_of`.
  

- 
  MAY be combined with `superset_of`, in which case the values of `value` MUST be a superset of the values of `superset_of`.
  

- 
  MAY be combined with `essential`, except when `value` is `null` and `essential` is true.
  

Order of application: First

Operator value merge: Allowed only when the operator values are equal. If not, this MUST produce a policy error.





###### add

Name: `add`

Action: The value or values of this operator MUST be added to the metadata parameter. Values that are already present in the metadata parameter MUST NOT be added another time. If the metadata parameter is absent, it MUST be initialized with the value of this operator.

Metadata parameter JSON values:

- 
  Mandatory to support: array of strings
  

- 
  Optional to support: array of objects, array of numbers
  

Operator JSON values:

- 
  Mandatory to support: array of strings
  

- 
  Optional to support: array of objects, array of numbers
  

Combination with other operators in a metadata parameter policy:

- 
  MAY be combined with `value`, in which case the values of `add` MUST be a subset of the values of `value`.
  

- 
  MAY be combined with `default`.
  

- 
  MAY be combined with `subset_of`, in which case the values of `add` MUST be a subset of the values of `subset_of`.
  

- 
  MAY be combined with `superset_of`.
  

- 
  MAY be combined with `essential`.
  

Order of application: After `value`.

Operator value merge: The result of merging the values of two `add` operators is the union of the values.





###### default

Name: `default`

Action: If the metadata parameter is absent, it MUST be set to the value of the operator. If the metadata parameter is present, this operator has no effect.

Metadata parameter JSON values:

- 
  Mandatory to support: string, number, boolean, array
  

Operator JSON values:

- 
  Mandatory to support: string, number, boolean, array
  

Combination with other operators in a metadata parameter policy:

- 
  MAY be combined with `value` if the value of `value` is not `null`.
  

- 
  MAY be combined with `add`.
  

- 
  MAY be combined with `one_of`.
  

- 
  MAY be combined with `subset_of`.
  

- 
  MAY be combined with `superset_of`.
  

- 
  MAY be combined with `essential`.
  

Order of application: After `add`.

Operator value merge: The operator values MUST be equal. If the values are not equal this MUST produce a policy error.





###### one_of

Name: `one_of`

Action: If the metadata parameter is present, its value MUST be one of those listed in the operator value.

Metadata parameter JSON values:

- 
  Mandatory to support: string
  

- 
  Optional to support: object, number
  

Operator JSON values:

- 
  Mandatory to support: array of strings
  

- 
  Optional to support: array of objects, array of numbers
  

Combination with other operators in a metadata parameter policy:

- 
  MAY be combined with `value`, in which case the value of `value` MUST be among the `one_of` values.
  

- 
  MAY be combined with `default`.
  

- 
  MAY be combined with `essential`.
  

Order of application: After `default`.

Operator value merge: The result of merging the values of two `one_of` operators is the intersection of the operator values. If the intersection is empty, this MUST result in a policy error.





###### subset_of

Name: `subset_of`

Action: If the metadata parameter is present, it is assigned the intersection between the values of the operator and the metadata parameter. Note that the resulting intersection may thus be an empty array `[]`. Also note that `subset_of` is a potential value modifier in addition to it being a value check.

Metadata parameter JSON values:

- 
  Mandatory to support: array of strings
  

- 
  Optional to support: array of objects, array of numbers
  

Operator JSON values:

- 
  Mandatory to support: array of strings
  

- 
  Optional to support: array of objects, array of numbers
  

Combination with other operators in a metadata parameter policy:

- 
  MAY be combined with `value`, in which case the values of `value` MUST be a subset of the values of `subset_of`.
  

- 
  MAY be combined with `add`, in which case the values of `add` MUST be a subset of the values of `subset_of`.
  

- 
  MAY be combined with `default`.
  

- 
  MAY be combined with `superset_of`, in which case the values of `subset_of` MUST be a superset of the values of `superset_of`.
  

- 
  MAY be combined with `essential`.
  

Order of application: After `one_of`.

Operator value merge: The result of merging the values of two `subset_of` operators is the intersection of the operator values. Note that the resulting intersection may thus be an empty array `[]`.





###### superset_of

Name: `superset_of`

Action: If the metadata parameter is present, its values MUST contain those specified in the operator value. By mathematically defining supersets, equality is included.

Metadata parameter JSON values:

- 
  Mandatory to support: array of strings
  

- 
  Optional to support: array of objects, array of numbers
  

Operator JSON values:

- 
  Mandatory to support: array of strings
  

- 
  Optional to support: array of objects, array of numbers
  

Combination with other operators in a metadata parameter policy:

- 
  MAY be combined with `value`, in which case the values of `value` MUST be a superset of the values of `superset_of`.
  

- 
  MAY be combined with `add`.
  

- 
  MAY be combined with `default`.
  

- 
  MAY be combined with `subset_of`, in which case the values of `subset_of` MUST be a superset of the values of `superset_of`.
  

- 
  MAY be combined with `essential`.
  

Order of application: After `subset_of`.

Operator value merge: The result of merging the values of two `superset_of` operators is the union of the operator values.





###### essential

Name: `essential`

Action: If the value of this operator is `true`, then the metadata parameter MUST be present. If `false`, the metadata parameter is voluntary and may be absent. If the `essential` operator is omitted, this is equivalent to including it with a value of `false`.

Metadata parameter JSON values:

- 
  Mandatory to support: string, number, boolean, object, array
  

Operator JSON values:

- 
  Mandatory to support: boolean
  

Combination with other operators in a metadata parameter policy:

- 
  MAY be combined with `value`, except when `value` is `null` and `essential` is true.
  

- 
  MAY be combined with any other operator.
  

Order of application: Last

Operator value merge: The result of merging the values of two `essential` operators is the logical disjunction (`OR`) of the operator values.





###### Notes on Operators

A "set equals" metadata parameter policy can be expressed by combining the operators `subset_of` and `superset_of` with identical array values.

Some JSON libraries may have issues comparing JSON objects. For this reason, support for applying metadata policy to metadata values that are JSON objects is mandatory only for the `essential` operator, which does not require comparison of values. It is OPTIONAL for the `add`, `one_of`, `subset_of`, and `superset_of` operators, which operate on JSON arrays and therefore require comparison of values. And it is OPTIONAL for the `value` and `default` operators, since merging values and defaults requires comparing the operators' values with existing ones.

The `scope` OAuth 2.0 client metadata parameter, defined in [[RFC7591](#RFC7591){.cite .xref}] and represented by a string of space-separated string values, is to be regarded and processed as a string array by policy operators, such as the operators `default`, `subset_of` and `superset_of`. The resulting `scope` metadata parameter is a space-separated string of individual scope values, where the scope values present are taken from the array of values produced by applying the metadata operators to the `scope` parameter.

The following table is a map of the outputs produced by combinations of the `essential` and `subset_of` policy operators with different input metadata parameter values. Note, the `subset_of` check is skipped when the metadata parameter is absent and designated as voluntary, as shown in the last row.

[]{#name-examples-of-outputs-with-co}

  Policy                  Metadata Parameter   
  ----------- ----------- -------------------- --------------
  essential   subset_of   input                output
  true        [a,b,c]   [a,e]              [a]
  false       [a,b,c]   [a,e]              [a]
  true        [a,b,c]   [d,e]              []
  false       [a,b,c]   [d,e]              []
  true        [a,b,c]   no parameter         error
  false       [a,b,c]   no parameter         no parameter

  : [Table 1](#table-1): [Examples of Outputs with Combinations of `essential` and `subset_of` for Different Inputs](#name-examples-of-outputs-with-co) {#table-1}







##### Additional Operators

Federations MAY specify and use additional metadata policy operators that conform with the principles in [Section 6.1.1](#metadata_policy_principles){.auto .internal .xref} and in [Section 6.1.3](#metadata_policy_operators){.auto .internal .xref}.

Additional operators MUST observe the following rules in regard to the order of their application relative to the standard operators defined in [Section 6.1.3.1](#standard_metadata_policy_operators){.auto .internal .xref}:

- 
  Additional operators that modify metadata parameters MUST be applied after the `value` operator that is specified in [Section 6.1.3.1.1](#value_operator){.auto .internal .xref}.
  

- 
  Additional operators that check metadata parameters MUST be applied before the `essential` operator that is specified in [Section 6.1.3.1.7](#essential_operator){.auto .internal .xref}.
  

Implementations MUST ignore additional operators that are not understood, unless the operator name is included in the `metadata_policy_crit` Subordinate Statement claim, in which case the operator MUST be understood and processed. If an additional operator listed in `metadata_policy_crit` is not understood or cannot be processed, then this MUST produce a policy error and the Trust Chain MUST be considered invalid.







#### Enforcement

This section describes the resolution of the metadata policy for a Trust Chain and its application to the metadata of the Federation Entity that is the Trust Chain subject.

If a policy error or another error is encountered during the metadata policy resolution or its application, the Trust Chain MUST be considered invalid.



##### Resolution

The metadata policy for a Trust Chain is determined by the sequence of the present `metadata_policy` claims of the Subordinate Statements that make up the chain.

The resolution process MUST first gather the names of all policy operators other than the standard ones defined in [Section 6.1.3.1](#standard_metadata_policy_operators){.auto .internal .xref} that are declared as critical. This is done by checking each Subordinate Statement in the Trust Chain for the optional `metadata_policy_crit` claim, described in [Section 3](#entity-statement){.auto .internal .xref}, and collecting any operator names that are found in it.

The resolution process proceeds by iterating through the Subordinate Statements. The sequence of this iteration is crucial - it MUST begin with the Subordinate Statement issued by the most Superior Entity and end with the Subordinate Statement issued by the Immediate Superior of the Trust Chain subject.

An important task during the iteration is the `metadata_policy` validation. It MUST ensure the data structure is compliant and that every metadata parameter policy contains only allowed operator combinations, as described in [Section 6.1.3](#metadata_policy_operators){.auto .internal .xref}, and in accordance with the specifications of the operators. It MUST also be ensured that the `metadata_policy` contains no operators that cannot be understood and processed whose names are among the collected `metadata_policy_crit` values. An unsuccessful validation MUST produce a policy error.

At each iteration step, the Subordinate Statement MUST be checked for the presence of a `metadata_policy` claim. The first encountered `metadata_policy` MUST be validated as described above, after which it becomes the current metadata policy.

If the iteration yields a next subordinate `metadata_policy` claim, it MUST be validated as described above, then merged into the current metadata policy.

The merge is performed at each of the three levels of the `metadata_policy` data structure described in [Section 6.1.2](#metadata_policy_structure){.auto .internal .xref}, by starting from the top level:

1.  
    The metadata policies for Entity Types.
    

2.  
    The metadata parameter policies for an Entity Type.
    

3.  
    The operators for a metadata parameter policy for an Entity Type.
    

At the level of metadata policies for Entity Types, the merge proceeds as follows:

- 
  If the next subordinate `metadata_policy` claim contains a metadata policy for an Entity Type that is already present in the current metadata policy, it MUST be merged according to the rules of the next lower level (the metadata parameter policies).
  

- 
  Entity Type metadata policies in the next subordinate `metadata_policy` claim that are not present in the current metadata policy MUST be copied to it.
  

At the level of metadata parameter policies, the merge proceeds as follows:

- 
  If a metadata parameter policy is already present in the current Entity Type metadata policy, it MUST be merged according to the rules of the next lower level (the operators for the metadata parameter policy). If the resulting metadata parameter policy contains combinations that are not allowed, as described in [Section 6.1.3](#metadata_policy_operators){.auto .internal .xref} and in accordance with the specifications of the operators, this MUST produce a policy error.
  

- 
  Subordinate metadata parameter policies that are not present in the current Entity Type metadata policy MUST be copied to it.
  

At the level of operators, the merge proceeds as follows:

- 
  If an operator is already present in the current metadata parameter policy, the values of the subordinate operator MUST be merged, as described in [Section 6.1.3](#metadata_policy_operators){.auto .internal .xref} and in accordance with the operator specification. If an operator value merge is not allowed or otherwise unsuccessful this MUST produce a policy error.
  

- 
  Subordinate operators that are not present in the current metadata parameter policy MUST be copied to it.
  

If no further Subordinate Statements with a `metadata_policy` claim are found, the current metadata policy becomes the resolved one for the Trust Chain.





##### Application

If the Subordinate Statement about the Trust Chain subject contains a `metadata` claim, this MUST first be applied, as described in the claim definition in [Section 3](#entity-statement){.auto .internal .xref}, and only then it can be proceeded with applying the resolved metadata policy.

If the process described in [Section 6.1.4.1](#metadata_policy_resolution){.auto .internal .xref} found no Subordinate Statements in the Trust Chain with a `metadata_policy` claim, the metadata of the Trust Chain subject resolves simply to the `metadata` found in its Entity Configuration, with any `metadata` parameters provided by the Immediate Superior applied to it.

If a metadata policy was resolved for the Trust Chain, for every Entity Type metadata and metadata parameter for which a corresponding metadata parameter policy is present, the included policy operators MUST be applied as described in [Section 6.1.3](#metadata_policy_operators){.auto .internal .xref} and in accordance with the specifications of the operators. The operators MUST be applied to the metadata parameter in a sequence that is determined by the absolute or relative order specified for each operator.

If the application of metadata policies results in illegal or otherwise incorrect Resolved Metadata, then the metadata MUST be regarded as broken and MUST NOT be used.

The Trust Chain subject is responsible to verify that it is able to support and comply with the Resolved Metadata that results from the application of federation metadata policies. For instance, this may involve a check that cryptographic algorithms required by the resulting Resolved Metadata are supported. Likewise, the Trust Chain subject MUST verify that all the required metadata parameters for its Entity Types are present and all the metadata values in the Resolved Metadata are valid. When metadata policies change, Trust Chain subjects may need to reevaluate their support and compliance.







#### Metadata Policy Example

The following is a non-normative example of resolving and applying Trust Chain metadata policies for an OpenID relying party.

We start with a federation Trust Anchor's `metadata_policy` for RPs:

[]{#name-example-metadata-policy-of-}

<figure id="figure-12">
<div id="section-6.1.5-3.1" class="alignLeft art-text artwork">
<pre><code>&quot;metadata_policy&quot;: {
  &quot;openid_relying_party&quot;: {
    &quot;grant_types&quot;: {
       &quot;default&quot;: [
        &quot;authorization_code&quot;
      ],
      &quot;subset_of&quot;: [
        &quot;authorization_code&quot;,
        &quot;refresh_token&quot;
      ],
      &quot;superset_of&quot;: [
        &quot;authorization_code&quot;
      ]
    },
    &quot;token_endpoint_auth_method&quot;: {
      &quot;one_of&quot;: [
        &quot;private_key_jwt&quot;,
        &quot;self_signed_tls_client_auth&quot;
      ],
      &quot;essential&quot;: true
    },
    &quot;token_endpoint_auth_signing_alg&quot; : {
      &quot;one_of&quot;: [
        &quot;PS256&quot;,
        &quot;ES256&quot;
      ]
    },
    &quot;subject_type&quot;: {
      &quot;value&quot;: &quot;pairwise&quot;
    },
    &quot;contacts&quot;: {
      &quot;add&quot;: [
        &quot;helpdesk@federation.example.org&quot;
      ]
    }
  }
}</code></pre>
</div>
<figcaption><a href="#figure-12" class="selfRef">Figure 12</a>: <a href="#name-example-metadata-policy-of-" class="selfRef">Example Metadata Policy of a Trust Anchor for RPs</a></figcaption>
</figure>

Next, we have an Intermediate organization's `metadata_policy` for Subordinate RPs together with `metadata` values for its Immediate Subordinate RPs:

[]{#name-example-metadata-policy-and}

<figure id="figure-13">
<div id="section-6.1.5-5.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;metadata_policy&quot;: {
    &quot;openid_relying_party&quot;: {
      &quot;grant_types&quot;: {
        &quot;subset_of&quot;: [
          &quot;authorization_code&quot;
        ]
      },
      &quot;token_endpoint_auth_method&quot;: {
        &quot;one_of&quot;: [
          &quot;self_signed_tls_client_auth&quot;
        ]
      },
      &quot;contacts&quot;: {
        &quot;add&quot;: [
          &quot;helpdesk@org.example.org&quot;
        ]
      }
    }
  },
  &quot;metadata&quot;: {
    &quot;openid_relying_party&quot;: {
      &quot;sector_identifier_uri&quot;: &quot;https://org.example.org/sector-ids.json&quot;,
      &quot;policy_uri&quot;: &quot;https://org.example.org/policy.html&quot;
    }
  }
}</code></pre>
</div>
<figcaption><a href="#figure-13" class="selfRef">Figure 13</a>: <a href="#name-example-metadata-policy-and" class="selfRef">Example Metadata Policy and Metadata Values of an Intermediate Entity for RPs</a></figcaption>
</figure>

Merging the example RP metadata policy of the Intermediate Entity into the RP metadata policy of the Trust Anchor produces the following policy for the Trust Chain:

[]{#name-example-merged-metadata-pol}

<figure id="figure-14">
<div id="section-6.1.5-7.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;grant_types&quot;: {
    &quot;default&quot;: [
      &quot;authorization_code&quot;
    ],
    &quot;superset_of&quot;: [
      &quot;authorization_code&quot;
    ],
    &quot;subset_of&quot;: [
      &quot;authorization_code&quot;
    ]
  },
  &quot;token_endpoint_auth_method&quot;: {
    &quot;one_of&quot;: [
      &quot;self_signed_tls_client_auth&quot;
    ],
    &quot;essential&quot;: true
  },
  &quot;token_endpoint_auth_signing_alg&quot;: {
    &quot;one_of&quot;: [
      &quot;PS256&quot;,
      &quot;ES256&quot;
    ]
  },
  &quot;subject_type&quot;: {
    &quot;value&quot;: &quot;pairwise&quot;
  },
  &quot;contacts&quot;: {
    &quot;add&quot;: [
      &quot;helpdesk@federation.example.org&quot;,
      &quot;helpdesk@org.example.org&quot;
    ]
  }
}</code></pre>
</div>
<figcaption><a href="#figure-14" class="selfRef">Figure 14</a>: <a href="#name-example-merged-metadata-pol" class="selfRef">Example Merged Metadata Policy for RPs</a></figcaption>
</figure>

The Trust Chain subject is a Leaf Entity, which publishes the following RP metadata in its Entity Configuration:

[]{#name-example-entity-configuratio}

<figure id="figure-15">
<div id="section-6.1.5-9.1" class="alignLeft art-text artwork">
<pre><code>&quot;metadata&quot;: {
  &quot;openid_relying_party&quot;: {
    &quot;redirect_uris&quot;: [
      &quot;https://rp.example.org/callback&quot;
    ],
    &quot;response_types&quot;: [
      &quot;code&quot;
    ],
    &quot;token_endpoint_auth_method&quot;: &quot;self_signed_tls_client_auth&quot;,
    &quot;contacts&quot;: [
      &quot;rp_admins@rp.example.org&quot;
    ]
  }
}</code></pre>
</div>
<figcaption><a href="#figure-15" class="selfRef">Figure 15</a>: <a href="#name-example-entity-configuratio" class="selfRef">Example Entity Configuration RP Metadata</a></figcaption>
</figure>

The `metadata` values specified by the Intermediate Entity for its Immediate Subordinates are applied to the Trust Chain subject `metadata`. After that, the merged metadata policy is applied, to produce the following resulting resolved RP metadata:

[]{#name-the-resulting-resolved-rp-m}

<figure id="figure-16">
<div id="section-6.1.5-11.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;redirect_uris&quot;: [
    &quot;https://rp.example.org/callback&quot;
  ],
  &quot;grant_types&quot;: [
    &quot;authorization_code&quot;
  ],
  &quot;response_types&quot;: [
    &quot;code&quot;
  ],
  &quot;token_endpoint_auth_method&quot;: &quot;self_signed_tls_client_auth&quot;,
  &quot;subject_type&quot;: &quot;pairwise&quot;,
  &quot;sector_identifier_uri&quot;: &quot;https://org.example.org/sector-ids.json&quot;,
  &quot;policy_uri&quot;: &quot;https://org.example.org/policy.html&quot;,
  &quot;contacts&quot;: [
    &quot;rp_admins@rp.example.org&quot;,
    &quot;helpdesk@federation.example.org&quot;,
    &quot;helpdesk@org.example.org&quot;
  ]
}</code></pre>
</div>
<figcaption><a href="#figure-16" class="selfRef">Figure 16</a>: <a href="#name-the-resulting-resolved-rp-m" class="selfRef">The Resulting Resolved RP Metadata for the Trust Chain Subject</a></figcaption>
</figure>







### Constraints

Trust Anchors and Intermediate Entities MAY define constraining criteria that apply to their Subordinates. They are expressed in the `constraints` claim of a Subordinate Statement, as described in [Section 3](#entity-statement){.auto .internal .xref}.

The following constraint parameters are defined:

[]{.break}

max_path_length
:   OPTIONAL. Integer specifying the maximum number of Intermediate Entities between the Entity setting the constraint and the Trust Chain subject.
:   

naming_constraints
:   OPTIONAL. JSON object specifying restrictions on the URIs of the Entity Identifiers of Subordinate Entities. Restrictions are defined in terms of permitted and excluded URI name subtrees.
:   

allowed_entity_types
:   OPTIONAL. Array of string Entity Type Identifiers. Entity Type Identifiers are defined in [Section 5.1](#entity_types){.auto .internal .xref}. This constraint specifies the Entity Types and hence the metadata that Subordinate Entities are allowed to have.
:   

Additional constraint parameters MAY be defined and used. If they are not understood, they MUST be ignored.

The following is a non-normative example of a set of constraints:

[]{#name-example-set-of-constraints}

<figure id="figure-17">
<div id="section-6.2-6.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;max_path_length&quot;: 2,
  &quot;naming_constraints&quot;: {
    &quot;permitted&quot;: [
      &quot;.example.com&quot;
    ],
    &quot;excluded&quot;: [
      &quot;east.example.com&quot;
    ]
  },
  &quot;allowed_entity_types&quot;: [
    &quot;openid_provider&quot;,
    &quot;openid_relying_party&quot;
  ]
}</code></pre>
</div>
<figcaption><a href="#figure-17" class="selfRef">Figure 17</a>: <a href="#name-example-set-of-constraints" class="selfRef">Example Set of Constraints</a></figcaption>
</figure>

When resolving the Trust Chain for an Entity the `constraints` claim in each Subordinate Statement MUST be independently applied, if present. If any of the `constraints` checks fails, the Trust Chain MUST be considered invalid.



#### Max Path Length

The `max_path_length` constraint specifies the maximum allowed number of Intermediate Entities in a Trust Chain between a Trust Anchor or Intermediate that sets the constraint and the Trust Chain subject.

A `max_path_length` constraint of zero indicates that no Intermediates MAY appear between this Entity and the Trust Chain subject. The `max_path_length` constraint MUST have a value greater than or equal to zero.

Omitting the `max_path_length` constraint means that there are no additional constraints apart from those already in effect.

Assuming that we have a Trust Chain with four Entity Statements:

1.  
    Entity Configuration of a Leaf Entity (LE)
    

2.  
    Subordinate Statement by an Intermediate 1 (I1) about LE
    

3.  
    Subordinate Statement by an Intermediate 2 (I2) about I1
    

4.  
    Subordinate Statement by a Trust Anchor (TA) about I2
    

Then the Trust Chain fulfills the constraints if, for instance:

- 
  The TA specifies a `max_path_length` that is greater than or equal to 2.
  

- 
  TA specifies a `max_path_length` of 2, I2 specifies a `max_path_length` of 1, and I1 omits the `max_path_length` constraint.
  

- 
  Neither TA nor I2 specifies any `max_path_length` constraint while I1 sets `max_path_length` to 0.
  

The Trust Chain does not fulfill the constraints if, for instance, the:

- 
  The TA sets the `max_path_length` to 1.
  





#### Naming Constraints

The `naming_constraints` member specifies a URI namespace within which the Entity Identifiers of Subordinate Entities in a Trust Chain MUST be located.

Restrictions are defined in terms of URI name subtrees, using `permitted` and/or `excluded` members within the `naming_constraints` member, each of which contains an array of names to be permitted or excluded. Any name matching a restriction in the excluded list is invalid, regardless of the information appearing in the permitted list.

This specification uses the syntax of domain name constraints specified in Section 4.2.1.10 of [[RFC5280](#RFC5280){.cite .xref}]. As stated there, a domain name constraint MUST be specified as a fully qualified domain name and MAY specify a host or a domain. Examples are "host.example.com" and ".example.com". When the domain name constraint begins with a period, it MAY be expanded with one or more labels. That is, the domain name constraint ".example.com" is satisfied by both host.example.com and my.host.example.com. However, the domain name constraint ".example.com" is not satisfied by "example.com". When the domain name constraint does not begin with a period, it specifies a host. As in RFC 5280, domain name constraints apply to the host part of the URI.





#### Entity Type Constraints

The `allowed_entity_types` constraint specifies the acceptable metadata Entity Types of Subordinate Entities in a Trust Chain. If there is no `allowed_entity_types` constraint, it means that any Entity Type is allowed. The `federation_entity` Entity Type Identifier, specified in [Section 5.1.1](#federation_entity){.auto .internal .xref}, is always allowed and MUST NOT be included in the constraint. If the constraint is the empty array `[]`, it means that only the `federation_entity` Entity Type is allowed.

To apply the `allowed_entity_types` constraint during Trust Chain Resolution all Entity Types that are not listed in the `allowed_entity_types` constraint MUST be removed from the metadata claim in the subject's Entity Configuration. The `federation_entity` Entity Type MUST NOT be removed. This MUST be done before applying Metadata Policies but after applying Metadata from a direct superior's Subordinate Statement.









## Trust Marks

Per the definition in [Section 1.2](#Terminology){.auto .internal .xref}, Trust Marks are statements of conformance to sets of criteria determined by an accreditation authority. Trust Marks used by this specification are signed JWTs. Entity Statements MAY include Trust Marks, as described in the `trust_marks` claim definition in [Section 3](#entity-statement){.auto .internal .xref}.

Trust Marks are signed by a federation-accredited authority called a Trust Mark Issuer. All Trust Mark Issuers MUST be represented in the federation by an Entity. The fact that a Trust Mark Issuer is accepted by the federation is expressed in the `trust_mark_issuers` claim of the Trust Anchor's Entity Configuration.

The key used by the Trust Mark issuer to sign its Trust Marks MUST be one of the private keys in its set of Federation Entity Keys.

Trust Mark JWTs MUST include the `kid` (Key ID) header parameter with its value being the Key ID of the signing key used.

Note that a federation MAY allow an Entity to self-sign Trust Marks.

Trust Mark JWTs MUST be explicitly typed by using the `typ` header parameter to prevent cross-JWT confusion, per Section 3.11 of [[RFC8725](#RFC8725){.cite .xref}]. The `typ` header parameter value MUST be `trust-mark+jwt` unless the trust framework in use defines a more specific media type value for the particular kind of Trust Mark. Trust Marks without a `typ` header parameter or an unrecognized `typ` value MUST be rejected.



### Trust Mark Claims

The claims in a Trust Mark are:

[]{.break}

iss
:   REQUIRED. String. The Entity Identifier of the issuer of the Trust Mark.
:   

sub
:   REQUIRED. String. The Entity Identifier of the Entity this Trust Mark applies to.
:   

trust_mark_type
:   REQUIRED. The `trust_mark_type` claim defined in [Section 7.1](#trust_mark_claims){.auto .internal .xref} is used in Trust Marks to provide the identifier of the type of the Trust Mark. The Trust Mark type identifier MUST be collision-resistant across multiple federations. It is RECOMMENDED that the identifier value is built using a URL that uniquely identifies the federation or the trust framework within which it was issued. This is required to prevent Trust Marks issued in different federations from having colliding identifiers.
:   

iat
:   REQUIRED. Number. Time when this Trust Mark was issued. This is expressed as Seconds Since the Epoch, per [[RFC7519](#RFC7519){.cite .xref}].
:   

logo_uri
:   OPTIONAL. String. URL that references a logo for the issued Trust Mark. The value of this field MUST point to a valid image file.
:   

exp
:   OPTIONAL. Number. Time when this Trust Mark is no longer valid. This is expressed as Seconds Since the Epoch, per [[RFC7519](#RFC7519){.cite .xref}]. If not present, it means that the Trust Mark does not expire.
:   

ref
:   OPTIONAL. The `ref` (reference) claim defined in [Section 13.5](#refClaim){.auto .internal .xref} is used in Trust Marks to provide a URL referring to human-readable information about the issuance of the Trust Mark.
:   

delegation
:   OPTIONAL. The `delegation` claim defined in [Section 13.6](#delegationClaim){.auto .internal .xref} is used in Trust Marks to delegate the right to issue Trust Marks with a particular identifier. Its value is a Trust Mark delegation JWT, as defined in [Section 7.2.1](#delegation_jwt){.auto .internal .xref}.
:   

Additional claims MAY be defined and used in conjunction with the claims above.





### Trust Mark Delegation

There will be cases where the owner of a Trust Mark for some reason does not match the Trust Mark Issuer due to administrative or technical requirements. Take as an example vehicle inspection. Vehicle inspection is a procedure mandated by national or subnational governments in many countries, in which a vehicle is inspected to ensure that it conforms to regulations governing safety, emissions, or both. The body that mandates the inspections does not perform them; instead, there may be commercial companies performing the inspections, after which they issue the Trust Mark.

The fact that a Trust Mark is issued by a Trust Mark Issuer that is not the owner of the Trust Mark is expressed by including a `delegation` claim in the Trust Mark, whose value is a Trust Mark delegation JWT, as defined in [Section 7.2.1](#delegation_jwt){.auto .internal .xref}.

If the Federation Operator knows that Trust Marks with a certain Trust Mark type identifier may legitimately be issued by Trust Mark Issuers that are not the owner of the Trust Mark type identifier, then information about the owner and the Trust Mark type identifier MUST be included in the `trust_mark_owners` claim in the Trust Anchor's Entity Configuration.



#### Trust Mark Delegation JWT

A Trust Mark Delegation JWT is a signed JWT issued by a Trust Mark Owner that identifies a legitimate delegated issuer of Trust Marks with a particular identifier.

A Trust Mark delegation JWT MUST be explicitly typed, by setting the `typ` header parameter to `trust-mark-delegation+jwt` to prevent cross-JWT confusion, per Section 3.11 of [[RFC8725](#RFC8725){.cite .xref}]. Trust Mark delegation JWTs without a `typ` header parameter or with a different `typ` value MUST be rejected. It is signed with a Federation Entity Key.

Trust Mark delegation JWTs MUST include the `kid` (Key ID) header parameter with its value being the Key ID of the signing key used.

The claims in a Trust Mark delegation JWT are:

[]{.break}

iss
:   REQUIRED. String. The owner of the Trust Mark.
:   

sub
:   REQUIRED. String. The Entity this delegation applies to.
:   

trust_mark_type
:   REQUIRED. String. The identifier for the type of the Trust Mark.
:   

iat
:   REQUIRED. Number. Time when this delegation was issued. This is expressed as Seconds Since the Epoch, per [[RFC7519](#RFC7519){.cite .xref}].
:   

exp
:   OPTIONAL. Number. Time when this delegation stops being valid. This is expressed as Seconds Since the Epoch, per [[RFC7519](#RFC7519){.cite .xref}]. If not present, it means that the delegation does not expire.
:   

ref
:   OPTIONAL. String. URL that points to human-readable information connected to the Trust Mark.
:   

Additional claims MAY be defined and used in conjunction with the claims above.





#### Validating a Trust Mark Delegation

Validating a Trust Mark Delegation means validating a Trust Mark Delegation instance represented by a specific signed JWT.

Henceforth, "delegation" is used as a shorthand for a Trust Mark Delegation JWT. The Trust Anchor referenced below is a Trust Anchor that has been successfully used when establishing trust in the Trust Mark Issuer.

To validate a delegation, the following validation steps MUST be performed. Please note that if any of these validation checks fail, the entire validation process fails and the delegation is considered invalid.

1.  
    The delegation MUST be a signed JWT
    

2.  
    The delegation MUST have a `typ` header with the value `trust-mark-delegation+jwt`
    

3.  
    The delegation MUST have an `alg` (algorithm) header parameter with a value that is an acceptable JWS signing algorithm; it MUST NOT be `none`
    

4.  
    The Entity Identifier of the Trust Mark Issuer MUST match the value of `sub` in the delegation
    

5.  
    The Entity Identifier of the Trust Mark Owner MUST match the value of `iss` in the delegation.
    

6.  
    The current time MUST be after the time represented by the iat (issued at) Claim in the delegation (possibly allowing for some small leeway to account for clock skew).
    

7.  
    The current time MUST be before the time represented by the exp (expiration) Claim in the delegation (possibly allowing for some small leeway to account for clock skew).
    

8.  
    The value of the Claim `trust_mark_type` in the delegation MUST be the same as the value of the Claim `trust_mark_type` in the Trust Mark.
    

9.  
    The delegation's signature MUST validate using one of the Trust Mark Owner's keys identified by the value of the header parameter `kid`. The Trust Mark Owner's keys can be found in the `trust_mark_owners` claim in the Trust Anchor's Entity Configuration.
    







### Validating a Trust Mark

Validating a Trust Mark means validating a Trust Mark instance represented by a specific signed JWT. It is NOT about validating whether a Trust Mark of a particular kind can exist or not.

The trust in the Trust Mark Issuer comes before the trust in the trust mark. If the Trust Mark Issuer is not trusted then the trust mark cannot be trusted. An Entity MUST therefore establish trust in the Trust Mark Issuer by following the procedure defined in [Section 10](#resolving_trust){.auto .internal .xref} prior to starting the Trust Mark validation process defined below.

Henceforth, "instance" is used as a shorthand for Trust Mark instance. The Trust Anchor referenced below is a Trust Anchor that has been successfully used when establishing trust in the Trust Mark Issuer.

To validate an instance, the following validation steps MUST be performed. Please note that if any of these validation checks fail, the entire validation process fails and the instance is considered invalid.

1.  
    The instance MUST be a signed JWT
    

2.  
    The instance MUST have a `typ` header with the value `trust-mark+jwt`
    

3.  
    The instance MUST have an `alg` (algorithm) header parameter with a value that is an acceptable JWS signing algorithm; it MUST NOT be `none`.
    

4.  
    The Entity Identifier of the Entity whose Entity Configuration contains the instance MUST match the value of the claim `sub` in the Trust Mark.
    

5.  
    The current time MUST be after the time represented by the `iat` (issued at) Claim (possibly allowing for some small leeway to account for clock skew).
    

6.  
    The current time MUST be before the time represented by the `exp` (expiration) Claim (possibly allowing for some small leeway to account for clock skew).
    

7.  
    The instance's signature MUST validate using the Trust Mark issuer's key identified by the `kid` value.
    

8.  
    If the `trust_mark_type` of the instance appears in the `trust_mark_owners` claim in the Trust Anchor's Entity Configuration, then the instance MUST contain a `delegation` claim.
    

9.  
    If there is a `delegation` claim in the instance, the value of that claim MUST be validated according to [Section 7.2.2](#trust-mark-delegation-validation){.auto .internal .xref}.
    

If Trust Marks are issued without an expiration time, it is RECOMMENDED that a mechanism be provided to validate them, such as the Trust Mark Status endpoint and/or the Trust Marked Entities Listing endpoint.

As an alternative to the above procedure for validating Trust Marks, implementations MAY use the Trust Mark Status endpoint to verify that the Trust Mark is valid and still active, as described in [Section 8.4](#status_endpoint){.auto .internal .xref}.





### Trust Mark Examples

A non-normative example of a `trust_marks` claim in the JWT Claims Set for an Entity Configuration is:

[]{#name-trust-mark-in-an-entity-con}

<figure id="figure-18">
<div id="section-7.4-2.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;iss&quot;: &quot;https://rp.example.it/spid/&quot;,
  &quot;sub&quot;: &quot;https://rp.example.it/spid/&quot;,
  &quot;iat&quot;: 1516239022,
  &quot;exp&quot;: 1516298022,
  &quot;trust_marks&quot;: [
    {
     &quot;trust_mark_type&quot;: &quot;https://www.spid.gov.it/certification/rp&quot;,
     &quot;trust_mark&quot;:
       &quot;eyJ0eXAiOiJ0cnVzdC1tYXJrK2p3dCIsImFsZyI6IlJTMjU2Iiwia2lkIjoia29
        zR20yd3VaaDlER21OeEF0a3VPNDBwUGpwTDMtakNmMU4tcVBPLVllVSJ9.
        eyJpc3MiOiJodHRwczovL3d3dy5hZ2lkLmdvdi5pdCIsInN1YiI6Imh0dHBzOi8
        vcnAuZXhhbXBsZS5pdC9zcGlkIiwiaWF0IjoxNTc5NjIxMTYwLCJ0cnVzdF9tYX
        JrX3R5cGUiOiJodHRwczovL3d3dy5zcGlkLmdvdi5pdC9jZXJ0aWZpY2F0aW9uL
        3JwIiwibG9nb191cmkiOiJodHRwczovL3d3dy5hZ2lkLmdvdi5pdC90aGVtZXMv
        Y3VzdG9tL2FnaWQvbG9nby5zdmciLCJyZWYiOiJodHRwczovL2RvY3MuaXRhbGl
        hLml0L2RvY3Mvc3BpZC1jaWUtb2lkYy1kb2NzL2l0L3ZlcnNpb25lLWNvcnJlbn
        RlLyJ9.
        L_pSh1InEiFAcs3E-1HBM7fNZYwF5ru3UGA_8yc80dGS3sszfA_sbj4AoW_zAJW
        QBdZpjxnHBBmybYXFrfZBcqxcedsrvUYrmbt1nPYxbUE54fRRoZK-sJmVqh1GzS
        an5nOmkxuAtMinU8k_-aWnPWj83sYe2AzT2mMgkXiz8zhda3jZm8hoxZ4jR6B0Y
        AvbMlq2pPWO5OWKdZhiFRMSprwh0GYluQkK0j1aLNMGXD3keMJd2zEoWX9D7w2f
        XShAA48W3cNhuXyBVnCoum1K4IWK3s_fx4nIkp6W-V4jCBOpxp7Yo8LZ30o_xpE
        OzGTIECGWVR86azOAlwVC8XSiAA&quot;
    }
  ],
  &quot;metadata&quot;: {
    &quot;openid_relying_party&quot;: {
      &quot;application_type&quot;: &quot;web&quot;,
      &quot;client_registration_types&quot;: [&quot;automatic&quot;],
      &quot;client_name&quot;: &quot;https://rp.example.it/spid/&quot;,
      &quot;contacts&quot;: [
        &quot;ops@rp.example.it&quot;
      ]
    }
  }
}</code></pre>
</div>
<figcaption><a href="#figure-18" class="selfRef">Figure 18</a>: <a href="#name-trust-mark-in-an-entity-con" class="selfRef">Trust Mark in an Entity Configuration JWT Claims Set</a></figcaption>
</figure>

An example of a decoded Trust Mark payload issued to an RP, attesting to conformance to a national public service profile:

[]{#name-trust-mark-for-a-national-p}

<figure id="figure-19">
<div id="section-7.4-4.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;trust_mark_type&quot;:&quot;https://mushrooms.federation.example.com/openid_relying_party/public/&quot;,
  &quot;iss&quot;: &quot;https://epigeo.tm-issuer.example.it&quot;,
  &quot;sub&quot;: &quot;https://porcino.example.com/rp&quot;,
  &quot;iat&quot;: 1579621160,
  &quot;organization_name&quot;: &quot;Porcino Mushrooms &amp; Co.&quot;,
  &quot;policy_uri&quot;: &quot;https://porcino.example.com/privacy_policy&quot;,
  &quot;tos_uri&quot;: &quot;https://porcino.example.com/info_policy&quot;,
  &quot;service_documentation&quot;: &quot;https://porcino.example.com/api/v1/get/services&quot;,
  &quot;ref&quot;: &quot;https://porcino.example.com/documentation/manuale_operativo.pdf&quot;
}
</code></pre>
</div>
<figcaption><a href="#figure-19" class="selfRef">Figure 19</a>: <a href="#name-trust-mark-for-a-national-p" class="selfRef">Trust Mark for a National Profile</a></figcaption>
</figure>

An example of a decoded Trust Mark payload issued to an RP, attesting to its conformance to the rules for data management of underage users:

[]{#name-trust-mark-issued-to-an-rp}

<figure id="figure-20">
<div id="section-7.4-6.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;trust_mark_type&quot;:&quot;https://mushrooms.federation.example.com/openid_relying_party/private/under-age&quot;,
  &quot;iss&quot;: &quot;https://trustissuer.pinarolo.example.it&quot;,
  &quot;sub&quot;: &quot;https://vavuso.example.com/rp&quot;,
  &quot;iat&quot;: 1579621160,
  &quot;organization_name&quot;: &quot;Pinarolo Suillus luteus&quot;,
  &quot;policy_uri&quot;: &quot;https://vavuso.example.com/policy&quot;,
  &quot;tos_uri&quot;: &quot;https://vavuso.example.com/tos&quot;
}
</code></pre>
</div>
<figcaption><a href="#figure-20" class="selfRef">Figure 20</a>: <a href="#name-trust-mark-issued-to-an-rp" class="selfRef">Trust Mark Issued to an RP</a></figcaption>
</figure>

An example of a decoded Trust Mark payload attesting a stipulation of an agreement between two organization's Entities:

[]{#name-trust-mark-attesting-to-an-}

<figure id="figure-21">
<div id="section-7.4-8.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;trust_mark_type&quot;: &quot;https://mushrooms.federation.example.com/arrosto/agreements&quot;,
  &quot;iss&quot;: &quot;https://agaricaceae.example.it&quot;,
  &quot;sub&quot;: &quot;https://coppolino.example.com&quot;,
  &quot;iat&quot;: 1579621160,
  &quot;logo_uri&quot;: &quot;https://coppolino.example.com/sgd-cmyk-150dpi-90mm.svg&quot;,
  &quot;organization_type&quot;: &quot;public&quot;,
  &quot;id_code&quot;: &quot;123456&quot;,
  &quot;email&quot;: &quot;info@coppolino.example.com&quot;,
  &quot;organization_name#it&quot;: &quot;Mazza di Tamburo&quot;,
  &quot;policy_uri#it&quot;: &quot;https://coppolino.example.com/privacy_policy&quot;,
  &quot;tos_uri#it&quot;: &quot;https://coppolino.example.com/info_policy&quot;,
  &quot;service_documentation&quot;: &quot;https://coppolino.example.com/api/v1/get/services&quot;,
  &quot;ref&quot;: &quot;https://agaricaceae.example.it/documentation/agaricaceae.pdf&quot;
}</code></pre>
</div>
<figcaption><a href="#figure-21" class="selfRef">Figure 21</a>: <a href="#name-trust-mark-attesting-to-an-" class="selfRef">Trust Mark Attesting to an Agreement Between Entities</a></figcaption>
</figure>

An example of a decoded Trust Mark payload asserting conformance to a security profile:

[]{#name-trust-mark-asserting-confor}

<figure id="figure-22">
<div id="section-7.4-10.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;trust_mark_type&quot;: &quot;https://mushrooms.federation.example.com/ottimo/commestibile&quot;,
  &quot;iss&quot;: &quot;https://cantharellus.cibarius.example.org&quot;,
  &quot;sub&quot;: &quot;https://gallinaccio.example.com/op&quot;,
  &quot;iat&quot;: 1579621160,
  &quot;logo_uri&quot;: &quot;https://cantharellus.cibarius/static/images/cantharellus-cibarius.svg&quot;,
  &quot;ref&quot;: &quot;https://cantharellus.cibarius/cantharellus/cibarius&quot;
}</code></pre>
</div>
<figcaption><a href="#figure-22" class="selfRef">Figure 22</a>: <a href="#name-trust-mark-asserting-confor" class="selfRef">Trust Mark Asserting Conformance to a Security Profile</a></figcaption>
</figure>

An example of a decoded self-signed Trust Mark:

[]{#name-self-signed-trust-mark}

<figure id="figure-23">
<div id="section-7.4-12.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;trust_mark_type&quot;: &quot;https://mushrooms.federation.example.com/trust-marks/self-signed&quot;,
  &quot;iss&quot;: &quot;https://amanita.muscaria.example.com&quot;,
  &quot;sub&quot;: &quot;https://amanita.muscaria.example.com&quot;,
  &quot;iat&quot;: 1579621160,
  &quot;logo_uri&quot;: &quot;https://amanita.muscaria.example.com/img/amanita-mus.svg&quot;,
  &quot;ref&quot;: &quot;https://amanita.muscaria.example.com/uploads/cookbook.zip&quot;
}</code></pre>
</div>
<figcaption><a href="#figure-23" class="selfRef">Figure 23</a>: <a href="#name-self-signed-trust-mark" class="selfRef">Self-Signed Trust Mark</a></figcaption>
</figure>

An example of a third-party accreditation authority for Trust Marks:

[]{#name-third-party-accreditation-a}

<figure id="figure-24">
<div id="section-7.4-14.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;iss&quot;: &quot;https://swamid.se&quot;,
  &quot;sub&quot;: &quot;https://umu.se/op&quot;,
  &quot;iat&quot;: 1577833200,
  &quot;exp&quot;: 1609369200,
  &quot;trust_mark_type&quot;: &quot;https://refeds.org/wp-content/uploads/2016/01/Sirtfi-1.0.pdf&quot;
}</code></pre>
</div>
<figcaption><a href="#figure-24" class="selfRef">Figure 24</a>: <a href="#name-third-party-accreditation-a" class="selfRef">Third-Party Accreditation Authority for Trust Marks</a></figcaption>
</figure>





### Trust Mark Delegation Example

A non-normative example of a `trust_marks` claim in the JWT Claims Set for an Entity Configuration in which the Trust Mark is issued by an Entity that issues Trust Marks on behalf of another Entity. The fact that a Trust Mark is issued by a Trust Mark Issuer that is not the owner of the Trust Mark is expressed by including a `delegation` claim in the Trust Mark, whose value is a signed JWT.

[]{#name-example-of-a-trust-mark-usi}

<figure id="figure-25">
<div id="section-7.5-2.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;delegation&quot;:
    &quot;eyJ0eXAiOiJ0cnVzdC1tYXJrLWRlbGVnYXRpb24rand0IiwiYWxnIjoiUl
    MyNTYiLCJraWQiOiJrb3NHbTJ3dVpoOURHbU54QXRrdU80MHBQanBMMy1qQ
    2YxTi1xUE8tWWVVIn0.
    eyJzdWIiOiJodHRwczovL3RtaS5leGFtcGxlLm9yZyIsInRydXN0X21hcmt
    fdHlwZSI6Imh0dHBzOi8vcmVmZWRzLm9yZy9zaXJ0ZmkiLCJpc3MiOiJodH
    RwczovL3RtX293bmVyLmV4YW1wbGUub3JnIiwiaWF0IjoxNzI1MTc2MzAyfQ.
    ao0rWGpVjEgpNyFxsKawps8q71eYnp78TzRdY4P52
    CT8QX6etXt-2L2Z1Vw5A6jx2mhjpPwWi_sOxfiOSA5TugJfN0Gbwj7teTzM
    0IMciuasCWgnLrKyLZjS147ZE50I9e9P8Ot8UQwhmXcLiuwsbDxSdqM4pVp
    75lfWnmzPH0L2pDZG5COFgIgSOAlK3TVMBOR8fziF-VmWNPzAfB0lSc-hjH
    -7q66GyT43o3Exnm6DsoLxyB8bxG99BQltLxURDT90CzM6szGcF3OG64Rbe
    0I4lT_LAOfvhlrRbT56eK4sJNCsbVbGnDBfFmyfB_HIeBMGP0L7T5JPMOUU
    9bjIlA&quot;,
  &quot;iat&quot;: 1725176302,
  &quot;trust_mark_type&quot;: &quot;https://refeds.org/sirtfi&quot;,
  &quot;sub&quot;: &quot;https://entity.example.org&quot;,
  &quot;exp&quot;: 1727768302,
  &quot;iss&quot;: &quot;https://tmi.example.org&quot;
}</code></pre>
</div>
<figcaption><a href="#figure-25" class="selfRef">Figure 25</a>: <a href="#name-example-of-a-trust-mark-usi" class="selfRef">Example of a Trust Mark using delegation. Only the JWT Claims Set of the Trust Mark is shown.</a></figcaption>
</figure>

JWS Header Parameters for the Trust Mark delegation JWT in the "delegation" claim above

[]{#name-trust-mark-delegation-jwt-j}

<figure id="figure-26">
<div id="section-7.5-4.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;typ&quot;: &quot;trust-mark-delegation+jwt&quot;,
  &quot;alg&quot;: &quot;RS256&quot;,
  &quot;kid&quot;: &quot;kosGm2wuZh9DGmNxAtkuO40pPjpL3-jCf1N-qPO-YeU&quot;
}</code></pre>
</div>
<figcaption><a href="#figure-26" class="selfRef">Figure 26</a>: <a href="#name-trust-mark-delegation-jwt-j" class="selfRef">Trust Mark delegation JWT JWS Header Parameters</a></figcaption>
</figure>

JWT Claims Set of the Trust Mark delegation JWT in the "delegation" claim above

[]{#name-trust-mark-delegation-jwt-c}

<figure id="figure-27">
<div id="section-7.5-6.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;sub&quot;: &quot;https://tmi.example.org&quot;,
  &quot;trust_mark_type&quot;: &quot;https://refeds.org/sirtfi&quot;,
  &quot;iss&quot;: &quot;https://tm_owner.example.org&quot;,
  &quot;iat&quot;: 1725176302
}</code></pre>
</div>
<figcaption><a href="#figure-27" class="selfRef">Figure 27</a>: <a href="#name-trust-mark-delegation-jwt-c" class="selfRef">Trust Mark delegation JWT Claim Set</a></figcaption>
</figure>







## Federation Endpoints

The federation endpoints of an Entity can be found in the configuration response as described in [Section 9](#federation_configuration){.auto .internal .xref} or by other means.

For all federation endpoints, additional request parameters beyond those initially specified MAY be defined and used. If they are not understood, they MUST be ignored.



### Fetching a Subordinate Statement

The fetch endpoint is used to collect Subordinate Statements one-by-one when assembling Trust Chains. An Entity with Subordinates MUST expose a fetch endpoint. An Entity MUST publish Subordinate Statements about its Immediate Subordinates via its fetch endpoint.

The fetch endpoint location is published in the Entity's `federation_entity` metadata in the `federation_fetch_endpoint` parameter defined in [Section 5.1.1](#federation_entity){.auto .internal .xref}. Since this endpoint is used when building and validating Trust Chains, its location MUST be available before metadata and metadata policies from Superiors can be applied. Therefore, this endpoint MUST be published directly in Entity Configuration metadata and not in Subordinate Statements.

To fetch a Subordinate Statement, one needs to know the identifier of the Entity to ask (the issuer), the fetch endpoint of that Entity, and the Entity Identifier of the subject of the Subordinate Statement. The issuer is normally the Immediate Superior of the subject of the Subordinate Statement.



#### Fetch Subordinate Statement Request

When client authentication is not used, the request MUST be an HTTP request using the GET method to a fetch endpoint with the following query parameter, encoded in `application/x-www-form-urlencoded` format. The request is made to the fetch endpoint of the specified issuer.

[]{.break}

sub
:   REQUIRED. The Entity Identifier of the subject for which the Subordinate Statement is being requested.
:   

When client authentication is used, the request MUST be an HTTP request using the POST method, with the parameters passed in the POST body.

The following is a non-normative example of an HTTP GET request for a Subordinate Statement from edugain.org about https://openid.sunet.se:

[]{#name-api-request-for-a-subordina}

<figure id="figure-28">
<div id="section-8.1.1-5.1" class="alignLeft art-text artwork">
<pre><code>GET /federation_fetch_endpoint?
sub=https%3A%2F%2Fopenid%2Esunet%2Ese HTTP/1.1
Host: edugain.org</code></pre>
</div>
<figcaption><a href="#figure-28" class="selfRef">Figure 28</a>: <a href="#name-api-request-for-a-subordina" class="selfRef">API Request for a Subordinate Statement</a></figcaption>
</figure>





#### Fetch Subordinate Statement Response

A successful response MUST use the HTTP status code 200 with the content type `application/entity-statement+jwt`, to make it clear that the response contains an Entity Statement. If it is an error response, it will be a JSON object and the content type MUST be `application/json`. If the fetch endpoint cannot provide data for the requested `sub` parameter, returning the `not_found` error code is RECOMMENDED. If the `sub` parameter references the Entity Identifier of the Issuing Entity, returning the `invalid_request` error code is RECOMMENDED. See more about error responses in [Section 8.9](#error_response){.auto .internal .xref}.

The following is a non-normative example of the JWT Claims Set for a fetch response:

[]{#name-fetch-response-jwt-claims-s}

<figure id="figure-29">
<div id="section-8.1.2-3.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;iss&quot;: &quot;https://edugain.org&quot;,
  &quot;sub&quot;: &quot;https://openid.sunet.se&quot;,
  &quot;exp&quot;: 1568397247,
  &quot;iat&quot;: 1568310847,
  &quot;source_endpoint&quot;: &quot;https://edugain.org/federation_fetch_endpoint&quot;,
  &quot;jwks&quot;: {
    &quot;keys&quot;: [
      {
        &quot;e&quot;: &quot;AQAB&quot;,
        &quot;kid&quot;: &quot;dEEtRjlzY3djcENuT01wOGxrZlkxb3RIQVJlMTY0...&quot;,
        &quot;kty&quot;: &quot;RSA&quot;,
        &quot;n&quot;: &quot;x97YKqc9Cs-DNtFrQ7_vhXoH9bwkDWW6En2jJ044yH...&quot;
      }
    ]
  },
  &quot;metadata&quot;:{
    &quot;federation_entity&quot;: {
        &quot;organization_name&quot;:&quot;SUNET&quot;
    }
  }
  &quot;metadata_policy&quot;: {
    &quot;openid_provider&quot;: {
      &quot;subject_types_supported&quot;: {
        &quot;value&quot;: [
          &quot;pairwise&quot;
        ]
      },
      &quot;token_endpoint_auth_methods_supported&quot;: {
        &quot;default&quot;: [
          &quot;private_key_jwt&quot;
        ],
        &quot;subset_of&quot;: [
          &quot;private_key_jwt&quot;,
          &quot;client_secret_jwt&quot;
        ],
        &quot;superset_of&quot;: [
          &quot;private_key_jwt&quot;
        ]
      }
    }
  }
}</code></pre>
</div>
<figcaption><a href="#figure-29" class="selfRef">Figure 29</a>: <a href="#name-fetch-response-jwt-claims-s" class="selfRef">Fetch Response JWT Claims Set</a></figcaption>
</figure>







### Subordinate Listings

The listing endpoint is exposed by Federation Entities acting as a Trust Anchor, Intermediate, or Trust Mark Issuer. The endpoint lists the Immediate Subordinates about which the Trust Anchor, Intermediate, or Trust Mark Issuer issues Entity Statements.

As a Trust Mark Issuer, the endpoint MAY list the Immediate Subordinates for which Trust Marks have been issued and are still valid, if the issuer exposing this endpoint supports Trust Mark filtering, as defined below.

In both cases, the list contained in the result MAY be a very large list.

The list endpoint location is published in the Entity's `federation_entity` metadata in the `federation_list_endpoint` parameter defined in [Section 5.1.1](#federation_entity){.auto .internal .xref}. This endpoint MUST be published directly in Entity Configuration metadata and not in Subordinate Statements.

The following example shows a tree of Entities belonging to the same federation including the Trust Anchor, Intermediate Entities, and Leaf Entities, discovered and collected through the Subordinate listing endpoints:

[]{#name-tree-of-entities-in-a-feder}

<figure id="figure-30">
<div id="section-8.2-6.1" class="alignLeft art-text artwork">
<pre><code>                       +----------------------+
                       |    Trust Anchor      |
                       +----------------------+
       +---------------+ Subordinate Listing  +--------------+
       |               +----------+-----------+              |
       |                          |                          |
       |                          |                          |
       |                          |                          |
       |                          |                          |
       |                          |                          |
+------v-------+       +----------v-----------+       +------v-------+
|     Leaf     |       |     Intermediate     |       |     Leaf     |
+--------------+       +----------------------+       +--------------+
                  +----+ Subordinate Listing  |
                  |    +------------+---------+
                  |                 |
                  |                 |
                  |                 |
       +----------v-----------+     |
       |     Intermediate     |     |
       +----------------------+     |
       | Subordinate Listing  |     |
       +-+---------+----------+     |
         |         |                |
         |         |                |
 +-------v--+     +v--------+    +--v------+
 |  Leaf    |     |  Leaf   |    |  Leaf   |
 +----------+     +---------+    +---------+</code></pre>
</div>
<figcaption><a href="#figure-30" class="selfRef">Figure 30</a>: <a href="#name-tree-of-entities-in-a-feder" class="selfRef">Tree of Entities in a Federation Collected through Subordinate Listing Endpoints</a></figcaption>
</figure>



#### Subordinate Listing Request

When client authentication is not used, the request MUST be an HTTP request using the GET method to a list endpoint with the following query parameters, encoded in `application/x-www-form-urlencoded` format.

[]{.break}

entity_type
:   OPTIONAL. The value of this parameter is an Entity Type Identifier. If the responder knows the Entity Types of its Immediate Subordinates, the result MUST be filtered to include only those that include the specified Entity Type. When multiple `entity_type` parameters are present, for example `entity_type=openid_provider&entity_type=openid_relying_party`, the result MUST be filtered to include all specified Entity Types. If the responder does not support this feature, it MUST use the HTTP status code 400 and the content type `application/json`, with the error code `unsupported_parameter`.
:   

trust_marked
:   OPTIONAL. Boolean. If the parameter `trust_marked` is present and set to `true`, the result contains only the Immediate Subordinates for which at least one Trust Mark has been issued and is still valid. If the responder does not support this feature, it MUST use the HTTP status code 400 and set the content type to `application/json`, with the error code `unsupported_parameter`.
:   

trust_mark_type
:   OPTIONAL. The value of this parameter is the identifier for the type of the Trust Mark. If the responder has issued Trust Marks with the specified Trust Mark type identifier, the list in the response is filtered to include only the Immediate Subordinates for which that Trust Mark type identifier has been issued and is still valid. If the responder does not support this feature, it MUST use the HTTP status code 400 and set the content type to `application/json`, with the error code `unsupported_parameter`.
:   

intermediate
:   OPTIONAL. Boolean. If the parameter `intermediate` is present and set to `true`, then if the responder knows whether its Immediate Subordinates are Intermediates or not, the result MUST be filtered accordingly. If the responder does not support this feature, it MUST use the HTTP status code 400 and the content type `application/json`, with the error code `unsupported_parameter`.
:   

When client authentication is used, the request MUST be an HTTP request using the POST method, with the parameters passed in the POST body.

The following is a non-normative example of an HTTP GET request for a list of Immediate Subordinates:

[]{#name-subordinate-listing-request-2}

<figure id="figure-31">
<div id="section-8.2.1-5.1" class="alignLeft art-text artwork">
<pre><code>GET /list HTTP/1.1
Host: openid.sunet.se</code></pre>
</div>
<figcaption><a href="#figure-31" class="selfRef">Figure 31</a>: <a href="#name-subordinate-listing-request-2" class="selfRef">Subordinate Listing Request</a></figcaption>
</figure>





#### Subordinate Listing Response

A successful response MUST use the HTTP status code 200 with the content type `application/json`, containing a JSON array with the known Entity Identifiers.

An error response is as defined in [Section 8.9](#error_response){.auto .internal .xref}.

The following is a non-normative example of a response containing the Immediate Subordinate Entities:

[]{#name-subordinate-listing-response}

<figure id="figure-32">
<div id="section-8.2.2-4.1" class="alignLeft art-text artwork">
<pre><code>200 OK
Content-Type: application/json

[
  &quot;https://ntnu.andreas.labs.uninett.no/&quot;,
  &quot;https://blackboard.ntnu.no/openid/callback&quot;,
  &quot;https://serviceprovider.andreas.labs.uninett.no/application17&quot;
]</code></pre>
</div>
<figcaption><a href="#figure-32" class="selfRef">Figure 32</a>: <a href="#name-subordinate-listing-response" class="selfRef">Subordinate Listing Response</a></figcaption>
</figure>







### Resolve Entity

An Entity MAY use a resolve endpoint to fetch Resolved Metadata and Trust Marks for an Entity. The resolver fetches the subject's Entity Configuration, assembles a Trust Chain that starts with the subject's Entity Configuration and ends with the specified Trust Anchor's Entity Configuration, verifies the Trust Chain, and then applies all the policies present in the Trust Chain to the subject's metadata.

The resolver MUST verify that all present Trust Marks with identifiers recognized within the Federation are active. The response set MUST include only verified Trust Marks.

The resolve endpoint location is published in the Entity's `federation_entity` metadata in the `federation_resolve_endpoint` parameter defined in [Section 5.1.1](#federation_entity){.auto .internal .xref}.



#### Resolve Request

When client authentication is not used, the request MUST be an HTTP request using the GET method to a resolve endpoint with the following query parameters, encoded in `application/x-www-form-urlencoded` format.

[]{.break}

sub
:   REQUIRED. The Entity Identifier of the Entity whose resolved data is requested.
:   

trust_anchor

:   REQUIRED. The Trust Anchor that the resolve endpoint MUST use when resolving the metadata. The value is an Entity identifier.

    The `trust_anchor` request parameter MAY occur multiple times, in which case, the resolver MAY return a successful resolve response using any of the Trust Anchor values provided.

:   

entity_type

:   OPTIONAL. A specific Entity Type to resolve. Its value is an Entity Type Identifier, as specified in [Section 5.1](#entity_types){.auto .internal .xref}. If this parameter is not present, then all Entity Types are returned.

    The `entity_type` request parameter MAY occur multiple times, in which case, data for each Entity Type whose Entity Type Identifier is in an `entity_type` parameter value is returned.

:   

When client authentication is used, the request MUST be an HTTP request using the POST method, with the parameters passed in the POST body.

The following is a non-normative example of a resolve request:

[]{#name-example-resolve-request}

<figure id="figure-33">
<div id="section-8.3.1-5.1" class="alignLeft art-text artwork">
<pre><code>GET /resolve?
sub=https%3A%2F%2Fop.example.it%2Fspid&amp;
entity_type=openid_provider&amp;
trust_anchor=https%3A%2F%2Fswamid.se HTTP/1.1
Host: openid.sunet.se</code></pre>
</div>
<figcaption><a href="#figure-33" class="selfRef">Figure 33</a>: <a href="#name-example-resolve-request" class="selfRef">Example Resolve Request</a></figcaption>
</figure>





#### Resolve Response

A successful response MUST use the HTTP status code 200 with the content type `application/resolve-response+jwt`, containing Resolved Metadata and verified Trust Marks.

The response is a signed JWT that is explicitly typed by setting the `typ` header parameter to `resolve-response+jwt` to prevent cross-JWT confusion, per Section 3.11 of [[RFC8725](#RFC8725){.cite .xref}]. Resolve responses without a `typ` header parameter or with a different `typ` value MUST be rejected. It is signed with a Federation Entity Key.

The resolve response JWT MUST include the `kid` (Key ID) header parameter with its value being the Key ID of the signing key used.

The resolve response JWT MUST return the Trust Chain from the subject to the Trust Anchor in its `trust_chain` parameter, sorted as shown in [Section 4](#trust_chain){.auto .internal .xref}.

The resolve response MAY also return the Trust Chain from its issuer to the Trust Anchor in the `trust_chain` JWS header parameter, as specified in [Section 4.3](#trust_chain_head_param){.auto .internal .xref}. When this is present, the Trust Anchor in the Trust Chain MUST match the Trust Anchor requested in the related request in the `trust_anchor` parameter.

An issuer that provides its Trust Chain within the resolve response makes it evident that it is part of the same federation as the subject of the response. Thus, when the Trust Chains of both the issuer and the subject are available and the Federation Historical Keys endpoint is provided by the Trust Anchor, the resolve response becomes a long-lived attestation; it can be always verified, even when the Federation Keys change in the future.

The response SHOULD contain the `aud` claim only if the requesting party is authenticated, in which case, the value MUST be the Entity Identifier of the requesting party and MUST NOT include any other values.

The claims in a resolve response are:

[]{.break}

iss
:   REQUIRED. String. Entity Identifier of the issuer of the resolve response.
:   

sub
:   REQUIRED. String. Entity Identifier of the subject of the resolve response.
:   

iat
:   REQUIRED. Number. Time when this resolution was issued. This is expressed as Seconds Since the Epoch, per [[RFC7519](#RFC7519){.cite .xref}].
:   

exp
:   REQUIRED. Number. Time when this resolution is no longer valid. This is expressed as Seconds Since the Epoch, per [[RFC7519](#RFC7519){.cite .xref}]. It MUST correspond to the `exp` value of the Trust Chain from which the resolve response was derived.
:   

metadata
:   REQUIRED. JSON object containing the resolved subject metadata, according to the requested `type` and expressed in the `metadata` format defined in [Section 3](#entity-statement){.auto .internal .xref}.
:   

trust_chain
:   REQUIRED. Array containing the sequence of Entity Statements that compose the Trust Chain, starting with the subject and ending with the selected Trust Anchor, sorted as shown in [Section 4](#trust_chain){.auto .internal .xref}.
:   

trust_marks
:   OPTIONAL. Array of objects, each representing a Trust Mark, as defined in [Section 3](#entity-statement){.auto .internal .xref}. Only valid Trust Marks that have been issued by Trust Mark issuers trusted by the Trust Anchor to issue such Trust Marks MAY appear in the resolver response.
:   

Additional claims MAY be defined and used in conjunction with the claims above.

An error response is as defined in [Section 8.9](#error_response){.auto .internal .xref}.

The following is a non-normative example of the JWT Claims Set for a resolve response:

[]{#name-resolve-response-jwt-claims}

<figure id="figure-34">
<div id="section-8.3.2-13.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;iss&quot;: &quot;https://resolver.spid.gov.it/&quot;,
  &quot;sub&quot;: &quot;https://op.example.it/spid/&quot;,
  &quot;iat&quot;: 1516239022,
  &quot;exp&quot;: 1516298022,
  &quot;metadata&quot;: {
    &quot;openid_provider&quot;: {
      &quot;contacts&quot;: [&quot;legal@example.it&quot;, &quot;technical@example.it&quot;],
      &quot;logo_uri&quot;:
        &quot;https://op.example.it/static/img/op-logo.svg&quot;,
      &quot;op_policy_uri&quot;:
        &quot;https://op.example.it/en/about-the-website/legal-information&quot;,
      &quot;federation_registration_endpoint&quot;:&quot;https://op.example.it/spid/fedreg&quot;,
      &quot;authorization_endpoint&quot;:
        &quot;https://op.example.it/spid/authorization&quot;,
      &quot;token_endpoint&quot;: &quot;https://op.example.it/spid/token&quot;,
      &quot;response_types_supported&quot;: [
        &quot;code&quot;,
        &quot;code id_token&quot;,
        &quot;token&quot;
      ],
      &quot;grant_types_supported&quot;: [
        &quot;authorization_code&quot;,
        &quot;urn:ietf:params:oauth:grant-type:jwt-bearer&quot;
      ],
      &quot;subject_types_supported&quot;: [&quot;pairwise&quot;],
      &quot;id_token_signing_alg_values_supported&quot;: [&quot;RS256&quot;],
      &quot;issuer&quot;: &quot;https://op.example.it/spid&quot;,
      &quot;jwks&quot;: {
        &quot;keys&quot;: [
          {
            &quot;kty&quot;: &quot;RSA&quot;,
            &quot;use&quot;: &quot;sig&quot;,
            &quot;n&quot;: &quot;1Ta-sE ...&quot;,
            &quot;e&quot;: &quot;AQAB&quot;,
            &quot;kid&quot;: &quot;FANFS3YnC9tjiCaivhWLVUJ3AxwGGz_98uRFaqMEEs&quot;
          }
        ]
      }
    }
  },
  &quot;trust_marks&quot;: [
    {&quot;trust_mark_type&quot;: &quot;https://www.spid.gov.it/certification/op/&quot;,
     &quot;trust_mark&quot;:
       &quot;eyJ0eXAiOiJ0cnVzdC1tYXJrK2p3dCIsImFsZyI6IlJTMjU2Iiwia2lkIjoiOH
        hzdUtXaVZmd1NnSG9mMVRlNE9VZGN5NHE3ZEpyS2ZGUmxPNXhoSElhMCJ9.
        eyJpc3MiOiJodHRwczovL3d3dy5hZ2lkLmdvdi5pdCIsInN1YiI6Imh0dHBzOi
        8vb3AuZXhhbXBsZS5pdC9zcGlkLyIsImlhdCI6MTU3OTYyMTE2MCwidHJ1c3Rf
        bWFya190eXBlIjoiaHR0cHM6Ly93d3cuc3BpZC5nb3YuaXQvY2VydGlmaWNhdG
        lvbi9vcC8iLCJsb2dvX3VyaSI6Imh0dHBzOi8vd3d3LmFnaWQuZ292Lml0L3Ro
        ZW1lcy9jdXN0b20vYWdpZC9sb2dvLnN2ZyIsInJlZiI6Imh0dHBzOi8vZG9jcy
        5pdGFsaWEuaXQvaXRhbGlhL3NwaWQvc3BpZC1yZWdvbGUtdGVjbmljaGUtb2lk
        Yy9pdC9zdGFiaWxlL2luZGV4Lmh0bWwifQ.
        xyz-PDQ_...&quot;
    }
  ],
  &quot;trust_chain&quot; : [
    &quot;eyJhbGciOiJSUzI1NiIsImtpZCI6Ims1NEhRdERpYnlHY3M5WldWTWZ2aUhm ...&quot;,
    &quot;eyJhbGciOiJSUzI1NiIsImtpZCI6IkJYdmZybG5oQU11SFIwN2FqVW1BY0JS ...&quot;,
    &quot;eyJhbGciOiJSUzI1NiIsImtpZCI6IkJYdmZybG5oQU11SFIwN2FqVW1BY0JS ...&quot;
  ]
}</code></pre>
</div>
<figcaption><a href="#figure-34" class="selfRef">Figure 34</a>: <a href="#name-resolve-response-jwt-claims" class="selfRef">Resolve Response JWT Claims Set</a></figcaption>
</figure>





#### Trust Considerations

The basic assumption of this specification is that an Entity should have direct trust in no one except the Trust Anchor and its own capabilities. However, Entities MAY establish a kind of transitive trust in other Entities. For example, the Trust Anchor states who its Immediate Subordinates are, and Entities MAY choose to trust them. If a party uses the resolve service of another Entity to obtain federation data, it is trusting the resolver to perform validation of the cryptographically protected metadata correctly and to provide it with authentic results.







### Trust Mark Status

This enables determining whether a Trust Mark Instance that has been issued to an Entity is still active. The query MUST be sent to the Trust Mark Issuer.

The Trust Mark Status endpoint location is published in the Entity's `federation_entity` metadata in the `federation_trust_mark_status_endpoint` parameter defined in [Section 5.1.1](#federation_entity){.auto .internal .xref}. This endpoint MUST be published directly in Entity Configuration metadata and not in Subordinate Statements.



#### Trust Mark Status Request

The request MUST be an HTTP request using the POST method to a Trust Mark Status endpoint with the following parameter, encoded in `application/x-www-form-urlencoded` format.

[]{.break}

trust_mark
:   REQUIRED. The Trust Mark to be validated.
:   

When client authentication is used, the request MUST be an HTTP request using the POST method, with the parameters passed in the POST body.

The following is a non-normative example of a Trust Mark Status request:

[]{#name-trust-mark-status-request-2}

<figure id="figure-35">
<div id="section-8.4.1-5.1" class="alignLeft art-text artwork">
<pre><code>POST /federation_trust_mark_status_endpoint HTTP/1.1
Host: op.example.org
Content-Type: application/x-www-form-urlencoded

trust_mark=eyJ0eXAiOiJ0cnVzdC1tYXJrK2p3dCIsImFsZyI6 ...</code></pre>
</div>
<figcaption><a href="#figure-35" class="selfRef">Figure 35</a>: <a href="#name-trust-mark-status-request-2" class="selfRef">Trust Mark Status Request</a></figcaption>
</figure>





#### Trust Mark Status Response

A successful response MUST use the HTTP status code 200 with the content type `application/trust-mark-status-response+jwt`, containing a signed JWT that is a Trust Mark Status Response.

The Trust Mark Status Response is a signed JWT that is explicitly typed by setting the `typ` header parameter to `trust-mark-status-response+jwt` to prevent cross-JWT confusion, per Section 3.11 of [[RFC8725](#RFC8725){.cite .xref}]. Trust Mark Status Responses without a `typ` header parameter or with a different `typ` value MUST be rejected. It is signed with a Federation Entity Key.

The Trust Mark Status Response JWT MUST include the `kid` (Key ID) header parameter with its value being the Key ID of the signing key used.

The JWT Claims Set of the Trust Mark Status JWT is a JSON object containing the following claims:

[]{.break}

iss
:   REQUIRED. String. Entity Identifier of the issuer of the Trust Mark Status JWT.
:   

iat
:   REQUIRED. Number. Time when this Trust Mark Status JWT was issued. This is expressed as Seconds Since the Epoch, per [[RFC7519](#RFC7519){.cite .xref}].
:   

trust_mark
:   REQUIRED. String. The Trust Mark JWT that this status response is about.
:   

status

:   REQUIRED. Case-sensitive string value indicating the status of the Trust Mark. Values defined by this specification are:

    []{.break}

    active
    :   The Trust Mark is active
    :   

    expired
    :   The Trust Mark has expired
    :   

    revoked
    :   The Trust Mark was revoked
    :   

    invalid
    :   Signature validation failed or another error was detected
    :   

:   

:   Additional status values MAY be defined and used in addition to those above.
:   

Additional Trust Mark Status claims MAY be defined and used in addition to the one above.

The following is a non-normative example of the JWT Claims Set for a response with the status `active`:

[]{#name-active-trust-mark-status-re}

<figure id="figure-36">
<div id="section-8.4.2-8.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;iss&quot;: &quot;https://www.agid.gov.it&quot;,
  &quot;trust_mark&quot;: &quot;eyJ0eXAiOiJ0cnVzdC1tYXJrK2p3dCIsImFsZyI6IlJTMjU2Iiwia2
    lkIjoia29zR20yd3VaaDlER21OeEF0a3VPNDBwUGpwTDMtakNmMU4tcVBPLVllVSJ9.
    eyJpc3MiOiJodHRwczovL3d3dy5hZ2lkLmdvdi5pdCIsInN1YiI6Imh0dHBzOi8
    vcnAuZXhhbXBsZS5pdC9zcGlkIiwiaWF0IjoxNTc5NjIxMTYwLCJ0cnVzdF9tYX
    JrX3R5cGUiOiJodHRwczovL3d3dy5zcGlkLmdvdi5pdC9jZXJ0aWZpY2F0aW9uL
    3JwIiwibG9nb191cmkiOiJodHRwczovL3d3dy5hZ2lkLmdvdi5pdC90aGVtZXMv
    Y3VzdG9tL2FnaWQvbG9nby5zdmciLCJyZWYiOiJodHRwczovL2RvY3MuaXRhbGl
    hLml0L2RvY3Mvc3BpZC1jaWUtb2lkYy1kb2NzL2l0L3ZlcnNpb25lLWNvcnJlbn
    RlLyJ9.
    L_pSh1InEiFAcs3E-1HBM7fNZYwF5ru3UGA_8yc80dGS3sszfA_sbj4AoW_zAJW
    QBdZpjxnHBBmybYXFrfZBcqxcedsrvUYrmbt1nPYxbUE54fRRoZK-sJmVqh1GzS
    an5nOmkxuAtMinU8k_-aWnPWj83sYe2AzT2mMgkXiz8zhda3jZm8hoxZ4jR6B0Y
    AvbMlq2pPWO5OWKdZhiFRMSprwh0GYluQkK0j1aLNMGXD3keMJd2zEoWX9D7w2f
    XShAA48W3cNhuXyBVnCoum1K4IWK3s_fx4nIkp6W-V4jCBOpxp7Yo8LZ30o_xpE
    OzGTIECGWVR86azOAlwVC8XSiAA&quot;,
  &quot;iat&quot;: 1759897995,
  &quot;status&quot;: &quot;active&quot;
}</code></pre>
</div>
<figcaption><a href="#figure-36" class="selfRef">Figure 36</a>: <a href="#name-active-trust-mark-status-re" class="selfRef">Active Trust Mark Status Response JWT Claims Set</a></figcaption>
</figure>

The following is a non-normative example of the JWT Claims Set for a response with the status `revoked`:

[]{#name-revoked-trust-mark-status-r}

<figure id="figure-37">
<div id="section-8.4.2-10.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;iss&quot;: &quot;https://www.agid.gov.it&quot;,
  &quot;trust_mark&quot;: &quot;eyJ0eXAiOiJ0cnVzdC1tYXJrK2p3dCIsImFsZyI6IlJTMjU2Iiwia2
    lkIjoia29zR20yd3VaaDlER21OeEF0a3VPNDBwUGpwTDMtakNmMU4tcVBPLVllVSJ9.
    eyJpc3MiOiJodHRwczovL3d3dy5hZ2lkLmdvdi5pdCIsInN1YiI6Imh0dHBzOi8
    vcnAuZXhhbXBsZS5pdC9zcGlkIiwiaWF0IjoxNTc5NjIxMTYwLCJ0cnVzdF9tYX
    JrX3R5cGUiOiJodHRwczovL3d3dy5zcGlkLmdvdi5pdC9jZXJ0aWZpY2F0aW9uL
    3JwIiwibG9nb191cmkiOiJodHRwczovL3d3dy5hZ2lkLmdvdi5pdC90aGVtZXMv
    Y3VzdG9tL2FnaWQvbG9nby5zdmciLCJyZWYiOiJodHRwczovL2RvY3MuaXRhbGl
    hLml0L2RvY3Mvc3BpZC1jaWUtb2lkYy1kb2NzL2l0L3ZlcnNpb25lLWNvcnJlbn
    RlLyJ9.
    L_pSh1InEiFAcs3E-1HBM7fNZYwF5ru3UGA_8yc80dGS3sszfA_sbj4AoW_zAJW
    QBdZpjxnHBBmybYXFrfZBcqxcedsrvUYrmbt1nPYxbUE54fRRoZK-sJmVqh1GzS
    an5nOmkxuAtMinU8k_-aWnPWj83sYe2AzT2mMgkXiz8zhda3jZm8hoxZ4jR6B0Y
    AvbMlq2pPWO5OWKdZhiFRMSprwh0GYluQkK0j1aLNMGXD3keMJd2zEoWX9D7w2f
    XShAA48W3cNhuXyBVnCoum1K4IWK3s_fx4nIkp6W-V4jCBOpxp7Yo8LZ30o_xpE
    OzGTIECGWVR86azOAlwVC8XSiAA&quot;,
  &quot;iat&quot;: 1759898057,
  &quot;status&quot;: &quot;revoked&quot;
}</code></pre>
</div>
<figcaption><a href="#figure-37" class="selfRef">Figure 37</a>: <a href="#name-revoked-trust-mark-status-r" class="selfRef">Revoked Trust Mark Status Response JWT Claims Set</a></figcaption>
</figure>

An error response to a Trust Mark Status request is as defined in [Section 8.9](#error_response){.auto .internal .xref}.

If the Trust Mark Issuer receives a request about the status of an unknown Trust Mark, something it did not issue or is not aware of, it MUST respond with an HTTP status code 404 (Not found).







### Trust Marked Entities Listing

The Trust Marked Entities Listing endpoint is exposed by Trust Mark Issuers and lists all the Entities for which Trust Marks have been issued and are still valid.

The Trust Marked Entities Listing endpoint location is published in the Entity's `federation_entity` metadata in the `federation_trust_mark_list_endpoint` parameter defined in [Section 5.1.1](#federation_entity){.auto .internal .xref}.



#### Trust Marked Entities Listing Request

When client authentication is not used, the request MUST be an HTTP request using the GET method to a list endpoint with the following query parameters, encoded in `application/x-www-form-urlencoded` format.

[]{.break}

trust_mark_type
:   REQUIRED. Identifier for the type of the Trust Mark. If the responder has issued Trust Marks with the specified Trust Mark type identifier, the list in the response is filtered to include only the Entities for which that Trust Mark type identifier has been issued and is still valid.
:   

sub
:   OPTIONAL. The Entity Identifier of the Entity to which the Trust Mark was issued. The list obtained in the response MUST be filtered to only the Entity matching this value.
:   

When client authentication is used, the request MUST be an HTTP request using the POST method, with the parameters passed in the POST body.

The following is a non-normative example of an HTTP GET request for a list of Trust Marked Entities:

[]{#name-trust-marked-entities-listing}

<figure id="figure-38">
<div id="section-8.5.1-5.1" class="alignLeft art-text artwork">
<pre><code>GET /trust_marked_list?trust_mark_type=https%3A%2F%2Ffederation.example.org%2Fopenid_relying_party%2Fprivate%2Funder-age HTTP/1.1
Host: trust-mark-issuer.example.org</code></pre>
</div>
<figcaption><a href="#figure-38" class="selfRef">Figure 38</a>: <a href="#name-trust-marked-entities-listing" class="selfRef">Trust Marked Entities Listing Request</a></figcaption>
</figure>





#### Trust Marked Entities Listing Response

A successful response MUST use the HTTP status code 200 with the content type `application/json`, containing a JSON array with Entity Identifiers.

An error response is as defined in [Section 8.9](#error_response){.auto .internal .xref}.

The following is a non-normative example of a response, containing the Trust Marked Entities:

[]{#name-trust-marked-entities-listing-r}

<figure id="figure-39">
<div id="section-8.5.2-4.1" class="alignLeft art-text artwork">
<pre><code>200 OK
Content-Type: application/json

[
  &quot;https://blackboard.ntnu.no/openid/rp&quot;,
  &quot;https://that-rp.example.org&quot;
]</code></pre>
</div>
<figcaption><a href="#figure-39" class="selfRef">Figure 39</a>: <a href="#name-trust-marked-entities-listing-r" class="selfRef">Trust Marked Entities Listing Response</a></figcaption>
</figure>







### Trust Mark Endpoint

The Trust Mark endpoint is exposed by a Trust Mark Issuer to provide Trust Marks to subjects.

The Trust Mark endpoint location is published in the Entity's `federation_entity` metadata in the `federation_trust_mark_endpoint` parameter defined in [Section 5.1.1](#federation_entity){.auto .internal .xref}.



#### Trust Mark Request

When client authentication is not used, the request MUST be an HTTP request using the GET method with the following query parameters, encoded in `application/x-www-form-urlencoded` format.

[]{.break}

trust_mark_type
:   REQUIRED. Identifier for the type of the Trust Mark.
:   

sub
:   REQUIRED. The Entity Identifier of the Entity to which the Trust Mark is issued.
:   

When client authentication is used, the request MUST be an HTTP request using the POST method, with the parameters passed in the POST body. The Trust Mark endpoint MAY choose to allow authenticated requests from clients that are not the Trust Mark subject, as indicated by the `sub` parameter. An example use case is to let a Federation Entity retrieve the Trust Mark for another Entity.

The following is a non-normative example of an HTTP request for a Trust Mark with a specific Trust Mark type identifier and subject:

[]{#name-trust-mark-request-2}

<figure id="figure-40">
<div id="section-8.6.1-5.1" class="alignLeft art-text artwork">
<pre><code>GET /trust_mark?trust_mark_type=https%3A%2F%2Fwww.spid.gov.it%2Fcertification%2Frp&amp;sub=https%3A%2F%2Frp.example.it%2Fspid HTTP/1.1
Host: tuber.cert.example.org</code></pre>
</div>
<figcaption><a href="#figure-40" class="selfRef">Figure 40</a>: <a href="#name-trust-mark-request-2" class="selfRef">Trust Mark Request</a></figcaption>
</figure>





#### Trust Mark Response

A successful response MUST use the HTTP status code 200 with the content type `application/trust-mark+jwt`, containing the Trust Mark.

If the specified Entity does not have the specified Trust Mark, the response is an error response and MUST use the HTTP status code 404.

The following is a non-normative example of a response, containing the Trust Mark for the specified Entity (with line wraps within values for display purposes only):

[]{#name-trust-mark-response-2}

<figure id="figure-41">
<div id="section-8.6.2-4.1" class="alignLeft art-text artwork">
<pre><code>200 OK
Content-Type: application/trust-mark+jwt

eyJ0eXAiOiJ0cnVzdC1tYXJrK2p3dCIsImFsZyI6IlJTMjU2Iiwia2lkIjoia29zR20yd3Va
aDlER21OeEF0a3VPNDBwUGpwTDMtakNmMU4tcVBPLVllVSJ9.
eyJpc3MiOiJodHRwczovL3d3dy5hZ2lkLmdvdi5pdCIsInN1YiI6Imh0dHBzOi8vcnAuZXhh
bXBsZS5pdC9zcGlkIiwiaWF0IjoxNTc5NjIxMTYwLCJ0cnVzdF9tYXJrX3R5cGUiOiJodHRw
czovL3d3dy5zcGlkLmdvdi5pdC9jZXJ0aWZpY2F0aW9uL3JwIiwibG9nb191cmkiOiJodHRw
czovL3d3dy5hZ2lkLmdvdi5pdC90aGVtZXMvY3VzdG9tL2FnaWQvbG9nby5zdmciLCJyZWYi
OiJodHRwczovL2RvY3MuaXRhbGlhLml0L2RvY3Mvc3BpZC1jaWUtb2lkYy1kb2NzL2l0L3Zl
cnNpb25lLWNvcnJlbnRlLyJ9.
L_pSh1InEiFAcs3E-1HBM7fNZYwF5ru3UGA_8yc80dGS3sszfA_sbj4AoW_zAJWQBdZpjxnH
BBmybYXFrfZBcqxcedsrvUYrmbt1nPYxbUE54fRRoZK-sJmVqh1GzSan5nOmkxuAtMinU8k_
-aWnPWj83sYe2AzT2mMgkXiz8zhda3jZm8hoxZ4jR6B0YAvbMlq2pPWO5OWKdZhiFRMSprwh
0GYluQkK0j1aLNMGXD3keMJd2zEoWX9D7w2fXShAA48W3cNhuXyBVnCoum1K4IWK3s_fx4nI
kp6W-V4jCBOpxp7Yo8LZ30o_xpEOzGTIECGWVR86azOAlwVC8XSiAA</code></pre>
</div>
<figcaption><a href="#figure-41" class="selfRef">Figure 41</a>: <a href="#name-trust-mark-response-2" class="selfRef">Trust Mark Response</a></figcaption>
</figure>







### Federation Historical Keys Endpoint

Each Federation Entity MAY publish its previously used Federation Entity Keys at the historical keys endpoint defined in [Section 5.1.1](#federation_entity){.auto .internal .xref}. The purpose of this endpoint is to provide the list of keys previously used by the Federation Entity to provide non-repudiation of statements signed by it after key rotation. This endpoint also discloses the reason for the retraction of the keys and whether they were expired or revoked, including the reason for the revocation.

Note that an expired key can be later additionally marked as revoked, to indicate a key compromise event discovered after the expiration of the key.

The publishing of the historical keys guarantees that Trust Chains will remain verifiable and usable as inputs to trust decisions after the key expiration, unless the key becomes revoked for a security reason.



#### Federation Historical Keys Request

When client authentication is not used, the request MUST be an HTTP request using the GET method to the federation historical keys endpoint.

When client authentication is used, the request MUST be an HTTP request using the POST method, with the client authentication parameters passed in the POST body.

The following is a non-normative example of a historical keys request:

[]{#name-federation-historical-keys-re}

<figure id="figure-42">
<div id="section-8.7.1-4.1" class="alignLeft art-text artwork">
<pre><code>GET /federation_historical_keys HTTP/1.1
Host: trust-anchor.example.com</code></pre>
</div>
<figcaption><a href="#figure-42" class="selfRef">Figure 42</a>: <a href="#name-federation-historical-keys-re" class="selfRef">Federation Historical Keys Request</a></figcaption>
</figure>





#### Federation Historical Keys Response

The response is a signed JWK Set containing the historical keys. It is signed with a Federation Entity Key. A signed JWK Set is a signed JWT with a JWK Set [[RFC7517](#RFC7517){.cite .xref}] as its payload. A successful response MUST use the HTTP status code 200 with the content type `application/jwk-set+jwt`.

Historical keys JWTs are explicitly typed by setting the `typ` header parameter to `jwk-set+jwt` to prevent cross-JWT confusion, per Section 3.11 of [[RFC8725](#RFC8725){.cite .xref}]. Historical keys JWTs without a `typ` header parameter or with a different `typ` value MUST be rejected.

Historical keys JWTs MUST include the `kid` (Key ID) header parameter with its value being the Key ID of the signing key used.

The claims in a historical keys JWT are:

[]{.break}

iss
:   REQUIRED. String. The Entity's Entity Identifier.
:   

iat
:   REQUIRED. Number. Time when this historical keys JWT was issued. This is expressed as Seconds Since the Epoch, per [[RFC7519](#RFC7519){.cite .xref}].
:   

keys

:   REQUIRED. Array of JSON objects containing the signing keys for the Entity in JWK format.

    JWKs in the `keys` claim use the following parameters:

    []{.break}

    kid
    :   REQUIRED. Parameter used to match a specific key. It is RECOMMENDED that the Key ID be the JWK Thumbprint [[RFC7638](#RFC7638){.cite .xref}] of the key using the SHA-256 hash function.
    :   

    iat
    :   OPTIONAL. Number. Time when this key was issued. This is expressed as Seconds Since the Epoch, per [[RFC7519](#RFC7519){.cite .xref}].
    :   

    exp
    :   REQUIRED. Number. Expiration time for the key. This is expressed as Seconds Since the Epoch, per [[RFC7519](#RFC7519){.cite .xref}], After this time the key MUST NOT be considered valid.
    :   

    revoked

    :   OPTIONAL. JSON object that contains the properties of the revocation, as defined below:

        []{.break}

        revoked_at
        :   REQUIRED. Time when the key was revoked or must be considered revoked, using the time format defined for the `iat` claim in [[RFC7519](#RFC7519){.cite .xref}].
        :   

        reason
        :   OPTIONAL. String that identifies the reason for the key revocation, as defined in [Section 8.7.3](#hist-keys-reasons){.auto .internal .xref}.
        :   

        Additional members of the `revoked` object MAY be defined and used.

    :   

:   

Additional claims MAY be defined and used in conjunction with the claims above.

JWKs in the `keys` claim MAY also contain the `nbf` parameter. For use in Historical Keys, `iat` and `exp` are sufficient to establish the key lifetime, making `nbf` typically superfluous; however, it is registered for use by profiles that may choose to issue keys that do not immediately become valid at the time of issuance. Its definition is:

[]{.break}

nbf
:   OPTIONAL. Time before which the key is not valid, using the time format defined for the `nbf` claim in [[RFC7519](#RFC7519){.cite .xref}].
:   

The following is a non-normative example of the JWT Claims Set for a historical keys response:

[]{#name-federation-historical-keys-resp}

<figure id="figure-43">
<div id="section-8.7.2-10.1" class="alignLeft art-text artwork">
<pre><code>{
    &quot;iss&quot;: &quot;https://trust-anchor.federation.example.com&quot;,
    &quot;iat&quot;: 123972394272,
    &quot;keys&quot;:
        [
            {
                &quot;kty&quot;:&quot;RSA&quot;,
                &quot;n&quot;:&quot;5s4qi …&quot;,
                &quot;e&quot;:&quot;AQAB&quot;,
                &quot;kid&quot;:&quot;2HnoFS3YnC9tjiCaivhWLVUJ3AxwGGz_98uRFaqMEEs&quot;,
                &quot;iat&quot;: 123972394872,
                &quot;exp&quot;: 123974395972
            },
            {
                &quot;kty&quot;:&quot;RSA&quot;,
                &quot;n&quot;:&quot;ng5jr …&quot;,
                &quot;e&quot;:&quot;AQAB&quot;,
                &quot;kid&quot;:&quot;8KnoFS3YnC9tjiCaivhWLVUJ3AxwGGz_98uRFaqMJJr&quot;,
                &quot;iat&quot;: 123972394872,
                &quot;exp&quot;: 123974394972,
                &quot;revoked&quot;: {
                  &quot;revoked_at&quot;: 123972495172,
                  &quot;reason&quot;: &quot;compromised&quot;,
                }
            }
        ]
}</code></pre>
</div>
<figcaption><a href="#figure-43" class="selfRef">Figure 43</a>: <a href="#name-federation-historical-keys-resp" class="selfRef">Federation Historical Keys Response JWT Claims Set</a></figcaption>
</figure>





#### Federation Historical Keys Revocation Reasons

Federation Entities are strongly encouraged to use a meaningful `reason` value when indicating the revocation reason for a Federation Entity Key. The `reason` MAY be omitted instead of using the `unspecified` value.

Below is the definition of the reason values.

The following table defines Federation Entity Keys revocation reasons. These reasons are inspired by Section 5.3.1 of [[RFC5280](#RFC5280){.cite .xref}].

[]{#name-federation-entity-keys-revo}

  Reason        Description
  ------------- ----------------------------------------------------------
  unspecified   General or unspecified reason for the JWK status change.
  compromised   The private key is believed to have been compromised.
  superseded    The JWK is no longer active.

  : [Table 2](#table-2): [Federation Entity Keys Revocation Reasons.](#name-federation-entity-keys-revo) {#table-2}

A federation MAY specify and utilize additional reasons depending on the trust or security framework in use.





#### Rationale for the Federation Historical Keys Endpoint

The Federation Historical Keys endpoint solves the problem of verifying historical Trust Chains when the Federation Entity Keys have changed, either due to expiration or revocation.

The Federation Historical Keys endpoint publishes the list of public keys used in the past by the Entity. These keys are needed to verify Trust Chains created in the past with Entity keys no longer published in the Entity's Entity Configuration.

The Federation Historical Keys endpoint response contains a signed JWT that attests to all the expired and revoked Entity keys.

Based on the attributes contained in the Entity Statements that form a Trust Chain, it MAY also be possible to verify the non-federation public keys used in the past by Leaf Entities for signature operations for OpenID Connect requests and responses. For example, an Entity Statement issued for a Leaf MAY also include the `jwks` claim for the Leaf's Entity Types, in its `metadata` or `metadata_policy` claims.

A simple example: In the following Trust Chain, the Federation Intermediate attests to the Leaf's OpenID Connect RP `jwks` in the Subordinate Statement issued about the Leaf. The result is a Trust Chain that contains the Leaf's OpenID Connect RP JWK Set, needed to verify historical signature on Request Objects and any other signed JWT issued by the Leaf as an RP. This example Trust Chain contains:

1.  
    an Entity Configuration about the RP published by the RP,
    

2.  
    a Subordinate Statement about the RP published by Organization A, with the claim `jwks` contained in `metadata` or `metadata_policy` attesting the Leaf's OpenID Connect RP `jwks`, and
    

3.  
    a Subordinate Statement about Organization A published by Federation F.
    







### Client Authentication at Federation Endpoints

Client authentication is not used at any of the federation endpoints, by default. Federations can choose to make client authentication OPTIONAL, REQUIRED, and/or not allowed at particular federation endpoints.

Client authentication with `private_key_jwt` is the default client authentication method to federation endpoints when client authentication is supported. This client authentication method is described in Section 9 of [OpenID Connect Core 1.0](#OpenID.Core){.internal .xref} [[OpenID.Core](#OpenID.Core){.cite .xref}]. The client authentication JWT MUST be signed with a Federation Entity Key. The audience of the JWT MUST be the Entity Identifier of the Entity whose federation endpoint is being authenticated to. The endpoint MUST NOT accept JWTs containing audience values other than its Entity Identifier. When client authentication is used, the request MUST be an HTTP request using the POST method, with the client authentication and endpoint request parameters passed in the POST body. Federations can choose to also use other client authentication methods.



#### Client Authentication Metadata for Federation Endpoints

Like other OAuth and OpenID endpoints supporting client authentication, this specification defines metadata parameters saying which client authentication methods are supported for each endpoint. These largely parallel the `token_endpoint_auth_methods_supported` metadata value defined in Section 3 of [OpenID Connect Discovery 1.0](#OpenID.Discovery){.internal .xref} [[OpenID.Discovery](#OpenID.Discovery){.cite .xref}].

Specifically, for each of the federation endpoints defined in [Section 5.1.1](#federation_entity){.auto .internal .xref}, parameters named `*_auth_methods` are defined, where the `*` represents the federation endpoint names `federation_fetch_endpoint`, `federation_list_endpoint`, ..., `federation_historical_keys_endpoint`.

The `*_auth_methods` metadata parameters list supported client authentication methods for these endpoints, just as `token_endpoint_auth_methods_supported` does for the Token Endpoint. In addition, the value `none` MAY be used to indicate that client authentication is not required at the endpoint.

So, for instance, this metadata declaration states that requests authenticated with `private_key_jwt` are REQUIRED at the `federation_trust_mark_endpoint`:

[]{#name-declaring-that-client-authe}

<figure id="figure-44">
<div id="section-8.8.1-5.1" class="alignLeft art-text artwork">
<pre><code>&quot;federation_trust_mark_endpoint_auth_methods&quot;: [&quot;private_key_jwt&quot;]</code></pre>
</div>
<figcaption><a href="#figure-44" class="selfRef">Figure 44</a>: <a href="#name-declaring-that-client-authe" class="selfRef">Declaring that client authentication is REQUIRED at an endpoint</a></figcaption>
</figure>

If omitted, the default value for these methods is `["none"]`, indicating that only unauthenticated requests are accepted.

The `endpoint_auth_signing_alg_values_supported` metadata parameter lists supported client authentication signing algorithms supported for these endpoints, just as `token_endpoint_auth_signing_alg_values_supported` does for the Token Endpoint.







### Error Responses

If the request was malformed or an error occurred during the processing of the request, the response body SHOULD be a JSON object with the content type `application/json`. In compliance with [[RFC6749](#RFC6749){.cite .xref}], the following standardized error format SHOULD be used:

[]{.break}

error

:   REQUIRED. Error codes in the IANA "OAuth Extensions Error Registry" [[IANA.OAuth.Parameters](#IANA.OAuth.Parameters){.cite .xref}] MAY be used. This specification also defines the following error codes:

    []{.break}

    invalid_request
    :   The request is incomplete or does not comply with current specifications. The HTTP response status code SHOULD be 400 (Bad Request).
    :   

    invalid_client
    :   The Client cannot be authorized or is not a valid participant of the federation. The HTTP response status code SHOULD be 401 (Unauthorized).
    :   

    invalid_issuer
    :   The endpoint cannot serve the requested issuer. The HTTP response status code SHOULD be 404 (Not Found).
    :   

    invalid_subject
    :   The endpoint cannot serve the requested subject. The HTTP response status code SHOULD be 404 (Not Found).
    :   

    invalid_trust_anchor
    :   The Trust Anchor cannot be found or used. The HTTP response status code SHOULD be 404 (Not Found).
    :   

    invalid_trust_chain
    :   The Trust Chain cannot be validated. The HTTP response status code SHOULD be 400 (Bad Request).
    :   

    invalid_metadata
    :   Metadata or Metadata Policy values are invalid or conflict. The HTTP response status code SHOULD be 400 (Bad Request).
    :   

    not_found
    :   The requested Entity Identifier cannot be found. The HTTP response status code SHOULD be 404 (Not Found).
    :   

    server_error
    :   The server encountered an unexpected condition that prevented it from fulfilling the request. The HTTP response status code SHOULD be one in the 5xx range, like 500 (Internal Server Error).
    :   

    temporarily_unavailable
    :   The server hosting the federation endpoint is currently unable to handle the request due to temporary overloading or maintenance. The HTTP response status code SHOULD be 503 (Service Unavailable).
    :   

    unsupported_parameter
    :   The server does not support the requested parameter. The HTTP response status code SHOULD be 400 (Bad Request).
    :   

:   

error_description
:   REQUIRED. Human-readable text providing additional information used to assist the developer in understanding the error that occurred.
:   

The following is a non-normative example of an error response:

[]{#name-example-error-response}

<figure id="figure-45">
<div id="section-8.9-4.1" class="alignLeft art-text artwork">
<pre><code>400 Bad request
Content-Type: application/json

{
  &quot;error&quot;: &quot;invalid_request&quot;,
  &quot;error_description&quot;:
    &quot;Required request parameter [sub] was missing.&quot;
}</code></pre>
</div>
<figcaption><a href="#figure-45" class="selfRef">Figure 45</a>: <a href="#name-example-error-response" class="selfRef">Example Error Response</a></figcaption>
</figure>







## Obtaining Federation Entity Configuration Information

The Entity Configuration of all Trust Anchor and Intermediate Federation Entities MUST be published at their configuration endpoint and the Entity Configuration for Leaf Entities SHOULD be published there. Its location is determined by concatenating the string `/.well-known/openid-federation` to the Entity Identifier (which MUST use the `https` scheme and contain a host component and MAY also contain port and path components). For instance, the configuration endpoint for the Entity Identifier `https://entity.example` is the URL `https://entity.example/.well-known/openid-federation`. If the Entity Identifier contains a trailing "/" character, it MUST be removed before concatenating `/.well-known/openid-federation`.

Furthermore, any Entity Configuration including the Entity Type Identifier `federation_entity` MUST be published at its configuration endpoint.

While Leaf Federation Entities SHOULD make an Entity Configuration document available at their configuration endpoints, an exception to this requirement is that clients that use a client registration method that results in the server having the client's Entity Configuration MAY omit doing so. For instance, since an RP using Explicit Registration posts its Entity Configuration to the OP during client registration, the OP has everything it needs from the RP. Profiles of this specification MAY define other exceptions for Leaf Entities that do not use the the Entity Type Identifier `federation_entity` and processing rules that accompany them.



### Federation Entity Configuration Request

An Entity Configuration document MUST be queried using an HTTP GET request at the previously specified path.

In this example, the requesting party would make the following request to the Entity `https://openid.sunet.se` to obtain its Entity Configuration:

[]{#name-request-for-entity-configur}

<figure id="figure-46">
<div id="section-9.1-3.1" class="alignLeft art-text artwork">
<pre><code>  GET /.well-known/openid-federation HTTP/1.1
  Host: openid.sunet.se</code></pre>
</div>
<figcaption><a href="#figure-46" class="selfRef">Figure 46</a>: <a href="#name-request-for-entity-configur" class="selfRef">Request for Entity Configuration</a></figcaption>
</figure>





### Federation Entity Configuration Response

The response is an Entity Configuration, as described in [Section 3](#entity-statement){.auto .internal .xref}. If the Entity is an Intermediate Entity or a Trust Anchor, the response MUST contain metadata for a federation Entity (`federation_entity`).

A successful response MUST use the HTTP status code 200 with the content type `application/entity-statement+jwt`, to make it clear that the response contains an Entity Statement. In case of an error, the response is as defined in [Section 8.9](#error_response){.auto .internal .xref}.

The following is a non-normative example JWT Claims Set for a response from an Intermediate Entity:

[]{#name-entity-configuration-respon}

<figure id="figure-47">
<div id="section-9.2-4.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;iss&quot;: &quot;https://openid.sunet.se&quot;,
  &quot;sub&quot;: &quot;https://openid.sunet.se&quot;,
  &quot;iat&quot;: 1516239022,
  &quot;exp&quot;: 1516298022,
  &quot;metadata&quot;: {
    &quot;federation_entity&quot;: {
      &quot;contacts&quot;: [&quot;ops@sunet.se&quot;],
      &quot;federation_fetch_endpoint&quot;: &quot;https://sunet.se/openid/fedapi&quot;,
      &quot;organization_uri&quot;: &quot;https://www.sunet.se&quot;,
      &quot;organization_name&quot;: &quot;SUNET&quot;
    },
    &quot;openid_provider&quot;: {
      &quot;issuer&quot;: &quot;https://openid.sunet.se&quot;,
      &quot;signed_jwks_uri&quot;: &quot;https://openid.sunet.se/jwks.jose&quot;,
      &quot;authorization_endpoint&quot;:
        &quot;https://openid.sunet.se/authorization&quot;,
      &quot;client_registration_types_supported&quot;: [
        &quot;automatic&quot;,
        &quot;explicit&quot;
      ],
      &quot;grant_types_supported&quot;: [
        &quot;authorization_code&quot;
      ],
      &quot;id_token_signing_alg_values_supported&quot;: [
        &quot;ES256&quot;, &quot;RS256&quot;
      ],
      &quot;logo_uri&quot;:
        &quot;https://www.umu.se/img/umu-logo-left-neg-SE.svg&quot;,
      &quot;op_policy_uri&quot;:
        &quot;https://www.umu.se/en/website/legal-information/&quot;,
      &quot;response_types_supported&quot;: [
        &quot;code&quot;
      ],
      &quot;subject_types_supported&quot;: [
        &quot;pairwise&quot;,
        &quot;public&quot;
      ],
      &quot;token_endpoint&quot;: &quot;https://openid.sunet.se/token&quot;,
      &quot;federation_registration_endpoint&quot;:
        &quot;https://op.umu.se/openid/fedreg&quot;,
      &quot;token_endpoint_auth_methods_supported&quot;: [
        &quot;private_key_jwt&quot;
      ]

    }
  },
  &quot;jwks&quot;: {
    &quot;keys&quot;: [
      {
        &quot;alg&quot;: &quot;RS256&quot;,
        &quot;e&quot;: &quot;AQAB&quot;,
        &quot;kid&quot;: &quot;key1&quot;,
        &quot;kty&quot;: &quot;RSA&quot;,
        &quot;n&quot;: &quot;pnXBOusEANuug6ewezb9J_...&quot;,
        &quot;use&quot;: &quot;sig&quot;
      }
    ]
  },
  &quot;authority_hints&quot;: [
    &quot;https://edugain.org/federation&quot;
  ]
}</code></pre>
</div>
<figcaption><a href="#figure-47" class="selfRef">Figure 47</a>: <a href="#name-entity-configuration-respon" class="selfRef">Entity Configuration Response JWT Claims Set</a></figcaption>
</figure>







## Resolving the Trust Chain and Metadata

An Entity (Party A) that wants to establish trust with another Entity (Party B) MUST have Party B's Entity Identifier and a list of Entity Identifiers of Trust Anchors and their public signing keys. Party A will first have to fetch sufficient Entity Statements to establish at least one chain of trust from Party B to one or more of the Trust Anchors. After that, Party A MUST validate the Trust Chains independently, and if there are multiple valid Trust Chains and if the application demands it, choose one to use.

To delegate the Trust Chain evaluation to a trusted third party, the Entity (Party A) that wants to establish trust with another Entity (Party B) MAY use a resolve endpoint, as defined in [Section 8.3](#resolve){.auto .internal .xref}.



### Fetching Entity Statements to Establish a Trust Chain

Depending on the circumstances, Party A MAY be handed Party B's Entity Configuration, or it may have to fetch it by itself. If it needs to fetch it, it will use the process described in [Section 9](#federation_configuration){.auto .internal .xref} based on the Entity Identifier of Party B.

The next step is to iterate through the list of Intermediates listed in `authority_hints`, ignoring the authority hints that end in an unknown Trust Anchor, requesting an Entity Configuration from each of the Intermediates. If the received Entity Configuration contains an authority hint, this process is repeated.

With the list of all Intermediates and the Trust Anchor, the respective fetch endpoints, as defined in [Section 8.1](#fetch_endpoint){.auto .internal .xref}, are used to fetch Entity Statements about the Intermediates and Party B.

Federation participants MUST NOT attempt to fetch Entity Statements they already have obtained during this process to prevent loops. If a loop is detected, the authority hint that led to it MUST NOT be used.

A successful operation will return one or more lists of Entity Statements. Each of the lists terminating in a self-signed Entity Statement is issued by a Trust Anchor.

If there is no path from Party B to at least one of the trusted Trust Anchors, then the list will be empty and there is no way of establishing trust in Party B's information. How Party A deals with this is out of scope for this specification.

The following sequence diagram represents the interactions between the RP, the OP, and the Trust Anchor during a trust evaluation made by the OP about the RP. Relating this to the preceding description, in this diagram, Party A is the OP and Party B is the RP.

[]{#name-resolving-trust-chain-and-m}

<figure id="figure-48">
<div id="section-10.1-8.1" class="alignLeft art-text artwork">
<pre><code>+-----+                         +-----+                             +--------------+
| RP  |                         | OP  |                             | Trust Anchor |
+-----+                         +-----+                             +--------------+
   |                               |                                        |
   | Entity Configuration Request  |                                        |
   |&lt;------------------------------|                                        |
   |                               |                                        |
   | Entity Configuration Response |                                        |
   |------------------------------&gt;|                                        |
   |                               |                                        |
   |                               | Evaluates authority_hints              |
   |                               |--------------------------              |
   |                               |                         |              |
   |                               |&lt;-------------------------              |
   |                               |                                        |
   |                               | Entity Configuration Request           |
   |                               |---------------------------------------&gt;|
   |                               |                                        |
   |                               |        Entity Configuration Response   |
   |                               |&lt;---------------------------------------|
   |                               |                                        |
   |                               | Obtains Fetch Endpoint                 |
   |                               |-----------------------                 |
   |                               |                      |                 |
   |                               |&lt;----------------------                 |
   |                               |                                        |
   |                               | Request Subordinate Statement about RP |
   |                               |---------------------------------------&gt;|
   |                               |                                        |
   |                               |        Subordinate Statement about RP  |
   |                               |&lt;---------------------------------------|
   |                               |                                        |
   |                               | Evaluates the Trust Chain              |
   |                               |--------------------------              |
   |                               |                         |              |
   |                               |&lt;-------------------------              |
   |                               |                                        |
   |                               | Applies Metadata Policies              |
   |                               |--------------------------              |
   |                               |                         |              |
   |                               |&lt;-------------------------              |
   |                               |                                        |
   |                               | Applies Constraints                    |
   |                               |--------------------                    |
   |                               |                   |                    |
   |                               |&lt;-------------------                    |
   |                               |                                        |
   |                               | Derives the RP&#39;s Resolved Metadata     |
   |                               |-----------------------------------     |
   |                               |                                  |     |
   |                               |&lt;----------------------------------     |</code></pre>
</div>
<figcaption><a href="#figure-48" class="selfRef">Figure 48</a>: <a href="#name-resolving-trust-chain-and-m" class="selfRef">Resolving Trust Chain and Metadata from the Perspective of an OP</a></figcaption>
</figure>





### Validating a Trust Chain

As described in [Section 4](#trust_chain){.auto .internal .xref}, a Trust Chain consists of an ordered list of Entity Statements. So however Party A has acquired the set of Entity Statements, it MUST now verify that it is a proper Trust Chain using the rules laid out in that section.

Let us refer to the Entity Statements in the Trust Chain as ES[j], where j = 0,...,i, with 0 being the index of the first Entity Statement and i being the zero-based index of the last. To validate the Trust Chain, the following MUST be done:

- 
  For each Entity Statement ES[j], where j = 0,..,i:

  - 
    Verify that the statement contains all the required claims.
    

  - 
    Verify that `iat` has a value in the past.
    

  - 
    Verify that `exp` has a value that is in the future.
    
  

- 
  For ES[0] (the Entity Configuration of the Trust Chain subject), verify that `iss` == `sub`.
  

- 
  For ES[0], verify that its signature validates with a public key in ES[0]["jwks"].
  

- 
  For each j = 0,...,i-1, verify that ES[j]["iss"] == ES[j+1]["sub"].
  

- 
  For each j = 0,...,i-1, verify that the signature of ES[j] validates with a public key in ES[j+1]["jwks"].
  

- 
  For ES[i] (the Trust Anchor's Entity Configuration), verify that the issuer matches the Entity Identifier of the Trust Anchor.
  

- 
  For ES[i], verify that its signature validates with a public key of the Trust Anchor.
  

Verifying the signature is a much more expensive operation than verifying the correctness of the statement and the timestamps. An implementer MAY therefore choose not to verify the signatures until all the other checks have been done.

Federation participants MAY cache Entity Statements and signature verification results until they expire, per [Section 10.4](#trust_lifetime){.auto .internal .xref}.

After the preceding validation, metadata MUST be resolved to the subject of the Trust Chain, as described in [Section 6.1.4](#metadata_policy_enforcement){.auto .internal .xref}. Furthermore, constraints MUST be enforced for each Subordinate Statement of the Trust Chain, as explained in [Section 6.2](#chain_constraints){.auto .internal .xref}.





### Choosing One of the Valid Trust Chains

If multiple valid Trust Chains are found, Party A will need to decide on which one to use. One simple rule would be to prefer a shorter chain over a longer one. Federation participants MAY follow other rules according to local policy.





### Calculating the Expiration Time of a Trust Chain

Each Entity Statement in a Trust Chain is signed and MUST have an expiration time (`exp`). The expiration time of the whole Trust Chain is the minimum (`exp`) value within the Trust Chain.





### Transient Trust Chain Validation Errors

If the federation topology is being updated, for example when a set of Leaf Entities is moved to a new Intermediate Entity, the Trust Chain validation may fail in a transient manner. Retrying after a period of time may resolve the situation.





### Resolving the Trust Chain and Metadata with a Resolver

Note that an alternative method for resolving a Trust Chain for an Entity (Party B) using the methods described above is to use a resolve endpoint, as described in [Section 8.3](#resolve){.auto .internal .xref}. This lets the resolver do the work that otherwise the Entity (Party A) wanting to establish trust would have to do for itself.







## Updating Metadata, Key Rollover, and Revocation

This specification facilitates smoothly updating metadata and public keys.

As described in [Section 10.4](#trust_lifetime){.auto .internal .xref}, each Trust Chain has an expiration time. Federation participants MUST support refreshing a Trust Chain when it expires. How often a participant reevaluates the Trust Chain depends on how quickly it wants to find out that something has changed.



### Protocol Key Rollover

If a Leaf Entity publishes its public keys in its metadata using `jwks`, the expiration time of its Entity Configuration can be used to control how often the receiving Entity needs to fetch an updated set of public keys.





### Key Rollover for a Trust Anchor

A Trust Anchor MUST publish an Entity Configuration about itself. The expiration time (exp) set on this Entity Configuration should be chosen such that it ensures that federation participants re-fetch it at reasonable intervals. When a Trust Anchor rolls over its signing keys, it needs to:

1.  
    Add the new keys to the `jwks` representing the Trust Anchor's signing keys in its Entity Configuration.
    

2.  
    Keep signing the Entity Configuration and the Entity Statements using the old keys for a long enough time period to allow all Subordinates to have obtained the new keys.
    

3.  
    Switch to signing with the new keys.
    

4.  
    After a reasonable time period, remove the old keys. What is regarded as a reasonable time is dependent on the security profile and risk assessment of the Trust Anchor.
    





### Redundant Retrieval of Trust Anchor Keys

It is RECOMMENDED that Federation Operators provide a means of retrieving the public keys for the Trust Anchors it administers that is independent of those Trust Anchors' Entity Configurations. This is intended to provide redundancy in the eventuality of the compromise of the Web PKI infrastructure underlying retrieval of public keys from Entity Configurations.

The keys retrieved via the independent mechanism specified by the Federation Operator SHOULD be compared to those retrieved via the Trust Anchor's Entity Configuration. If they do not match, both SHOULD be retrieved again. If they still do not match, it is indicative of a security or configuration problem. The appropriate remediation steps in that eventuality SHOULD be specified by the Federation Operator.





### Revocation

Since the participants in federations are expected to check the Trust Chain on a regular frequent basis, this specification does not define a revocation process. Specific federations MAY make a different choice and will then have to define their own revocation process.







## OpenID Connect Client Registration

This section describes how the mechanisms defined in this specification can be used to establish trust between an RP and an OP that have no prior explicit configuration or registration between them. It defines two client registration methods, Automatic Registration and Explicit Registration, that use Trust Chains, per [Section 10](#resolving_trust){.auto .internal .xref}. Federations can use other appropriate methods for client registration.

Federations with OpenID Connect Entities SHOULD agree on the supported client registration methods.

Note that both Automatic Registration and Explicit Registration can also be used for OAuth 2.0 profiles other than OpenID Connect. To do so, rather than using the Entity Type Identifiers `openid_relying_party` and `openid_provider`, one would instead use the Entity Type Identifiers `oauth_client` and `oauth_authorization_server`, or possibly other Entity Type Identifiers defined for the specific OAuth 2.0 profile being used.



### Automatic Registration

Automatic Registration enables an RP to make Authentication Requests without a prior registration step with the OP. The OP resolves the RP's Entity Configuration from the Client ID in the Authentication Request, following the process defined in [Section 10](#resolving_trust){.auto .internal .xref}.

The RP MUST perform Trust Chain and metadata resolution for the OP, as specified in [Section 10](#resolving_trust){.auto .internal .xref} before it sends the Authentication Request. If the resolution is not successful, the RP MUST NOT attempt further interactions with the OP.

Automatic Registration has the following characteristics:

- 
  In all interactions with the OP, the RP employs its Entity Identifier as the Client ID. The OP retrieves the RP's Entity Configuration from the URL derived from the Entity Identifier, as described in [Section 9](#federation_configuration){.auto .internal .xref}.
  

- 
  Since there is no registration step prior to the Authentication Request, asymmetric cryptography MUST be used to authenticate requests when using Automatic Registration. Asymmetric cryptography is used to authenticate requests; therefore, the OP neither assigns a Client Secret to the RP nor returns it as a result of the registration process.
  

An OP that supports Automatic Registration MUST include the `automatic` keyword in its `client_registration_types_supported` metadata parameter.



#### Authentication Request

The Authentication Request is performed by passing a Request Object by value or by reference, as described in Section 6 of [OpenID Connect Core 1.0](#OpenID.Core){.internal .xref} [[OpenID.Core](#OpenID.Core){.cite .xref}] and [The OAuth 2.0 Authorization Framework: JWT-Secured Authorization Request (JAR)](#RFC9101){.internal .xref} [[RFC9101](#RFC9101){.cite .xref}], or using a pushed authorization request, as described in [Pushed Authorization Requests](#RFC9126){.internal .xref} [[RFC9126](#RFC9126){.cite .xref}].

Authentication requests MUST demonstrate that the requesting Entity controls the Entity's RP keys, using one of the methods described below. Attempted authentication requests that do not do so MUST be rejected.

Deployments MAY choose not to support passing the request object by reference (using the `request_uri` request parameter) because allowing this would make it easier for attackers to mount denial of service attacks against OAuth 2.0 Authorization Servers or OpenID Providers. They can do this by using the `request_uri_parameter_supported` OP metadata parameter with the value `false`. If the request parameters are too large to practically be passed by value as query parameters, the request parameters can instead be sent via HTTP POST or a [Pushed Authorization Request](#RFC9126){.internal .xref} [[RFC9126](#RFC9126){.cite .xref}], as described in [Section 12.1.1.2](#using-par){.auto .internal .xref}.



##### Using a Request Object

When a Request Object is used at the Authorization Endpoint or the Pushed Authorization Request Endpoint, the value of the `request` parameter is a JWT whose Claims are the request parameters specified in Section 3.1.2 in [OpenID Connect Core 1.0](#OpenID.Core){.internal .xref} [[OpenID.Core](#OpenID.Core){.cite .xref}]. The JWT MUST be signed and MAY be encrypted. The following parameters are used in the Request Object:

[]{.break}

aud
:   REQUIRED. The Audience (aud) value MUST be the OP's Entity Identifier and MUST NOT include any other values.
:   

client_id
:   REQUIRED. The `client_id` value MUST be the RP's Entity Identifier.
:   

iss
:   REQUIRED. The `iss` value MUST be the RP's Entity Identifier.
:   

sub
:   MUST NOT be present. This prevents reuse of the statement for `private_key_jwt` client authentication.
:   

jti
:   REQUIRED. JWT ID. A unique identifier for the JWT, which can be used to prevent reuse of the Request Object. A Request Object MUST only be used once, unless conditions for reuse were negotiated between the parties; any such negotiation is beyond the scope of this specification.
:   

exp
:   REQUIRED. Number. Expiration time after which the JWT MUST NOT be accepted for processing. This is expressed as Seconds Since the Epoch, per [[RFC7519](#RFC7519){.cite .xref}].
:   

iat
:   OPTIONAL. Number. Time when this Request Object was issued. This is expressed as Seconds Since the Epoch, per [[RFC7519](#RFC7519){.cite .xref}].
:   

trust_chain

:   
    OPTIONAL. Array containing the sequence of Entity Statements that comprise the Trust Chain between the RP making the request and the selected Trust Anchor, sorted as in [Section 4](#trust_chain){.auto .internal .xref}. When the RP and the OP are part of the same federation, the RP MUST select a Trust Anchor that it has in common with the OP; otherwise, the RP is free to select the Trust Anchor to use.
    

:   



###### Authorization Request with a Trust Chain

When the `trust_chain` request parameter is used in the authentication request, the Relying Party informs the OP of the sequence of Entity Statements that proves the trust relationship between it and the selected Trust Anchor.

Due to the large size of a Trust Chain, it may be necessary to use the HTTP POST method, a `request_uri`, or a [Pushed Authorization Request](#RFC9126){.internal .xref} [[RFC9126](#RFC9126){.cite .xref}] for the request.

The following is a non-normative example of the JWT Claims Set in a Request Object:

[]{#name-request-object-jwt-claims-s}

<figure id="figure-49">
<div id="section-12.1.1.1.1-4.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;typ&quot;: &quot;oauth-authz-req+jwt&quot;,
  &quot;alg&quot;: &quot;RS256&quot;,
  &quot;kid&quot;: &quot;that-kid-which-points-to-a-jwk-contained-in-the-trust-chain&quot;,
}
.
{
  &quot;aud&quot;: &quot;https://op.example.org&quot;,
  &quot;client_id&quot;: &quot;https://rp.example.com&quot;,
  &quot;exp&quot;: 1589699162,
  &quot;iat&quot;: 1589699102,
  &quot;iss&quot;: &quot;https://rp.example.com&quot;,
  &quot;jti&quot;: &quot;4d3ec0f81f134ee9a97e0449be6d32be&quot;,
  &quot;nonce&quot;: &quot;4LX0mFMxdBjkGmtx7a8WIOnB&quot;,
  &quot;redirect_uri&quot;: &quot;https://rp.example.com/authz_cb&quot;,
  &quot;response_type&quot;: &quot;code&quot;,
  &quot;scope&quot;: &quot;openid profile email address phone&quot;,
  &quot;state&quot;: &quot;YmX8PM9I7WbNoMnnieKKBiptVW0sP2OZ&quot;,
  &quot;trust_chain&quot; : [
    &quot;eyJhbGciOiJSUzI1NiIsImtpZCI6Ims1NEhRdERpYnlHY3M5WldWTWZ2aUhm ...&quot;,
    &quot;eyJhbGciOiJSUzI1NiIsImtpZCI6IkJYdmZybG5oQU11SFIwN2FqVW1BY0JS ...&quot;,
    &quot;eyJhbGciOiJSUzI1NiIsImtpZCI6IkJYdmZybG5oQU11SFIwN2FqVW1BY0JS ...&quot;
  ]
}</code></pre>
</div>
<figcaption><a href="#figure-49" class="selfRef">Figure 49</a>: <a href="#name-request-object-jwt-claims-s" class="selfRef">Request Object JWT Claims Set</a></figcaption>
</figure>

The following is a non-normative example of an Authentication Request using the `request` parameter (with line wraps within values for display purposes only):

[]{#name-authentication-request-usin}

<figure id="figure-50">
<div id="section-12.1.1.1.1-6.1" class="alignLeft art-text artwork">
<pre><code>Host: server.example.com
GET /authorize?
    redirect_uri=https%3A%2F%2Frp.example.com%2Fauthz_cb
    &amp;scope=openid+profile+email+address+phone
    &amp;response_type=code
    &amp;client_id=https%3A%2F%2Frp.example.com
    &amp;request=eyJ0eXAiOiJvYXV0aC1hdXRoei1yZXErand0IiwiYWxnIjoiUlMyNTYiLCJ
        raWQiOiJOX19EOThJdkI4TmFlLWt3QTZuck90LWlwVGhqSGtEeDM3bmljRE1IM04
        0In0.
        eyJhdWQiOiJodHRwczovL29wLmV4YW1wbGUub3JnIiwiY2xpZW50X2lkIjoiaHR0
        cHM6Ly9ycC5leGFtcGxlLmNvbSIsImV4cCI6MTU4OTY5OTE2MiwiaWF0IjoxNTg5
        Njk5MTAyLCJpc3MiOiJodHRwczovL3JwLmV4YW1wbGUuY29tIiwianRpIjoiNGQz
        ZWMwZjgxZjEzNGVlOWE5N2UwNDQ5YmU2ZDMyYmUiLCJub25jZSI6IjRMWDBtRk14
        ZEJqa0dtdHg3YThXSU9uQiIsInJlZGlyZWN0X3VyaSI6Imh0dHBzOi8vcnAuZXhh
        bXBsZS5jb20vYXV0aHpfY2IiLCJyZXNwb25zZV90eXBlIjoiY29kZSIsInNjb3Bl
        Ijoib3BlbmlkIHByb2ZpbGUgZW1haWwgYWRkcmVzcyBwaG9uZSIsInN0YXRlIjoi
        WW1YOFBNOUk3V2JOb01ubmllS0tCaXB0Vlcwc1AyT1oiLCJ0cnVzdF9jaGFpbiI6
        WyJleUpoYkdjaU9pSlNVekkxTmlJc0ltdHBaQ0k2SW1zMU5FaFJkRVJwWW5sSFkz
        TTVXbGRXVFdaMmFVaG0gLi4uIiwiZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJ
        NklrSllkbVp5Ykc1b1FVMTFTRkl3TjJGcVZXMUJZMEpTIC4uLiIsImV5SmhiR2Np
        T2lKU1V6STFOaUlzSW10cFpDSTZJa0pZZG1aeWJHNW9RVTExU0ZJd04yRnFWVzFC
        WTBKUyAuLi4iXX0.
        Rv0isfuku0FcRFintgxgKDk7EnhFkpQRg3Tm6N6fCHAHEKFxVVdjy4
        9JboJtxKcQVZKN9TKn3lEYM1wtF1e9PQrNt4HZ21ICfnzxXuNx1F5SY1GXCU2n2y
        FVKtz3N0YkAFbTStzy-sPRTXB0stLBJH74RoPiLs2c6dDvrwEv__GA7oGkg2gWt6
        VDvnfDpnvFi3ZEUR1J8MOeW_VFsayrT9sNjyjsz62Po4LzvQKQMKxq0dNwPNYuuS
        fUmb-YvmFguxDb3weYl8WS-
        48EIkP1h4b_KGU9x9n7a1fUOHrS02ATQZmaL8jUil7yLJqx5MiCsPr4pCAXV0doA
        4pwhs_FIw HTTP/1.1
</code></pre>
</div>
<figcaption><a href="#figure-50" class="selfRef">Figure 50</a>: <a href="#name-authentication-request-usin" class="selfRef">Authentication Request Using Request Object</a></figcaption>
</figure>





###### Processing the Authentication Request

When the OP receives an incoming Authentication Request, the OP supports OpenID Federation, the incoming Client ID is a valid URL, and the OP does not have the Client ID registered as a known client, then the OP SHOULD resolve the Trust Chains related to the requestor.

An RP MAY present a Trust Chain related to itself to the OP in the Request Object using the `trust_chain` request parameter defined in [Section 12.1.1.1](#trust_chain_param){.auto .internal .xref}. If the OP does not have a valid registration for the RP or its registration has expired, the OP MAY use the received Trust Chain as a hint for which path to take from the RP's Entity to the Trust Anchor. The OP MAY evaluate the statements in the provided Trust Chain to make its Federation Entity Discovery procedure more efficient, especially if the RP contains multiple authority hints in its Entity Configuration. If the OP already has a valid registration for the RP, it MAY use the received Trust Chain to update the RP's registration.

A Trust Chain may be relied upon by the OP because it has validated all of its statements. This is true whether these statements are retrieved from their URLs or whether they are provided via the `trust_chain` request parameter in the Request Object. In both cases, the OP MUST fully verify the Trust Chain including every Entity Statement contained in it.

If the RP does not include the `trust_chain` request parameter in the Request Object, the OP for some reason decides not to use the provided Trust Chain, or the OP does not support this feature, it then MUST validate the possible Trust Chains, starting with the RP's Entity Configuration, as described in [Section 10.1](#fetching-es){.auto .internal .xref}, and resolve the RP metadata with Entity Type `openid_relying_party`.

The OP SHOULD furthermore verify that the Resolved Metadata of the RP complies with the client metadata specification [OpenID Connect Dynamic Client Registration 1.0](#OpenID.Registration){.internal .xref} [[OpenID.Registration](#OpenID.Registration){.cite .xref}].

Once the OP has the RP's metadata, it MUST verify that the client was actually the one sending the Authentication Request by verifying the signature of the Request Object using the key material the client published in its metadata for the `openid_relying_party` Entity Type. If the signature does not verify, the OP MUST reject the request.







##### Using Pushed Authorization

[Pushed Authorization Requests](#RFC9126){.internal .xref} [[RFC9126](#RFC9126){.cite .xref}] provide an interoperable way to push authentication request parameters directly to the AS in exchange for a one-time-use `request_uri`. The standard PAR metadata parameters are used in the RP and OP metadata to indicate its use.

When using PAR with Automatic Registration, either a Request Object MUST be used as a PAR parameter, with the Request Object being as described in [Section 12.1.1.1](#UsingAuthzRequestObject){.auto .internal .xref}, or a client authentication method for the PAR endpoint MUST be used that proves possession of one of the RP's private keys. Furthermore, the corresponding public key MUST be in the Entity's RP JWK Set.

The two applicable PAR client authentication methods are:

- 
  JWT Client authentication, as described for `private_key_jwt` in Section 9 of [OpenID Connect Core 1.0](#OpenID.Core){.internal .xref} [[OpenID.Core](#OpenID.Core){.cite .xref}]. In this case, the audience of the client authentication JWT MUST be the OP's Entity Identifier and MUST NOT include any other values.
  

- 
  mTLS using self-signed certificates, as described in Section 2.2 of [[RFC8705](#RFC8705){.cite .xref}]. In this case, the self-signed certificate MUST be present as the value of an `x5c` claim for a key in the Entity's RP JWK Set. In this case, the server MUST omit certificate chain validation.
  

A Pushed Authorization Request to the OP could look like this:

[]{#name-pushed-authorization-reques}

<figure id="figure-51">
<div id="section-12.1.1.2-6.1" class="alignLeft art-text artwork">
<pre><code>POST /par HTTP/1.1
Host: op.example.org
Content-Type: application/x-www-form-urlencoded

redirect_uri=https%3A%2F%2Frp.example.com%2Fauthz_cb
&amp;scope=openid+profile+email+address+phone
&amp;response_type=code
&amp;nonce=4LX0mFMxdBjkGmtx7a8WIOnB
&amp;state=YmX8PM9I7WbNoMnnieKKBiptVW0sP2OZ
&amp;client_id=https%3A%2F%2Frp.example.com
&amp;client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3A
  client-assertion-type%3Ajwt-bearer
&amp;client_assertion=eyJhbGciOiJSUzI1NiIsImtpZCI6ImRVTjJ
  hMDF3Umtoa1NXcGxRVGh2Y1ZCSU5VSXdUVWRPVUZVMlRtVnJTbW
  hFUVhnelpYbHBUemRRTkEifQ.
  eyJzdWIiOiAiaHR0cHM6Ly9ycC5leGFtcGxlLmNvbSIsICJpc3M
  iOiAiaHR0cHM6Ly9ycC5leGFtcGxlLmNvbSIsICJpYXQiOiAxNT
  g5NzA0NzAxLCAiZXhwIjogMTU4OTcwNDc2MSwgImF1ZCI6ICJod
  HRwczovL29wLmV4YW1wbGUub3JnIiwgImp0aSI6ICIzOWQ1YWU1
  NTJkOWM0OGYwYjkxMmRjNTU2OGVkNTBkNiJ9.
  oUt9Knx_lxb4V2S0tyNFH
  CNZeP7sImBy5XDsFxv1cUpGkAojNXSy2dnU5HEzscMgNW4wguz6
  KDkC01aq5OfN04SuVItS66bsx0h4Gs7grKAp_51bClzreBVzU4g
  _-dFTgF15T9VLIgM_juFNPA_g4Lx7Eb5r37rWTUrzXdmfxeou0X
  FC2p9BIqItU3m9gmH0ojdBCUX5Up0iDsys6_npYomqitAcvaBRD
  PiuUBa5Iar9HVR-H7FMAr7aq7s-dH5gx2CHIfM3-qlc2-_Apsy0
  BrQl6VePR6j-3q6JCWvNw7l4_F2UpHeanHb31fLKQbK-1yoXDNz
  DwA7B0ZqmuSmMFQ</code></pre>
</div>
<figcaption><a href="#figure-51" class="selfRef">Figure 51</a>: <a href="#name-pushed-authorization-reques" class="selfRef">Pushed Authorization Request to the OP</a></figcaption>
</figure>



###### Processing the Pushed Authentication Request

The requirements specified in [Section 12.1.1.1.2](#AuthzRequestProcessing){.auto .internal .xref} also apply to [Pushed Authorization Requests](#RFC9126){.internal .xref} [[RFC9126](#RFC9126){.cite .xref}].

Once the OP has the RP's metadata, it MUST verify the client is using the keys published for the `openid_relying_party` Entity Type Identifier. If the RP does not verify, the OP MUST reject the request.

The means of verification depends on the client authentication method used:

[]{.break}

private_key_jwt
:   If this method is used, then the OP verifies the signature of the signed JWT using the key material published by the RP in its metadata. If the authentication is successful, then the registration is valid. The audience of the signed JWT MUST be the Authorization Server's Entity Identifier and MUST NOT include any other values.
:   

self_signed_tls_client_auth
:   If mTLS is used with a self-signed certificate, then the certificate MUST be present as the value of an `x5c` claim for a key in the JWK Set containing the RP's keys.
:   









#### Successful Authentication Response

The response to a successful authentication request when using Automatic Registration is the same as the successful authentication responses defined in [[OpenID.Core](#OpenID.Core){.cite .xref}]. It is a successful OAuth 2.0 authorization response sent to the Client's redirection URI.





#### Authentication Error Response

The error response to an unsuccessful authentication request when using Automatic Registration is the same as the error authentication responses defined in [[OpenID.Core](#OpenID.Core){.cite .xref}]. It is an OAuth 2.0 authorization error response sent to the Client's redirection URI, unless a [Pushed Authorization Request](#RFC9126){.internal .xref} [[RFC9126](#RFC9126){.cite .xref}] was used for the request.

However, as in both [[OpenID.Core](#OpenID.Core){.cite .xref}] and [[RFC6749](#RFC6749){.cite .xref}], if the redirection URI is invalid, redirection MUST NOT be performed, and instead the Authorization Server SHOULD inform the End-User of the error in the user interface. The Authorization Server MAY also choose to do this if it has reason to believe that the redirection URI might be being used as a component of an open redirector.

If the OP fails to establish trust with the RP or finds the RP metadata to be invalid or in conflict with metadata policy, it MUST treat the redirection URI as invalid and not perform redirection. This means that the error codes `invalid_trust_anchor`, `invalid_trust_chain`, and `invalid_metadata`, which are about reasons trust failed to be established, SHOULD only be returned in [Pushed Authorization Request](#RFC9126){.internal .xref} [[RFC9126](#RFC9126){.cite .xref}] error responses, and not to the Client's redirection URI.

In addition to the error codes contained in the IANA "OAuth Extensions Error Registry" registry [[IANA.OAuth.Parameters](#IANA.OAuth.Parameters){.cite .xref}], this specification also defines the error codes in [Section 8.9](#error_response){.auto .internal .xref}, which MAY also be used.

The following is a non-normative example authentication error response:

[]{#name-authentication-error-respons}

<figure id="figure-52">
<div id="section-12.1.3-6.1" class="alignLeft art-text artwork">
<pre><code>HTTP/1.1 302 Found
  Location: https://client.example.org/cb?
    error=consent_required
    &amp;error_description=
      Consent%20by%20the%20End-User%20required
    &amp;state=af0ifjsldkj</code></pre>
</div>
<figcaption><a href="#figure-52" class="selfRef">Figure 52</a>: <a href="#name-authentication-error-respons" class="selfRef">Authentication Error Response</a></figcaption>
</figure>





#### Automatic Registration and Client Authentication

Note that when using Automatic Registration, the client authentication methods that the client can use are declared to the OP using RP Metadata parameters: either the `token_endpoint_auth_methods_supported` parameter defined in [[OpenID.RP.Choices](#OpenID.RP.Choices){.cite .xref}] or the `token_endpoint_auth_method` parameter. Those that the OP can use are likewise declared to the RP using OP Metadata parameters. However, if there are multiple methods supported by both the RP and the OP, the OP does not know which one the RP will pick in advance of it being used, since this isn't declared at the time the Automatic Registration occurs.

OPs SHOULD accept any client authentication method that is mutually supported and RPs MUST only use mutually supported methods. Because some OPs may be coded in such a way that they expect the RP to always use the same client authentication method for subsequent interactions, note that interoperability may be improved by the RP doing so.





#### Possible Other Uses of Automatic Registration

Automatic Registration is designed to be able to be employed for OAuth 2.0 use cases beyond OpenID Connect, as noted in [Section 12](#client_registration){.auto .internal .xref}. For instance, ecosystems using bare [OAuth 2.0](#RFC6749){.internal .xref} [[RFC6749](#RFC6749){.cite .xref}] or [FAPI](#FAPI){.internal .xref} [[FAPI](#FAPI){.cite .xref}] can utilize Automatic Registration.

Also note that Client ID values that are Entity Identifiers could be used to identify clients using Automatic Registration at endpoints other than the Authorization Endpoint and Token Endpoint in OAuth 2.0 deployments, such as the Pushed Authorization Request (PAR) Endpoint and Introspection Endpoint. Describing particular such scenarios is beyond the scope of this specification.







### Explicit Registration

Using this method, the RP establishes its client registration with the OP by means of a dedicated registration request, similar to [[OpenID.Registration](#OpenID.Registration){.cite .xref}], but instead of its metadata, the RP submits its Entity Configuration or an entire Trust Chain. When the Explicit Registration is completed, the RP can proceed to make regular OpenID authentication requests to the OP.

An OP that supports Explicit Registration MUST include the `explicit` keyword in its `client_registration_types_supported` metadata parameter and set the `federation_registration_endpoint` metadata parameter to the URL at which it receives Explicit Registration requests.

Explicit Registration is suitable for implementation on top of the [OpenID Connect Dynamic Client Registration 1.0](#OpenID.Registration){.internal .xref} [[OpenID.Registration](#OpenID.Registration){.cite .xref}] endpoint of an OP deployment. In contrast to Automatic Registration, it enables an OP to provision a Client ID, potentially a Client Secret, and other metadata parameters.

An example of an Explicit Registration is provided in [Appendix A.3.2](#ExplicitRegExample){.auto .internal .xref}.



#### Explicit Client Registration Request

The RP performs Explicit Client Registration as follows:

1.  
    Once the RP has determined a set of Trust Anchors it has in common with the OP, it chooses the subset it wants to proceed with. This may be a single Trust Anchor, or it can also be more than one. The RP MUST perform Trust Chain and metadata resolution for the OP, as specified in [Section 10](#resolving_trust){.auto .internal .xref}. If the resolution is not successful, the RP MUST abort the request.
    

2.  
    Using this subset of Trust Anchors, the RP chooses a set of `authority_hints` from the hints that are available to it. Each hint MUST, when used as a starting point for Trust Chain collection, lead to at least one of the Trust Anchors in the subset. If the RP has more than one Trust Anchor in common with the OP it MUST select a subset of Trust Anchors to proceed with. The subset may be as small as a single Trust Anchor, or include multiple ones.
    

3.  
    The RP will then construct its Entity Configuration, where the metadata statement chosen is influenced by the OP's metadata and where the `authority_hints` included are picked by the process described above. From its Immediate Superiors, the RP MUST select one or more `authority_hints` so that every hint, when used as the starting point for a Trust Chain collection, leads to at least one of the Trust Anchors in the subset selected above.
    

4.  
    The RP MAY include its Entity Configuration in a Trust Chain regarding itself. In that case, the Registration Request will contain an array with the sequence of Entity Statements that compose the Trust Chain between the RP that is making the request and the selected Trust Anchor. The RP MUST include its metadata in its Entity Configuration and use the `authority_hints` selected above.

    The RP SHOULD select its metadata parameters to comply with the resolved OP metadata and thus ensure a successful registration with the OP. Note that if the submitted RP metadata is not compliant with the metadata of the OP, the OP may choose to modify it to make it compliant rather than reject the request with an error response.
    

5.  
    The Entity Configuration, or the entire Trust Chain, is sent, using HTTP POST, to the `federation_registration_endpoint`. The Entity Configuration or Trust Chain is the entire POST body. The RP MUST sign its Entity Configuration with a current Federation Entity Key in its possession.
    

6.  
    The content type of the Registration Request MUST be `application/entity-statement+jwt` when it contains only the Entity Configuration of the requestor. Otherwise, when it contains the Trust Chain, the content type of the Registration Request MUST be `application/trust-chain+json`. The RP MAY include its Entity Configuration in a Trust Chain that leads to the RP. In this case, the registration request will contain an array consisting of the sequence of statements that make up the Trust Chain between the RP and the Trust Anchor the RP selected.
    

The following Entity Configuration claims are specified for use in Explicit Registration requests. Their full descriptions are in [Section 3](#entity-statement){.auto .internal .xref}.

[]{.break}

iss
:   REQUIRED. Its value MUST be the Entity Identifier of the RP.
:   

sub
:   REQUIRED. Its value MUST be the Entity Identifier of the RP.
:   

iat
:   REQUIRED.
:   

exp
:   REQUIRED.
:   

jwks
:   REQUIRED.
:   

aud
:   REQUIRED. Its value MUST be the Entity Identifier of the OP. This claim is only used in Explicit Registration requests, since it is not a general Entity Statement claim.
:   

authority_hints
:   REQUIRED.
:   

metadata
:   REQUIRED. It MUST contain the RP metadata under the `openid_relying_party` Entity Type Identifier.
:   

crit
:   OPTIONAL.
:   

trust_marks
:   OPTIONAL.
:   

The request MUST be an HTTP request to the `federation_registration_endpoint` of the OP, using the POST method.

When the RP submits an Entity Configuration the content type of the request MUST be `application/entity-statement+jwt`. When the RP submits a Trust Chain, the content type MUST be `application/trust-chain+json`.





#### Processing Explicit Client Registration Request by OP

The OP processes the request as follows:

1.  
    Upon receiving a registration request, the OP MUST inspect the content type to determine whether it contains an Entity Configuration or an entire Trust Chain.
    

2.  
    The OP MUST validate the RP's explicit registration request JWT. All the normal Entity Statement validation rules apply. In addition, if the `aud` (audience) claim value is not the Entity Identifier of the OP, then the request MUST be rejected.
    

3.  
    If the request contains an Entity Configuration the OP MUST use it to complete the Federation Entity Discovery by collecting and evaluating the Trust Chains that start with the `authority_hints` in the Entity Configuration of the RP. After validating at least one Trust Chain, the OP MUST verify the signature of the received Entity Configuration. If the OP finds more than one acceptable Trust Chain, it MUST choose one Trust Chain from among them as the one to proceed with.
    

4.  
    If the request contains a Trust Chain, the OP MAY evaluate the statements in the Trust Chain to save the HTTP calls that are necessary to perform the Federation Entity Discovery, especially if the RP included more than one authority hint in its Entity Configuration. Otherwise, the OP MUST extract the RP Entity Configuration from the Trust Chain and proceed according to Step 2, as if only an Entity Configuration was received.
    

5.  
    At this point, if the OP finds that it already has an existing client registration for the requesting RP, then that registration MUST be invalidated. The precise time of the invalidation is at the OP's discretion, as the OP may want to ensure the completion of concurrent OpenID authentication requests initiated by the RP while the registration request is being processed.

    The OP MAY retain client credentials and key material from the invalidated registration in order to verify past RP signatures and perform other cryptographic operations on past RP data.
    

6.  
    The OP uses the Resolved Metadata for the RP to create a client registration compliant with its own OP metadata and other applicable policies.

    The OP MAY provision the RP with a `client_id` other than the RP's Entity Identifier. This enables Explicit Registration to be implemented on top of the [OpenID Connect Dynamic Client Registration 1.0](#OpenID.Registration){.internal .xref} [[OpenID.Registration](#OpenID.Registration){.cite .xref}] endpoint of an OP.

    If the RP is provisioned with a `client_secret` it MUST NOT expire before the expiration of the registration Entity Statement that will be returned to the RP.

    The OP SHOULD NOT provision the RP with a `registration_access_token` and a `registration_client_uri` because the expected way for the RP to update its registration is to make a new Explicit Registration request. If the RP is provisioned with a `registration_access_token` for some purpose, for example to let it independently check its registered metadata, the token MUST NOT allow modification of the registration.

    The OP MAY modify the received RP metadata, for example by substituting an invalid or unsupported parameter, to make it compliant with its own OP metadata and other policies. If the OP does not accept the RP metadata or is unwilling to modify it to make it compliant, it MUST return a client registration error response, with an appropriate error, such as `invalid_client_metadata` or `invalid_redirect_uri`, as specified in Section 3.3 of [[OpenID.Registration](#OpenID.Registration){.cite .xref}].
    

7.  
    The OP MUST assign an expiration time to the created registration. This time MUST NOT exceed the expiration time of the Trust Chain that the OP selected to process the request.
    





#### Successful Explicit Client Registration Response

If the OP created a client registration for the RP, it MUST then construct a success response in the form of an Entity Statement.

The OP MUST set the `trust_anchor` claim of the Entity Statement to the Trust Anchor it selected to process the request. The `authority_hints` claim MUST be set to the RP's Immediate Superior in the selected Trust Chain.

The OP MUST set the `exp` claim to the expiration time of the created registration. The OP MAY choose to invalidate the registration before that, as explained in [Section 12.2.6](#AfterExplicitReg){.auto .internal .xref}.

The OP MUST express the client registration it created for the RP by means of the `metadata` claim, by placing the metadata parameters under the `openid_relying_party` Entity Type Identifier. The parameters MUST include the `client_id` that was provisioned for the RP. If the RP was provisioned with credentials, for example a `client_secret`, these MUST be included as well.

The OP SHOULD include metadata parameters that have a default value, for example `token_endpoint_auth_method` which has a default value of `client_secret_basic`, to simplify the processing of the response by the RP.

The OP MUST sign the registration Entity Statement with a current Federation Entity Key in its possession.

The following Entity Statement claims are used in Explicit Registration responses, as specified in [Section 3](#entity-statement){.auto .internal .xref}:

[]{.break}

iss
:   REQUIRED. Its value MUST be the Entity Identifier of the OP.
:   

sub
:   REQUIRED. Its value MUST be the Entity Identifier of the RP.
:   

iat
:   REQUIRED. Time when this statement was issued.
:   

exp
:   REQUIRED. Expiration time after which the statement MUST NOT be accepted for processing.
:   

jwks
:   OPTIONAL. If present, it MUST be a verbatim copy of the `jwks` Entity Statement claim from the received Entity Configuration of the RP. Note that this is distinct from the identically named RP metadata parameter.
:   

aud
:   REQUIRED. Its value MUST be the RP's Entity Identifier and MUST NOT include any other values. This claim is used in Explicit Registration responses but is not a general Entity Statement claim.
:   

trust_anchor
:   REQUIRED. Its value MUST be the Entity Identifier of the Trust Anchor that the OP selected to process the Explicit Registration request. This claim is specific to Explicit Registration responses and is not a general Entity Statement claim.
:   

authority_hints
:   REQUIRED. It MUST be a single-element array, whose value references the Immediate Superior of the RP in the Trust Chain that the OP selected to process the request.
:   

metadata
:   REQUIRED. It MUST contain the registered RP metadata under the `openid_relying_party` Entity Type Identifier.
:   

crit
:   OPTIONAL. Set of claims that MUST be understood and processed, as specified in [Section 3](#entity-statement){.auto .internal .xref}.
:   

A successful response MUST have an HTTP status code 200 and the content type `application/explicit-registration-response+jwt`. Furthermore, the `typ` header parameter value in the response MUST be `explicit-registration-response+jwt` (and not `entity-statement+jwt`) to prevent confusion between the Explicit Registration response and normal Entity Statements.





#### Explicit Client Registration Error Response

For a client registration error, the response is as defined in [Section 8.9](#error_response){.auto .internal .xref} and MAY use errors defined there and in Section 3.3 of [[OpenID.Registration](#OpenID.Registration){.cite .xref}] and Section 3.2.2 of [[RFC7591](#RFC7591){.cite .xref}].





#### Processing Explicit Client Registration Response by RP

1.  
    If the response indicates success, the RP MUST verify that its content is a valid Entity Statement and issued by the OP.

    The RP MUST ensure the signing Federation Entity Key used by the OP is present in the `jwks` claim of the Subordinate Statement issued by the OP's Immediate Superior in a Trust Chain that the RP successfully resolved for the OP when it prepared the Explicit Registration request.
    

2.  
    The RP MUST verify that the `aud` (audience) claim value is its Entity Identifier.
    

3.  
    The RP MUST verify that the `trust_anchor` represents one of its own Trust Anchors.
    

4.  
    The RP MUST verify that at least one of the `authority_hints` it specified in the Explicit Registration request leads to the Trust Anchor that the OP set in the `trust_anchor` claim.
    

5.  
    The RP MUST first ensure that the information it was registered with at the OP contains the same set of entity_types as the request does. After having collected a Trust Chain using the response claim `trust_anchor` as the Entity Identifier for the Trust Anchor and `authority_hints` as starting points for the Trust Chain collection, the RP SHOULD verify that the response metadata for each entity type is valid by applying the resolved policies to the received metadata, as specified in [Section 6.1.4.1](#metadata_policy_resolution){.auto .internal .xref}.
    

6.  
    If the received registration Entity Statement does not pass the above checks, the RP MUST reject it. The RP MAY choose to retry the Explicit Registration request to work around a transient exception, for example due to a recent change of Entity metadata or metadata policy causing temporary misalignment of metadata.
    





#### After an Explicit Client Registration

An RP can utilize the `exp` claim of the registration Entity Statement to devise a suitable strategy for renewing its client registration. RP implementers should note that if the OP expiration of the `client_id` coincides with an OAuth 2.0 flow that was just initiated by the RP, this may cause OpenID Connect authentication requests, token requests, or UserInfo requests to suddenly fail. Renewing the RP registration prior to its expiration can prevent such errors from occurring and ensure the end-user experience is not disrupted.

An OP MAY invalidate a client registration before the expiration that is indicated in the registration Entity Statement for the RP. An example reason could be the OP leaving the federation that was used to register the RP.







### Registration Validity and Trust Reevaluation

The validity of an Automatic or Explicit Registration at an OP MUST NOT exceed the lifetime of the Trust Chain the OP used to create the registration. An OP MAY choose to expire the registration at some earlier time, or choose to perform additional periodic reevaluations of the Trust Chain for the registered RP before the Trust Chain reaches its expiration time.

Similarly, an RP that obtained an Automatic or Explicit Registration MUST NOT use it past the expiration of the Trust Chain the RP used to establish trust in the OP. For an RP using Automatic Registration, the trust in the OP MUST be successfully reevaluated before continuing to make requests to the OP. For an RP using Explicit Registration, the RP MUST successfully renew its registration. An RP MAY choose to perform additional periodic reevaluations of the Trust Chain for the OP before the Trust Chain reaches its expiration time.





### Differences between Automatic Registration and Explicit Registration

The primary differences between Automatic Registration and Explicit Registration are:

- 
  With Automatic Registration, there is no registration step prior to the Authentication Request, whereas with Explicit Registration, there is. ([OpenID Connect Dynamic Client Registration 1.0](#OpenID.Registration){.internal .xref} [[OpenID.Registration](#OpenID.Registration){.cite .xref}] and [OAuth 2.0 Dynamic Client Registration](#RFC7591){.internal .xref} [[RFC7591](#RFC7591){.cite .xref}] also employ a prior registration step.)
  

- 
  With Automatic Registration, the Client ID value is the RP's Entity Identifier and is supplied to the OP by the RP, whereas with Explicit Registration, a Client ID is assigned by the OP and supplied to the RP.
  

- 
  With Automatic Registration, the Client is authenticated by means of the RP proving that it controls a private key corresponding to one of its Entity Configuration's public keys. Whereas with Explicit Registration, a broader set of options is available for authenticating the Client, including the use of a Client Secret.
  





### Rationale for the Trust Chain in the Request

Both Automatic and Explicit Client Registration support the submission of the Trust Chain embedded in the Request, calculated by the requestor, and related to itself. This provides the following benefits:

- 
  It solves the problem of OPs using RP metadata that has become stale. This stale data may occur when the OP uses cached RP metadata from a Trust Chain that has not reached its expiration time yet. The RP MAY notify the OP that a change has taken place by including the `trust_chain` header parameter or the `trust_chain` request parameter in the request, thus letting the OP update its Client Registration and preventing potential temporary faults due to stale metadata.
  

- 
  It enables the RP to pass a verifiable hint for which trust path to take to build the Trust Chain. This can reduce the costs of RP Federation Entity Discovery for OPs in complex federations where the RP has multiple Trust Anchors or the Trust Chain resolution may result in dead-ends.
  

- 
  It enables direct passing of the Entity Configuration, including any present Trust Marks, thus saving the OP from having to make an HTTP request to the RP `/.well-known/openid-federation` endpoint.
  







## General-Purpose JWT Claims

This section defines general-purpose JWT claims designed to be used by many different JWT profiles. They are also used in specific kinds of JWTs defined by this specification.



### "jwks" (JSON Web Key Set) Claim

The `jwks` (JSON Web Key Set) claim value is a JWK Set, as defined in [[RFC7517](#RFC7517){.cite .xref}]. It is used to convey a set of cryptographic keys. Use of this claim is OPTIONAL.

For instance, the `jwks` (JSON Web Key Set) claim might be used to represent a set of signing keys for an application. This claim is used in this specification in [Section 3](#entity-statement){.auto .internal .xref} to represent the public keys used to sign the Entity Statement.





### "metadata" Claim

The `metadata` claim is used for conveying metadata pertaining to the JWT. Its value is a JSON object. The details of the metadata contained are application-specific. Use of this claim is OPTIONAL.

For instance, the `metadata` claim might be used to represent a set of endpoint URLs and algorithm identifiers in an API description. This claim is used in this specification in [Section 3](#entity-statement){.auto .internal .xref} to represent metadata about the Entity.





### "constraints" Claim

The `constraints` claim is used for conveying constraints pertaining to the JWT. Its value is a JSON object. The details of the constraints contained are application-specific. Use of this claim is OPTIONAL.

For instance, the `constraints` claim might be used to impose material thickness limits for a physical object. This claim is used in this specification in [Section 3](#entity-statement){.auto .internal .xref} to represent constraints on the Trust Chain for the Entity.





### "crit" (Critical) Claim

The `crit` (critical) claim indicates that extensions to the set of claims specified for use in this type of JWT are being used that MUST be understood and processed. It is used in the same way that the `crit` header parameter is used for extension JOSE header parameters that MUST be understood and processed. Its value is an array listing the claim names present in the JWT that use those extensions. If any of the listed claims are not understood and supported by the recipient, then the JWT is invalid. Producers MUST NOT include claim names already specified for use in this type of JWT, duplicate names, or names that do not occur as claim names in the JWT in the `crit` list. Producers MUST NOT use the empty array `[]` as the `crit` value. Use of this claim is OPTIONAL.

This claim is used in this specification in [Section 3](#entity-statement){.auto .internal .xref} to identify claims not defined by this specification that MUST be understood and processed when used in Entity Statements.





### "ref" (Reference) Claim

The `ref` (reference) claim is used for conveying the URI for a resource pertaining to the JWT. It plays a similar role in a JWT as the `href` property does in HTML. The nature of the content at the referenced resource is generally application specific. The `ref` value is a case-sensitive string containing a URI value. Use of this claim is OPTIONAL.

For instance, a JWT referring to a contract between two parties might use the `ref` (reference) claim to refer to a resource at which the contract terms can be read. This claim is used in this specification in [Section 7.1](#trust_mark_claims){.auto .internal .xref} to provide a URL referring to human-readable information about the issuance of the Trust Mark.





### "delegation" Claim

The `delegation` claim expresses that authority is being delegated to the party referenced in the claim value. The `delegation` value is a case-sensitive string containing a StringOrURI value. Use of this claim is OPTIONAL.

For instance, the `delegation` claim might be used to express that the referenced party may sign a legal document on behalf of the subject. This claim is used in this specification in [Section 7.1](#trust_mark_claims){.auto .internal .xref} to represent delegation of the right to issue Trust Marks with a particular identifier.





### "logo_uri" (Logo URI) Claim

The `logo_uri` claim value is a URI that references a logo pertaining to the JWT. Use of this claim is OPTIONAL.

For instance, the `logo_uri` claim might be used to represent the location from which to retrieve the logo of an organization to display in a user interface. This claim is used in this specification in [Section 7.1](#trust_mark_claims){.auto .internal .xref} to convey a URL that references a logo for the Entity.







## Claims Languages and Scripts

Human-readable claim values and claim values that reference human-readable values MAY be represented in multiple languages and scripts. This specification enables such representations in the same manner as defined in Section 5.2 of [OpenID Connect Core 1.0](#OpenID.Core){.internal .xref} [[OpenID.Core](#OpenID.Core){.cite .xref}].

As described in OpenID Connect Core, to specify the languages and scripts, [BCP47](#RFC5646){.internal .xref} [[RFC5646](#RFC5646){.cite .xref}] language tags are added to member names, delimited by a `#` character. For example, `family_name#ja-Kana-JP` expresses the Family Name in Katakana in Japanese, which is commonly used to index and represent the phonetics of the Kanji representation of the same name represented as `family_name#ja-Hani-JP`.

Language tags can be used in any data structures containing or referencing human-readable values, including metadata parameters and Trust Mark parameters. For instance, both `organization_name` and `organization_name#de` might occur together in metadata.





## Media Types

These media types [[RFC2046](#RFC2046){.cite .xref}] are defined by this specification.



### "application/entity-statement+jwt" Media Type

The `application/entity-statement+jwt` media type is used to specify that the associated content is an Entity Statement, as defined in [Section 3](#entity-statement){.auto .internal .xref}. No parameters are used with this media type.





### "application/trust-mark+jwt" Media Type

The `application/trust-mark+jwt` media type is used to specify that the associated content is a Trust Mark, as defined in [Section 7](#trust_marks){.auto .internal .xref}. No parameters are used with this media type.





### "application/resolve-response+jwt" Media Type

The `application/resolve-response+jwt` media type is used to specify that the associated content is a Resolve Response, as defined in [Section 8.3.2](#resolve-response){.auto .internal .xref}. No parameters are used with this media type.





### "application/trust-chain+json" Media Type

The `application/trust-chain+json` media type is used to specify that the associated content is a JSON array representing a Trust Chain, as defined in [Section 4](#trust_chain){.auto .internal .xref}. No parameters are used with this media type.





### "application/trust-mark-delegation+jwt" Media Type

The `application/trust-mark-delegation+jwt` media type is used to specify that the associated content is a Trust Mark delegation, as defined in [Section 7.2.1](#delegation_jwt){.auto .internal .xref}. No parameters are used with this media type.





### "application/jwk-set+jwt" Media Type

The `application/jwk-set+jwt` media type is used to specify that the associated content is a signed JWK Set, as defined in [Section 8.7.2](#HistKeysResp){.auto .internal .xref}. No parameters are used with this media type.





### "application/explicit-registration-response+jwt" Media Type

The `application/explicit-registration-response+jwt` media type is used to specify that the associated content is an Explicit Registration response, as defined in [Section 12.2.3](#cliregresp){.auto .internal .xref}. No parameters are used with this media type.





### "application/trust-mark-status-response+jwt" Media Type

The `application/trust-mark-status-response+jwt` media type is used to specify that the associated content is a Trust Mark Status Response, as defined in [Section 8.4.2](#tm-status-response){.auto .internal .xref}. No parameters are used with this media type.







## String Operations

Processing some OpenID Federation messages requires comparing values in the messages to other values. For example, the Entity Identifier in an `iss` claim might be compared to the Entity Identifier in a `sub` claim. Comparing Unicode [[UNICODE](#UNICODE){.cite .xref}] strings, however, has significant security implications.

Therefore, comparisons between JSON strings and other Unicode strings MUST be performed as specified below:

1.  
    Remove any JSON applied escaping to produce an array of Unicode code points.
    

2.  
    Unicode Normalization [[USA15](#USA15){.cite .xref}] MUST NOT be applied at any point to either the JSON string or to the string it is to be compared against.
    

3.  
    Comparisons between the two strings MUST be performed as a Unicode code point to code point equality comparison.
    

Note that this is the same comparison procedure as specified in Section 14 of [OpenID Connect Core 1.0](#OpenID.Core){.internal .xref} [[OpenID.Core](#OpenID.Core){.cite .xref}].





## Implementation Considerations

This section provides guidance to implementers and deployers of Federations on situations and properties that they should consider for their Federations.



### Federation Topologies

It is possible to construct Federation topologies that have multiple trust paths between Entities. The specification does not disallow this, but it can create ambiguities that deployers need to be aware of.

Consider the following Federation topology:

[]{#name-example-topology-with-multi}

<figure id="figure-53">
<div id="section-17.1-3.1" class="alignLeft art-text artwork">
<pre><code>              .--------------.
              | Trust Anchor |
              &#39;--.---.-----.-&#39;
                 |   |     |
              .--&#39;   &#39;--.  &#39;---------------.
              |         |                  |
.-------------v--.   .--v-------------.    |
| Intermediate 1 |   | Intermediate 2 |    |
&#39;-------------.--&#39;   &#39;--.-------------&#39;    |
              |         |                .-v--.
              |         |                | OP |
           .--v---------v---.            &#39;----&#39;
           | Intermediate 3 |
           &#39;-------.--------&#39;
                   |
                   |
                 .-v--.
                 | RP |
                 &#39;----&#39;</code></pre>
</div>
<figcaption><a href="#figure-53" class="selfRef">Figure 53</a>: <a href="#name-example-topology-with-multi" class="selfRef">Example topology with multiple trust paths between Entities</a></figcaption>
</figure>

In this topology, there are multiple trust paths between the RP and the Trust Anchor, meaning that multiple different Trust Chains could be built between them. If the metadata policies of Intermediate 1 and Intermediate 2 are different, this could result in the Resolved Metadata for the RP differing, depending upon which Intermediate is used when building the Trust Chain. Some such differences will be innocuous and some can cause failures.

It is the job of the Federation architects to deploy topologies and metadata policies that work as intended, no matter which trust path is chosen when building a Trust Chain. Of course, one way to avoid potential ambiguities is to only use topologies that are trees, without multiple paths between two Entities. Topologies that are not trees are permitted, but should be used consciously and with care.

Even when a Federation topology contains loops, Trust Chains built from them MUST NOT contain loops, as mandated in [Section 10.1](#fetching-es){.auto .internal .xref}.





### Federation Discovery and Trust Chain Resolution Patterns

This section describes different patterns that implementers may use for discovering entities within a federation and for resolving Trust Chains. It is important to distinguish between two related but distinct concepts:

- 
  **Discovery**: The process of finding entities that are part of a federation, typically to build directories or catalogs of available service providers.
  

- 
  **Trust Chain Resolution**: The process of building and validating a Trust Chain from a known entity to a Trust Anchor, as described in [Section 10](#resolving_trust){.auto .internal .xref}. This process is also known as Federation Entity Discovery in this specification (see the definition in the Terminology section).
  

While these patterns may involve both discovery and Trust Chain resolution, they serve different purposes and may be used independently depending on the use case. For example, a discovery service building a directory of OpenID Providers may collect entity identifiers without necessarily resolving Trust Chains for all discovered entities, since the actual Trust Chain resolution may occur later on, for instance: when an RP and OP engage in an authentication transaction, and according to the Trust Chain resolution process described in [Section 10](#resolving_trust){.auto .internal .xref}.

Implementers may support one or more of the following patterns:

- 
  [Top-Down Discovery](#top_down_discovery){.internal .xref} ([Section 17.2.1](#top_down_discovery){.auto .internal .xref}): Finding entities within a federation by starting from a Trust Anchor and traversing down the hierarchy.
  

- 
  [Bottom-Up Trust Chain Resolution](#bottom_up_discovery){.internal .xref} ([Section 17.2.2](#bottom_up_discovery){.auto .internal .xref}): Resolving Trust Chains starting from a known entity identifier, as described in [Section 10](#resolving_trust){.auto .internal .xref}.
  

- 
  [Single Point of Trust Resolution](#single_point_discovery){.internal .xref} ([Section 17.2.3](#single_point_discovery){.auto .internal .xref}): Delegating Trust Chain resolution to a trusted resolver implementing the Resolve Endpoint.
  

Federation operators may choose to support multiple patterns to accommodate different use cases and integration scenarios. The choice of supported patterns affects the federation's usability and the types of applications that can effectively integrate with it.



#### Top-Down Discovery

Top-down discovery is the process of finding entities that are part of a federation by starting from a known Trust Anchor and traversing down the federation hierarchy. This pattern is used when the goal is to discover available entities, particularly entities of specific Entity Types, without necessarily knowing their Entity Identifiers in advance.

This pattern is particularly useful for:

- 
  Service discovery by Relying Parties looking for OpenID Providers
  

- 
  Resource discovery by Clients looking for specific service types
  

- 
  Federation browsing and exploration
  

- 
  Building provider directories and catalogs (e.g., WAYF services, Seamless Access)
  

The top-down discovery process follows these steps:

1.  
    Start with a known Trust Anchor that the discovering entity trusts
    

2.  
    Use the list endpoint (as defined in [Section 8.2](#entity_listing){.auto .internal .xref}) to discover Immediate Subordinate entities
    

3.  
    Filter by `entity_type` parameter to find protocol-specific providers (e.g., `openid_provider`)
    

4.  
    For Intermediate entities, recursively traverse their Subordinates
    

5.  
    Collect Entity Identifiers and optionally Entity Configurations for discovered entities
    

Note that top-down discovery may or may not include Trust Chain resolution, depending on the use case. For example, when building a directory of OpenID Providers for user selection at login time, the discovery service may collect entity identifiers and basic metadata without resolving Trust Chains for all discovered entities. However, if the discovery service needs to verify that entities are properly registered in the federation before including them in the directory, it may choose to perform Trust Chain resolution as part of the discovery process.





#### Bottom-Up Trust Chain Resolution

Bottom-up Trust Chain resolution is the process described in [Section 10](#resolving_trust){.auto .internal .xref}, also known as Federation Entity Discovery (see the definition in the Terminology section). This process starts with a known Entity Identifier and builds a Trust Chain by traversing up the federation hierarchy until reaching a Trust Anchor. This pattern is not discovery in the sense of finding unknown entities, but rather trust resolution for a known entity.

This pattern is used when an entity needs to verify the trustworthiness of another entity whose Entity Identifier is already known. This is typically used for:

- 
  OpenID Providers verifying Relying Parties during authentication requests
  

- 
  Resource servers verifying Client trustworthiness
  

- 
  Any Entity validating incoming requests from known parties
  

- 
  Dynamic trust establishment in federated environments
  

The bottom-up Trust Chain resolution process follows these steps, as described in [Section 10](#resolving_trust){.auto .internal .xref}:

1.  
    Start with the subject entity's Entity Configuration (provided or fetched using the process defined in [Section 9](#federation_configuration){.auto .internal .xref})
    

2.  
    Use `authority_hints` to identify immediate superior entities
    

3.  
    Fetch Entity Configuration for each Superior Entity
    

4.  
    Use Fetch Endpoints (as defined in [Section 8.1.1](#fetch_statement){.auto .internal .xref}) to obtain Subordinate Statements about the subject entity
    

5.  
    Recursively traverse up the hierarchy until reaching a Trust Anchor
    

6.  
    Build and validate the complete Trust Chain
    

7.  
    Apply federation policies to derive resolved metadata
    





#### Single Point of Trust Resolution

Single point of trust resolution delegates the entire Trust Chain resolution process to a trusted resolver implementing the Resolve Endpoint defined in [Section 8.3](#resolve){.auto .internal .xref}. This pattern allows entities to offload the complexity of Trust Chain resolution to a specialized service.

This pattern is useful for:

- 
  Entities that want to offload Trust Chain resolution complexity
  

- 
  Centralized trust evaluation services
  

- 
  Performance optimization by caching resolved metadata
  

- 
  Simplified integration for lightweight clients
  

Note that this pattern requires the Entity Identifier of the subject entity to be known in advance. It does not provide discovery functionality in the sense of finding unknown entities within a federation.

The single point of trust resolution process follows these steps:

1.  
    Identify a trusted resolver with a Resolve Endpoint
    

2.  
    Submit the subject Entity Identifier and Trust Anchor to the resolver
    

3.  
    The resolver performs complete Trust Chain resolution internally (following the bottom-up pattern)
    

4.  
    The resolver returns resolved metadata and Trust Marks
    

5.  
    Optionally verify the resolver's own Trust Chain
    

6.  
    Use resolved metadata for protocol operations
    







### Trust Anchors and Resolvers Go Together

If only one resolver is present in a federation, that entity should be both Trust Anchor and Resolver. If so, users of the resolver will not have to collect and evaluate Trust Chains for the Resolver. The Trust Anchor is by definition trusted and if the entity also serves as Resolver, that service will be implicitly trusted.





### One Entity, One Service

Apart from letting an entity provide both the Trust Anchor and Resolver services, there is a good reason for having each entity only do one thing. The reason is that, later in time, it will be much easier to share specific services between federations.





### Trust Mark Policies

When validating trust marks in an Entity Statement, it can be split into three parts.

[]{.break}

Validating Trust Marks in the Context of Validating an Entity Statement
:   According to the text on Entity Statement Validation in [Section 3.5](#ESValidation){.auto .internal .xref}, validating a Trust Mark is confined to validating the syntax of the claim value, including that the `trust_mark_type` value is consistent.
:   

Validating a Specific Trust Mark
:   This is what is described in [Section 7.3](#trust-mark-validation){.auto .internal .xref}. In order to validate a Trust Mark, the entity must find a Trust Chain for the Trust Mark Issuer to a Trust Anchor the Entity trusts. This has nothing to do with which federation that will later be used for the application protocol.
:   

Deciding which Trust Marks to Use

:   A federation MAY have a policy that states that only Trust Marks that match certain criteria SHOULD be used.

    An example of such criteria could be that the Trust Mark's `trust_mark_type` must be listed in the Trust Anchor's `trust_mark_issuers`, and if so, that the instance's `iss` appears in the corresponding list of Entity Identifiers. Note that the list may be an empty list, which signifies that anyone can issue a Trust Mark with the `trust_mark_type` in question. Such Trust Marks can appear for various reasons, such as the Entity Configuration including Trust Marks associated with another federation, or Trust Marks intended for specific purposes or Entity audiences.

    An Entity MAY also choose, at its own discretion, to utilize Trust Marks presented to it that are not recognized within the federation, and where the accreditation authority is established by an out-of-band mechanism.

:   







## Security Considerations



### Denial-of-Service Attack Prevention

Some of the interfaces defined in this specification could be used for Denial-of-Service attacks (DoS), most notably, the resolve endpoint ([Section 8.2](#entity_listing){.auto .internal .xref}), Explicit Client Registration ([Section 12.2](#explicit){.auto .internal .xref}), and Automatic Client Registration ([Section 12.1](#automatic){.auto .internal .xref}) can be exploited as vectors of HTTP propagation attacks. Below is an explanation of how such an attack can occur and the countermeasures to prevent it.

An adversary, providing hundreds of fake `authority_hints` in its Entity Configuration, could exploit the Federation Entity Discovery mechanism to propagate many HTTP requests. Imagine an adversary controlling an RP that sends an authorization request to an OP. For each request crafted by the adversary, the OP produces one request for the adversary's Entity Configuration and another one for each URL found in the `authority_hints`.

If these endpoints are provided, some adequate defense methods are required, such as those described below and in [[RFC4732](#RFC4732){.cite .xref}].

Implementations should set a limit on the number of `authority_hints` they are willing to inspect. This is to protect against attacks where an adversary might define a large count of false `authority_hints` in their Entity Configuration.

Entities may be required to include a [Trust Chain](#trust_chain_head_param){.internal .xref} ([Section 4.3](#trust_chain_head_param){.auto .internal .xref}) in their requests, as explained in [Section 12.1.1.1](#UsingAuthzRequestObject){.auto .internal .xref}. The static Trust Chain gives a predefined trust path, meaning that Federation Entity Discovery need not be performed. In this case, the propagation attacks will be prevented since the Trust Chain can be statically validated with a public key of the Trust Anchor.

A Trust Mark can be statically validated using the public key of its issuer. The static validation of the Trust Marks represents a filter against propagation attacks. If the OpenID Provider (OP) discovers at least one valid Trust Mark within an Entity Configuration, this may serve as evidence of the reliability of the Relying Party that initiated the request. Given that the Trust Mark is optional, the decision to require one is at the discretion of the federation implementation, where a federation may define and require Trust Marks according to specific needs.

If Client authentication is not required at the resolve endpoint, then incoming requests should not automatically trigger the collection (Federation Entity Discovery process) and assessment of Trust Chains. Instead, the resolve endpoint should only respond to unauthenticated Client requests with cached information about Entities that have already been evaluated and deemed trustworthy. The initiation of the Federation Entity Discovery process should not be the default action for the resolve endpoint in this case.

Passing request objects by reference (using the `request_uri` request parameter) may not be supported by some deployments, as described in [Section 12.1.1](#authn-request){.auto .internal .xref}, to eliminate a mechanism by which an attacker could otherwise require OPs to retrieve arbitrary content under the control of the attacker.





### Unsigned Error Messages

One of the fundamental design goals of this protocol is to protect messages end-to-end. This cannot be accomplished by demanding TLS since TLS, in lots of cases, is not end-to-end but ends in an HTTPS to HTTP Reverse Proxy. Allowing unsigned error messages therefore opens an attack vector for someone who wants to run a Denial-of-Service attack. This is not specific to OpenID Federation but equally valid for other protocols when HTTPS to HTTP reverse proxies are used.







## Privacy Considerations

Implementers should be aware of these privacy considerations:

[]{.break}

Entity Statements
:   Entity Statements are designed to establish trust relationships between organizational entities within a federation, rather than between individuals or for specific business applications. Trust and reputation assessments for individuals or legal entities, such as those required for Know Your Customer and Anti-Money Laundering processes, should be managed through specialized platforms tailored for those purposes. Given that Entity Statements facilitate trust relationships using a public infrastructure, they should be limited to the essential information necessary for federation operations and organizational trust establishment.
:   

Trust Mark Status
:   The Trust Mark Status endpoint enables querying the status of Trust Marks in real time. Similar to the Federation Fetch endpoint, in cases where the Trust Mark Status endpoint is not protected by any client authentication method, requests to validate Trust Marks may not necessarily indicate an actual interaction or relationship between Entities, as they could simply be part of routine network inspection or discovery processes. This could potentially enable Trust Mark issuers to track Entities evaluating Trust Marks about other Entities through standard network diagnostic tools like IPv4/IPv6 addresses and DNS Whois entries. To mitigate tracking risks, implementers can use short-lived Trust Marks, or use the Trust Marked Entities Listing ([Section 8.5](#tm_listing){.auto .internal .xref}) with only the `trust_mark_type` parameter and not the `sub` parameter, reducing the need to use the Trust Mark Status endpoint.
:   

Federation Fetch Endpoint
:   The Federation Fetch endpoint enables querying Subordinate Statements in real time. Similar to Trust Mark Status validation, in the cases where the federation infrastructure is public and widely browsable and endpoints are not protected by any client authentication method, requests to fetch Subordinate Statements may not necessarily indicate an actual interaction or relationship between Entities, as they could simply be part of routine network inspection or discovery processes. However, this could potentially enable Trust Anchors or Intermediates to track Entities evaluating trust relationships with other Entities through standard network diagnostic tools like IPv4/IPv6 addresses and DNS Whois entries. To mitigate tracking risks around Entities inspecting and interacting with other Entities, implementers should consider using static and short-lived Trust Chains where appropriate, which can reduce the need for real-time fetching of Subordinate Statements.
:   





## IANA Considerations



### OAuth Authorization Server Metadata Registration

This specification registers the following metadata entries in the IANA "OAuth Authorization Server Metadata" registry [[IANA.OAuth.Parameters](#IANA.OAuth.Parameters){.cite .xref}] established by [[RFC8414](#RFC8414){.cite .xref}].



#### Registry Contents

- 
  Metadata Name: `client_registration_types_supported`
  

- 
  Metadata Description: Client Registration Types Supported
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.1.3](#OP_metadata){.auto .internal .xref} of this specification
  



- 
  Metadata Name: `federation_registration_endpoint`
  

- 
  Metadata Description: Federation Registration Endpoint
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.1.3](#OP_metadata){.auto .internal .xref} of this specification
  



- 
  Metadata Name: `signed_jwks_uri`
  

- 
  Metadata Description: URL referencing a signed JWT having this authorization server's JWK Set document as its payload
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.1](#jwks_metadata){.auto .internal .xref} of this specification
  



- 
  Metadata Name: `jwks`
  

- 
  Metadata Description: JSON Web Key Set document, passed by value
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.1](#jwks_metadata){.auto .internal .xref} of this specification
  



- 
  Metadata Name: `organization_name`
  

- 
  Metadata Description: Human-readable name representing the organization owning this authorization server
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.2](#informational_metadata){.auto .internal .xref} of this specification
  



- 
  Metadata Name: `display_name`
  

- 
  Metadata Description: Human-readable name of the authorization server to be presented to the End-User
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.2](#informational_metadata){.auto .internal .xref} of this specification
  



- 
  Metadata Name: `description`
  

- 
  Metadata Description: Human-readable brief description of this authorization server presentable to the End-User
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.2](#informational_metadata){.auto .internal .xref} of this specification
  



- 
  Metadata Name: `keywords`
  

- 
  Metadata Description: JSON array with one or more strings representing search keywords, tags, categories, or labels that apply to this authorization server
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.2](#informational_metadata){.auto .internal .xref} of this specification
  



- 
  Metadata Name: `contacts`
  

- 
  Metadata Description: Array of strings representing ways to contact people responsible for this authorization server, typically email addresses
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.2](#informational_metadata){.auto .internal .xref} of this specification
  



- 
  Metadata Name: `logo_uri`
  

- 
  Metadata Description: URL that references a logo for the organization owning this authorization server
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.2](#informational_metadata){.auto .internal .xref} of this specification
  



- 
  Metadata Name: `information_uri`
  

- 
  Metadata Description: URL for documentation of additional information about this authorization server viewable by the End-User
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.2](#informational_metadata){.auto .internal .xref} of this specification
  



- 
  Metadata Name: `organization_uri`
  

- 
  Metadata Description: URL of a Web page for the organization owning this authorization server
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.2](#informational_metadata){.auto .internal .xref} of this specification
  







### OAuth Dynamic Client Registration Metadata Registration

This specification registers the following client metadata entries in the IANA "OAuth Dynamic Client Registration Metadata" registry [[IANA.OAuth.Parameters](#IANA.OAuth.Parameters){.cite .xref}] established by [[RFC7591](#RFC7591){.cite .xref}].



#### Registry Contents

- 
  Client Metadata Name: `client_registration_types`
  

- 
  Client Metadata Description: An array of strings specifying the client registration types the RP wants to use
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.1.2](#RP_metadata){.auto .internal .xref} of this specification
  



- 
  Client Metadata Name: `signed_jwks_uri`
  

- 
  Client Metadata Description: URL referencing a signed JWT having the client's JWK Set document as its payload
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.1](#jwks_metadata){.auto .internal .xref} of this specification
  



- 
  Client Metadata Name: `organization_name`
  

- 
  Client Metadata Description: Human-readable name representing the organization owning this client
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.2](#informational_metadata){.auto .internal .xref} of this specification
  



- 
  Client Metadata Name: `description`
  

- 
  Client Metadata Description: Human-readable brief description of this client presentable to the End-User
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.2](#informational_metadata){.auto .internal .xref} of this specification
  



- 
  Client Metadata Name: `keywords`
  

- 
  Client Metadata Description: JSON array with one or more strings representing search keywords, tags, categories, or labels that apply to this client
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.2](#informational_metadata){.auto .internal .xref} of this specification
  



- 
  Client Metadata Name: `information_uri`
  

- 
  Client Metadata Description: URL for documentation of additional information about this client viewable by the End-User
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.2](#informational_metadata){.auto .internal .xref} of this specification
  



- 
  Client Metadata Name: `organization_uri`
  

- 
  Client Metadata Description: URL of a Web page for the organization owning this client
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.2](#informational_metadata){.auto .internal .xref} of this specification
  







### OAuth Extensions Error Registration

This section registers the following values in the IANA "OAuth Extensions Error Registry" registry [[IANA.OAuth.Parameters](#IANA.OAuth.Parameters){.cite .xref}] established by [[RFC6749](#RFC6749){.cite .xref}].



#### Registry Contents

- 
  Name: invalid_request
  

- 
  Usage Location: authorization endpoint
  

- 
  Protocol Extension: OpenID Federation
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Reference: [Section 8.9](#error_response){.auto .internal .xref} of this specification
  



- 
  Name: invalid_client
  

- 
  Usage Location: authorization endpoint
  

- 
  Protocol Extension: OpenID Federation
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Reference: [Section 8.9](#error_response){.auto .internal .xref} of this specification
  



- 
  Name: invalid_issuer
  

- 
  Usage Location: authorization endpoint
  

- 
  Protocol Extension: OpenID Federation
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Reference: [Section 8.9](#error_response){.auto .internal .xref} of this specification
  



- 
  Name: invalid_subject
  

- 
  Usage Location: authorization endpoint
  

- 
  Protocol Extension: OpenID Federation
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Reference: [Section 8.9](#error_response){.auto .internal .xref} of this specification
  



- 
  Name: invalid_trust_anchor
  

- 
  Usage Location: authorization endpoint
  

- 
  Protocol Extension: OpenID Federation
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Reference: [Section 8.9](#error_response){.auto .internal .xref} of this specification
  



- 
  Name: invalid_trust_chain
  

- 
  Usage Location: authorization endpoint
  

- 
  Protocol Extension: OpenID Federation
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Reference: [Section 8.9](#error_response){.auto .internal .xref} of this specification
  



- 
  Name: invalid_metadata
  

- 
  Usage Location: authorization endpoint
  

- 
  Protocol Extension: OpenID Federation
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Reference: [Section 8.9](#error_response){.auto .internal .xref} of this specification
  



- 
  Name: not_found
  

- 
  Usage Location: authorization endpoint
  

- 
  Protocol Extension: OpenID Federation
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Reference: [Section 8.9](#error_response){.auto .internal .xref} of this specification
  



- 
  Name: server_error
  

- 
  Usage Location: authorization endpoint
  

- 
  Protocol Extension: OpenID Federation
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Reference: [Section 8.9](#error_response){.auto .internal .xref} of this specification
  



- 
  Name: temporarily_unavailable
  

- 
  Usage Location: authorization endpoint
  

- 
  Protocol Extension: OpenID Federation
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Reference: [Section 8.9](#error_response){.auto .internal .xref} of this specification
  



- 
  Name: unsupported_parameter
  

- 
  Usage Location: authorization endpoint
  

- 
  Protocol Extension: OpenID Federation
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Reference: [Section 8.9](#error_response){.auto .internal .xref} of this specification
  







### Media Type Registration

This section registers the following media types [[RFC2046](#RFC2046){.cite .xref}] in the "Media Types" registry [[IANA.MediaTypes](#IANA.MediaTypes){.cite .xref}] in the manner described in [[RFC6838](#RFC6838){.cite .xref}].



#### Registry Contents

- 
  Type name: application
  

- 
  Subtype name: entity-statement+jwt
  

- 
  Required parameters: n/a
  

- 
  Optional parameters: n/a
  

- 
  Encoding considerations: binary; An Entity Statement is a JWT; JWT values are encoded as a series of base64url-encoded values (some of which may be the empty string) separated by period ('.') characters.
  

- 
  Security considerations: See [Section 18](#Security){.auto .internal .xref} of this specification
  

- 
  Interoperability considerations: n/a
  

- 
  Published specification: [Section 15.1](#entity-statement_jwt){.auto .internal .xref} of this specification
  

- 
  Applications that use this media type: Applications that use this specification
  

- 
  Fragment identifier considerations: n/a
  

- 
  Additional information:

  - 
    Magic number(s): n/a
    

  - 
    File extension(s): n/a
    

  - 
    Macintosh file type code(s): n/a
    
  

- 
  Person & email address to contact for further information:

  Michael B. Jones, michael_b_jones@hotmail.com
  

- 
  Intended usage: COMMON
  

- 
  Restrictions on usage: none
  

- 
  Author: Michael B. Jones, michael_b_jones@hotmail.com
  

- 
  Change controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Provisional registration? No
  



- 
  Type name: application
  

- 
  Subtype name: trust-mark+jwt
  

- 
  Required parameters: n/a
  

- 
  Optional parameters: n/a
  

- 
  Encoding considerations: binary; A Trust Mark is a JWT; JWT values are encoded as a series of base64url-encoded values (some of which may be the empty string) separated by period ('.') characters.
  

- 
  Security considerations: See [Section 18](#Security){.auto .internal .xref} of this specification
  

- 
  Interoperability considerations: n/a
  

- 
  Published specification: [Section 15.2](#trust-mark_jwt){.auto .internal .xref} of this specification
  

- 
  Applications that use this media type: Applications that use this specification
  

- 
  Fragment identifier considerations: n/a
  

- 
  Additional information:

  - 
    Magic number(s): n/a
    

  - 
    File extension(s): n/a
    

  - 
    Macintosh file type code(s): n/a
    
  

- 
  Person & email address to contact for further information:

  Michael B. Jones, michael_b_jones@hotmail.com
  

- 
  Intended usage: COMMON
  

- 
  Restrictions on usage: none
  

- 
  Author: Michael B. Jones, michael_b_jones@hotmail.com
  

- 
  Change controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Provisional registration? No
  



- 
  Type name: application
  

- 
  Subtype name: resolve-response+jwt
  

- 
  Required parameters: n/a
  

- 
  Optional parameters: n/a
  

- 
  Encoding considerations: binary; An Entity Resolve Response is a signed JWT; JWT values are encoded as a series of base64url-encoded values (some of which may be the empty string) separated by period ('.') characters.
  

- 
  Security considerations: See [Section 18](#Security){.auto .internal .xref} of this specification
  

- 
  Interoperability considerations: n/a
  

- 
  Published specification: [Section 15.3](#resolve-response_jwt){.auto .internal .xref} of this specification
  

- 
  Applications that use this media type: Applications that use this specification
  

- 
  Fragment identifier considerations: n/a
  

- 
  Additional information:

  - 
    Magic number(s): n/a
    

  - 
    File extension(s): n/a
    

  - 
    Macintosh file type code(s): n/a
    
  

- 
  Person & email address to contact for further information:

  Michael B. Jones, michael_b_jones@hotmail.com
  

- 
  Intended usage: COMMON
  

- 
  Restrictions on usage: none
  

- 
  Author: Michael B. Jones, michael_b_jones@hotmail.com
  

- 
  Change controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Provisional registration? No
  



- 
  Type name: application
  

- 
  Subtype name: trust-chain+json
  

- 
  Required parameters: n/a
  

- 
  Optional parameters: n/a
  

- 
  Encoding considerations: binary; A Trust Chain is a JSON array of JWTs; JWT values are encoded as a series of base64url-encoded values (some of which may be the empty string) separated by period ('.') characters.
  

- 
  Security considerations: See [Section 18](#Security){.auto .internal .xref} of this specification
  

- 
  Interoperability considerations: n/a
  

- 
  Published specification: [Section 15.4](#trust-chain_json){.auto .internal .xref} of this specification
  

- 
  Applications that use this media type: Applications that use this specification
  

- 
  Fragment identifier considerations: n/a
  

- 
  Additional information:

  - 
    Magic number(s): n/a
    

  - 
    File extension(s): n/a
    

  - 
    Macintosh file type code(s): n/a
    
  

- 
  Person & email address to contact for further information:

  Michael B. Jones, michael_b_jones@hotmail.com
  

- 
  Intended usage: COMMON
  

- 
  Restrictions on usage: none
  

- 
  Author: Michael B. Jones, michael_b_jones@hotmail.com
  

- 
  Change controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Provisional registration? No
  



- 
  Type name: application
  

- 
  Subtype name: trust-mark-delegation+jwt
  

- 
  Required parameters: n/a
  

- 
  Optional parameters: n/a
  

- 
  Encoding considerations: binary; A Trust Mark delegation is a signed JWT; JWT values are encoded as a series of base64url-encoded values (some of which may be the empty string) separated by period ('.') characters.
  

- 
  Security considerations: See [Section 18](#Security){.auto .internal .xref} of this specification
  

- 
  Interoperability considerations: n/a
  

- 
  Published specification: [Section 15.5](#trust-mark-delegation_jwt){.auto .internal .xref} of this specification
  

- 
  Applications that use this media type: Applications that use this specification
  

- 
  Fragment identifier considerations: n/a
  

- 
  Additional information:

  - 
    Magic number(s): n/a
    

  - 
    File extension(s): n/a
    

  - 
    Macintosh file type code(s): n/a
    
  

- 
  Person & email address to contact for further information:

  Roland Hedberg, roland@catalogix.se
  

- 
  Intended usage: COMMON
  

- 
  Restrictions on usage: none
  

- 
  Author: Roland Hedberg, roland@catalogix.se
  

- 
  Change controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Provisional registration? No
  



- 
  Type name: application
  

- 
  Subtype name: jwk-set+jwt
  

- 
  Required parameters: n/a
  

- 
  Optional parameters: n/a
  

- 
  Encoding considerations: binary; A signed JWK Set is a signed JWT; JWT values are encoded as a series of base64url-encoded values (some of which may be the empty string) separated by period ('.') characters.
  

- 
  Security considerations: See [Section 18](#Security){.auto .internal .xref} of this specification
  

- 
  Interoperability considerations: n/a
  

- 
  Published specification: [Section 15.6](#jwk-set_jwt){.auto .internal .xref} of this specification
  

- 
  Applications that use this media type: Applications that use this specification
  

- 
  Fragment identifier considerations: n/a
  

- 
  Additional information:

  - 
    Magic number(s): n/a
    

  - 
    File extension(s): n/a
    

  - 
    Macintosh file type code(s): n/a
    
  

- 
  Person & email address to contact for further information:

  Michael B. Jones, michael_b_jones@hotmail.com
  

- 
  Intended usage: COMMON
  

- 
  Restrictions on usage: none
  

- 
  Author: Michael B. Jones, michael_b_jones@hotmail.com
  

- 
  Change controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Provisional registration? No
  



- 
  Type name: application
  

- 
  Subtype name: explicit-registration-response+jwt
  

- 
  Required parameters: n/a
  

- 
  Optional parameters: n/a
  

- 
  Encoding considerations: binary; An Explicit Registration response is a JWT; JWT values are encoded as a series of base64url-encoded values (some of which may be the empty string) separated by period ('.') characters.
  

- 
  Security considerations: See [Section 18](#Security){.auto .internal .xref} of this specification
  

- 
  Interoperability considerations: n/a
  

- 
  Published specification: [Section 15.7](#explicit-registration-response_jwt){.auto .internal .xref} of this specification
  

- 
  Applications that use this media type: Applications that use this specification
  

- 
  Fragment identifier considerations: n/a
  

- 
  Additional information:

  - 
    Magic number(s): n/a
    

  - 
    File extension(s): n/a
    

  - 
    Macintosh file type code(s): n/a
    
  

- 
  Person & email address to contact for further information:

  Michael B. Jones, michael_b_jones@hotmail.com
  

- 
  Intended usage: COMMON
  

- 
  Restrictions on usage: none
  

- 
  Author: Michael B. Jones, michael_b_jones@hotmail.com
  

- 
  Change controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Provisional registration? No
  



- 
  Type name: application
  

- 
  Subtype name: trust-mark-status-response+jwt
  

- 
  Required parameters: n/a
  

- 
  Optional parameters: n/a
  

- 
  Encoding considerations: binary; A Trust Mark Status Response is a signed JWT; JWT values are encoded as a series of base64url-encoded values (some of which may be the empty string) separated by period ('.') characters.
  

- 
  Security considerations: See [Section 18](#Security){.auto .internal .xref} of this specification
  

- 
  Interoperability considerations: n/a
  

- 
  Published specification: [Section 15.8](#trust-mark-status-response_jwt){.auto .internal .xref} of this specification
  

- 
  Applications that use this media type: Applications that use this specification
  

- 
  Fragment identifier considerations: n/a
  

- 
  Additional information:

  - 
    Magic number(s): n/a
    

  - 
    File extension(s): n/a
    

  - 
    Macintosh file type code(s): n/a
    
  

- 
  Person & email address to contact for further information:

  Michael B. Jones, michael_b_jones@hotmail.com
  

- 
  Intended usage: COMMON
  

- 
  Restrictions on usage: none
  

- 
  Author: Michael B. Jones, michael_b_jones@hotmail.com
  

- 
  Change controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Provisional registration? No
  







### OAuth Parameters Registration

This specification registers the following parameter name in the IANA "OAuth Parameters" registry [[IANA.OAuth.Parameters](#IANA.OAuth.Parameters){.cite .xref}] established by [[RFC6749](#RFC6749){.cite .xref}].



#### Registry Contents

- 
  Parameter Name: `trust_chain`
  

- 
  Parameter Usage Location: authorization request
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 12.1.1.1.1](#trust_chain_authz){.auto .internal .xref} of this specification
  







### JSON Web Signature and Encryption Header Parameters Registration

This specification registers the following JWS header parameter in the IANA "JSON Web Signature and Encryption Header Parameters" registry [[IANA.JOSE](#IANA.JOSE){.cite .xref}] established by [[RFC7515](#RFC7515){.cite .xref}].



#### Registry Contents

- 
  Header Parameter Name: `trust_chain`
  

- 
  Header Parameter Description: OpenID Federation Trust Chain
  

- 
  Header Parameter Usage Location: JWS
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 4](#trust_chain){.auto .internal .xref} of this specification
  







### JSON Web Token Claims Registration

This specification registers the following claims in the IANA "JSON Web Token Claims" registry [[IANA.JWT.Claims](#IANA.JWT.Claims){.cite .xref}] established by [[RFC7519](#RFC7519){.cite .xref}].



#### Registry Contents

- 
  Claim Name: `jwks`
  

- 
  Claim Description: JSON Web Key Set
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 13.1](#jwksClaim){.auto .internal .xref} of this specification
  



- 
  Claim Name: `metadata`
  

- 
  Claim Description: Metadata object
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 13.2](#metadataClaim){.auto .internal .xref} of this specification
  



- 
  Claim Name: `constraints`
  

- 
  Claim Description: Constraints object
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 13.3](#constraintsClaim){.auto .internal .xref} of this specification
  



- 
  Claim Name: `crit`
  

- 
  Claim Description: Critical Claim
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 13.4](#critClaim){.auto .internal .xref} of this specification
  



- 
  Claim Name: `ref`
  

- 
  Claim Description: Reference
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 13.5](#refClaim){.auto .internal .xref} of this specification
  



- 
  Claim Name: `delegation`
  

- 
  Claim Description: Delegation
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 13.6](#delegationClaim){.auto .internal .xref} of this specification
  



- 
  Claim Name: `logo_uri`
  

- 
  Claim Description: URI referencing a logo
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 13.7](#logo_uriClaim){.auto .internal .xref} of this specification
  



- 
  Claim Name: `authority_hints`
  

- 
  Claim Description: Authority Hints
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 3](#entity-statement){.auto .internal .xref} of this specification
  



- 
  Claim Name: `metadata_policy`
  

- 
  Claim Description: Metadata Policy object
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 3](#entity-statement){.auto .internal .xref} of this specification
  



- 
  Claim Name: `metadata_policy_crit`
  

- 
  Claim Description: Critical Metadata Policy Operators
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 3](#entity-statement){.auto .internal .xref} of this specification
  



- 
  Claim Name: `trust_marks`
  

- 
  Claim Description: Trust Marks
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 3](#entity-statement){.auto .internal .xref} of this specification
  



- 
  Claim Name: `trust_mark_issuers`
  

- 
  Claim Description: Trust Mark Issuers
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 3](#entity-statement){.auto .internal .xref} of this specification
  



- 
  Claim Name: `trust_mark_owners`
  

- 
  Claim Description: Trust Mark Owners
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 3](#entity-statement){.auto .internal .xref} of this specification
  



- 
  Claim Name: `source_endpoint`
  

- 
  Claim Description: Source Endpoint URL
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 3](#entity-statement){.auto .internal .xref} of this specification
  



- 
  Claim Name: `trust_chain`
  

- 
  Claim Description: Trust Chain
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 4](#trust_chain){.auto .internal .xref} of this specification
  



- 
  Claim Name: `keys`
  

- 
  Claim Description: Array of JWK values in a JWK Set
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2](#common_metadata){.auto .internal .xref} of this specification
  



- 
  Claim Name: `trust_mark_type`
  

- 
  Claim Description: Trust Mark Type Identifier
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 7.1](#trust_mark_claims){.auto .internal .xref} of this specification
  



- 
  Claim Name: `trust_anchor`
  

- 
  Claim Description: Trust Anchor ID
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 12.2.2](#ExplicitRegOP){.auto .internal .xref} of this specification
  







### JSON Web Key Parameters Registration

This specification registers the following parameters in the IANA "JSON Web Key Parameters" registry [[IANA.JOSE](#IANA.JOSE){.cite .xref}] established by [[RFC7517](#RFC7517){.cite .xref}].



#### Registry Contents

- 
  Parameter Name: `iat`
  

- 
  Parameter Description: Issued At, as defined in RFC 7519
  

- 
  Used with "kty" Value(s): *
  

- 
  Parameter Information Class: Public
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 8.7.2](#HistKeysResp){.auto .internal .xref} of this specification
  



- 
  Parameter Name: `nbf`
  

- 
  Parameter Description: Not Before, as defined in RFC 7519
  

- 
  Used with "kty" Value(s): *
  

- 
  Parameter Information Class: Public
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 8.7.2](#HistKeysResp){.auto .internal .xref} of this specification
  



- 
  Parameter Name: `exp`
  

- 
  Parameter Description: Expiration Time, as defined in RFC 7519
  

- 
  Used with "kty" Value(s): *
  

- 
  Parameter Information Class: Public
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 8.7.2](#HistKeysResp){.auto .internal .xref} of this specification
  



- 
  Parameter Name: `revoked`
  

- 
  Parameter Description: Revoked Key Properties
  

- 
  Used with "kty" Value(s): *
  

- 
  Parameter Information Class: Public
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 8.7.2](#HistKeysResp){.auto .internal .xref} of this specification
  







### Well-Known URI Registry

This specification registers the following well-known URI in the IANA "Well-Known URIs" registry [[IANA.well-known](#IANA.well-known){.cite .xref}] established by [[RFC5785](#RFC5785){.cite .xref}].



#### Registry Contents

- 
  URI suffix: `openid-federation`
  

- 
  Change controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification document: [Section 9](#federation_configuration){.auto .internal .xref} of this specification
  

- 
  Related information: (none)
  







### OAuth Protected Resource Metadata Registration

This specification registers the following protected resource metadata entries in the IANA "OAuth Protected Resource Metadata" registry [[IANA.OAuth.Parameters](#IANA.OAuth.Parameters){.cite .xref}] established by [[RFC9728](#RFC9728){.cite .xref}].



#### Registry Contents

- 
  Metadata Name: `signed_jwks_uri`
  

- 
  Metadata Description: URL referencing a signed JWT having the protected resource's JWK Set document as its payload
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.1](#jwks_metadata){.auto .internal .xref} of this specification
  



- 
  Metadata Name: `jwks`
  

- 
  Metadata Description: JSON Web Key Set document, passed by value
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.1](#jwks_metadata){.auto .internal .xref} of this specification
  



- 
  Metadata Name: `organization_name`
  

- 
  Metadata Description: Human-readable name representing the organization owning this protected resource
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.2](#informational_metadata){.auto .internal .xref} of this specification
  



- 
  Metadata Name: `description`
  

- 
  Metadata Description: Human-readable brief description of this protected resource presentable to the End-User
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.2](#informational_metadata){.auto .internal .xref} of this specification
  



- 
  Metadata Name: `keywords`
  

- 
  Metadata Description: JSON array with one or more strings representing search keywords, tags, categories, or labels that apply to this protected resource
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.2](#informational_metadata){.auto .internal .xref} of this specification
  



- 
  Metadata Name: `contacts`
  

- 
  Metadata Description: Array of strings representing ways to contact people responsible for this protected resource, typically email addresses
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.2](#informational_metadata){.auto .internal .xref} of this specification
  



- 
  Metadata Name: `logo_uri`
  

- 
  Metadata Description: URL that references a logo for the organization owning this protected resource
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.2](#informational_metadata){.auto .internal .xref} of this specification
  



- 
  Metadata Name: `organization_uri`
  

- 
  Metadata Description: URL of a Web page for the organization owning this protected resource
  

- 
  Change Controller: OpenID Foundation Artifact Binding Working Group - openid-specs-ab@lists.openid.net
  

- 
  Specification Document(s): [Section 5.2.2](#informational_metadata){.auto .internal .xref} of this specification
  








## References


### Normative References

[OpenID.Core]
:   [Sakimura, N.]{.refAuthor}, [Bradley, J.]{.refAuthor}, [Jones, M.B.]{.refAuthor}, [de Medeiros, B.]{.refAuthor}, and [C. Mortimore]{.refAuthor}, ["OpenID Connect Core 1.0"]{.refTitle}, 15 December 2023, <<https://openid.net/specs/openid-connect-core-1_0.html>>.
:   

[OpenID.Discovery]
:   [Sakimura, N.]{.refAuthor}, [Bradley, J.]{.refAuthor}, [Jones, M.B.]{.refAuthor}, and [E. Jay]{.refAuthor}, ["OpenID Connect Discovery 1.0"]{.refTitle}, 15 December 2023, <<https://openid.net/specs/openid-connect-discovery-1_0.html>>.
:   

[OpenID.Registration]
:   [Sakimura, N.]{.refAuthor}, [Bradley, J.]{.refAuthor}, and [M.B. Jones]{.refAuthor}, ["OpenID Connect Dynamic Client Registration 1.0"]{.refTitle}, 15 December 2023, <<https://openid.net/specs/openid-connect-registration-1_0.html>>.
:   

[OpenID.RP.Choices]
:   [Jones, M.B.]{.refAuthor}, [Hedberg, R.]{.refAuthor}, [Bradley, J.]{.refAuthor}, and [F. Skokan]{.refAuthor}, ["OpenID Connect Relying Party Metadata Choices 1.0"]{.refTitle}, 24 April 2025, <<https://openid.net/specs/openid-connect-rp-metadata-choices-1_0.html>>.
:   

[RFC2119]
:   [Bradner, S.]{.refAuthor}, ["Key words for use in RFCs to Indicate Requirement Levels"]{.refTitle}, [BCP 14]{.seriesInfo}, [RFC 2119]{.seriesInfo}, [DOI 10.17487/RFC2119]{.seriesInfo}, March 1997, <<https://www.rfc-editor.org/info/rfc2119>>.
:   

[RFC4732]
:   [Handley, M., Ed.]{.refAuthor}, [Rescorla, E., Ed.]{.refAuthor}, and [IAB]{.refAuthor}, ["Internet Denial-of-Service Considerations"]{.refTitle}, [RFC 4732]{.seriesInfo}, [DOI 10.17487/RFC4732]{.seriesInfo}, December 2006, <<https://www.rfc-editor.org/info/rfc4732>>.
:   

[RFC5280]
:   [Cooper, D.]{.refAuthor}, [Santesson, S.]{.refAuthor}, [Farrell, S.]{.refAuthor}, [Boeyen, S.]{.refAuthor}, [Housley, R.]{.refAuthor}, and [W. Polk]{.refAuthor}, ["Internet X.509 Public Key Infrastructure Certificate and Certificate Revocation List (CRL) Profile"]{.refTitle}, [RFC 5280]{.seriesInfo}, [DOI 10.17487/RFC5280]{.seriesInfo}, May 2008, <<https://www.rfc-editor.org/info/rfc5280>>.
:   

[RFC5646]
:   [Phillips, A., Ed.]{.refAuthor} and [M. Davis, Ed.]{.refAuthor}, ["Tags for Identifying Languages"]{.refTitle}, [BCP 47]{.seriesInfo}, [RFC 5646]{.seriesInfo}, [DOI 10.17487/RFC5646]{.seriesInfo}, September 2009, <<https://www.rfc-editor.org/info/rfc5646>>.
:   

[RFC6749]
:   [Hardt, D., Ed.]{.refAuthor}, ["The OAuth 2.0 Authorization Framework"]{.refTitle}, [RFC 6749]{.seriesInfo}, [DOI 10.17487/RFC6749]{.seriesInfo}, October 2012, <<https://www.rfc-editor.org/info/rfc6749>>.
:   

[RFC7515]
:   [Jones, M.]{.refAuthor}, [Bradley, J.]{.refAuthor}, and [N. Sakimura]{.refAuthor}, ["JSON Web Signature (JWS)"]{.refTitle}, [RFC 7515]{.seriesInfo}, [DOI 10.17487/RFC7515]{.seriesInfo}, May 2015, <<https://www.rfc-editor.org/info/rfc7515>>.
:   

[RFC7516]
:   [Jones, M.]{.refAuthor} and [J. Hildebrand]{.refAuthor}, ["JSON Web Encryption (JWE)"]{.refTitle}, [RFC 7516]{.seriesInfo}, [DOI 10.17487/RFC7516]{.seriesInfo}, May 2015, <<https://www.rfc-editor.org/info/rfc7516>>.
:   

[RFC7517]
:   [Jones, M.]{.refAuthor}, ["JSON Web Key (JWK)"]{.refTitle}, [RFC 7517]{.seriesInfo}, [DOI 10.17487/RFC7517]{.seriesInfo}, May 2015, <<https://www.rfc-editor.org/info/rfc7517>>.
:   

[RFC7519]
:   [Jones, M.]{.refAuthor}, [Bradley, J.]{.refAuthor}, and [N. Sakimura]{.refAuthor}, ["JSON Web Token (JWT)"]{.refTitle}, [RFC 7519]{.seriesInfo}, [DOI 10.17487/RFC7519]{.seriesInfo}, May 2015, <<https://www.rfc-editor.org/info/rfc7519>>.
:   

[RFC7591]
:   [Richer, J., Ed.]{.refAuthor}, [Jones, M.]{.refAuthor}, [Bradley, J.]{.refAuthor}, [Machulak, M.]{.refAuthor}, and [P. Hunt]{.refAuthor}, ["OAuth 2.0 Dynamic Client Registration Protocol"]{.refTitle}, [RFC 7591]{.seriesInfo}, [DOI 10.17487/RFC7591]{.seriesInfo}, July 2015, <<https://www.rfc-editor.org/info/rfc7591>>.
:   

[RFC7638]
:   [Jones, M.]{.refAuthor} and [N. Sakimura]{.refAuthor}, ["JSON Web Key (JWK) Thumbprint"]{.refTitle}, [RFC 7638]{.seriesInfo}, [DOI 10.17487/RFC7638]{.seriesInfo}, September 2015, <<https://www.rfc-editor.org/info/rfc7638>>.
:   

[RFC8174]
:   [Leiba, B.]{.refAuthor}, ["Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words"]{.refTitle}, [BCP 14]{.seriesInfo}, [RFC 8174]{.seriesInfo}, [DOI 10.17487/RFC8174]{.seriesInfo}, May 2017, <<https://www.rfc-editor.org/info/rfc8174>>.
:   

[RFC8259]
:   [Bray, T., Ed.]{.refAuthor}, ["The JavaScript Object Notation (JSON) Data Interchange Format"]{.refTitle}, [STD 90]{.seriesInfo}, [RFC 8259]{.seriesInfo}, [DOI 10.17487/RFC8259]{.seriesInfo}, December 2017, <<https://www.rfc-editor.org/info/rfc8259>>.
:   

[RFC8414]
:   [Jones, M.]{.refAuthor}, [Sakimura, N.]{.refAuthor}, and [J. Bradley]{.refAuthor}, ["OAuth 2.0 Authorization Server Metadata"]{.refTitle}, [RFC 8414]{.seriesInfo}, [DOI 10.17487/RFC8414]{.seriesInfo}, June 2018, <<https://www.rfc-editor.org/info/rfc8414>>.
:   

[RFC8705]
:   [Campbell, B.]{.refAuthor}, [Bradley, J.]{.refAuthor}, [Sakimura, N.]{.refAuthor}, and [T. Lodderstedt]{.refAuthor}, ["OAuth 2.0 Mutual-TLS Client Authentication and Certificate-Bound Access Tokens"]{.refTitle}, [RFC 8705]{.seriesInfo}, [DOI 10.17487/RFC8705]{.seriesInfo}, February 2020, <<https://www.rfc-editor.org/info/rfc8705>>.
:   

[RFC9101]
:   [Sakimura, N.]{.refAuthor}, [Bradley, J.]{.refAuthor}, and [M. Jones]{.refAuthor}, ["The OAuth 2.0 Authorization Framework: JWT-Secured Authorization Request (JAR)"]{.refTitle}, [RFC 9101]{.seriesInfo}, [DOI 10.17487/RFC9101]{.seriesInfo}, August 2021, <<https://www.rfc-editor.org/info/rfc9101>>.
:   

[RFC9126]
:   [Lodderstedt, T.]{.refAuthor}, [Campbell, B.]{.refAuthor}, [Sakimura, N.]{.refAuthor}, [Tonge, D.]{.refAuthor}, and [F. Skokan]{.refAuthor}, ["OAuth 2.0 Pushed Authorization Requests"]{.refTitle}, [RFC 9126]{.seriesInfo}, [DOI 10.17487/RFC9126]{.seriesInfo}, September 2021, <<https://www.rfc-editor.org/info/rfc9126>>.
:   

[RFC9728]
:   [Jones, M.B.]{.refAuthor}, [Hunt, P.]{.refAuthor}, and [A. Parecki]{.refAuthor}, ["OAuth 2.0 Protected Resource Metadata"]{.refTitle}, [RFC 9728]{.seriesInfo}, [DOI 10.17487/RFC9728]{.seriesInfo}, April 2025, <<https://www.rfc-editor.org/info/rfc9728>>.
:   

[UNICODE]
:   [The Unicode Consortium]{.refAuthor}, ["The Unicode Standard"]{.refTitle}, <<http://www.unicode.org/versions/latest/>>.
:   

[USA15]
:   [Whistler, K.]{.refAuthor}, ["Unicode Normalization Forms"]{.refTitle}, [Unicode Standard Annex 15]{.seriesInfo}, 12 August 2023, <<https://www.unicode.org/reports/tr15/>>.
:   



### Informative References

[FAPI]
:   [Sakimura, N.]{.refAuthor}, [Bradley, J.]{.refAuthor}, and [E. Jay]{.refAuthor}, ["Financial-grade API Security Profile 1.0 - Part 2: Advanced"]{.refTitle}, 12 March 2021, <<https://openid.net/specs/openid-financial-api-part-2-1_0.html>>.
:   

[IANA.JOSE]
:   [IANA]{.refAuthor}, ["JSON Object Signing and Encryption (JOSE)"]{.refTitle}, <<https://www.iana.org/assignments/jose>>.
:   

[IANA.JWT.Claims]
:   [IANA]{.refAuthor}, ["JSON Web Token Claims"]{.refTitle}, <<https://www.iana.org/assignments/jwt>>.
:   

[IANA.MediaTypes]
:   [IANA]{.refAuthor}, ["Media Types"]{.refTitle}, <<https://www.iana.org/assignments/media-types>>.
:   

[IANA.OAuth.Parameters]
:   [IANA]{.refAuthor}, ["OAuth Parameters"]{.refTitle}, <<https://www.iana.org/assignments/oauth-parameters>>.
:   

[IANA.well-known]
:   [IANA]{.refAuthor}, ["Well-Known URIs"]{.refTitle}, <<https://www.iana.org/assignments/well-known-uris>>.
:   

[RFC2046]
:   [Freed, N.]{.refAuthor} and [N. Borenstein]{.refAuthor}, ["Multipurpose Internet Mail Extensions (MIME) Part Two: Media Types"]{.refTitle}, [RFC 2046]{.seriesInfo}, [DOI 10.17487/RFC2046]{.seriesInfo}, November 1996, <<https://www.rfc-editor.org/info/rfc2046>>.
:   

[RFC5785]
:   [Nottingham, M.]{.refAuthor} and [E. Hammer-Lahav]{.refAuthor}, ["Defining Well-Known Uniform Resource Identifiers (URIs)"]{.refTitle}, [RFC 5785]{.seriesInfo}, [DOI 10.17487/RFC5785]{.seriesInfo}, April 2010, <<https://www.rfc-editor.org/info/rfc5785>>.
:   

[RFC6838]
:   [Freed, N.]{.refAuthor}, [Klensin, J.]{.refAuthor}, and [T. Hansen]{.refAuthor}, ["Media Type Specifications and Registration Procedures"]{.refTitle}, [BCP 13]{.seriesInfo}, [RFC 6838]{.seriesInfo}, [DOI 10.17487/RFC6838]{.seriesInfo}, January 2013, <<https://www.rfc-editor.org/info/rfc6838>>.
:   

[RFC8725]
:   [Sheffer, Y.]{.refAuthor}, [Hardt, D.]{.refAuthor}, and [M. Jones]{.refAuthor}, ["JSON Web Token Best Current Practices"]{.refTitle}, [BCP 225]{.seriesInfo}, [RFC 8725]{.seriesInfo}, [DOI 10.17487/RFC8725]{.seriesInfo}, February 2020, <<https://www.rfc-editor.org/info/rfc8725>>.
:   





## Example OpenID Provider Information Discovery and Client Registration

Let us assume the following: The project LIGO would like to offer access to its wiki to all OPs in eduGAIN. LIGO is registered with the InCommon federation.

The following depicts a federation under the eduGAIN Trust Anchor:

[]{#name-participants-within-the-edu}

<figure id="figure-54">
<div id="appendix-A-3.1" class="alignLeft art-text artwork">
<pre><code>                       eduGAIN
                          |
       +------------------+------------------+
       |                                     |
    SWAMID                               InCommon
       |                                     |
     umu.se                                  |
       |                                     |
   op.umu.se                           wiki.ligo.org</code></pre>
</div>
<figcaption><a href="#figure-54" class="selfRef">Figure 54</a>: <a href="#name-participants-within-the-edu" class="selfRef">Participants Within the eduGAIN Federation</a></figcaption>
</figure>

Both SWAMID and InCommon are identity federations in their own right. They also have in common that they are both members of the eduGAIN federation.

SWAMID and InCommon are different in how they register Entities. SWAMID registers organizations and lets the organizations register Entities that belong to the organization, while InCommon registers all Entities directly and not beneath any organization Entity. Hence the differences in depth in the federations.

Let us assume a researcher from Umeå University would like to login to the LIGO Wiki. At the Wiki, the researcher will use some kind of discovery service to find the home identity provider (op.umu.se).

Once the RP part of the Wiki knows which OP it SHOULD talk to, it has to find out a few things about the OP. All those things can be found in the metadata. But finding the metadata is not enough; the RP also has to trust the metadata.

Let us make a detour and start with what it takes to build a federation.



### Setting Up a Federation

These are the steps to set up a federation infrastructure:

- 
  Generation of Trust Anchor signing keys. These MUST be public/private key pairs.
  

- 
  Set up a signing service that can sign JWTs/Entity Statements using the Federation Entity Keys.
  

- 
  Set up web services that can publish signed Entity Statements, one for the URL corresponding to the federation's Entity Identifier returning an Entity Configuration and the other one providing the fetch endpoint, as described in [Section 8.1.1](#fetch_statement){.auto .internal .xref}.
  

Once these requirements have been satisfied, a Federation Operator can add Entities to the federation. Adding an Entity comes down to:

- 
  Providing the Entity with the federation's Entity Identifier and the public part of the key pairs used by the federation operator for signing Entity Statements.
  

- 
  Getting the Entity's Entity Identifier and the JWK Set that the Entity plans to publish in its Entity Configuration.
  

Before the federation operator starts adding Entities, there must be policies on who can be part of the federation and the layout of the federation. Is it supposed to be a one-layer federation like InCommon, a two-layer one like the SWAMID federation, or a multi-layer federation? The federation may also want to consider implementing other policies using the federation policy framework, as described in [Section 6](#federation_policy){.auto .internal .xref}.

With the federation in place, things can start happening.





### The LIGO Wiki Discovers the OP's Metadata

Federation Entity Discovery is a sequence of steps that starts with the RP fetching the Entity Configuration of the OP's Entity (in this case, https://op.umu.se) using the process defined in [Section 9](#federation_configuration){.auto .internal .xref}. What follows that is this sequence of steps:

1.  
    Pick out the Immediate Superior Entities using the authority hints.
    

2.  
    Fetch the Entity Configuration for each such Entity. This uses the process defined in [Section 9](#federation_configuration){.auto .internal .xref}.
    

3.  
    Use the fetch endpoint of each Immediate Superior to obtain Subordinate Statements about the Immediate Subordinate Entity, per [Section 8.1.1](#fetch_statement){.auto .internal .xref}.
    

How many times this has to be repeated depends on the depth of the federation. What follows below is the result of each step the RP has to take to find the OP's metadata using the federation setup described above.

When building the Trust Chain, the Subordinate Statements issued by each Immediate Superior about their Immediate Subordinates are used together with the Entity Configuration of the Trust Chain subject.

The Entity Configurations of Intermediates are not part of the Trust Chain.



#### Entity Configuration for https://op.umu.se

The LIGO WIKI RP fetches the Entity Configuration from the OP (op.umu.se) using the process defined in [Section 9](#federation_configuration){.auto .internal .xref}.

The result is this Entity Configuration:

[]{#name-entity-configuration-issued}

<figure id="figure-55">
<div id="appendix-A.2.1-3.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;authority_hints&quot;: [
    &quot;https://umu.se&quot;
  ],
  &quot;exp&quot;: 1568397247,
  &quot;iat&quot;: 1568310847,
  &quot;iss&quot;: &quot;https://op.umu.se&quot;,
  &quot;sub&quot;: &quot;https://op.umu.se&quot;,
  &quot;jwks&quot;: {
    &quot;keys&quot;: [
      {
        &quot;e&quot;: &quot;AQAB&quot;,
        &quot;kid&quot;: &quot;dEEtRjlzY3djcENuT01wOGxrZlkxb3RIQVJlMTY0...&quot;,
        &quot;kty&quot;: &quot;RSA&quot;,
        &quot;n&quot;: &quot;x97YKqc9Cs-DNtFrQ7_vhXoH9bwkDWW6En2jJ044yH...&quot;
      }
    ]
  },
  &quot;metadata&quot;: {
    &quot;openid_provider&quot;: {
      &quot;issuer&quot;: &quot;https://op.umu.se/openid&quot;,
      &quot;signed_jwks_uri&quot;: &quot;https://op.umu.se/openid/jwks.jose&quot;,
      &quot;authorization_endpoint&quot;:
        &quot;https://op.umu.se/openid/authorization&quot;,
      &quot;client_registration_types_supported&quot;: [
        &quot;automatic&quot;,
        &quot;explicit&quot;
      ],
      &quot;request_parameter_supported&quot;: true,
      &quot;grant_types_supported&quot;: [
        &quot;authorization_code&quot;,
        &quot;implicit&quot;,
        &quot;urn:ietf:params:oauth:grant-type:jwt-bearer&quot;
      ],
      &quot;id_token_signing_alg_values_supported&quot;: [
        &quot;ES256&quot;, &quot;RS256&quot;
      ],
      &quot;logo_uri&quot;:
        &quot;https://www.umu.se/img/umu-logo-left-neg-SE.svg&quot;,
      &quot;op_policy_uri&quot;:
        &quot;https://www.umu.se/en/website/legal-information/&quot;,
      &quot;response_types_supported&quot;: [
        &quot;code&quot;,
        &quot;code id_token&quot;,
        &quot;token&quot;
      ],
      &quot;subject_types_supported&quot;: [
        &quot;pairwise&quot;,
        &quot;public&quot;
      ],
      &quot;token_endpoint&quot;: &quot;https://op.umu.se/openid/token&quot;,
      &quot;federation_registration_endpoint&quot;:
        &quot;https://op.umu.se/openid/fedreg&quot;,
      &quot;token_endpoint_auth_methods_supported&quot;: [
        &quot;client_secret_post&quot;,
        &quot;client_secret_basic&quot;,
        &quot;client_secret_jwt&quot;,
        &quot;private_key_jwt&quot;
      ]
    }
  }
}</code></pre>
</div>
<figcaption><a href="#figure-55" class="selfRef">Figure 55</a>: <a href="#name-entity-configuration-issued" class="selfRef">Entity Configuration Issued by https://op.umu.se</a></figcaption>
</figure>

The `authority_hints` points to the Intermediate Entity `https://umu.se`. So that is the next step.

This Entity Configuration is the first link in the Trust Chain.





#### Entity Configuration for https://umu.se

The LIGO RP fetches the Entity Configuration from https://umu.se using the process defined in [Section 9](#federation_configuration){.auto .internal .xref}.

The request will look like this:

[]{#name-entity-configuration-issued-}

<figure id="figure-56">
<div id="appendix-A.2.2-3.1" class="alignLeft art-text artwork">
<pre><code>GET /.well-known/openid-federation HTTP/1.1
Host: umu.se</code></pre>
</div>
<figcaption><a href="#figure-56" class="selfRef">Figure 56</a>: <a href="#name-entity-configuration-issued-" class="selfRef">Entity Configuration Issued by https://umu.se</a></figcaption>
</figure>

And the GET will return:

[]{#name-entity-configuration-jwt-cl}

<figure id="figure-57">
<div id="appendix-A.2.2-5.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;authority_hints&quot;: [
    &quot;https://swamid.se&quot;
  ],
  &quot;exp&quot;: 1568397247,
  &quot;iat&quot;: 1568310847,
  &quot;iss&quot;: &quot;https://umu.se&quot;,
  &quot;sub&quot;: &quot;https://umu.se&quot;,
  &quot;jwks&quot;: {
    &quot;keys&quot;: [
      {
        &quot;e&quot;: &quot;AQAB&quot;,
        &quot;kid&quot;: &quot;endwNUZrNTJsX2NyQlp4bjhVcTFTTVltR2gxV2RV...&quot;,
        &quot;kty&quot;: &quot;RSA&quot;,
        &quot;n&quot;: &quot;vXdXzZwQo0hxRSmZEcDIsnpg-CMEkor50SOG-1XUlM...&quot;
      }
    ]
  },
  &quot;metadata&quot;: {
    &quot;federation_entity&quot;: {
      &quot;contacts&quot;: [&quot;ops@umu.se&quot;],
      &quot;federation_fetch_endpoint&quot;: &quot;https://umu.se/oidc/fedapi&quot;,
      &quot;organization_uri&quot;: &quot;https://www.umu.se&quot;,
      &quot;organization_name&quot;: &quot;UmU&quot;
    }
  }
}</code></pre>
</div>
<figcaption><a href="#figure-57" class="selfRef">Figure 57</a>: <a href="#name-entity-configuration-jwt-cl" class="selfRef">Entity Configuration JWT Claims Set</a></figcaption>
</figure>

The only piece of information that is used from this Entity Configuration in this process is the `federation_fetch_endpoint`, which is used in the next step.





#### Subordinate Statement Published by https://umu.se about https://op.umu.se

The RP uses the fetch endpoint provided by https://umu.se, as defined in [Section 8.1.1](#fetch_statement){.auto .internal .xref}, to fetch information about https://op.umu.se.

The request will look like this:

[]{#name-request-subordinate-stateme}

<figure id="figure-58">
<div id="appendix-A.2.3-3.1" class="alignLeft art-text artwork">
<pre><code>GET /oidc/fedapi?sub=https%3A%2F%2Fop.umu.se&amp;
iss=https%3A%2F%2Fumu.se HTTP/1.1
Host: umu.se</code></pre>
</div>
<figcaption><a href="#figure-58" class="selfRef">Figure 58</a>: <a href="#name-request-subordinate-stateme" class="selfRef">Request Subordinate Statement from https://umu.se about https://op.umu.se</a></figcaption>
</figure>

And the result is this:

[]{#name-subordinate-statement-issue}

<figure id="figure-59">
<div id="appendix-A.2.3-5.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;exp&quot;: 1568397247,
  &quot;iat&quot;: 1568310847,
  &quot;iss&quot;: &quot;https://umu.se&quot;,
  &quot;sub&quot;: &quot;https://op.umu.se&quot;,
  &quot;source_endpoint&quot;: &quot;https://umu.se/oidc/fedapi&quot;,
  &quot;jwks&quot;: {
    &quot;keys&quot;: [
      {
        &quot;e&quot;: &quot;AQAB&quot;,
        &quot;kid&quot;: &quot;dEEtRjlzY3djcENuT01wOGxrZlkxb3RIQVJlMTY0...&quot;,
        &quot;kty&quot;: &quot;RSA&quot;,
        &quot;n&quot;: &quot;x97YKqc9Cs-DNtFrQ7_vhXoH9bwkDWW6En2jJ044yH...&quot;
      }
    ]
  },
  &quot;metadata_policy&quot;: {
    &quot;openid_provider&quot;: {
      &quot;contacts&quot;: {
        &quot;add&quot;: [
          &quot;ops@swamid.se&quot;
        ]
      },
      &quot;organization_name&quot;: {
        &quot;value&quot;: &quot;University of Umeå&quot;
      },
      &quot;subject_types_supported&quot;: {
        &quot;value&quot;: [
          &quot;pairwise&quot;
        ]
      },
      &quot;token_endpoint_auth_methods_supported&quot;: {
        &quot;default&quot;: [
          &quot;private_key_jwt&quot;
        ],
        &quot;subset_of&quot;: [
          &quot;private_key_jwt&quot;,
          &quot;client_secret_jwt&quot;
        ],
        &quot;superset_of&quot;: [
          &quot;private_key_jwt&quot;
        ]
      }
    }
  }
}</code></pre>
</div>
<figcaption><a href="#figure-59" class="selfRef">Figure 59</a>: <a href="#name-subordinate-statement-issue" class="selfRef">Subordinate Statement Issued by https://umu.se about https://op.umu.se</a></figcaption>
</figure>

This Subordinate Statement is the second link in the Trust Chain.





#### Entity Configuration for https://swamid.se

The LIGO Wiki RP fetches the Entity Configuration from https://swamid.se using the process defined in [Section 9](#federation_configuration){.auto .internal .xref}.

The request will look like this:

[]{#name-request-entity-configuratio}

<figure id="figure-60">
<div id="appendix-A.2.4-3.1" class="alignLeft art-text artwork">
<pre><code>GET /.well-known/openid-federation HTTP/1.1
Host: swamid.se</code></pre>
</div>
<figcaption><a href="#figure-60" class="selfRef">Figure 60</a>: <a href="#name-request-entity-configuratio" class="selfRef">Request Entity Configuration from https://swamid.se</a></figcaption>
</figure>

And the GET will return:

[]{#name-entity-configuration-issued-b}

<figure id="figure-61">
<div id="appendix-A.2.4-5.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;authority_hints&quot;: [
    &quot;https://edugain.geant.org&quot;
  ],
  &quot;exp&quot;: 1568397247,
  &quot;iat&quot;: 1568310847,
  &quot;iss&quot;: &quot;https://swamid.se&quot;,
  &quot;sub&quot;: &quot;https://swamid.se&quot;,
  &quot;jwks&quot;: {
    &quot;keys&quot;: [
      {
        &quot;e&quot;: &quot;AQAB&quot;,
        &quot;kid&quot;: &quot;N1pQTzFxUXZ1RXVsUkVuMG5uMnVDSURGRVdhUzdO...&quot;,
        &quot;kty&quot;: &quot;RSA&quot;,
        &quot;n&quot;: &quot;3EQc6cR_GSBq9km9-WCHY_lWJZWkcn0M05TGtH6D9S...&quot;
      }
    ]
  },
  &quot;metadata&quot;: {
    &quot;federation_entity&quot;: {
      &quot;contacts&quot;: [&quot;ops@swamid.se&quot;],
      &quot;federation_fetch_endpoint&quot;:
        &quot;https://swamid.se/fedapi&quot;,
      &quot;organization_uri&quot;: &quot;https://www.sunet.se/swamid/&quot;,
      &quot;organization_name&quot;: &quot;SWAMID&quot;
    }
  }
}</code></pre>
</div>
<figcaption><a href="#figure-61" class="selfRef">Figure 61</a>: <a href="#name-entity-configuration-issued-b" class="selfRef">Entity Configuration Issued by https://swamid.se</a></figcaption>
</figure>

The only piece of information that is used from this Entity Configuration in this process is the `federation_fetch_endpoint`, which is used in the next step.





#### Subordinate Statement Published by https://swamid.se about https://umu.se

The LIGO Wiki RP uses the fetch endpoint provided by https://swamid.se as defined in [Section 8.1.1](#fetch_statement){.auto .internal .xref} to fetch information about https://umu.se.

The request will look like this:

[]{#name-request-to-https-swamidse-f}

<figure id="figure-62">
<div id="appendix-A.2.5-3.1" class="alignLeft art-text artwork">
<pre><code>GET /fedapi?sub=https%3A%2F%2Fumu.se&amp;
iss=https%3A%2F%2Fswamid.se HTTP/1.1
Host: swamid.se</code></pre>
</div>
<figcaption><a href="#figure-62" class="selfRef">Figure 62</a>: <a href="#name-request-to-https-swamidse-f" class="selfRef">Request to https://swamid.se for Subordinate Statement about https://umu.se</a></figcaption>
</figure>

And the result is this:

[]{#name-subordinate-statement-issued}

<figure id="figure-63">
<div id="appendix-A.2.5-5.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;exp&quot;: 1568397247,
  &quot;iat&quot;: 1568310847,
  &quot;iss&quot;: &quot;https://swamid.se&quot;,
  &quot;sub&quot;: &quot;https://umu.se&quot;,
  &quot;source_endpoint&quot;: &quot;https://swamid.se/fedapi&quot;,
  &quot;jwks&quot;: {
    &quot;keys&quot;: [
      {
        &quot;e&quot;: &quot;AQAB&quot;,
        &quot;kid&quot;: &quot;endwNUZrNTJsX2NyQlp4bjhVcTFTTVltR2gxV2RV...&quot;,
        &quot;kty&quot;: &quot;RSA&quot;,
        &quot;n&quot;: &quot;vXdXzZwQo0hxRSmZEcDIsnpg-CMEkor50SOG-1XUlM...&quot;
      }
    ]
  },
  &quot;metadata_policy&quot;: {
    &quot;openid_provider&quot;: {
      &quot;id_token_signing_alg_values_supported&quot;: {
        &quot;subset_of&quot;: [
          &quot;RS256&quot;,
          &quot;ES256&quot;,
          &quot;ES384&quot;,
          &quot;ES512&quot;
        ]
      },
      &quot;token_endpoint_auth_methods_supported&quot;: {
        &quot;subset_of&quot;: [
          &quot;client_secret_jwt&quot;,
          &quot;private_key_jwt&quot;
        ]
      },
      &quot;userinfo_signing_alg_values_supported&quot;: {
        &quot;subset_of&quot;: [
          &quot;ES256&quot;,
          &quot;ES384&quot;,
          &quot;ES512&quot;
        ]
      }
    }
  }
}</code></pre>
</div>
<figcaption><a href="#figure-63" class="selfRef">Figure 63</a>: <a href="#name-subordinate-statement-issued" class="selfRef">Subordinate Statement Issued by https://swamid.se about https://umu.se</a></figcaption>
</figure>

This Subordinate Statement is the third link in the Trust Chain.

If we assume that the issuer of this Subordinate Statement is not in the list of Trust Anchors the LIGO Wiki RP has access to, we have to go one step further.





#### Entity Configuration for https://edugain.geant.org

The RP fetches the Entity Configuration from https://edugain.geant.org using the process defined in [Section 9](#federation_configuration){.auto .internal .xref}.

The request will look like this:

[]{#name-entity-configuration-reques}

<figure id="figure-64">
<div id="appendix-A.2.6-3.1" class="alignLeft art-text artwork">
<pre><code>GET /.well-known/openid-federation HTTP/1.1
Host: edugain.geant.org</code></pre>
</div>
<figcaption><a href="#figure-64" class="selfRef">Figure 64</a>: <a href="#name-entity-configuration-reques" class="selfRef">Entity Configuration Requested from https://edugain.geant.org</a></figcaption>
</figure>

And the GET will return:

[]{#name-entity-configuration-issued-by}

<figure id="figure-65">
<div id="appendix-A.2.6-5.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;exp&quot;: 1568397247,
  &quot;iat&quot;: 1568310847,
  &quot;iss&quot;: &quot;https://edugain.geant.org&quot;,
  &quot;sub&quot;: &quot;https://edugain.geant.org&quot;,
  &quot;jwks&quot;: {
    &quot;keys&quot;: [
      {
        &quot;e&quot;: &quot;AQAB&quot;,
        &quot;kid&quot;: &quot;Sl9DcjFxR3hrRGdabUNIR21KT3dvdWMyc2VUM2Fr...&quot;,
        &quot;kty&quot;: &quot;RSA&quot;,
        &quot;n&quot;: &quot;xKlwocDXUw-mrvDSO4oRrTRrVuTwotoBFpozvlq-1q...&quot;
      }
    ]
  },
  &quot;metadata&quot;: {
    &quot;federation_entity&quot;: {
      &quot;federation_fetch_endpoint&quot;: &quot;https://geant.org/edugain/api&quot;
    }
  }
}</code></pre>
</div>
<figcaption><a href="#figure-65" class="selfRef">Figure 65</a>: <a href="#name-entity-configuration-issued-by" class="selfRef">Entity Configuration issued by https://edugain.geant.org</a></figcaption>
</figure>

Within the Trust Anchor Entity Configuration, the Relying Party looks for the `federation_fetch_endpoint` and gets the updated Federation Entity Keys of the Trust Anchor. Each Entity within a Federation may change their Federation Entity Keys, or any other attributes, at any time. See [Section 11.2](#key_rollover_anchor){.auto .internal .xref} for further details.





#### Subordinate Statement Published by https://edugain.geant.org about https://swamid.se

The LIGO Wiki RP uses the fetch endpoint of https://edugain.geant.org as defined in [Section 8.1.1](#fetch_statement){.auto .internal .xref} to fetch information about "https://swamid.se".

The request will look like this:

[]{#name-request-to-https-edugaingea}

<figure id="figure-66">
<div id="appendix-A.2.7-3.1" class="alignLeft art-text artwork">
<pre><code>GET /edugain/api?sub=https%3A%2F%2Fswamid.se&amp;
iss=https%3A%2F%2Fedugain.geant.org HTTP/1.1
Host: geant.org</code></pre>
</div>
<figcaption><a href="#figure-66" class="selfRef">Figure 66</a>: <a href="#name-request-to-https-edugaingea" class="selfRef">Request to https://edugain.geant.org for Subordinate Statement about https://swamid.se</a></figcaption>
</figure>

And the result is this:

[]{#name-subordinate-statement-issued-}

<figure id="figure-67">
<div id="appendix-A.2.7-5.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;exp&quot;: 1568397247,
  &quot;iat&quot;: 1568310847,
  &quot;iss&quot;: &quot;https://edugain.geant.org&quot;,
  &quot;sub&quot;: &quot;https://swamid.se&quot;,
  &quot;source_endpoint&quot;: &quot;https://edugain.geant.org/edugain/api&quot;,
  &quot;jwks&quot;: {
    &quot;keys&quot;: [
      {
        &quot;e&quot;: &quot;AQAB&quot;,
        &quot;kid&quot;: &quot;N1pQTzFxUXZ1RXVsUkVuMG5uMnVDSURGRVdhUzdO...&quot;,
        &quot;kty&quot;: &quot;RSA&quot;,
        &quot;n&quot;: &quot;3EQc6cR_GSBq9km9-WCHY_lWJZWkcn0M05TGtH6D9S...&quot;
      }
    ]
  },
  &quot;metadata_policy&quot;: {
    &quot;openid_provider&quot;: {
      &quot;contacts&quot;: {
        &quot;add&quot;: [&quot;ops@edugain.geant.org&quot;]
      }
    },
    &quot;openid_relying_party&quot;: {
      &quot;contacts&quot;: {
        &quot;add&quot;: [&quot;ops@edugain.geant.org&quot;]
      }
    }
  }
}</code></pre>
</div>
<figcaption><a href="#figure-67" class="selfRef">Figure 67</a>: <a href="#name-subordinate-statement-issued-" class="selfRef">Subordinate Statement issued by https://edugain.geant.org about https://swamid.se</a></figcaption>
</figure>

If we assume that the issuer of this statement appears in the list of Trust Anchors the LIGO Wiki RP has access to, this Subordinate Statement would be the fourth link in the Trust Chain. The Trust Anchor's Entity Configuration MAY also be included in the Trust Chain; in this case, it would be the fifth and final link.

We have now retrieved all the members of the Trust Chain. Recapping, these Entity Statements were obtained:

- 
  Entity Configuration for the Leaf Entity https://op.umu.se - the first link in the Trust Chain
  

- 
  Entity Configuration for https://umu.se - not included in the Trust Chain
  

- 
  Subordinate Statement issued by https://umu.se about https://op.umu.se - the second link in the Trust Chain
  

- 
  Entity Configuration for https://swamid.se - not included in the Trust Chain
  

- 
  Subordinate Statement issued by https://swamid.se about https://umu.se - the third link in the Trust Chain
  

- 
  Entity Configuration for https://edugain.geant.org - optionally, the fifth and last link in the Trust Chain
  

- 
  Subordinate Statement issued by https://edugain.geant.org about https://swamid.se - the fourth link in the Trust Chain
  

Using the public keys of the Trust Anchor that the LIGO Wiki RP has been provided within some secure out-of-band way, it can now verify the Trust Chain as described in [Section 10.2](#trust_chain_validation){.auto .internal .xref}.





#### Verified Metadata for https://op.umu.se

Having verified the chain, the LIGO Wiki RP can proceed with the next step.

Combining the metadata policies from the three Subordinate Statements we have by Immediate Superiors about their Immediate Subordinates and applying the combined policy to the metadata statement that the Leaf Entity presented, we get:

[]{#name-resolved-metadata-derived-f}

<figure id="figure-68">
<div id="appendix-A.2.8-3.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;authorization_endpoint&quot;:
    &quot;https://op.umu.se/openid/authorization&quot;,
  &quot;contacts&quot;: [
    &quot;ops@swamid.se&quot;,
    &quot;ops@edugain.geant.org&quot;
  ],
  &quot;federation_registration_endpoint&quot;:
    &quot;https://op.umu.se/openid/fedreg&quot;,
  &quot;client_registration_types_supported&quot;: [
    &quot;automatic&quot;,
    &quot;explicit&quot;
  ],
  &quot;grant_types_supported&quot;: [
    &quot;authorization_code&quot;,
    &quot;implicit&quot;,
    &quot;urn:ietf:params:oauth:grant-type:jwt-bearer&quot;
  ],
  &quot;id_token_signing_alg_values_supported&quot;: [
    &quot;RS256&quot;,
    &quot;ES256&quot;
  ],
  &quot;issuer&quot;: &quot;https://op.umu.se/openid&quot;,
  &quot;signed_jwks_uri&quot;: &quot;https://op.umu.se/openid/jwks.jose&quot;,
  &quot;logo_uri&quot;:
    &quot;https://www.umu.se/img/umu-logo-left-neg-SE.svg&quot;,
  &quot;organization_name&quot;: &quot;University of Umeå&quot;,
  &quot;op_policy_uri&quot;:
    &quot;https://www.umu.se/en/website/legal-information/&quot;,
  &quot;request_parameter_supported&quot;: true,
  &quot;response_types_supported&quot;: [
    &quot;code&quot;,
    &quot;code id_token&quot;,
    &quot;token&quot;
  ],
  &quot;subject_types_supported&quot;: [
    &quot;pairwise&quot;
  ],
  &quot;token_endpoint&quot;: &quot;https://op.umu.se/openid/token&quot;,
  &quot;token_endpoint_auth_methods_supported&quot;: [
    &quot;private_key_jwt&quot;,
    &quot;client_secret_jwt&quot;
  ]
}
</code></pre>
</div>
<figcaption><a href="#figure-68" class="selfRef">Figure 68</a>: <a href="#name-resolved-metadata-derived-f" class="selfRef">Resolved Metadata Derived from Trust Chain by Applying Metadata Policies</a></figcaption>
</figure>

We have now reached the end of the Provider Discovery process.







### Examples of the Two Ways of Doing Client Registration

As described in [Section 12](#client_registration){.auto .internal .xref}, there are two ways that can be used to do client registration:

[]{.break}

Automatic
:   No negotiation between the RP and the OP is made regarding what features the client SHOULD use in future communication occurs. The RP's published metadata filtered by the chosen Trust Chain's metadata policies defines the metadata that is to be used.
:   

Explicit
:   The RP will access the `federation_registration_endpoint`, which provides the RP's metadata. The OP MAY return a metadata policy that adds restrictions over and above what the Trust Chain already has defined.
:   



#### RP Sends Authentication Request (Automatic Client Registration)

The LIGO Wiki RP does not do any registration but goes directly to sending an Authentication Request.

Here is an example of such an Authentication Request:

[]{#name-authentication-request-using}

<figure id="figure-69">
<div id="appendix-A.3.1-3.1" class="alignLeft art-text artwork">
<pre><code>GET /openid/authorization?
  request=eyJ0eXAiOiJvYXV0aC1hdXRoei1yZXErand0IiwiYWxnIjoiU
    lMyNTYiLCJraWQiOiJkVU4yYTAxd1JraGtTV3BsUVRodmNWQklOVUl3
    VFVkT1VGVTJUbVZyU21oRVFYZ3paWGxwVHpkUU5BIn0.
    eyJyZXNwb25zZV90eXBlIjogImNvZGUiLCAic2NvcGUiOiAib3Blbml
    kIHByb2ZpbGUgZW1haWwiLCAiY2xpZW50X2lkIjogImh0dHBzOi8vd2
    lraS5saWdvLm9yZyIsICJzdGF0ZSI6ICIyZmY3ZTU4OS0zODQ4LTQ2Z
    GEtYTNkMi05NDllMTIzNWU2NzEiLCAibm9uY2UiOiAiZjU4MWExODYt
    YWNhNC00NmIzLTk0ZmMtODA0ODQwODNlYjJjIiwgInJlZGlyZWN0X3V
    yaSI6ICJodHRwczovL3dpa2kubGlnby5vcmcvb3BlbmlkL2NhbGxiYW
    NrIiwgImlzcyI6ICJodHRwczovL3dpa2kubGlnby5vcmciLCAiaWF0I
    jogMTU5MzU4ODA4NSwgImF1ZCI6ICJodHRwczovL29wLnVtdS5zZSJ9
    .cRwSFNcDx6VsacAQDcIx
    5OAt_Pj30I_uUKRh04N4QJd6MZ0f50sETRv8uspSt9fMa-5yV3uzthX
    _v8OtQrV33gW1vzgOSRCdHgeCN40StbzjFk102seDwtU_Uzrcsy7KrX
    YSBp8U0dBDjuxC6h18L8ExjeR-NFjcrhy0wwua7Tnb4QqtN0QCia6DD
    8QBNVTL1Ga0YPmMdT25wS26wug23IgpbZB20VUosmMGgGtS5yCI5AwK
    Bhozv-oBH5KxxHzH1Oss-RkIGiQnjRnaWwEOTITmfZWra1eHP254wFF
    2se-EnWtz1q2XwsD9NSsOEJwWJPirPPJaKso8ng6qrrOSgw
  &amp;response_type=code
  &amp;client_id=https%3A%2F%2Fwiki.ligo.org
  &amp;redirect_uri=https%3A%2F%2Fwiki.ligo.org/openid/callback
  &amp;scope=openid+profile+email
  HTTP/1.1
Host: op.umu.se</code></pre>
</div>
<figcaption><a href="#figure-69" class="selfRef">Figure 69</a>: <a href="#name-authentication-request-using" class="selfRef">Authentication Request Using Automatic Client Registration</a></figcaption>
</figure>

The OP receiving this Authentication Request will, unless the RP is already registered, start to dynamically fetch and establish trust with the RP.



##### OP Fetches Entity Statements

The OP needs to establish a Trust Chain for the RP (wiki.ligo.org). The OP in this example is configured with public keys of two federations:

- 
  https://edugain.geant.org
  

- 
  https://swamid.se
  

The OP starts to resolve metadata for the Client Identifier https://wiki.ligo.org by fetching the Entity Configuration using the process described in [Section 9](#federation_configuration){.auto .internal .xref}.

The process is the same as described in [Appendix A.2](#op_discovery){.auto .internal .xref} and will result in a Trust Chain with the following Entity Statements:

1.  
    Entity Configuration for the Leaf Entity https://wiki.ligo.org
    

2.  
    Subordinate Statement issued by https://incommon.org about https://wiki.ligo.org
    

3.  
    Subordinate Statement issued by https://edugain.geant.org about https://incommon.org
    





##### OP Evaluates the RP Metadata

Using the public keys of the Trust Anchor that the LIGO Wiki RP has been provided within some secure out-of-band way, it can now verify the Trust Chain as described in [Section 10.2](#trust_chain_validation){.auto .internal .xref}.

We will not list the complete Entity Statements but only the `metadata` and `metadata_policy` parts. There are two metadata policies:

[]{.break}

edugain.geant.org:
:   []{#name-metadata-policies-related-t}
    <figure id="figure-70">
    <div id="appendix-A.3.1.2-3.2.1.1" class="alignLeft art-text artwork">
    <pre><code>&quot;metadata_policy&quot;: {
      &quot;openid_provider&quot;: {
        &quot;contacts&quot;: {
          &quot;add&quot;: [&quot;ops@edugain.geant.org&quot;]
        }
      },
      &quot;openid_relying_party&quot;: {
        &quot;contacts&quot;: {
          &quot;add&quot;: [&quot;ops@edugain.geant.org&quot;]
        }
      }
    }</code></pre>
    </div>
    <figcaption><a href="#figure-70" class="selfRef">Figure 70</a>: <a href="#name-metadata-policies-related-t" class="selfRef">Metadata Policies Related to Multiple Metadata Types</a></figcaption>
    </figure>

    
:   

[]{.break}

incommon.org:
:   []{#name-metadata-policy-related-to-}
    <figure id="figure-71">
    <div id="appendix-A.3.1.2-4.2.1.1" class="alignLeft art-text artwork">
    <pre><code>&quot;metadata_policy&quot;: {
      &quot;openid_relying_party&quot;: {
        &quot;application_type&quot;: {
          &quot;one_of&quot;: [
            &quot;web&quot;,
            &quot;native&quot;
          ]
        },
        &quot;contacts&quot;: {
          &quot;add&quot;: [&quot;ops@incommon.org&quot;]
        },
        &quot;grant_types&quot;: {
          &quot;subset_of&quot;: [
            &quot;authorization_code&quot;,
            &quot;refresh_token&quot;
          ]
        }
      }
    }</code></pre>
    </div>
    <figcaption><a href="#figure-71" class="selfRef">Figure 71</a>: <a href="#name-metadata-policy-related-to-" class="selfRef">Metadata Policy Related to the RP</a></figcaption>
    </figure>

    
:   

Next, combine these and apply them to the metadata for wiki.ligo.org:

[]{#name-combined-metadata-with-meta}

<figure id="figure-72">
<div id="appendix-A.3.1.2-6.1" class="alignLeft art-text artwork">
<pre><code>&quot;metadata&quot;: {
  &quot;openid_relying_party&quot;: {
    &quot;application_type&quot;: &quot;web&quot;,
    &quot;client_name&quot;: &quot;LIGO Wiki&quot;,
    &quot;contacts&quot;: [
      &quot;ops@ligo.org&quot;
    ],
    &quot;grant_types&quot;: [
      &quot;authorization_code&quot;,
      &quot;refresh_token&quot;
    ],
    &quot;id_token_signing_alg_values_supported&quot;:
      [&quot;ES256&quot;, &quot;PS256&quot;, &quot;RS256&quot;],
    &quot;signed_jwks_uri&quot;: &quot;https://wiki.ligo.org/jwks.jose&quot;,
    &quot;redirect_uris&quot;: [
      &quot;https://wiki.ligo.org/openid/callback&quot;
    ],
    &quot;response_types&quot;: [
      &quot;code&quot;
    ],
    &quot;subject_type&quot;: &quot;public&quot;,
    &quot;token_endpoint_auth_method&quot;: &quot;private_key_jwt&quot;
  }
}</code></pre>
</div>
<figcaption><a href="#figure-72" class="selfRef">Figure 72</a>: <a href="#name-combined-metadata-with-meta" class="selfRef">Combined Metadata with Metadata Policy yet to be applied</a></figcaption>
</figure>

The final result is:

[]{#name-resolved-metadata-after-met}

<figure id="figure-73">
<div id="appendix-A.3.1.2-8.1" class="alignLeft art-text artwork">
<pre><code>&quot;metadata&quot;: {
  &quot;openid_relying_party&quot;: {
    &quot;application_type&quot;: &quot;web&quot;,
    &quot;client_name&quot;: &quot;LIGO Wiki&quot;,
    &quot;contacts&quot;: [
      &quot;ops@ligo.org&quot;,
      &quot;ops@edugain.geant.org&quot;,
      &quot;ops@incommon.org&quot;
    ],
    &quot;grant_types&quot;: [
      &quot;refresh_token&quot;,
      &quot;authorization_code&quot;
    ],
    &quot;id_token_signing_alg_values_supported&quot;:
      [&quot;ES256&quot;, &quot;PS256&quot;, &quot;RS256&quot;],
    &quot;signed_jwks_uri&quot;: &quot;https://wiki.ligo.org/jwks.jose&quot;,
    &quot;redirect_uris&quot;: [
      &quot;https://wiki.ligo.org/openid/callback&quot;
    ],
    &quot;response_types&quot;: [
      &quot;code&quot;
    ],
    &quot;subject_type&quot;: &quot;public&quot;,
    &quot;token_endpoint_auth_method&quot;: &quot;private_key_jwt&quot;
  }
}</code></pre>
</div>
<figcaption><a href="#figure-73" class="selfRef">Figure 73</a>: <a href="#name-resolved-metadata-after-met" class="selfRef">Resolved Metadata After Metadata Policy has been Applied</a></figcaption>
</figure>

Once the Trust Chain and the final Relying Party metadata have been obtained, the OpenID Provider has everything needed to validate the signature of the Request Object in the Authentication Request, using the public keys made available at the `signed_jwks_uri` endpoint.







#### RP Starts with Client Registration (Explicit Client Registration)

Here the LIGO Wiki RP sends an Explicit Registration request to the `federation_registration_endpoint` of the OP (op.umu.se). The request contains the RP's Entity Configuration.

An example JWT Claims Set for the RP's Entity Configuration is:

[]{#name-rps-entity-configuration-jw}

<figure id="figure-74">
<div id="appendix-A.3.2-3.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;iss&quot;: &quot;https://wiki.ligo.org&quot;,
  &quot;sub&quot;: &quot;https://wiki.ligo.org&quot;,
  &quot;iat&quot;: 1676045527,
  &quot;exp&quot;: 1676063610,
  &quot;aud&quot;: &quot;https://op.umu.se&quot;,
  &quot;metadata&quot;: {
    &quot;openid_relying_party&quot;: {
      &quot;application_type&quot;: &quot;web&quot;,
      &quot;client_name&quot;: &quot;LIGO Wiki&quot;,
      &quot;contacts&quot;: [&quot;ops@ligo.org&quot;],
      &quot;grant_types&quot;: [&quot;authorization_code&quot;],
      &quot;id_token_signed_response_alg&quot;: &quot;RS256&quot;,
      &quot;signed_jwks_uri&quot;: &quot;https://wiki.ligo.org/jwks.jose&quot;,
      &quot;redirect_uris&quot;: [
        &quot;https://wiki.ligo.org/openid/callback&quot;
      ],
      &quot;response_types&quot;: [&quot;code&quot;],
      &quot;subject_type&quot;: &quot;public&quot;
    }
  },
  &quot;jwks&quot;: {
    &quot;keys&quot;: [
      {
        &quot;kty&quot;: &quot;RSA&quot;,
        &quot;use&quot;: &quot;sig&quot;,
        &quot;kid&quot;:
          &quot;U2JTWHY0VFg0a2FEVVdTaHptVDJsNDNiSDk5MXRBVEtNSFVkeXZwb&quot;,
        &quot;e&quot;: &quot;AQAB&quot;,
        &quot;n&quot;:
          &quot;4AZjgqFwMhTVSLrpzzNcwaCyVD88C_Hb3Bmor97vH-2AzldhuVb8K...&quot;
      }
    ]
  },
  &quot;authority_hints&quot;: [&quot;https://incommon.org&quot;]
}</code></pre>
</div>
<figcaption><a href="#figure-74" class="selfRef">Figure 74</a>: <a href="#name-rps-entity-configuration-jw" class="selfRef">RP's Entity Configuration JWT Claims Set</a></figcaption>
</figure>

The OP receives the RP's Entity Configuration and proceeds with the sequence of steps laid out in [Appendix A.2](#op_discovery){.auto .internal .xref}.

The OP successfully resolves the same RP metadata described in [Appendix A.3.1.2](#rp_metadata_eval){.auto .internal .xref}. It then registers the RP in compliance with its own OP metadata and returns the result in a registration Entity Statement.

Assuming the OP does not support refresh tokens it will register the RP for the `authorization_code` grant type only. This is reflected in the metadata returned to the RP.

The returned metadata also includes the `client_id`, the `client_secret` and other parameters that the OP provisioned for the RP.

Here is an example JWT Claims Set of the registration Entity Statement returned by the OP to the RP after successful explicit client registration:

[]{#name-jwt-claims-set-of-registrat}

<figure id="figure-75">
<div id="appendix-A.3.2-9.1" class="alignLeft art-text artwork">
<pre><code>{
  &quot;iss&quot;: &quot;https://op.umu.se&quot;,
  &quot;sub&quot;: &quot;https://wiki.ligo.org&quot;,
  &quot;aud&quot;: &quot;https://wiki.ligo.org&quot;,
  &quot;iat&quot;: 1601457619,
  &quot;exp&quot;: 1601544019,
  &quot;trust_anchor&quot;: &quot;https://edugain.geant.org&quot;,
  &quot;metadata&quot;: {
    &quot;openid_relying_party&quot;: {
      &quot;client_id&quot;: &quot;m3GyHw&quot;,
      &quot;client_secret_expires_at&quot;: 1604049619,
      &quot;client_secret&quot;:
        &quot;cb44eed577f3b5edf3e08362d47a0dc44630b3dc6ea99f7a79205&quot;,
      &quot;client_id_issued_at&quot;: 1601457619,
      &quot;application_type&quot;: &quot;web&quot;,
      &quot;client_name&quot;: &quot;LIGO Wiki&quot;,
      &quot;contacts&quot;: [
        &quot;ops@edugain.geant.org&quot;,
        &quot;ops@incommon.org&quot;,
        &quot;ops@ligo.org&quot;
      ],
      &quot;grant_types&quot;: [
        &quot;authorization_code&quot;
      ],
      &quot;id_token_signed_response_alg&quot;: &quot;RS256&quot;,
      &quot;signed_jwks_uri&quot;: &quot;https://wiki.ligo.org/jwks.jose&quot;,
      &quot;redirect_uris&quot;: [
        &quot;https://wiki.ligo.org/openid/callback&quot;
      ],
      &quot;response_types&quot;: [
        &quot;code&quot;
      ],
      &quot;subject_type&quot;: &quot;public&quot;
    }
  },
  &quot;authority_hints&quot;: [
    &quot;https://incommon.org&quot;
  ],
  &quot;jwks&quot;: {
    &quot;keys&quot;: [
      {
        &quot;kty&quot;: &quot;RSA&quot;,
        &quot;use&quot;: &quot;sig&quot;,
        &quot;kid&quot;:
          &quot;U2JTWHY0VFg0a2FEVVdTaHptVDJsNDNiSDk5MXRBVEtNSFVkeXZwb&quot;,
        &quot;e&quot;: &quot;AQAB&quot;,
        &quot;n&quot;:
          &quot;4AZjgqFwMhTVSLrpzzNcwaCyVD88C_Hb3Bmor97vH-2AzldhuVb8K...&quot;
      },
      {
        &quot;kty&quot;: &quot;EC&quot;,
        &quot;use&quot;: &quot;sig&quot;,
        &quot;kid&quot;: &quot;LWtFcklLOGdrW&quot;,
        &quot;crv&quot;: &quot;P-256&quot;,
        &quot;x&quot;: &quot;X2S1dFE7zokQDST0bfHdlOWxOc8FC1l4_sG1Kwa4l4s&quot;,
        &quot;y&quot;: &quot;812nU6OCKxgc2ZgSPt_dkXbYldG_smHJi4wXByDHc6g&quot;
      }
    ]
  }
}</code></pre>
</div>
<figcaption><a href="#figure-75" class="selfRef">Figure 75</a>: <a href="#name-jwt-claims-set-of-registrat" class="selfRef">JWT Claims Set of Registration Entity Statement Returned by OP to RP after Explicit Client Registration</a></figcaption>
</figure>









## Notices

Copyright (c) 2025 The OpenID Foundation.

The OpenID Foundation (OIDF) grants to any Contributor, developer, implementer, or other interested party a non-exclusive, royalty free, worldwide copyright license to reproduce, prepare derivative works from, distribute, perform and display, this Implementers Draft, Final Specification, or Final Specification Incorporating Errata Corrections solely for the purposes of (i) developing specifications, and (ii) implementing Implementers Drafts, Final Specifications, and Final Specification Incorporating Errata Corrections based on such documents, provided that attribution be made to the OIDF as the source of the material, but that such attribution does not indicate an endorsement by the OIDF.

The technology described in this specification was made available from contributions from various sources, including members of the OpenID Foundation and others. Although the OpenID Foundation has taken steps to help ensure that the technology is available for distribution, it takes no position regarding the validity or scope of any intellectual property or other rights that might be claimed to pertain to the implementation or use of the technology described in this specification or the extent to which any license under such rights might or might not be available; neither does it represent that it has made any independent effort to identify any such rights. The OpenID Foundation and the contributors to this specification make no (and hereby expressly disclaim any) warranties (express, implied, or otherwise), including implied warranties of merchantability, non-infringement, fitness for a particular purpose, or title, related to this specification, and the entire risk as to implementing this specification is assumed by the implementer. The OpenID Intellectual Property Rights policy (found at openid.net) requires contributors to offer a patent promise not to assert certain patent claims against other contributors and against implementers. OpenID invites any interested party to bring to its attention any copyrights, patents, patent applications, or other proprietary rights that may cover technology that may be required to practice this specification.





## Document History

[[ To be removed from the final specification ]]

-45

- 
  Fixed #233: Gave examples of the use of Automatic Registration at OAuth 2.0 endpoints other than the Authorization Endpoint and Token Endpoint.
  

- 
  Fixed #216: Added two more implementation considerations.
  

- 
  Fixed #237: The order of the elements resulting from merging two sets is not defined.
  

- 
  Fixed #241: Restructured Entity Statement section.
  

- 
  Fixed #84: Added section on validating Entity Statements.
  

- 
  Fixed #243: Be explicit about what to do when server validation fails in automatic registration.
  

- 
  Added Discovery and Trust Chain Resolution Patterns considerations
  

-44

- 
  Fixed #127: Explained Trust Mark Issuer validation in more detail.
  

- 
  Corrected location of Constraints in Trust Chain Example figure.
  

- 
  Fixed #147: Added a note about client authentication methods and Automatic Registration.
  

- 
  Applied must-have feature requests for Swedish government use cases, specifically:

  - 
    Make it optional to publish an Entity Configuration at the Entity Identifier's `/.well-known/openid-federation` URL (which is already true when using Explicit Registration).
    

  - 
    Allow non-https Entity Identifiers (while leaving defining how to retrieve Entity Configurations for them to future extensions).
    

  - 
    Make use of `client_registration_types` and `client_registration_types_supported` RECOMMENDED rather than REQUIRED.
    

  - 
    Remove recommendation that informational metadata values be in the Entity's `federation_entity` metadata.
    
  

- 
  Applied strong requests for Swedish government use cases, specifically:

  - 
    Fixes #244: Sign Trust Mark Status responses and extend the set of defined status values.
    

  - 
    Added warning about using JWKS representations where they may not be understood.
    
  

-43

- 
  Fixed #194: Renamed `trust_mark_id` to `trust_mark_type`.
  

- 
  Fixed #24, #25, #212: Simplified Trust Mark Status endpoint by having it take the Trust Mark to be validated as a parameter.
  

- 
  Fixed #35: Removed requirement for the `value` and `default` operators to support JSON object values, since merging them requires comparison. Added note about metadata policy and comparing JSON objects.
  

- 
  Fixed #167: Added Privacy Considerations section.
  

- 
  Fixed #196: The output of rows 3 and 4 in Table 1: Examples of Outputs with Combinations of essential and subset_of for Different Inputs must be empty JSON array [].
  

- 
  OAuth 2.0 Protected Resource Metadata is now RFC 9728.
  

- 
  Follow Implementation Considerations in [[OpenID.RP.Choices](#OpenID.RP.Choices){.cite .xref}].
  

- 
  Added note about using different signing algorithms for different Entity Statements in a Trust Chain.
  

- 
  Fixed #172: Added Resolved Metadata as defined term.
  

- 
  Fixed #166: Recommend way to validate non-expiring Trust Marks.
  

- 
  Fixed #193: Clarified that "metadata" declarations declare the roles that the Entity plays - its Entity Types.
  

- 
  Fixed #202: Added String Operations section.
  

- 
  Fixed #173: Use Resolved Metadata term in Explicit Registration.
  

- 
  Added informational metadata parameters `display_name`, `description`, `keywords`, and `information_uri` and added IANA registrations for them.
  

- 
  Added protected resource metadata IANA registrations.
  

- 
  Renamed `homepage_uri` to `organization_uri`.
  

-42

- 
  Addresses #11, #180: Allows the following unconditional operator combinations: add + superset_of. Makes the following previously conditional operator combinations unconditional: default + one_of, default + subset_of, default + superset_of. Makes the following previously unconditional operator combination conditional: value + essential. Allows the following conditional operator combinations: value + add, value + default, value + one_of, value + subset_of, value + superset_of.
  

- 
  Addresses #182: When applying the subset_of operator on a metadata parameter, if the resulting intersection is empty, then the metadata is made empty. Previously it was removed, which may lead to policy override for metadata parameters that have a default value, for instance grant_types RP metadata or grant_types_supported OP metadata. The merge of two subset_of operators is changed to allow empty intersection as well.
  

- 
  Addresses #129: Clarifies that the combination rules for a metadata policy operator apply to both individual as well as merged metadata parameter policies.
  

- 
  Fixed #184: Clarified that Request Objects can be passed by value or by reference.
  

- 
  Fixed #178: Clarify that other client methods (than automatic and explicit) are allowed.
  

- 
  Fixed #181: Contributed metadata policies must be logically sound and consistent with one another.
  

- 
  Fixed #130: Allow multiple Trust Anchor values to be passed in resolve requests.
  

- 
  Require `trust_chain` claim in resolve response.
  

- 
  Fixed #161: Prohibit loops in Trust Chains.
  

- 
  Fixed #47: Described using and not using a provided Trust Chain during Automatic Registration.
  

- 
  Fixed #85: Clarified Trust Chain selection during Explicit Registration. Also corrected the `authority_hints` value in the Explicit Registration response to be the Immediate Superior of the RP in the Trust Chain selected for it by the OP.
  

- 
  Fixed #35: Clarified that using non-interoperable JSON, as per Sections 4 and 8 of RFC 8259, can result in unpredictable metadata and metadata policy behavior.
  

- 
  Fixed #162: Trust Mark claim `id` renamed to `trust_mark_id`. Other more specific Trust Mark JWT `typ` header parameter values can be used if defined by trust frameworks in use and understood by the implementation.
  

-41

- 
  Fixed #131: Changed `anchor` request parameter to `trust_anchor`, changed `trust_anchor_id` claim to `trust_anchor`, and changed `type` request parameter to `entity_type`.
  

- 
  Explicitly typed base64url-encoded examples that were previously untyped. Also added missing `client_id` and `iss` values in some examples.
  

- 
  Fixed #7, #86, #134, and #148: Provides implementation considerations on Federation topologies.
  

- 
  Fixed #136: Defined additional error codes and rationalized naming. Renamed `trust_chain_validation_failed` to `invalid_trust_chain` and renamed `missing_trust_anchor` to `invalid_trust_anchor`.
  

- 
  Fixed #133: Refined wording about client authentication when using Automatic Registration and added `token_endpoint_auth_methods_supported` in RP metadata example.
  

- 
  Reference OpenID Connect Relying Party Metadata Choices 1.0.
  

- 
  Fixed #143: Added Trust Mark Issuer and Trust Mark Owner to Terminology section.
  

- 
  Fixed #139: Clarified description of using request objects.
  

- 
  Fixed #140: Federation Entity Keys MUST NOT appear in metadata.
  

- 
  Fixed #105 and #106: Informatively say that the `require_signed_request_object` and `require_pushed_authorization_requests` metadata parameters can be used.
  

- 
  Fixed #107: Clarified how to validate Trust Marks.
  

- 
  Fixed #114: Described why it may make sense to not support the use of `request_uri` other than in conjunction with a PAR request.
  

- 
  Fixed #108: Removed remark about trust mark delegation revocation.
  

- 
  Fixed #120: Required `kid` (Key ID) header parameter in Signed JWK Set JWTs.
  

- 
  Define media type for Explicit Registration responses `application/explicit-registration-response+jwt` distinct from `application/entity-statement+jwt`.
  

- 
  Restrict audience values to the single Entity Identifier of the intended recipient.
  

-40

- 
  Renamed `validation_failed` error code to `trust_chain_validation_failed`. Fixed #89: Improved Entity Statement `jwks` claim description.
  

- 
  Fixed #88: Explicitly require audience validation for explicit registration requests and responses.
  

- 
  Fixed #28: Described validation of resolved metadata.
  

- 
  Fixed #98: Require that the audience of JWTs used for client authentication at federation endpoints with `private_key_jwt` be the Entity Identifier of the endpoint's Entity.
  

- 
  Fixed #34: Deleted `request_authentication_methods_supported` and `request_authentication_signing_alg_values_supported` and replaced with the use of standard Request Object and PAR metadata values. Also restricted PAR authentication methods to those performing signing with the RP's keys.
  

- 
  Fixed #98: Required audience when using `private_key_jwt` with PAR to be the Authorization Server's Entity Identifier.
  

- 
  Reverted change to allow PAR requests not using Request Objects when the client authentication method uses a signature with a Federation Entity Key.
  

- 
  Fixed #104: Removed the `*_auth_signing_algs` metadata parameters in favor of `endpoint_auth_signing_alg_values_supported`.
  

- 
  Required that the `issuer` OP and AS metadata values match the Entity Identifier.
  

- 
  Fixed #69: Specified more details of successful and error responses to authentication requests using Automatic Registration.
  

- 
  Fixed #24: Removed `trust_mark` and `iat` from Trust Mark Status endpoint. Use `GET` at Trust Mark Status endpoint.
  

-39

- 
  Fixed #33: Corrected "add" operator values in examples to be arrays.
  

- 
  Endpoint URLs are not form-urlencoded in JSON metadata parameter values.
  

- 
  Fixed #52: Clarified that PAR requests must use Request Objects.
  

- 
  Fixed #64: Explicitly typed signed JWK Sets.
  

- 
  Fixed #55: Required validating explicit typing of JWTs.
  

- 
  Fixed #53: State that JWS and JWE Compact Serializations are used.
  

- 
  Fixed #49: Added request_authentication_signing_alg_values_supported to example.
  

- 
  Fixed #46: Corrected statement about preventing use of Request Object for `private_key_jwt` client authentication.
  

- 
  Fixed #43: Allow multiple `type` request parameters in resolve requests.
  

- 
  Fixed #40: Changed section name from "Resolve Entity Statement" to "Resolve Entity".
  

- 
  Improved descriptions of the JWT Claims being registered, per feedback from the IANA Designated Expert.
  

- 
  Fixes #45: Tightened Trust Chain signature validation wording.
  

- 
  Fixed #40: Changed section name from "Resolve Entity Statement" to "Resolve Entity".
  

- 
  Fixed #39: Removed `iss` parameter from fetch endpoint.
  

- 
  Fixed #54: MAY NOT -> MUST NOT.
  

- 
  Fixed #58: Require `authority_hints` value to contain the Entity Identifiers of all Immediate Superiors.
  

-38

- 
  Added section defining each media type, per IANA Designed Expert review.
  

- 
  Fixed #36: Simplified obtaining Entity Configurations.
  

- 
  Fixed #30: Simplified fetch endpoint to only return Subordinate Statements.
  

- 
  Removed text describing endpoints as being "encoded in `application/x-www-form-urlencoded` format".
  

-37

- 
  Note that after the approval and publication of the fourth Implementer's Draft https://openid.net/specs/openid-federation-1_0-ID4.html, work on the OpenID Federation specification moved from the repository https://bitbucket.org/openid/connect/ to the repository https://github.com/openid/federation. Issue numbers before this draft are from the Bitbucket repository. Issue numbers starting with this draft are from the GitHub repository.
  

- 
  Fixed #15: Clarified that OpenID Federation can be used for trust establishment with any application protocols.
  

- 
  Fixed #18: Clarified use of valid Trust Marks.
  

- 
  Fixed #19: Clarified that Federation Entities publish their public keys.
  

- 
  Fixed #22: Defined that Entity Identifiers MUST use the `https` scheme.
  

- 
  Clarified that additional `client_registration_types` MAY be defined and used.
  

- 
  Corrected Usage Location values in IANA "OAuth Extensions Error Registry" registrations.
  

-36

- 
  Made the definition of `iat` and `exp` consistent.
  

-35

- 
  Addresses #2124: The constraints claim must only appear in Subordinate Statements.
  

- 
  Fixes #2126: The max_path_length definition must be expressed in terms of Intermediate Entities, not Entity Statements, to prevent ambiguity.
  

- 
  Addresses #2139: Clarifies metadata statements (vs metadata policies).
  

- 
  Addressed #2143: Additional policy operators that modify metadata parameters must be applied after the value operator. Additional policy operators that check metadata parameters must be applied before the essential operator.
  

- 
  Addressed #2140: The historical keys response JWT uses reason values without referencing the ones defined in the X.509 CRL spec [RFC 5280].
  

- 
  Addressed #1802: Added a definition of "trust" within the introduction, to clarify its meaning within the context of the specification.
  

- 
  Fixed #2082: Allow Trust Chains to start with any Entity Configuration and not just those of Leaf Entities.
  

- 
  Fixed #2123: Stated that all members of a Trust Chain have an equal opportunity to contribute to metadata policies.
  

- 
  Fixed #2148: Require fetch, listing, and trust mark status federation endpoints to be published in Entity Configuration metadata.
  

- 
  Fixed #2149: Stated that entities MUST publish Subordinate Statements about their Immediate Subordinates.
  

- 
  Fixed #2150: Defined `invalid_issuer` error code.
  

- 
  Fixed #2152: Stated that "if not understood MUST be ignored" processing rule applies to federation endpoint request parameters.
  

- 
  Fixed #2153: Made reference to [[RFC6749](#RFC6749){.cite .xref}] normative.
  

- 
  Fixed #2151: Put federation endpoints in a more logical order.
  

-34

- 
  Addressed #2090: Metadata policy: Clarified the order in which the standard policy operators must be applied, the merge process when multiple Subordinate Statements contain metadata_policy claims for a given Entity Type, and that asserted metadata must be applied before the merged metadata policy.
  

- 
  Addressed #2078: Metadata policy: Specified mandatory and optional to support JSON types for each standard policy operator.
  

- 
  Addressed #2087: Metadata policy: Added an explicit description of the principles governing the metadata_policy and common requirements for all policy operators, standard as well as additional operators that maybe defined in future specifications that rely on OpenID Federation 1.0. In particular, stated the principles in regard to the 1) hierarchy of Entities in Trust Chains, 2) the specificity of policies, 3) the granularity of policies, 4) the action types of operators, 5) that metadata policy enforcement is integral to the Trust Chain, 6) the determinism of the metadata policy specification.
  

- 
  Addressed #2116: Metadata policy: Clarified that the metadata_policy is not intended to JSON type check metadata parameters.
  

- 
  Addressed #2135: Metadata policy: The space-separated list of strings exception should apply only to the "scope" oauth_client metadata parameter.
  

- 
  Addressed #2136: Metadata policy: Merges the two metadata policy examples and expands the example to include all standard policy operators
  

- 
  Addressed #2124: Renames the allowed_leaf_entity_types constraint to allowed_entity_types, changes its definition to also apply to Subordinate Entities that are Intermediate Entities, not only Subordinates that are Leaves.
  

- 
  Fixed #2119: Federation metadata_policy example figures and descriptions.
  

- 
  Changed the HTTP method used at the Trust Mark Status Endpoint back to POST because of the potentially large size of the request.
  

- 
  Fixed #2128: The federation_trust_mark_endpoint requires a sub request parameter.
  

- 
  Fixed #2120: Defined and consistently used the terms Subordinate, Immediate Subordinate, Superior, and Immediate Superior.
  

- 
  Fixed #2089: Added section Resolving the Trust Chain and Metadata with a Resolver.
  

- 
  Fixed #2131: JSON in Entity Statement example in figure 2.
  

- 
  Fixed #2088: Clarified tls_client_auth rules.
  

- 
  Fixed #2130: Clarified wording describing common metadata parameters.
  

- 
  Fixed #2133: Made treatment of `jwks` Entity Statement claim consistent when doing Explicit Registration.
  

- 
  Registered `application/jwk-set+jwt` media type.
  

- 
  Registered `nbf` JWK parameter.
  

- 
  Fixed #2132: Stated that Automatic Registration and Explicit Registration can also be used for OAuth 2.0 profiles other than OpenID Connect.
  

- 
  Fixed #2138: Required `kid` (Key ID) header parameter in JWTs.
  

-33

- 
  Addressed #2111: The metadata_policy_crit claim MAY only appear in Subordinate Statements and its values apply to all metadata_policies found in the Trust Chain.
  

- 
  Fixed #2096: Authorization Signed Request Object may contain `trust_chain` in its payload and should not in its JWS header parameters.
  

- 
  Strengthen language requiring client verification with automatic registration.
  

- 
  Fixed #2076: Promoted Trust Marks to be a top-level section.
  

- 
  Added General-Purpose JWT Claims section.
  

- 
  Moved Federation Endpoints section before Obtaining Federation Entity Configuration Information section.
  

- 
  Fixed #2110: Explanatory text when multiple `entity_type` parameters are provided in the Subordinate Listing endpoint.
  

- 
  Fixed #2112, #2113, and #2114: Defined that client authentication is not used by default and that the default client authentication method, when used, is `private_key_jwt`. Specified that requests using client authentication use HTTP POST.
  

- 
  Fixed #2104: Allow trust marks in Subordinate Statements for implementation profiles that might want this.
  

- 
  Fixed #2103: Addressed ambiguities in the definition of constraints.
  

-32

- 
  Tightened OpenID Connect Client Registration section.
  

- 
  Tightened appendix examples.
  

- 
  Fixed #2075: Trust Mark endpoint for the provisioning of the Trust Marks.
  

- 
  Fixed #2085: Trust Marked Entities Listing; added `sub` URL query parameter.
  

- 
  Made fetch issuer unambiguous by making the `iss` parameter REQUIRED.
  

- 
  Introduced the term "Subordinate Statement" and applied it throughout the specification. Also consistently use the term "registration Entity Statement" for Explicit Client Registration results.
  

- 
  Clarified where Entity Statement claims can and cannot occur.
  

- 
  Renamed `policy_language_crit` to `metadata_policy_crit`.
  

- 
  Fixed #2093: Numbered the list defining the order policy operators are applied in.
  

-31

- 
  Fixed #2068: Clarifications that authority_hints must only appear in Leaf and Intermediate Entity Configurations.
  

- 
  Fixed #2062: Verified that metadata examples are consistent with the specification.
  

- 
  Fixed #1724: Numbered figures by adding captions to them.
  

- 
  Fixed #2033: Described the advantage of having the Trust Anchor host the resolve endpoint.
  

- 
  Fixed #1659: Described use of the `trust_chain` header parameter in resolve responses.
  

- 
  Addressed #2060: A concrete order must be specified for the application of all metadata policy operators.
  

- 
  Addressed #2069: Metadata parameters with null value must be treated as error when applying metadata policies.
  

- 
  Fixed #2034: Other use cases of Federation Automatic Registration, FAPI included.
  

- 
  Moved the definitions of `organization_name`, `contacts`, `logo_uri`, `policy_uri`, and `homepage_uri` from the `federation_entity` Entity Type definition to the set of metadata parameters applicable to all Entity Types.
  

- 
  Clarified the role of the Trust Anchor's Entity Configuration in Trust Chains.
  

- 
  Fixed #1751: Provided guidance to prevent stuck clients.
  

- 
  Fixed #2071: Removed most of the Claims Languages and Scripts text duplicating that in OpenID Connect Core.
  

- 
  Tightened descriptions of constraints and Trust Marks.
  

- 
  Tightened descriptions of Federation Endpoints.
  

- 
  Fixed #2080: Defined `federation_historical_keys_endpoint` metadata parameter.
  

- 
  Fixed #2081: Renamed `trust_marks_owners` to `trust_mark_owners` and renamed `trust_marks_issuers` to `trust_mark_issuers`.
  

- 
  Fixed #2079: Renamed the `id` Trust Mark status request parameter to `trust_mark_id` to match the parameter names used in Subordinate listing requests and Trust Marked entities listing requests.
  

-30

- 
  Clarifies that when the metadata policy combination process encounters an error, such as being unable to merge two operators, the policy combination MUST be considered invalid.
  

- 
  Applications and protocols using Entity Statements MAY specify additional claims.
  

- 
  Addressed #1655: The definitions of `aud` and `trust_anchor_id` are moved to the Explicit Client Registration section.
  

- 
  Fixed #1957: Explicit Registration: Clarifies OP processing of Explicit Registration requests with a Trust Chain.
  

- 
  Addressed #1655: Explicit Registration: Simplifies the expression of the registered RP metadata - the metadata policy mechanism is no longer used.
  

- 
  Addressed #1655: Explicit Registration: An RP MUST always verify the metadata with which it was registered. The standard Trust Chain resolution and application of `metadata_policy` is the SHOULD method, but RPs are free to utilize other methods yielding the same result, e.g., by calling a helper web endpoint provided by the Trust Anchor.
  

- 
  Addressed #1655: Explicit Registration: Exact specification of the EC claims used in requests and the ES claims used in responses; normative language where necessary to remove ambiguities and guarantee interop.
  

- 
  Explicit Registration: The `exp` (expiration) of the response ES must be the time when the OP is going to expire the client registration. Intended to inform the RP when its registration is going to expire so that it can re-register in advance and prevent stuck end-users if the registration expires in the middle of an OAuth flow.
  

- 
  Explicit Registration: Replaces the hard requirement that OPs must periodically ensure that a valid Trust Chain for the RP can be established with non-normative text common to both Automatic and Explicit Registration. The Trust Chain expiration MUST be the primary method for OPs to control the validity of registrations.
  

- 
  Resolving Trust Chain and Metadata: Adds a subsection describing the possibility of transient Trust Chain validation errors when the topology of a federation is updated and suggests a retry strategy.
  

- 
  Updated contributor affiliations.
  

- 
  Fixed #1947: Added `source_endpoint` as OPTIONAL in Entity Statement JWS.
  

- 
  Historical keys response: `iat` is OPTIONAL.
  

- 
  Added text on Trust Mark delegation.
  

- 
  Fixed #2041: Added normative language mandating the https scheme and any optional port and path, for all the Entity Identifiers and the federation endpoints.
  

- 
  Cite draft-ietf-oauth-resource-metadata.
  

- 
  Fixed #2002: Added sequence diagrams and representations of the general architecture and the Trust Chain.
  

- 
  Fixed #2061: Clarified that the `constraints` claim may only appear in statements issued by authorities.
  

- 
  Fixed #2062: Require metadata statements for all Entity Type Identifiers that apply to an Entity.
  

- 
  Replaced duplicative term "metadata type" with "Entity Type" and defined the term "Entity Type Identifier".
  

- 
  Added section headings to the Metadata section to more cleanly separate Entity Type Identifiers from Metadata Extensions.
  

- 
  Fixed #2064: Prohibited the use of `null` as a metadata value.
  

- 
  Rewrote `request_authentication_methods_supported` description.
  

- 
  Registered `keys` JWT claim and `iat`, `exp`, and `revoked` JWK parameters.
  

- 
  Fixed #1727: Changed specification name from "OpenID Connect Federation" to "OpenID Federation".
  

-29

- 
  Fixed #1921: brand new JWS Header parameter, `trust_chain`.
  

- 
  Fixed #1879, #1881, #1883, #1885: introductory text, metadata section, editorials.
  

-28

- 
  Fixed #1823: Metadata policy clarification text on essential term and subset_of, one_of, superset_of.
  

- 
  Listing endpoint - added the parameters `trust_marked` and `trust_mark_id`.
  

- 
  Historical keys endpoint, references to rfc7517 and use of claim keys, correction in the normative text.
  

- 
  Added the endpoint `federation_trust_mark_list_endpoint` for Trust Mark issuers.
  

-27

- 
  Fixed #1756: Clarified that the authority_hints in the Entity Statement of an explicit registration response is single-valued.
  

- 
  Added text about the usage of metadata besides metadata_policy.
  

- 
  Fixed #1745 #1746: Non-normative example of Trust Mark status endpoint.
  

- 
  Fixed #1747: Client Registration Response content-type set to application/entity-statement+jwt.
  

- 
  Fixed #1740: Metadata policy example for OpenID Connect, removed scopes.
  

- 
  Fixed #1755: clarification on the key to be used for Federation operations.
  

- 
  The "essential" policy operator can be used in conjunction with one_of, subset_of, superset_of to make their presence optional.
  

- 
  Fixed #1779: Entity Type as defined term.
  

- 
  Added Sequence Diagram representing a Federation Entity Discovery and metadata evaluation.
  

- 
  Fixed #1757: Federation Historical Keys endpoint support for revocation status. Endpoint enabled for all the Federation Entities.
  

- 
  Fixed #1803: Subordinate is a defined term and other small editorial changes after Heather's revision.
  

- 
  Added Cryptographic Trust Mechanism description.
  

- 
  Fixed #1660: Clarified that Explicit Registration uses no parameters and added Explicit Registration example.
  

- 
  Fixed #1782: Reference protected resource metadata draft.
  

- 
  Fixed #1809: Replaced use of "trust issuers" terminology.
  

- 
  Fixed #1661: Tightened descriptions of metadata standards used.
  

- 
  Fixed #1801: Better described function of "essential" metadata operator.
  

-26

- 
  Applied editorial improvements by Heather Flanagan.
  

-25

- 
  Fixed #1684: disambiguation on the role federation_entity.
  

- 
  Fixed #1703: Unsigned error response.
  

- 
  Fixed #1693: Trust mark status endpoint HTTP method changed to POST.
  

- 
  Fixed #1681: `RS256` is not mandatory anymore for federation operations signatures.
  

- 
  Fixed #1701: trust_chain parameter with PAR and redirect_uri.
  

- 
  Fixed #1695: entity_type URL parameter for Listing endpoint.
  

- 
  Fixed #1712: federation_entity claims: organization_name is recommended, logo_uri is added.
  

- 
  Fixed #1702: removal of metadata type trust_mark_issuer, federation_status_endpoint moved in the federation_entity metadata and renamed to federation_trust_mark_status_endpoint.
  

- 
  Fixed #1716: Clarified how to retrieve the RP's Entity Configuration when using Automatic Registration.
  

- 
  Fixed #1656: Introductory text review.
  

-24

- 
  Fixed #1654: Text cleanup on how and who publishes the Entity Configuration.
  

- 
  Relaxed JWK Set representations: jwks, signed_jwks_uri and jwks_uri can coexist in the same Metadata for interoperability purposes across different Federations.
  

- 
  Fixed #1641: Federation Historical Keys endpoint.
  

- 
  Fixed #1673: added resolve endpoint JWT claims.
  

- 
  Fixed #1663: clarifications on the http status codes returning from /.well-known/openid-federation.
  

- 
  Fixed #1667: serialization format for federation endpoint made explicit.
  

- 
  Fixed #1662: ID Token section removed.
  

- 
  Fixed #1680: removal of the claim operation in Generic Errors.
  

- 
  Fixed #1669: error types in the section Generic Error Response.
  

- 
  Added Vladimir Dzhuvinov as an editor.
  

-23

- 
  Text on max_path_length
  

- 
  Fixed #1634: Trust Chain explanatory text.
  

- 
  Federation Entity Discovery as defined term.
  

- 
  Fixed #1645: Federation Entity Keys as defined term.
  

- 
  Fixed #1633: Explanatory text about who signs the Trust Mark and how.
  

- 
  Fixed #1650: typo in signed_jws_uri and jwks metadata claims.
  

-22

- 
  Rewritten text about metadata processing when doing Explicit Client Registration.
  

- 
  Fixed #1581: OP metadata claims enabled for AS.
  

- 
  Fixed #1594: self-issued trust_chain in the Authorization Request.
  

- 
  Fixed #1583: metadata policy one_of operator, explanatory text for multiple values.
  

- 
  Fixed #1584: Stated that domain name constraints are as specified in Section 4.2.1.10 of [[RFC5280](#RFC5280){.cite .xref}].
  

- 
  Added more text on considerations when using the resolve endpoint.
  

- 
  Removal of aud parameter in the fetch endpoint request.
  

- 
  General rewording with the terms Leaf Entity and Entity Configuration.
  

- 
  Fixed #1588: Explicitly described the differences between Automatic and Explicit Registration.
  

- 
  Fixed #1606: Described situation in which the requirement for signed requests with Automatic Registration could be relaxed.
  

- 
  Fixed #1629: authz Request Object, audience explanatory text.
  

- 
  Fixed #1630: Resolve endpoint response content type.
  

- 
  Fixed #1608: submission of the Trust Chain in the Explicit Client Registration Request.
  

-21

- 
  Fixed #1547: Metadata section restructured.
  

- 
  Fixed #1548: request_authentication_signing_alg_values_supported clarification text about the usage of private_key_jwt and request_object.
  

- 
  Added a new constraint, allowed_leaf_entity_types.
  

- 
  Resolve endpoint: Removed iss parameter in the request and specified the usage of the aud claim in the response.
  

- 
  Fixed #1513: request_authentication_methods_supported according to IANA OAuth 2.0 AS metadata registered names.
  

- 
  Fixed #1527: Defined the Trust Mark term and added non-normative examples.
  

- 
  Added Security Considerations regarding the role of the Trust Marks.
  

- 
  Editorial review, normative language, and typos.
  

- 
  Fixed #1535: Removed the sub claim from the non-normative example of the Authorization Request Object.
  

- 
  Fixed #1528: Added Claims Languages and Scripts section.
  

- 
  Fixed #1456: Added language about space-delimited string parameters from RFC 6749.
  

-20

- 
  Updated example for Resolve endpoint.
  

- 
  Recommended that Key IDs be the JWK Thumbprint of the key.
  

- 
  Fixed #1502 - Corrected usage of `id_token_signed_response_alg`.
  

- 
  Fixed #1506 - Referenced RFC 9126 for Pushed Authorization Requests.
  

- 
  Added Giuseppe De Marco as an editor and removed Samuel Gulliksson as an editor.
  

- 
  Fixed #1508 - Editorial review.
  

- 
  Fixed #1505 - Defined that `signed_jwks_uri` is preferred over `jwks_uri` and `jwks`.
  

- 
  Fixed #1514 - Corrected RP metadata examples to use `id_token_signed_response_alg`.
  

- 
  Fixed #1512 - Registered the error codes `missing_trust_anchor` and `validation_failed`.
  

- 
  Also registered the OP Metadata parameters `organization_name`, `request_authentication_methods_supported`, `request_authentication_signing_alg_values_supported`, `signed_jwks_uri`, and `jwks` and the RP Metadata parameters `client_registration_types` (which was previously misregistered as `client_registration_type`), `organization_name`, and `signed_jwks_uri`.
  

- 
  Defined the term Federation Operator and described redundant retrieval of Trust Anchor keys.
  

- 
  Fixed #1519 - Corrected instances of `client_registration_type` to `client_registration_types_supported`.
  

- 
  Fixed #1520 - Renamed `federation_api_endpoint` to `federation_fetch_endpoint`.
  

- 
  Fixed #1521 - Changed swamid.sunet.se to swamid.se in examples.
  

- 
  Fixed #1523 - Added `request_parameter_supported` to examples.
  

- 
  Capitalized defined terms.
  

-19

- 
  Fixed #1497 - Removed `trust_mark` claim from federation entity metadata.
  

- 
  Fixed #1446 - Added `trust_chain` and removed `is_leaf`.
  

- 
  Fixed #1479 - Added `jwks` claim in OP metadata.
  

- 
  Fixed #1477 - Added `request_authentication_methods_supported`.
  

- 
  Fixed #1474 - Added the `request_authentication_signing_alg_values_supported` OP metadata parameter.
  

- 
  Rewrote `trust_marks` definition.
  

-18

- 
  Added the jwks claim name for OP Metadata.
  

- 
  Moved from a federation API to separate endpoints per operation.
  

- 
  Added descriptions of the list, resolve and status federation endpoints.
  

- 
  Registered the `client_registration_types_supported` and `federation_registration_endpoint` authorization server metadata parameters.
  

- 
  Registered the `client_registration_type` dynamic client registration parameter.
  

- 
  Registered and used the `application/entity-statement+jwt` media type.
  

- 
  Registered the `application/trust-mark+jwt` media type.
  

- 
  Trust marks: the claim `mark` has been renamed to `logo_uri`.
  

- 
  New federation endpoint: Resolve Entity Statement.
  

- 
  New federation endpoint: Trust Mark Status.
  

- 
  Federation API, Fetch: `iss` parameter is optional.
  

- 
  Trust Marks: added non-normative examples.
  

- 
  Expanded Section 8.1: Included Entity configurations.
  

- 
  Fixed #1373 - More clearly defined the term Entity Statement, both in the Terminology section and in the Entity Statement section.
  

-17

- 
  Addressed many working group review comments. Changes included adding the Overall Architecture section, the `trust_marks_issuers` claim, and the Setting Up a Federation section.
  

-16

- 
  Added Security Considerations section on Denial-of-Service attacks.
  

-15

- 
  Added `signed_jwks_uri`, which had been lost somewhere along the road.
  

-14

- 
  Rewrote the federation policy section.
  

- 
  What previously was referred to as `client authentication` in connection with automatic client registration is now changed to be `request authentication`.
  

- 
  Made a distinction between parameters and claims.
  

- 
  Corrected the description of the intended behavior when `essential` is absent.
  

-13

- 
  Added `trust_marks` as a parameter in the Entity Statement.
  

- 
  An RP's self-signed Entity Statement MUST have the OP's issuer identifier as the value of `aud`.
  

- 
  Added `RS256` as default signing algorithm for Entity Statements.
  

- 
  Specified that the value of `aud` in the Entity Statement used in automatic client registration MUST be the AS's authorization endpoint URL. Also, the `sub` claim MUST NOT be present.
  

- 
  Separating the usage of merge and combine as proposed by Vladimir Dzhuvinov in Bitbucket issue #1157.
  

- 
  An RP doing only explicit client registration is not required to publish and support `/.well-known/openid-federation`.
  

- 
  Every key in a JWK Set in an Entity Statement MUST have a `kid`.
  

-12

- 
  Made JWK Set OPTIONAL in an Entity Statement returned by OP as response of an explicit client registration
  

- 
  Renamed Entity type to client registration type.
  

- 
  Added more text describing client_registration_auth_methods_supported.
  

- 
  Made "Processing the Authentication Request" into two separate sections: one for Authentication Request and one for Pushed Authorization Request.
  

- 
  Added example of URLs to some examples in the appendix.
  

- 
  Changed the automatic client registration example in the appendix to use Request Object instead of a client_assertion.
  

-11

- 
  Added section on Trust Marks.
  

- 
  Clarified private_key_jwt usage in the authentication request.
  

- 
  Fixed Bitbucket issues #1150 and #1155 by Vladimir Dzhuvinov.
  

- 
  Fixed some examples to make them syntactically correct.
  

-10

- 
  Incorporated additional review feedback from Marcos Sanz. The primary change was moving constraints to their own section of the Entity Statement.
  

-09

- 
  Incorporated review feedback from Marcos Sanz. Major changes were as follows.
  

- 
  Separated Entity configuration discovery from operations provided by the federation API.
  

- 
  Defined new authentication error codes.
  

- 
  Also incorporated review feedback from Michael B. Jones.
  

-08

- 
  Incorporated review feedback from Michael B. Jones. Major changes were as follows.
  

- 
  Deleted `sub_is_leaf` Entity Statement since it was redundant.
  

- 
  Added `client_registration_type` RP registration metadata value and `client_registration_types_supported` OP metadata value.
  

- 
  Deleted `openid_discovery` metadata type identifier since its purpose is already served by `openid_provider`.
  

- 
  Entity identifier paths are now included when using the Federation API, enabling use in multi-tenant deployments sharing a common domain name.
  

- 
  Renamed `sub_is_leaf` to `is_leaf` in the Entity Listings Request operation parameters.
  

- 
  Added `crit` and `policy_language_crit`, enabling control over which Entity Statement and policy language extensions MUST be understood and processed.
  

- 
  Renamed `openid_client` to `openid_relying_party`.
  

- 
  Renamed `oauth_service` to `oauth_authorization_server`.
  

- 
  Renamed `implicit` registration to `automatic` registration to avoid naming confusion with the implicit grant type.
  

- 
  Renamed `op` to `operation` to avoid naming confusion with the use of "OP" as an acronym for "OpenID Provider".
  

- 
  Renamed `url` to `uri` in several identifiers.
  

- 
  Restored Open Issues appendix.
  

- 
  Corrected document formatting issues.
  

-07

- 
  Split metadata into metadata and metadata_policy
  

- 
  Updated example
  

-06

- 
  Some rewrites
  

- 
  Added example of explicit client registration
  

-05

- 
  A major rewrite.
  

-04

- 
  Changed client metadata names `scopes` to `rp_scopes` and `claims` to `rp_claims`.
  

- 
  Added Open Issues appendix.
  

- 
  Added additional references.
  

- 
  Editorial improvements.
  

- 
  Added standard Notices section, which is present in all OpenID specifications.
  





## Acknowledgements

The authors wish to acknowledge the contributions of the following individuals and organizations to this specification: Marcus Almgren, Pasquale Barbaro, Brian Campbell, David Chadwick, Michele D'Amico, Andrii Deinega, Erick Domingues, Heather Flanagan, Michael Fraser, Samuel Gulliksson, Joseph Heenan, Pedram Hosseyni, Marko Ivančić, Łukasz Jaromin, Takahiko Kawasaki, Torsten Lodderstedt, Francesco Marino, John Melati, Alexey Melnikov, Eduardo Perottoni, Roberto Polli, Justin Richer, Jouke Roorda, Nat Sakimura, Mischa Sallé, Stefan Santesson, Marcos Sanz, Peter Brand, Michael Schwartz, Giada Sciarretta, Amir Sharif, Sean Turner, Niels van Dijk, Tim Würtele, Kristina Yasuda, Gabriel Zachmann, and the JRA3T3 task force of GEANT4-2.





## Authors' Addresses


[Roland Hedberg ([editor]{.role})]{.fn .nameRole}



[independent]{.org}



Email: <roland@catalogix.se>



[Michael B. Jones]{.fn .nameRole}



[Self-Issued Consulting]{.org}



Email: <michael_b_jones@hotmail.com>



URI: [https://self-issued.info/](https://self-issued.info/){.url}



[Andreas Åkre Solberg]{.fn .nameRole}



[Sikt]{.org}



Email: <Andreas.Solberg@sikt.no>



URI: [https://www.linkedin.com/in/andreassolberg/](https://www.linkedin.com/in/andreassolberg/){.url}



[John Bradley]{.fn .nameRole}



[Yubico]{.org}



Email: <ve7jtb@ve7jtb.com>



URI: [http://www.thread-safe.com/](http://www.thread-safe.com/){.url}



[Giuseppe De Marco]{.fn .nameRole}



[independent]{.org}



Email: <demarcog83@gmail.com>



URI: [https://www.linkedin.com/in/giuseppe-de-marco-bb054245/](https://www.linkedin.com/in/giuseppe-de-marco-bb054245/){.url}



[Vladimir Dzhuvinov]{.fn .nameRole}



[Connect2id]{.org}



Email: <vladimir@connect2id.com>



URI: [https://www.linkedin.com/in/vladimirdzhuvinov/](https://www.linkedin.com/in/vladimirdzhuvinov/){.url}



