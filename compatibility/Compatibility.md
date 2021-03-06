#Compatibility

## {{ book.must }} Don’t Break Backward Compatibility

Change APIs, but keep all consumers
running. Consumers usually have independent release lifecycles, focus on
stability, and avoid changes that do not provide additional value. APIs are
service contracts that cannot be broken via unilateral decisions.

There are two techniques to change APIs without breaking them:

- follow rules for compatible extensions
- introduce new API versions and still support older versions

We strongly encourage using compatible API extensions and discourage versioning.
With Postel’s Law in mind, here are some rules for providers and consumers that
allow us to make compatible changes without versioning:

## {{ book.should }} Prefer Compatible Extensions

Apply the following rules to evolve RESTful APIs in a backward-compatible way:

* Ignore unknown fields in the payload
* Add optional, non mandatory fields
* Never change and only extend a field’s meaning
* Enum ranges cannot not be extended when used for output parameters — clients
  may not be prepared to handle it. However, enum ranges can be extended when
  used for input parameters.
* Enum ranges can be reduced when used as input parameters, only if the server
  is ready to accept and handle old range values too. Enum values can be reduced
  when used as output parameters.
* Use x-extensible-enum, if range is used for output parameters and likely to
  be extended with growing functionality. It defines an open list of explicit
  values and clients must be agnostic to new values (e.g. via casuistic with
  default behavior).
* Support redirection in case an URL has to change
 ([301 Moved Permanently](https://en.wikipedia.org/wiki/HTTP_301))


## {{ book.must }} Prepare Clients for Compatible API Extensions (the Robustness Principle)

How to do this:

* Ignore new and unknown fields in the payload (see also Fowler’s
  “[TolerantReader](http://martinfowler.com/bliki/TolerantReader.html)” post)
* Be prepared for new enum values declared with x-extensible-enum (see above);
  provide default behavior for unknown values, if applicable
* Follow the redirect when the server returns an “HTTP 301 Moved Permanently” response code

## {{ book.should }} Avoid Versioning

When changing your RESTful APIs,
do so in a compatible way and avoid generating additional API versions. Multiple
versions can significantly complicate understanding, testing, maintaining,
evolving, operating and releasing our systems ([supplementary reading](http://martinfowler.com/articles/enterpriseREST.html)).

If changing an API can’t be done in a compatible way, then proceed in one of these
three ways:

* create a new resource (variant) in addition to the old resource variant
* create a new service endpoint — i.e. a new microservice application with a new API (variant)
* create a new API version supported in parallel with the old API by the same microservice

Avoiding versioning also means preferring the first two variants.

## {{ book.must }} Use Media Type Versioning

When API versioning is unavoidable, you have to design your multi-version
RESTful APIs using media type versioning (instead of URI versioning, see below).
Media type versioning is less tightly coupled since it supports content
negotiation and hence reduces complexity of release management.

Media type versioning: Here, version information and media type are provided
together via the HTTP Content-Type header — e.g.
application/x.zalando.cart+json;version=2. For incompatible changes, a new
media type version for the resource is created. To generate the new
representation version, consumer and producer can do content negotiation using
the HTTP Content-Type and Accept headers. Note: This versioning only applies to
the content schema, not to URI or method semantics.

In this example, a client wants only the new version:

    Content-Type: application/x.zalando.cart+json;version=2
    Accept: application/x.zalando.cart+json;version=2

Using header versioning should:

* include versions in request and response headers to increase visibility
* include Content-Type in the Vary header to enable proxy caches to differ between versions

Hint: [OpenAPI currently doesn’t support content
negotiation](https://github.com/OAI/OpenAPI-Specification/issues/146), though [a comment in this
issue](https://github.com/OAI/OpenAPI-Specification/issues/146#issuecomment-117288707) mentions
a workaround (using a fragment identifier that
gets stripped off).

## {{ book.must }} Do Not Use URI Versioning

With URI versioning a (major) version number is included in the path, e.g.
/v1/customers. The consumer has to wait until the provider has been released
and deployed. If the consumer also supports hypermedia links — even in their
APIs — to drive workflows (HATEOAS), this quickly becomes complex. So does
coordinating version upgrades — especially with hyperlinked service
dependencies — when using URL versioning. To avoid this tighter coupling and
complexer release management we do not use URI versioning, and go instead with
media type versioning and content negotiation (see above).

## {{ book.should }} Provide Version Information in OpenAPI Documentation

Only the documentation, not the API itself, needs version information.

Example:

    "swagger": "2.0",
    "info": {
      "title": "Parcel service API",
      "description": "API for <...>",
      "version": "1.0.0",
        <...>
    }

During a (possibly) long-running API review phase you need different versions
of the API description. These versions may include changes that are incompatible
with earlier draft versions. So we apply the following version schema
MAJOR.MINOR.DRAFT that increments the...

* MAJOR version, when you make incompatible API changes
* MINOR version, when you add functionality in a backwards-compatible manner
* DRAFT version, when you make changes during the review phase that are not
  related to production releases

We recommend using the DRAFT version only for unreleased API definitions that
are still under review; for example:

    version 1.4.0  -- current version
    version 1.4.1  -- first draft and call for review of API extensions compatible with 1.4.0
    version 1.4.2  -- second draft and call for review of API extensions that are still compatible
                       with 1.4.0 but possibly incompatible with 1.4.1
    version 1.5.0  -- approved version for implementation and release
    version 1.5.1  -- first draft for next review and API change cycle;
                       compatible with 1.4.0 and 1.5.0

Hint: This versioning scheme differs in the less strict DRAFT aspect from
[semantic version information](http://semver.org) used for released APIs and
service applications.
