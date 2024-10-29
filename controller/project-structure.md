# Project structure

The `/app/` folder contains on its root level namespaces belonging to external
n2n-modules installed by composer or project specific bounded contexts
following the example set by Domain Driven Design

## Bounded Contexts

The namespace of a Bounded Context could be `volagic` for example and its
contents would be placed in `/app/volagic/`. Each bounded context must be seen
as an isolated part of the application. Each Bounded Context has its own
database and must only communicate with other Bounded Contexts over the
message broker. This way they could be spilt easily into multiple n2n instances
and run in different containers when necessary.

The namespace of the Bounded Context usually has two sub namespaces `biz`
for the business logic and `site` which contains api controllers and user
interfaces. `biz` does not contain any controllers for example. These will be
placed in `site`.

### Business logic namespace

Contents of the business logic namespace in the example above would be placed
in `/app/volagic/biz/`. `biz` can have an unlimited number of sub
namespaces which can help as logical separation of the functionality. We wil
call them "topic namespaces" in this article.

#### Structure of a biz topic namespace

Topic namespace could be named `org` for example and its contents would be
placed in `/app/volagic/biz/org/`. A topic namespace may contain the following
namespaces and all its classes should be documented and defined in the uml
except those of the namespace `model`.

- `bo` contains business objects like entities. May be accessed from the whole
  `biz` namespace but it's encouraged to access its contents only from its topic
  namespace.

- `listener` contains EntityListeners which a registered on entities preferably from
  the same topic namespace but never on entities from outside its `biz`
  namespace.

- `facade` contains services and other classes. This is the only namespace which
  may be accessed from its whole bounded context. Most services should have the
  ending `Facade` (e.g. `OrganisationFacade`).

- `accord` contains documented rules, services, bind tasks and other logic.
  Its contents should be mainly accessed by its topic namespace but are allowed
  to be accessed by its whole `biz` namespace.

- `model` contains undocumented logic (e.g. Data Access Objects). Its contents
  **must only be accessed from its topic namespace**.

- `batch` contains batch jobs which shouldn't be accesses from anywhere.

### Site namespace

Contents of the site namespace in the example above would be placed
in `/app/volagic/site/`. As in `biz` the `site` namespace can have an unlimited
number of  sub  namespaces which also can help as logical separation of its
functionality. We will also call them "topic namespaces" in this article.


#### Structure of a site topic namespace

Each topic namespace may contain an independent website, web app, subdomain,
etc. or whatever makes sense to be logically separated. A topic namespace could
be named `admin` for example and its contents would be placed in
`/app/volagic/site/admin/`. A topic namespace of `site` may contain the same
sub namespaces as in a `biz` namespace. The same rules do apply also but with
namespace `biz` replaced with `site` when it is mentioned the documentation
above.

There are two namespaces however which should only exists in a site topic
namespace:

- `controller` contains controllers which access rules are not yet definitely
  defined.

- `view` contains view files which must be only accessed by its topic namespace.
