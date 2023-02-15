---
title: Amazon Ion Hash
permalink: /
---
Amazon Ion Hash defines an algorithm for constructing a hash for any Ion value.
For a given Ion value and consistent hash function, the algorithm guarantees
hashing the value will always produce the same hash, independent of the value's
encoding (text or binary).  The hash function to use is not declared by the
specification&mdash;this enables the user to select the hash function most appropriate
to their use case.

Ion hash is useful when determining whether two Ion values represent the same value,
or determining whether an Ion value has changed.  For example, a storage system might
use Ion hashes to assert the integrity of its data.

For more information, see the [Ion Hash Specification][1].

<br/>

### Latest News

---
{% for post in site.posts limit:3 %}
  **<a href="{{site.baseurl}}{{post.url}}">{{ post.title }}</a>**<br/>
  *{{post.date | date_to_long_string}}*<br/>
  {{post.content}}
{% endfor %}
---
Visit the [News][2] page for more announcements related to Ion Hash.

<br/>

<!-- References -->
[1]: docs/spec.html
[2]: news.html
[3]: https://amazon-ion.github.io/ion-docs/

